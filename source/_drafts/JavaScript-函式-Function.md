---
title: JavaScript - 函式(Function)
tags:
  - JavaScript
categories:
  - JavaScript
---

在 JavaScript 中將函式視為一等公民，代表函式實際上被視為一個值，所以可以把函式作為參數傳給另一函式。

<!-- more -->

# 一級函式 vs. 高階函式

- 一級函式 : 為程式語言的特性。
- 高階函式 : 為一級函式特性的實現。

## 一級函式(First-Class Function)

- JavaScript 將函式視為一等公民(first-class citizens)。
- 函式實際上被視為一個值。
- 函式為另一種類型的物件。

因為 JS 將函式視為一個值，所以可以做到 :

```js
// 將函式存於變數或是物件方法中
const add = (a, b) => a + b

const aben = {
  name: "ABen",
  sayHi: function () {
    console.log(`${this.name} says Hi!`)
  },
}

// 將函式作為參數傳遞
const onClicked = () => console.log("🎉")

btn.addEventListener("click", onClicked)

// 函式回傳函式
function func() {
  return function () {
    console.log("🎊")
  }
}

// 使用函式的方法(method)
aben.sayHi.bind(someOtherObject)()
```

## 高階函式(Higher-Order Function)

因為 JS 中的函式為一級函式，所以才可以在程式中建立高階函式，高階函式的特點為**函式接收另一函式作為參數，或是函式回傳另一函式**。

```js
const onClicked = () => console.log("🎉")
btn.addEventListener("click", onClicked)
```

上面程式中的 `addEventListener` 即為高階函式，接收 `onClicked` 函式作為參數，在這裡 `onClicked` 被稱為回調函式(Callback function)。

# 立即執行函式表達式(IIFE)

立即執行函式表達式(Immediately invoked function expression)，為在 JavaScript 中只執行一次的函式。

```js
;(function () {
  console.log("這只會執行一次!")
})()
;(() => console.log("這也只會執行一次!"))()
```

在之前也用於封裝私有變數，因為外部作用域無法存取立即執行函式內的變數，所以可以達到變數私有化的效果，但 `let` 與 `const` 的出現帶來區塊作用域，也可以做到變數私有化，所以取代立即執行函式。

# 函式的方法

## bind

借用已建立的函式來創建新的函式，並將指定的物件綁訂為新函式的 `this` :

```js
const aben = {
  age: 2,
}
function getAge() {
  return this.age
}

const newFunc = getAge.bind(aben)

console.log(newFunc()) // 2
```

也可以在創建新函式的同時預先傳入參數，此種模式稱為部分應用程式(partial application) :

```js
const sum = (a, b) => {
  console.log(this) // window
  return a + b
}

const newFunc = sum.bind(this, 1)

console.log(newFunc(1)) // 2
```

與 Event Listener 配合使用 :

```html
<button class="btn">點我</button>
```

```js
const obj = {
  count: 0,
  add() {
    console.log(this) // <button class="btn">點我</button>
    this.count++
    console.log(this.count) // NaN
  },
}

document.querySelector(".btn").addEventListener("click", obj.add)
```

因為在事件監聽的回調函數中，`this` 為綁定事件的 DOM 元素，所以執行 `this.count++` 得到的結果為 `NaN`。

可以使用 `bind` 將事件回調函式的 `this` 綁定為 `obj` 物件，程式即可成功執行 :

```js
const obj = {
  count: 0,
  add() {
    console.log(this) // {count: 0, add: f}
    this.count++
    console.log(this.count) // 1
  },
}

document.querySelector(".btn").addEventListener("click", obj.add.bind(obj))
```

## call & apply

使用給定的物件呼叫函式，並將 `this` 綁定為該物件。

使用 `call` :

```js
const aben = {
  name: "ABen",
}

function callName(age, breed) {
  console.log(`${this.name} is a ${age}-year-old ${breed} breed dog.`)
}

callName.call(aben, 2, "mixed") // ABen is a 2-year-old mixed breed dog.
```

使用 `apply` :

```js
const aben = {
  name: "ABen",
}

function callName(age, breed) {
  console.log(`${this.name} is a ${age}-year-old ${breed} breed dog.`)
}

callName.apply(aben, [2, "mixed"]) // ABen is a 2-year-old mixed breed dog.
```

兩者差別在於傳遞參數的類型，`call` 為一般參數形式傳遞，`apply` 則是陣列形式傳遞。

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
