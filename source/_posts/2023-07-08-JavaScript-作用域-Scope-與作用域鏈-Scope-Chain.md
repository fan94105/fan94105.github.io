---
title: JavaScript - 作用域(Scope)與作用域鏈(Scope Chain)
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-08 20:32:19
---

作用域定義 JavaScript 中變數的影響範圍。作用域鏈為作用域對父級作用域的引用形成的關聯。

<!-- more -->

# 作用域(Scope)

程式碼中定義變數時，這些變數在程式中的可見性(visibility)和可存取性(accessibility)的範圍。

[w3schools][1] 上描述 :

> Scope determines the accessibility (visibility) of variables.

JavaScript 有下列作用域 :

- 全域作用域(Global Scope)
- 函式作用域(Function Scope)
- 區塊作用域(Block Scope)

## 全域作用域(Global Scope)

在函式(function)及區塊(block)外宣告的變數，可以在任何地方被存取。

```js
var name = "ABen" // Global scope

let age = 2 // Global scope

const color = "brindle" // Global scope
```

## 函式作用域(Function Scope)

每一個函式都會建立一個函式作用域。在函式內部宣告的變數，只能在函式中被存取。

```js
function func() {
  let name = "ABen"
  console.log(name) // ABen
}

func()
```

以 `var` 宣告的變數具有函式作用域 :

```js
function func() {
  if (true) {
    var scope = "function scope"
  }

  console.log(scope) // function scope
}

func()

console.log(scope) // ReferenceError: scope is not defined
```

在**嚴格模式**下，函式為區塊作用域 :

```js
"use strict"

if (true) {
  function func() {
    console.log("block scope")
  }

  func() // block scope
}

func() // ReferenceError: func is not defined
```

## 區塊作用域(Block Scope)

因 ES6 的 `let` 與 `const` 變數宣告，`{}`內部宣告的變數於外部無法存取。

```js
for (let i = 0; i < 5; i++) {
  console.log(i)
}

console.log(i) // Uncaught ReferenceError: i is not defined
```

# 作用域鏈(Scope Chain)

作用域擁有對父級作用域變數的引用，因此多個嵌套的作用域即形成作用域鏈。

變數查找(Variable Lookup) : 當一個作用域中需要使用某個變數，但在當前作用域下無法存取時，會在作用域鏈中查找，父層作用域是否存在該變數，若最終在全域沒找到則拋出錯誤。

```js
const name = "ABen"

function first() {
  const age = 2

  function second() {
    const color = "brindle"
    console.log(`${name} is a ${age}-year-old ${color} dog.`)
  }
  second()
}

first() // ABen is a 2-year-old brindle dog.
```

程式碼中在 second 函式存取的 `name` 與 `age` 並非是在自身函式內部宣告的變數，而是通過作用域鏈向外部作用域查找所取得的變數。

# 語彙環境(Lexical Environment)

是 JS 引擎內部的(抽象)結構，儲存變數和函式名稱與對應的物件或原始值之間的映射關係。每個語彙環境會保有對父級語彙環境的引用。

當定義一個變數或函式時，它們會被儲存在當前的語彙環境中。當在程式碼中使用這些變數時，JS 引擎會通過查找語彙環境鏈來找到對應的值。

語彙環境由兩部分組成 :

1. 環境紀錄(Environmental Record) : 保存當前語彙環境的變數資料。
2. 外部語彙環境的引用(a reference to outer lexical environment) : 保存對父級語彙環境的引用。

# 語彙範疇(Lexical Scope)

程式執行時 JavaScript 是如何將值對應到正確的變數呢 ?

程式語言一般有兩種類型 :

1. 靜態範疇(Static Scoping) :
   - 注重變數在程式碼中聲明的位置。當前位置找不到變數則向父級查找。
   - C、Java、JavaScript、Python 等語言採用靜態範疇。
2. 動態範疇(Dynamic Scoping) :
   - 注重函式呼叫的位置。
   - Perl 採用動態範疇。

程式範例 :

```js
let name = "ABen"

function foo() {
  console.log(name)
}

function boo() {
  let name = "Black"
  foo()
}

boo() // ABen
```

上面的程式在全域作用域及函式作用域中分別各宣告一個 `name` 變數，在 boo 函式執行時調用了 foo 函式，因為 JavaScript 採用靜態範疇，所以當在 foo 中找不到 `name` 時，不會因為是在 boo 中被呼叫就使用 boo 中宣告的 `name`，而是向當前位置的父級範疇查找，最後就在全域中取得變數。

# 參考資料

[The Complete JavaScript Course 2023: From Zero to Expert!][2]  
[Scope, Lexical Environment and Scope Chain in JavaScript][3]  
[[JavaScript] Javascript 的作用域 (Scope) 與範圍鏈 (Scope Chain)：往外找][4]

[1]: https://www.w3schools.com/js/js_scope.asp
[2]: https://www.udemy.com/course/the-complete-javascript-course/
[3]: https://medium.com/@anilakgunes/scope-lexical-environment-and-scope-chain-in-javascript-559aadb7dca8
[4]: https://medium.com/itsems-frontend/javascript-scope-and-scope-chain-ca17a1068c96
