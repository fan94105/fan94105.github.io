---
title: CSS - 格線佈局(Grid)
tags:
  - CSS
  - Grid
categories:
  - CSS
---

不同於 Flex 一維「由上到下」的排版方式，Grid 為二維的 CSS 格線系統，可以定義容器區塊中的 row 及 column，並將元素依照格線規則設置任意的排列布局。

<!-- more -->

# 使用方式

定義結構 :

```html
<div class="container">
  <div class="A">A</div>
  <div class="B">B</div>
  <div class="C">C</div>
  <div class="D">D</div>
</div>
```

## 基本用法

設置固定的 `grid-template-rows` 與 `grid-template-columns`，在限制的區域範圍內布局。

```css
.container {
  width: 90%;
  height: 800px;
  display: grid;
  grid-template-rows: 200px 200px 200px auto;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  grid-template-areas:
    "B1 B3 B4 B4"
    "B1 B3 B4 B4"
    "B1 B3 B4 B4"
    "B2 B2 B2 B2";
  grid-gap: 10px;
}

.A {
  grid-area: B1;
  background-color: #d4d2d5;
}
.B {
  grid-area: B2;
  background-color: #bfafa6;
}
.C {
  grid-area: B3;
  background-color: #aa968a;
}
.D {
  grid-area: B4;
  background-color: #6e6a6f;
}
```

{% iframe https://codepen.io/fan94105/embed/bGQdGgg?default-tab=html%2Cresult&theme-id=light %}

## 進階用法

設置 `grid-auto-rows` 與 `grid-auto-columns`，當布局超出範圍時則自動生成格線。

```css
.container {
  width: 90%;
  height: 800px;
  outline: 2px solid #f00;
  display: grid;
  grid-gap: 10px;
  grid-auto-rows: 1fr;
  grid-auto-columns: 1fr;
}

.A {
  grid-row: 1/2;
  grid-column: 1/6;
  background-color: #d4d2d5;
}
.B {
  grid-row: 2/6;
  grid-column: 1/3;
  background-color: #bfafa6;
}
.C {
  grid-row: 2/4;
  grid-column: 3/6;
  background-color: #aa968a;
}
.D {
  grid-row: 4/6;
  grid-column: 3/6;
  background-color: #6e6a6f;
}
```

{% iframe https://codepen.io/fan94105/embed/gOQpbVd?default-tab=html%2Cresult&theme-id=light %}

# CSS Grid 屬性

> 資料來源 : [A Complete Guide to CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)

## Grid Container

### display

設定 grid 容器。

```css
.container {
  display: grid | inline-grid;
}
```

### grid-template-rows & grid-template-columns

設定 template 中 rows 與 columns 數量及大小。

```css
.container {
  grid-template-rows: ... ...;
  grid-template-columns: ... ...;
}
```

### grid-template-area

搭配 Grid Item 的 `grid-area` 名稱，定義 Grid Item 在 template 上的位置。

```css
.container {
  grid-template-area:
    "header header header"
    "main main main"
    "footer footer footer";
}

.item-a {
  grid-area: header;
}
.item-b {
  grid-area: main;
}
.item-c {
  grid-area: footer;
}
```

### grid-auto-rows & grid-auto-columns

不明確指定 rows 與 columns 的數量及大小，而使用自動生成的 grid 網格。

```css
.container {
  grid-auto-columns: ...;
  grid-auto-rows: ...;
}
```

### grid-auto-flow

設定 Grid Items 的排列方向。

```css
.container {
  grid-auto-flow: row | column | row dense | column dense;
}
```

- `dense` : 使較小的項目先填入網格中。可能改變項目顯示順序，不利於 accessibility。

### gap

設定 Grid Items 之間的間隔。

```css
.container {
  gap: <grid-row-gap> <grid-column-gap>;
}
```

### justify-items

設定 Grid Items 在網格中的主軸對齊。

```css
.container {
  justify-items: start | end | center | stretch;
}
```

- `stretch` : 預設值，填滿網格。

### align-items

設定 Grid Items 在網格中的次軸對齊。

```css
.container {
  align-items: start | end | center | stretch;
}
```

### place-items

同時設定 `align-items` 與 `justify-items`。

```css
.container {
  place-items: <align-items> <justify-items>;
}
```

### justify-content

設定整體網格在 Grid Container 中的主軸對齊。

```css
.container {
  justify-content: start | end | center | stretch | space-around | space-between
    | space-evenly;
}
```

### align-content

設定整體網格在 Grid Container 中的次軸對齊。

```css
.container {
  align-content: start | end | center | stretch | space-around | space-between |
    space-evenly;
}
```

### place-content

同時設定 `align-content` 與 `justify-content`。

```css
.container {
  place-content: <align-content> <justify-content>;
}
```

## Grid Items

### grid-row & grid-column

設定 row 與 column 的起始即結束位置。

```css
.item {
  grid-row: <start-line> / <end-line> | <start-line> / span <value>;
  grid-column: <start-line> / <end-line> | <start-line> / span <value>;
}
```

- `span` : 表示跨越幾個面。
- 可以使用負數值，`-1` 代表最後一條線。
- 當沒有設定結束位置時，預設為跨越一個面。

### grid-area

設定 Grid Item 的名稱，也可以作為 `grid-row-start` + `grid-column-start` + `grid-row-end` + `grid-column-end` 的簡寫。

```css
.item {
  grid-area: <name> | <row-start> / <column-start> / <row-end> / <column-end>;
}
```

### justify-self

設定 Grid Item 在網格內的主軸對齊。

```css
.item {
  justify-self: start | end | center | stretch;
}
```

### align-self

設定 Grid Item 在網格內的次軸對齊。

```css
.item {
  align-self: start | end | center | stretch;
}
```

### place-self

同時設定 `align-self` 與 `justify-self`。

```css
.item {
  place-self: <align-self> <justify-self>;
}
```

# Special Units & Function

## fr units

代表剩餘空間的一部分。

```css
grid-template-columns: 1fr 3fr;
```

上例表示設定兩欄，將空間分為 4 等份，第一欄分配一等份，因此寬度為總網格寬的 1/4，第二欄分配三等份，寬度為總網格寬的 3/4。

## min-content & max-content

網格大小依內容空間改變。

```css
.container {
  grid-template-rows: max-content min-content;
}
```

- `min-content` : 依內容中最長單字的尺寸為整體尺寸。
- `max-content` : 依內容的最大尺寸為整體尺寸。

## fit-content()

設定尺寸為 `min-content` 到給定參數之間的大小。

```css
grid-template-rows: fit-content(150px);
```

上例 `fit-content(150px)`，表示最終渲染尺寸為 `min-content ~ 150px` 之間。但若 `fit-content()` 中的值比 `max-content` 大，則最終渲染尺寸為 `min-content ~ max-content` 之間。

## minmax()

設定尺寸的最小值與最大值。

```css
grid-template-rows: minmax(min, max);
```

## repeat()

設定重複的尺寸值。

```css
grid-template-rows: 1fr 1fr 1fr 1fr 1fr;

/* 等同於 */
grid-template-rows: repeat(5, 1fr);

grid-template-rows: repeat(auto-fit, minmax(250px, 1fr));
```

- `auto-fill` : 自動填滿軌道。
- `auto-fit` : 延展每一個 Grid Item，填滿軌道。

`auto-fill` 與 `auto-fit` 比較 :

{% iframe https://codepen.io/fan94105/embed/abQOyLO?default-tab=html%2Cresult&theme-id=light %}

# Grid Garden

認識 CSS Grid 的 [小遊戲 🥕](https://cssgridgarden.com/#zh-tw)。
