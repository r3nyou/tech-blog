---
title: "這次是對 Generic programming 的思考"
date: 2022-05-15T20:46:22+08:00
draft: false
tags: ["程式設計技巧"]
categories: ["Programming paradigm"]
---
程式寫了幾年後有了些感性認識，無論是想寫的更好，或是想學其他語言但覺得太多種學不完、也不知道怎麼學，只學了語法和表面。

後來發現了 Programming Paradigm 程式設計方法，透過程式語言的發展歷史，瞭解主流的幾種程式語言的特性，在比較與歸納中去思考程式設計的本質為何？一但在更高的維度思考，就能提高自己的程式設計能力。

Programming Paradigm 將程式語言分成不同的設計風格，這系列文章會透過幾種語言(C、C++、Java、Go、Python、JavaScript、Scheme、Prolog)來了解「如何寫出更通用、可重用的程式」，第一篇是對泛型 Generic programming 的思考。

## C 語言
許多人第一個學習的是 C 語言，可以做到微觀非常精細的操作，讓工程師自由控制底層、系統的細節，但在程式碼的組織、功能上有許多缺點，因此現在有許多 C Like 語言都是以 C 語言為基礎拓展，並改善 C 語言的問題。

C 語言的特性
- 靜態弱型別：使用變數需要定義資料型態，但類型之間會自動轉型。
- 不同變數可以用 struct 組合成新的資料型態。
- 可以用 typedef 關鍵字定義資料型態別名，達到變數資料型態的抽象。
- 有變數的作用域、遞迴功能。
- 參數以值傳遞，也可以傳指標。
- 透過指標能對記憶體做底層控制，但也增加了寫作的複雜度。
- 編譯預處理可以讓編譯更有彈性，做跨平台。

## C 語言的 Swap
```c
void swap(int* x, int* y) {
    int tmp = *x;
    *x = *y;
    *y = tmp;
}
```
- 傳指標是因為，函式參數只是引數的 copy，函式內修改參數無法影響外部的引數。
- 只有 int 能用，double、float 都不能用，這是靜態類型語言的問題。

## 資料型態
- 泛型程式設計，對於靜態類性語言首先要做的就是抽象資料型態。
- C 語言的編譯器會做自動轉型，讓寫程式方便一點，也可以做到一點點的泛型。
- C 語言做轉型，可能出現問題；例如一個 double 類型陣列，a[2] 尋址公式為 a + sizeof(double) * 2，如果轉成 int 陣列，尋址公式變成 a + sizeof(int) * 2，於是訪問到不同記憶體位址而出現錯誤。

## C 語言的泛型
C 語言的可以透過 void\* 做到泛型
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

為了泛型，增加了程式寫作的複雜度；而且如果要 swap 字串陣列，類型是 char\*，那參數的資料型態要用 void\*\*，那介面就無法定義了。

除了 void\* 以外，C 語言還能用巨集(macro) 做到泛型
```c
#define swap(x, y, size) {\
    char tmp[size]; \
    memcpy(tmp, x, size); \
    memcpy(x, y, size); \
    memcpy(y, tmp, size); \
}
```
巨集雖然不像 void\* 會隱藏資料型態，可以用 sizeof(x) 來檢查傳入參數的 size，但如果類型是 char\*，sizeof(x) 取得的是指標的 size，而非 char 的 size，因此還是必須把 size 交給呼叫的人去處理。

另外，巨集還有重複執行問題，例如
```c
#define min(x, y) ((x)<(y) ? (x) : (y))
```
- min(i++, j++)，本意是比較完後，對變數做累加，但因為巨集替換，會導致 i 或 j 其中一個被累加二次
- min(foo(), bar())，本意是比較 foo()、bar() 的返回值，但巨集替換後，foo()、bar() 其中一個函數會被呼叫二次。

C 語言的泛型，無論使用 void\* 或巨集，介面都變更複雜，且太過寬鬆，無法檢查資料型態，只能直接在記憶體上操作、複製，就像個未爆彈，可能在某些條件就會出錯。

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
- 用 cmpFn 比較是否相等，因為不同資料型態的比較實作方式不同，int 用 == 即可，而 struct 就必須自己實作比較方法。
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
目前 search function 為了支援各種資料型態，已經變得複雜，但還只能支援陣列資料結構，如果想要支援其他非順序型的資料結構，如 stack、heap、hash table、tree、graph，那實在是複雜到寫不下去了。

以上範例可以發現 C 語言的問題
- 如果一個算法想支援不同的資料型態，必須使用 void\* 或巨集，導致型態檢查過於寬鬆，容易產生錯誤。
- 抽象後的資料型態，無法取得 size，必須由呼叫者自己處理。
- 更複雜的泛型不只要支援不同的資料型態，而是要支援不同的資料結構（資料是在結構裡面），而不同資料結構要處理記憶體分配與釋放、物件之間的複製，其中還有 shallow copy 的問題，造成程式碼複雜度劇增。

C 語言至今沒有解決這些問題，因為當初設計的目的、思想就已經決定 C 語言注定不適合高階、抽象的程式設計方法，所以才有了 C++。
