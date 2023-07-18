---
title: [MERN07]Frontend - React Form with Redux & RTK-Query
tags:
  - React
  - Redux
  - RTK Query
categories:
  - MERN
  - Frontend
---

建立完善 CRUD HTTP 請求方法，透過 RTK Query 發出 HTTP 請求，完成 user 及 note 編輯頁面與功能。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# Redux & RTK Query

## usersApiSlice.js

```js
// features/users/usersApiSlice.js
export const usersApiSlice = apiSlice.injectEndpoints({
  endpoints: builder => ({
    getUsers: builder.query({
      query: () => ({
        url: "/users",
        validateStatus: (response, result) => {
          return response.status === 200 && !result.isError
        },
      }),
      // 將 query 回應物件改為標準化物件
      transformResponse: responseData => {
        const loadedUsers = responseData.map(user => {
          // 為了 ids，設置 id
          user.id = user._id
          return user
        })
        return usersAdapter.setAll(initialState, loadedUsers)
      },
      providesTags: (result, error, arg) => {
        if (result?.ids) {
          return [
            { type: "User", id: "LIST" },
            ...result.ids.map(id => ({ type: "User", id })),
          ]
        } else {
          return [{ type: "User", id: "LIST" }]
        }
      },
    }),
    addNewUser: builder.mutation({
      query: initialUserData => ({
        url: "/users",
        method: "POST",
        body: {
          ...initialUserData,
        },
      }),
      invalidatesTags: [{ type: "User", id: "LIST" }],
    }),
    updateUser: builder.mutation({
      query: initialUserData => ({
        url: "/users",
        method: "PATCH",
        body: {
          ...initialUserData,
        },
      }),
      invalidatesTags: (result, err, arg) => [{ type: "User", id: arg.id }],
    }),
    deleteUser: builder.mutation({
      query: ({ id }) => ({
        url: "/users",
        method: "DELETE",
        body: {
          id,
        },
      }),
      invalidatesTags: (result, err, arg) => [{ type: "User", id: arg.id }],
    }),
  }),
})

export const {
  useGetUsersQuery,
  useAddNewUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation,
} = usersApiSlice
```

## notesApiSlice.js

```js
// features/notes/notesApiSlice.js
export const notesApiSlice = apiSlice.injectEndpoints({
  endpoints: builder => ({
    getNotes: builder.query({
      query: () => ({
        url: "/notes",
        validateStatus: (response, result) => {
          return response.status === 200 && !result.isError
        },
      }),
      // 將 query 回應物件改為標準化物件
      transformResponse: responseData => {
        const loadedNotes = responseData.map(note => {
          // 為了 ids，設置 id
          note.id = note._id
          return note
        })
        return notesAdapter.setAll(initialState, loadedNotes)
      },
      providesTags: (result, error, arg) => {
        if (result?.ids) {
          return [
            { type: "Note", id: "LIST" },
            ...result.ids.map(id => ({ type: "Note", id })),
          ]
        } else {
          return [{ type: "Note", id: "LIST" }]
        }
      },
    }),
    addNewNote: builder.mutation({
      query: initialNoteData => ({
        url: "/notes",
        method: "POST",
        body: {
          ...initialNoteData,
        },
      }),
      invalidatesTags: [{ type: "Note", id: "LIST" }],
    }),
    updateNote: builder.mutation({
      query: initialNoteData => ({
        url: "/notes",
        method: "PATCH",
        body: {
          ...initialNoteData,
        },
      }),
      invalidatesTags: (result, err, arg) => [{ type: "Note", id: arg.id }],
    }),
    deleteNote: builder.mutation({
      query: ({ id }) => ({
        url: "/notes",
        method: "DELETE",
        body: { id },
      }),
      invalidatesTags: (result, err, arg) => [{ type: "Note", id: arg.id }],
    }),
  }),
})

export const {
  useGetNotesQuery,
  useAddNewNoteMutation,
  useUpdateNoteMutation,
  useDeleteNoteMutation,
} = notesApiSlice
```

# RTK Quert Subscriptions

實現數據即時更新的功能。當伺服器端的數據變化時，將自動更新用戶端數據，使應用程式可以實時獲取最新數據，無需手動刷新頁面。

## Prefetch.jsx

```js
// features/auth/Prefetch.jsx
import { useEffect } from "react"
import { Outlet } from "react-router-dom"

import { store } from "../../app/store"
import { notesApiSlice } from "../notes/notesApiSlice"
import { usersApiSlice } from "../users/usersApiSlice"

const Prefetch = () => {
  useEffect(() => {
    store.dispatch(
      notesApiSlice.util.prefetch("getNotes", "notesList", { force: true })
    )
    store.dispatch(
      usersApiSlice.util.prefetch("getUsers", "usersList", { force: true })
    )
  }, [])

  return <Outlet />
}

export default Prefetch
```

