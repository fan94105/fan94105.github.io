---
title: [MERN09]Frontend - React Login Authentication with Redux
tags:
  - React
  - Redux
  - RTK Query
categories:
  - MERN
  - Frontend
---

完成登入及登出功能，使用者登入後取得並儲存 token，登出後即清除 token。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# Redux

## authSlice.js

儲存登入使用者的資訊(accessToken)。

```js
// features/auth/authSlice.js
import { createSlice } from "@reduxjs/toolkit"

const authSlice = createSlice({
  name: "auth",
  initialState: {
    token: null,
  },
  reducers: {
    setCredentials: (state, action) => {
      const { accessToken } = action.payload
      state.token = accessToken
    },
    logOut: (state, action) => {
      state.token = null
    },
  },
})

export const { setCredentials, logOut } = authSlice.actions

export default authSlice.reducer

export const selectCurrentToken = state => state.auth.token
```

## authApiSlice.js

處理 login、refresh、logout 請求。

```js
// features/auth/authApiSlice.js
import { apiSlice } from "../../app/api/apiSlice"
import { logOut } from "./authSlice"
import { setCredentials } from "./authSlice"

export const authApiSlice = apiSlice.injectEndpoints({
  endpoints: builder => ({
    login: builder.mutation({
      query: credentials => ({
        url: "/auth",
        method: "POST",
        body: {
          ...credentials,
        },
      }),
    }),
    sendLogout: builder.mutation({
      query: () => ({
        url: "/auth/logout",
        method: "POST",
      }),
      async onQueryStarted(arg, { dispatch, queryFulfilled }) {
        try {
          const { data } = await queryFulfilled
          console.log(data)

          // 移除 Redux state 中的 token
          dispatch(logOut())

          // 重置 Redux 中的 api
          setTimeout(() => {
            dispatch(apiSlice.util.resetApiState())
          }, 1000)
        } catch (err) {
          console.log(err)
        }
      },
    }),
    refresh: builder.mutation({
      query: () => ({
        url: "/auth/refresh",
        method: "GET",
      }),
      async onQueryStarted(arg, { dispatch, queryFulfilled }) {
        try {
          const { data } = await queryFulfilled
          console.log(data)
          const { accessToken } = data
          dispatch(setCredentials({ accessToken }))
        } catch (err) {
          console.log(err)
        }
      },
    }),
  }),
})

export const { useLoginMutation, useSendLogoutMutation, useRefreshQuery } =
  authApiSlice
```

## store.js

註冊 authReducer。

```js
// api/store.js
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
  devTools: true,
})

// 啟用 refetchOnFocus 與 refetchOnReconnect
setupListeners(store.dispatch)
```

# Login & Logout

## Login.jsx

建立登入頁面。

```js
// features/auth/Login.jsx
import { useState, useEffect, useRef } from "react"
import { useNavigate, Link } from "react-router-dom"
import { useDispatch } from "react-redux"
import SyncLoader from "react-spinners/SyncLoader"

import { useLoginMutation } from "./authApiSlice"
import { setCredentials } from "./authSlice"

const Login = () => {
  const userRef = useRef()
  const errRef = useRef()
  const [username, setUsername] = useState("")
  const [password, setPassword] = useState("")
  const [errMsg, setErrMsg] = useState("")

  const navigate = useNavigate()
  const dispatch = useDispatch()

  const [login, { isLoading }] = useLoginMutation()

  useEffect(() => {
    userRef.current.focus()
  }, [])

  useEffect(() => {
    setErrMsg("")
  }, [username, password])

  const handleSubmit = async e => {
    e.preventDefault()

    try {
      const { accessToken } = await login({ username, password }).unwrap()
      dispatch(setCredentials({ accessToken }))
      setUsername("")
      setPassword("")
      navigate("/dash")
    } catch (err) {
      if (!err.status) {
        setErrMsg("No Server Response")
      } else if (err.status === 400) {
        setErrMsg("Missing Username or Password")
      } else if (err.status === 401) {
        setErrMsg("Unauthorized")
      } else {
        setErrMsg(err.data?.message)
      }
      errRef.current.focus()
    }
  }

  const handleUserInput = e => setUsername(e.target.value)
  const handlePwdInput = e => setPassword(e.target.value)

  const errClass = errMsg ? "errmsg" : "offscreen"

  if (isLoading) {
    return (
      <div className="flex">
        <SyncLoader color="#d9d9d9" />
      </div>
    )
  }

  const content = (
    <section className="public">
      <header>
        <h1>Employee Login</h1>
      </header>
      <main className="login">
        <p ref={errRef} className={errClass} aria-live="assertive">
          {errMsg}
        </p>

        <form className="form" onSubmit={handleSubmit}>
          <label htmlFor="username" className="form__label">
            Username:
          </label>
          <input
            type="text"
            className="form__input"
            id="username"
            name="username"
            ref={userRef}
            value={username}
            onChange={handleUserInput}
            autoComplete="off"
            required
          />
          <label htmlFor="password" className="form__label">
            Password:
          </label>
          <input
            type="password"
            className="form__input"
            id="password"
            name="password"
            ref={userRef}
            value={password}
            onChange={handlePwdInput}
            required
          />
          <button className="form__submit-button border">Sign In</button>
        </form>
      </main>
      <footer>
        <Link className="backHome" to="/">
          Back to Home 🚀
        </Link>
      </footer>
    </section>
  )

  return content
}

export default Login
```

## DashHeader.jsx

在 DashHeader 中加入 Logout 按鈕。

```js
// components/DashHeader.jsx
import { useEffect } from "react"
import { useNavigate, Link } from "react-router-dom"
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faRightFromBracket } from "@fortawesome/free-solid-svg-icons"
import SyncLoader from "react-spinners/SyncLoader"

import { useSendLogoutMutation } from "../features/auth/authApiSlice"

const DashHeader = () => {
  const navigate = useNavigate()

  const [sendLogout, { isLoading, isSuccess, isError, error }] =
    useSendLogoutMutation()

  useEffect(() => {
    if (isSuccess) {
      navigate("/")
    }
  }, [isSuccess, navigate])

  const onLogoutClicked = () => {
    sendLogout()
  }

  if (isLoading) {
    return <SyncLoader color="#d9d9d9" />
  }

  if (isError) {
    return <p>Error: {error?.data?.message}</p>
  }

  const logoutButton = (
    <button className="icon-button" title="Logout" onClick={onLogoutClicked}>
      <FontAwesomeIcon icon={faRightFromBracket} />
    </button>
  )

  const content = (
    <header className="dash-header flex">
      <div className="dash-header__container">
        <Link to="/dash">
          <h1 className="dash-header__title">techNotes</h1>
        </Link>
        <nav className="dash-header__nav">
          {/* nav buttons */}
          {logoutButton}
        </nav>
      </div>
    </header>
  )

  return content
}

export default DashHeader
```
