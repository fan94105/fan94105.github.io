---
title: [MERN11]Frontend - React 使用者權限控制
tags:
  - React
categories:
  - MERN
  - Frontend
---

依據使用者登入的身分，提供不同的控制與訪問權限。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# 判別登入身分

安裝 jwt-decode，解碼 jwt :

```
npm i jwt-decode
```

## useAuth.js

解碼 access token，判斷登入使用者的身分別。

```js
// hooks/useAuth.js
import { useSelector } from "react-redux"
import jwtDecode from "jwt-decode"

import { selectCurrentToken } from "../features/auth/authSlice"

const useAuth = () => {
  const token = useSelector(selectCurrentToken)

  let isManager = false
  let isAdmin = false
  let status = "Employee"

  if (token) {
    const decoded = jwtDecode(token)
    const { username, roles } = decoded.UserInfo

    isManager = roles.includes("Manager")
    isAdmin = roles.includes("Admin")

    if (isManager) {
      status = "Manager"
    }
    if (isAdmin) {
      status = "Admin"
    }

    return {
      username,
      roles,
      isManager,
      isAdmin,
      status,
    }
  }

  return {
    username: "",
    roles: [],
    isManager,
    isAdmin,
    status,
  }
}

export default useAuth
```

## DashFooter.jsx

添加使用者資訊。

```js
// components/DashFooter.jsx
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faHouse } from "@fortawesome/free-solid-svg-icons"
import { useNavigate, useLocation } from "react-router-dom"

import useAuth from "../hooks/useAuth"

const DashFooter = () => {
  const { username, status } = useAuth()

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
      <p>User: {username}</p>
      <p>Status: {status}</p>
    </footer>
  )

  return content
}

export default DashFooter
```

# 權限控制

## Welcome.jsx

只有 Manager 和 Admin 才能編輯或新增使用者。

```js
// features/auth/Welcome.jsx
import { Link } from "react-router-dom"

import useAuth from "../../hooks/useAuth"

const Welcome = () => {
  const { username, isManager, isAdmin } = useAuth()

  const date = new Date()
  const today = new Intl.DateTimeFormat("zh-TW", {
    dateStyle: "full",
    timeStyle: "medium",
  }).format(date)

  const content = (
    <section className="welcome">
      <p>{today}</p>
      <h1>Welcome {username}!</h1>
      <div className="link">
        <div className="notes">
          <p className="border">
            <Link to="/dash/notes">View techNotes</Link>
          </p>
          <p className="border">
            <Link to="/dash/notes/new">Add New techNotes</Link>
          </p>
        </div>

        {/* 只有 Manager 和 Admin 才能編輯或新增使用者 */}
        {(isManager || isAdmin) && (
          <div className="users">
            <p className="border">
              <Link to="/dash/users">View User Settings</Link>
            </p>
            <p className="border">
              <Link to="/dash/users/new">Add New User</Link>
            </p>
          </div>
        )}
      </div>
    </section>
  )

  return content
}

export default Welcome
```

## DashHeader.jsx

加入新的 icon。

```js
// components/DashHeader.jsx
import { useEffect } from "react"
import { useNavigate, Link } from "react-router-dom"
import { useLocation } from "react-router-dom"
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import {
  faRightFromBracket,
  faFileCirclePlus,
  faFilePen,
  faUserGear,
  faUserPlus,
} from "@fortawesome/free-solid-svg-icons"
import SyncLoader from "react-spinners/SyncLoader"

import { useSendLogoutMutation } from "../features/auth/authApiSlice"
import useAuth from "../hooks/useAuth"

const DASH_REGEX = /^\/dash(\/)?$/
const NOTES_REGEX = /^\/dash\/notes(\/)?$/
const USERS_REGEX = /^\/dash\/users(\/)?$/

const DashHeader = () => {
  const { isManager, isAdmin } = useAuth()

  const navigate = useNavigate()
  const { pathname } = useLocation()

  const [sendLogout, { isLoading, isSuccess, isError, error }] =
    useSendLogoutMutation()

  useEffect(() => {
    if (isSuccess) {
      navigate("/")
    }
  }, [isSuccess, navigate])

  const onNewNoteClicked = () => {
    navigate("/dash/notes/new")
  }
  const onNewUserClicked = () => {
    navigate("/dash/users/new")
  }
  const onNotesClicked = () => {
    navigate("/dash/notes")
  }
  const onUsersClicked = () => {
    navigate("/dash/users")
  }

  const onLogoutClicked = () => {
    sendLogout()
  }

  let newNoteButton = null
  if (NOTES_REGEX.test(pathname)) {
    newNoteButton = (
      <button
        className="icon-button"
        title="New Note"
        onClick={onNewNoteClicked}
      >
        <FontAwesomeIcon icon={faFileCirclePlus} />
      </button>
    )
  }

  let newUserButton = null
  if (USERS_REGEX.test(pathname)) {
    newUserButton = (
      <button
        className="icon-button"
        title="New User"
        onClick={onNewUserClicked}
      >
        <FontAwesomeIcon icon={faUserPlus} />
      </button>
    )
  }

  let usersButton = null
  if (isManager || isAdmin) {
    if (!USERS_REGEX.test(pathname) && pathname.includes("/dash")) {
      usersButton = (
        <button className="icon-button" title="Users" onClick={onUsersClicked}>
          <FontAwesomeIcon icon={faUserGear} />
        </button>
      )
    }
  }

  let notesButton = null
  if (!NOTES_REGEX.test(pathname) && pathname.includes("/dash")) {
    notesButton = (
      <button className="icon-button" title="Notes" onClick={onNotesClicked}>
        <FontAwesomeIcon icon={faFilePen} />
      </button>
    )
  }

  const logoutButton = (
    <button className="icon-button" title="Logout" onClick={onLogoutClicked}>
      <FontAwesomeIcon icon={faRightFromBracket} />
    </button>
  )

  const errClass = isError ? "errmsg" : "offscreen"

  let buttonContent
  if (isLoading) {
    buttonContent = <SyncLoader color="#fff" />
  } else {
    buttonContent = (
      <>
        {newNoteButton}
        {notesButton}
        {newUserButton}
        {usersButton}
        {logoutButton}
      </>
    )
  }

  const content = (
    <>
      <p className={errClass}>{error?.data?.message}</p>

      <header className="dash-header flex">
        <div className="dash-header__container">
          <Link to="/dash">
            <h1 className="dash-header__title">techNotes</h1>
          </Link>
          <nav className="dash-header__nav">
            {/* nav buttons */}
            {buttonContent}
          </nav>
        </div>
      </header>
    </>
  )

  return content
}

export default DashHeader
```

