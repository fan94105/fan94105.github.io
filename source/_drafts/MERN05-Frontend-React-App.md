---
title: [MERN05]Frontend - React App
tags:
  - React
  - React Router
categories:
  - MERN
  - Frontend
---

後端 API 建立完成後，即開始建構前端頁面，使用 React Router 搭建網頁的基本路由，並完成使用者登入後所展示的頁面。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# React Router

安裝 react-router-dom :

```
npm i react-router-dom
```

main.jsx 建立 Router 模板 :

```js
// main.jsx
import React from "react"
import ReactDOM from "react-dom/client"
import { BrowserRouter as Router, Routes, Route } from "react-router-dom"

import App from "./App.jsx"
import "./index.css"

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <Router>
      <Routes>
        <Route path="/*" element={<App />} />
      </Routes>
    </Router>
  </React.StrictMode>
)
```

App.jsx 建構網站前端路由 :

```js
// App.jsx
import { Routes, Route } from "react-router-dom"

import "./App.css"
import Layout from "./components/Layout"
import Public from "./components/Public"
import Login from "./features/auth/Login"
import DashLayout from "./components/DashLayout"
import Welcome from "./features/auth/Welcome"
import NotesList from "./features/notes/NotesList"
import UsersList from "./features/users/UsersList"

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Public />} />
        <Route path="login" element={<Login />} />

        {/* Dash */}
        <Route path="dash" element={<DashLayout />}>
          <Route index element={<Welcome />} />

          {/* Notes */}
          <Route path="notes">
            <Route index element={<NotesList />} />
          </Route>

          {/* Users */}
          <Route path="users">
            <Route index element={<UsersList />} />
          </Route>
        </Route>
        {/* End Dash */}
      </Route>
    </Routes>
  )
}

export default App
```

# Dash 頁面

安裝 [Font Awesome](https://fontawesome.com/v6/docs/web/use-with/react/) 的 React Component，在頁面中加入 icon。

```
npm i --save @fortawesome/fontawesome-svg-core

# Free icons styles
npm i --save @fortawesome/free-solid-svg-icons
npm i --save @fortawesome/free-regular-svg-icons
npm i --save @fortawesome/react-fontawesome@latest
```

使用者登入後展示的頁面 :

```js
// components/DashLayout.jsx
import { Outlet } from "react-router-dom"

import DashHeader from "./DashHeader"
import DashFooter from "./DashFooter"

const DashLayout = () => {
  return (
    <div className="dash-layout border">
      <DashHeader />
      <div className="dash-container">
        <Outlet />
      </div>
      <DashFooter />
    </div>
  )
}

export default DashLayout
```

Dash 頁面的 Header 組件 :

```js
// components/DashHeader.jsx
import { Link } from "react-router-dom"

const DashHeader = () => {
  const content = (
    <header className="dash-header flex">
      <div className="dash-header__container">
        <Link to="/dash">
          <h1 className="dash-header__title">techNotes</h1>
        </Link>
        <nav className="dash-header__nav">{/* nav buttons */}</nav>
      </div>
    </header>
  )

  return content
}

export default DashHeader
```

Dash 頁面的子組件 :

```js
// features/auth/Welcome.jsx
import { Link } from "react-router-dom"

const Welcome = () => {
  const date = new Date()
  const today = new Intl.DateTimeFormat("zh-TW", {
    dateStyle: "full",
    timeStyle: "medium",
  }).format(date)

  const content = (
    <section className="welcome">
      <p>{today}</p>
      <h1>Welcome!</h1>
      <div className="link">
        <div className="notes">
          <p className="border">
            <Link to="/dash/notes">View techNotes</Link>
          </p>
        </div>
        <div className="users">
          <p className="border">
            <Link to="/dash/users">View User Settings</Link>
          </p>
        </div>
      </div>
    </section>
  )

  return content
}

export default Welcome
```

Dash 頁面的 Footer 組件 :

```js
// components/DashFooter.jsx
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faHouse } from "@fortawesome/free-solid-svg-icons"
import { useNavigate, useLocation } from "react-router-dom"

const DashFooter = () => {
  const navigate = useNavigate()

  const { pathname } = useLocation()

  const onGoHomeClicked = () => navigate("/dash")

  let goHomeButton = null
  if (pathname !== "/dash") {
    goHomeButton = (
      <button className="icon-button" title="Home" onClick={onGoHomeClicked}>
        <FontAwesomeIcon icon={faHouse} />
      </button>
    )
  }

  const content = (
    <footer className="dash-footer flex">
      {goHomeButton}
      <p className="nowrap">Current User:</p>
      <p>Status:</p>
    </footer>
  )

  return content
}

export default DashFooter
```
