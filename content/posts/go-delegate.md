---
title: "請協助做好委派的工作"
date: 2022-07-02T10:42:50+08:00
draft: false
ShowToc: true
tags: ["程式設計"]
categories: ["Programming paradigm"]
---

之前介紹過 Prototype-based programming，Golang 利用委派 delegate 也能做到。

## 委派的範例
```go
type Widget struct {
	X, Y int
}

type Label struct {
	Widget
	Text string
	X    int
}

func (label Label) Paint() {
	fmt.Printf("[%p] - Label.Paint(%q)\n", &label, label.Text)
}
```
上面這段程式碼有幾個重點：
- 定義了 struct，Widget
- 跟 C 語言一樣，把 struct 放到另一個 struct 中，定義了 Label，把 Widget 委派進去
- Label 實作了 Paint() 方法

接著可以這樣使用：
```go
label := Label{Widget{10, 10}, "State", 100}

fmt.Printf("X=%d, Y=%d, Text=%s, Widget.X=%d\n",  // X=100, Y=10, Text=State, Widget.X=10
		label.X, label.Y, label.Text,
		label.Widget.X)

fmt.Printf("%+v\n%v\n", label, label) // {Widget:{X:10 Y:10} Text:State X:100}
	                                  // {{10 10} State 100}

label.Paint() // [0xc0000a81e0] - Label.Paint("State")
```
可以發現衝突的屬性名稱 X，需要手動處理。

再來繼續擴充程式，新增一個 Button：
```go
type Button struct {
	Label
}

func NewButton(x, y int, text string) Button {
	return Button{Label{Widget{x, y}, text, x}}
}

// override
func (button Button) Paint() {
	fmt.Printf("[%p] - Button.Paint(%q)\n",
		&button, button.Text)
}

func (button Button) Click() {
	fmt.Printf("[%p] - Button.Click()\n", &button)
}
```

再新增一個 ListBox：
```go
type ListBox struct {
	Widget
	Texts []string
	Index int
}

func (listBox ListBox) Paint() {
	fmt.Printf("[%p] - ListBox.Paint(%q)\n",
		&listBox, listBox.Texts)
}

func (listBox ListBox) Click() {
	fmt.Printf("[%p] - ListBox.Click()\n", &listBox)
}
```

然後定義二個介面，用來多型：
```go
type Painter interface {
	Paint()
}

type Clicker interface {
	Click()
}
```

接著就能泛型的使用：
```go
label := Label{Widget{10, 10}, "State", 100}
button1 := Button{Label{Widget{10, 20}, "OK", 10}}
button2 := NewButton(20, 40, "Exit")
listBox := ListBox{Widget{10, 20},
[]string{"AA", "BB", "CC", "DD"}, 0}

// [0xc000012270] - Label.Paint("State")
// [0xc0000122a0] - Button.Paint("OK")
// [0xc0000122d0] - Button.Paint("Exit")
// [0xc000012300] - ListBox.Paint(["AA" "BB" "CC" "DD"])
for _, painter := range []Painter{label, button1, button2, listBox} {
	painter.Paint()
}

// [0xc0000123f0] - Button.Click()
// [0xc000012420] - Button.Click()
// [0xc000012450] - ListBox.Click()
for _, widget := range []Painter{label, button1, button2, listBox} {
    if clicker, ok := widget.(Clicker); ok {
	    clicker.Click()
    }
}
```

以上就是 Go 中的委派與介面多型的寫法，並融合了原型設計。

## 重構委派的範例

先來看一個資料容器的範例，其中有 Add()、Delete()、Contains()、轉字串的方法：
```go
type IntSet struct {
	data map[int]bool
}

func NewIntSet() IntSet {
	return IntSet{make(map[int]bool)}
}

func (set *IntSet) Add(x int) {
	set.data[x] = true
}

func (set *IntSet) Delete(x int) {
	delete(set.data, x)
}

func (set *IntSet) Contains(x int) bool {
	return set.data[x]
}

func (set *IntSet) String() string {
	if len(set.data) == 0 {
		return "{}"
	}
	ints := make([]int, 0, len(set.data))
	for i := range set.data {
		ints = append(ints, i)
	}
	sort.Ints(ints)
	parts := make([]string, 0, len(ints))
	for _, i := range ints {
		parts = append(parts, fmt.Sprint(i))
	}
	return "{" + strings.Join(parts, ",") + "}"
}
```

