# TanStack Query & Zustand - Hướng Dẫn Toàn Diện

## Mục Lục

1. [TanStack Query (React Query)](#tanstack-query-react-query)
2. [Zustand](#zustand)
3. [So Sánh TanStack Query vs Zustand](#so-sánh-tanstack-query-vs-zustand)
4. [Khi Nào Dùng Gì?](#khi-nào-dùng-gì)
5. [Kết Hợp TanStack Query + Zustand](#kết-hợp-tanstack-query--zustand)

---

# TanStack Query (React Query)

## Giới Thiệu

**TanStack Query** (trước đây là React Query) là thư viện quản lý server state, giúp fetch, cache, sync và update data từ server.

### Tại Sao Cần TanStack Query?

```javascript
// ❌ Traditional approach (nhiều boilerplate)
function Users() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch("/api/users")
      .then((res) => res.json())
      .then((data) => {
        setUsers(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}

// ✅ With TanStack Query (clean & powerful)
function Users() {
  const {
    data: users,
    isLoading,
    error,
  } = useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then((res) => res.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

### Lợi Ích

✅ **Automatic caching**: Cache data tự động
✅ **Background refetching**: Tự động refetch khi cần
✅ **Stale data management**: Quản lý data cũ
✅ **Request deduplication**: Gộp requests giống nhau
✅ **Pagination & infinite scroll**: Built-in support
✅ **Optimistic updates**: Update UI trước khi server response
✅ **DevTools**: Debug dễ dàng

---

## Setup

```bash
npm install @tanstack/react-query
# or
yarn add @tanstack/react-query
```

```javascript
// App.js
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      cacheTime: 1000 * 60 * 10, // 10 minutes
      refetchOnWindowFocus: false,
      retry: 1,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

---

## useQuery - Fetching Data

### Basic Usage

```javascript
import { useQuery } from "@tanstack/react-query";

function Users() {
  const { data, isLoading, error, isError, isFetching } = useQuery({
    queryKey: ["users"],
    queryFn: async () => {
      const response = await fetch("/api/users");
      if (!response.ok) throw new Error("Failed to fetch");
      return response.json();
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return (
    <div>
      {isFetching && <div>Updating...</div>}
      {data.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

### Query Keys

```javascript
// Simple key
useQuery({ queryKey: ["users"], queryFn: fetchUsers });

// Key with parameters
useQuery({
  queryKey: ["user", userId],
  queryFn: () => fetchUser(userId),
});

// Complex key
useQuery({
  queryKey: ["users", { status: "active", page: 1 }],
  queryFn: () => fetchUsers({ status: "active", page: 1 }),
});

// Hierarchical keys
useQuery({ queryKey: ["users", userId, "posts"], queryFn: fetchUserPosts });
```

### Query Options

```javascript
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,

  // Caching
  staleTime: 1000 * 60 * 5, // Data fresh trong 5 phút
  cacheTime: 1000 * 60 * 10, // Cache tồn tại 10 phút

  // Refetching
  refetchOnMount: true, // Refetch khi component mount
  refetchOnWindowFocus: false, // Refetch khi focus window
  refetchOnReconnect: true, // Refetch khi reconnect
  refetchInterval: 1000 * 30, // Refetch mỗi 30 giây

  // Retry
  retry: 3, // Retry 3 lần nếu fail
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),

  // Enabled
  enabled: !!userId, // Chỉ fetch khi có userId

  // Callbacks
  onSuccess: (data) => {
    console.log("Success:", data);
  },
  onError: (error) => {
    console.error("Error:", error);
  },
  onSettled: (data, error) => {
    console.log("Settled");
  },

  // Select (transform data)
  select: (data) => data.filter((user) => user.isActive),
});
```

### Dependent Queries

```javascript
function UserPosts({ userId }) {
  // Query 1: Fetch user
  const { data: user } = useQuery({
    queryKey: ["user", userId],
    queryFn: () => fetchUser(userId),
  });

  // Query 2: Fetch posts (depends on user)
  const { data: posts } = useQuery({
    queryKey: ["posts", user?.id],
    queryFn: () => fetchPosts(user.id),
    enabled: !!user?.id, // Chỉ fetch khi có user
  });

  return <div>{/* ... */}</div>;
}
```

### Parallel Queries

```javascript
function Dashboard() {
  // Fetch multiple queries in parallel
  const users = useQuery({ queryKey: ["users"], queryFn: fetchUsers });
  const posts = useQuery({ queryKey: ["posts"], queryFn: fetchPosts });
  const comments = useQuery({ queryKey: ["comments"], queryFn: fetchComments });

  if (users.isLoading || posts.isLoading || comments.isLoading) {
    return <div>Loading...</div>;
  }

  return <div>{/* ... */}</div>;
}

// Hoặc dùng useQueries
function Dashboard() {
  const results = useQueries({
    queries: [
      { queryKey: ["users"], queryFn: fetchUsers },
      { queryKey: ["posts"], queryFn: fetchPosts },
      { queryKey: ["comments"], queryFn: fetchComments },
    ],
  });

  const isLoading = results.some((result) => result.isLoading);

  return <div>{/* ... */}</div>;
}
```

---

## useMutation - Updating Data

### Basic Usage

```javascript
import { useMutation, useQueryClient } from "@tanstack/react-query";

function CreateUser() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newUser) => {
      return fetch("/api/users", {
        method: "POST",
        body: JSON.stringify(newUser),
        headers: { "Content-Type": "application/json" },
      }).then((res) => res.json());
    },
    onSuccess: () => {
      // Invalidate và refetch
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    mutation.mutate({ name: "John Doe", email: "john@example.com" });
  };

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit" disabled={mutation.isLoading}>
        {mutation.isLoading ? "Creating..." : "Create User"}
      </button>
      {mutation.isError && <div>Error: {mutation.error.message}</div>}
      {mutation.isSuccess && <div>User created!</div>}
    </form>
  );
}
```

### Mutation Options

```javascript
const mutation = useMutation({
  mutationFn: createUser,

  // Callbacks
  onMutate: async (newUser) => {
    // Optimistic update
    await queryClient.cancelQueries({ queryKey: ["users"] });
    const previousUsers = queryClient.getQueryData(["users"]);

    queryClient.setQueryData(["users"], (old) => [...old, newUser]);

    return { previousUsers };
  },

  onError: (err, newUser, context) => {
    // Rollback on error
    queryClient.setQueryData(["users"], context.previousUsers);
  },

  onSuccess: (data, variables, context) => {
    console.log("Success:", data);
  },

  onSettled: () => {
    // Refetch sau khi done
    queryClient.invalidateQueries({ queryKey: ["users"] });
  },
});
```

### Update, Delete Examples

```javascript
// Update
const updateMutation = useMutation({
  mutationFn: ({ id, data }) => {
    return fetch(`/api/users/${id}`, {
      method: "PUT",
      body: JSON.stringify(data),
      headers: { "Content-Type": "application/json" },
    }).then((res) => res.json());
  },
  onSuccess: (data, variables) => {
    queryClient.invalidateQueries({ queryKey: ["users"] });
    queryClient.invalidateQueries({ queryKey: ["user", variables.id] });
  },
});

