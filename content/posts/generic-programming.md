---
title: "這次是對 Generic programming 的思考"
date: 2022-05-01T20:46:22+08:00
draft: false
ShowToc: true
tags: ["程式設計"]
categories: ["Programming paradigm"]
---
程式寫了幾年後，有了些感性認識，無論是想寫的更好，或想學其他語言但覺得太多種學不完、也不知道怎麼學，只學了表面。

如果你也有一樣的疑問，不妨瞭解下 Programming Paradigm 程式設計方法，透過程式語言的發展歷史，瞭解主流的幾種程式語言的特性，在比較與歸納中去思考程式設計的本質為何？從更高的維度去思考，就能提高程式設計能力。

Programming Paradigm 將程式語言分成不同的設計風格，這系列文章會透過幾種語言(C、C++、Java、Go、Python、JavaScript)來了解「如何寫出更通用、可重用的程式」。

這次是對泛型 Generic programming 的思考。

## C 語言
許多人第一個學習的是 C 語言，能做到微觀非常精細的操作，讓工程師自由控制底層、系統的細節，但在程式碼的組織、功能上有許多缺點，因此現在有許多 C Like 語言都是以 C 語言為基礎拓展，並改善 C 語言的問題。

C 語言的特性
- 靜態弱型別：使用變數需要定義資料型別，但類型之間會自動轉型。
- 不同變數可以用 struct 組合成新的資料型別。
- 可以用 typedef 關鍵字定義資料型別的別名，達到變數資料型別的抽象。
- 有變數的作用域、遞迴功能。
- 參數以值傳遞，也可以傳指標。
- 透過指標能對記憶體做底層控制，但也增加了寫作的複雜度。
- 編譯預處理可以讓編譯更有彈性，做跨平台。

## C 語言的 swap
```c
void swap(int* x, int* y) {
    int tmp = *x;
    *x = *y;
    *y = tmp;
}
```
這段程式重點有二個

- 傳指標是因為函式的參數 x、y 只是引數的 copy，函式內修改參數無法影響外部的引數。
- 問題在於這函式只適用於 int，double、float 都不能用，這是靜態類型語言面臨的問題。

## 資料型別

透過 swap() 能發現，靜態類型的語言要做到泛型，首先要處理的是資料型別的問題

- 泛型程式設計，對於靜態類性語言首先要做的就是抽象資料型別。
- C 語言的編譯器會做自動轉型，讓寫程式方便一點，也可以做到一點點的泛型。
- C 語言做轉型，可能出現問題；例如一個 double 類型陣列，a[2] 尋址公式為 a + sizeof(double) * 2，如果轉成 int 陣列，尋址公式變成 a + sizeof(int) * 2，於是訪問到不同記憶體位址而出現錯誤。

## C 語言的泛型
要讓 swap() 能通用各種型別，C 語言可以透過 void\* 做到泛型
```c
void swap(void* x, void* y, size_t size) {
    char tmp[size];
    memcpy(tmp, x, size);
    memcpy(x, y, size);
    memcpy(y, tmp, size);
}
```
比起 int 版本，有幾個改變
- 增加了 size 參數，因為用 void\* 後，類型被抽象掉了，編譯器無法知道類型的大小，需要手動處理。
- 用 memcpy()，因為類型被抽象後，沒辦法用 = 表達式賦值了，傳進來的甚至可能是 struct，因此只能用記憶體複製的方法。
- buffer 改成用 tmp[size] 陣列。

為了泛型，增加了程式寫作的複雜度；而且如果要 swap 字串陣列，類型是 char\*，那參數的資料型別要用 void\*\*，介面不一致，就無法定義了。

