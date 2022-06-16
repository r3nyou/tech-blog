---
title: "Functional programming 到底是什麼鬼"
date: 2022-05-15T21:21:19+08:00
draft: false
ShowToc: true
tags: ["程式設計"]
categories: ["Programming paradigm"]
---

在[這次是對泛型 Generic programming 的思考](../generic-programming)文中，講了泛型的本質就是對型別的抽象，這篇文將介紹更抽象的程式設計方法，函式語言程式設計(Functional programming) 到底是甚麼鬼。

## 函式語言程式設計
Functional programming 的理念來自數學的代數。
```
f(x)=3x^2+x+1
g(x)=2f(x)+4=6x^2+2x+6
h(x)=f(x)+g(x)=9x^2+3x+7
```
f(x) 是函式，g(x) 是把 f(x) 帶入並展開，h(x) 則是二個函式組成的二元函式。

函式還能做遞迴，例如費式數列
```
f(x)=f(x-1)+f(x-2)
```

Functional programming 就像數學表達式，定義輸入資料和輸出資料的關係，是一種 mapping 的概念。

Functional programming 的特點:
- stateless: 無狀態，重複執行的話，函式內部的資料是不變的
- immutable: 不改動輸入資料，每次都返回新的資料

優點
- 沒有狀態，所以可以瘋狂並行(Parallelism)
- 寫程式可以複製貼上，無狀態，不用擔心全域變數
- 惰性求值
- 確定性 f(x)=y 永遠都有同樣的結果

缺點
- 大量複製資料

## Functional programming 的技巧
- first class function: 函數可以當變數用，可以傳遞、返回、嵌套
- recursive: 簡化程式碼，把複查的問題，能夠簡單的描述
- tail recursion: 避免 function stack 過多，需要編譯器有支援
- map、reduce
- pipeline: 把函式放到一個陣列、list 中，資料會依照順序被各個函式操作，最終得到結果
- high order function: 把函式當參數，把傳入的函式做封裝，然後返回封裝後的函式
- currying: 簡化函式參數，把函式多個參數分解成多個函式封裝，每層函式都返回一個函式去接收下一個參數

舉個例子說明 stateless，下面的程式是有狀態的，因為是對全域變數做 ++，多執行緒是不安全的。
```c
int cnt;
void increment() {
	cnt++;
}
```

改成用參數傳遞，並行執行不需要鎖
```c
int increment(int cnt) {
	return cnt+1;
}
```

另一個 python 的範例
```python
def inc(x):
	def incx(y):
		return x+y;
    return incx;

inc1 = inc(1);
inc2 = inc(2);

print(inc1(3)); # 4
print(inc2(4)); # 6
```
inc() 返回另一個函式 incx()，可以用 inc() 來做各種版本的 inc 函式，這就是上面提到的 currying 技巧。

計算過程抽象，函式可以是變數，去描述一個值；函式也可以是參數、返回值，關注的是表達式，是問題的描述，是 what to do，而不是 how to do。

## Functional programming 思考方式
Functional programming 思考是 descirbe what to do, rather than how to do，平常習慣的 procedure 的寫法稱為 Imperative programming，而 Functional programming 被稱為 Declarative porgramming。

舉個例子來看看傳統寫法與函式語言的差別；比如現在有 3 隻馬，每隻有 5 次機會往前跑，每次往前的機率是 50%。

Imperative programming 寫法如下(Python)：
```python
from random import random
time = 5
horse_pos = [1,1,1]

while time:
    time -= 1
    
    print('')
    for i in range(len(horse_pos)):
        if random() > 0.5:
            horse_pos[i] += 1
        
        print('-' * horse_pos[i])
```

