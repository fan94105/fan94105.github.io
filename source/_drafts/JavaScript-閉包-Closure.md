---
title: JavaScript - 閉包(Closure)
tags:
  - JavaScript
categories:
  - JavaScript
---

<!-- more -->

# 閉包(Closure)

在 JavaScript 中，函式可以存取創建它的執行環境中變數環境(variable environment, VE)裡的變數，即使該執行環境已經從 Call Stack 中移除。

看看以下範例 :

```js
const secureBooking = function () {
  let passengerCount = 0

  return function () {
    passengerCount++
    console.log(`${passengerCount} passengers`)
  }
}

const booker = secureBooking()
```

分析上面程式的執行環境(EC) :  
![執行環境示意圖](callstack.webp)

1. 程式執行，創建全域執行環境(Global EC)與其內部變數環境物件(Variable Environment, VE)，VE 中包含 `secureBooking` 函式。
2. 執行 `secureBooking` 函式，創建其 EC 與 VO，VO 中包含 `passengerCount` 變數。
3. `secureBooking` 函式中回傳另一函式，並儲存於全域變數 `booker` 中。Global VO 中宣告 `booker` 函式。
4. `secureBooking` 函式執行完畢，從 Call Stack 中移除。

分析作用域(Scope) :

- 全域作用域中有 `secureBooking` 與 `booker` 函式。
- `secureBooking` 函式作用域中有 `passengerCount` 變數，且可藉由作用域鏈存取全域作用域中的 `secureBooking` 與 `booker` 函式。

當調用 `booker` 函式 :

```js
const secureBooking = function () {
  let passengerCount = 0

  return function () {
    passengerCount++
    console.log(`${passengerCount} passengers`)
  }
}

const booker = secureBooking()

booker() // 1 passengers
booker() // 2 passengers
```

![執行環境展現閉包示意圖](callstack_closure.webp)

`booker` 函式進入 Call Stack 中，雖然 `secureBooking` 函式已從 Call Stack 移除，但是上面程式顯示出 `booker` 函式仍然可以存取 `secureBooking` 函式 VE 中的變數，這就是閉包的作用。

## 閉包如何運作 ?

根據上例程式，在執行 `booker` 函式時，即產生了閉包，使 `booker` 函式在 `secureBooking` 函式移除後還可以存取到 `secureBooking` 中的變數，也使得這些變數永遠附加在 `booker` 函式中。

值得注意的是，閉包優先於作用域鏈，所以當第一次調用 `booker` 函式時，JS 引擎會先通過閉包尋找 `passengerCount` 變數，此時附加在 `booker` 函式中的 `passengerCount` 等於 `0`，於是執行 `passengerCount++` 後得到 `1 passengers` 的結果。

第二次調用 `booker` 函式時，執行相同流程，但此時附加在 `booker` 函式中的 `passengerCount` 等於 `1`，因此最終結果為 `2 passengers`。

# 閉包案例

## 重新賦值函式

並非只有被回傳的函式會產生閉包，在不同的函式作用域中重新賦值函式也會產生閉包。

```js
let f

const g = function () {
  const a = 1
  f = function () {
    console.log(a * 2)
  }
}

const h = function () {
  const b = 10
  f = function () {
    console.log(b * 2)
  }
}
g()
f() // 2
console.dir(f) // Closure(g) {a: 1}
h()
f() // 20
console.dir(f) // Closure(h) {b: 10}
```

上面程式碼執行 `g` 函式後，`f` 變數賦值為函式，因為是在 `g` 函式的 EC 中產生，所以可以存取 `g` 函式中的變數 `a`，之後執行 `f` 函式，執行時通過閉包找到依附的 `a` 變數，計算後得到結果 `2`。

接著執行 `h` 函式，將 `f` 變數重新賦值為另一函式，而這次是在 `h` 函式的 EC 中產生，所以可以存取的變數變為 `h` 函式中的 `b`，之後執行 `f` 函式，執行時通過閉包找到依附的 `b` 變數，計算後得到結果 `20`。

## 函式參數

函式也可以通過閉包存取宣告它的函式執行環境中呼叫時傳入的參數。

```js
const boardPassengers = function (n, wait) {
  const perGroup = n / 3

  setTimeout(function () {
    console.log(`We are now boarding all ${n} passengers`)
    console.log(`There are 3 groups, each group has ${perGroup} passengers`)
  }, wait * 1000)

  console.log(`Will start boarding in ${wait} seconds`)
}

boardPassengers(60, 3)
// Will start boarding in 3 seconds
// We are now boarding all 60 passengers
// There are 3 groups, each group has 20 passengers
```

上例中可以看到，`setTimeout` 的回調函式可以存取傳入的參數 `n` 以及 `perGroup` 變數，這是因為回調函式在 `boardPassengers` 函式中創建，所以可以通過閉包存取創建它的執行環境裡存在變數環境中的變數，而變數就包括宣告的變數以及呼叫函式時傳入的參數。

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
