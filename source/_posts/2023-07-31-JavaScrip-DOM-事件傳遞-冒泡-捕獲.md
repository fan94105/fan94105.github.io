---
title: JavaScript - DOM 事件傳遞(冒泡 & 捕獲)
tags:
  - JavaScript
  - DOM
categories:
  - JavaScript
date: 2023-07-31 17:34:00
---


當元素綁定的事件被觸發時，會經過三個階段，捕獲階段(CAPTURING_PHASE)、目標階段(AT_TARGET)與冒泡階段(BUBBLING_PHASE)。

<!-- more -->

# 事件傳遞

首先看看事件傳遞的表現，在 HTML 中設定一個外部元素 `outer` 與內部元素 `inner`。

```html
<div class="outer">
  <div class="inner"></div>
</div>
```

將兩個元素分別綁定點擊事件。

```js
const outer = document.querySelector(".outer")
const inner = document.querySelector(".inner")

outer.addEventListener("click", e => {
  console.log("outer")
})
inner.addEventListener("click", e => {
  console.log("inner")
})
```

執行的結果 :

![事件傳遞的表現](dom-bubbling.webp)

可以發現，當內部元素 `inner` 被點擊時，在 console 中不只印出 "inner"，也印出了 "outer"，這顯示出當在觸發內部元素事件的同時，外部元素的相同事件也會一起被觸發。

# 傳遞階段

DOM 事件傳遞分成三個階段 :

- 捕獲階段(CAPTURING_PHASE) : 事件從 `window` 向下傳遞到目標元素的過程。
- 目標階段(AT_TARGET) : 事件傳遞到目標本身。
- 冒泡階段(BUBBLING_PHASE) : 事件由目標向上傳遞回 `window`。

![DOM 事件傳遞示意圖](w3c-dom-event-flow.webp)  
(圖片來源 : [W3C - event flow][1] )

在 `addEventListener()` 回調函式中，可以通過事件物件 `e` 的 `eventPhase` 屬性，確認當前事件在傳遞中的哪一個階段被觸發。

而 `addEventListener()` 的第三個參數為 `useCapture`，意思是 listener 是否在捕獲階段被觸發。若為 `true`，即決定回調函式觸發於捕獲階段，為 `false` 則觸發於冒泡階段。

用與上例相同的 HTML 結構，看看以下範例 :

```js
const outer = document.querySelector(".outer")
const inner = document.querySelector(".inner")

outer.addEventListener(
  "click",
  e => {
    console.log("outer CAPTURING:", e.eventPhase)
  },
  true
)
outer.addEventListener(
  "click",
  e => {
    console.log("outer BUBBLING:", e.eventPhase)
  },
  false
)
inner.addEventListener(
  "click",
  e => {
    console.log("inner CAPTURING:", e.eventPhase)
  },
  true
)
inner.addEventListener(
  "click",
  e => {
    console.log("inner BUBBLING:", e.eventPhase)
  },
  false
)
```

當點擊內部元素 `inner` 後，得到的結果為 : ( `1` : 捕獲階段，`2` : 目標階段，`3` : 冒泡階段。)

```
"outer CAPTURING:" 1
"inner CAPTURING:" 2
"inner BUBBLING:" 2
"outer BUBBLING:" 3
```

可以看到首先事件捕獲到外部元素 `outer`，接著來到目標元素 `inner`，最後再冒泡傳回 `outer`。

而在過程中內部元素的 `eventPhase` 都是 `2`，也就是說當事件已經傳到目標身上進入目標階段(AT_TARGET)時，就沒有捕獲與冒泡的分別。

{% iframe https://codepen.io/fan94105/embed/preview/GRwwNBM?default-tab=html%2Cresult&theme-id=light % }

最後總結事件傳遞的兩個原則 :

1. 先捕獲，再冒泡。
2. 當事件傳遞到目標本身，就沒有捕獲與冒泡的分別。

## 取消事件傳遞

為了讓使事件停止傳遞，可以使用 `stopPropagation()` 方法。

使用第一個案例說明，在 `inner` 元素的事件回調函式中加入 `e.stopPropagation()` :

```js
const outer = document.querySelector(".outer")
const inner = document.querySelector(".inner")

outer.addEventListener("click", e => {
  console.log("outer")
})
inner.addEventListener("click", e => {
  e.stopPropagation()
  console.log("inner")
})
```

得到的結果為 :

```
"inner"
```

與第一個案例的結果不同，點擊內部元素 `inner` 只觸發自身綁定的事件回調函式，而沒有觸發外部元素的相應事件，成功阻止事件傳遞。

# 事件代理(Event Delegation)

利用事件傳遞的機制，當父元素下有多個子元素都綁定相同的事件時，可以將事件統一綁定到父元素身上，透過父元素處理子元素的事件。

範例 :

```html
<div class="parent" data-scope="parent">
  <div class="child--1" data-scope="child--1"></div>
  <div class="child--2" data-scope="child--2"></div>
</div>
```

```js
const parent = document.querySelector(".parent")

parent.addEventListener("click", e => {
  console.log(e.target.dataset.scope)
})
```

上面程式碼中只有將事件綁定在父元素 `parent` 身上，但是當點擊 `child--1` 或 `child--2` 元素時，分別都可以得到相對應的結果。

{% iframe https://codepen.io/fan94105/embed/preview/QWJJmdR?default-tab=js%2Cresult&theme-id=light %}

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [DOM 的事件傳遞機制：捕獲與冒泡](https://blog.techbridge.cc/2017/07/15/javascript-event-propagation/)
- [W3C - event flow][1]

[1]: https://www.w3.org/TR/DOM-Level-3-Events/#event-flow
