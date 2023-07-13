---
layout: post
title: JavaScript - 解構賦值(Destructuring Assignment)
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-12 22:45:33
---

解構賦值可以用來提取陣列或物件中的資料，讓原本可能需要迴圈或迭代的功能可以用更簡易的語句來達成。而展開運算符與其餘運算符可以讓我們更靈活的使用函式，在處理從 API 取得的資料時也更加容易。

<!-- more -->

# 解構陣列(Destructuring Array)

範例 :

```js
// 基本用法
const [a, b] = [1, 2] // a = 1, b = 2

// 略過某些值
const [a, , b] = [1, 2, 3] // a = 1, b = 3

// 交換變數值
let [a, b] = [1, 2]
;[a, b] = [b, a] // a = 2, b = 1

// 嵌套的陣列
const [a, [b, c]] = [1, [2, 3]] // a = 1, b = 2, c = 3

// 設定默認值
const [a = 0, b = 1, c] = [1] // a = 1, b = 1, c = undefined

// 字串
const [a] = "ABen" // a = A

// 其餘運算符(Rest Operator)
const [a, ...b] = [1, 2, 3] // a = 1, b = [2, 3]
```

# 解構物件(Destructuring Object)

範例 :

```js
// 基本用法
const { dog } = { dog: "ABen" } // dog = ABen

// 賦予新的變數名
const { dog: dogName } = { dog: "ABen" } // dogName = ABen

// 設定默認值
const { dog, age = 2, color } = { dog: "ABen" } // dog = ABen, age = 2, color = undefined

// 改變變數
let a = 111
let b = 222(({ a, b } = { a: 1, b: 2 })) // a = 1, b = 2
/*
 *  在 JS 中 `{}` 代表程式區塊，所以無法被賦值，需要在外部包 `()` 代表運算，才可正常執行。
 */

// 嵌套的物件
const {
  a: { b, c },
} = { a: { b: 2, c: 3 } } // b = 2, c = 3
```

# 展開運算符(Spread Operator)與其餘運算符(Rest Operator)

為 ES6 的新特性，兩者語法皆為 `...`。

## 展開運算符(Spread Operator)

可以將陣列展開成個別的值。在定義陣列或函式呼叫傳入陣列時使用。

組合陣列 :

```js
const arr = [1, 2, 3]
const newArr = [0, ...arr]
console.log(newArr) // [0, 1, 2, 3]
```

複製陣列 :

```js
const arr = [1, 2, [3, 4]]
const copyArr = [...arr]
console.log(copyArr) // [1, 2, [3, 4]]
```

需要注意的是展開運算符為淺複製(shallow-copy)，對於子物件只會複製其參考值(reference)。

呼叫函式時傳入陣列 :

```js
const arr = [1, 2, 3]

function foo(a, b, c) {
  console.log(a, b, c) // 1, 2, 3
}
foo(...arr)
```

只有可迭代的值可以使用展開運算符傳入函式。可迭代的值包含 String、Array、Map、Set，但不包括 Object。

組合物件 :

```js
const obj = { a: 1 }
const newObj = { ...obj, b: 2 }
console.log(newObj) // {a: 1, b: 2}
```

## 其餘運算符(Rest Operator)

將剩餘的值包裝成一個陣列。

解構陣列 :

```js
const [a, b, ...c] = [1, 2, 3, 4] // a = 1, b = 2, c = [3, 4]
```

特別注意，使用其餘運算符的元素必須要是最後一個元素，並且在每次解構賦值中只能使用一次。

解構物件 :

```js
const { a, ...obj } = { a: 1, b: { c: 2 } } // a = 1, obj = {b: {c: 2}}
```

函式其餘參數 :

```js
function sum(...args) {
  return args.reduce((a, b) => a + b)
}

sum(1, 2, 3, 4, 5) // 15

// 若參數原本為陣列
const arr = [1, 2, 3]
sum(...arr) // 6
```

當其餘參數沒有傳入實際值時，會變空陣列，而不是 `undefined`。

其餘參數的設計是為了取代函式中的 `arguments` 關鍵字，`arguments` 實際為包含所有傳入參數的類陣列(array-like)物件，所以本身不具備陣列相關的方法，因此不建議使用 `arguments`。

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [解構賦值· 從 ES6 開始的 JavaScript 學習生活](https://eyesofkids.gitbooks.io/javascript-start-from-es6/content/part4/destructuring.html)
- [展開運算符與其餘運算符· 從 ES6 開始的 JavaScript 學習生活](https://eyesofkids.gitbooks.io/javascript-start-from-es6/content/part4/rest_spread.html)
