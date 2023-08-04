---
title: JavaScript - 原型繼承(Prototypal Inheritance)與原型鏈(Prototype Chain)
tags:
  - JavaScript
categories:
  - JavaScript
---

<!-- more -->

# 物件間的繼承

在 JavaScript 中的繼承與 OOP 的繼承意義不同，從技術上來說，原型繼承更像是實例將方法委託給其原型，模擬 OOP 子類別繼承父類別方法。

## 構造函式實現

使用 `Object.create()` 指定一構造函式的 `prototype` 屬性為另一構造函式 `prototype` 屬性物件的原型，實現兩構造函式間的原型繼承。

```js
const Dog = function (firstName, birthYear) {
  this.firstName = firstName
  this.birthYear = birthYear
}

Dog.prototype.calcAge = function () {
  console.log(2023 - this.birthYear)
}

const Pet = function (firstName, birthYear, skill) {
  // call() 呼叫 Dog 函式
  Dog.call(this, firstName, birthYear)
  this.skill = skill
}

Pet.prototype = Object.create(Dog.prototype)
Pet.prototype.constructor = Pet

Pet.prototype.introduce = function () {
  console.log(`My name is ${this.firstName} and I can ${this.skill}.`)
}

const aben = new Pet("ABen", 2021, "bark at strangers")

console.log(aben) // Pet {firstName: 'ABen', birthYear: 2021, skill: 'bark at strangers'}
aben.introduce() // My name is ABen and I can bark at strangers.
aben.calcAge() // 2
```

使用 `Object.create()` 將 `Dog.prototype` 屬性物件設為 `Pet.prototype` 屬性物件的原型，使 `Pet` 的實例 `aben` 可以使用 `Dog` 上的 `calcAge()` 方法，但因為 `Object.create()` 不會設置新物件的 `constructor` 屬性，所以 `Pet.prototype.constructor` 仍然指向 `Dog` ，只能以 `Pet.prototype.constructor = Pet` 校正。

原型鏈示意圖 :

![構造函式間的原型鏈圖](constructor-function-prototypal-inheritance.webp)

## ES6 Classes 實現

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

class Pet extends Dog {
  constructor(firstName, birthYear, skill) {
    // super 負責創建子類別的 this
    super(firstName, birthYear)
    this.skill = skill
  }

  introduce() {
    console.log(`My name is ${this.firstName} and I can ${this.skill}.`)
  }
}
const aben = new Pet("ABen", 2021, "bark at strangers")

console.log(aben) // Pet {firstName: 'ABen', birthYear: 2021, skill: 'bark at strangers'}

aben.calcAge() // 2

aben.introduce() // My name is ABen and I can bark at strangers.
```

使用 `extends` 擴展類別作為子類別的父類，並於子類別的 `constructor()` 函式中呼叫 `super()` 方法，自動呼叫父類別的 `constructor()` 函式。

若子類別中只有與父類別相同的屬性，沒有新增自己的屬性，就可以不用定義 `constructor()` 函式，子類別會自動調用父類別的 `constructor()`。

## Object.create( ) 實現

`Object.create()` 不用在意 `prototype` 屬性與 `constructor()`方法，只要創建物件並指定原型，就可以實現物件間的原型繼承。

```js
const DogPrototype = {
  init(firstName, birthYear) {
    this.firstName = firstName
    this.birthYear = birthYear
  },

  calcAge() {
    console.log(2023 - this.birthYear)
  },
}

const PetPrototype = Object.create(DogPrototype)
PetPrototype.init = function (firstName, birthYear, skill) {
  DogPrototype.init.call(this, firstName, birthYear)
  this.skill = skill
}
PetPrototype.introduce = function () {
  console.log(`My name is ${this.firstName} and I can ${this.skill}.`)
}

const aben = Object.create(PetPrototype)
aben.init("ABen", 2021, "bark at strangers")

console.log(aben) // {firstName: 'ABen', birthYear: 2021, skill: 'bark at strangers'}

aben.calcAge() // 2

aben.introduce() // My name is ABen and I can bark at strangers.
```

原型鏈示意圖 :

![Object.create() 物件間的原型鏈圖](object-create-prototypal-inheritance.webp)

# 原型鏈(Prototype Chain)

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [面試官最愛考的 JS 原型鏈](https://maxleebk.com/2020/07/25/prototype/)
