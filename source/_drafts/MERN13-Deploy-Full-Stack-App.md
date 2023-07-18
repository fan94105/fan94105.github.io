---
title: [MERN13]Deploy Full Stack App
tags:
  - MongoDB
  - React
  - Deploy
categories:
  - [MERN, Frontend]
  - [MERN, Backend]
---

使用 [render](https://render.com/) 部署前端頁面及後端 API。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# Frontend - 前置作業

安裝 disable-react-devtools :

```
npm i @fvilers/disable-react-devtools
```

## main.jsx

```js
// main.jsx
import React from "react"
import ReactDOM from "react-dom/client"
import { BrowserRouter as Router, Routes, Route } from "react-router-dom"
import { disableReactDevTools } from "@fvilers/disable-react-devtools"

import { Provider } from "react-redux"
import { store } from "./app/store"

import App from "./App.jsx"
import "./index.css"

if (process.env.NODE_ENV === "production") {
  disableReactDevTools()
}

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <Provider store={store}>
      <Router>
        <Routes>
          <Route path="/*" element={<App />} />
        </Routes>
      </Router>
    </Provider>
  </React.StrictMode>
)
```

## store.js

將 `devTools` 屬性設為 `false`。

```js
// app/store.js
import { configureStore } from "@reduxjs/toolkit"
import { setupListeners } from "@reduxjs/toolkit/query"

import { apiSlice } from "./api/apiSlice"
import authReducer from "../features/auth/authSlice"

export const store = configureStore({
  reducer: {
    [apiSlice.reducerPath]: apiSlice.reducer,
    auth: authReducer,
  },
  middleware: getDefaultMiddleware =>
    getDefaultMiddleware().concat(apiSlice.middleware),
  devTools: false,
})

// 啟用 refetchOnFocus 與 refetchOnReconnect
setupListeners(store.dispatch)
```

## apiSlice.js

將 `baseUrl` 修改為後端伺服器的 url。

```js
// app/api/apiSlice.js
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react"

import { setCredentials } from "../../features/auth/authSlice"

const baseQuery = fetchBaseQuery({
  // 後端伺服器的 url !
  baseUrl: "https://technotes-api-msr9.onrender.com",
  // 在每個請求中都發送 token
  credentials: "include",
  // 設置請求 headers
  prepareHeaders: (headers, { getState }) => {
    const token = getState().auth.token

    if (token) {
      headers.set("Authorization", `Bearer ${token}`)
    }

    return headers
  },
})

const baseQueryWithReauth = async (args, api, extraOptions) => {
  let result = await baseQuery(args, api, extraOptions)

  if (result?.error?.status === 403) {
    console.log("sending refresh token")

    // 發送 refresh 請求與 refresh token 以取得新的 access token
    const refreshResult = await baseQuery("/auth/refresh", api, extraOptions)

    if (refreshResult.data) {
      // 儲存新的 token
      api.dispatch(setCredentials({ ...refreshResult.data }))

      // 以新的 access token 重新請求
      result = await baseQuery(args, api, extraOptions)
    } else {
      if (refreshResult?.error?.status === 403) {
        refreshResult.error.data.message = "Your login has expired."
      }
      return refreshResult
    }
  }

  return result
}

export const apiSlice = createApi({
  baseQuery: baseQueryWithReauth,
  tagTypes: ["Note", "User"],
  endpoints: builder => ({}),
})
```

# Frontend - Deploy

## render

將 Frontend 程式碼上傳至 GitHub 後，使用 [render](https://render.com/) 部署靜態網站。

1. 創建 Static Site(靜態網站) : 選擇要連結的 repository。
2. 基礎設定。

   > Publish directory : 因使用 vite 開發，所以設為 dist。

   ![基礎設定畫面](basic-setting.webp)

3. Redirects/Rewrites 設定 : 使 refresh 正常運作。  
   ![重定向設定畫面](redirect-setting.webp)
4. 建立成功。  
   ![建立成功畫面](build-success.webp)

# Backend - 前置作業

## allowedOrigins.js

將允許訪問的 url 設為前端靜態網站的 url。

```js
// config/allowedOrigin.js
const allowedOrigins = ["https://technotes-6dqc.onrender.com"]

module.exports = allowedOrigins
```

# Backend - Deploy

## render

如同 Frontend ，將程式碼上傳至 GitHub 後，使用 [render](https://render.com/) 部署後端 API。

1. 創建 Web Service : 選擇要連結的 repository。
2. 基礎設定。  
   ![基礎設定畫面](backend-setting.webp)
3. 設定 Environment。

   > NODE_VERSION : 16.16.0

   ![Environment 設定畫面](env-setting.webp)

4. 建立成功。  
   ![建立成功畫面](backend-success.webp)
