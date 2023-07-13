---
title: JavaScript - 可選鏈(Optional Chaining)
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-13 23:23:20
---

在取得 API 響應並存取數據時，時常會遇到 `TypeError: Cannot read property 'xxx' of null` 或是 `TypeError: Cannot read property 'xxx' of undefined` 的錯誤，而造成錯誤的原因就是試圖從 nullish value 上讀取屬性或調用方法，為了解決這個問題，可以在取值時使用可選鏈，以免程式拋出錯誤。

<!-- more -->

# 可選鏈(Optional Chaining)

用來避免在從 nullish value(`null` 或 `undefined`)上讀取屬性或調用方法時拋出錯誤，語法為 `?.`。

使用範例 :

```js
const pets = {
  dog: {
    name: "ABen",
  },
}

// 未使用可選鏈
console.log(pets.cat.name) // TypeError: Cannot read properties of undefined (reading 'name')

// 使用可選鏈
console.log(pets.cat?.name) // undefined
```

當存取物件中不存在的屬性時，若沒有使用可選鏈就會報錯，而使用可選鏈則會回傳 `undefined`。

在可選鏈出現之前，為了避免錯誤，通常會這麼做 :

```js
const pets = {
  dog: {
    name: "ABen",
  },
}

if (pets.cat && pets.cat.name) {
  console.log(pets.cat.name)
}
```

這樣的寫法雖然可以避免錯誤，但是若需要存取巢狀物件的深層屬性時，那就必須要更多的判斷才能確保程式不報錯，所以為了簡潔與可讀性，使用可選鏈 `?.` 是更好的選擇。

調用方法 :

```js
const aben = {
  name: "ABen",
}

console.log(aben.fly?.() ?? "Method does not exist") // Method does not exist
```

上面程式碼中呼叫 `aben` 物件中不存在的 `fly` 方法，但因為使用可選鏈，所以回傳 `undefined` 而不會報錯，並且搭配空值合併運算符(Nullish Coalescing Operator) `??`，以回傳除了 `undefined` 與 `null` 的其他值。

存取陣列數據 :

```js
const pets = [
  {
    name: "ABen",
    age: 2,
  },
]

console.log(pets[0]?.name ?? "Pet array empty") // ABen
```

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [JavaScript 可選鏈運算符 (?.) 與 空值合併運算符 (??) 介紹](https://www.tpisoftware.com/tpu/articleDetails/2533)