## NotesList.jsx

只有 Manager 與 Admin 可以編輯與訪問所有 note，而 Employee 只能編輯與訪問自己的 note。

```js
// features/notes/NotesList.jsx
import SyncLoader from "react-spinners/SyncLoader"

import { useGetNotesQuery } from "./notesApiSlice"
import Note from "./Note"
import useAuth from "../../hooks/useAuth"

const NotesList = () => {
  const { username, isManager, isAdmin } = useAuth()

  const {
    data: notes,
    isLoading,
    isSuccess,
    isError,
    error,
  } = useGetNotesQuery("notesList", {
    pollingInterval: 15000, // 輪詢間隔(ms)
    refetchOnFocus: true, // 是否在獲取焦點時重新加載數據
    refetchOnMountOrArgChange: true, // 是否每次都重新加載數據，false 正常使用快取
    refetchOnReconnect: true, // 是否在重新連接時重新加載數據
  })

  let content

  if (isLoading) {
    content = (
      <div className="flex">
        <SyncLoader color="#d9d9d9" />
      </div>
    )
  }

  if (isError) {
    content = <p className="errmsg">{error?.data.message}</p>
  }

  if (isSuccess) {
    const { ids, entities } = notes

    let filteredIds
    // Manager 與 Admin 可操作所有 notes
    if (isManager || isAdmin) {
      filteredIds = [...ids]
    } else {
      filteredIds = ids.filter(noteId => entities[noteId].username === username)
    }

    if (filteredIds.length === 0) {
      return (
        <div className="flex">
          <p className="errmsg">No notes found...</p>
        </div>
      )
    }

    const tableContent =
      ids?.length &&
      filteredIds.map(noteId => <Note key={noteId} noteId={noteId} />)

    content = (
      <table className="table">
        <thead className="table__thead">
          <tr>
            <th className="table__th" scope="col">
              Status
            </th>
            <th className="table__th" scope="col">
              Created
            </th>
            <th className="table__th" scope="col">
              Updated
            </th>
            <th className="table__th" scope="col">
              Title
            </th>
            <th className="table__th" scope="col">
              Owner
            </th>
            <th className="table__th" scope="col">
              Edit
            </th>
          </tr>
        </thead>
        <tbody>{tableContent}</tbody>
      </table>
    )
  }

  return content
}

export default NotesList
```

## EditNoteForm.jsx

只有 Manager 與 Admin 可以刪除 note。

