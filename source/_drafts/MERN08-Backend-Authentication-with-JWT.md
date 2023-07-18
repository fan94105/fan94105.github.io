---
title: [MERN08]Backend - Authentication with JWT
tags:
  - Node.js
  - Express.js
  - JWT
categories:
  - MERN
  - Backend
---

建立 auth 路由，登入、刷新與登出的 API，並使用 JWT 設定 access token 與 refresh token，當用戶登入時，取得 access token 以確認身分，當 access token 失效，使用 cookie 中的 refresh token 以取得新的 access token，保持登入狀態。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# Rate Limit

安裝 express-rate-limit，限制請求登入的次數 :

```
npm i express-rate-limit
```

## loginLimiter.js

```js
// middleware/loginLimiter.js
const rateLimit = require("express-rate-limit")

const { logEvent } = require("./logger")

const loginLimiter = rateLimit({
  windowMs: 60 * 1000, // 請求時間範圍
  max: 5, // 指定時間範圍內的請求數量
  message: {
    message:
      "Too many login attempts from this IP, please try again after a 60 second pause",
  },
  handler: (req, res, next, options) => {
    logEvent(
      `Too Many Request: ${options.message.message}\t${req.method}\t${req.url}\t${req.headers.origin}`,
      "errLog.log"
    )
    res.status(options.statusCode).send(options.message)
  },
  standardHeaders: true, // 在 headers 的 RateLimit-* 中包含 rate limit 資訊
  legacyHeaders: false, // 禁用 X-RateLimit-* headers
})

module.exports = loginLimiter
```

# JWT

安裝 jsonwebtoken :

```
npm i jsonwebtoken
```

藉由 Node 內建的 crypto 模組，生成 `ACCESS_TOKEN_SECRET` 與 `REFRESH_TOKEN_SECRET` :

```
node
require('crypto').randomBytes(64).toString('hex')
```

## .env

```
ACCESS_TOKEN_SECRET=951fadcde6a0069229b51c15638ab7404a97ce65f3b4d2453dcee45aeffa329337937e4fac62759ab0671f9e321960ac72ae891528b92a3badb366cea4742dd5
REFRESH_TOKEN_SECRET=4a950dcda36c1f2b52d5bdc481f752695f0d740ca70112b9f741b454ebb1f00bd56a158650ab85744146c72b2181c7bf41f5485bc10b8e90f493339618504abf
```

## authController.js

```js
// controllers/authController.js
const bcrypt = require("bcrypt")
const jwt = require("jsonwebtoken")

const User = require("../models/User")

// @desc Login
// @route POST /auth
// @access Public
const login = async (req, res) => {
  const { username, password } = req.body

  if (!username || !password) {
    return res.status(400).json({ message: "All fields are required" })
  }

  const foundUser = await User.findOne({ username }).exec()

  if (!foundUser || !foundUser.active) {
    return res.status(401).json({ message: "Unauthorized" })
  }

  const match = await bcrypt.compare(password, foundUser.password)

  if (!match) {
    return res.status(401).json({ message: "Unauthorized" })
  }

  const accessToken = jwt.sign(
    {
      UserInfo: {
        username: foundUser.username,
        roles: foundUser.roles,
      },
    },
    process.env.ACCESS_TOKEN_SECRET,
    {
      expiresIn: "15m",
    }
  )
  const refreshToken = jwt.sign(
    {
      username: foundUser.username,
    },
    process.env.REFRESH_TOKEN_SECRET,
    {
      expiresIn: "7d",
    }
  )

  // create secure cookie with refresh token
  res.cookie("jwt", refreshToken, {
    httpOnly: true, // 是否只能在 HTTP 請求訪問 cookie，不能透過 JS 訪問
    secure: true, // 是否只在 HTTPS 中傳送 cookie
    sameSite: "None", // 是否在跨站請求中發送 cookie
    maxAge: 7 * 24 * 60 * 60 * 1000, // cookie 的有效時間
  })

  res.json({ accessToken })
}

// @desc Refresh
// @route GET /auth/refresh
// @access Public
const refresh = (req, res) => {
  const cookies = req.cookies

  if (!cookies?.jwt) {
    return res.status(401).json({ message: "Unauthorized" })
  }

  const refreshToken = cookies.jwt

  jwt.verify(
    refreshToken,
    process.env.REFRESH_TOKEN_SECRET,
    async (err, decoded) => {
      if (err) {
        return res.status(403).json({ message: "Forbidden" })
      }

      const foundUser = await User.findOne({
        username: decoded.username,
      }).exec()

      if (!foundUser) {
        return res.status(401).json({ message: "Unauthorized" })
      }

      const accessToken = jwt.sign(
        {
          UserInfo: {
            username: foundUser.username,
            roles: foundUser.roles,
          },
        },
        process.env.ACCESS_TOKEN_SECRET,
        {
          expiresIn: "15m",
        }
      )

      res.json({ accessToken })
    }
  )
}

// @desc Logout
// @route POST /auth/logout
// @access Public
const logout = (req, res) => {
  const cookies = req.cookies

  if (!cookies?.jwt) {
    return res.sendStatus(204) // No content
  }

  res.clearCookie("jwt", {
    httpOnly: true, // 是否只能在 HTTP 請求訪問 cookie，不能透過 JS 訪問
    secure: true, // 是否只在 HTTPS 中傳送 cookie
    sameSite: "None", // 是否在跨站請求中發送 cookie
    maxAge: 7 * 24 * 60 * 60 * 1000, // cookie 的有效時間
  })

  res.json({ message: "Cookie cleared" })
}

module.exports = {
  login,
  refresh,
  logout,
}
```

## authRoutes.js

```js
// routes/authRoutes.js
const express = require("express")
const router = express.Router()

const authController = require("../controllers/authController")
const loginLimiter = require("../middleware/loginLimiter")

router.route("/").post(loginLimiter, authController.login)

router.route("/refresh").get(authController.refresh)

router.route("/logout").post(authController.logout)

module.exports = router
```

## server.js

啟用 auth 路由。

```js
// server.js
app.use("/auth", require("./routes/authRoutes"))
```

# Middleware

## verifyJWT.js

驗證 JWT 的 middleware。

```js
// middleware/verifyJWT.js
const jwt = require("jsonwebtoken")

const verifyJWT = (req, res, next) => {
  const authHeader = req.headers.authorization || req.headers.Authorization

  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ message: "Unauthorized" })
  }

  const token = authHeader.split(" ")[1]

  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, decoded) => {
    if (err) {
      return res.status(403).json({ message: "Forbidden" })
    }

    req.user = decoded.UserInfo.username
    req.roles = decoded.UserInfo.roles

    next()
  })
}

module.exports = verifyJWT
```

## userRoutes.js & noteRoutes.js

在 userRoutes.js 與 noteRoutes.js 中啟用 middleware，確保只有登入後的使用者才能訪問數據。

```js
const verifyJWT = require("../middleware/verifyJWT")

router.use(verifyJWT)
```
