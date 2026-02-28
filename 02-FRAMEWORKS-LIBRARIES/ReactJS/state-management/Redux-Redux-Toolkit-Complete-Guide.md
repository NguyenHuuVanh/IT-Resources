# Redux & Redux Toolkit - Hướng Dẫn Toàn Diện

## Mục Lục

1. [Redux Core Concepts](#redux-core-concepts)
2. [Redux Toolkit (RTK)](#redux-toolkit-rtk)
3. [Redux Saga](#redux-saga)
4. [Redux Thunk](#redux-thunk)
5. [So Sánh & Best Practices](#so-sánh--best-practices)

---

# Redux Core Concepts

## Giới Thiệu

**Redux** là thư viện quản lý state cho JavaScript apps, đặc biệt phổ biến với React.

### Tại Sao Cần Redux?

```javascript
// ❌ Prop Drilling Problem
function App() {
  const [user, setUser] = useState(null);
  return <Dashboard user={user} setUser={setUser} />;
}

function Dashboard({ user, setUser }) {
  return <Sidebar user={user} setUser={setUser} />;
}

function Sidebar({ user, setUser }) {
  return <UserProfile user={user} setUser={setUser} />;
}

function UserProfile({ user, setUser }) {
  return <div>{user.name}</div>;
}

// ✅ Redux Solution
function UserProfile() {
  const user = useSelector((state) => state.user);
  const dispatch = useDispatch();

  return <div>{user.name}</div>;
}
```

### 3 Principles of Redux

1. **Single Source of Truth**: Toàn bộ state trong một store
2. **State is Read-Only**: Chỉ thay đổi qua actions
3. **Changes are Made with Pure Functions**: Reducers là pure functions

---

## Core Components

### 1. Store

```javascript
import { createStore } from "redux";

// Store là nơi lưu trữ toàn bộ state
const store = createStore(rootReducer);

// Methods
store.getState(); // Lấy state hiện tại
store.dispatch(action); // Dispatch action
store.subscribe(listener); // Subscribe to changes
```

### 2. Actions

```javascript
// Action là plain object mô tả "what happened"
const action = {
  type: "ADD_TODO",
  payload: {
    id: 1,
    text: "Learn Redux",
    completed: false,
  },
};

// Action Creators
function addTodo(text) {
  return {
    type: "ADD_TODO",
    payload: {
      id: Date.now(),
      text,
      completed: false,
    },
  };
}

// Action Types (constants)
const ADD_TODO = "ADD_TODO";
const TOGGLE_TODO = "TOGGLE_TODO";
const DELETE_TODO = "DELETE_TODO";
```

### 3. Reducers

```javascript
// Reducer là pure function: (state, action) => newState
const initialState = {
  todos: [],
};

function todoReducer(state = initialState, action) {
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...state,
        todos: [...state.todos, action.payload],
      };

    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload.id ? { ...todo, completed: !todo.completed } : todo,
        ),
      };

    case "DELETE_TODO":
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload.id),
      };

    default:
      return state;
  }
}
```

### 4. Combine Reducers

```javascript
import { combineReducers } from "redux";

// User Reducer
const userReducer = (state = { user: null }, action) => {
  switch (action.type) {
    case "SET_USER":
      return { ...state, user: action.payload };
    case "LOGOUT":
      return { ...state, user: null };
    default:
      return state;
  }
};

// Todo Reducer
const todoReducer = (state = { todos: [] }, action) => {
  // ... todo logic
  return state;
};

// Combine
const rootReducer = combineReducers({
  user: userReducer,
  todos: todoReducer,
});

// State shape:
// {
//   user: { user: null },
//   todos: { todos: [] }
// }
```

---

## React Redux Integration

### Setup

```bash
npm install redux react-redux
```

```javascript
// store.js
import { createStore } from "redux";
import { Provider } from "react-redux";
import rootReducer from "./reducers";

const store = createStore(rootReducer);

// App.js
import { Provider } from "react-redux";
import store from "./store";

function App() {
  return (
    <Provider store={store}>
      <YourApp />
    </Provider>
  );
}
```

### useSelector & useDispatch

```javascript
import { useSelector, useDispatch } from "react-redux";

function TodoList() {
  // Select state
  const todos = useSelector((state) => state.todos.todos);
  const user = useSelector((state) => state.user.user);

  // Get dispatch
  const dispatch = useDispatch();

  const handleAddTodo = (text) => {
    dispatch({
      type: "ADD_TODO",
      payload: { id: Date.now(), text, completed: false },
    });
  };

  const handleToggle = (id) => {
    dispatch({ type: "TOGGLE_TODO", payload: { id } });
  };

  return (
    <div>
      {todos.map((todo) => (
        <div key={todo.id}>
          <input type="checkbox" checked={todo.completed} onChange={() => handleToggle(todo.id)} />
          {todo.text}
        </div>
      ))}
    </div>
  );
}
```

### Selector Optimization

```javascript
import { createSelector } from "reselect";

// ❌ BAD: Tạo array mới mỗi lần render
function TodoList() {
  const completedTodos = useSelector((state) => state.todos.filter((todo) => todo.completed));
  // Component re-render mỗi lần state thay đổi!
}

// ✅ GOOD: Memoized selector
const selectCompletedTodos = createSelector([(state) => state.todos], (todos) =>
  todos.filter((todo) => todo.completed),
);

function TodoList() {
  const completedTodos = useSelector(selectCompletedTodos);
  // Chỉ re-render khi completedTodos thực sự thay đổi
}
```

---

# Redux Toolkit (RTK)

## Giới Thiệu

**Redux Toolkit** là official toolset của Redux, giúp viết Redux code dễ dàng hơn với ít boilerplate.

### Tại Sao Dùng Redux Toolkit?

```javascript
// ❌ Traditional Redux (verbose)
// Action Types
const ADD_TODO = "ADD_TODO";
const TOGGLE_TODO = "TOGGLE_TODO";

// Action Creators
const addTodo = (text) => ({
  type: ADD_TODO,
  payload: { id: Date.now(), text, completed: false },
});

// Reducer
const todoReducer = (state = { todos: [] }, action) => {
  switch (action.type) {
    case ADD_TODO:
      return { ...state, todos: [...state.todos, action.payload] };
    case TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload.id ? { ...todo, completed: !todo.completed } : todo,
        ),
      };
    default:
      return state;
  }
};

// ✅ Redux Toolkit (concise)
import { createSlice } from "@reduxjs/toolkit";

const todoSlice = createSlice({
  name: "todos",
  initialState: { todos: [] },
  reducers: {
    addTodo: (state, action) => {
      state.todos.push(action.payload); // Immer allows "mutation"
    },
    toggleTodo: (state, action) => {
      const todo = state.todos.find((t) => t.id === action.payload.id);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
  },
});

export const { addTodo, toggleTodo } = todoSlice.actions;
export default todoSlice.reducer;
```

### Lợi Ích

✅ **Less Boilerplate**: Ít code hơn
✅ **Immer Integration**: "Mutate" state safely
✅ **DevTools**: Redux DevTools tích hợp sẵn
✅ **TypeScript**: Type-safe
✅ **Best Practices**: Built-in best practices

---

## Setup

```bash
npm install @reduxjs/toolkit react-redux
```

```javascript
// store.js
import { configureStore } from "@reduxjs/toolkit";
import todoReducer from "./features/todoSlice";
import userReducer from "./features/userSlice";

const store = configureStore({
  reducer: {
    todos: todoReducer,
    user: userReducer,
  },
  // DevTools, middleware tự động configure
});

export default store;

// App.js
import { Provider } from "react-redux";
import store from "./store";

function App() {
  return (
    <Provider store={store}>
      <YourApp />
    </Provider>
  );
}
```

---

## createSlice

### Basic Usage

```javascript
// features/todoSlice.js
import { createSlice } from "@reduxjs/toolkit";

const todoSlice = createSlice({
  name: "todos",
  initialState: {
    todos: [],
    filter: "all", // 'all' | 'active' | 'completed'
  },
  reducers: {
    addTodo: (state, action) => {
      state.todos.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
      });
    },

    toggleTodo: (state, action) => {
      const todo = state.todos.find((t) => t.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },

    deleteTodo: (state, action) => {
      state.todos = state.todos.filter((t) => t.id !== action.payload);
    },

    setFilter: (state, action) => {
      state.filter = action.payload;
    },

    clearCompleted: (state) => {
      state.todos = state.todos.filter((t) => !t.completed);
    },
  },
});

export const { addTodo, toggleTodo, deleteTodo, setFilter, clearCompleted } = todoSlice.actions;

export default todoSlice.reducer;
```

### Prepare Callback

```javascript
const todoSlice = createSlice({
  name: "todos",
  initialState: { todos: [] },
  reducers: {
    addTodo: {
      reducer: (state, action) => {
        state.todos.push(action.payload);
      },
      prepare: (text) => {
        return {
          payload: {
            id: Date.now(),
            text,
            completed: false,
            createdAt: new Date().toISOString(),
          },
        };
      },
    },
  },
});

// Usage
dispatch(addTodo("Learn Redux")); // Chỉ cần pass text
```

---

## createAsyncThunk

### Basic Usage

```javascript
// features/userSlice.js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

// Async thunk
export const fetchUser = createAsyncThunk("user/fetchUser", async (userId, thunkAPI) => {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) {
    throw new Error("Failed to fetch user");
  }
  return response.json();
});

const userSlice = createSlice({
  name: "user",
  initialState: {
    user: null,
    loading: false,
    error: null,
  },
  reducers: {
    logout: (state) => {
      state.user = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export const { logout } = userSlice.actions;
export default userSlice.reducer;
```

### Usage in Component

```javascript
import { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import { fetchUser } from "./features/userSlice";

function UserProfile({ userId }) {
  const dispatch = useDispatch();
  const { user, loading, error } = useSelector((state) => state.user);

  useEffect(() => {
    dispatch(fetchUser(userId));
  }, [dispatch, userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Advanced createAsyncThunk

```javascript
// With error handling
export const createUser = createAsyncThunk("user/createUser", async (userData, { rejectWithValue }) => {
  try {
    const response = await fetch("/api/users", {
      method: "POST",
      body: JSON.stringify(userData),
      headers: { "Content-Type": "application/json" },
    });

    if (!response.ok) {
      const error = await response.json();
      return rejectWithValue(error.message);
    }

    return response.json();
  } catch (error) {
    return rejectWithValue(error.message);
  }
});

// With thunkAPI
export const fetchUserPosts = createAsyncThunk(
  "posts/fetchUserPosts",
  async (userId, { getState, dispatch, rejectWithValue }) => {
    // Access state
    const state = getState();
    const token = state.auth.token;

    // Dispatch other actions
    dispatch(setLoading(true));

    try {
      const response = await fetch(`/api/users/${userId}/posts`, {
        headers: { Authorization: `Bearer ${token}` },
      });
      return response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  },
);
```

---

## RTK Query

### Setup

```javascript
// services/api.js
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";

export const api = createApi({
  reducerPath: "api",
  baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
  tagTypes: ["User", "Post"],
  endpoints: (builder) => ({
    // Get users
    getUsers: builder.query({
      query: () => "/users",
      providesTags: ["User"],
    }),

    // Get user by ID
    getUser: builder.query({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: "User", id }],
    }),

    // Create user
    createUser: builder.mutation({
      query: (user) => ({
        url: "/users",
        method: "POST",
        body: user,
      }),
      invalidatesTags: ["User"],
    }),

    // Update user
    updateUser: builder.mutation({
      query: ({ id, ...user }) => ({
        url: `/users/${id}`,
        method: "PUT",
        body: user,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: "User", id }],
    }),

    // Delete user
    deleteUser: builder.mutation({
      query: (id) => ({
        url: `/users/${id}`,
        method: "DELETE",
      }),
      invalidatesTags: ["User"],
    }),
  }),
});

export const {
  useGetUsersQuery,
  useGetUserQuery,
  useCreateUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation,
} = api;
```

### Configure Store

```javascript
// store.js
import { configureStore } from "@reduxjs/toolkit";
import { api } from "./services/api";

const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer,
    // other reducers
  },
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(api.middleware),
});

export default store;
```

### Usage in Components

```javascript
import { useGetUsersQuery, useCreateUserMutation, useUpdateUserMutation, useDeleteUserMutation } from "./services/api";

function UserList() {
  // Query
  const { data: users, isLoading, error, refetch } = useGetUsersQuery();

  // Mutations
  const [createUser, { isLoading: isCreating }] = useCreateUserMutation();
  const [updateUser] = useUpdateUserMutation();
  const [deleteUser] = useDeleteUserMutation();

  const handleCreate = async () => {
    try {
      await createUser({ name: "John", email: "john@example.com" }).unwrap();
      alert("User created!");
    } catch (error) {
      alert("Error: " + error.message);
    }
  };

  const handleUpdate = async (id) => {
    await updateUser({ id, name: "Jane" });
  };

  const handleDelete = async (id) => {
    await deleteUser(id);
  };

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <button onClick={handleCreate} disabled={isCreating}>
        Create User
      </button>

      {users.map((user) => (
        <div key={user.id}>
          <span>{user.name}</span>
          <button onClick={() => handleUpdate(user.id)}>Edit</button>
          <button onClick={() => handleDelete(user.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}
```

### Polling & Refetching

```javascript
function UserList() {
  const { data, refetch } = useGetUsersQuery(undefined, {
    pollingInterval: 5000, // Poll every 5 seconds
    refetchOnMountOrArgChange: true,
    refetchOnFocus: true,
    refetchOnReconnect: true,
  });

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      {/* ... */}
    </div>
  );
}
```

---

# Redux Saga

## Giới Thiệu

**Redux Saga** là middleware sử dụng ES6 Generators để quản lý side effects phức tạp.

### Tại Sao Dùng Redux Saga?

```javascript
// ❌ Redux Thunk (callback hell với logic phức tạp)
const fetchUserWithPosts = (userId) => async (dispatch) => {
  dispatch({ type: "FETCH_START" });

  try {
    const user = await fetch(`/api/users/${userId}`).then((r) => r.json());
    dispatch({ type: "USER_SUCCESS", payload: user });

    const posts = await fetch(`/api/users/${userId}/posts`).then((r) => r.json());
    dispatch({ type: "POSTS_SUCCESS", payload: posts });

    const comments = await fetch(`/api/posts/${posts[0].id}/comments`).then((r) => r.json());
    dispatch({ type: "COMMENTS_SUCCESS", payload: comments });
  } catch (error) {
    dispatch({ type: "FETCH_ERROR", payload: error.message });
  }
};

// ✅ Redux Saga (clean, testable)
function* fetchUserWithPostsSaga(action) {
  try {
    yield put({ type: "FETCH_START" });

    const user = yield call(api.fetchUser, action.payload.userId);
    yield put({ type: "USER_SUCCESS", payload: user });

    const posts = yield call(api.fetchUserPosts, action.payload.userId);
    yield put({ type: "POSTS_SUCCESS", payload: posts });

    const comments = yield call(api.fetchComments, posts[0].id);
    yield put({ type: "COMMENTS_SUCCESS", payload: comments });
  } catch (error) {
    yield put({ type: "FETCH_ERROR", payload: error.message });
  }
}
```

---

## Setup

```bash
npm install redux-saga
```

```javascript
// sagas/index.js
import { all } from 'redux-saga/effects';
import userSaga from './userSaga';
import todoSaga from './todoSaga';

export default function* rootSaga() {
  yield all([
    userSaga(),
    todoSaga()
  ]);
}

// store.js
import { configureStore } from '@reduxjs/toolkit';
import createSagaMiddleware from 'redux-saga';
import rootReducer from './reducers';
import rootSaga from './sagas';

const sagaMiddleware = createSagaMiddleware();

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({ thunk: false }).concat(sagaMiddleware)
});

sagaMiddleware.run(rootSaga);

export default store;
```

---

## Saga Effects

### call - Call Functions

```javascript
import { call, put } from "redux-saga/effects";

function* fetchUserSaga(action) {
  try {
    // call(fn, ...args) - Call function and wait for result
    const user = yield call(fetch, `/api/users/${action.payload}`);
    const data = yield call([user, "json"]); // user.json()

    yield put({ type: "FETCH_USER_SUCCESS", payload: data });
  } catch (error) {
    yield put({ type: "FETCH_USER_ERROR", payload: error.message });
  }
}
```

### put - Dispatch Actions

```javascript
import { put } from "redux-saga/effects";

function* loginSaga(action) {
  const { username, password } = action.payload;

  try {
    const user = yield call(api.login, username, password);

    // put = dispatch
    yield put({ type: "LOGIN_SUCCESS", payload: user });
    yield put({ type: "FETCH_USER_DATA", payload: user.id });
  } catch (error) {
    yield put({ type: "LOGIN_ERROR", payload: error.message });
  }
}
```

### takeEvery - Listen to Every Action

```javascript
import { takeEvery, call, put } from "redux-saga/effects";

function* fetchUserSaga(action) {
  const user = yield call(api.fetchUser, action.payload);
  yield put({ type: "FETCH_USER_SUCCESS", payload: user });
}

function* watchFetchUser() {
  // Listen to every FETCH_USER action
  yield takeEvery("FETCH_USER", fetchUserSaga);
}
```

### takeLatest - Cancel Previous, Take Latest

```javascript
import { takeLatest } from "redux-saga/effects";

function* searchSaga(action) {
  const results = yield call(api.search, action.payload);
  yield put({ type: "SEARCH_SUCCESS", payload: results });
}

function* watchSearch() {
  // Cancel previous search if new one comes
  yield takeLatest("SEARCH", searchSaga);
}
```

### takeLeading - Take First, Ignore Others

```javascript
import { takeLeading } from "redux-saga/effects";

function* submitFormSaga(action) {
  yield call(api.submitForm, action.payload);
  yield put({ type: "SUBMIT_SUCCESS" });
}

function* watchSubmit() {
  // Ignore subsequent submits until first completes
  yield takeLeading("SUBMIT_FORM", submitFormSaga);
}
```

### select - Get State

```javascript
import { select, call, put } from "redux-saga/effects";

function* fetchUserPostsSaga(action) {
  // Get state
  const userId = yield select((state) => state.user.id);
  const token = yield select((state) => state.auth.token);

  const posts = yield call(api.fetchPosts, userId, token);
  yield put({ type: "FETCH_POSTS_SUCCESS", payload: posts });
}
```

### fork & spawn - Non-blocking Calls

```javascript
import { fork, spawn, call } from "redux-saga/effects";

function* backgroundTask() {
  while (true) {
    yield call(delay, 5000);
    yield call(api.syncData);
  }
}

function* mainSaga() {
  // fork: attached to parent (cancelled when parent cancelled)
  yield fork(backgroundTask);

  // spawn: detached from parent (continues even if parent cancelled)
  yield spawn(backgroundTask);
}
```

### race - Race Conditions

```javascript
import { race, call, put, delay } from "redux-saga/effects";

function* fetchWithTimeoutSaga(action) {
  const { response, timeout } = yield race({
    response: call(api.fetchUser, action.payload),
    timeout: delay(5000),
  });

  if (response) {
    yield put({ type: "FETCH_SUCCESS", payload: response });
  } else {
    yield put({ type: "FETCH_TIMEOUT" });
  }
}
```

### all - Parallel Execution

```javascript
import { all, call, put } from "redux-saga/effects";

function* fetchAllDataSaga() {
  try {
    // Fetch in parallel
    const [users, posts, comments] = yield all([call(api.fetchUsers), call(api.fetchPosts), call(api.fetchComments)]);

    yield put({ type: "FETCH_ALL_SUCCESS", payload: { users, posts, comments } });
  } catch (error) {
    yield put({ type: "FETCH_ALL_ERROR", payload: error.message });
  }
}
```

---

## Real-World Examples

### Login Flow

```javascript
// sagas/authSaga.js
import { call, put, takeLatest } from "redux-saga/effects";
import * as api from "../api";

function* loginSaga(action) {
  try {
    yield put({ type: "LOGIN_START" });

    const { username, password } = action.payload;
    const response = yield call(api.login, username, password);

    // Save token
    localStorage.setItem("token", response.token);

    yield put({ type: "LOGIN_SUCCESS", payload: response.user });

    // Fetch user data
    yield put({ type: "FETCH_USER_DATA" });
  } catch (error) {
    yield put({ type: "LOGIN_ERROR", payload: error.message });
  }
}

function* logoutSaga() {
  try {
    yield call(api.logout);
    localStorage.removeItem("token");
    yield put({ type: "LOGOUT_SUCCESS" });
  } catch (error) {
    yield put({ type: "LOGOUT_ERROR", payload: error.message });
  }
}

export default function* authSaga() {
  yield takeLatest("LOGIN_REQUEST", loginSaga);
  yield takeLatest("LOGOUT_REQUEST", logoutSaga);
}
```

### Polling

```javascript
import { call, put, delay, race, take } from "redux-saga/effects";

function* pollDataSaga() {
  while (true) {
    try {
      const data = yield call(api.fetchData);
      yield put({ type: "POLL_SUCCESS", payload: data });

      // Wait 5 seconds or until STOP_POLLING action
      const { stopped } = yield race({
        stopped: take("STOP_POLLING"),
        timeout: delay(5000),
      });

      if (stopped) break;
    } catch (error) {
      yield put({ type: "POLL_ERROR", payload: error.message });
      yield delay(5000);
    }
  }
}

export default function* pollingSaga() {
  yield takeLatest("START_POLLING", pollDataSaga);
}
```

### Debounce Search

```javascript
import { call, put, debounce } from "redux-saga/effects";

function* searchSaga(action) {
  try {
    const results = yield call(api.search, action.payload);
    yield put({ type: "SEARCH_SUCCESS", payload: results });
  } catch (error) {
    yield put({ type: "SEARCH_ERROR", payload: error.message });
  }
}

export default function* searchWatcherSaga() {
  // Debounce search by 500ms
  yield debounce(500, "SEARCH_REQUEST", searchSaga);
}
```

---

# Redux Thunk

## Giới Thiệu

**Redux Thunk** là middleware đơn giản cho phép dispatch functions thay vì actions.

```bash
npm install redux-thunk
```

## Basic Usage

```javascript
// Action creator returns function
const fetchUser = (userId) => {
  return async (dispatch, getState) => {
    dispatch({ type: "FETCH_USER_START" });

    try {
      const response = await fetch(`/api/users/${userId}`);
      const user = await response.json();

      dispatch({ type: "FETCH_USER_SUCCESS", payload: user });
    } catch (error) {
      dispatch({ type: "FETCH_USER_ERROR", payload: error.message });
    }
  };
};

// Usage
dispatch(fetchUser(123));
```

## With Redux Toolkit

```javascript
// Redux Toolkit includes thunk by default
import { createAsyncThunk } from "@reduxjs/toolkit";

export const fetchUser = createAsyncThunk("user/fetchUser", async (userId) => {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});
```

---

# So Sánh & Best Practices

## Redux vs Redux Toolkit

| Feature            | Redux         | Redux Toolkit         |
| ------------------ | ------------- | --------------------- |
| **Boilerplate**    | Nhiều         | Ít                    |
| **Setup**          | Manual        | Automatic             |
| **Immutability**   | Manual spread | Immer (mutate safely) |
| **DevTools**       | Manual setup  | Built-in              |
| **Middleware**     | Manual        | Pre-configured        |
| **TypeScript**     | Manual types  | Better inference      |
| **Learning Curve** | Steep         | Easier                |

```javascript
// Redux: ~50 lines
const ADD_TODO = "ADD_TODO";
const addTodo = (text) => ({ type: ADD_TODO, payload: text });
const todoReducer = (state = [], action) => {
  switch (action.type) {
    case ADD_TODO:
      return [...state, { id: Date.now(), text: action.payload }];
    default:
      return state;
  }
};

// Redux Toolkit: ~10 lines
const todoSlice = createSlice({
  name: "todos",
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      state.push({ id: Date.now(), text: action.payload });
    },
  },
});
```

---

## Redux Saga vs Redux Thunk

| Feature                | Redux Saga            | Redux Thunk    |
| ---------------------- | --------------------- | -------------- |
| **Complexity**         | High                  | Low            |
| **Learning Curve**     | Steep (generators)    | Easy           |
| **Testing**            | Easy (pure functions) | Harder (async) |
| **Cancellation**       | ✅ Built-in           | ❌ Manual      |
| **Debounce/Throttle**  | ✅ Built-in           | ❌ Manual      |
| **Race Conditions**    | ✅ Easy               | ❌ Hard        |
| **Parallel Execution** | ✅ Easy               | ⚠️ Promise.all |
| **Bundle Size**        | ~5KB                  | ~1KB           |

### Khi Nào Dùng Saga?

✅ **Complex async flows**

```javascript
// Multiple sequential API calls
function* checkoutSaga() {
  const cart = yield select((state) => state.cart);
  const order = yield call(api.createOrder, cart);
  const payment = yield call(api.processPayment, order.id);
  const shipping = yield call(api.scheduleShipping, order.id);
  yield put({ type: "CHECKOUT_SUCCESS", payload: { order, payment, shipping } });
}
```

✅ **Cancellation**

```javascript
function* searchSaga() {
  yield takeLatest("SEARCH", function* (action) {
    // Previous search automatically cancelled
    const results = yield call(api.search, action.payload);
    yield put({ type: "SEARCH_SUCCESS", payload: results });
  });
}
```

✅ **Debounce/Throttle**

```javascript
function* autoSaveSaga() {
  yield debounce(1000, "UPDATE_FORM", function* (action) {
    yield call(api.saveForm, action.payload);
  });
}
```

### Khi Nào Dùng Thunk?

✅ **Simple async operations**

```javascript
const fetchUser = (id) => async (dispatch) => {
  const user = await api.fetchUser(id);
  dispatch({ type: "FETCH_USER_SUCCESS", payload: user });
};
```

✅ **Quick prototyping**
✅ **Small projects**
✅ **Team không quen generators**

---

## Redux Toolkit vs TanStack Query vs Zustand

| Feature            | Redux Toolkit    | TanStack Query | Zustand      |
| ------------------ | ---------------- | -------------- | ------------ |
| **Purpose**        | Global state     | Server state   | Client state |
| **Boilerplate**    | Medium           | Low            | Very Low     |
| **Caching**        | Manual           | Automatic      | Manual       |
| **Async**          | createAsyncThunk | Built-in       | Manual       |
| **DevTools**       | ✅               | ✅             | ✅           |
| **Bundle Size**    | ~11KB            | ~13KB          | ~1KB         |
| **Learning Curve** | Medium           | Medium         | Easy         |

### Decision Tree

```
Cần quản lý state?
│
├─ Server state (API data)?
│  └─ ✅ TanStack Query
│     - Automatic caching
│     - Background refetching
│     - Optimistic updates
│
├─ Complex global state với nhiều features?
│  └─ ✅ Redux Toolkit
│     - Time-travel debugging
│     - Middleware ecosystem
│     - Predictable state updates
│
└─ Simple client state?
   └─ ✅ Zustand
      - Minimal boilerplate
      - No providers
      - Fast & lightweight
```

---

## Best Practices

### 1. Structure Your Redux Code

```
src/
├─ features/
│  ├─ auth/
│  │  ├─ authSlice.js
│  │  ├─ authSaga.js
│  │  └─ authSelectors.js
│  ├─ todos/
│  │  ├─ todoSlice.js
│  │  └─ todoSelectors.js
│  └─ users/
│     ├─ userSlice.js
│     └─ userAPI.js
├─ store.js
└─ rootSaga.js
```

### 2. Use Selectors

```javascript
// ❌ BAD: Direct state access
function TodoList() {
  const todos = useSelector((state) => state.todos.items);
  const filter = useSelector((state) => state.todos.filter);

  const filteredTodos = todos.filter((todo) => {
    if (filter === "active") return !todo.completed;
    if (filter === "completed") return todo.completed;
    return true;
  });
  // Recalculates every render!
}

// ✅ GOOD: Memoized selector
import { createSelector } from "@reduxjs/toolkit";

const selectTodos = (state) => state.todos.items;
const selectFilter = (state) => state.todos.filter;

const selectFilteredTodos = createSelector([selectTodos, selectFilter], (todos, filter) => {
  if (filter === "active") return todos.filter((t) => !t.completed);
  if (filter === "completed") return todos.filter((t) => t.completed);
  return todos;
});

function TodoList() {
  const filteredTodos = useSelector(selectFilteredTodos);
  // Only recalculates when todos or filter changes
}
```

### 3. Normalize State

```javascript
// ❌ BAD: Nested data
const state = {
  posts: [
    {
      id: 1,
      title: "Post 1",
      author: { id: 1, name: "John" },
      comments: [{ id: 1, text: "Comment 1", author: { id: 2, name: "Jane" } }],
    },
  ],
};

// ✅ GOOD: Normalized data
const state = {
  users: {
    1: { id: 1, name: "John" },
    2: { id: 2, name: "Jane" },
  },
  posts: {
    1: { id: 1, title: "Post 1", authorId: 1, commentIds: [1] },
  },
  comments: {
    1: { id: 1, text: "Comment 1", authorId: 2, postId: 1 },
  },
};

// Use normalizr library
import { normalize, schema } from "normalizr";

const user = new schema.Entity("users");
const comment = new schema.Entity("comments", { author: user });
const post = new schema.Entity("posts", {
  author: user,
  comments: [comment],
});

const normalizedData = normalize(originalData, [post]);
```

### 4. Keep Reducers Pure

```javascript
// ❌ BAD: Side effects in reducer
const todoReducer = (state, action) => {
  switch (action.type) {
    case "ADD_TODO":
      localStorage.setItem("todos", JSON.stringify(state)); // Side effect!
      return [...state, action.payload];
  }
};

// ✅ GOOD: Side effects in middleware
const localStorageMiddleware = (store) => (next) => (action) => {
  const result = next(action);

  if (action.type === "ADD_TODO") {
    localStorage.setItem("todos", JSON.stringify(store.getState().todos));
  }

  return result;
};
```

### 5. Use TypeScript

```typescript
// Define types
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodoState {
  todos: Todo[];
  loading: boolean;
  error: string | null;
}

// Typed slice
const todoSlice = createSlice({
  name: "todos",
  initialState: {
    todos: [],
    loading: false,
    error: null,
  } as TodoState,
  reducers: {
    addTodo: (state, action: PayloadAction<string>) => {
      state.todos.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
      });
    },
  },
});