# React Form

建立與 user 和 note 的新增與編輯頁面。

## roles.js

```js
// config/roles.js
export const ROLES = {
  Employee: "Employee",
  Manager: "Manager",
  Admin: "Admin",
}
```

## 新增 User

### NewUserForm.jsx

```js
// features/users/NewUserForm.jsx
import { useState, useEffect } from "react"
import { useNavigate } from "react-router-dom"
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faSave } from "@fortawesome/free-solid-svg-icons"

import { useAddNewUserMutation } from "./usersApiSlice"
import { ROLES } from "../../config/roles"

const USER_REGEX = /^[A-z]{3,20}$/
const PWD_REGEX = /^[A-z0-9!@#$%]{4,12}$/

const NewUserForm = () => {
  const [addNewUser, { isLoading, isSuccess, isError, error }] =
    useAddNewUserMutation()

  const navigate = useNavigate()

  const [username, setUsername] = useState("")
  const [validUsername, setValidUsername] = useState(false)
  const [password, setPassword] = useState("")
  const [validPassword, setValidPassword] = useState(false)
  const [roles, setRoles] = useState(["Employee"])

  useEffect(() => {
    setValidUsername(USER_REGEX.test(username))
  }, [username])

  useEffect(() => {
    setValidPassword(PWD_REGEX.test(password))
  }, [password])

  useEffect(() => {
    if (isSuccess) {
      setUsername("")
      setPassword("")
      setRoles([])
      navigate("/dash/users")
    }
  }, [isSuccess, navigate])

  const onUsernameChanged = e => setUsername(e.target.value)
  const onPasswordChanged = e => setPassword(e.target.value)

  const onRolesChanged = e => {
    const values = Array.from(
      e.target.selectedOptions, //HTMLCollection
      option => option.value
    )
    setRoles(values)
  }

  const canSave =
    [roles.length, validUsername, validPassword].every(Boolean) && !isLoading

  const onSaveUserClicked = async e => {
    e.preventDefault()
    if (canSave) {
      await addNewUser({ username, password, roles })
    }
  }

  const options = Object.values(ROLES).map(role => {
    return (
      <option key={role} value={role}>
        {" "}
        {role}
      </option>
    )
  })

  const errClass = isError ? "errmsg" : "offscreen"
  const validUserClass = !validUsername ? "form__input--incomplete" : ""
  const validPwdClass = !validPassword ? "form__input--incomplete" : ""
  const validRolesClass = !Boolean(roles.length)
    ? "form__input--incomplete"
    : ""

  const content = (
    <>
      <p className={errClass}>{error?.data?.message}</p>

      <form className="form" onSubmit={onSaveUserClicked}>
        <div className="form__title-row">
          <h2>New User</h2>
          <div className="form__action-buttons">
            <button className="icon-button" title="Save" disabled={!canSave}>
              <FontAwesomeIcon icon={faSave} />
            </button>
          </div>
        </div>
        <label className="form__label" htmlFor="username">
          Username: <span className="nowrap">[3-20 letters]</span>
        </label>
        <input
          className={`form__input ${validUserClass}`}
          id="username"
          name="username"
          type="text"
          autoComplete="off"
          value={username}
          onChange={onUsernameChanged}
        />

        <label className="form__label" htmlFor="password">
          Password: <span className="nowrap">[4-12 chars incl. !@#$%]</span>
        </label>
        <input
          className={`form__input ${validPwdClass}`}
          id="password"
          name="password"
          type="password"
          value={password}
          onChange={onPasswordChanged}
        />

        <label className="form__label" htmlFor="roles">
          ASSIGNED ROLES:
        </label>
        <select
          id="roles"
          name="roles"
          className={`form__select ${validRolesClass}`}
          multiple={true}
          size="3"
          value={roles}
          onChange={onRolesChanged}
        >
          {options}
        </select>
      </form>
    </>
  )

  return content
}
export default NewUserForm
```

## 編輯 User

### EditUser.jsx

```js
// features/users/EditUser.jsx
import { useParams } from "react-router-dom"
import SyncLoader from "react-spinners/SyncLoader"

import { useGetUsersQuery } from "./usersApiSlice"
import EditUserForm from "./EditUserForm"

const EditUser = () => {
  const { id } = useParams()

  const { user } = useGetUsersQuery("usersList", {
    selectFromResult: ({ data }) => ({
      user: data?.entities[id],
    }),
  })

  if (!user) {
    return (
      <div className="flex">
        <SyncLoader color="#d9d9d9" />
      </div>
    )
  }

  const content = <EditUserForm user={user} />

  return content
}

export default EditUser
```

### EditUserForm.jsx