除了 void\* 以外，C 語言還能用巨集(macro) 做到泛型
```c
#define swap(x, y, size) {\
    char tmp[size]; \
    memcpy(tmp, x, size); \
    memcpy(x, y, size); \
    memcpy(y, tmp, size); \
}
```
巨集雖然不像 void\* 會隱藏資料型別，可以用 sizeof(x) 來檢查傳入參數的 size，但如果類型是 char\*，sizeof(x) 取得的是指標的 size，而非 char 的 size，因此還是必須把 size 交給呼叫的人去處理。

另外，巨集還有重複執行問題，例如
```c
#define min(x, y) ((x)<(y) ? (x) : (y))
```
- min(i++, j++)，本意是比較完後，對變數做累加，但因為巨集替換，會導致 i 或 j 其中一個被累加二次
- min(foo(), bar())，本意是比較 foo()、bar() 的返回值，但巨集替換後，foo()、bar() 其中一個函數會被呼叫二次。

C 語言的泛型，無論使用 void\* 或巨集，介面都變更複雜，且太過寬鬆，無法檢查資料型別，只能直接在記憶體上操作、複製，風險較大，可能在某些條件就會出錯。

## 更複雜的泛型範例 search function
在 int 陣列中搜尋 target，並返回索引，沒找到返回 -1
```c
int search(int* a, size_t size, int target) {
    for(int i=0; i<size; i++) {
        if (a[i] == target) {
            return i;
        }
    }
    return -1;
}
```
如果把這個 search function 改成泛型的
```c
int search(void* a, size_t size, void* target,
           size_t elem_size, int(*cmpFn)(void*, void*))
{
    for(int i=0; i<size; i++) {
        if ( cmpFn ((unsigned char *)a + elem_size * i, target) == 0 ) {
            return i;
        }
    }
    return -1;
}
```
- 增加 elem_size 參數，表示陣列裡元素的 size ，走訪陣列時才能正確移動指標到下一個元素。
- 用 cmpFn 比較是否相等，因為不同資料型別的比較實作方式不同，int 用 == 即可，而 struct 就必須自己實作比較方法。
- 沒有用 memcmp() 是因為如果陣列是一個指標陣列，或者是 struct 陣列且 struct 內有指標成員，用 memcmp() 是在比較指標的記憶體位址，但實際要比較的應是指標指向的值，而非指標本身。

呼叫者必須提供類似以下的函數做比較
```c
int int_cmp(int* x, int* y) { return *x - *y; }

int string_cmp(char* x, char* y) { return strcmp(x, y); }

typedef struct _account {
    char name[10];
    char id[20];
} Account;

int account_cmp(Account* x, Account* y) {
    int n = strcmp(x->name, y->name);
    if (n != 0) return n;
    return strcmp(x->id, y->id);
}
```
目前 search function 為了支援各種資料型別，已經變得複雜，但還是只能支援陣列、順序型的資料結構，如果想要支援非順序型的資料結構，如 stack、heap、hash table、tree、graph，那實在是複雜到寫不下去了。

透過以上幾個範例可以發現 C 語言的問題
- 如果一個算法想支援不同的資料型別，必須使用 void\* 或巨集，導致型別檢查過於寬鬆，容易產生錯誤。
- 抽象後的資料型別，無法取得 size，必須由呼叫者自己處理。
- 更複雜的泛型不只要支援不同的資料型別，還要支援不同的資料結構（資料是在結構裡面），而不同資料結構要處理記憶體分配與釋放、物件之間的複製，其中還有 shallow copy 的問題，造成程式碼複雜度劇增。

C 語言至今沒有解決這些問題，因為當初設計的目的、思想就已經決定 C 語言注定不適合高階、抽象的程式設計方法，所以才有了 C++。

## C++ 語言

早期 C++ 許多功能是對 C 語言的強化、改進，且把兼容 C 作為強制要求，所以 C++ 才設計得這麼複雜；而 C 語言的 C89、C99 也參考了 C++ 做了很多改進。

> 九二共識就是沒有共識，我只知道九九共識，依循 C99 規格開發程式。

C++ 解決了 C 的許多問題

