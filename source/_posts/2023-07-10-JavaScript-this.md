---
title: JavaScript - this ?
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-10 14:19:44
---

在 JavaScript 中，`this` 指的是對目前執行環境的 ThisBinding。而在多數情況下，`this` 會因為函式的呼叫方式而有所不同。

<!-- more -->

# 關於 this

- 呼叫函式即生成 `this`，指向當前函式執行環境。
- `this` 非靜態值，影響它的因素為函式的呼叫方法。
- 多數情況下，`this` 代表呼叫函式的物件。

|        呼叫方法         |                          this                          |
| :---------------------: | :----------------------------------------------------: |
|         Method          |                     呼叫函式的物件                     |
|  Simple function call   | undefined(嚴格模式) / window(瀏覽器) / global(Node.js) |
|     Arrow function      |            父級函式的 `this` (lexical this)            |
|     Event listener      |                  添加事件的 DOM 元素                   |
| `call`、`apply`、`bind` |                     指定綁定的物件                     |

## 以方法形式(Method)呼叫函式

```js
const aben = {
  name: "ABen",
  year: 2021,
  calcAge: function () {
    return 2023 - this.year
  },
}
aben.calcAge() // 2
```

上面以 aben 物件呼叫 calcAge 方法，`this` 即指向 aben 物件，所以 `this.year` 就等於 2021。

## 直接呼叫函式(Simple function call)

在嚴格模式下， `this` 為 `undefined`。在非嚴格模式，瀏覽器中 `this` 為 `window`，Node.js 中 `this` 為 `global`。

```js
const obj = {
  foo: function () {
    console.log(this) // obj

    const boo = function () {
      console.log(this) // window
    }
    boo()
  },
}

obj.foo()
```

上面程式中雖然 boo 函式是在 obj 物件的 foo 方法中被呼叫，但因為 boo 函式中沒有特別指明 `this`，所以預設綁定為 `window`。

所以 **this 代表的是呼叫函式的物件，而非函式本身**。

而為了使 boo 函式的 `this` 綁定為 obj 物件，在較舊的程式碼中通常會宣告 `self` 變數儲存 foo 函式的 `this` :

```js
const obj = {
  foo: function () {
    console.log(this) // obj

    const self = this
    const boo = function () {
      console.log(self) // obj
    }
    boo()
  },
}

obj.foo()
```

而在 ES6 之後的方法為使用箭頭函式 :

```js
const obj = {
  foo: function () {
    console.log(this) // obj

    const boo = () => {
      console.log(this) // obj
    }
    boo()
  },
}

obj.foo()
```

## 箭頭函式(Arrow function)

箭頭函式本身沒有 `this`，它的 `this` 必須從父級函式或父級作用域取得。

```js
const obj = {
  foo: function () {
    const boo = () => {
      console.log(this) // obj
    }
    boo()
  },
}

obj.foo()
```

上面程式碼中因為 boo 函式為箭頭函式，所以 `this` 為從父級函式 foo 中所取得，指向 obj 物件。

陷阱範例 :

```js
const obj = {
  foo: () => {
    console.log(this) // window
  },
}

obj.foo()
```

雖然上面程式中以 obj 呼叫 foo 函式，但因為 foo 函式為箭頭函式，所以從父級作用域取得的 `this` 為 `window`。

特別注意，建立物件使用的 `{}` 不具有區塊作用域 ! 所以 foo 函式的父級作用域為全域作用域。

因此**永遠不要使用箭頭函式作為物件的方法**。

## 事件監聽(Event listener)

事件監聽中的 `this` 為綁定事件的 DOM 元素。

```html
<button id="btn">點我</button>
```

```js
const btnEl = document.querySelector("#btn")

btnEl.addEventListener("click", function (e) {
  console.log(this) // <button id="btn">點我</button>
})
```

特別注意，若回調函式為箭頭函式，則 `this` 為從父級作用域也就是全域作用域取得之 `window`。

## bind 與 call / apply

使用 `bind`、`call`、`apply` 來強制綁定 `this`。

### bind

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

也可以在創建新函式的同時預先傳入參數 :

```js
const sum = (a, b) => {
  console.log(this) // window
  return a + b
}

const newFunc = sum.bind(this, 1)

console.log(newFunc(1)) // 2
```

### call / apply

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

[What's THIS in JavaScript ? ](https://kuro.tw/posts/2017/10/12/What-is-THIS-in-JavaScript-%E4%B8%8A/)  
[The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
