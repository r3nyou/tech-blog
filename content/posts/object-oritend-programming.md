---
title: "光看標題是學不會物件導向的"
date: 2022-05-27T10:20:05+08:00
draft: false
ShowToc: true
tags: ["程式設計"]
categories: ["Programming paradigm"]
---
既然點進來看內文了，這篇文章是介紹物件導向程式設計方法。

之前在 [Functional programming 的文章](../functional-programming)介紹過，Functional programming 是透過組合各種函式來完成功能，函式是計算過程的抽象，但程式碼除了計算過程以外，還需要處理資料，也就是所謂的「狀態」，關於狀態的處理，就需要提到物件導向程式設計 (Object-oriented programming)。

所謂物件指的是類別的實體，把物件當成程式的基本單位，並將資料封裝在其中，程式會被設計成一個個獨立的物件，每個物件都要能接受資料、處理資料、傳資料給其他物件，好比一個個小型的機器一樣，各自獨立，且彼此互相關聯。這與傳統的程式設計方法不同，傳統的想法是把程式設計成一系列對機器的指令、或者一系列函式的集合。

目前物件導向被廣泛使用在大型專案中，優點是靈活性、可維護性；另外有部分人認為物件導向更容易分析、理解程式。

物件導向領域有一本經典書籍，[Design Pattern](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8)，這本書介紹了 23 個設計模式，說的是二個觀念：
**Program to an interface, not an implementation.**
- 呼叫者不需要知道資料型別、資料結構、算法的細節
- 呼叫者不需要知道實作細節，只要知道提供什麼介面
- 方便封裝、抽象、多型、動態綁定
- 符合物件(導向)的特質

**Favor object composition over class inheritance.**
- 繼承需要暴露父類別的設計、實作給子類別
- 父類別改變會造成子類別也要改變
- 很多人有誤區，認為繼承是為了程式碼重用，實際上子類別仍需要重新實作很多父類別的方法，其實繼承更多是為了多型

## 範例一 組合物件
假設一個家具的系統，需求如下：
- 四個物體：塑膠桌、塑膠椅、木頭桌、木頭椅
- 四種屬性：價格、重量、密度、燃點

物件導向設計為下圖：
![object composition uml](/img/object-oritend-programming/object-composition-uml.png)

- 材質類別 Material，有密度、燃點屬性
- 家具類別 Furniture，有價格、體積屬性
- Furniture 耦合了 Material，具體是木製、塑膠製，在建立家具物件時注入材質物件即可
- 家具類能用自己的體積，與材質物件的密度，計算出重量

這樣設計的優點是
- 好理解，與現實世界對應
- 材質類別可重複使用，前面有提到物件導向喜歡組合，而非繼承
- 這個設計方式叫 Bridge pattern

Functional programming 強調動詞，而 Object-oriented programming 強調名詞，關注介面之間的關係，利用多型來達成不同的具體實作。

## 範例二 組合功能
來看另一個例子，有個電商系統要處理不同折扣的訂單，有的原價、有的要打折。

以 Java 為例，先寫一個介面，輸入原始價格，返回根據不同方式的折扣價。
```java
interface BillingStrategy {
	public double getActPrice(double rawPrice);
}
```

然後實作 BillingStrategy 介面
```java
class NormalStrategy implements BillingStrategy {
	@Override
	public double getActPrice(double rawPrice) {
		return rawPrice;
	}
}

class XmasStrategy implements BillingStrategy {
	@Override
	public double getActPrice(double rawPrice) {
		return rawPrice * 0.5;
	}
}
```
上面實作了二種折扣方式，NormalStrategy 是原價，XmasStrategy 是聖誕節半價。

接著，訂單的品項除了有商品的價格、數量等，還包含「折扣方式」。
```java
class OrderItem {
	public String Name;
	public double Price;
	public int Quantity;
	public BillingStrategy Strategy;
	public OrderItem(String name, double price, int quantity, BillingStrategy strategy) {
		this.Name = name;
		this.Price = price;
		this.Quantity = quantity;
		this.Strategy = strategy;
	}
}
```

最後，在訂單類別 Order 中封裝 OrderItem 的串列；加入商品時要給一個折扣方式 BillingStrategy；在 PayBill() 則計算訂單的總價。
```java
calss Order {
	private List<OrderItem> orderItems = new ArrayList<OrderItem>();
	private BillingStrategy strategy = new NormalStrategy();
	
	public void Add(String name, double price, int quantity, BillingStrategy strategy) {
		orderItems.add(new OrderItem(name, price, quantity, strategy));
	}
	
	public void PayBill() {
		double sum = 0;
		for (OrderItem item : orderItems) {
			actPrice = item.Strategy.getActPrice(item.price * item.quantity);
			sum += actPrice;
		}
		System.out.println("Total due: " + sum);
	}
}
```
上面範例把折扣計算與訂單處理流程分開，可以將不同商品注入不同的折扣方式，有很高的靈活度。這個設計方式叫 Strategy pattern。