把嵌套的迴圈重構成幾個函式，比較容易閱讀。
```python
from random import random

def run_step():
    global time
    time -= 1
    move_horse()

def move_horse():
    for i in range(len(horse_pos)):
        if random() > 0.5:
            horse_pos[i] += 1

def draw():
    print('')
    for pos in horse_pos:
        draw_horse(pos)

def draw_horse(pos):
    print('-' * pos)

time = 5
horse_pos = [1,1,1]

while time:
    run_step()
    draw()
```
上面的程式從主要的循環中，可以清楚看到有 run_step()、draw() 二個主要邏輯，各邏輯被各自封裝成函式，抽象了計算過程，於是程式碼變成自我解釋的。

但這段程式有個問題，封裝成函式後，這些函式都依賴共享的變數來同步狀態，於是在讀程式碼時，當函式內部訪問某個全域變數，我們就要去查這個變數的上下文，然後心算這個變數的狀態，才能知道程式的邏輯；相當於這些函式必須要知道其他函式怎麼修改共用變數的，所以這些函式是有狀態的。

有狀態的程式，無論是對並行計算、程式碼重用性，都是缺點；那讓我們來看 Functional programming 的寫法。

```python
from random import random

def race(state):
    draw(state)
    if state['time']:
        race(run_step(state))
        
def run_step(state):
    return {
        'time': state['time'] - 1,
        'horse_pos': move_horse(state['horse_pos'])
    }

def move_horse(horse_pos):
    return [x+1 if random() > 0.5 else x for x in horse_pos]

def draw(state):
    print('')
    print('\n'.join(map(draw_horse, state['horse_pos'])))

def draw_horse(pos):
    return '-' * pos

race({'time': 5, 'horse_pos': [1, 1, 1]})
```
一樣把邏輯封裝成函式，且都是 functional programming 的設計，注意以下幾點
- 函式間不再共享變數，改用參數、返回值來傳遞資料
- 函式內沒有 temporary variable
- for 迴圈被遞迴取代
- 程式碼更接近於描述問題本身

## Map、Reduce、Filter
現在很多語言都支援 Map、Reduce、Filter，這三個函式的概念也是源於 functional programming。

來看一個範例，把字串陣列都轉成小寫，平常的寫法如下
```python
upcase = ['HELLO', 'FUNCTIONAL', 'PROGRAMMING']
lowcase = []
for i in range(len(upcase)):
	lowcase.append(upcase[i].lower())
```

如果用 Map 寫成 Functional programming
```python
def toLower(item):
	return item.lower();
	
lowcase = map(toLower, ['HELLO', 'FUNCTIONAL', 'PROGRAMMING'])
```
toLower() 單純對傳進來的參數做操作，然後在 map 函式就能很清楚描述做了什麼，而不像平常的寫法，要去讀迴圈裡面到底做了什麼。

再舉另一個例子。
```python
num = [1, 3, -2, 0, 9, -3, 5, 8, 2, -4]
positive_cnt = 0
positive_sum = 0
for i in range(len(num)):
    if num[i] > 0:
        positive_cnt += 1
        positive_sum += num[i]
    	
if positive_cnt > 0:
	avg = positive_sum / positive_cnt

print(avg)
```
這次稍微難讀了一點，目的是只計算正數的平均值。

再來看 filter、reduce 的版本。
```python
positive_num = filter(lambda x: x>0, num)
avg = reduce(lambda x,y: x+y, positive_num) / len(positive_num)
```
先 filter x>0 的資料並返回新的陣列 positive_num，然後對 positive_num 進行求和 x+y 的 reduce 操作。

map、reudce 不管輸入的資料是什麼，只單純控制「怎麼做」what to do(控制邏輯)，而 lambda 則是描述「做什麼」what to do(業務邏輯)，我們把控制邏輯藏在 map、reduce 中，讓程式只剩下業務邏輯，使程式碼更容易理解。

## pipeline 模式
pipeline 源於 unix shell，把多個命令串起來，前面命令的輸出成為後面命令的輸入，完成流式計算。比如 shell command
```
ps aux | awk '{print $2}' | sort -n | xargs echo
```
這個命令會把所有的 process ID 取出來，排序，顯示。

