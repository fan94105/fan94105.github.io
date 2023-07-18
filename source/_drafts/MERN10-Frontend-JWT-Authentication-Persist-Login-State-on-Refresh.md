---
title: [MERN10]Frontend - JWT Authentication & Persist Login State on Refresh
tags:
  - React
  - Redux
  - RTK Query
categories:
  - MERN
  - Frontend
---

將登入時收到的 token 加入 headers 中，且於後續的請求中一起發送至伺服器，以此辨別使用者是否登入，並完成保持使用者登入狀態的功能。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# RTK Query

## apiSlice.js

修改 apiSlice.js，取得 Redux state 中的 token，並在發送請求時一起發送至伺服器，以驗證使用者身分。並在 token 過期時發送 refresh 請求以 refresh token 取得新的 access token。

```js
// app/api/apiSlice.js
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react"

import { setCredentials } from "../../features/auth/authSlice"

const baseQuery = fetchBaseQuery({
  baseUrl: "http://localhost:3500",
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
        refreshResult.error.data.message = "Your login has expired. "
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

# Persist Login

## usePersist.js

是否在登錄時記住使用者以保持登入狀態。

```js
// hooks/usePersist.js
import { useState, useEffect } from "react"

const usePersist = () => {
  const [persist, setPersist] = useState(
    JSON.parse(localStorage.getItem("persist")) || false
  )

  useEffect(() => {
    localStorage.setItem("persist", JSON.stringify(persist))
  }, [persist])

  return [persist, setPersist]
}

export default usePersist
```

## Login.jsx

加入 Remember Me 選框，確認是否記住使用者。

```js
//features/auth/Login.jsx
import { useState, useEffect, useRef } from "react"
import { useNavigate, Link } from "react-router-dom"
import { useDispatch } from "react-redux"
import SyncLoader from "react-spinners/SyncLoader"

import { useLoginMutation } from "./authApiSlice"
import { setCredentials } from "./authSlice"
import usePersist from "../../hooks/usePersist"

const Login = () => {
  const userRef = useRef()
  const errRef = useRef()
  const [username, setUsername] = useState("")
  const [password, setPassword] = useState("")
  const [errMsg, setErrMsg] = useState("")
  const [persist, setPersist] = usePersist()

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
  const handleToggle = () => setPersist(prev => !prev)

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
            value={password}
            onChange={handlePwdInput}
            required
          />
          <button className="form__submit-button border">Sign In</button>

          <label htmlFor="persist" className="form__label">
            <input
              type="checkbox"
              name="persist"
              id="persist"
              onChange={handleToggle}
              checked={persist}
              className="form__checkbox"
            />
            &nbsp;Remember Me
          </label>
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

## PersistLogin.jsx

用以保持登入狀態的組件。

```js
// features/auth/PersistLogin.jsx
import { useState, useRef, useEffect } from "react"
import { Outlet, Link } from "react-router-dom"
import { useSelector } from "react-redux"
import SyncLoader from "react-spinners/SyncLoader"

import { useRefreshMutation } from "./authApiSlice"
import { selectCurrentToken } from "./authSlice"
import usePersist from "../../hooks/usePersist"

const PersistLogin = () => {
  const [persist] = usePersist()
  const token = useSelector(selectCurrentToken)
  const effectRan = useRef(false)

  const [trueSuccess, setTrueSuccess] = useState(false)

  const [refresh, { isUninitialized, isLoading, isSuccess, isError, error }] =
    useRefreshMutation()

  useEffect(() => {
    if (effectRan.current === true || process.env.NODE_ENV !== "development") {
      const verifyRefreshToken = async () => {
        try {
          await refresh()

          // 確保 refresh 請求確實成功
          setTrueSuccess(true)
        } catch (err) {
          console.log(err)
        }
      }

      if (!token && persist) {
        verifyRefreshToken()
      }
    }

    return () => (effectRan.current = true)
    // eslint-disable-next-line
  }, [])

  let content
  if (!persist) {
    // persist: no
    console.log("no persist")
    content = <Outlet />
  } else if (isLoading) {
    // persist: yes, token: no
    console.log("loading")
    content = (
      <div className="flex">
        <SyncLoader color="#d9d9d9" />
      </div>
    )
  } else if (isError) {
    // persist: yes, token: no
    console.log("error")
    content = (
      <p className="errmsg">
        {error?.data?.message}
        <Link to="/login">Please login again</Link>
      </p>
    )
  } else if (isSuccess && trueSuccess) {
    // persist: yes, token: yes
    console.log("success")
    content = <Outlet />
  } else if (token && isUninitialized) {
    // persist: yes, token: yes
    console.log("token and uninit")
    content = <Outlet />
  }

  return content
}

export default PersistLogin
```

## App.jsx

```js
// App.jsx
import { Routes, Route } from "react-router-dom"

import "./App.css"
import Layout from "./components/Layout"
import Public from "./components/Public"
import Login from "./features/auth/Login"
import Prefetch from "./features/auth/Prefetch"
import PersistLogin from "./features/auth/PersistLogin"
import DashLayout from "./components/DashLayout"
import Welcome from "./features/auth/Welcome"
import UsersList from "./features/users/UsersList"
import EditUser from "./features/users/EditUser"
import NewUserForm from "./features/users/NewUserForm"
import NotesList from "./features/notes/NotesList"
import EditNote from "./features/notes/EditNote"
import NewNote from "./features/notes/NewNote"

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Public />} />
        <Route path="login" element={<Login />} />

        <Route element={<PersistLogin />}>
          <Route element={<Prefetch />}>
            {/* Dash */}
            <Route path="dash" element={<DashLayout />}>
              <Route index element={<Welcome />} />

              {/* Users */}
              <Route path="users">
                <Route index element={<UsersList />} />
                <Route path=":id" element={<EditUser />} />
                <Route path="new" element={<NewUserForm />} />
              </Route>

              {/* Notes */}
              <Route path="notes">
                <Route index element={<NotesList />} />
                <Route path=":id" element={<EditNote />} />
                <Route path="new" element={<NewNote />} />
              </Route>
            </Route>
            {/* End Dash */}
          </Route>
        </Route>
      </Route>
    </Routes>
  )
}

export default App
```