// Typed hooks
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### 6. Split Large Slices

```javascript
// ❌ BAD: One large slice
const appSlice = createSlice({
  name: "app",
  initialState: {
    users: [],
    posts: [],
    comments: [],
    settings: {},
    // ... 20 more properties
  },
  reducers: {
    // ... 50 reducers
  },
});

// ✅ GOOD: Multiple focused slices
const userSlice = createSlice({
  name: "users",
  initialState: { users: [] },
  reducers: {
    /* user-related reducers */
  },
});

const postSlice = createSlice({
  name: "posts",
  initialState: { posts: [] },
  reducers: {
    /* post-related reducers */
  },
});

const settingsSlice = createSlice({
  name: "settings",
  initialState: {},
  reducers: {
    /* settings-related reducers */
  },
});
```

### 7. Handle Loading States

```javascript
const userSlice = createSlice({
  name: "user",
  initialState: {
    data: null,
    status: "idle", // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.status = "loading";
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.status = "succeeded";
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.status = "failed";
        state.error = action.error.message;
      });
  },
});

// Usage
function UserProfile() {
  const { data, status, error } = useSelector((state) => state.user);

  if (status === "loading") return <Spinner />;
  if (status === "failed") return <Error message={error} />;
  if (status === "succeeded") return <Profile user={data} />;

  return null;
}
```