## 範例三 資源管理
先看一段程式碼：
```cpp
mutex m;
void test() {
	m.lock();
	Func();
	if (! ok()) return;
	// do something else...
	m.unlock();
}
```

這段程式碼有個問題，如果 if 條件為真 early return 的話，沒有 release lock，所以要在 return 前先釋放。
```cpp
mutex m;
void test() {
	m.lock();
	Func();
	if (! ok()) {
		m.unlock();
		return;
	}
	// do something else...
	m.unlock();
}
```

但這樣做的缺點是，所有 return 的地方都要加上 m.unlock(); 導致難以維護，這時可以用物件導向的設計方法，先設計一個代理類別。
```cpp
class lock_guard {
	private:
		mutex &_m;
	public:
		lock_guard(mutex &m):_m(m) { _m.lock(); }
		~lock_guard() { _m.unlock(); }
};
```

之後的程式碼就可以這樣使用：
```cpp
mutex m;
void test() {
	lock_guard guard(m);
	Func();
	if(! ok()) {
		return;
	}
	// do something else...
}
```
這個設計方式叫 Proxy pattern，用 Proxy pattern 來達成 C++ 的 RAII 技術(Resource Acquisition Is Initialization)，把控制資源分配、釋放的邏輯都交給代理類別，然後我們只要管業務邏輯即可。

上面三個範例，介紹了物件導向的幾個特點：
- 用介面抽象具體類別
- 類別之間耦合的是介面，而非具體類別；也就是多型，能強化擴展性
- 這就是 Program to an interface
- 這些就是物件導向的核心概念，也是 IoC 控制反轉、DIP 控制反轉的原理

## IoC 控制反轉
控制反轉 Inversion of control 的概念，不只用於程式設計上，也用於系統設計，甚至真實世界也能看到實際例子。

先舉一個程式設計的範例，設計一個燈的控制開關：
![ioc uml 1](/img/object-oritend-programming/ioc-uml-1.png)

然後新的需求是要擴展不同的燈，於是變成以下這樣：
![ioc uml 2](/img/object-oritend-programming/ioc-uml-2.png)

但是有一天開關除了控制燈以外，還要控制其他東西，結果因為開關類別耦合了燈泡類別，導致無法擴展。

看看 IoC 怎麼解決這個問題，就像真實世界一樣，開關工廠只做好開關本身，把電接通、把電中斷，根本不需要管開關要控制什麼東西；而燈泡工廠也一樣，只要把電源開關介面做好，符合標準的介面的開關就可以裝在燈泡上使用。

IoC 的設計會是：
![ioc uml 3](/img/object-oritend-programming/ioc-uml-3.png)

舉一個真實世界的例子，假設有個賣電器的公司，想把產品放在各大賣場銷售，結果各大賣場的規則都不同，隨著銷售管道越多就越複雜；而 IoC 是電器公司自己制定標準，各大賣場想在櫃上賣電器產品，就要符合電器公司的標準，才能成為「經銷商」。

## 總結
總結一下物件導向程式設計的優點：
- 物件與真實世界的概念對應，容易理解
- 強調名詞，而非動詞，關注的是物件之間的介面
- 根據需求的特徵形成高內聚的物件，分離抽象與具體實作，強化擴展性、可重用性
- 有大量優秀設計原則、模式可參考
- S.O.L.I.D、IoC、DIP

而物件導向的缺點是：
- 程式碼必須要在類別中，換言之，鼓勵了型別
- 程式碼要透過物件做抽象，導致厚重的黏合層(Glue code)
- 大量封裝、與鼓勵使用狀態，造成不透明，在併發特別容易出問題

透過物件來做抽象，把程式碼分散到不同類別中，執行起來就需要把這些類別黏合起來，導致厚重的黏合層(Glue code)；像 Java Spring 中用到注入、鼓勵黏合、大量封裝，完全不知道裡面做了什麼，這些都是物件導向的缺點。

物件導向是目前的主流，但其實有許多人不喜歡這種設計，特別是 Generic、Functional programming 的支持者，彼此甚至有種宗教情節。

我認為在沒有確認場景、開發何種系統的前提下，單純討論程式設計方法，動機不是別有居心，就是出於個人偏好講爽的，沒有意義，也不會有結果。實際使用上，可以同時使用多種的設計方法，例如 Java 8 後也支援了 Lambda。

總之，這篇文章介紹了物件導向程式設計，透過範例說明核心的概念，同時介紹了幾種 Design pattern，接著透過程式設計、真實世界的範例說明 IoC，最後總結了優缺點。

reference
- https://en.cppreference.com/w/cpp/language/raii
- https://en.wikipedia.org/wiki/Inversion_of_control
- https://en.wikipedia.org/wiki/Glue_code
- https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html