```js
// features/users/EditUserForm.jsx
import { useState, useEffect } from "react"
import { useNavigate } from "react-router-dom"
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faSave, faTrashCan } from "@fortawesome/free-solid-svg-icons"

import { useUpdateUserMutation, useDeleteUserMutation } from "./usersApiSlice"
import { ROLES } from "../../config/roles"

const USER_REGEX = /^[A-z]{3,20}$/
const PWD_REGEX = /^[A-z0-9!@#$%]{4,12}$/

const EditUserForm = ({ user }) => {
  const [updateUser, { isLoading, isSuccess, isError, error }] =
    useUpdateUserMutation()

  const [
    deleteUser,
    { isSuccess: isDelSuccess, isError: isDelError, error: delerror },
  ] = useDeleteUserMutation()

  const navigate = useNavigate()

  const [username, setUsername] = useState(user.username)
  const [validUsername, setValidUsername] = useState(false)
  const [password, setPassword] = useState("")
  const [validPassword, setValidPassword] = useState(false)
  const [roles, setRoles] = useState(user.roles)
  const [active, setActive] = useState(user.active)

  useEffect(() => {
    setValidUsername(USER_REGEX.test(username))
  }, [username])

  useEffect(() => {
    setValidPassword(PWD_REGEX.test(password))
  }, [password])

  useEffect(() => {
    if (isSuccess || isDelSuccess) {
      setUsername("")
      setPassword("")
      setRoles([])
      navigate("/dash/users")
    }
  }, [isSuccess, isDelSuccess, navigate])

  const onUsernameChanged = e => setUsername(e.target.value)
  const onPasswordChanged = e => setPassword(e.target.value)

  const onRolesChanged = e => {
    const values = Array.from(e.target.selectedOptions, option => option.value)
    setRoles(values)
  }

  const onActiveChanged = () => setActive(prev => !prev)

  const onSaveUserClicked = async () => {
    if (password) {
      await updateUser({ id: user.id, username, password, roles, active })
    } else {
      await updateUser({ id: user.id, username, roles, active })
    }
  }

  const onDeleteUserClicked = async () => {
    await deleteUser({ id: user.id })
  }

  let canSave
  if (password) {
    canSave =
      [roles.length, validUsername, validPassword].every(Boolean) && !isLoading
  } else {
    canSave = [roles.length, validUsername].every(Boolean) && !isLoading
  }

  const options = Object.values(ROLES).map(role => {
    return (
      <option key={role} value={role}>
        {" "}
        {role}
      </option>
    )
  })

  const errClass = isError || isDelError ? "errmsg" : "offscreen"
  const validUserClass = !validUsername ? "form__input--incomplete" : ""
  const validPwdClass =
    password && !validPassword ? "form__input--incomplete" : ""
  const validRolesClass = !Boolean(roles.length)
    ? "form__input--incomplete"
    : ""

  const errContent = (error?.data.message || delerror?.data.message) ?? ""

  const content = (
    <>
      <p className={errClass}>{errContent}</p>

      <form className="form" onSubmit={e => e.preventDefault()}>
        <div className="form__title-row">
          <h2>Edit User</h2>
          <div className="form__action-buttons">
            <button
              className="icon-button"
              title="Save"
              onClick={onSaveUserClicked}
              disabled={!canSave}
            >
              <FontAwesomeIcon icon={faSave} />
            </button>
            <button
              className="icon-button"
              title="Delete"
              onClick={onDeleteUserClicked}
            >
              <FontAwesomeIcon icon={faTrashCan} />
            </button>
          </div>
        </div>
        <label className="form__label" htmlFor="username">
          Username: <span className="nowrap">[3-20 letters]</span>
        </label>
        <input
          className={`form__input ${validUserClass}`}
          id="username"
          name="username"
          type="text"
          autoComplete="off"
          value={username}
          onChange={onUsernameChanged}
        />

        <label className="form__label" htmlFor="password">
          Password: <span className="nowrap">[4-12 chars incl. !@#$%]</span>
        </label>
        <input
          className={`form__input ${validPwdClass}`}
          id="password"
          name="password"
          type="password"
          value={password}
          onChange={onPasswordChanged}
        />

        <label htmlFor="user-active" className="form__label">
          ACTIVE:
        </label>
        <input
          type="checkbox"
          name="user-active"
          id="user-active"
          className="form__checkbox"
          checked={active}
          onChange={onActiveChanged}
        />

        <label className="form__label" htmlFor="roles">
          ASSIGNED ROLES:
        </label>
        <select
          id="roles"
          name="roles"
          className={`form__select ${validRolesClass}`}
          multiple={true}
          size="3"
          value={roles}
          onChange={onRolesChanged}
        >
          {options}
        </select>
      </form>
    </>
  )

  return content
}

export default EditUserForm
```

