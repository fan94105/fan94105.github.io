---
title: JavaScript - DOM
tags:
  - JavaScript
  - DOM
categories:
  - JavaScript
date: 2023-07-31 17:32:08
---


文件物件模型(Document Object Model，DOM)，為 HTML 文件的程式介面，提供 HTML 文件樹狀結構的模型，本身並不是 JavaScript 的一部分。

<!-- more -->

# DOM API 的組織

在 DOM tree 中的每個元素都是一個節點(Node)，每個節點以 JavaScript 物件表示，擁有許多屬性及方法。

節點分為四種類型 :

- 元素(Element) : 元素節點具有子類型 HTMLElement。
  - HTMLElement : 每個 HTML 元素都有各自的類型。
- 文字(Text)
- 注釋(Comment)
- 文檔(Document)

節點之間存在繼承關係，即子節點可以存取父節點的屬性與方法。

而在節點(Node)之上，存在一個抽象節點 EventTarget，其擁有事件監聽的方法，如 `addEventListener()`，所以在 DOM 中的其他節點才能夠進行事件監聽。

示意圖 :

![DOM API 組織圖](dom_organized.webp)  
(圖片來源 : The Complete JavaScript Course 2023: From Zero to Expert!)

# 操作 DOM

選取(Select)、創建(Create)與刪除(Delete) HTML 元素的各種方法。

## Select

`querySelector()` 回傳第一個符合選擇器(tag、id 或 class)的元素。

```js
const element = document.querySelector(selectors)
```

`querySelectorAll()` 回傳所有符合選擇器(tag、id 或 class)的節點清單(NodeList)。

```js
const elements = document.querySelectorAll(selectors)
```

`getElementById()` 回傳符合特定 id 選擇器的元素。

```js
const element = document.getElementById(id)
```

`getElementsByTagName()` 回傳特定標籤(tag)的 HTML 元素集合(HTMLCollection)。

```js
const elements = document.getElementsByTagName(name)
```

`getElementsByClassName()` 回傳符合特定的 class 選擇器的 HTML 元素集合(HTMLCollection)。

```js
const elements = document.getElementsByClassName(name)
```

**HTMLCollection 為動態集合**，當 DOM 更新時，在 HTMLCollection 中的元素也會自動更新。

## Create

`insertAdjacentHTML()` 將字串解析成 HTML，並插入到 DOM tree 中的指定位置，可以同時創建並插入元素。

```js
// <div></div>
const parent = document.querySelector("div")
const text = `<p>ABen 🐾</p>`
parent.insertAdjacentHTML("afterend", text)

console.log(parent)
/*
 *  <div>
 *    <p>ABen 🐾</p>
 *  </div>
 */
```

`createElement()` 創建 HTML 元素。

```js
const element = document.createElement(tagName)
```

在下面的 HTML 中插入元素 :

```html
<div>
  <p>ABen 🐾</p>
</div>
```

`prepend()` 在元素的第一個子節點前插入元素。

```js
const parent = document.querySelector("div")
const span = document.createElement("span")

parent.prepend(span)

console.log(parent)
/*
 *  <div>
 *    <span></span>
 *    <p>ABen 🐾</p>
 *  </div>
 */
```

`append()` 在元素的最後一個子節點後插入元素。

```js
const parent = document.querySelector("div")
const span = document.createElement("span")

parent.append(span)

console.log(parent)
/*
 *  <div>
 *    <p>ABen 🐾</p>
 *    <span></span>
 *  </div>
 */
```

`before()` 在節點前插入節點。

```js
const p = document.querySelector("p")
const span = document.createElement("span")

p.before(span)

console.log(parent)
/*
 *  <div>
 *    <span></span>
 *    <p>ABen 🐾</p>
 *  </div>
 */
```

`after()` 在節點後插入節點。

```js
const p = document.querySelector("p")
const span = document.createElement("span")

p.after(span)

console.log(parent)
/*
 *  <div>
 *    <p>ABen 🐾</p>
 *    <span></span>
 *  </div>
 */
```

## Delete

`remove()` 將指定元素從 DOM tree 中刪除。

```js
const element = document.querySelector(selectors)
element.remove()
```

較舊的方法是使用 `removeChild()` 從父元素中刪除指定的子節點 :

```js
const element = document.querySelector(selectors)
element.parentElement.removeChild(element)
```

# Style、Attribute 與 Class

## Style

```html
<div style="color: red">ABen 🐾</div>
```

```css
div {
  font-size: 20px;
}
```

`style()` 只能取得以行內樣式(Inline styles)設定的屬性。

```js
const div = document.querySelector("div")

console.log(div.style.color) // red
console.log(div.style.fontSize) // ""
```

`getComputedStyle()` 可以取得元素的所有樣式。

```js
const div = document.querySelector("div")

console.log(getComputedStyle(div).color) // "rgb(255, 0, 0)"
console.log(getComputedStyle(div).fontSize) // "20px"
```

`setProperty()` 設定或修改 CSS 變數 :

```css
:root {
  --color-primary: "red";
}
```

```js
document.documentElement.style.setProperty("--color-primary", "orangered")
```

## Attribute

存取 HTML 標籤中設定的屬性。

```html
<img
  src="img/aben.jpg"
  alt="ABen"
  class="img"
  age="2"
  data-version-number="1.0"
/>
```

```js
const img = document.querySelector("img")

console.log(img.src) // (絕對路徑) http://xxxx/img/aben.jpg
console.log(img.alt) // ABen
console.log(img.className) // img

// 存取非標準屬性
console.log(img.age) // undefined
console.log(img.getAttribute("age")) // 2

console.log(img.getAttribute("src")) // (相對路徑) img/aben.jpg

// 存取 data-*
console.log(img.dataset.versionNumber) // 1.0
```

當元素直接存取 `src` 屬性時，取得的會是絕對路徑，若是想要取得相對路徑，則需要使用 `getAttribute()` 方法。

當存取自定義的 `data-*` 屬性時，屬姓名必須轉換為 camelCase。

可以使用 `setAttribute()` 方法新增或更新屬性。

## Class

```js
// <div class="box"></div>
const box = document.querySelector(".box")

// 添加 class
box.classList.add("active", "hidden") // <div class="box active hidden"></div>

// 移除 class
box.classList.remove("active", "hidden") // <div class="box"></div>

// 切換 class，有則移除，無則添加
box.classList.toggle("hidden") // <div class="box hidden"></div>

// 是否包含 class
box.classList.contains("hidden") // true
```

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [MDN - Document](https://developer.mozilla.org/en-US/docs/Web/API/Document)
