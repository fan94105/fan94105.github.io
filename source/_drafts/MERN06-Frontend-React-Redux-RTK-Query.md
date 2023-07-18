---
title: [MERN06]Frontend - React Redux & RTK Query
tags:
  - React
  - Redux
  - RTK Query
categories:
  - MERN
  - Frontend
---

使用 Redux 管理 App 中的數據，藉由 RTK Query 向後端 API 發送請求獲取數據並進行快取。

<!-- more -->

> Code from [Dave Gray - MERN Stack Full Tutorial & Project](https://www.youtube.com/watch?v=CvCiNeLnZ00&ab_channel=DaveGray)

# React Redux & RTK Query

安裝 React Redux 和 Redux Toolkit :

```
npm i @reduxjs/toolkit react-redux
```

## apiSlice.js

建立 apiSlice.js ，集中管理向後端 API 發送請求的方法 :

```js
// app/api/apiSlice.js
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react"

export const apiSlice = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: "http://localhost:3500" }),
  tagTypes: ["Note", "User"],
  endpoints: builder => ({}),
})
```

## store.js

建立 store 並定義 reducer :

```js
// app/store.js
import { configureStore } from "@reduxjs/toolkit"
import { setupListeners } from "@reduxjs/toolkit/query"

import { apiSlice } from "./api/apiSlice"

export const store = configureStore({
  reducer: {
    [apiSlice.reducerPath]: apiSlice.reducer,
  },
  middleware: getDefaultMiddleware =>
    getDefaultMiddleware().concat(apiSlice.middleware),
  devTools: true,
})

// 啟用 refetchOnFocus 與 refetchOnReconnect
setupListeners(store.dispatch)
```

將 store 提供給 App :

```js
// main.jsx
import React from "react"
import ReactDOM from "react-dom/client"
import { BrowserRouter as Router, Routes, Route } from "react-router-dom"

import { Provider } from "react-redux"
import { store } from "./app/store"

import App from "./App.jsx"
import "./index.css"

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    {/* 向 App 提供 store */}
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

## usersApiSlice.js & notesApiSlice.js

usersApiSlice.js，處理與 user 相關的請求。

```js
// features/users/usersApiSlice.js
import { createSelector, createEntityAdapter } from "@reduxjs/toolkit"

import { apiSlice } from "../../app/api/apiSlice"

// 提供數據標準化方法
const usersAdapter = createEntityAdapter({})

// 生成空的 {ids: [], entities: {}} 物件
const initialState = usersAdapter.getInitialState()

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
  }),
})

export const { useGetUsersQuery } = usersApiSlice

// 生成新的 selector，並回傳 query 結果物件
export const selectUsersResult = usersApiSlice.endpoints.getUsers.select()

// 創建 memoized selector
const selectUsersData = createSelector(
  selectUsersResult,
  usersResult => usersResult.data
)

// 生成 selector
export const {
  selectAll: selectAllUsers,
  selectById: selectUserById,
  selectIds: selectUserIds,
} = usersAdapter.getSelectors(state => selectUsersData(state) ?? initialState)
```

notesApiSlice.js，處理與 note 相關的請求。程式碼與 usersApiSlice.js 相同，將 user 改為 note 即可。

```js
// features/notes/notesApiSlice.js
import { createSelector, createEntityAdapter } from "@reduxjs/toolkit"

import { apiSlice } from "../../app/api/apiSlice"

// 提供數據標準化方法
const notesAdapter = createEntityAdapter({
  // 數據排序
  sortComparer: (a, b) =>
    a.completed === b.completed ? 0 : a.completed ? 1 : -1,
})

// 生成空的 {ids: [], entities: {}} 物件
const initialState = notesAdapter.getInitialState()

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
  }),
})

export const { useGetNotesQuery } = notesApiSlice

// 生成新的 selector，並回傳 query 結果物件
export const selectNotesResult = notesApiSlice.endpoints.getNotes.select()

// 創建 memoized selector
const selectNotesData = createSelector(
  selectNotesResult,
  notesResult => notesResult.data
)

// 生成 selector
export const {
  selectAll: selectAllNotes,
  selectById: selectNoteById,
  selectIds: selectNoteIds,
} = notesAdapter.getSelectors(state => selectNotesData(state) ?? initialState)
```

### 內容說明

1. 調用 `createEntityAdapter` 為數據提供標準化方法。
2. 由 `usersAdapter.getInitialState()` 生成空的 `{ids: [], entities:{}}` 物件。
3. 由 `apiSlice.injectEndpoints()` 創建 `usersApiSlice` 並設定 endpoints。
   - `transformResponse` 中設定 `usersAdapter` 數據。
4. 為了創建 memoized selector，以 `usersApiSlice.endpoints.getNotes.select()` 生成新的 selector，並回傳 query 結果物件。
5. `createSelector` 創建 memoized selector。
6. `usersAdapter.getSelector()` 取得預設的 selector。

# 使用 RTK Query

安裝 react-spinners :

```
npm i react-spinners
```

## 展示 Users

### UsersList.jsx

```js
// features/users/UsersList.jsx
import SyncLoader from "react-spinners/SyncLoader"

import { useGetUsersQuery } from "./usersApiSlice"
import User from "./User"

const UsersList = () => {
  const {
    data: users,
    isLoading,
    isSuccess,
    isError,
    error,
  } = useGetUsersQuery("usersList", {
    pollingInterval: 60000, // 輪詢間隔(ms)
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
    const { ids } = users

    const tableContent = ids?.length
      ? ids.map(userId => <User key={userId} userId={userId} />)
      : null

    content = (
      <table className="table">
        <thead className="table__thead">
          <tr>
            <th className="table__th" scope="col">
              Username
            </th>
            <th className="table__th" scope="col">
              Roles
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

export default UsersList
```

### User.jsx

```js
// features/users/User.jsx
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faPenToSquare } from "@fortawesome/free-solid-svg-icons"
import { useNavigate } from "react-router-dom"

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
  }
}

export default User
```

## 展示 Notes

### NotesList.jsx

```js
// features/notes/NotesList.jsx
import SyncLoader from "react-spinners/SyncLoader"

import { useGetNotesQuery } from "./notesApiSlice"
import Note from "./Note"

const NotesList = () => {
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
    const { ids } = notes

    const tableContent = ids?.length
      ? ids.map(noteId => <Note key={noteId} noteId={noteId} />)
      : null

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
  } else {
    return null
  }

  return content
}

export default NotesList
```

### Note.jsx

```js
// features/notes/Note.jsx
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome"
import { faPenToSquare } from "@fortawesome/free-solid-svg-icons"
import { useNavigate } from "react-router-dom"

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

export default Note
```