## 新增 Note

### NewNote.jsx

```js
// features/notes/NewNote.jsx
import SyncLoader from "react-spinners/SyncLoader"

import { useGetUsersQuery } from "../users/usersApiSlice"
import NewNoteForm from "./NewNoteForm"

const NewNote = () => {
  const { users } = useGetUsersQuery("usersList", {
    selectFromResult: ({ data }) => ({
      users: data?.ids.map(id => data?.entities[id]),
    }),
  })

  if (!users?.length) {
    return (
      <div className="flex">
        <SyncLoader color="#d9d9d9" />
      </div>
    )
  }

  const content = <NewNoteForm users={users} />

  return content
}

export default NewNote
```

### NewNoteForm.jsx

```js
// features/notes/NewNoteForm.jsx
import { useState, useEffect } from "react"
import { useNavigate } from "react-router-dom"
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faSave } from "@fortawesome/free-solid-svg-icons"

import { useAddNewNoteMutation } from "./notesApiSlice"

const NewNoteForm = ({ users }) => {
  const [addNewNote, { isLoading, isSuccess, isError, error }] =
    useAddNewNoteMutation()

  const [title, setTitle] = useState("")
  const [text, setText] = useState("")
  const [userId, setUserId] = useState(users[0].id)

  const navigate = useNavigate()

  useEffect(() => {
    if (isSuccess) {
      setTitle("")
      setText("")
      setUserId("")
      navigate("/dash/notes")
    }
  }, [isSuccess, navigate])

  const onTitleChanged = e => setTitle(e.target.value)
  const onTextChanged = e => setText(e.target.value)
  const onUserIdChanged = e => setUserId(e.target.value)

  const canSave = [title, text, userId].every(Boolean) && !isLoading

  const onSaveNoteClicked = async () => {
    if (canSave) {
      await addNewNote({ title, text, user: userId })
    }
  }

  const usersOptions = users
    .filter(user => user.active)
    .map(user => (
      <option key={user.id} value={user.id}>
        {user.username}
      </option>
    ))

  const errClass = isError ? "errmsg" : "offscreen"
  const validTitleClass = !title ? "form__input--incomplete" : ""
  const validTextClass = !text ? "form__input--incomplete" : ""

  const errContent = error?.data.message ?? ""

  const content = (
    <>
      <p className={errClass}>{errContent}</p>

      <form className="form" onSubmit={e => e.preventDefault()}>
        <div className="form__title-row">
          <h2>New Note</h2>
          <div className="form__action-buttons">
            <button
              className="icon-button"
              title="Save"
              onClick={onSaveNoteClicked}
              disabled={!canSave}
            >
              <FontAwesomeIcon icon={faSave} />
            </button>
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
      </form>
    </>
  )
  return content
}

export default NewNoteForm
```

## 編輯 Note

### EditNote.jsx

```js
// features/notes/EditNote.jsx
import { useParams } from "react-router-dom"
import SyncLoader from "react-spinners/SyncLoader"

import { useGetNotesQuery } from "./notesApiSlice"
import { useGetUsersQuery } from "../users/usersApiSlice"
import EditNoteForm from "./EditNoteForm"

const EditNote = () => {
  const { id } = useParams()

  const { note } = useGetNotesQuery("notesList", {
    selectFromResult: ({ data }) => ({
      note: data?.entities[id],
    }),
  })

  const { users } = useGetUsersQuery("usersList", {
    selectFromResult: ({ data }) => ({
      users: data?.ids.map(id => data?.entities[id]),
    }),
  })

  if (!note || !users.length) {
    return (
      <div className="flex">
        <SyncLoader color="#d9d9d9" />
      </div>
    )
  }

  const content = <EditNoteForm note={note} users={users} />

  return content
}

export default EditNote
```

### EditNoteForm.jsx

```js
// features/notes/EditNoteForm.jsx
import { useState, useEffect } from "react"
import { useNavigate } from "react-router-dom"
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faSave, faTrashCan } from "@fortawesome/free-solid-svg-icons"

import { useUpdateNoteMutation, useDeleteNoteMutation } from "./notesApiSlice"

const EditNoteForm = ({ note, users }) => {
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
            <button
              className="icon-button"
              title="Delete"
              onClick={onDeleteNoteClicked}
            >
              <FontAwesomeIcon icon={faTrashCan} />
            </button>
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

## App.jsx

設定 route :

```js
// App.jsx
import { Routes, Route } from "react-router-dom"

import "./App.css"
import Layout from "./components/Layout"
import Public from "./components/Public"
import Login from "./features/auth/Login"
import Prefetch from "./features/auth/Prefetch"
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
    </Routes>
  )
}

export default App
```
