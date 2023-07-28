---
title: CSS-單位-Units
tags:
  - CSS
categories:
  - CSS
---

<!-- more -->

# 絕對單位(Absolute units)

指在任何情況下都相同的單位，如 `px` (pixel)。

# 相對單位(Relative units)

指會受上下文影響而改變的單位。

## em & rem

`em` 與 `rem` 都是基於字體大小計算的單位。

`em` 與**父元素**的 `font-size` 成倍數關係。

```css
.parent {
  font-size: 16px;
}

.child {
  width: 1em; /* width: 16px */
  height: 0.5em; /* height: 8px */
  font-size: 2em; /* font-size: 32px */
}
```

`rem` 則是與**根元素**的 `font-size` 成倍數關係，根元素為 `:root` 或 `html`，其計算方式與 `em` 相同。

## 百分比(%)

`%` 會以父元素的值計算其百分比。

```css
.parent {
  width: 200px;
  font-size: 16px;
}
.child {
  width: 50%; /* width: 100px */
  font-size: 50%; /* font-size: 8px */
}
```

## vw & vh

`vw` (viewport width) 與 `vh` (viewport height) 為整個視窗寬度與高度的百分比值。

# 參考資料

- [CSS values and units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units)
- [CSS Units](https://www.theodinproject.com/lessons/node-path-intermediate-html-and-css-css-units#relative-units)
