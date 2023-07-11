---
title: JavaScript - 提升(Hoisting)與暫時死區(TDZ)
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-08 23:00:26
---

JavaScript 在編譯階段，會掃描宣告的變數及函式，後儲存在 variable object 裡，使它們可以在宣告前被存取。

<!-- more -->

# 提升(Hoisting)

JavaScript 在執行程式碼之前(編譯階段)，會先掃描程式中宣告的變數，並將它們儲存在變數環境物件(variable environment object)中，使某些類型的變數在宣告之前允許被存取。

<table >
  <tr>
    <td></td>
    <td align="center">提升</td>
    <td align="center">原始值</td>
    <td align="center">作用域</td>
  </tr>
  <tr>
    <td align="center">函式宣告(function declaration)</td>
    <td align="center">⭕</td>
    <td align="center">實際函式</td>
    <td align="center">區塊作用域</td>
  </tr>
  <tr>
    <td align="center">var 宣告的變數</td>
    <td align="center">⭕</td>
    <td align="center">undefined</td>
    <td align="center">函式作用域</td>
  </tr>
  <tr>
    <td align="center">let / const 宣告的變數</td>
    <td align="center">⭕</td>
    <td align="center">&lt;uninitialized&gt;, TDZ</td>
    <td align="center">區塊作用域</td>
  </tr>
  <tr>
    <td align="center">函式表達式(function expression)、箭頭函式(arrow function)</td>
    <td  align="center" colspan="3">跟據 var 宣告或 let / const 宣告而不同</td>
  </tr>
</table>

## let 與 const 的提升

以 let 或 const 宣告的變數也會提升，但提升後的行為與 var 變數不同，初始值被設定為 unintialized，因此在宣告前存取變數會拋出 `ReferenceError: Cannot access 'a' before initialization` 的錯誤。

程式範例 :

```js
var a = 10
function foo() {
  console.log(a) // ReferenceError: Cannot access 'a' before initialization
  let a
}

foo()
```

## 為什麼需要提升 ?

1. 在任何地方呼叫函式，不一定要先宣告後呼叫。
2. 函式可以互相呼叫。

## 提升如何運作 ?

當呼叫一個函式時，即產生一個執行環境(Execution Context，以下簡稱 EC)，每個 EC 會有相對應的 variable object(以下簡稱 VO)，在 EC 中宣告的變數及函式都會儲存在 VO 中，若是函式，參數也會儲存到 VO 裡。

在進入 EC 時，JS 引擎會依照以下順序將變數儲存到 VO 中 :

對於函式的參數，會被儲存為 VO 中的屬性，若是沒有傳值，會被初始化為 undefined :

```js
function foo(a, b, c) {}
foo(10)
```

那 VO 會像 :

```js
{
  a: 10,
  b: undefined,
  c: undefined
}
```

對於函式宣告，同樣會儲存在 VO 裡面，值為函式的回傳值 :  
假如 VO 中已有同名屬性，就會將其值覆蓋，

```js
function foo(a) {
  function a() {}
}

foo(1)
```

所以 VO 會像 :

```js
{
  a: function a
}
```

最後是變數宣告，同樣存在 VO 裡面，值設為 undefined :  
若 VO 中已存在同名屬性，值不會改變。

```js
function foo(a) {
  console.log(a)
  var a = 20
}

foo(10)
```

因此 VO 會像 :

```js
{
  a: 10
}
```

最後總結一下，當程式執行進入 EC，會依照順序做三件事 :

1. 將參數存入 VO 並設定值，若無傳值則為 undefined。
2. 將函式宣告存入 VO，若已有同名屬性則將其值覆蓋。
3. 將變數宣告存入 VO，若已有同名屬性則將其忽略。

# 暫時死區(TDZ)

Temporal Dead Zone。指在變數「提升之後」及「賦值之前」的期間。

```js
// ---- firstName 的暫時死區 ----
console.log(firstName) // ReferenceError: Cannot access 'firstName' before initialization
// ----------------------------
const firstName = "ABen"

console.log(color) // ReferenceError: color is not defined
```

從上面的程式碼中可以看出，因為 JS 引擎在程式執行前會先掃描程式碼中的變數，進而知道變數尚未初始化，所以若在變數宣告之前存取變數，引擎會將暫時死區中的變數值設為 uninitialized。當程式執行到宣告變數的地方，離開該變數的暫時死區，才可以被存取。

## 為什麼需要暫時死區 ?

1. 避免在宣告前使用變數。
2. 為了實作 **const**。
   - 假如沒有暫時死區，若在以 const 宣告變數前存取該變數，那這個變數就會像以 var 宣告的變數一樣，初始值為 undefined，如此一來就違背 const 為常量的定義。

# 參考資料

[huli - 我知道你懂 hoisting，可是你了解到多深？](https://blog.techbridge.cc/2018/11/10/javascript-hoisting/)  
[Jonas Schmedtmann - The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