如果類比成函式的方式，就是多層的嵌套。
```
xargs(echo, sort(n, awk('print $2', ps aux)))
```

順帶一提，這些函式就好比 micro service 的服務編排一樣，這些高大上技術的身上，往往能看到古老的經典思想，例如 unix；很多分布式的架構，都借鑒了傳統微觀的技術，所以理論是相通的，只是工程上有不同實踐。

那接下來看一個 Python 的例子，這段程式有三個步驟
1. 找偶數
2. 乘以 3
3. 轉成字串返回

傳統的非函式寫法:
```python
def process(num):
    if num % 2 != 0:
        return
    num = num * 3
    return 'Number: %s' % num
    
nums = [1,2,3,4,5,6,7,8,9]

for num in nums:
    print(process(num))

# None
# Number: 6
# None
# Number: 12
# None
# Number: 18
# None
# Number: 24
# None
```
這段程式有幾個問題
1. 偶數會輸出 None
2. process() 的程式碼不好讀

接下來把程式改成 pipeline。

首先把三個步驟寫成子函式
```python
def even_filter(nums):
    for num in nums:
        if num % 2 == 0:
            yield num

def multiple_by_three(nums):
    for num in nums:
        yield num * 3

def convert_to_string(nums):
    for num in nums:
        yield 'Number: %s' % num
```

再來把函式串起來用
```python
nums = [1,2,3,4,5,6,7,8,9]

pipeline = convert_to_string(multiple_by_three(even_filter(nums)))

for num in pipeline:
    print(num)
```
上面用到 yield 關鍵字，是 lazy evluation 的概念，函式返回一個 generator，可以被迭代的物件，而沒有真的去執行函式，等到 generator 被迭代時，yield 函式才會真的執行，每次執行會停在 yield 的語句，等下一次迭代。

再來改成用 Map & Reduce(迴圈只能處理順序型的資料結構)
```python
def even_filter(nums):
    return filter(lambda x: x%2==0, nums)

def multiple_by_three(nums):
    return map(lambda x: x*3, nums)

def convert_to_string(nums):
    return map(lambda x: 'Number: %s' % x, nums)
    
nums = [1,2,3,4,5,6,7,8,9]

pipeline = convert_to_string(multiple_by_three(even_filter(nums)))

for num in pipeline:
    print(num)
```

程式碼更容易讀了，但必須用嵌套的方式使用 pipeline，理想的方式要能像下面這樣
```
pipeline_func(nums, [
	even_filter,
	multiple_by_three,
	convert_to_string
])
```
其實就是 reduce 多個函式
```python
def pipeline_func(data, fns):
	return reduce(lambda a, x: x(a), fns, data)
```

可以用 Python 的 force()、裝飾模式(decorator)，寫的更像 pipeline
```python
class Pipe(object):
    def __init__(self, func):
        self.func = func
    
    def __ror__(self, other):
        def generator():
            for obj in other:
                if obj is not None:
                    yield self.func(obj)
        return generator()

@Pipe
def even_filter(num):
    return num if num % 2 == 0 else None

@Pipe
def multiple_by_three(num):
    return num * 3

@Pipe
def convert_to_string(num):
    return 'Number: %s' % num
    
@Pipe
def echo(item):
    print(item)
    return item

def force(sqs):
    for item in sqs: pass
    
nums = [1,2,3,4,5,6,7,8,9]

force(nums | even_filter | multiple_by_three | convert_to_string | echo)
```
上面範例中，利用了 Python 的 Decorator、Generator 功能，把多個函式組合成 pipeline，接下來會介紹裝飾模式(Decorator) 如何運作。

## Python 的裝飾模式 Decorator

Python 的 Decorator 使用上類似 Java 的 Annoration 類似，但相對更單純，簡單說就是 Functional porgramming 的技巧，「用一個函式來構造另一個函式」。

