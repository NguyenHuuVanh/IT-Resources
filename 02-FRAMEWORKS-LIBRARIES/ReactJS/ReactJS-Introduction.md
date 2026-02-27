# ReactJS - Giới Thiệu và Đặc Điểm

## Mục Lục

1. [ReactJS là gì?](#reactjs-là-gì)
2. [Lịch Sử và Phát Triển](#lịch-sử-và-phát-triển)
3. [Đặc Điểm Chính](#đặc-điểm-chính)
4. [Virtual DOM](#virtual-dom)
5. [Component-Based Architecture](#component-based-architecture)
6. [JSX](#jsx)
7. [One-Way Data Flow](#one-way-data-flow)
8. [React Ecosystem](#react-ecosystem)
9. [Ưu và Nhược Điểm](#ưu-và-nhược-điểm)
10. [Khi Nào Nên Dùng React](#khi-nào-nên-dùng-react)

---

## ReactJS là gì?

### Định Nghĩa

**React** (hay React.js, ReactJS) là một JavaScript library mã nguồn mở do Facebook phát triển, dùng để xây dựng user interfaces (UI), đặc biệt là Single Page Applications (SPA).

```
React = Library for building UI
      ≠ Framework (như Angular, Vue)
```

### Triết Lý Cốt Lõi

React tập trung vào:

- **Declarative**: Mô tả UI nên trông như thế nào, React lo việc update
- **Component-Based**: Chia UI thành các component độc lập, tái sử dụng
- **Learn Once, Write Anywhere**: Dùng React cho web, mobile (React Native), desktop

### React vs Framework

```javascript
// React: Library (chỉ lo UI)
import React from "react";
import ReactDOM from "react-dom";

function App() {
  return <h1>Hello React</h1>;
}

ReactDOM.render(<App />, document.getElementById("root"));

// Cần thêm:
// - Routing: React Router
// - State Management: Redux, Zustand
// - HTTP Client: Axios, Fetch
// - Form Handling: Formik, React Hook Form

// Angular: Framework (all-in-one)
// - Routing: Built-in
// - State Management: RxJS
// - HTTP Client: Built-in
// - Form Handling: Built-in
```

---

## Lịch Sử và Phát Triển

### Timeline

```
2011: Jordan Walke tạo FaxJS (prototype của React)
      ↓
2012: React được dùng trong Facebook News Feed
      ↓
2013: React được open-source tại JSConf US
      ↓
2015: React Native ra mắt (mobile development)
      ↓
2016: React 15 - Cải thiện performance
      ↓
2017: React 16 (Fiber) - Rewrite core algorithm
      ↓
2018: React 16.8 - Hooks ra mắt (game changer!)
      ↓
2020: React 17 - No new features, focus on upgrades
      ↓
2022: React 18 - Concurrent features, Suspense
      ↓
2024: React 19 - React Compiler, Server Components
```

### Các Mốc Quan Trọng

**React Hooks (2018)**

- Thay đổi cách viết React components
- Không cần class components nữa
- useState, useEffect, custom hooks

**Concurrent Mode (2022)**

- Render có thể bị interrupt
- Automatic batching
- Transitions
- Suspense for data fetching

**React Server Components (2023-2024)**

- Components render trên server
- Zero JavaScript to client
- Better performance

---

## Đặc Điểm Chính

### 1. Declarative Programming

**Imperative (Vanilla JS)**:

```javascript
// Mô tả CÁC BƯỚC để làm gì
const button = document.createElement("button");
button.textContent = "Click me";
button.addEventListener("click", () => {
  const div = document.createElement("div");
  div.textContent = "Hello!";
  document.body.appendChild(div);
});
document.body.appendChild(button);
```

**Declarative (React)**:

```javascript
// Mô tả KẾT QUẢ muốn có
function App() {
  const [show, setShow] = useState(false);

  return (
    <div>
      <button onClick={() => setShow(true)}>Click me</button>
      {show && <div>Hello!</div>}
    </div>
  );
}
```

### 2. Component-Based

Chia UI thành các component nhỏ, độc lập:

```javascript
// Component nhỏ, tái sử dụng
function Button({ onClick, children }) {
  return (
    <button className="btn" onClick={onClick}>
      {children}
    </button>
  );
}

function Card({ title, content }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{content}</p>
    </div>
  );
}

// Component lớn, kết hợp các component nhỏ
function Dashboard() {
  return (
    <div>
      <Card title="Users" content="1,234 users" />
      <Card title="Revenue" content="$12,345" />
      <Button onClick={() => alert("Refresh")}>Refresh</Button>
    </div>
  );
}
```

### 3. Virtual DOM

React sử dụng Virtual DOM để tối ưu performance:

```
User Action
    ↓
State Change
    ↓
React creates new Virtual DOM
    ↓
Compare with old Virtual DOM (Diffing)
    ↓
Calculate minimal changes needed
    ↓
Update Real DOM (Reconciliation)
    ↓
Browser re-renders only changed parts
```

### 4. Unidirectional Data Flow

Data chỉ chảy một chiều: từ parent → child

```javascript
// ✅ GOOD: Data flows down
function Parent() {
  const [data, setData] = useState("Hello");

  return <Child data={data} />;
}

function Child({ data }) {
  return <div>{data}</div>;
}

// ❌ BAD: Child cannot directly modify parent's state
// Child phải gọi callback từ parent
function Parent() {
  const [count, setCount] = useState(0);

  return <Child count={count} onIncrement={() => setCount(count + 1)} />;
}

function Child({ count, onIncrement }) {
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={onIncrement}>Increment</button>
    </div>
  );
}
```

### 5. JSX (JavaScript XML)

Syntax extension cho phép viết HTML-like code trong JavaScript:

```javascript
// JSX
const element = (
  <div className="container">
    <h1>Hello, {name}!</h1>
    <p>Welcome to React</p>
  </div>
);

// Compiled to JavaScript
const element = React.createElement(
  "div",
  { className: "container" },
  React.createElement("h1", null, "Hello, ", name, "!"),
  React.createElement("p", null, "Welcome to React"),
);
```

### 6. Reusability & Composition

```javascript
// Reusable components
function Avatar({ user }) {
  return <img src={user.avatar} alt={user.name} />;
}

function UserInfo({ user }) {
  return (
    <div>
      <Avatar user={user} />
      <h3>{user.name}</h3>
    </div>
  );
}

// Composition
function Comment({ comment }) {
  return (
    <div className="comment">
      <UserInfo user={comment.author} />
      <p>{comment.text}</p>
      <span>{comment.date}</span>
    </div>
  );
}
```

### 7. React Hooks

Hooks cho phép sử dụng state và lifecycle trong function components:

```javascript
import { useState, useEffect, useContext, useReducer } from "react";

function Example() {
  // State management
  const [count, setCount] = useState(0);

  // Side effects
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  // Context
  const theme = useContext(ThemeContext);

  // Complex state logic
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

---

## Virtual DOM

### Virtual DOM là gì?

Virtual DOM là một representation (bản sao) của Real DOM trong memory, dưới dạng JavaScript object.

```javascript
// Real DOM
<div id="app">
  <h1>Hello</h1>
  <p>World</p>
</div>

// Virtual DOM (simplified)
{
  type: 'div',
  props: { id: 'app' },
  children: [
    { type: 'h1', props: {}, children: ['Hello'] },
    { type: 'p', props: {}, children: ['World'] }
  ]
}
```

### Tại Sao Cần Virtual DOM?

**Problem**: Manipulating Real DOM is slow

```javascript
// Slow: Direct DOM manipulation
for (let i = 0; i < 1000; i++) {
  const div = document.createElement("div");
  div.textContent = `Item ${i}`;
  document.body.appendChild(div); // Reflow/repaint 1000 times!
}
```

**Solution**: Virtual DOM batches updates

```javascript
// Fast: React batches updates
function List() {
  const items = Array.from({ length: 1000 }, (_, i) => i);

  return (
    <div>
      {items.map((i) => (
        <div key={i}>Item {i}</div>
      ))}
    </div>
  );
}
// React updates Real DOM once!
```

### Virtual DOM Workflow

```
1. Initial Render
   ├─ Create Virtual DOM tree
   ├─ Create Real DOM from Virtual DOM
   └─ Display on screen

2. State Changes
   ├─ Create NEW Virtual DOM tree
   ├─ Diff with OLD Virtual DOM (Reconciliation)
   │  ├─ Find differences
   │  └─ Calculate minimal changes
   ├─ Batch updates
   └─ Update Real DOM efficiently

3. Browser Re-renders
   └─ Only changed parts
```

### Reconciliation Algorithm

```javascript
// Example: Update list

// Old Virtual DOM
[
  { id: 1, text: "Apple" },
  { id: 2, text: "Banana" },
  { id: 3, text: "Cherry" },
][
  // New Virtual DOM
  ({ id: 1, text: "Apple" },
  { id: 2, text: "Blueberry" }, // Changed
  { id: 3, text: "Cherry" },
  { id: 4, text: "Date" }) // Added
];

// React's Diff Algorithm
// ├─ id: 1 → No change
// ├─ id: 2 → Update text
// ├─ id: 3 → No change
// └─ id: 4 → Insert new element

// Real DOM updates
// ├─ Update text of element with id=2
// └─ Append new element with id=4
```

### Keys trong Lists

```javascript
// ❌ BAD: No keys
{
  items.map((item) => <div>{item.name}</div>);
}
// React re-renders all items on change

// ✅ GOOD: With keys
{
  items.map((item) => <div key={item.id}>{item.name}</div>);
}
// React only updates changed items

// ❌ BAD: Index as key (when list can change)
{
  items.map((item, index) => <div key={index}>{item.name}</div>);
}
// Can cause bugs when reordering
```

### Performance Benefits

```javascript
// Without Virtual DOM (Direct DOM manipulation)
// Time: ~100ms for 1000 elements

// With Virtual DOM (React)
// Time: ~10ms for 1000 elements
// 10x faster!

// Benchmark
console.time("Direct DOM");
for (let i = 0; i < 1000; i++) {
  document.body.appendChild(document.createElement("div"));
}
console.timeEnd("Direct DOM"); // ~100ms

console.time("React");
ReactDOM.render(
  <div>
    {Array.from({ length: 1000 }, (_, i) => (
      <div key={i}>Item {i}</div>
    ))}
  </div>,
  document.getElementById("root"),
);
console.timeEnd("React"); // ~10ms
```

---

## Component-Based Architecture

### Component là gì?

Component là building block của React app, giống như LEGO blocks.

```javascript
// Component = Function that returns JSX
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// Component with props
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Component with state
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### Component Hierarchy

```
App
├── Header
│   ├── Logo
│   ├── Navigation
│   │   ├── NavItem
│   │   ├── NavItem
│   │   └── NavItem
│   └── UserMenu
│       ├── Avatar
│       └── Dropdown
├── Main
│   ├── Sidebar
│   │   ├── Filter
│   │   └── Categories
│   └── Content
│       ├── ProductList
│       │   ├── ProductCard
│       │   ├── ProductCard
│       │   └── ProductCard
│       └── Pagination
└── Footer
    ├── Links
    └── Copyright
```

### Types of Components

**1. Presentational Components (Dumb Components)**

```javascript
// Chỉ nhận props và render UI
function Button({ onClick, children, variant = "primary" }) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}

function Card({ title, content, image }) {
  return (
    <div className="card">
      <img src={image} alt={title} />
      <h3>{title}</h3>
      <p>{content}</p>
    </div>
  );
}
```

**2. Container Components (Smart Components)**

```javascript
// Quản lý state và logic
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUsers().then((data) => {
      setUsers(data);
      setLoading(false);
    });
  }, []);

  if (loading) return <Spinner />;

  return <UserList users={users} />;
}
```

### Component Composition

```javascript
// Composition over Inheritance
function Dialog({ title, children }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      <div className="dialog-content">{children}</div>
    </div>
  );
}

// Reuse with different content
function WelcomeDialog() {
  return (
    <Dialog title="Welcome">
      <p>Thank you for visiting!</p>
    </Dialog>
  );
}

function ConfirmDialog() {
  return (
    <Dialog title="Confirm">
      <p>Are you sure?</p>
      <button>Yes</button>
      <button>No</button>
    </Dialog>
  );
}
```

### Props vs State

```javascript
// Props: Data từ parent
function Child({ name, age }) {
  // Cannot modify props
  // name = "New Name"; // ❌ Error!

  return (
    <div>
      {name} is {age} years old
    </div>
  );
}

// State: Data của component
function Parent() {
  const [name, setName] = useState("John");

  // Can modify state
  const changeName = () => setName("Jane");

  return (
    <div>
      <Child name={name} age={30} />
      <button onClick={changeName}>Change Name</button>
    </div>
  );
}
```

### Component Lifecycle

```javascript
// Function Component với Hooks
function Component() {
  // Mount
  useEffect(() => {
    console.log("Component mounted");

    // Cleanup (Unmount)
    return () => {
      console.log("Component unmounted");
    };
  }, []);

  // Update (when dependency changes)
  useEffect(() => {
    console.log("Dependency changed");
  }, [dependency]);

  // Every render
  useEffect(() => {
    console.log("Component rendered");
  });

  return <div>Component</div>;
}
```