```js
// features/notes/EditNoteForm.jsx
import { useState, useEffect } from "react"
import { useNavigate } from "react-router-dom"
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faSave, faTrashCan } from "@fortawesome/free-solid-svg-icons"

import { useUpdateNoteMutation, useDeleteNoteMutation } from "./notesApiSlice"
import useAuth from "../../hooks/useAuth"

const EditNoteForm = ({ note, users }) => {
  const { isManager, isAdmin } = useAuth()

  const [updateNote, { isLoading, isSuccess, isError, error }] =
    useUpdateNoteMutation()

  const [
    deleteNote,
    { isSuccess: isDelSuccess, isError: isDelError, error: delerror },
  ] = useDeleteNoteMutation()

  const navigate = useNavigate()

  const [title, setTitle] = useState(note.title)
  const [text, setText] = useState(note.text)
  const [completed, setCompleted] = useState(note.completed)
  const [userId, setUserId] = useState(note.user)

  useEffect(() => {
    if (isSuccess || isDelSuccess) {
      setTitle("")
      setText("")
      setUserId("")
      navigate("/dash/notes")
    }
  }, [isSuccess, isDelSuccess, navigate])

  const onTitleChanged = e => setTitle(e.target.value)
  const onTextChanged = e => setText(e.target.value)
  const onCompletedChanged = () => setCompleted(prev => !prev)
  const onUserIdChanged = e => setUserId(e.target.value)

  const canSave = [title, text, userId].every(Boolean) && !isLoading

  const onSaveNoteClicked = async () => {
    if (canSave) {
      await updateNote({ id: note.id, title, text, completed, user: userId })
    }
  }

  const onDeleteNoteClicked = async () => {
    await deleteNote({ id: note.id })
  }

  const created = new Date(note.createdAt).toLocaleString("zh-TW", {
    dateStyle: "medium",
    timeStyle: "medium",
  })
  const updated = new Date(note.updatedAt).toLocaleString("zh-TW", {
    dateStyle: "medium",
    timeStyle: "medium",
  })

  const usersOptions = users.map(user => (
    <option key={user.id} value={user.id}>
      {user.username}
    </option>
  ))

  const errClass = isError || isDelError ? "errmsg" : "offscreen"
  const validTitleClass = !title ? "form__input--incomplete" : ""
  const validTextClass = !text ? "form__input--incomplete" : ""

  const errContent = (error?.data.message || delerror?.data.message) ?? ""

  let deleteButton = null
  if (isManager || isAdmin) {
    deleteButton = (
      <button
        className="icon-button"
        title="Delete"
        onClick={onDeleteNoteClicked}
      >
        <FontAwesomeIcon icon={faTrashCan} />
      </button>
    )
  }

  const content = (
    <>
      <p className={errClass}>{errContent}</p>

      <form className="form" onSubmit={e => e.preventDefault()}>
        <div className="form__title-row">
          <h2>Edit Note #{note.ticket}</h2>
          <div className="form__action-buttons">
            <button
              className="icon-button"
              title="Save"
              onClick={onSaveNoteClicked}
              disabled={!canSave}
            >
              <FontAwesomeIcon icon={faSave} />
            </button>
            {deleteButton}
          </div>
        </div>
        <label htmlFor="note-title" className="form__label">
          Title:
        </label>
        <input
          type="text"
          className={`form__input ${validTitleClass}`}
          id="note-title"
          name="title"
          autoComplete="off"
          value={title}
          onChange={onTitleChanged}
        />
        <label htmlFor="note-text" className="form__label">
          Text:
        </label>
        <textarea
          className={`form__input ${validTextClass}`}
          id="note-text"
          name="text"
          value={text}
          onChange={onTextChanged}
        />
        <label
          htmlFor="note-completed"
          className="form__label form__checkbox-container"
        >
          Completed:
        </label>
        <input
          className="form__checkbox"
          type="checkbox"
          name="completed"
          id="note-completed"
          checked={completed}
          onChange={onCompletedChanged}
        />
        <label htmlFor="note-username" className="form__label">
          USER:
        </label>
        <select
          className="form__select"
          name="username"
          id="note-username"
          value={userId}
          onChange={onUserIdChanged}
        >
          {usersOptions}
        </select>
        <div className="form__divider">
          <div className="form__label">
            Created:
            <p>{created}</p>
          </div>
          <div className="form__label">
            Updated:
            <p>{updated}</p>
          </div>
        </div>
      </form>
    </>
  )

  return content
}

export default EditNoteForm
```

# Protected Routes

## RequireAuth.jsx

判斷子路由是否允許目前使用者訪問。

```js
// features/auth/RequireAuth.jsx
import { useLocation, Navigate, Outlet } from "react-router-dom"

import useAuth from "../hooks/useAuth"

const RequireAuth = ({ allowedRoles }) => {
  const location = useLocation()
  const { roles } = useAuth()

  const content = roles.some(role => allowedRoles.includes(role)) ? (
    <Outlet />
  ) : (
    <Navigate to="/login" state={{ from: location }} replace />
  )

  return content
}

export default RequireAuth
```

## App.jsx

啟用 RequireAuth.jsx，設定每個路由具有訪問權限的身分。

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
import RequireAuth from "./features/auth/RequireAuth"
import { ROLES } from "./config/roles"

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        {/* public routes */}
        <Route index element={<Public />} />
        <Route path="login" element={<Login />} />

        {/* protected routes */}
        <Route element={<PersistLogin />}>
          <Route
            element={<RequireAuth allowedRoles={[...Object.values(ROLES)]} />}
          >
            <Route element={<Prefetch />}>
              {/* Dash */}
              <Route path="dash" element={<DashLayout />}>
                <Route index element={<Welcome />} />

                {/* Users */}
                <Route
                  element={
                    <RequireAuth allowedRoles={[ROLES.Manager, ROLES.Admin]} />
                  }
                >
                  <Route path="users">
                    <Route index element={<UsersList />} />
                    <Route path=":id" element={<EditUser />} />
                    <Route path="new" element={<NewUserForm />} />
                  </Route>
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
      </Route>
    </Routes>
  )
}

export default App
```
