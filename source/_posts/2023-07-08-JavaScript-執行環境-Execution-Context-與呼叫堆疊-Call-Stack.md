---
title: JavaScript - 執行環境(Execution Context)與執行堆疊(Call Stack)
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-08 18:13:24
---

每呼叫一個函式就會產生一個 Execution Context，並且放入 Call Stack 中等待執行，JavaScript 就是藉由 Call
Stack 來追蹤程式的運行。

<!-- more -->

# 執行環境(Execution Context)

全域與函式程式碼的 JS 內部構造。

JavaScript 的兩種執行環境 :

- 全域執行環境(Global Execution Context)
- 函式執行環境(Function Execution Context)

執行環境包含兩個階段 : 創建階段(creation phase)、執行階段(execution phase)。

## 全域執行環境(Global Execution Context)

一開始執行 JS 程式碼時所創建的執行環境。

創建階段(creation phase) :

1. 創建 global object。(瀏覽器的 window object 或 Node.js 的 global object。)
2. 建立 scope。
3. 創建 `this`，並綁定至 global object。
4. 將 variables、class 和 function 分配至記憶體。(hoisting)

執行階段(execution phase) :

1. 逐行執行程式碼。
2. 遇到遞迴時，使用 Call Stack 排定執行順序。

## 函式執行環境(Function Execution Context)

每次調用函式即創建的執行環境。

創建階段(creation phase) :

1. 創建 arguments object。
2. 建立 scope。
3. 創建 `this`。
4. 將 variables、class 和 function 分配至記憶體。(hoisting)

執行階段(execution phase) :

1. 逐行執行程式碼。
2. 遇到遞迴時，使用 Call Stack 排定執行順序。

# 呼叫堆疊(Call Stack)

是 JS 引擎追蹤本身在調用多個函式的程式碼中所在位置的機制。可以幫助我們知道 JS 引擎當前正在運行什麼函式，以及該從函式中調用那些函式。

JavaScript 為單執行緒(single thread)的程式語言，指一次只能執行一個任務，而其他任務會依序添加到堆疊中，等待被執行。

> 堆疊(Stack) : 具有後進先出(LIFO, Last In First Out)特性的資料結構。

執行以下程式碼 :

```js
function func1() {
  console.log("func1 開始執行")
  func2()
  console.log("func1 結束執行")
}

function func2() {
  console.log("func2 開始執行")
  console.log("func2 結束執行")
}

func1()
// func1 開始執行
// func2 開始執行
// func2 結束執行
// func1 結束執行
```

Call Stack 看起來會像 :  
![Call Stack](call_stack.jpg)

執行流程 :

1. 首先處於全域執行環境(Global Execution Context)的程式會先進入 stack。
2. func1 被呼叫，放入 stack 的最上方，並執行函式。執行 func1 過程中呼叫 func2，停止執行 func1。
3. func2 被呼叫，將其放入 stack 的最上方，並執行函式。執行結束後將函式從 stack 中移除。
4. 從 func1 停止位置繼續執行程式，完成後從 stack 中移除。
5. 程式執行完畢，便將全域執行環境從 stack 中移除。

> 若 Call Stack 堆疊過高，超出記憶體所分配的空間，會導致 **stack overflow** 的問題。

# 參考資料

[Wilson Ren - 2023 網頁全端開發](https://www.udemy.com/course/wilson-full-stack-web-development/)  
[Jonas Schmedtmann - The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