使用方法如下：
```go
ints := NewIntSet()
for _, i := range []int{1, 3, 5, 7} {
    ints.Add(i)
    fmt.Println(ints)
}

for _, i := range []int{1, 2, 3, 4, 5, 6, 7} {
    fmt.Print(i, ints.Contains(i), " ")
    ints.Delete(i)
    fmt.Println(ints)
}
```

重點來了，我們需要擴充一個 Undo 的還原功能，方法如下：
```go
type UndoableIntSet struct {
	IntSet
	functions []func()
}

func NewUndoableIntSet() UndoableIntSet {
	return UndoableIntSet{NewIntSet(), nil}
}

// override
func (set *UndoableIntSet) Add(x int) {
	if !set.Contains(x) {
		set.data[x] = true
		set.functions = append(set.functions, func() { set.Delete(x) })
	} else {
		set.functions = append(set.functions, nil)
	}
}

// override
func (set *UndoableIntSet) Delete(x int) {
	if set.Contains(x) {
		delete(set.data, x)
		set.functions = append(set.functions, func() { set.Add(x) })
	} else {
		set.functions = append(set.functions, nil)
	}

}

func (set *UndoableIntSet) Undo() error {
	if len(set.functions) == 0 {
		return errors.New("No functions to undo")
	}
	index := len(set.functions) - 1
	if function := set.functions[index]; function != nil {
		function()
		set.functions[index] = nil // free closure for garbage collection
	}
	set.functions = set.functions[:index]
	return nil
}
```

可以如下使用 Undo() 功能：
```go
ints := NewUndoableIntSet()
for _, i := range []int{1, 3, 5, 7} {
	ints.Add(i)
	fmt.Println(ints)
}

for _, i := range []int{1, 2, 3, 4, 5, 6, 7} {
	fmt.Print(i, ints.Contains(i), " ")
	ints.Delete(i)
	fmt.Println(ints)
}

for {
	if err := ints.Undo(); err != nil {
		break
	}
	fmt.Println(ints)
}
```

為了 Undo 功能，NewUndoableIntSet 幾乎重寫了 IntSet 和「寫」有關的所有方法，但是，別的類別可能也需要 Undo，難道每個類別都必須重寫嗎?

嘗試使用之前幾篇文章提到的泛型、函式語言程式設計、IoC 控制反轉來解決這個問題。

首先我們需要做幾件事情：
- 定義一個 Undo[] 的函式陣列（當成 stack 用）
- 實做通用的 Add() 方法；用函式指標，並把指標存到 Undo[] 陣列中
- Undo() 函式中，循環 Undo[] 陣列，並執行，執行完做 stack pop

```go
type Undo []func()

func (undo *Undo) Add(function func()) {
	*undo = append(*undo, function)
}

func (undo *Undo) Undo() error {
	functions := *undo
	if len(functions) == 0 {
		return errors.New("No functions to undo")
	}
	index := len(functions) - 1
	if function := functions[index]; function != nil {
		function()
		functions[index] = nil
	}
	*undo = functions[:index]
	return nil
}
```

接著 IntSet 可以改成：
```go
type IntSet struct {
	data map[int]bool
	undo Undo
}

func NewIntSet() IntSet {
	return IntSet{data: make(map[int]bool)}
}

func (set *IntSet) Add(x int) {
	if !set.Contains(x) {
		set.data[x] = true
		set.undo.Add(func() { set.Delete(x) })
	} else {
		set.undo.Add(nil)
	}
}

func (set *IntSet) Delete(x int) {
	if set.Contains(x) {
		delete(set.data, x)
		set.undo.Add(func() { set.Add(x) })
	} else {
		set.undo.Add(nil)
	}
}

func (set *IntSet) Contains(x int) bool {
	return set.data[x]
}

func (set *IntSet) Undo() error {
	return set.undo.Undo()
}
```
其中 Add()、Delete()、Undo() 函式中都使用 Undo 介面。

從原本的流程中抽象出 Undo (Control、how to do)，至於實際的業務邏輯 (Logic、what to undo)，則交給 struct 自己負責；IntSet.Add() 的 undo 是 set.Delete()；而 set.Delete(x) 的 undo 是 set.Add(x)。

這和 [C++ 的泛型](../generic-programming/#更多的抽象)有點像，也和 map、reduce、filter 把控制邏輯、業務邏輯分開的概念類似。

另外從一開始的 UndoableIntSet 包裝 IntSet 類別，到反過來 IntSet 依賴 Undo 類別，正是 IoC 控制反轉。