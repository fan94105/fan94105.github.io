---
title: [MERN01]Backend - API 的靜態頁面
tags:
  - Node.js
  - Express.js
categories:
  - MERN
  - Backend
---

當訪問 API url 時所展示的頁面，用於測試 API 是否可以正常訪問。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# API 的靜態頁面

安裝 Express :

```
npm i express
```

## server.js

在 server.js 中提供靜態頁面，並設置 root 路由。

```js
// server.js
const express = require("express")
const app = express()
const path = require("path")

const PORT = process.env.PORT || 3500

// 提供靜態檔案
app.use("/", express.static(path.join(__dirname, "/public")))
// 訪問 API 的根路由
app.use("/", require("./routes/root"))

// 接收未知 url，並回應 404 頁面
app.all("*", (req, res) => {
  res.status(404)
  if (req.accepts("html")) {
    res.sendFile(path.join(__dirname, "views", "404.html"))
  } else if (req.accepts("json")) {
    res.json({ message: "404 Not Found" })
  } else {
    res.type("txt").send("404 Not Found")
  }
})

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

## root.js

root 路由中響應 index.html 檔案。

```js
// routes/root.js
const express = require("express")
const router = express.Router()
const path = require("path")

// 響應 index.html 檔案
router.get("^/$|/index(.html)?", (req, res) => {
  res.sendFile(path.join(__dirname, "..", "views", "index.html"))
})

module.exports = router
```

## index.html

```js
<!-- views/index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>techNotes API</title>
    <!-- 因為 server.js 中有提供 '/public' 靜態檔案，所以可以直接找到 style.css -->
    <link rel="stylesheet" href="css/style.css" />
  </head>
  <body>
    <h1>techNotes</h1>
  </body>
</html>
```

## 404.html

訪問未知 url 展示 404.html。

```js
<!-- views/404.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>404 Not Found</title>
    <link rel="stylesheet" href="css/style.css" />
  </head>
  <body>
    <h1>Sorry!</h1>
    <p>The resource you have requested does not exist.</p>
  </body>
</html>
```
