---
title: "SOLID Principles without React - LSP"
date: 2022-06-25T17:03:35+08:00
draft: false
---

Hi 大家好我是 Curt 家人

# 系列相關文章

- [SOLID Principles With React](/posts/solid-principles-with-react/)
- [Single responsibility principle (SRP) - 單一職責原則](/posts/solid-principles-with-react-srp/)
- [Open–closed principle (OCP) - 開放封閉原則](/posts/solid-principles-with-react-ocp/)
- Interface segregation principle (ISP) - 介面隔離原則
- Dependency inversion principle (DIP) - 依賴反轉原則

# LSP

## 前言

這篇文章要來介紹不好理解的原則：`里氏替換原則`，開始讀的時候可能很容易一頭霧水，並且網路上有蠻多 Bird 、 Duck 與 Triangle 的範例，自己看完之後還是有點沒想法，
不過隨著看越多的相關文章，好像就可以慢慢地體會到核心理念，並且要達到他的要求好像也不難。

[wiki](https://en.wikipedia.org/wiki/Liskov_substitution_principle#:~:text=In%20the%20same%20paper%2C%20Liskov%20and%20Wing%20detailed%20their%20notion%20of%20behavioral%20subtyping%20in%20an%20extension%20of%20Hoare%20logic%2C%20which%20bears%20a%20certain%20resemblance%20to%20Bertrand%20Meyer%27s%20design%20by%20contract%20in%20that%20it%20considers%20the%20interaction%20of%20subtyping%20with%20preconditions%2C%20postconditions%20and%20invariants.) 的資料中有提到：LSP 在論文內說明 subtyping 的擴充行為與 design by contract 有著相似的概念，因此以下也會將其一起說明。

在開始之前需要提醒一下，因為 LSP 強調的是物件繼承間的問題，也就是子類與父類的繼承關係合不合適，然而 React 的開發風格不需使用繼承，每個 component 只需要 extends React 就好，
因此本篇不會有 React 的範例。

## 開始介紹

LSP 的全文叫做 Liskov substitution principle，Liskov 是作者的名字，substitution 則是替換性也是這個原則所要探討的點。

何謂替換性？

> if S is a subtype of T, then objects of type T in a program may be replaced with objects of type S - [wiki](https://en.wikipedia.org/wiki/Liskov_substitution_principle#:~:text=if%20S%20is%20a%20subtype%20of%20T%2C%20then%20objects%20of%20type%20T%20in%20a%20program%20may%20be%20replaced%20with%20objects%20of%20type%20S)

如果 S 是 T 的子類型，那麼程序中的所有 T 都能被換成 Ｓ

除此之外後面有繼續補充

> without altering any of the desirable properties of that program - [wiki](https://en.wikipedia.org/wiki/Liskov_substitution_principle#cite_note-liskov1994-1:~:text=without%20altering%20any%20of%20the%20desirable%20properties%20of%20that%20program)

白話一點就是：取代之後，不用改 code 的程序也不會因為我的替換 (substitution) 而壞掉。

舉個簡單的範例：

假設有兩個 class: `A` `B`, B 繼承 A

```js
class A {
  doWork() {
    console.log("do work");
  }
}

class B extends A {
  doAnotherWork() {
    console.log("do another work");
  }
}
```

然後在程式的某部分，使用了 A 的功能

```js
// ...
const a = new A();
a.doWork();
// ...
```

假如這時 a 替換成 b 程式會壞嗎？

```js
// ...
const a = new B(); // A 換成 B
a.doWork();
// ...
```

不會，程式還是能照常的運行，也不會有錯誤，
因此這個情況符合 LSP

這只是個簡單的範例，讓你了解替換的概念是什麼，而實際上有一些準則告訴我們，子類繼承父類時有哪些事情我要需要遵守，才不會破壞 LSP

## Design by contract (Contract Rules)

根據合約來設計程式，總共有三條規則需要遵守。

1. Preconditions cannot be strengthened in a subtype.
2. Postconditions cannot be weakened.
3. Invariants of the supertype must be preserved in a subtype.

### Preconditions cannot be strengthened in a subtype.

> 子類的前置條件不能比父類 strengthened （我會翻成嚴苛）

不過我們先來理解 Precondition (前置條件) 是什麼，有時候我們執行 function 時，會希望有滿足某些條件才做裡面的事

簡單的例子： 計算正方形面積

```js
function calcSquareArea(length) {
  return length * length;
}
```

但可能會希望 length 是大於 0 的數字，因此改成

```js
function calcSquareArea(length) {
  // if 的條件就是前置條件
  if (typeof length === "number" && length > 0) {
    return length * length;
  }
}
```

那如果子類要重新實作父類的 function 時，前置條件比較嚴格會怎樣？

```js
class A {
  getValue(value) {
    if (typeof value === "number") {
      return value + 10;
    }
  }
}

class B extends A {
  getValue(value) {
    // B 除了判斷是否為數字之外，還多一個限制是要大於 0
    if (typeof value === "number" && value > 0) {
      return value + 20;
    }
  }
}
```

某個地方的 class A 呼叫 getValue 替換成 class B 時有可能會壞掉

```js
let a = new A();
a.getValue(-10); // result 0

// 使用替換

let a = new B();
b.getValue(-10); // 什麼都沒有 undefined
```

這樣一來就不能替換了，因此違反 LSP

### Postconditions cannot be weakened

> 後置條件不能比較 weakened (我會翻成鬆散)

後置條件主要是為了確保 function 回傳的值是有效的，像是型別有沒有正確、資料範圍有沒有誤

舉個簡單例子：計算訂單金額

```js
function calcOrderPrice(orders) {
  let price = 0;
  // 計算訂單金額，假設有一系列算法：運費、會員身份...等

  //最後 return 前的後置條件
  if (typeof price === "number" && price > 0) {
    return price;
  }
}
```

一樣如果子類要重新實作父類的 function 時，後置條件比較鬆散會怎樣？

EX: 移除大於 0 判斷。

```js
function calcOrderPrice(orders) {
  let price = 0;
  if (typeof price === "number") {
    return price;
  }
}
```

這樣一來替換成子類的時候，client 端可能會拿到小於 0 的訂單金額，導致後續的系統流程有誤，因此不符合 LSP

### Invariants of the supertype must be preserved in a subtype

> 父類的 invariant 邏輯應該被子類保留 (`不變性`)

主要是在強調 `資料` 的不變性，假如父類別上有一些保護機制，能夠確保資料是有效的，而子類在實作時就應該維持這個特性，不該去改變它。

保護機制有可能像是前面提到的 precondition 或是 postcondition。

舉個例子：取得訂單價格

```js
class OrderPrice {
  constructor() {
    this.price = 0;
  }

  setPrice(price) {
    if (price < 0) {
      throw "price should not less than 0";
    }
    this.price = price;
  }
}

class VipUserOrderPrice extends OrderPrice {
  setPrice(price) {
    this.price = price;
  }
}

const vipOrderPrice = new VipUserOrderPrice();
vipOrderPrice.setPrice(-20);
```

在 contructor 時，父類為了避免價格小於零的訂單出現，因此設定了 price < 0 throw error 的條件，

而子類是個給 vip 客戶使用的類別，價格可以隨便調整，因此多了 setPrice 的 function，但這樣造成資料有可能出現小於 0 的狀況

問題又來，如果在有使用 OrderPrice 的地方，將它替換成 VipUserOrderPrice，會讓程式壞嗎？當然有機會，因此也不符合 LSP

在這個案例中，比較好的做法應該是將 set price 的邏輯，放回父類，由父類來保留他原生的邏輯來達到不變性。

```js
class OrderPrice {
  setPrice(price) {
    if (price < 0) {
      throw "price should not less than 0";
    }
    this.price = price;
  }
}
```

## 替換的目的

文章到目前為止都在說明什麼情況替換類別才符合 LSP，或是替換的條件有哪些以及該注意的點，但既然已經知道了這些，那我就會思考說為什麼要替換？它的好處是什麼？

我覺得主要是為了讓 client 端在使用這些子類別時，能夠對它有些基本的認知，因而不容易產生 bug，

還是要回到鳥的例子一下，假如宣告了一隻 Bird 的類別，上面有 fly、walk 的功能，然後另外宣告了企鵝的類別，讓其繼承 Bird

```js
class Bird {
  walk() {
    console.log("walk");
  }
  fly() {
    console.log("fly");
  }
}

class Penguin extends Bird {
  fly() {
    throw "I can not fly";
  }
}
```

讀到這裡我們也知道它違反了 LSP，在有使用 Bird 的地方，如果將他替換成 Penguin 一定有問題，企鵝就不會飛，只要一呼叫到程式就壞了，不過因為我們讀過生物，所以知道這件事情，因此可能就不會亂 call fly function。

但程式在開發時候，client 可能沒有那麼多時間去了解每個類別的細節，有時候會先透過父類的功能來對它有個基本認識，而如果這時子類有個 funtion 跟父類有衝突不能呼叫或是邏輯大不相同，
client 第一時間可能不會發現，導致 bug 的發生。

這種情況也不禁讓我們懷疑，他們之間真的適合繼承嗎？ 而這個父類別好像也做了兩件事情，要走又要飛似乎違反單一職責 (SRP)，因此我們可以稍微改一下

```js
class WalkBird {
  walk() {
    console.log('walk')
  }
}

class FlyBird {
  fly() {
    console.log("fly");
  }
}

class Penguin extends WalkBird {
  ...
}
```

## LSP 總結

LSP 雖然很強調替換的部分，但我覺得更重要的是，當我們要提升某個抽象概念的父類時（也是就是所謂 `一般化 (generalization)`），應該要想清楚他的功能組合，是否都能應用到子類別上並且達到合約要求。

## References

[wiki](https://en.wikipedia.org/wiki/Liskov_substitution_principle)
[The Liskov Substitution Principle - Microsoft Press Store](https://www.microsoftpressstore.com/articles/article.aspx?p=2255313&seqNum=2)
[object oriented design - Liskov substitution principle* clarification about the \_history rule* - Software Engineering Stack Exchange](https://softwareengineering.stackexchange.com/questions/408686/liskov-substitution-principle-clarification-about-the-history-rule)
[深入淺出 Liskov 替換原則 Liskov Substitution Principle](https://www.jyt0532.com/2020/03/22/lsp/)