// Delete
const deleteMutation = useMutation({
  mutationFn: (id) => {
    return fetch(`/api/users/${id}`, { method: "DELETE" });
  },
  onSuccess: (data, id) => {
    queryClient.invalidateQueries({ queryKey: ["users"] });
    queryClient.removeQueries({ queryKey: ["user", id] });
  },
});

// Usage
updateMutation.mutate({ id: 1, data: { name: "Jane" } });
deleteMutation.mutate(1);
```

---

## Optimistic Updates

```javascript
function TodoList() {
  const queryClient = useQueryClient();

  const updateTodo = useMutation({
    mutationFn: ({ id, completed }) => {
      return fetch(`/api/todos/${id}`, {
        method: "PATCH",
        body: JSON.stringify({ completed }),
        headers: { "Content-Type": "application/json" },
      });
    },

    // Optimistic update
    onMutate: async ({ id, completed }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ["todos"] });

      // Snapshot previous value
      const previousTodos = queryClient.getQueryData(["todos"]);

      // Optimistically update
      queryClient.setQueryData(["todos"], (old) => old.map((todo) => (todo.id === id ? { ...todo, completed } : todo)));

      // Return context with snapshot
      return { previousTodos };
    },

    // Rollback on error
    onError: (err, variables, context) => {
      queryClient.setQueryData(["todos"], context.previousTodos);
    },

    // Always refetch after error or success
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ["todos"] });
    },
  });

  return (
    <div>
      {todos.map((todo) => (
        <div key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={(e) =>
              updateTodo.mutate({
                id: todo.id,
                completed: e.target.checked,
              })
            }
          />
          {todo.title}
        </div>
      ))}
    </div>
  );
}
```

---

## Pagination

```javascript
function Posts() {
  const [page, setPage] = useState(1);

  const { data, isLoading, isPlaceholderData } = useQuery({
    queryKey: ["posts", page],
    queryFn: () => fetchPosts(page),
    placeholderData: keepPreviousData, // Keep previous data while fetching
    staleTime: 5000,
  });

  return (
    <div>
      {data.posts.map((post) => (
        <div key={post.id}>{post.title}</div>
      ))}

      <button onClick={() => setPage((old) => Math.max(old - 1, 1))} disabled={page === 1}>
        Previous
      </button>

      <span>Page {page}</span>

      <button onClick={() => setPage((old) => old + 1)} disabled={isPlaceholderData || !data.hasMore}>
        Next
      </button>
    </div>
  );
}
```

---

## Infinite Queries

```javascript
import { useInfiniteQuery } from "@tanstack/react-query";