來看看 Hello, world 範例。
```python
def hello(fn):
    def wrapper():
        print("hello, %s" % fn.__name__)
        fn()
        print("bye, %s" % fn.__name__)
    return wrapper
    
@hello
def Marcus():
    print("I am Marcus")

Marcus()

# result:
# hello, Marcus
# I am Marcus
# bye, Marcus
```
上面程式的重點：
1. Marcus() 前面有個 @hello 註解，就是前面定義的 hello()
2. hello() 中需要一個 fn 參數，用來做 callback
3. hello() 返回一個 inner 函式 wrapper()，wrapper() 呼叫了傳進來的 fn，並在 callback 前後加了另外二行

使用某個 @decorator 裝飾某個函式時，Python 解釋器會做下面範例的轉換
```python
@decorator
def func():
	pass

# 轉換成
func = decorator(func)
```
這不只是把 func 當成參數傳到 decorator 中，再進行 callback 而已，還把 decorator 的返回值指派給原本的 func。

再來看一個帶參數的 decorator 範例。
```python
def makeHtmlTag(tag, *args, **kwds):
    def real_decorator(fn):
        css_class = " class='{0}'".format(kwds["css_class"]) if "css_class" in kwds else ""
        
        def wrapper(*args, **kwds):
            return "<"+tag+css_class+">" + fn(*args, **kwds) + "</"+tag+">"
        return wrapper
    return real_decorator

@makeHtmlTag(tag="b", css_class="bold_css")
@makeHtmlTag(tag="i", css_class="italic_css")
def hello():
    return "hello, world"
    
print(hello())

# <b class='bold_css'><i class='italic_css'>hello, world</i></b>
```
為了讓 makeHtmlTag 帶二個參數，所以先返回一個 decorator，就可以做到 hello = makeHtmlTag(arg1, arg2)(hello)；然後進到真正 decorator 的邏輯，返回一個 wrapper，wrapper 中 callback hello。

此外，Python 也支援類別方式的 decorator：
```python
class myDecorator(object):
    def __init__(self, fn):
        print("myDecorator.__init__()")
        self.fn = fn
    
    def __call__(self):
        self.fn()
        print("myDecorator.__call__()")

@myDecorator
def aFunc():
    print("aFunc()")

print("decorate complete")

aFunc()

# myDecorator.__init__()
# decorate complete
# aFunc()
# myDecorator.__call__()
```
可以看到執行的順序
1. \_\_init\_\_() 在給某個函式 decorator 時就被呼叫；所以參數需要一個 fn，就是被 decorate 的函式
2. \_\_call\_\_() 是在被 decorator 的函式被呼叫時才執行

來看一個比較實際的範例，下面的程式功能是用 URL 註冊，並呼叫相關的函式：
```python
class MyApp():
    def __init__(self):
        self.func_map = {}
    
    def register(self, name):
        def func_wrapper(func):
            self.func_map[name] = func
            return func
        return func_wrapper
    
    def call_method(self, name=None):
        func = self.func_map.get(name, None)
        if func is None:
            raise Exception("unregistered function - " + str(name))
        return func()

app = MyApp()

@app.register('/')
def main_page_func():
    return "This is main page"
    
@app.register('/next_page')
def next_page_func():
    return "This is next page"

print(app.call_method('/'))
print(app.call_method('/next_page'))
```
上面範例和一般 decorator 不同，一般是用 \_\_call\_\_() ， 但以上範例是用 call_method 去主動呼叫。

## Go 的裝飾模式 Decorator
Python 有 syntax surgar，所以寫起來比較簡潔，那接下來看一下沒有 syntax sugar 該如何做到，以下是 Go 的範例。