### 8. Avoid Dispatch in Render

```javascript
// ❌ BAD: Dispatch in render
function Component() {
  const count = useSelector((state) => state.count);

  if (count > 10) {
    dispatch(resetCount()); // Causes infinite loop!
  }

  return <div>{count}</div>;
}

// ✅ GOOD: Dispatch in useEffect
function Component() {
  const count = useSelector((state) => state.count);
  const dispatch = useDispatch();

  useEffect(() => {
    if (count > 10) {
      dispatch(resetCount());
    }
  }, [count, dispatch]);

  return <div>{count}</div>;
}
```

### 9. Use Redux DevTools

```javascript
// Redux Toolkit includes DevTools by default
const store = configureStore({
  reducer: rootReducer,
  devTools: process.env.NODE_ENV !== "production",
});

// Custom DevTools options
const store = configureStore({
  reducer: rootReducer,
  devTools: {
    name: "My App",
    trace: true,
    traceLimit: 25,
  },
});
```

### 10. Test Your Redux Code

```javascript
// Test reducer
import todoReducer, { addTodo } from "./todoSlice";

test("should add todo", () => {
  const initialState = { todos: [] };
  const action = addTodo("Learn Redux");
  const newState = todoReducer(initialState, action);

  expect(newState.todos).toHaveLength(1);
  expect(newState.todos[0].text).toBe("Learn Redux");
});

// Test saga
import { call, put } from "redux-saga/effects";
import { fetchUserSaga } from "./userSaga";

test("should fetch user", () => {
  const gen = fetchUserSaga({ payload: 1 });

  expect(gen.next().value).toEqual(call(api.fetchUser, 1));
  expect(gen.next({ id: 1, name: "John" }).value).toEqual(
    put({ type: "FETCH_USER_SUCCESS", payload: { id: 1, name: "John" } }),
  );
});

// Test async thunk
import { fetchUser } from "./userSlice";

test("should fetch user", async () => {
  const dispatch = jest.fn();
  const getState = jest.fn();

  await fetchUser(1)(dispatch, getState, undefined);

  expect(dispatch).toHaveBeenCalledWith(expect.objectContaining({ type: "user/fetchUser/pending" }));
});
```