function Posts() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isLoading } = useInfiniteQuery({
    queryKey: ["posts"],
    queryFn: ({ pageParam = 1 }) => fetchPosts(pageParam),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
    initialPageParam: 1,
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      {data.pages.map((page, i) => (
        <div key={i}>
          {page.posts.map((post) => (
            <div key={post.id}>{post.title}</div>
          ))}
        </div>
      ))}

      <button onClick={() => fetchNextPage()} disabled={!hasNextPage || isFetchingNextPage}>
        {isFetchingNextPage ? "Loading more..." : hasNextPage ? "Load More" : "Nothing more to load"}
      </button>
    </div>
  );
}
```

---

## Query Invalidation

```javascript
const queryClient = useQueryClient();

// Invalidate specific query
queryClient.invalidateQueries({ queryKey: ["users"] });

// Invalidate all queries starting with 'users'
queryClient.invalidateQueries({ queryKey: ["users"] });

// Invalidate exact query
queryClient.invalidateQueries({
  queryKey: ["user", 1],
  exact: true,
});

// Invalidate multiple queries
queryClient.invalidateQueries({ queryKey: ["users"] });
queryClient.invalidateQueries({ queryKey: ["posts"] });

// Refetch immediately
queryClient.invalidateQueries({
  queryKey: ["users"],
  refetchType: "active", // 'active' | 'inactive' | 'all' | 'none'
});
```

---

## Manual Cache Updates

```javascript
const queryClient = useQueryClient();

// Get query data
const users = queryClient.getQueryData(["users"]);

// Set query data
queryClient.setQueryData(["users"], newUsers);

// Update query data
queryClient.setQueryData(["users"], (old) => [...old, newUser]);

// Remove query
queryClient.removeQueries({ queryKey: ["user", 1] });

// Cancel queries
queryClient.cancelQueries({ queryKey: ["users"] });

// Reset queries
queryClient.resetQueries({ queryKey: ["users"] });
```

---

# Zustand

## Giới Thiệu

**Zustand** là thư viện quản lý client state đơn giản, nhẹ và không cần boilerplate như Redux.

### Tại Sao Dùng Zustand?

```javascript
// ❌ Context API (verbose, re-render issues)
const UserContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");

  return <UserContext.Provider value={{ user, setUser, theme, setTheme }}>{children}</UserContext.Provider>;
}

// ✅ Zustand (simple, performant)
import { create } from "zustand";

const useStore = create((set) => ({
  user: null,
  theme: "light",
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme }),
}));
```

### Lợi Ích

✅ **Simple API**: Dễ học, ít boilerplate
✅ **No providers**: Không cần wrap components
✅ **TypeScript support**: Type-safe
✅ **DevTools**: Redux DevTools integration
✅ **Middleware**: Persist, immer, devtools
✅ **Small bundle**: ~1KB gzipped
✅ **No re-render issues**: Chỉ re-render khi cần

---

## Setup

```bash
npm install zustand
# or
yarn add zustand
```

---

## Basic Usage

### Create Store

```javascript
import { create } from "zustand";

const useStore = create((set) => ({
  // State
  count: 0,
  user: null,

  // Actions
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  setUser: (user) => set({ user }),
  reset: () => set({ count: 0, user: null }),
}));
```

### Use in Components

```javascript
function Counter() {
  // Subscribe to specific state
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  const decrement = useStore((state) => state.decrement);

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}