來看看 Hello, world 範例。
```go
package main

import "fmt"

func decorator(f func(s string)) func(s string) {
	return func(s string) {
		fmt.Println("start")
		f(s)
		fmt.Println("end")
	}
}

func Hello(s string) {
	fmt.Println(s)
}

func main() {
	decorator(Hello)("Hello, World")
}
```
這段程式的重點：
- 用 high order function，就是 decorator()，先把 Hello() 當成參數傳入，並返回一個匿名函式，這個匿名函式中，除了呼叫傳進來的 Hello() 以外，也另外增加了語句
- Go 沒有像 Python 一樣支援 @decorator，所以裝飾 hello 看起來就比較醜；稍微改善可以像下面這樣用：
```go
hello := decorator(hello)
hello("Hello, World")
```

再來看一個 HTTP route 的例子：
```go
func WithServerHeader(h http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Println("--->WithServerHeader()")
        w.Header().Set("Server", "HelloServer v0.0.1")
        h(w, r)
    }
}
 
func WithAuthCookie(h http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Println("--->WithAuthCookie()")
        cookie := &http.Cookie{Name: "Auth", Value: "Pass", Path: "/"}
        http.SetCookie(w, cookie)
        h(w, r)
    }
}
 
func WithBasicAuth(h http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Println("--->WithBasicAuth()")
        cookie, err := r.Cookie("Auth")
        if err != nil || cookie.Value != "Pass" {
            w.WriteHeader(http.StatusForbidden)
            return
        }
        h(w, r)
    }
}
 
func WithDebugLog(h http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Println("--->WithDebugLog")
        r.ParseForm()
        log.Println(r.Form)
        log.Println("path", r.URL.Path)
        log.Println("scheme", r.URL.Scheme)
        log.Println(r.Form["url_long"])
        for k, v := range r.Form {
            log.Println("key:", k)
            log.Println("val:", strings.Join(v, ""))
        }
        h(w, r)
    }
}

func hello(w http.ResponseWriter, r *http.Request) {
    log.Printf("Received Request %s from %s\n", r.URL.Path, r.RemoteAddr)
    fmt.Fprintf(w, "Hello, World! "+r.URL.Path)
}
```
上面的程式有很多函式，有寫 HTTP header 的、有寫入 Cookie 的、有認證 Cookie 的與寫 log 的，使用的時候可以嵌套多個裝飾。
```go
func main() {
    http.HandleFunc("/v1/hello", WithServerHeader(WithAuthCookie(hello)))
    http.HandleFunc("/v2/hello", WithServerHeader(WithBasicAuth(hello)))
    http.HandleFunc("/v3/hello", WithServerHeader(WithBasicAuth(WithDebugLog(hello))))
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```
如果覺得嵌套比較醜，想改成 pipeline 的話，要先寫一個函式來走訪並呼叫各 decorator：
```go
type HttpHandlerDecorator func(http.HandlerFunc) http.HandlerFunc

func Handler(h http.HandlerFunc, decors ...HttpHandlerDecorator) http.HandlerFunc {
	for i := range decors {
		d := decors[len(decors) - i - 1] // iterate in reverse
		h = d(h)
	}
	return h
}
```
然後就能像 pipeline 使用了。
```go
http.HandleFunc("/v4/hello", Handler(hello, WithServerHeader, WithBasicAuth, WithDebugLog))
```

## 總結
Functional programming 是古早就提出的概念，源自於 λ 演算；這篇文章透過幾個例子對比了非函式與函式語言設計的不同，再來介紹了 Map、Reduce、Pipeline 等技術，最後透過 Python 與 Go 的範例，來說明裝飾模式。

總結一下裝飾模式就是在做：
- 擴展現有函式的功能，在現有函式的功能上增加其他功能
- 展現 Functional programming 擴充性的強大，函式能夠互相組合
- 裝飾模式的設計方式，是把控制邏輯(what to do)抽象出來，與業務邏輯(how to do)分開
  - 控制邏輯就是迴圈、router、print log 之類
  - 業務邏輯則例如馬有多少機率往前跑(前面的範例)


## reference
- https://en.wikipedia.org/wiki/Functional_programming
- https://zh.wikipedia.org/wiki/%E6%83%B0%E6%80%A7%E6%B1%82%E5%80%BC
- https://pkg.go.dev/net/http