- 用 reference 解決指標的問題
- 用 namespace 解決命名衝突問題
- 用 try-catch 解決檢查返回值的問題
- 用 class 解決物件建立、複製、刪除的問題
- 用 operator overloading 達到操作上的泛型，比如上一篇的 cmpFn 比較函式，再比如 \>\> 操作符消除 printf() 的資料結構不夠泛型的問題
- 用 template、virtual function、RTTI 達到更高層次的泛型與多型
- 用 RAII、smart pointers，解決 C 語言中為了釋放資源而寫的骯髒、容易錯的程式碼
- 用 STL 解決 C 語言中資料結構、演算法的問題

## C++ 的泛型

理想情況下算法與資料型別、資料結構都無關，各種資料結構應該自己處理份內的工作，而算法只關心一個標準的實作方法；而泛型程式設計需要解決幾個問題

- 算法的泛型
- 資料型別的泛型
- 資料結構(資料的容器)的泛型

C++ 透過幾種方式解決泛型問題，能夠寫出基於抽象的介面的泛型
- 透過類別(class)解決資料型別、資料結構的問題
  - 類別裡面有建構子、解構子，處理資源的分配、釋放
  - 複製建構子，表示對記憶體的複製
  - 運算子重載，處理比較的問題
  - 這樣自訂的資料型別用起來就跟內建的一樣
- 透過模板處理資料型別
  - 模板會根據呼叫者的類型，在編譯時去生成那個模板的程式碼
  - 模板可以用虛擬類型來綁定，就沒有資料型別轉換的問題
  - 模板取代了 C 語言的巨集，並解決了巨集的問題
- 虛擬函式、RTTI
  -  虛擬函式的多型，在語法上可以支援同一類的型別的泛型
  -  泛型時 RTTI 可以對具體類型做特殊處理

## C++ 泛型範例 search function

之前 C 版本的 search() 還有幾個問題
- 透過 for(int i=0; i<len; i++) 的走訪方式，不適用於非順序型資料結構，如 hash table、tree、graph，需要改成泛型的操作（走訪、增刪改查）
- search 返回 int 型別的索引位置，假設換成 hash table，因為資料不是連續的，索引位置沒有意義，所以返回的要改成某種 key，所以泛型的 search 要改成返回這個元素的指標才通用

來看看 C++ 如何解決以上問題

```cpp
template<typename T, typename Iter>
Iter search(Iter pStart, Iter pEnd, T target)
{
    for(Iter p = pStart; p!= pEnd; p++) {
        if (*p == target)
            return p;
    }
    return NULL;
}
```
這段程式用了 C++ 的幾個技術來解決 C 的問題
- 用 typename T 抽象了資料結構中儲存資料的型別
- 用 typename Iter 抽象了資料結構，不同資料結構自己要實作迭代器
- Iter 的 ++ 方法抽象了走訪方法，不同資料結構自己要實作運算子重載
- 用 \*Iter 取得指標指向的值，\* 取值操作符也需要重載

如果習慣 Java 可能會覺得 Iter.Next() 比 ++ 好，用 Iter.GetValue() 比 * 好，但主要是 C++ 需要兼容 C 的習慣。

實際 C++ STL 裡的 find() 是這樣，很類似我們寫的 search 方法，只是他用 while loop

```cpp
template<class InputIterator, class T>
InputIterator find (InputIterator first, InputIterator last, const T& val)
{
    while (first!=last) {
        if (*first==val) return first;
        ++first;
    }
    return last;
}
```
到此 search function 的泛型設計就完成了，但其實 search() 遇到的問題只是一小部分，還有其他問題能夠讓泛型設計變得更複雜。讓我們來看看另一個 sum function 的例子。