// Hoặc subscribe to multiple states
function Counter() {
  const { count, increment, decrement } = useStore((state) => ({
    count: state.count,
    increment: state.increment,
    decrement: state.decrement,
  }));

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

---

## Advanced Patterns

### Async Actions

```javascript
const useStore = create((set, get) => ({
  users: [],
  loading: false,
  error: null,

  fetchUsers: async () => {
    set({ loading: true, error: null });
    try {
      const response = await fetch("/api/users");
      const users = await response.json();
      set({ users, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },

  addUser: async (user) => {
    try {
      const response = await fetch("/api/users", {
        method: "POST",
        body: JSON.stringify(user),
        headers: { "Content-Type": "application/json" },
      });
      const newUser = await response.json();
      set((state) => ({ users: [...state.users, newUser] }));
    } catch (error) {
      set({ error: error.message });
    }
  },
}));
```

### Computed Values

```javascript
const useStore = create((set, get) => ({
  todos: [],

  addTodo: (todo) =>
    set((state) => ({
      todos: [...state.todos, todo],
    })),

  // Computed values (getters)
  get completedTodos() {
    return get().todos.filter((todo) => todo.completed);
  },

  get activeTodos() {
    return get().todos.filter((todo) => !todo.completed);
  },

  get todoCount() {
    return get().todos.length;
  },
}));

// Usage
function TodoStats() {
  const completedTodos = useStore((state) => state.completedTodos);
  const activeTodos = useStore((state) => state.activeTodos);

  return (
    <div>
      <p>Completed: {completedTodos.length}</p>
      <p>Active: {activeTodos.length}</p>
    </div>
  );
}
```

### Slices Pattern

```javascript
// userSlice.js
const createUserSlice = (set) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null }),
});

// cartSlice.js
const createCartSlice = (set) => ({
  items: [],
  addItem: (item) =>
    set((state) => ({
      items: [...state.items, item],
    })),
  removeItem: (id) =>
    set((state) => ({
      items: state.items.filter((item) => item.id !== id),
    })),
  clearCart: () => set({ items: [] }),
});

// store.js
const useStore = create((set, get) => ({
  ...createUserSlice(set, get),
  ...createCartSlice(set, get),
}));
```

### Nested State Updates

```javascript
const useStore = create((set) => ({
  user: {
    profile: {
      name: "John",
      email: "john@example.com",
    },
    settings: {
      theme: "light",
      notifications: true,
    },
  },

  // ❌ BAD: Mutating state
  updateName: (name) =>
    set((state) => {
      state.user.profile.name = name; // Don't do this!
      return state;
    }),

  // ✅ GOOD: Immutable update
  updateName: (name) =>
    set((state) => ({
      user: {
        ...state.user,
        profile: {
          ...state.user.profile,
          name,
        },
      },
    })),
}));
```

---

## Middleware

### Persist Middleware

```javascript
import { create } from "zustand";
import { persist } from "zustand/middleware";

const useStore = create(
  persist(
    (set) => ({
      user: null,
      theme: "light",
      setUser: (user) => set({ user }),
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: "app-storage", // localStorage key
      partialize: (state) => ({
        theme: state.theme, // Chỉ persist theme
      }),
    },
  ),
);
```

### Immer Middleware

```javascript
import { create } from "zustand";
import { immer } from "zustand/middleware/immer";

const useStore = create(
  immer((set) => ({
    user: {
      profile: {
        name: "John",
        email: "john@example.com",
      },
    },

    // ✅ Mutate directly với immer
    updateName: (name) =>
      set((state) => {
        state.user.profile.name = name; // OK với immer!
      }),
  })),
);
```

### DevTools Middleware

```javascript
import { create } from "zustand";
import { devtools } from "zustand/middleware";

const useStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }), false, "increment"),
      decrement: () => set((state) => ({ count: state.count - 1 }), false, "decrement"),
    }),
    { name: "CounterStore" },
  ),
);
```

### Combine Middleware

```javascript
import { create } from "zustand";
import { devtools, persist } from "zustand/middleware";
import { immer } from "zustand/middleware/immer";

const useStore = create(
  devtools(
    persist(
      immer((set) => ({
        user: null,
        todos: [],

        setUser: (user) =>
          set((state) => {
            state.user = user;
          }),

        addTodo: (todo) =>
          set((state) => {
            state.todos.push(todo);
          }),
      })),
      { name: "app-storage" },
    ),
    { name: "AppStore" },
  ),
);
```

---

## TypeScript Support

```typescript
import { create } from 'zustand';

interface User {
  id: string;
  name: string;
  email: string;
}

interface StoreState {
  user: User | null;
  count: number;
  setUser: (user: User | null) => void;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

const useStore = create<StoreState>((set) => ({
  user: null,
  count: 0,

  setUser: (user) => set({ user }),
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0, user: null })
}));

// Usage with TypeScript
function Counter() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);

  return (
    <div>
      <h1>{count}</h1>
      <button onClick={increment}>+</button>
    </div>
  );
}
```

---

## Selectors & Performance

### Shallow Comparison

```javascript
import { create } from "zustand";
import { shallow } from "zustand/shallow";

const useStore = create((set) => ({
  user: { name: "John", age: 30 },
  theme: "light",
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme }),
}));

// ❌ BAD: Re-renders on any state change
function Component() {
  const { user, theme } = useStore();
  // Re-renders khi user HOẶC theme thay đổi
}

// ✅ GOOD: Only re-renders when user or theme changes
function Component() {
  const { user, theme } = useStore((state) => ({ user: state.user, theme: state.theme }), shallow);
  // Chỉ re-render khi user hoặc theme thực sự thay đổi
}

// ✅ BEST: Subscribe to specific values
function Component() {
  const userName = useStore((state) => state.user.name);
  const theme = useStore((state) => state.theme);
  // Chỉ re-render khi userName hoặc theme thay đổi
}
```

### Custom Selectors

```javascript
// selectors.js
export const selectUser = (state) => state.user;
export const selectUserName = (state) => state.user?.name;
export const selectCompletedTodos = (state) => state.todos.filter((todo) => todo.completed);

// Component
import { selectUserName, selectCompletedTodos } from "./selectors";

function Component() {
  const userName = useStore(selectUserName);
  const completedTodos = useStore(selectCompletedTodos);

  return <div>{userName}</div>;
}
```

---

## Outside React

```javascript
// Access store outside React components
const useStore = create((set, get) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// Get state
const count = useStore.getState().count;

// Set state
useStore.setState({ count: 10 });

// Subscribe to changes
const unsubscribe = useStore.subscribe((state) => {
  console.log("State changed:", state);
});

// Unsubscribe
unsubscribe();

// Usage in non-React code
async function fetchData() {
  useStore.setState({ loading: true });

  try {
    const data = await fetch("/api/data").then((r) => r.json());
    useStore.setState({ data, loading: false });
  } catch (error) {
    useStore.setState({ error, loading: false });
  }
}
```

---

# So Sánh TanStack Query vs Zustand

## Mục Đích Khác Nhau

| Aspect                 | TanStack Query          | Zustand                     |
| ---------------------- | ----------------------- | --------------------------- |
| **Purpose**            | Server state management | Client state management     |
| **Data Source**        | Remote (API, database)  | Local (UI state, form data) |
| **Caching**            | ✅ Built-in             | ❌ Manual                   |
| **Refetching**         | ✅ Automatic            | ❌ Manual                   |
| **Optimistic Updates** | ✅ Built-in             | ⚠️ Manual                   |
| **DevTools**           | ✅ React Query DevTools | ✅ Redux DevTools           |
| **Bundle Size**        | ~13KB                   | ~1KB                        |
| **Learning Curve**     | Medium                  | Easy                        |

## Server State vs Client State

```javascript
// ❌ BAD: Dùng Zustand cho server state
const useStore = create((set) => ({
  users: [],
  loading: false,

  fetchUsers: async () => {
    set({ loading: true });
    const users = await fetch("/api/users").then((r) => r.json());
    set({ users, loading: false });
  },
}));
// Không có caching, refetching, stale data management

// ✅ GOOD: Dùng TanStack Query cho server state
const { data: users, isLoading } = useQuery({
  queryKey: ["users"],
  queryFn: () => fetch("/api/users").then((r) => r.json()),
});
// Có caching, automatic refetching, stale data management

// ✅ GOOD: Dùng Zustand cho client state
const useStore = create((set) => ({
  sidebarOpen: false,
  theme: "light",
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  setTheme: (theme) => set({ theme }),
}));
```

## Use Cases

### TanStack Query - Dùng Cho:

✅ **Fetching data từ API**

```javascript
const { data } = useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
});
```

✅ **Mutations (POST, PUT, DELETE)**

```javascript
const mutation = useMutation({
  mutationFn: createUser,
  onSuccess: () => queryClient.invalidateQueries(["users"]),
});
```

✅ **Pagination & Infinite scroll**

```javascript
const { data, fetchNextPage } = useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: fetchPosts,
  getNextPageParam: (lastPage) => lastPage.nextCursor,
});
```

✅ **Real-time data với polling**

```javascript
useQuery({
  queryKey: ["notifications"],
  queryFn: fetchNotifications,
  refetchInterval: 5000, // Poll every 5 seconds
});
```

### Zustand - Dùng Cho:

✅ **UI State**

```javascript
const useStore = create((set) => ({
  sidebarOpen: false,
  modalOpen: false,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
}));
```

✅ **Form State**

```javascript
const useFormStore = create((set) => ({
  formData: {},
  updateField: (field, value) =>
    set((state) => ({
      formData: { ...state.formData, [field]: value },
    })),
}));
```

✅ **Theme, Settings**

```javascript
const useStore = create(
  persist(
    (set) => ({
      theme: "light",
      language: "en",
      setTheme: (theme) => set({ theme }),
    }),
    { name: "settings" },
  ),
);
```

✅ **Shopping Cart**

```javascript
const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) =>
    set((state) => ({
      items: state.items.filter((i) => i.id !== id),
    })),
}));
```

---

# Khi Nào Dùng Gì?

## Decision Tree

```
Cần quản lý state?
│
├─ Data từ server/API?
│  └─ ✅ Dùng TanStack Query
│     - Fetching users, posts, products
│     - CRUD operations
│     - Pagination, infinite scroll
│     - Real-time polling
│
└─ Data local/UI state?
   └─ ✅ Dùng Zustand
      - Sidebar open/close
      - Theme, language
      - Form data
      - Shopping cart
      - Modal state
```

## Examples

### ✅ TanStack Query

```javascript
// Fetching users
const { data: users } = useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
});

// Creating user
const createMutation = useMutation({
  mutationFn: createUser,
  onSuccess: () => queryClient.invalidateQueries(["users"]),
});

// Pagination
const { data, fetchNextPage } = useInfiniteQuery({
  queryKey: ["posts"],
  queryFn: fetchPosts,
});
```

### ✅ Zustand

```javascript
// UI state
const useUIStore = create((set) => ({
  sidebarOpen: false,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
}));

// Theme
const useThemeStore = create(
  persist(
    (set) => ({
      theme: "light",
      setTheme: (theme) => set({ theme }),
    }),
    { name: "theme" },
  ),
);

// Cart
const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));
```

---

# Kết Hợp TanStack Query + Zustand

## Tại Sao Kết Hợp?

TanStack Query và Zustand giải quyết các vấn đề khác nhau và có thể kết hợp tốt:

- **TanStack Query**: Server state (API data)
- **Zustand**: Client state (UI, settings, cart)

## Architecture Pattern

```
┌─────────────────────────────────────────┐
│           React Application             │
├─────────────────────────────────────────┤
│  Components                             │
│  ├─ useQuery (TanStack Query)           │
│  │  └─ Server State (users, posts)      │
│  │                                       │
│  └─ useStore (Zustand)                  │
│     └─ Client State (theme, sidebar)    │
└─────────────────────────────────────────┘
```

## Real-World Example: E-commerce App

```javascript
// ============================================
// Zustand Store - Client State
// ============================================
import { create } from "zustand";
import { persist } from "zustand/middleware";

// UI Store
export const useUIStore = create((set) => ({
  sidebarOpen: false,
  modalOpen: false,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  openModal: () => set({ modalOpen: true }),
  closeModal: () => set({ modalOpen: false }),
}));

// Theme Store
export const useThemeStore = create(
  persist(
    (set) => ({
      theme: "light",
      language: "en",
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    { name: "theme-storage" },
  ),
);

// Cart Store
export const useCartStore = create(
  persist(
    (set, get) => ({
      items: [],

      addItem: (product) =>
        set((state) => {
          const existingItem = state.items.find((item) => item.id === product.id);

          if (existingItem) {
            return {
              items: state.items.map((item) =>
                item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item,
              ),
            };
          }

          return { items: [...state.items, { ...product, quantity: 1 }] };
        }),

      removeItem: (id) =>
        set((state) => ({
          items: state.items.filter((item) => item.id !== id),
        })),

      updateQuantity: (id, quantity) =>
        set((state) => ({
          items: state.items.map((item) => (item.id === id ? { ...item, quantity } : item)),
        })),

      clearCart: () => set({ items: [] }),

      // Computed values
      get total() {
        return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0);
      },

      get itemCount() {
        return get().items.reduce((sum, item) => sum + item.quantity, 0);
      },
    }),
    { name: "cart-storage" },
  ),
);

// ============================================
// TanStack Query - Server State
// ============================================
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

// Fetch products
export function useProducts(filters) {
  return useQuery({
    queryKey: ["products", filters],
    queryFn: () => fetch(`/api/products?${new URLSearchParams(filters)}`).then((res) => res.json()),
    staleTime: 1000 * 60 * 5, // 5 minutes
  });
}

// Fetch single product
export function useProduct(id) {
  return useQuery({
    queryKey: ["product", id],
    queryFn: () => fetch(`/api/products/${id}`).then((res) => res.json()),
    enabled: !!id,
  });
}

// Fetch user
export function useUser() {
  return useQuery({
    queryKey: ["user"],
    queryFn: () => fetch("/api/user").then((res) => res.json()),
    staleTime: Infinity, // User data rarely changes
  });
}

// Create order
export function useCreateOrder() {
  const queryClient = useQueryClient();
  const clearCart = useCartStore((state) => state.clearCart);

  return useMutation({
    mutationFn: (orderData) => {
      return fetch("/api/orders", {
        method: "POST",
        body: JSON.stringify(orderData),
        headers: { "Content-Type": "application/json" },
      }).then((res) => res.json());
    },
    onSuccess: () => {
      // Invalidate orders query
      queryClient.invalidateQueries({ queryKey: ["orders"] });

      // Clear cart (Zustand)
      clearCart();
    },
  });
}

// ============================================
// Components
// ============================================

// Product List Component
function ProductList() {
  // TanStack Query - Server state
  const { data: products, isLoading } = useProducts({ category: "electronics" });

  // Zustand - Client state
  const addItem = useCartStore((state) => state.addItem);
  const theme = useThemeStore((state) => state.theme);

  if (isLoading) return <div>Loading...</div>;

  return (
    <div className={theme}>
      {products.map((product) => (
        <div key={product.id}>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
          <button onClick={() => addItem(product)}>Add to Cart</button>
        </div>
      ))}
    </div>
  );
}

// Cart Component
function Cart() {
  // Zustand - Client state
  const items = useCartStore((state) => state.items);
  const total = useCartStore((state) => state.total);
  const removeItem = useCartStore((state) => state.removeItem);

  // TanStack Query - Server state
  const createOrder = useCreateOrder();

  const handleCheckout = () => {
    createOrder.mutate({
      items: items.map((item) => ({
        productId: item.id,
        quantity: item.quantity,
      })),
      total,
    });
  };

  return (
    <div>
      <h2>Shopping Cart</h2>
      {items.map((item) => (
        <div key={item.id}>
          <span>
            {item.name} x {item.quantity}
          </span>
          <span>${item.price * item.quantity}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      <div>Total: ${total}</div>
      <button onClick={handleCheckout} disabled={createOrder.isLoading}>
        {createOrder.isLoading ? "Processing..." : "Checkout"}
      </button>
    </div>
  );
}

// Header Component
function Header() {
  // Zustand - Client state
  const itemCount = useCartStore((state) => state.itemCount);
  const sidebarOpen = useUIStore((state) => state.sidebarOpen);
  const toggleSidebar = useUIStore((state) => state.toggleSidebar);

  // TanStack Query - Server state
  const { data: user } = useUser();

  return (
    <header>
      <button onClick={toggleSidebar}>{sidebarOpen ? "Close" : "Menu"}</button>

      <div>Cart ({itemCount})</div>

      {user && <div>Welcome, {user.name}</div>}
    </header>
  );
}
```

## Sync Zustand with TanStack Query

```javascript
// Sync cart with server
const useCartStore = create((set, get) => ({
  items: [],
  syncing: false,

  addItem: async (product) => {
    // Optimistic update
    set((state) => ({
      items: [...state.items, { ...product, quantity: 1 }],
    }));

    // Sync with server
    try {
      set({ syncing: true });
      await fetch("/api/cart", {
        method: "POST",
        body: JSON.stringify({ productId: product.id }),
        headers: { "Content-Type": "application/json" },
      });
      set({ syncing: false });
    } catch (error) {
      // Rollback on error
      set((state) => ({
        items: state.items.filter((item) => item.id !== product.id),
        syncing: false,
      }));
    }
  },
}));

// Or use TanStack Query mutation with Zustand
function useAddToCart() {
  const addItemToStore = useCartStore((state) => state.addItem);

  return useMutation({
    mutationFn: (product) => {
      return fetch("/api/cart", {
        method: "POST",
        body: JSON.stringify({ productId: product.id }),
        headers: { "Content-Type": "application/json" },
      });
    },
    onMutate: (product) => {
      // Optimistic update in Zustand
      addItemToStore(product);
    },
    onError: (error, product) => {
      // Rollback in Zustand
      const removeItem = useCartStore.getState().removeItem;
      removeItem(product.id);
    },
  });
}
```

---

## Best Practices

### 1. Separate Concerns

```javascript
// ✅ GOOD: Clear separation
// Server state → TanStack Query
const { data: users } = useQuery({ queryKey: ["users"], queryFn: fetchUsers });

// Client state → Zustand
const theme = useThemeStore((state) => state.theme);

// ❌ BAD: Mixing concerns
const useStore = create((set) => ({
  users: [], // Should be in TanStack Query
  theme: "light", // OK in Zustand
  fetchUsers: async () => {
    /* ... */
  }, // Should use TanStack Query
}));
```

### 2. Use Selectors

```javascript
// ✅ GOOD: Specific selectors
const itemCount = useCartStore((state) => state.itemCount);
const total = useCartStore((state) => state.total);

// ❌ BAD: Subscribe to entire store
const store = useCartStore();
// Re-renders on any state change
```

### 3. Persist Only Client State

```javascript
// ✅ GOOD: Persist client state
const useThemeStore = create(
  persist((set) => ({ theme: "light", setTheme: (theme) => set({ theme }) }), { name: "theme" }),
);

// ❌ BAD: Persist server state
// Don't persist API data, let TanStack Query handle caching
```

### 4. Combine Middleware Wisely

```javascript
// ✅ GOOD: Combine middleware for client state
const useStore = create(
  devtools(
    persist(
      immer((set) => ({
        // Client state
        settings: { theme: "light", language: "en" },
        updateSettings: (key, value) =>
          set((state) => {
            state.settings[key] = value;
          }),
      })),
      { name: "settings" },
    ),
    { name: "SettingsStore" },
  ),
);
```

---

## Performance Tips

### TanStack Query

```javascript
// 1. Set appropriate staleTime
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
  staleTime: 1000 * 60 * 5, // 5 minutes
});

// 2. Use select to transform data
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
  select: (data) => data.filter((user) => user.isActive),
});

// 3. Prefetch data
queryClient.prefetchQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
});

// 4. Use placeholderData for pagination
useQuery({
  queryKey: ["posts", page],
  queryFn: () => fetchPosts(page),
  placeholderData: keepPreviousData,
});
```

### Zustand

```javascript
// 1. Use shallow comparison
import { shallow } from "zustand/shallow";

const { theme, language } = useStore((state) => ({ theme: state.theme, language: state.language }), shallow);

// 2. Split stores
const useUIStore = create(/* UI state */);
const useCartStore = create(/* Cart state */);
const useThemeStore = create(/* Theme state */);

// 3. Use selectors
const itemCount = useCartStore((state) => state.itemCount);

// 4. Memoize computed values
const useStore = create((set, get) => ({
  items: [],
  get total() {
    return get().items.reduce((sum, item) => sum + item.price, 0);
  },
}));
```

---

## Testing

### TanStack Query

```javascript
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { renderHook, waitFor } from "@testing-library/react";

test("fetches users", async () => {
  const queryClient = new QueryClient();
  const wrapper = ({ children }) => <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;

  const { result } = renderHook(
    () =>
      useQuery({
        queryKey: ["users"],
        queryFn: fetchUsers,
      }),
    { wrapper },
  );

  await waitFor(() => expect(result.current.isSuccess).toBe(true));
  expect(result.current.data).toHaveLength(3);
});
```

### Zustand

```javascript
import { renderHook, act } from "@testing-library/react";

test("increments count", () => {
  const { result } = renderHook(() => useStore());

  expect(result.current.count).toBe(0);

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

---

## Tổng Kết

### Key Takeaways

1. **TanStack Query**: Server state (API, database)
2. **Zustand**: Client state (UI, settings, cart)
3. **Kết hợp**: Dùng cả hai cho best results
4. **Separation**: Tách biệt server state và client state
5. **Performance**: Dùng selectors, shallow comparison
6. **Persistence**: Chỉ persist client state

### Decision Matrix

| State Type    | Tool              | Example                      |
| ------------- | ----------------- | ---------------------------- |
| API Data      | TanStack Query    | Users, posts, products       |
| UI State      | Zustand           | Sidebar, modal, theme        |
| Form Data     | Zustand           | Form fields, validation      |
| Shopping Cart | Zustand           | Cart items, total            |
| User Session  | TanStack Query    | Current user, auth           |
| Settings      | Zustand (persist) | Theme, language, preferences |

### Resources

- [TanStack Query Docs](https://tanstack.com/query/latest)
- [Zustand Docs](https://docs.pmnd.rs/zustand/getting-started/introduction)
- [TanStack Query DevTools](https://tanstack.com/query/latest/docs/react/devtools)
- [Redux DevTools](https://github.com/reduxjs/redux-devtools)

---

**Lưu Ý**: Không có one-size-fits-all solution. Chọn tool phù hợp với use case cụ thể của bạn!
