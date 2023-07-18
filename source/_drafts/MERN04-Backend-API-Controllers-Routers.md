---
title: [MERN04]Backend - API Controllers & Routers
tags:
  - Node.js
  - Express.js
categories:
  - MERN
  - Backend
---

設置 API Controllers 及 Routers 處理前端請求，並回應響應數據。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# Controllers

安裝 bcrypt，用於 hash 密碼 :

```
npm i bcrypt
```

## usersController.js

```js
// controllers/usersController.js
const bcrypt = require("bcrypt")

const User = require("../models/User")
const Note = require("../models/Note")

// @desc Get all users
// @route GET /users
// @access Private
const getAllUsers = async (req, res) => {
  const users = await User.find({}).select("-password").lean().exec()
  if (!users) {
    return res.status(400).json({ message: "No users found" })
  }
  res.json(users)
}

// @desc Create new user
// @route POST /users
// @access Private
const createNewUser = async (req, res) => {
  const { username, password, roles } = req.body

  if (!username || !password) {
    return res.status(400).json({ message: "All fields are required" })
  }

  // 檢查重複 username
  const duplicate = await User.findOne({ username })
    .collation({ locale: "en", strength: 2 })
    .lean()
    .exec()
  if (duplicate) {
    return res.status(409).json({ message: "Duplicate username" })
  }

  // hash password
  const hashedPwd = await bcrypt.hash(password, 10) // salt rounds

  // 判斷是否提供 roles
  const userObject =
    !Array.isArray(roles) || !roles.length
      ? { username, password: hashedPwd }
      : { username, password: hashedPwd, roles }

  const user = await User.create(userObject)
  if (user) {
    res.status(201).json({ message: `New user ${username} created` })
  } else {
    res.status(400).json({ message: "Invalid user data received" })
  }
}

// @desc Update user
// @route PATCH /users
// @access Private
const updateUser = async (req, res) => {
  const { id, username, password, roles, active } = req.body

  if (
    !id ||
    !username ||
    !Array.isArray(roles) ||
    !roles.length ||
    typeof active !== "boolean"
  ) {
    return res
      .status(400)
      .json({ message: "All fields except password are required" })
  }

  const user = await User.findById(id).exec()
  if (!user) {
    return res.status(400).json({ message: "User not found" })
  }

  // 檢查重複 username
  const duplicate = await User.findOne({ username })
    .collation({ locale: "en", strength: 2 })
    .lean()
    .exec()
  if (duplicate && duplicate?._id.toString() !== id) {
    return res.status(409).json({ message: "Duplicate username" })
  }

  user.username = username
  user.roles = roles
  user.active = active

  if (password) {
    // hash password
    user.password = await bcrypt.hash(password, 10)
  }

  const updatedUser = await user.save()

  res.json({ message: `${updatedUser.username} updated` })
}

// @desc Delete user
// @route DELETE /users
// @access Private
const deleteUser = async (req, res) => {
  const { id } = req.body

  if (!id) {
    return res.status(400).json({ message: "User ID Required" })
  }

  const note = await Note.findOne({ user: id }).lean().exec()
  if (note) {
    return res.status(400).json({ message: "User has assigned notes" })
  }

  // 確認 user 存在
  const user = await User.findById(id).exec()
  if (!user) {
    return res.status(400).json({ message: "User not found" })
  }

  const result = await user.deleteOne()

  const reply = `Username ${result.username} with ID ${result._id} deleted`

  res.json(reply)
}

module.exports = {
  getAllUsers,
  createNewUser,
  updateUser,
  deleteUser,
}
```

## notesController.js

```js
// controllers/notesController.js
const Note = require("../models/Note")
const User = require("../models/User")

// @desc Get all notes
// @route GET /notes
// @access Private
const getAllNotes = async (req, res) => {
  const notes = await Note.find({}).lean().exec()
  if (!notes.length) {
    return res.status(400).json({ message: "No notes found" })
  }

  const notesWithUser = await Promise.all(
    notes.map(async note => {
      const user = await User.findById(note.user).lean().exec()
      return { ...note, username: user.username }
    })
  )

  res.json(notesWithUser)
}

// @desc Create new note
// @route POST /notes
// @access Private
const createNewNote = async (req, res) => {
  const { user, title, text } = req.body
  if (!user || !title || !text) {
    return res.status(400).json({ message: "All fields are required" })
  }

  // 檢查重複 title
  const duplicate = await Note.findOne({ title })
    .collation({ locale: "en", strength: 2 })
    .lean()
    .exec()
  if (duplicate) {
    return res.status(409).json({ message: "Duplicate note title" })
  }

  const note = await Note.create({ user, title, text })
  if (note) {
    return res.status(201).json({ message: "New note created" })
  } else {
    return res.status(400).json({ message: "Invalid note data received" })
  }
}

// @desc Update note
// @route PATCH /notes
// @access Private
const updateNote = async (req, res) => {
  const { id, user, title, text, completed } = req.body

  if (!id || !user || !title || !text || typeof completed !== "boolean") {
    return res.status(400).json({ message: "All fields are required" })
  }

  const note = await Note.findById(id).exec()
  if (!note) {
    return res.status(400).json({ message: "Note not found" })
  }

  // 檢查重複 title
  const duplicate = await Note.findOne({ title })
  if (duplicate && duplicate?._id.toString() !== id) {
    return res.status(409).json({ message: "Duplicate note title" })
  }

  note.user = user
  note.title = title
  note.text = text
  note.completed = completed

  const updatedNote = await note.save()

  res.json(`'${updatedNote.title}' updated`)
}
// @desc Delete note
// @route DELETE /notes
// @access Private
const deleteNote = async (req, res) => {
  const { id } = req.body

  if (!id) {
    return res.status(400).json({ message: "Note ID required" })
  }

  const note = await Note.findById(id).exec()
  if (!note) {
    return res.status(400).json({ message: "Note not found" })
  }

  const result = await note.deleteOne()

  const reply = `Note '${result.title}' with ID ${result._id} deleted`

  res.json(reply)
}

module.exports = {
  getAllNotes,
  createNewNote,
  updateNote,
  deleteNote,
}
```

# Routers

## userRoutes.js

```js
// routes/userRoutes.js
const express = require("express")
const router = express.Router()

const usersController = require("../controllers/usersController")

router
  .route("/")
  .get(usersController.getAllUsers)
  .post(usersController.createNewUser)
  .patch(usersController.updateUser)
  .delete(usersController.deleteUser)

module.exports = router
```

## noteRouter.js

```js
// routes/noteRouters.js
const express = require("express")
const router = express.Router()

const notesController = require("../controllers/notesController")

router
  .route("/")
  .get(notesController.getAllNotes)
  .post(notesController.createNewNote)
  .patch(notesController.updateNote)
  .delete(notesController.deleteNote)

module.exports = router
```

# server.js

安裝 express-async-errors，用於處理異步錯誤，不需編寫 `try/catch`。

```
npm i express-async-errors
```

啟用 Routers。

```js
// server.js
require("dotenv").config()
const express = require("express")
const app = express()
const path = require("path")
const cookieParser = require("cookie-parser")
const cors = require("cors")
const mongoose = require("mongoose")
require("express-async-errors")

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

// Routers
app.use("/users", require("./routes/userRoutes"))
app.use("/notes", require("./routes/noteRoutes"))

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
