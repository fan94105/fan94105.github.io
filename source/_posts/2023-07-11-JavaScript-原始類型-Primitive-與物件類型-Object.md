---
title: JavaScript - 原始類型(Primitive)與物件類型(Object)
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-11 14:25:54
---

原始類型(Primitive)與物件類型(Object)的差別在於兩者儲存的位置以及儲存的值，原始類型存於 Call Stack 中且儲存實際值，物件類型存於 Heap 中且儲存對物件的參考地址。

<!-- more -->

# 原始類型(Primitive)

在 JavaScript 中的原始數據類型有 :

- String
- Number
- Boolean
- undefined
- null
- BigInt
- Symbol

## call by value

原始數據類型操縱的值為實際的賦值(call by value) :

```js
let age = 2
let oldAge = age
age = 3

console.log(age) // 3
console.log(oldAge) // 2
```

上面程式碼中因為宣告的變數是原始數據類型，所以儲存於 Call Stack 中，首先宣告 `age` 變數，JS 以變數名稱創建唯一的標識符(Identifier)，然後分配一塊記憶體並提供記憶體位址(Address)，最後將賦值(Value) `2` 存到記憶體中。

第二步宣告 `oldAge` 等於 `age`，JS 先創建標識符，後分配到另一記憶體位置，再將 `age` 的值存入記憶體。

最後，將 `age` 重新賦值為 `3`，JS 通過唯一的標識符，取得儲存 `age` 的記憶體位址，再將原來的 `2` 改成 `3`。

在 JS 引擎中 :  
![原始類型儲存方式](primitive_in_engine.webp)

# 物件類型(Object)

在 JavaScript 中的物件類型有 :

- Object
- Function
- Array

## call by reference

物件類型操縱的值為對物件的參考地址(call by reference) :

```js
const aben = {
  age: 2,
}
const black = aben
black.age = 3

console.log("black:", black) // {age: 3}
console.log("aben:", aben) // {age: 3}
```

上面程式碼中，首先宣告 `aben` 物件，此物件會直接儲存在 Heap 裡面，JS 在 Call Stack 中創建唯一標識符 `aben`，分配記憶體空間，而裡面儲存的值為對 Heap 中物件記憶體位址的參考(reference)。

再來，宣告 `black` 物件等於 `aben` 物件，JS 同樣創建唯一標識符，分配記憶體空間，同樣儲存 Heap 中 `aben` 物件記憶體位址的參考，也就是說 `black` 與 `aben` 都指向同一物件。

最後 `black` 物件修改 `age` 值，因為 `black` 與 `aben` 實際上都指向同一物件，所以最終兩者都改為 `3`。

在 JS 引擎中 :  
![物件類型的儲存方式](object_in_engine.webp)

## call by sharing

讓作為參數傳入的物件與函式中的參數物件共享同一物件。

看看下面的範例 :

```js
const aben = {
  age: 2,
}
function changeAge(obj) {
  obj = {
    age: 3,
  }
  return obj
}

console.log(changeAge(aben)) // {age: 3}
console.log(aben) // {age: 2}
```

`aben` 物件竟然還是原來的值，若物件類型是 call by reference 的話，`aben` 應該要大一歲才對。

這是因為當 `aben` 物件作為參數傳入函式時，是讓函數中的 `obj` 參數與 `aben` 共享一個物件，若是使用 `obj.age` 的方式重新賦值，那 `aben` 的年齡確實會改變，但是實際上在函式中卻是將一個新的物件賦值給 `obj` 參數，這樣 `obj` 與 `aben` 就指向不同的物件，也就是說函式執行後另外創建了一個物件，因此 `aben` 的年齡並沒有改變。

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [從博物館寄物櫃理解變數儲存模型](https://hulitw.medium.com/variable-and-frontdesk-a53a0440af3c)
- [[筆記] 談談 JavaScript 中 by reference 和 by value 的重要觀念](https://pjchender.blogspot.com/2016/03/javascriptby-referenceby-value.html)
- [重新認識 JavaScript: Day 05 JavaScript 是「傳值」或「傳址」？](https://ithelp.ithome.com.tw/articles/10191057)
- [JS 基本觀念：call by value 還是 reference 又或是 sharing?](https://medium.com/@mengchiang000/js%E5%9F%BA%E6%9C%AC%E8%A7%80%E5%BF%B5-call-by-value-%E9%82%84%E6%98%AFreference-%E5%8F%88%E6%88%96%E6%98%AF-sharing-22a87ca478fc)
