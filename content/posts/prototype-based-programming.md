---
title: "Sorry，Prototype 真的可以為所欲為"
date: 2022-06-15T10:57:38+08:00
draft: false
ShowToc: true
tags: ["程式設計"]
categories: ["Programming paradigm"]
---

之前在 [物件導向的文章](../object-oritend-programming) 中介紹了基於類別的物件導向設計，還有另一種特別的物件導向設計，被稱為原型程式設計 Prototype-based programming。沒有 class，直接使用物件，例如 JavaScript。

與傳統物件導向的差別在：
- 傳統基於類別的程式中，類別定義物件的布局、函式；介面則是「能使用」的物件，是基於某些特定類別的樣式。這種設計中，類別是行為、結構的集合，而對介面而言，其所屬類別的行為、結構都是相同的。所以區分規則首先是行為、結構(類別)，然後才是狀態(物件實體)。
- 原型程式設計則認為不該先關注類別之間的關係，而是先關注一系列物件實體的行為，然後才將這些物件依照使用方式區分成相似的 prototype object，而不是分成類別。

因此原型程式設計，鼓勵 runtime 進行原型修改：
- 基於類別的語言裡，物件透過類別的建構子來實體化，物件實體是由類別設定的行為、布局建立
- 原型程式設計裡要實體化物件有二種方法，一透過複製其他物件，二是擴展空物件
- 基於類別的語言只有動態語言允許在 runtime 修改類別(PHP、Python、Ruby)

## JavaScript 的原型觀念
透過 JavaScript 說明開頭的觀念。

通常物件導向裡都要有 class，但原型程式的 JavaScript 認為不需要 class，直接在物件上改就好了。

來看一個簡單範例：
```javascript
var foo = {name: "foo", one: 1, two: 2};

var bar = {three: 3};
```

每個物件都有一個原型屬性 \_\_proto\_\_。所以能把 foo 指派給 bar.\_\_proto\_\_，代表 bar 的原型是 foo。
```javascript
bar.__proto__ = foo;
```

然後，就可以在 bar 裡面取得 foo 的屬性。
```javascript
bar.one; // resolves to 1

bar.three  // resolves to 3

// Own properties shadow prototype properties
bar.name = "bar";
foo.name; // unaffected, resolves to "foo"
```

這裡要先解釋一下 JavaScript 的二個概念， \_\_proto\_\_ 與 prototype，JavaScirpt 中物件有二種形式，一是 Object，二是 Function：
-  \_\_proto\_\_ 用來放一個物件實體，產生一個原型鏈接，用於尋找方法名稱、屬性，是指向原型的指標
- prototype 是 Function 物件的屬性，new 建立一個物件時，該物件的 \_\_proto\_\_ 會指向 Function 物件的 prototype

例如以下的程式碼：
```javascript
var a = {
	x : 10,
	calculate: function (z) {
		return this.x + this.y + z;
	}
};

var b = {
	y: 20,
	__proto__: a
};

var c = {
	y: 30,
	__proto__: a
}

b.calculate(30); // 60
c.calculate(40); // 80
```

原型鍊 prototype chain 如下圖：
![prototype-chain](/img/prototype-based-programming/prototype-chain.png)

再來看另一個範例：
```javascript
function Foo(y) {
	this.y = y;
}

Foo.prototype.x = 10;

Foo.prototype.calculate = function (z) {
	return this.x + this.y + z;
}

var b = new Foo(20);
var c = new Foo(30);

b.calculate(30); // 60
c.calculate(40); // 80
```
![constructor-proto-chain](/img/prototype-based-programming/constructor-proto-chain.png)

可以做個測試：
```javascript

b.__proto__ === Foo.prototype, // true
c.__proto__ === Foo.prototype, // true
 
b.constructor === Foo, // true
c.constructor === Foo, // true
Foo.prototype.constructor === Foo, // true
 
b.calculate === b.__proto__.calculate, // true
b.__proto__.calculate === Foo.prototype.calculate // true
```

注意 Foo.prototype 自動建立一個 constructor，指向函式自己，物件 b、c 就能訪問到繼承的 constructor。

## JavaScript Prototype-based OOP
先複習一下上面 \_\_proto\_\_ 與 prototype：
```javascript
function Person() {}
var p = new Person();

Person.prototype.name = "Marcus";
Person.prototype.sayHi = function () {
	console.log("Hi, I am " + this.name);
}

console.log(p.name); // "Marcus"
p.sayHi(); // "Hi, I am Marcus"
```
以上範例做了幾件事：
- 建立空的函式物件 Person()
- 用 Person() new 出一個物件，指派給 p
- 然後再改變 Person.prototype，新增一個 name 屬性、sayHi() 方法
- 結果發現 p 也被改變了

注意：
- 建立 function Person(){} 時，Person. \_\_proto\_\_ 指向 Function.prototype
- 建立 var p = new Person() 時，p. \_\_proto\_\_ 指向 Person.prototype
- 所以修改 Person.prototype，p. \_\_proto\_\_ 的內容也被改變

再來看物件導向 OOP 該怎麼做。

首先定義一個 Person 類別。
```javascript
var Person = function (name, email) {
	this.name = name;
	this.email = email;
	
	this.speak = function () {
		console.log("speak");
	}
	
	this.introduction = function () {
		console.log("Hi, I am " + this.name);
	}
}
```

接著可以定義 Student 類別。
```javascript
var Student = function (name, email, school, grade) {
	Person.call(this, name, email);
	
	this.school = school;
	this.grade = grade;
	
	this.introduction = function () {
		console.log("Hi, I am " + this.name +
		", I am a student of " + this.school);
	}
	
	this.showGrade() {
		console.log("I am in " + this.grade);
	}
}
```
- Person.call(this, name, email); 這裡 call()、apply() 都是要動態改變 this 指向的物件的內容
- override 了 introduction() 方法
- 新增了 showGrade() 方法

接下來要讓 Student 繼承 Person，所以需要修改 Student 的原型。

可以直接 Student.\_\_proto\_\_ = Person.prototype，但這樣太粗魯了；還是用正規的方式：
```javascript
Student.prototype = Object.create(Person.prototype);

Student.prototype.constructor = Student;
```
- 用 Object.create() 把 Student.prototype 關連到 Person.prototype
- 修改一下建構子

於是就可以這樣使用：
```javascript
var student = new Student("Marcus",
              "marcus@email",
              "ABC college",
              "Freshman");
student.introduction();
student.speak();
student.showGrade();

console.log(student instanceof Person); // true
console.log(student instanceof Student); // true
```

## 總結
原型設計的語言使用委派 (Delegation) 的概念，runtime 僅透過指標去找到正確的屬性，物件彼此關聯、共享的行為是透過委派的指標。

有別於傳統基於類別的物件導向語言中類別、介面的關係，原型並不需要子類別與父類別有相似的記憶體結構，所以子類別可以自由修改，而不需要整理結構，對原型設計的語言來說，資料、方法更像是外掛的插槽 (slots)。

這種在物件裡免直接修改的方式，非常的靈活，不只是在 runtime 增加屬性、方法，甚至能夠刪除屬性、方法，但也造成了執行期間的不確定性，讓程式碼變得難以預測，需要由工程師自己保證。

備註，ECMAScript 標準第六版有提供基於舊有原型架構的 syntax sugar  "class"，可以用來建構物件、處理繼承。

## reference
- https://en.wikipedia.org/wiki/Prototype-based_programming
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- https://262.ecma-international.org/5.1/#sec-15.2
- http://dmitrysoshnikov.com/ecmascript/javascript-the-core/