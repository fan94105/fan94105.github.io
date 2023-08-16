---
title: JavaScript - Module
tags:
  - JavaScript
categories:
  - JavaScript
---

<!-- more -->

# ES6 Module

```html
<script type="module" src="index.js"></script>
```

預設為 `defer`。

## 使用

### Named export

shoppingCart.js :

```js
const shippingCost = 10
const cart = []

// 定義時直接導出
export const addToCart = function (product, quantity) {
  cart.push({ product, quantity })
  console.log(`${quantity} ${product} added to cart`)
}

// 定義後一起導出
export { shippingCost }
```

定義後導出可以使用 `as` 設定別名。

index.js :

```js
import { addToCart, shippingCost as cost } from "./shoppingCart.js"

addToCart("phone", 2) // 2 phone added to cart

console.log(cost) // 10
```

使用 `as` 為導入的變數或函式設定別名。

或使用 `＊` 全部導入 :

```js
import * as shoppingCart from "./shoppingCart.js"

shoppingCart.addToCart("phone", 2) // 2 phone added to cart
```

### Default export

Module 中默認導出的一個值。

shoppingCart.js :

```js
export default function (product, quantity) {
  cart.push({ product, quantity })
  console.log(`${quantity} ${product} added to cart`)
}
```

index.js :

```js
import add from "./shoppingCart.js"

add("phone", 2) // 2 phone added to cart
```

導入時為默認導出值命名。

## 特性

- ES6 Module 預設執行嚴格模式(strict mode)。
- `import` 語句會提升(hoisting)。

## Module 執行流程 :

解析 index.js :

Module 以同步(synchronous)導入 index.js，使程式可以在最後執行 bundling 與 dead code elimination。

因為一個應用程式不會使用到模組中的所有程式碼，在使用 webpack 或 Parcel 等打包工具時，會執行 dead code elimination，刪除未使用的程式，所以`import` 與 `export` 語句必須在需要執行的程式區塊之外。

下載並執行 Module :

Module 以異步(asynchronous)下載，將 index.js 中的 `import` 與 Module 中的 `export` 連接，此連接並非複製，而是參考(reference)，然後執行 Module 中的程式。

# Module Pattern

使用閉包(Closure)特性與 IIFE(Immediately Invoked Function Expression) 實作模組，避免汙染全域變數環境，創建私有方法與屬性。

```js
const shoppingCart = (function () {
  const cart = []
  const shippingCost = 10

  const addToCart = function (product, quantity) {
    cart.push({ product, quantity })
    console.log(
      `${quantity} ${product} added to cart(shipping cost is ${shippingCost})`
    )
  }

  const orderStock = function (product, quantity) {
    console.log(`${quantity} ${product} ordered from supplier`)
  }

  return {
    addToCart,
    cart,
  }
})()

console.log(shoppingCart2) // {cart: Array(0), addToCart: ƒ}

shoppingCart.addToCart("pizza", 4) // 4 pizza added to cart(shipping cost is 10)
```

缺點 :

- 若要像 ESM 一樣一個 script 一個 Module 的話，要注意 HTML 中載入的順序。
- 無法使用 Module Bundler 打包。

# CommonJS Module

# 參考資料