## C++ 泛型範例 sum function
先看 C 語言版本
```c
long sum(int *a, size_t size) {
    long result = 0;
    for(int i=0; i<size; i++) {
        result += a[i];
    }
    return result;
}
```
C++ 的版本
```cpp
template<typename T, typename Iter>
T sum(Iter pStart, Iter pEnd) {
    T result = 0;
    for(Iter p=pStart; p!=pEnd; p++) {
        result += *p;
    }
    return result;  
}
```
這段程式有幾個問題，特別是 T result = 0; 這一行
- T 假設了 Iter 中返回的資料型別是 T
- = 0 假設資料型別是 int

如果資料型別不同，出現轉型，會有非常奇怪的 bug。

## C++ 泛型技術 迭代器
Iter 在呼叫時，會類似 `vector<int>::iterator`，這個 declaration 已經把 int 傳進 Iter，所以 result 的 T 可以從 Iter 取得，這樣可以保證資料型別一致。所以我們需要實作一個迭代器(參考 STL，省略部分程式)

```cpp
template <class T>
class container {
public:
    class iterator {
    public:
        typedef iterator self_type;
        typedef T   value_type;
        typedef T*  pointer;
        typedef T&  reference;

        reference operator*();
        pointer operator->();
        bool operator==(const self_type& rhs);
        bool operator!=(const self_type& rhs);
        self_type operator++() { self_type i = *this; ptr_++; return i; }
        self_type operator++(int junk) { ptr_++; return *this; }
        //...省略
    private:
        pointer _ptr;
    };

    iterator begin();
    iterator end();
    //...省略
};
```
有幾個關鍵點
- 迭代器是對某容器的具體實作，所以要跟容器在一起
- 要重載一些操作符，如：\* 取值操作、-> 成員操作、== 比較操作、!= 比較操作、++ 走訪操作
- typedef 一些資料型別，如 value_type，說明容器內的資料的實際型別為何
- begin()、end() 基本功能
- 有個指標 _ptr 用來指向當前資料，就是 T\*

再來要解決 T result = 0; 中 = 0 的問題，這需要由呼叫方傳入，改良後如下
```cpp
template <class Iter>
typename Iter::value_type
sum(Iter start, Iter end, T init) {
    typename Iter::value_type result = init;
    while (start != end) {
        result = result + *start;
        start++;
    }
    return result;
}
```
於是呼叫方可以這樣寫
```cpp
container<int> c;
container<int>::iterator it = c.begin();
sum(c.begin(), c.end(), 0);
```
解決了所有問題，這就是整個 STL 的泛型，包含了
- 泛型的資料結構(資料容器)
- 泛型資料結構的迭代器
- 泛型的算法，目前 sum() 是單純累加，但要寫其他算法也很容易擴充了
## 更多的抽象
來看一個更泛型的例子，有個資料結構 Employee，裡面有 vacation 休假日數、salary 薪資
```cpp
struct Employee {
    string id;
    string name;
    int vacation;
    double salary;
}
```
如果想計算員工的薪資總和、休假總天數
```cpp
vector<Employee> staff;
sum(staff.begin(), staff.end(), 0);
```
會發現 sum() 不知道要怎麼寫，因為加總的會是不同的欄位，就算 Employee 有重載 + 運算子，也不知道要加總哪個欄位。

還有更多需求，如求平均值、最小值、最大值、中位數，算法和 sum() 是一樣的，只是其中的累加 + 會變成其他操作。

我希望算法只要負責走訪，具體的操作由呼叫者自己定義，這樣程式碼會更通用，例如一個更抽象的算法 reduce()，用來把陣列 reduce 成一個值，至於怎麼變成一個值，則由呼叫者提供的函式來決定。
```cpp
template<class Iter, class T, class Op>
T reduce(Iter start, Iter end, T init, Op op) {
    T result = init;
    while (start != end) {
        result = op(result, *start);
        start++;
    }
    return result;
}
```
算法和 sum() 一樣，但不是單純累加，而是迭代器提供一個函數物件 operation 來處理具體操作。

