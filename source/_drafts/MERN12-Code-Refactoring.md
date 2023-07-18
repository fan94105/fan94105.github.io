---
title: [MERN12]Code Refactoring
tags:
  - React
  - Redux
  - RTK Query
categories:
  - [MERN, Frontend]
  - [MERN, Backend]
---

重構程式碼。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# Backend

## errorHandler.js

為了讓 RTK Query 更好的處理請求中未預期的錯誤，如伺服器端錯誤，可以在響應體中回傳 `isError`。

```js
// middleware/errorHandler.js
const { logEvent } = require("./logger")

const errorHandler = (err, req, res, next) => {
  logEvent(
    `${err.name}: ${err.message}\t${req.method}\t${req.url}\t${req.headers.origin}`,
    "errLog.log"
  )

  const status = res.statusCode ? res.statusCode : 500 // server error

  res.status(status)

  // 為 RTK Query 提供 isError
  res.json({ message: err.message, isError: true })
}

module.exports = errorHandler
```

# Frontend

## Note.jsx

將 Note 組件改為 memorized component，使其不會因為父組件的變動而重新渲染，只會因自身組件的數據改變而重新渲染。

```js
// features/notes/Note.jsx
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faPenToSquare } from "@fortawesome/free-solid-svg-icons"
import { useNavigate } from "react-router-dom"
import { memo } from "react"

import { useGetNotesQuery } from "./notesApiSlice"

const Note = ({ noteId }) => {
  const { note } = useGetNotesQuery("notesList", {
    selectFromResult: ({ data }) => ({
      note: data?.entities[noteId],
    }),
  })

  const navigate = useNavigate()

  if (note) {
    const created = new Date(note.createdAt).toLocaleString("zh-TW", {
      day: "numeric",
      month: "long",
    })
    const updated = new Date(note.updatedAt).toLocaleString("zh-TW", {
      day: "numeric",
      month: "long",
    })

    const handleEdit = () => navigate(`/dash/notes/${noteId}`)
    return (
      <tr className="table__row">
        <td className="table__cell note__status">
          {note.completed ? (
            <span className="note__status--completed">Completed</span>
          ) : (
            <span className="note__status--open">Open</span>
          )}
        </td>
        <td className="table__cell">{created}</td>
        <td className="table__cell">{updated}</td>
        <td className="table__cell">{note.title}</td>
        <td className="table__cell">{note.username}</td>
        <td className="table__cell">
          <button className="icon-button" onClick={handleEdit}>
            <FontAwesomeIcon icon={faPenToSquare} />
          </button>
        </td>
      </tr>
    )
  } else {
    return null
  }
}

const memoizedNote = memo(Note)

export default memoizedNote
```

## User.jsx

與 Note.jsx 相同，將組件改為 memorized component。

```js
// features/users/User.jsx
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faPenToSquare } from "@fortawesome/free-solid-svg-icons"
import { useNavigate } from "react-router-dom"
import { memo } from "react"

import { useGetUsersQuery } from "./usersApiSlice"

const User = ({ userId }) => {
  const { user } = useGetUsersQuery("usersList", {
    selectFromResult: ({ data }) => ({
      user: data?.entities[userId],
    }),
  })

  const navigate = useNavigate()

  if (user) {
    const handleEdit = () => navigate(`/dash/users/${userId}`)

    const userRolesString = user.roles.toString().replaceAll(",", ", ")

    const cellStatus = user.active ? "" : "table__cell--inactive"

    return (
      <tr className="table__row">
        <td className={`table__cell ${cellStatus}`}>{user.username}</td>
        <td className={`table__cell ${cellStatus}`}>{userRolesString}</td>
        <td className={`table__cell ${cellStatus}`}>
          <button className="icon-button" onClick={handleEdit}>
            <FontAwesomeIcon icon={faPenToSquare} />
          </button>
        </td>
      </tr>
    )
  } else {
    return null
  }
}

const memoizedUser = memo(User)

export default memoizedUser
```

## EditNote.jsx

增加編輯 Note 的權限限制。

```js
// features/notes/EditNote.jsx
import { useParams } from "react-router-dom"
import SyncLoader from "react-spinners/SyncLoader"

import { useGetNotesQuery } from "./notesApiSlice"
import { useGetUsersQuery } from "../users/usersApiSlice"
import EditNoteForm from "./EditNoteForm"
import useAuth from "../../hooks/useAuth"

const EditNote = () => {
  const { id } = useParams()

  const { username, isManager, isAdmin } = useAuth()

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

  if (!isManager && !isAdmin) {
    if (note.username !== username) {
      return <p className="errmsg">No access</p>
    }
  }

  const content = <EditNoteForm note={note} users={users} />

  return content
}

export default EditNote
```

## useTitle.js

改變網頁 title 的 hooks。

```js
// hooks/useTitle.js
import { useEffect } from "react"

const useTitle = title => {
  useEffect(() => {
    const prevTitle = document.title
    document.title = title

    return () => (document.title = prevTitle)
  }, [title])
}

export default useTitle
```
