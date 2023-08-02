---
title: JavaScript - 物件導向(Object-oriented programming，OOP)
tags:
  - JavaScript
  - OOP
categories:
  - JavaScript
---

物件導向程式設計(Object-oriented programming，OOP)是一種以物件概念為基礎的程式撰寫方式，使用物件模擬現實生活中的事物。

<!-- more -->

# OOP

## 基本概念

類別(class)可以視為定義物件的藍圖，其中包含事物的狀態(屬性)或行為(方法)。

類別的實例(instance)就是透過類別這張藍圖所創建出來的物件。創建的過程稱為實例化(instantiation)。

## 基本原則

設計類別的基本原則 :

- **抽象(Abstraction)** : 忽視其他無關的細節，只專注於所要實施的部分，從而獲得事物的概觀，而非無關的細枝末節。
- **封裝(Encapsulation)** : 在類別中擁有外部無法存取的私有屬性與方法。某些方法可以被暴露(exposed)作為公開介面(public interface)。
  - 避免外部程式操作內部狀態(state)。
  - 改變內部程式時不會影響外部。
- **繼承(Inheritance)** : 使子類別可以存取父類別的屬性與方法，並與父類別形成階層結構。
  - 可以減少重複的程式碼。
  - 更好的模擬現實事物之間的關係。
- **多型(Polymorphism)** : 子類別可以覆蓋從父類別繼承而來的方法。

# OOP in JavaScript

JavaScript 中以原型(prototype)表示類別，而每一個物件都連結到原型物件，透過原型繼承(prototypal inheritance)使物件可以存取原型物件中的屬性及方法。

> JS 中原型繼承(prototypal inheritance)與 OOP 中繼承(inheritance)的差別 :
>
> - 原型繼承是物件實例繼承原型物件的屬性及方法，或者也可以說是實例物件將方法委託給原型物件。
> - 繼承是子類別繼承父類別的屬性及方法。

## 原型的實現

三個實現原型的方法 :

- 構造函式(Constructor function) : 用於創建物件的函式，內建物件如 Array、Set 或 Map 就是以構造函式創建。
- ES6 Classes : 構造函式的語法糖。
- Object.create() : 連結原型物件最簡單且直接的方法。

### 構造函式(Constructor function)

構造函式可以使用函式聲明或是函式表達式，但不能使用箭頭函式，因為箭頭函示沒有 `this` 綁定。

使用 `new` 執行 :

1. 創建一個物件。
2. 呼叫構造函式，`this` 綁訂為新物件。
3. 新物件連結到原型物件，設置 `__proto__` 屬性並指向構造函式的 `prototype` 屬性。
4. 構造函式回傳新物件，此物件即構造函式的實例。

在構造函式中使用 `this` 添加屬性，而方法並不會定義於構造函式中，而是使用 `prototype` 屬性將方法添加於原型物件上，使用原型繼承的特性。如果將方法定義於構造函式，當創造出多個實例時，那麼每一個實例上都有相同的方法，這對於程式效能會產生很大的影響。

```js
const Dog = function (firstName, birthYear) {
  //console.log(this) // Dog {}

  // 實例屬性(Instance properties)
  this.firstName = firstName
  this.birthYear = birthYear
}

// 在 Dog.prototype 上添加屬性
Dog.prototype.species = "C. familiaris"

// 在 Dog.prototype 上添加方法
Dog.prototype.calcAge = function () {
  console.log(2023 - this.birthYear)
}
console.log(Dog.prototype) // {calcAge: f, constructor: f}

const aben = new Dog("ABen", 2021)
console.log(aben) // Dog { firstName: 'ABen', birthYear: 2021 }

// Dog 的實例物件通過原型呼叫 calcAge 函式
aben.calcAge() // 2
```

實例的 `__proto__` 屬性指向構造函式的 `prototype` 屬性。

```js
console.log(Dog.prototype === aben.__proto__) // true
```

可以使用 `isPrototypeOf()` 方法確認物件的原型，特別注意構造函式的 `prototype` 屬性並非指向構造函式的原型，而是指向構造函式創建的實例的原型。

```js
console.log(Dog.prototype.isPrototypeOf(aben)) // true
console.log(Dog.prototype.isPrototypeOf(Dog)) // false
```

使用 `hasOwnProperty()` 方法確認屬性是否為實例屬性。

```js
console.log(aben.hasOwnProperty("firstName")) // true
console.log(aben.hasOwnProperty("species")) // false
```

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [Wikipedia - Abstraction (computer science)](<https://en.wikipedia.org/wiki/Abstraction_(computer_science)>)