C++ STL 中，有個類似的函數 accumulate()，原始碼有二個版本。
```cpp
template<class InputIt, class T>
T accumulate(InputIt first, InputIt last, T init)
{
    for (; first != last; ++first) {
        init = init + *first;
    }
    return init;
}
```
第二個版本更抽象，因為需要傳入一個二元操作函式
```cpp
template<class InputIt, class T, class BinaryOperation>
T accumulate(InputIt first, InputIt last, T init, 
             BinaryOperation op)
{
    for (; first != last; ++first) {
        init = op(init, *first);
    }
    return init;
}
```

呼叫時，可以用 C++ 的 lambda 表達式
```cpp
double sum_salaries = 
    reduce(staff.begin(), staff.end(), 0.0,
        [](double s, Employee e)
            { return s + e.salary; });

double max_salary =
    reduce( staff.begin(), staff.end(), 0.0,
        [](double s, Employee e)
            { return s > e.salary ? s : e.salary; });
```
## Reduce 函式
reduce 還可以完成更複雜的功能，例如計算薪水超過 5 萬的員工人數。

先定義一個函式物件 counter，counter 需要一個 Cond 函式物件，用來判斷是否符合條件，符合的話會加 1，不符合加 0。
```cpp
template<class T, class Cond>
struct counter {
    size_t operator()(size_t c, T t) const {
        return c + (Cond(t) ? 1 : 0);
    }
};
```
然後用 counter、reduce 二個函式物件來寫一個 count_if 算法。
```cpp
template<class Iter, class Cond>
size_t count_if(Iter begin, Iter end, Cond c) {
    return reduce(begin, end, 0,
        counter<Iter::value_type, Cond>(c));
};
```
至於 Cond 實際是什麼條件，不是算法負責，而是由呼叫方決定，於是想要計算薪水超過 5 萬的員工人數，可以這樣寫。
```cpp
size_t count = count_if(staff.begin(), staff.end(),
    [](Employee e){ return e.salary > 50000; });
```
從 C 到 C++，與 reduce() 的範例，可以了解發現程式語言需要處理二件事情
- 資料型別問題
- 抽象、可重複用、組合

不同語言解決的方法不一定都是 C++ 這種方式，但原理是相通的；因此接下來繼續深入討論程式語言的型別系統、與泛型程式設計的本質。

## 型別系統 type system
型別系統用來把程式語言中的數值、表達式做分類，以及定義這些型別能如何操作，一般分成二種
- 基本資料型別，如 int、float、char
- 抽象資料型別，如 struct、class、function

型別系統的功能如下
- 安全性：編譯器會檢查語法錯誤，例如 “hello, world” + 1，強型別的語言可以提供更高安全性。
- 優化：靜態類型的語言，能讓編譯器知道變數的資料型別，編譯器就能做優化；如：int 可以知道這個類型以 4 byte 的倍數對齊，編譯器能使用更有效率的機器碼指令。
- 抽象化：型別可以讓人用更高層次思考；例如可以把 string 當成一個值，而不是底層的 char array；不同函式之間的參數、返回值的型別，也可以讓介面更語意化。

但就如前面遇到的問題，型別系統會造成，就算不同型別的算法的程式碼長得一樣，卻為了不同型別而需要寫 N 多種，如果要做泛型，就只能用更底層的方法；因此這個世界出現了動態類型的語言，例如：
```
x = 5;
x = "hi";
```
靜態語言在編譯時會出語法錯誤；但在動態語言中，會使用型別標記維持程式所有數值的標記（不是變數），在運算前會先檢查標記，所以變數的類型不是透過型別定義，而是解釋器在運行時動態標記的，在運行時才動態的對應底層指令、記憶體佈局。

