---
title: [MERN02]Backend - Middleware
tags:
  - Node.js
  - Express.js
categories:
  - MERN
  - Backend
---

Middleware : 中介軟體，在發出 HTTP 請求後，至應用程式收到回應前，處理特定用途的程式。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# Middleware

安裝 date-fns 與 uuid :

```
npm i date-fns uuid
```

## logger.js

創建 logger.js，紀錄請求。

```js
// middleware/logger.js
const { format } = require("date-fns")
const { v4: uuid } = require("uuid")
const fs = require("fs")
const fsPromises = require("fs").promises
const path = require("path")

const logEvent = async (message, logFileName) => {
  const dateTime = format(new Date(), "yyyyMMdd\tHH:mm:ss")
  const logItem = `${dateTime}\t${uuid()}\t${message}\n`

  try {
    // 若 logs 檔案不存在，則創建 logs
    if (!fs.existsSync(path.join(__dirname, "..", "logs"))) {
      await fsPromises.mkdir(path.join(__dirname, "..", "logs"))
    }
    await fsPromises.appendFile(
      path.join(__dirname, "..", "logs", logFileName),
      logItem
    )
  } catch (err) {
    console.log(err)
  }
}

const logger = (req, res, next) => {
  // 生成請求紀錄
  logEvent(`${req.method}\t${req.url}\t${req.headers.origin}`, "reqLog.log")
  next()
}

module.exports = {
  logEvent,
  logger,
}
```

## errorHandler.js

創建 errorHandler.js，處理錯誤並記錄。

```js
// middleware/errorHandler.js
const { logEvent } = require("./logger")

const errorHandler = (err, req, res, next) => {
  // 生成錯誤紀錄
  logEvent(
    `${err.name}: ${err.message}\t${req.method}\t${req.url}\t${req.headers.origin}`,
    "errLog.log"
  )

  const status = res.statusCode ? res.statusCode : 500 // server error

  res.status(status)

  res.json({ message: err.message })
}

module.exports = errorHandler
```

## cookie-parser

安裝 cookie-parser，用於解析 cookie :

```
npm i cookie-parser
```

## CORS

安裝 cors，用於解決跨域問題 :

```
npm i cors
```

創建 allowedOrigins.js，設定允許的 url。

```js
// config/allowedOrigins.js
const allowedOrigins = [
  // 部署前需更改
  "http://localhost:5173",
]

module.exports = allowedOrigins
```

創建 corsOptions.js，設定 cors 的 options。

```js
// config/corsOptions.js
const allowedOrigins = require("./allowedOrigins")

const corsOptions = {
  origin: (origin, callback) => {
    if (allowedOrigins.indexOf(origin) !== -1 || !origin) {
      callback(null, true)
    } else {
      callback(new Error("Not allowed by CORS"))
    }
  },
  credentials: true,
  optionsSuccessStatus: 200,
}

module.exports = corsOptions
```

## 啟用 Middleware

```js
// server.js
const express = require("express")
const app = express()
const path = require("path")
const cookieParser = require("cookie-parser")
const cors = require("cors")

const { logger } = require("./middleware/logger")
const errorHandler = require("./middleware/errorHandler")
const corsOptions = require("./config/corsOptions")

const PORT = process.env.PORT || 3500

// 啟用 logger middleware
app.use(logger)

// 啟用 cors
app.use(cors(corsOptions))

// 解析 json
app.use(express.json())

// 啟用 cookie-parser
app.use(cookieParser())

// 提供靜態頁面
app.use("/", express.static(path.join(__dirname, "/public")))

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

// 啟用 errorHandler
app.use(errorHandler)

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```
