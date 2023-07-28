---
title: JavaScript - DOM 事件傳播(冒泡 & 捕獲)
tags:
  - JavaScript
categories:
  - JavaScript
---

當元素綁定的事件被觸發時，會經過三個階段，捕獲階段(Capturing phase)、目標階段(Target phase)與冒泡階段(Bubbling phase)。

<!-- more -->

# 事件傳播

HTML 中設定一個外部元素 `outer` 與內部元素 `inner`。

```html
<div class="outer">
  <div class="inner"></div>
</div>
```

將兩個元素分別綁定事件，當元素被點擊時，在 console 中印出動應的內容。

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

可以發現，當內部元素 `inner` 被點擊時，在 console 中不只印出 "inner"，也印出了 "outer"，這顯示出當在觸發內部元素的事件的同時，外部元素的相同事件也會一起被觸發。

{% iframe https://codepen.io/fan94105/embed/preview/GRwwNBM?default-tab=html%2Cresult&theme-id=light % }

# 參考資料