再來像 JavaScript 還可以定義一個包含各種型別的類型
```javascript
var a = new Array();
a[0] = 2022;
a[1] = "hi";
a[2] = {name: "Marcus"};
```
其實這並不是 array，而是一個 key value，因為動態語言的型別是動態的，所以 key、value 的型別都可以不同，還可以寫成 a[“key”] = “value”;

因此在動態語言中會因為類型，而出現不確定的結果
```
x = 3;
y = "5";
z = x + y;
```
- JavaScript 會把數字 3 轉成字串，然後運算子 + 會把字串拼接起來，結果是 “35”
- Python 會發生 runtime error
- Visual Basic 會把字串 5 轉成數字，結果是 8

程式語言的型別系統歸納如下
- 靜態類型，在編譯時進行語法分析，型別錯誤是 syntax error
  - 如果允許自動轉型，則稱為弱型別
  - 如果強制類型規則（只允許資料不丟失的轉型），則稱為強型別
- 動態類型，在運行時做動態標記，型別錯誤是 runtime error；所以動態類型語言會提供很多類型檢查的函式：is_array()、in_int()、is_string()、typeof()...

總之，所有語言都有型別系統，型別是對底層記憶體佈局做抽象，讓我們可以專心在上層的邏輯；靜態語言好處是編譯器的檢查、增加運行時效能；而動態語言優點是程式碼更簡單、寫得更輕鬆快速。

但在程式碼複雜度方面，動態語言不一定永遠是比較簡單的，如果某個系統需要明確的型別，或者型別不符會產生錯誤時，就會看到動態語言需要寫額外的型別檢查，這不只囉唆，且性能比較差，例如 JavaScript 要寫一個轉型的函式。

```javascript
function ToNumber(x) {
    switch(typeof x) {
        case "number": return x;
        case "undefined": return NaN;
        case "boolean": return x ? 1 : 0;
        case "string": return Number(x); 
        case "object": return NaN;
        case "function": return NaN;    
    }
}
```

## 泛型的本質
泛型的本質，就是型別的本質。
- 型別是底層記憶體佈局的抽象，不同型別有不同的憶體佈局、分配方式
- 不同型別有不同操作

要做到泛型，需要滿足以下要求
- 抽象掉型別的記憶體訪問、分配、釋放
- 抽象掉型別的操作，比較、複製、I/O…
- 抽象掉資料結構(資料的容器)的操作，搜尋、過濾、aggregation…
- 抽象掉型別的特殊操作要有標準的介面去 callback 不同型別的實際具體操作

以 C++ 而言解決泛型的技術如下；許多程式語言的泛型設計，多少都參考了 C++
- 類別的建構子、解構子、複製建構子、運算子重載，抽象掉型別的記憶體訪問、分配、釋放
- 運算子重載，抽象掉比較之類的操作
- iostream 抽象掉型別的輸入、輸出控制
- 用 template 來幫不同型別生成專屬的程式碼
- 用迭代器抽象掉資料結構的走訪
- virtual function 來抽象掉特定型別在算法上的實際操作
- 函數物件抽象掉對於不同型別的特殊操作

泛型就是隱藏資料、操作資料的細節，讓算法更通用，讓我們能專注在算法，而不需要處理算法中不同的資料型別。

> Generic programming centers around the idea of abstracting from concrete, efficient algorithms to obtain generic algorithms that can be combined with different data representations to produce a wide variety of useful software.  — Musser, David R.; Stepanov, Alexander A., Generic Programming

程式語言作為人操作機器的媒介，究竟要給予更大的操作自由，還是隱藏底層，方便抽象的思考，二者的取捨，C 語言選擇了前者，而 Java 選擇了後者。

瞭解程式語言的設計與取捨，就明白為何這個語言發展成現在的模樣，無論在程式寫作、或學習新的語言時，都能從中有所收穫。

## reference
1. http://stepanovpapers.com/genprog.pdf
2. https://zh.wikipedia.org/wiki/%E9%A1%9E%E5%9E%8B%E7%B3%BB%E7%B5%B1
