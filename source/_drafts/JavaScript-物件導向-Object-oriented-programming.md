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

三個實現原型繼承的方法 :

- 構造函式(Constructor function) : 用於創建物件的函式，內建物件如 Array、Set 或 Map 就是以構造函式創建。
- ES6 Classes : 構造函式的語法糖。
- Object.create() : 連結原型物件最簡單且直接的方法。

## 構造函式(Constructor function)

構造函式可以使用函式聲明或是函式表達式，但不能使用箭頭函式，因為箭頭函示沒有 `this` 綁定。

使用 `new` 執行 :

1. 創建一個物件。
2. 呼叫構造函式，`this` 綁訂為新物件。
3. 新物件連結到原型物件，設置 `__proto__` 屬性並指向構造函式的 `prototype` 屬性。
4. 構造函式回傳新物件，此物件即構造函式的實例。

在構造函式中使用 `this` 添加屬性，而方法並不會定義於構造函式中，而是使用 `prototype` 屬性將方法添加於原型物件上，使用原型繼承的特性。如果將方法定義於構造函式，當創造出多個實例時，那麼每一個實例上都有相同的方法，這對於程式效能會產生很大的影響。

範例 :

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

### 靜態方法(static method)

靜態方法為依附在構造函式上的方法，不是在構造函式 `prototype` 屬性上的方法，所以只能由構造函示呼叫，而實例物件無法呼叫。

```js
const Dog = function (firstName, birthYear) {
  this.firstName = firstName
  this.birthYear = birthYear
}

// hey 方法存在於 Dog 構造函式中
Dog.hey = function () {
  console.log("Hey there 👋")
}

const aben = new Dog("Aben", 2021)

aben.hey() // Uncaught TypeError: aben.hey is not a function

Dog.hey() // Hey there 👋
```

## ES6 Classes

JavaScript 中的 `class` 並不是 OOP 中的類別(class)，它只是構造函式的語法糖，作用與構造函式相同。

語法 :

```js
// class declaration
class className {}

// class expression
const className = class {}
```

範例 :

```js
class Dog {
  constructor(firstName, birthYear) {
    this.firstName = firstName
    this.birthYear = birthYear
  }

  calcAge() {
    console.log(2023 - this.birthYear)
  }
}

const aben = new Dog("Aben", 2021)

console.log(aben) // Dog { firstName: 'Aben', birthYear: 2021 }

aben.calcAge() // 2
```

`constructor()` 構造器為 `class` 中的方法，相當於構造函式。

`calcAge()` 方法會自動添加在 `Dog.prototype` 中。

要特別注意 :

- `class` 不會提升(hoisted)。
- `class` 是一等公民(first-class citizen)，表示它可以做為函式參數，也可以被函式回傳。
- `class` 內部執行嚴格模式(strict mode)。

### 靜態方法(static method)

在 `class` 語法中靜態方法以 `static` 表示。

```js
class Dog {
  constructor(firstName, birthYear) {
    this.firstName = firstName
    this.birthYear = birthYear
  }

  // 實例方法(instance method)
  calcAge() {
    console.log(2023 - this.birthYear)
  }

  // 靜態方法(static method)
  static hey() {
    console.log("Hey there 👋")
  }
}

const aben = new Dog("Aben", 2021)

aben.hey() // Uncaught TypeError: aben.hey is not a function

Dog.hey() // Hey there 👋
```

### Getter & Setter

在 JS 物件中有兩種屬性，一般的屬性與存取器屬性(accessor property)。getter 與 setter 就屬於存取器屬性，本身其實是存取物件內數據的方法。

兩者差異 :

- getter : 用於取得物件中對屬性進行計算後得到的數據。
- setter : 用於設定物件中的屬性。

在一般的物件中使用 `get` 與 `set` 分別定義 getter 與 setter，使用時要注意它們是屬性而非方法。

```js
const account = {
  owner: "ABen",
  movements: [200, 100, 120],

  get latest() {
    return this.movements.slice(-1).pop()
  },

  set latest(mov) {
    this.movements.push(mov)
  },
}

console.log(account.latest) // 120

account.latest = 50

console.log(account.movements) // (4) [200, 100, 120, 50]
```

在 `class` 中 getter 與 setter 的使用方法與在一般物件中相同，且常用於驗證數據。

```js
class Dog {
  constructor(fullName, birthYear) {
    this.fullName = fullName
    this.birthYear = birthYear
  }

  get age() {
    return 2023 - this.birthYear
  }

  set fullName(name) {
    if (name.includes(" ")) this._fullName = name
    else alert(`${name} is not a full name!`)
  }

  get fullName() {
    return this._fullName
  }
}

const aben = new Dog("Aben Chang", 2021)

console.log(aben) // Dog {_fullName: 'Aben Chang', birthYear: 2021}

console.log(aben.age) // 2
console.log(aben.fullName) // Aben Chang
```

上面程式中使用 setter 驗證名字是否為全名，當 `constructor()` 中執行 `this.fullName = fullName` 時，即執行同名 setter 檢查輸入是否為全名，若是全名則存於 `_fullName` 屬性。

## Object.create( )

`Object.create()` 可以指定原型物件，也就是說將物件的 `__proto__` 屬性指向任何其他物件。

```js
const Dog = {
  init(firstName, birthYear) {
    this.firstName = firstName
    this.birthYear = birthYear
  },

  calcAge() {
    console.log(2023 - this.birthYear)
  },
}

const aben = Object.create(Dog) // {}

console.log(aben.__proto__ === Dog) // true

aben.init("Aben", 2021)

console.log(aben) // {firstName: 'Aben', birthYear: 2021}
```

上面程式中 `Object.create()` 創建一個空物件，並指定 Dog 為其原型物件。通常設定原型屬性的方法是在原型物件中定義一個像是 `constructor()` 的函式，如程式碼中的 `init()`，盡量不要使用像是 `aben.firstName = "ABen"` 直接賦值的方法添加屬性。

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [Wikipedia - Abstraction (computer science)](<https://en.wikipedia.org/wiki/Abstraction_(computer_science)>)