---

## Performance Tips

### 1. Use Shallow Equality

```javascript
import { shallowEqual, useSelector } from "react-redux";

// ❌ BAD: New object every time
const { user, theme } = useSelector((state) => ({
  user: state.user,
  theme: state.theme,
}));
// Re-renders on any state change!

// ✅ GOOD: Shallow equality
const { user, theme } = useSelector(
  (state) => ({
    user: state.user,
    theme: state.theme,
  }),
  shallowEqual,
);
// Only re-renders when user or theme changes
```

### 2. Split Selectors

```javascript
// ❌ BAD: One selector for everything
const data = useSelector((state) => ({
  users: state.users,
  posts: state.posts,
  comments: state.comments,
}));
// Re-renders when any of them changes

// ✅ GOOD: Separate selectors
const users = useSelector((state) => state.users);
const posts = useSelector((state) => state.posts);
const comments = useSelector((state) => state.comments);
// Only re-renders when specific data changes
```

### 3. Memoize Expensive Computations

```javascript
import { createSelector } from "@reduxjs/toolkit";

const selectExpensiveData = createSelector([(state) => state.data], (data) => {
  // Expensive computation
  return data.map((item) => ({
    ...item,
    computed: expensiveCalculation(item),
  }));
});
```

---

## Tổng Kết

### Key Takeaways

1. **Redux Toolkit**: Recommended way to write Redux
2. **Redux Saga**: For complex async flows
3. **Redux Thunk**: For simple async operations
4. **Normalize state**: Flat structure is better
5. **Use selectors**: Memoize derived data
6. **TypeScript**: Type safety is worth it
7. **Test**: Redux code is easy to test
8. **DevTools**: Use them for debugging

### Migration Path

```
Plain Redux
  ↓
Redux Toolkit (easier, less boilerplate)
  ↓
Consider alternatives:
  - TanStack Query (for server state)
  - Zustand (for simple client state)
  - Jotai/Recoil (for atomic state)
```

### Resources

- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [Redux Saga Docs](https://redux-saga.js.org/)
- [Redux Style Guide](https://redux.js.org/style-guide/)
- [Redux DevTools](https://github.com/reduxjs/redux-devtools)

---

**Lưu Ý**: Redux không phải lúc nào cũng cần thiết. Với React 18+, Context API + useReducer có thể đủ cho nhiều use cases. Chỉ dùng Redux khi thực sự cần global state management phức tạp.
