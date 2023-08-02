---
title: JavaScript - script 標籤屬性(async & defer)
tags:
  - JavaScript
  - HTML
categories:
  - JavaScript
date: 2023-08-01 15:47:47
---


當 JavaScript 的檔案越多，網頁載入的時間越久，為了縮減使用者等待的時間，可以使用`<script>` 標籤上的 `async` 或 `defer` 屬性，讓 script 檔案異步加載，加速網頁載入時間，提升使用者體驗。

<!-- more -->

# Regular

沒有加上其他屬性的 `<script>` 標籤。

```html
<script src="demo.js"></script>
```

根據在 HTML 中所處位置的不同，對程式會有不同的影響。

`<script>` 處於 `<head>` 中 :

用戶載入頁面，瀏覽器開始解析 HTML，構建 DOM tree，當解析到 `<script>` 後，停止解析 HTML，而開始載入 `<script>` 中的檔案， 並執行 JavaScript 程式，最後才繼續解析剩下的 HTML。

這代表 JavaScript 會在 DOM 建構完成之前執行，需要操作 DOM 的程式就會因此無法執行。

`<script>` 處於 `<body>` 的最後一行 :

瀏覽器解析 HTML 是一行一行向下執行，所以將 `<script>` 放到 `<body>` 的最後一行就可以保證瀏覽器已將 HTML 解析完畢，DOM tree 建構完成，如此執行 JavaScript 程式就不會出現問題。

# async

```html
<script async src="demo.js"></script>
```

`async` 屬性表示 `script` 中的檔案為異步加載，在瀏覽器解析 HTML 時，`<script>` 中的檔案正異步加載，當載入完成後停止解析 HTML，立即執行 JavaScrip 程式，結束後才繼續解析 HTML。

通常 `DOMContentLoaded` 事件會在所有 `<script>` 檔案載入後才觸發，但是當使用 `async` 屬性時，只要 HTML 解析完畢，該事件就會直接觸發。

使用時機 :

不操作 DOM，且不在意 `<script>` 檔案執行順序時可以使用。適用於獨立的第三方 script，如 Google Analytics。

# defer

```html
<script defer src="demo.js"></script>
```

`defer` 屬性表示將 JavaScript 程式推遲到 HTML 解析完畢後才執行。在瀏覽器解析 HTML 時，`<script>` 中的檔案正異步加載，當 HTML 解析完成後，才執行 JavaScrip 程式。

# 三者比較

不同屬性的執行流程圖 :

![不同屬性的執行流程圖](efficient-script-loading.webp)

<center>( 圖片來源 : The Complete JavaScript Course 2023: From Zero to Expert! )</center>

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
