---
title: JavaScript - Set 與 Map
tags:
  - JavaScript
categories:
  - JavaScript
date: 2023-07-15 17:27:11
---


Set 與 Map 都是 JavaScript 中特殊的資料結構，Set 與 Array 相似，但其中的每一個元素都是唯一值，不會有重複元素，主要用途是去除重複出現的元素與判斷元素存在與否；Map 則與 Object 相似，其中的 key 值映射對應 value，而 key 可以使用任意類型的值定義，且亦具備唯一性。

<!-- more -->

# Set

具有不重複元素的集合。存在的意義是判斷 Set 中是否包含某個元素，所以重點不在於從 Set 中將元素取出。

去除陣列中的重複元素 :

```js
const arr = [0, 1, 2, 2, 3, 3, 3]

const arrSet = new Set(arr)

console.log(arrSet) // Set(4) {0, 1, 2, 3}
```

Set 與陣列的轉換 :

```js
const arrSet = new Set([0, 1, 2, 2, 3, 3, 3])

let SetToArray = [...arrSet] // Array.from(arrSet)

let ArrayToSet = new Set(SetToArray)
```

## Set 的屬性與方法

### size

回傳 Set 中的元素數量。

```js
const arrSet = new Set([0, 1, 2, 2, 3, 3, 3])

console.log(arrSet.size) // 4
```

### has()

檢查 Set 中是否存在指定元素。

```js
const arrSet = new Set([0, 1, 2, 2, 3, 3, 3])

console.log(arrSet.has(1)) // true
console.log(arrSet.has(4)) // false
```

### add()

在 Set 中添加元素。

```js
const arrSet = new Set([0, 1, 2, 2, 3, 3, 3])

arrSet.add(4)

console.log(arrSet) // Set(5) {0, 1, 2, 3, 4}
```

### delete()

刪除 Set 中的元素。

```js
const arrSet = new Set([0, 1, 2, 2, 3, 3, 3])

arrSet.delete(3)

console.log(arrSet) // Set(3) {0, 1, 2}
```

### clear()

清空 Set 中的所有元素。

```js
const arrSet = new Set([0, 1, 2, 2, 3, 3, 3])

arrSet.clear()

console.log(arrSet) // Set(0) {}
```

# Map

Map 是與物件(Object)相似的鍵值對(key-value pair)數據結構，Map 中的鍵(key)會映射到對應的值(value)。兩者具有以下差異 :

- Map 中的 key 可以是任何類型的值，而 Object 的 key 只能是 `String` 或 `Symbol`。
- Map 中的 key 會依添加時間排序，Object 無順序性。
- Map 中的 key 是唯一值，若重複添加則舊的 value 會被覆蓋。
- 需要經常增添、刪減屬性的資料，使用 Map 的效能較 Object 佳。

建立 Map :

```js
// 建立空的 Map
const emptyMap = new Map()

// 建立時添加鍵值對
const newMap = new Map([
  ["name", "ABen"],
  [2, "age"],
])
```

Map 為可迭代(iterable)的資料結構，所以可以使用 for loop :

```js
const newMap = new Map([
  ["name", "ABen"],
  [2, "age"],
])

for (const [key, value] of newMap) {
  console.log(`${key}: ${value}`) // name: ABen  2: age
}
```

以物件類型的值作為 key 時 :

```js
const newMap = new Map()
const arr = [1, 2]
newMap.set(arr, "Test")

console.log(newMap.get([1, 2])) // undefined
console.log(newMap.get(arr)) // Test
```

要特別注意因為物件類型儲存的值為 Heap 中記憶體位址的參考(reference)，所以當上面程式中執行 `newMap.get([1, 2])` 取值時，指定的陣列 `[1, 2]` 與先前所設定的陣列 `arr` 為兩個不同的陣列，因此回傳 `undefined`。

物件轉換成 Map :

```js
const aben = {
  name: "ABen",
  age: 2,
}

console.log(new Map(Object.entries(aben))) // Map(2) { 'name' => 'ABen', 'age' => 2 }
```

Map 轉換成陣列 :

```js
const aben = new Map([
  ["name", "Aben"],
  ["age", 2],
])

console.log([...aben]) // [ [ 'name', 'Aben' ], [ 'age', 2 ] ]
// or
console.log([...aben.entries()]) // [ [ 'name', 'Aben' ], [ 'age', 2 ] ]

console.log([...aben.keys()]) // [ 'name', 'age' ]
console.log([...aben.values()]) // [ 'Aben', 2 ]
```

猜謎小範例 :

```js
const question = new Map([
  ["question", "What is the cutest animal in the world?"],
  [1, "ABen"],
  [2, "ACat"],
  [3, "ABird"],
  ["correct", 1],
  [true, "Correct!"],
  [false, "Wrong!"],
])

console.log(question.get("question"))
for (const [key, value] of question) {
  if (typeof key === "number") {
    console.log(`Answer ${key}: ${value}`)
  }
}
const answer = Number(prompt("Your answer"))
console.log(answer)

console.log(question.get(question.get("correct") === answer))
```

## Map 的屬性與方法

### size

回傳 Map 中的鍵值對數量。

```js
const newMap = new Map()

newMap.set("name", "ABen").set(2, "age").set(true, "ABen is cute <3")
console.log(newMap.size) // 3
```

### has()

檢查 Map 中是否有指定的 key。

```js
const newMap = new Map()
newMap.set("name", "ABen")

console.log(newMap.has("name")) // true
console.log(newMap.has(2)) // false
```

### set()

在 Map 中添加一個鍵值對，本身也會回傳更新後的 Map，所以可以進行鏈式調用。

```js
const newMap = new Map()

console.log(newMap.set("name", "ABen")) // Map(1) { 'name' => 'ABen' }

newMap.set(2, "age").set(true, "ABen is cute <3")
console.log(newMap) // Map(3) { 'name' => 'ABen', 2 => 'age', true => 'ABen is cute <3' }
```

### get()

取得 Map 中特定 key 值所映射的 value。

```js
const newMap = new Map()

newMap.set("name", "ABen") // Map(1) { 'name' => 'ABen' }

console.log(newMap.get("name")) // ABen
```

### delete()

刪除指定 key 的鍵值對。回傳是否刪除。

```js
const newMap = new Map()

newMap.set("name", "ABen") // Map(1) { 'name' => 'ABen' }

newMap.delete("name") // Map(0) {}
```

### clear()

清除 Map 中所有鍵值對。

```js
const newMap = new Map()

newMap.set("name", "ABen").set(2, "age").set(true, "ABen is cute <3")

newMap.clear()

console.log(newMap) // Map(0) {}
```

# 參考資料

- [The Complete JavaScript Course 2023: From Zero to Expert!](https://www.udemy.com/course/the-complete-javascript-course/)
- [[JS] JavaScript 集合（Set）](https://pjchender.dev/javascript/js-set/#%E9%99%A3%E5%88%97%E8%88%87%E9%9B%86%E5%90%88%E9%96%93%E8%BD%89%E6%8F%9B)
- [[JS] JavaScript Map](https://pjchender.dev/javascript/js-map/)
