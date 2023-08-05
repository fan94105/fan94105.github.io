---
title: JavaScript - 封裝(Encapsulation)
tags:
  - JavaScript
  - OOP
categories:
  - JavaScript
date: 2023-08-05 18:32:43
---


OOP 中的封裝，為類別擁有私有屬性或方法，使外部直接無法存取，必須呼叫特定的方法才可以存取類別內部的狀態，因此可以保護類別內的狀態不受外部引響。

<!-- more -->

# 受保護的屬性與方法

防止類別之外的程式存取類別內的屬性與方法，在 JS 中通常以 `_` 為前綴，表示為受保護的屬性或方法，不該從外部被存取。

下面以一個銀行 app 的功能為例 :

```js
class Account {
  constructor(owner, currency, pin) {
    this.owner = owner
    this.currency = currency
    this.pin = pin
    this.locale = navigator.language

    // Protected property
    this._movements = []
  }

  getMovements() {
    return this._movements
  }

  deposit(amount) {
    this._movements.push(amount)
  }

  withdraw(amount) {
    this.deposit(-amount)
  }

  // Protected method
  _approveLoan(amount) {
    return true
  }

  requestLoan(amount) {
    if (this._approveLoan(amount)) {
      this.deposit(amount)
    }
  }
}

const acc = new Account("ABen", "TWD", 5428)

acc.deposit(100)
acc.withdraw(50)
acc.requestLoan(300)

console.log(acc.getMovements()) // [100, -50, 300]
```

因為在外部不應該直接存取 `movements` 屬性，所以就將它設為受保護的屬性，並且定義 `getMovements()` 方法為公開介面(public interface)讓外部藉由呼叫此方法取得 `movements` 值。

存取受保護屬性的方法可以使用 `setter` 與 `getter` 語法，或是使用一般的函式，一般函式更加靈活，且可以接受多個參數，通常將它們以 `set` 或 `get` 開頭命名。

# 私有的類別屬性(Class field)與方法

類別屬性(Class field)為定義於任何函式之外的屬性，會存在於每一個實例中，而不會存在類別本身。

- 公開屬性(Public field) : 定義方式如同定義變數並賦值。
- 私有屬性(Private field) : 以 `#` 為前綴開頭，外部無法存取此屬性。

```js
class Account {
  // Public field (instance property)
  locale = navigator.language

  // Private field
  #pin
  #movements = []

  constructor(owner, currency, pin) {
    this.owner = owner
    this.currency = currency
    this.#pin = pin
  }

  getMovements() {
    return this.#movements
  }

  deposit(amount) {
    this.#movements.push(amount)
  }

  withdraw(amount) {
    this.deposit(-amount)
  }

  // Private method
  #approveLoan(amount) {
    return true
  }

  requestLoan(amount) {
    if (this.#approveLoan(amount)) {
      this.deposit(amount)
    }
  }
}

const acc = new Account("ABen", "TWD", 5428)

acc.deposit(100)
acc.withdraw(50)
acc.requestLoan(300)

console.log(acc.getMovements()) // [100, -50, 300]

console.log(acc.#movements) // Uncaught SyntaxError: Private field '#movements' must be declared in an enclosing class
```

子類別也無法直接存取從父類別的私有屬性，需要使用 `getter` 或 `setter` 存取。

```js
class SubAccount extends Account {
  constructor(owner, currency, pin, mainAccount) {
    super(owner, currency, pin)
    this.mainAccount = mainAccount
  }

  showPin() {
    return this.#pin
  }
}

const subAcc = new SubAccount("ABen", "TWD", 2845, acc)

subAcc.showPin() // Uncaught SyntaxError: Private field '#pin' must be declared in an enclosing class
```

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
