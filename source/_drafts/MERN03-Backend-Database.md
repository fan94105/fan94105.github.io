---
title: [MERN03]Backend - Database
tags:
  - Node.js
  - Express.js
  - MongoDB
categories:
  - MERN
  - Backend
---

建立 MongoDB Atlas Database ，使用 MongoDB，設定 Models 管理、儲存資料。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# .env

安裝 dotenv，用於管理環境變數 :

```
npm i dotenv
```

在 server.js 中引入 :

```js
// server.js
// As early as possible in your application.
require("dotenv").config()
```

創建 .env 檔，設定環境變數 :

```
NODE_ENV = development
DATABASE_URI = mongodb+srv://<username>:<password>@cluster0.zntifa9.mongodb.net/<databasename>?retryWrites=true&w=majority
```

# MongoDB

## 建立資料庫

MongoDB Atlas 建立 Database，取得 connection string :  
`mongodb+srv://<username>:<password>@cluster0.zntifa9.mongodb.net/<databasename>?retryWrites=true&w=majority`

## 建立 Models

安裝 mongoose :

```
npm i mongoose
```

創建 User Model。

```js
// models/User.js
const mongoose = require("mongoose")

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
  },
  password: {
    type: String,
    required: true,
  },
  roles: {
    type: [String],
    default: ["Employee"],
  },
  active: {
    type: Boolean,
    default: true,
  },
})

module.exports = mongoose.model("User", userSchema)
```

安裝 @typegoose/auto-increment，用於自增項 :

```
npm i @typegoose/auto-increment
```

創建 Note Model。

```js
// models/Note.js
const mongoose = require("mongoose")
const AutoIncrementID = require("@typegoose/auto-increment").AutoIncrementID

const noteSchema = new mongoose.Schema(
  {
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
    title: {
      type: String,
      required: true,
    },
    text: {
      type: String,
      required: true,
    },
    completed: {
      type: Boolean,
      default: false,
    },
    ticket: Number,
  },
  {
    timestamps: true,
  }
)

noteSchema.plugin(AutoIncrementID, { field: "ticket", startAt: 500 })

module.exports = mongoose.model("Note", noteSchema)
```

## 連接資料庫

```js
// config/dbConnection.js
const mongoose = require("mongoose")

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.DATABASE_URI)
  } catch (err) {
    console.log(err)
  }
}

module.exports = connectDB
```

```js
// server.js
require("dotenv").config()
const express = require("express")
const app = express()
const path = require("path")
const cookieParser = require("cookie-parser")
const cors = require("cors")
const mongoose = require("mongoose")

const { logger, logEvent } = require("./middleware/logger")
const errorHandler = require("./middleware/errorHandler")
const corsOptions = require("./config/corsOptions")
const connectDB = require("./config/dbConnection")

const PORT = process.env.PORT || 3500

// 連接資料庫
connectDB()

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

app.use(errorHandler)

// MongoDB 連接成功
mongoose.connection.once("open", () => {
  console.log("Connected to MongoDB")
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`)
  })
})

// MongoDB 連接失敗
mongoose.connection.once("error", err => {
  console.log(err)
  logEvent(
    `${err.no}: ${err.code}\t${err.syscall}\t${err.hostname}`,
    "mongoErrLog.log"
  )
})
```
