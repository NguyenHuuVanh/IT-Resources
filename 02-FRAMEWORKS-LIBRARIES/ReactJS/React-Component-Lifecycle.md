# React Component Lifecycle - Hướng dẫn Toàn diện

## Mục lục

1. [Tổng quan về Component Lifecycle](#1-tổng-quan-về-component-lifecycle)
2. [Class Component Lifecycle](#2-class-component-lifecycle)
3. [Functional Component với Hooks](#3-functional-component-với-hooks)
4. [useEffect chi tiết](#4-useeffect-chi-tiết)
5. [Các Hooks liên quan đến Lifecycle](#5-các-hooks-liên-quan-đến-lifecycle)
6. [So sánh Class vs Functional](#6-so-sánh-class-vs-functional)
7. [Common Patterns và Use Cases](#7-common-patterns-và-use-cases)
8. [Performance Optimization](#8-performance-optimization)
9. [Best Practices](#9-best-practices)
10. [Debugging Lifecycle](#10-debugging-lifecycle)

---

## 1. Tổng quan về Component Lifecycle

### Lifecycle là gì?

**Định nghĩa:** Lifecycle (vòng đời) của component là các giai đoạn mà component trải qua từ khi được tạo ra, cập nhật, cho đến khi bị hủy khỏi DOM.

### Ba giai đoạn chính

```
┌─────────────────────────────────────────────────────────────────┐
│                  REACT COMPONENT LIFECYCLE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  MOUNTING   │───>│  UPDATING   │───>│ UNMOUNTING  │         │
│  │  (Sinh ra)  │    │ (Cập nhật)  │    │  (Hủy bỏ)   │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│        │                  │                  │                  │
│        ▼                  ▼                  ▼                  │
│  • constructor      • New props        • Cleanup               │
│  • render           • setState         • Remove listeners      │
│  • componentDidMount• forceUpdate      • Cancel requests       │
│                     • render                                    │
│                     • componentDidUpdate                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Tại sao cần hiểu Lifecycle?

- **Fetch data** đúng thời điểm
- **Subscribe/Unsubscribe** events
- **Cleanup** resources (timers, listeners)
- **Optimize** performance
- **Debug** issues hiệu quả

---

## 2. Class Component Lifecycle

### Lifecycle Methods Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    MOUNTING PHASE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  constructor(props)                                              │
│       │                                                          │
│       ▼                                                          │
│  static getDerivedStateFromProps(props, state)                  │
│       │                                                          │
│       ▼                                                          │
│  render()                                                        │
│       │                                                          │
│       ▼                                                          │
│  componentDidMount() ◄─── Side effects, subscriptions           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    UPDATING PHASE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  New Props / setState() / forceUpdate()                         │
│       │                                                          │
│       ▼                                                          │
│  static getDerivedStateFromProps(props, state)                  │
│       │                                                          │
│       ▼                                                          │
│  shouldComponentUpdate(nextProps, nextState) ◄─── Performance   │
│       │                                                          │
│       ▼ (if true)                                               │
│  render()                                                        │
│       │                                                          │
│       ▼                                                          │
│  getSnapshotBeforeUpdate(prevProps, prevState)                  │
│       │                                                          │
│       ▼                                                          │
│  componentDidUpdate(prevProps, prevState, snapshot)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   UNMOUNTING PHASE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  componentWillUnmount() ◄─── Cleanup                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Chi tiết từng Lifecycle Method

```jsx
import React, { Component } from "react";

class LifecycleDemo extends Component {
  // ═══════════════════════════════════════════════════════════
  // MOUNTING PHASE
  // ═══════════════════════════════════════════════════════════

  // 1. Constructor - Khởi tạo
  constructor(props) {
    super(props); // BẮT BUỘC gọi trước khi dùng this

    console.log("1. Constructor");

    // ✅ Khởi tạo state
    this.state = {
      count: 0,
      data: null,
      loading: true,
    };

    // ✅ Bind methods
    this.handleClick = this.handleClick.bind(this);

    // ❌ KHÔNG gọi setState() ở đây
    // ❌ KHÔNG gọi side effects (fetch, subscriptions)
  }

  // 2. getDerivedStateFromProps - Sync state với props
  static getDerivedStateFromProps(props, state) {
    console.log("2. getDerivedStateFromProps");

    // Hiếm khi dùng - chỉ khi state phụ thuộc vào props
    // Return object để update state, hoặc null
    if (props.resetCount && state.count !== 0) {
      return { count: 0 };
    }
    return null;
  }

  // 3. render - Render UI
  render() {
    console.log("3. Render");

    // ✅ Return JSX
    // ❌ KHÔNG gọi setState() ở đây
    // ❌ KHÔNG side effects
    // Phải là PURE function

    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }

  // 4. componentDidMount - Sau khi mount vào DOM
  componentDidMount() {
    console.log("4. componentDidMount");

    // ✅ Fetch data
    this.fetchData();

    // ✅ Setup subscriptions
    this.subscription = eventEmitter.subscribe(this.handleEvent);

    // ✅ Setup timers
    this.timer = setInterval(() => {
      console.log("Tick");
    }, 1000);

    // ✅ DOM manipulation (nếu cần)
    this.inputRef.focus();

    // ✅ Gọi setState() được
    // Sẽ trigger re-render nhưng user không thấy intermediate state
  }

  // ═══════════════════════════════════════════════════════════
  // UPDATING PHASE
  // ═══════════════════════════════════════════════════════════

  // 5. shouldComponentUpdate - Quyết định có re-render không
  shouldComponentUpdate(nextProps, nextState) {
    console.log("5. shouldComponentUpdate");

    // Return true để re-render, false để skip
    // Dùng để optimize performance

    // Ví dụ: Chỉ re-render khi count thay đổi
    if (this.state.count !== nextState.count) {
      return true;
    }

    if (this.props.title !== nextProps.title) {
      return true;
    }

    return false;
  }

  // 6. getSnapshotBeforeUpdate - Capture info trước khi DOM update
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log("6. getSnapshotBeforeUpdate");

    // Chạy sau render, trước khi DOM được update
    // Return value sẽ được pass vào componentDidUpdate

    // Ví dụ: Lưu scroll position
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  // 7. componentDidUpdate - Sau khi update xong
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log("7. componentDidUpdate");

    // ✅ So sánh props/state trước khi làm gì đó
    if (prevProps.userId !== this.props.userId) {
      this.fetchData(this.props.userId);
    }

    // ✅ Sử dụng snapshot
    if (snapshot !== null) {
      const list = this.listRef;
      list.scrollTop = list.scrollHeight - snapshot;
    }

    // ⚠️ CẨN THẬN: Gọi setState() phải có điều kiện
    // Nếu không sẽ gây infinite loop
    if (prevState.count !== this.state.count) {
      // OK - có điều kiện
      this.logAnalytics(this.state.count);
    }
  }

  // ═══════════════════════════════════════════════════════════
  // UNMOUNTING PHASE
  // ═══════════════════════════════════════════════════════════

  // 8. componentWillUnmount - Trước khi bị remove khỏi DOM
  componentWillUnmount() {
    console.log("8. componentWillUnmount");

    // ✅ Cleanup subscriptions
    if (this.subscription) {
      this.subscription.unsubscribe();
    }

    // ✅ Clear timers
    if (this.timer) {
      clearInterval(this.timer);
    }

    // ✅ Cancel pending requests
    if (this.abortController) {
      this.abortController.abort();
    }

    // ✅ Remove event listeners
    window.removeEventListener("resize", this.handleResize);

    // ❌ KHÔNG gọi setState() - component sắp bị hủy
  }

  // ═══════════════════════════════════════════════════════════
  // ERROR HANDLING
  // ═══════════════════════════════════════════════════════════

  // 9. componentDidCatch - Bắt errors từ children
  componentDidCatch(error, errorInfo) {
    console.log("9. componentDidCatch");

    // Log error
    logErrorToService(error, errorInfo);

    // Update state để show fallback UI
    this.setState({ hasError: true });
  }

  // 10. getDerivedStateFromError - Update state khi có error
  static getDerivedStateFromError(error) {
    // Return state để show fallback UI
    return { hasError: true };
  }

  // ═══════════════════════════════════════════════════════════
  // HELPER METHODS
  // ═══════════════════════════════════════════════════════════

  handleClick() {
    this.setState((prevState) => ({
      count: prevState.count + 1,
    }));
  }

  async fetchData() {
    this.abortController = new AbortController();

    try {
      const response = await fetch("/api/data", {
        signal: this.abortController.signal,
      });
      const data = await response.json();
      this.setState({ data, loading: false });
    } catch (error) {
      if (error.name !== "AbortError") {
        this.setState({ error, loading: false });
      }
    }
  }
}

export default LifecycleDemo;
```

### Error Boundary Component

```jsx
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error("Error caught:", error);
    console.error("Error info:", errorInfo.componentStack);

    // Log to error reporting service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>Try again</button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>;
```

---

## 3. Functional Component với Hooks

### Hooks thay thế Lifecycle Methods

```
┌─────────────────────────────────────────────────────────────────┐
│           CLASS METHODS  →  HOOKS EQUIVALENT                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  constructor          →  useState (khởi tạo state)              │
│                          useRef (instance variables)            │
│                                                                  │
│  componentDidMount    →  useEffect(() => {}, [])                │
│                                                                  │
│  componentDidUpdate   →  useEffect(() => {}, [deps])            │
│                                                                  │
│  componentWillUnmount →  useEffect(() => {                      │
│                            return () => { cleanup }             │
│                          }, [])                                  │
│                                                                  │
│  shouldComponentUpdate → React.memo() + useMemo/useCallback     │
│                                                                  │
│  getDerivedStateFromProps → useState + useEffect                │
│                             hoặc tính toán trong render         │
│                                                                  │
│  getSnapshotBeforeUpdate → useLayoutEffect (gần tương đương)    │
│                                                                  │
│  componentDidCatch    →  Không có hook tương đương              │
│                          Vẫn cần Error Boundary class           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Functional Component Lifecycle Flow

```
┌─────────────────────────────────────────────────────────────────┐
│              FUNCTIONAL COMPONENT LIFECYCLE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Component được gọi (render)                                     │
│       │                                                          │
│       ▼                                                          │
│  useState, useRef khởi tạo (chỉ lần đầu)                        │
│       │                                                          │
│       ▼                                                          │
│  useMemo, useCallback tính toán                                 │
│       │                                                          │
│       ▼                                                          │
│  Return JSX                                                      │
│       │                                                          │
│       ▼                                                          │
│  React commit changes to DOM                                     │
│       │                                                          │
│       ▼                                                          │
│  useLayoutEffect chạy (đồng bộ, block paint)                    │
│       │                                                          │
│       ▼                                                          │
│  Browser paint                                                   │
│       │                                                          │
│       ▼                                                          │
│  useEffect chạy (bất đồng bộ, sau paint)                        │
│                                                                  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                  │
│  Khi UPDATE (props/state thay đổi):                             │
│       │                                                          │
│       ▼                                                          │
│  Component được gọi lại                                          │
│       │                                                          │
│       ▼                                                          │
│  useEffect cleanup chạy (nếu deps thay đổi)                     │
│       │                                                          │
│       ▼                                                          │
│  useEffect mới chạy                                              │
│                                                                  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                  │
│  Khi UNMOUNT:                                                    │
│       │                                                          │
│       ▼                                                          │
│  Tất cả useEffect cleanup chạy                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Basic Functional Component với Hooks

```jsx
import React, { useState, useEffect, useRef, useCallback, useMemo } from "react";

function LifecycleDemo({ userId, title }) {
  // ═══════════════════════════════════════════════════════════
  // STATE & REFS (thay thế constructor)
  // ═══════════════════════════════════════════════════════════

  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // useRef cho instance variables (không trigger re-render)
  const renderCount = useRef(0);
  const abortControllerRef = useRef(null);
  const previousUserId = useRef(userId);

  // Đếm số lần render
  renderCount.current += 1;
  console.log(`Render #${renderCount.current}`);

  // ═══════════════════════════════════════════════════════════
  // COMPUTED VALUES (useMemo)
  // ═══════════════════════════════════════════════════════════

  const expensiveValue = useMemo(() => {
    console.log("Computing expensive value...");
    return data?.items?.reduce((sum, item) => sum + item.value, 0) || 0;
  }, [data]);

  // ═══════════════════════════════════════════════════════════
  // CALLBACKS (useCallback)
  // ═══════════════════════════════════════════════════════════

  const handleClick = useCallback(() => {
    setCount((prev) => prev + 1);
  }, []);

  const fetchData = useCallback(async (id) => {
    // Cancel previous request
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();

    setLoading(true);
    setError(null);

    try {
      const response = await fetch(`/api/users/${id}`, {
        signal: abortControllerRef.current.signal,
      });

      if (!response.ok) throw new Error("Failed to fetch");

      const result = await response.json();
      setData(result);
    } catch (err) {
      if (err.name !== "AbortError") {
        setError(err.message);
      }
    } finally {
      setLoading(false);
    }
  }, []);

  // ═══════════════════════════════════════════════════════════
  // EFFECTS
  // ═══════════════════════════════════════════════════════════

  // Effect 1: componentDidMount equivalent
  useEffect(() => {
    console.log("Effect: Component mounted");

    // Setup
    fetchData(userId);

    // Cleanup function = componentWillUnmount
    return () => {
      console.log("Cleanup: Component will unmount");
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []); // Empty deps = chỉ chạy 1 lần khi mount

  // Effect 2: componentDidUpdate for userId
  useEffect(() => {
    // Skip first render
    if (previousUserId.current === userId) {
      previousUserId.current = userId;
      return;
    }

    console.log(`Effect: userId changed from ${previousUserId.current} to ${userId}`);
    previousUserId.current = userId;

    fetchData(userId);
  }, [userId, fetchData]);

  // Effect 3: Run on every render (không có deps array)
  useEffect(() => {
    console.log("Effect: Runs after every render");
  });

  // Effect 4: Subscription example
  useEffect(() => {
    const handleResize = () => {
      console.log("Window resized");
    };

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  // Effect 5: Timer example
  useEffect(() => {
    const timer = setInterval(() => {
      console.log("Tick");
    }, 1000);

    return () => {
      clearInterval(timer);
    };
  }, []);

  // Effect 6: Document title
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  // ═══════════════════════════════════════════════════════════
  // RENDER
  // ═══════════════════════════════════════════════════════════

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>{title}</h1>
      <p>Count: {count}</p>
      <p>Total value: {expensiveValue}</p>
      <button onClick={handleClick}>Increment</button>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}

export default LifecycleDemo;
```

---

## 4. useEffect chi tiết

### Cú pháp và Dependencies

```jsx
useEffect(
  () => {
    // Effect code

    return () => {
      // Cleanup code (optional)
    };
  },
  [dependencies] // Dependency array (optional)
);
```

### Các trường hợp Dependencies

```jsx
// ═══════════════════════════════════════════════════════════
// CASE 1: Không có dependency array
// Chạy SAU MỖI RENDER
// ═══════════════════════════════════════════════════════════
useEffect(() => {
  console.log("Runs after every render");
});

// ═══════════════════════════════════════════════════════════
// CASE 2: Empty dependency array []
// Chạy 1 LẦN sau mount, cleanup khi unmount
// = componentDidMount + componentWillUnmount
// ═══════════════════════════════════════════════════════════
useEffect(() => {
  console.log("Runs once after mount");

  return () => {
    console.log("Runs once before unmount");
  };
}, []);

// ═══════════════════════════════════════════════════════════
// CASE 3: Có dependencies
// Chạy khi mount VÀ khi dependencies thay đổi
// ═══════════════════════════════════════════════════════════
useEffect(() => {
  console.log(`userId changed to: ${userId}`);
  fetchUser(userId);

  return () => {
    console.log(`Cleanup for userId: ${userId}`);
  };
}, [userId]);

// ═══════════════════════════════════════════════════════════
// CASE 4: Multiple dependencies
// Chạy khi BẤT KỲ dependency nào thay đổi
// ═══════════════════════════════════════════════════════════
useEffect(() => {
  console.log("userId or postId changed");
  fetchData(userId, postId);
}, [userId, postId]);
```

### Cleanup Function chi tiết

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    console.log(`Connecting to room: ${roomId}`);

    // Setup connection
    const connection = createConnection(roomId);
    connection.connect();

    // Subscribe to messages
    connection.on("message", (msg) => {
      setMessages((prev) => [...prev, msg]);
    });

    // Cleanup function
    return () => {
      console.log(`Disconnecting from room: ${roomId}`);
      connection.disconnect();
    };
  }, [roomId]);

  // Khi roomId thay đổi:
  // 1. Cleanup của effect cũ chạy (disconnect room cũ)
  // 2. Effect mới chạy (connect room mới)

  return <MessageList messages={messages} />;
}
```

### Thứ tự thực thi Effects

```jsx
function Parent() {
  useEffect(() => {
    console.log("Parent effect");
    return () => console.log("Parent cleanup");
  });

  return <Child />;
}

function Child() {
  useEffect(() => {
    console.log("Child effect");
    return () => console.log("Child cleanup");
  });

  return <div>Child</div>;
}

// Mount:
// 1. Child effect
// 2. Parent effect

// Update:
// 1. Child cleanup
// 2. Parent cleanup
// 3. Child effect
// 4. Parent effect

// Unmount:
// 1. Child cleanup
// 2. Parent cleanup
```

### Common Mistakes với useEffect

```jsx
// ═══════════════════════════════════════════════════════════
// ❌ MISTAKE 1: Missing dependencies
// ═══════════════════════════════════════════════════════════
function BadExample({ userId }) {
  const [user, setUser] = useState(null);

  // ❌ userId không có trong deps → stale closure
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []); // ESLint sẽ warning

  // ✅ Correct
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
}

// ═══════════════════════════════════════════════════════════
// ❌ MISTAKE 2: Object/Array trong dependencies
// ═══════════════════════════════════════════════════════════
function BadExample({ options }) {
  // ❌ options là object mới mỗi render → infinite loop
  useEffect(() => {
    doSomething(options);
  }, [options]);
}

// ✅ Solution 1: Destructure
function GoodExample({ options }) {
  const { page, limit } = options;

  useEffect(() => {
    doSomething({ page, limit });
  }, [page, limit]);
}

// ✅ Solution 2: useMemo
function GoodExample({ page, limit }) {
  const options = useMemo(() => ({ page, limit }), [page, limit]);

  useEffect(() => {
    doSomething(options);
  }, [options]);
}

// ✅ Solution 3: JSON.stringify (simple cases)
function GoodExample({ options }) {
  const optionsString = JSON.stringify(options);

  useEffect(() => {
    doSomething(JSON.parse(optionsString));
  }, [optionsString]);
}

// ═══════════════════════════════════════════════════════════
// ❌ MISTAKE 3: Function trong dependencies
// ═══════════════════════════════════════════════════════════
function BadExample({ userId }) {
  // ❌ fetchData được tạo mới mỗi render
  const fetchData = () => {
    return fetch(`/api/users/${userId}`);
  };

  useEffect(() => {
    fetchData().then(/* ... */);
  }, [fetchData]); // Infinite loop!
}

// ✅ Solution 1: useCallback
function GoodExample({ userId }) {
  const fetchData = useCallback(() => {
    return fetch(`/api/users/${userId}`);
  }, [userId]);

  useEffect(() => {
    fetchData().then(/* ... */);
  }, [fetchData]);
}

// ✅ Solution 2: Move function inside effect
function GoodExample({ userId }) {
  useEffect(() => {
    const fetchData = () => {
      return fetch(`/api/users/${userId}`);
    };

    fetchData().then(/* ... */);
  }, [userId]);
}

// ═══════════════════════════════════════════════════════════
// ❌ MISTAKE 4: Async function trực tiếp
// ═══════════════════════════════════════════════════════════
// ❌ useEffect không thể return Promise
useEffect(async () => {
  const data = await fetchData();
  setData(data);
}, []);

// ✅ Correct: Define async function inside
useEffect(() => {
  const loadData = async () => {
    const data = await fetchData();
    setData(data);
  };

  loadData();
}, []);

// ✅ Hoặc dùng IIFE
useEffect(() => {
  (async () => {
    const data = await fetchData();
    setData(data);
  })();
}, []);
```

---

## 5. Các Hooks liên quan đến Lifecycle

### useLayoutEffect

```jsx
// useLayoutEffect vs useEffect
// ═══════════════════════════════════════════════════════════

/*
 * useEffect:
 * - Chạy SAU khi browser paint
 * - Non-blocking
 * - Dùng cho hầu hết cases
 *
 * useLayoutEffect:
 * - Chạy TRƯỚC khi browser paint
 * - Blocking (đồng bộ)
 * - Dùng khi cần đo DOM hoặc prevent flicker
 */

import { useLayoutEffect, useEffect, useRef, useState } from "react";

function TooltipExample() {
  const [tooltipHeight, setTooltipHeight] = useState(0);
  const tooltipRef = useRef(null);

  // ✅ useLayoutEffect để đo DOM trước khi paint
  // Tránh flicker khi tooltip xuất hiện
  useLayoutEffect(() => {
    if (tooltipRef.current) {
      const height = tooltipRef.current.getBoundingClientRect().height;
      setTooltipHeight(height);
    }
  }, []);

  return (
    <div ref={tooltipRef} style={{ top: -tooltipHeight }}>
      Tooltip content
    </div>
  );
}

// Animation example
function AnimatedComponent() {
  const boxRef = useRef(null);

  // ❌ useEffect có thể gây flicker
  useEffect(() => {
    boxRef.current.style.transform = "translateX(100px)";
  }, []);

  // ✅ useLayoutEffect - smooth animation
  useLayoutEffect(() => {
    boxRef.current.style.transform = "translateX(100px)";
  }, []);

  return <div ref={boxRef}>Animated Box</div>;
}
```

### useRef cho Lifecycle

```jsx
function useRef_Examples() {
  // 1. Lưu giá trị previous
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    prevCountRef.current = count;
  });

  const prevCount = prevCountRef.current;
  console.log(`Current: ${count}, Previous: ${prevCount}`);

  // 2. Track mounted state
  const isMountedRef = useRef(true);

  useEffect(() => {
    return () => {
      isMountedRef.current = false;
    };
  }, []);

  const fetchData = async () => {
    const data = await fetch("/api/data");
    // Chỉ setState nếu component còn mounted
    if (isMountedRef.current) {
      setData(data);
    }
  };

  // 3. Store interval/timeout ID
  const intervalRef = useRef(null);

  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(intervalRef.current);
  };

  // 4. Track first render
  const isFirstRender = useRef(true);

  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }
    // Code chỉ chạy từ lần render thứ 2
    console.log("Not first render");
  });
}
```

### Custom Hooks cho Lifecycle

```jsx
// ═══════════════════════════════════════════════════════════
// usePrevious - Lưu giá trị trước đó
// ═══════════════════════════════════════════════════════════
function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>
        Current: {count}, Previous: {prevCount}
      </p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════
// useMount - Chỉ chạy khi mount
// ═══════════════════════════════════════════════════════════
function useMount(callback) {
  useEffect(() => {
    callback();
  }, []); // eslint-disable-line react-hooks/exhaustive-deps
}

// Usage
useMount(() => {
  console.log("Component mounted");
  initializeAnalytics();
});

// ═══════════════════════════════════════════════════════════
// useUnmount - Chỉ chạy khi unmount
// ═══════════════════════════════════════════════════════════
function useUnmount(callback) {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  useEffect(() => {
    return () => {
      callbackRef.current();
    };
  }, []);
}

// Usage
useUnmount(() => {
  console.log("Component will unmount");
  saveFormDraft();
});

// ═══════════════════════════════════════════════════════════
// useUpdateEffect - Chỉ chạy khi update (skip mount)
// ═══════════════════════════════════════════════════════════
function useUpdateEffect(callback, deps) {
  const isFirstRender = useRef(true);

  useEffect(() => {
    if (isFirstRender.current) {
      isFirstRender.current = false;
      return;
    }

    return callback();
  }, deps); // eslint-disable-line react-hooks/exhaustive-deps
}

// Usage
useUpdateEffect(() => {
  console.log("Count updated (not on mount)");
  sendAnalytics("count_changed", count);
}, [count]);

// ═══════════════════════════════════════════════════════════
// useIsMounted - Check component còn mounted không
// ═══════════════════════════════════════════════════════════
function useIsMounted() {
  const isMounted = useRef(false);

  useEffect(() => {
    isMounted.current = true;

    return () => {
      isMounted.current = false;
    };
  }, []);

  return useCallback(() => isMounted.current, []);
}

// Usage
function AsyncComponent() {
  const [data, setData] = useState(null);
  const isMounted = useIsMounted();

  useEffect(() => {
    fetchData().then((result) => {
      // Chỉ setState nếu còn mounted
      if (isMounted()) {
        setData(result);
      }
    });
  }, [isMounted]);
}

// ═══════════════════════════════════════════════════════════
// useEffectOnce - Effect chỉ chạy 1 lần
// ═══════════════════════════════════════════════════════════
function useEffectOnce(effect) {
  useEffect(effect, []);
}

// ═══════════════════════════════════════════════════════════
// useInterval - setInterval với cleanup tự động
// ═══════════════════════════════════════════════════════════
function useInterval(callback, delay) {
  const savedCallback = useRef(callback);

  // Cập nhật callback ref
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // Setup interval
  useEffect(() => {
    if (delay === null) return;

    const id = setInterval(() => {
      savedCallback.current();
    }, delay);

    return () => clearInterval(id);
  }, [delay]);
}

// Usage
function Timer() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(true);

  useInterval(
    () => setCount((c) => c + 1),
    isRunning ? 1000 : null // null để pause
  );

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setIsRunning(!isRunning)}>{isRunning ? "Pause" : "Resume"}</button>
    </div>
  );
}

// ═══════════════════════════════════════════════════════════
// useTimeout - setTimeout với cleanup
// ═══════════════════════════════════════════════════════════
function useTimeout(callback, delay) {
  const savedCallback = useRef(callback);

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    if (delay === null) return;

    const id = setTimeout(() => {
      savedCallback.current();
    }, delay);

    return () => clearTimeout(id);
  }, [delay]);
}

// ═══════════════════════════════════════════════════════════
// useDebounce - Debounce value
// ═══════════════════════════════════════════════════════════
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState("");
  const debouncedSearch = useDebounce(searchTerm, 500);

  useEffect(() => {
    if (debouncedSearch) {
      searchAPI(debouncedSearch);
    }
  }, [debouncedSearch]);

  return <input value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)} placeholder="Search..." />;
}

// ═══════════════════════════════════════════════════════════
// useWindowEvent - Event listener với cleanup
// ═══════════════════════════════════════════════════════════
function useWindowEvent(event, callback, options) {
  const savedCallback = useRef(callback);

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    const handler = (e) => savedCallback.current(e);

    window.addEventListener(event, handler, options);

    return () => {
      window.removeEventListener(event, handler, options);
    };
  }, [event, options]);
}

// Usage
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);

  useWindowEvent("scroll", () => {
    setScrollY(window.scrollY);
  });

  return <div>Scroll position: {scrollY}</div>;
}
```

---

## 6. So sánh Class vs Functional

### Side-by-side Comparison

```jsx
// ═══════════════════════════════════════════════════════════
// CLASS COMPONENT
// ═══════════════════════════════════════════════════════════
class UserProfile extends Component {
  constructor(props) {
    super(props);
    this.state = {
      user: null,
      loading: true,
      error: null,
    };
  }

  componentDidMount() {
    this.fetchUser();
    window.addEventListener("resize", this.handleResize);
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }

  componentWillUnmount() {
    window.removeEventListener("resize", this.handleResize);
    if (this.abortController) {
      this.abortController.abort();
    }
  }

  handleResize = () => {
    console.log("Window resized");
  };

  fetchUser = async () => {
    this.abortController = new AbortController();
    this.setState({ loading: true });

    try {
      const response = await fetch(`/api/users/${this.props.userId}`, {
        signal: this.abortController.signal,
      });
      const user = await response.json();
      this.setState({ user, loading: false });
    } catch (error) {
      if (error.name !== "AbortError") {
        this.setState({ error: error.message, loading: false });
      }
    }
  };

  render() {
    const { user, loading, error } = this.state;

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;

    return <div>{user?.name}</div>;
  }
}

// ═══════════════════════════════════════════════════════════
// FUNCTIONAL COMPONENT (tương đương)
// ═══════════════════════════════════════════════════════════
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const abortControllerRef = useRef(null);

  // Fetch user
  useEffect(() => {
    const fetchUser = async () => {
      abortControllerRef.current = new AbortController();
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(`/api/users/${userId}`, {
          signal: abortControllerRef.current.signal,
        });
        const data = await response.json();
        setUser(data);
      } catch (err) {
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchUser();

    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [userId]);

  // Window resize listener
  useEffect(() => {
    const handleResize = () => {
      console.log("Window resized");
    };

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return <div>{user?.name}</div>;
}
```

### Khi nào dùng Class vs Functional?

```
┌─────────────────────────────────────────────────────────────────┐
│              CLASS vs FUNCTIONAL COMPONENTS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL COMPONENTS (Khuyến nghị)                            │
│  ✅ Mặc định cho mọi component mới                              │
│  ✅ Code ngắn gọn, dễ đọc                                       │
│  ✅ Dễ test                                                      │
│  ✅ Hooks cho phép reuse logic                                  │
│  ✅ Tương lai của React                                         │
│                                                                  │
│  CLASS COMPONENTS (Khi cần)                                     │
│  ⚠️ Error Boundaries (bắt buộc)                                 │
│  ⚠️ Legacy codebase                                             │
│  ⚠️ getSnapshotBeforeUpdate (hiếm khi cần)                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Common Patterns và Use Cases

### Data Fetching Pattern

```jsx
// ═══════════════════════════════════════════════════════════
// Pattern 1: Basic fetch với loading/error states
// ═══════════════════════════════════════════════════════════
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url, {
          signal: abortController.signal,
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    return () => abortController.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data: users, loading, error } = useFetch("/api/users");

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// ═══════════════════════════════════════════════════════════
// Pattern 2: Fetch với refetch và caching
// ═══════════════════════════════════════════════════════════
function useFetchWithCache(url, options = {}) {
  const { cacheTime = 5 * 60 * 1000 } = options; // 5 minutes default
  const cache = useRef({});

  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null,
  });

  const fetchData = useCallback(
    async (skipCache = false) => {
      // Check cache
      const cached = cache.current[url];
      if (!skipCache && cached && Date.now() - cached.timestamp < cacheTime) {
        setState({ data: cached.data, loading: false, error: null });
        return;
      }

      setState((prev) => ({ ...prev, loading: true, error: null }));

      try {
        const response = await fetch(url);
        const data = await response.json();

        // Update cache
        cache.current[url] = { data, timestamp: Date.now() };

        setState({ data, loading: false, error: null });
      } catch (error) {
        setState({ data: null, loading: false, error: error.message });
      }
    },
    [url, cacheTime]
  );

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  const refetch = useCallback(() => fetchData(true), [fetchData]);

  return { ...state, refetch };
}
```

### Subscription Pattern

```jsx
// ═══════════════════════════════════════════════════════════
// WebSocket subscription
// ═══════════════════════════════════════════════════════════
function useWebSocket(url) {
  const [messages, setMessages] = useState([]);
  const [isConnected, setIsConnected] = useState(false);
  const wsRef = useRef(null);

  useEffect(() => {
    const ws = new WebSocket(url);
    wsRef.current = ws;

    ws.onopen = () => {
      console.log("WebSocket connected");
      setIsConnected(true);
    };

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      setMessages((prev) => [...prev, message]);
    };

    ws.onerror = (error) => {
      console.error("WebSocket error:", error);
    };

    ws.onclose = () => {
      console.log("WebSocket disconnected");
      setIsConnected(false);
    };

    return () => {
      ws.close();
    };
  }, [url]);

  const sendMessage = useCallback((message) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(message));
    }
  }, []);

  return { messages, isConnected, sendMessage };
}

// Usage
function ChatRoom({ roomId }) {
  const { messages, isConnected, sendMessage } = useWebSocket(`wss://api.example.com/chat/${roomId}`);

  return (
    <div>
      <div>Status: {isConnected ? "Connected" : "Disconnected"}</div>
      <MessageList messages={messages} />
      <MessageInput onSend={sendMessage} />
    </div>
  );
}

// ═══════════════════════════════════════════════════════════
// Event Emitter subscription
// ═══════════════════════════════════════════════════════════
function useEventSubscription(eventEmitter, eventName, handler) {
  useEffect(() => {
    eventEmitter.on(eventName, handler);

    return () => {
      eventEmitter.off(eventName, handler);
    };
  }, [eventEmitter, eventName, handler]);
}

// ═══════════════════════════════════════════════════════════
// Redux-like store subscription
// ═══════════════════════════════════════════════════════════
function useStore(store, selector) {
  const [state, setState] = useState(() => selector(store.getState()));

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      const newState = selector(store.getState());
      setState(newState);
    });

    return unsubscribe;
  }, [store, selector]);

  return state;
}
```

### Form Handling Pattern

```jsx
// ═══════════════════════════════════════════════════════════
// Form với auto-save draft
// ═══════════════════════════════════════════════════════════
function useFormWithDraft(formId, initialValues) {
  const [values, setValues] = useState(() => {
    // Load draft từ sessionStorage khi mount
    const draft = sessionStorage.getItem(`form_draft_${formId}`);
    return draft ? JSON.parse(draft) : initialValues;
  });

  const [isDirty, setIsDirty] = useState(false);

  // Auto-save draft khi values thay đổi
  useEffect(() => {
    if (isDirty) {
      sessionStorage.setItem(`form_draft_${formId}`, JSON.stringify(values));
    }
  }, [values, isDirty, formId]);

  // Clear draft khi unmount (nếu đã submit)
  const clearDraft = useCallback(() => {
    sessionStorage.removeItem(`form_draft_${formId}`);
  }, [formId]);

  // Warn before leaving với unsaved changes
  useEffect(() => {
    const handleBeforeUnload = (e) => {
      if (isDirty) {
        e.preventDefault();
        e.returnValue = "";
      }
    };

    window.addEventListener("beforeunload", handleBeforeUnload);

    return () => {
      window.removeEventListener("beforeunload", handleBeforeUnload);
    };
  }, [isDirty]);

  const handleChange = useCallback((name, value) => {
    setValues((prev) => ({ ...prev, [name]: value }));
    setIsDirty(true);
  }, []);

  const reset = useCallback(() => {
    setValues(initialValues);
    setIsDirty(false);
    clearDraft();
  }, [initialValues, clearDraft]);

  return { values, handleChange, isDirty, reset, clearDraft };
}

// Usage
function ContactForm() {
  const { values, handleChange, isDirty, reset, clearDraft } = useFormWithDraft("contact", {
    name: "",
    email: "",
    message: "",
  });

  const handleSubmit = async (e) => {
    e.preventDefault();
    await submitForm(values);
    clearDraft();
    reset();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={values.name} onChange={(e) => handleChange("name", e.target.value)} />
      {/* ... */}
      {isDirty && <span>Unsaved changes</span>}
    </form>
  );
}
```

### Animation Pattern

```jsx
// ═══════════════════════════════════════════════════════════
// Mount/Unmount animation
// ═══════════════════════════════════════════════════════════
function useAnimatedMount(isVisible, duration = 300) {
  const [shouldRender, setShouldRender] = useState(isVisible);
  const [isAnimating, setIsAnimating] = useState(false);

  useEffect(() => {
    if (isVisible) {
      setShouldRender(true);
      // Delay để trigger animation
      requestAnimationFrame(() => {
        setIsAnimating(true);
      });
    } else {
      setIsAnimating(false);
      // Đợi animation xong rồi mới unmount
      const timer = setTimeout(() => {
        setShouldRender(false);
      }, duration);

      return () => clearTimeout(timer);
    }
  }, [isVisible, duration]);

  return { shouldRender, isAnimating };
}

// Usage
function Modal({ isOpen, onClose, children }) {
  const { shouldRender, isAnimating } = useAnimatedMount(isOpen, 300);

  if (!shouldRender) return null;

  return (
    <div className={`modal ${isAnimating ? "modal-enter" : "modal-exit"}`} onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>
  );
}

// CSS
/*
.modal {
  opacity: 0;
  transition: opacity 300ms;
}
.modal.modal-enter {
  opacity: 1;
}
.modal.modal-exit {
  opacity: 0;
}
*/
```

---

## 8. Performance Optimization

### React.memo

```jsx
// ═══════════════════════════════════════════════════════════
// React.memo - Prevent unnecessary re-renders
// ═══════════════════════════════════════════════════════════

// Không có memo - re-render mỗi khi parent re-render
function ExpensiveComponent({ data }) {
  console.log("ExpensiveComponent rendered");
  return <div>{/* expensive rendering */}</div>;
}

// Có memo - chỉ re-render khi props thay đổi
const MemoizedComponent = React.memo(function ExpensiveComponent({ data }) {
  console.log("MemoizedComponent rendered");
  return <div>{/* expensive rendering */}</div>;
});

// Custom comparison function
const MemoizedWithCompare = React.memo(
  function Component({ user, onClick }) {
    return <div onClick={onClick}>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true nếu props GIỐNG nhau (không cần re-render)
    // Return false nếu props KHÁC nhau (cần re-render)
    return prevProps.user.id === nextProps.user.id;
  }
);
```

### useMemo và useCallback

```jsx
// ═══════════════════════════════════════════════════════════
// useMemo - Memoize computed values
// ═══════════════════════════════════════════════════════════
function ProductList({ products, filterTerm }) {
  // ❌ Tính toán lại mỗi render
  const filteredProducts = products.filter((p) => p.name.includes(filterTerm));

  // ✅ Chỉ tính lại khi products hoặc filterTerm thay đổi
  const filteredProducts = useMemo(() => {
    console.log("Filtering products...");
    return products.filter((p) => p.name.includes(filterTerm));
  }, [products, filterTerm]);

  // ✅ Expensive calculation
  const statistics = useMemo(() => {
    console.log("Calculating statistics...");
    return {
      total: products.length,
      avgPrice: products.reduce((sum, p) => sum + p.price, 0) / products.length,
      categories: [...new Set(products.map((p) => p.category))],
    };
  }, [products]);

  return (
    <div>
      <Stats data={statistics} />
      {filteredProducts.map((p) => (
        <ProductCard key={p.id} product={p} />
      ))}
    </div>
  );
}

// ═══════════════════════════════════════════════════════════
// useCallback - Memoize functions
// ═══════════════════════════════════════════════════════════
function ParentComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("");

  // ❌ Tạo function mới mỗi render
  // → ChildComponent re-render dù không cần
  const handleClick = () => {
    console.log("Clicked");
  };

  // ✅ Function được memoize
  const handleClick = useCallback(() => {
    console.log("Clicked");
  }, []);

  // ✅ Với dependencies
  const handleSubmit = useCallback(() => {
    submitData(name);
  }, [name]);

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <MemoizedChild onClick={handleClick} />
    </div>
  );
}

const MemoizedChild = React.memo(function Child({ onClick }) {
  console.log("Child rendered");
  return <button onClick={onClick}>Click me</button>;
});
```

### Lazy Loading và Code Splitting

```jsx
import React, { Suspense, lazy } from "react";

// ═══════════════════════════════════════════════════════════
// Lazy load components
// ═══════════════════════════════════════════════════════════
const HeavyComponent = lazy(() => import("./HeavyComponent"));
const AdminPanel = lazy(() => import("./AdminPanel"));

function App() {
  const [showHeavy, setShowHeavy] = useState(false);

  return (
    <div>
      <button onClick={() => setShowHeavy(true)}>Load Heavy Component</button>

      {showHeavy && (
        <Suspense fallback={<div>Loading...</div>}>
          <HeavyComponent />
        </Suspense>
      )}
    </div>
  );
}

// ═══════════════════════════════════════════════════════════
// Route-based code splitting
// ═══════════════════════════════════════════════════════════
const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));
const Dashboard = lazy(() => import("./pages/Dashboard"));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Suspense>
  );
}

// ═══════════════════════════════════════════════════════════
// Preload on hover
// ═══════════════════════════════════════════════════════════
const HeavyComponent = lazy(() => import("./HeavyComponent"));

// Preload function
const preloadHeavyComponent = () => {
  import("./HeavyComponent");
};

function App() {
  return (
    <button onMouseEnter={preloadHeavyComponent} onClick={() => setShowHeavy(true)}>
      Show Heavy Component
    </button>
  );
}
```

### Virtualization cho Long Lists

```jsx
import { useVirtualizer } from "@tanstack/react-virtual";

function VirtualizedList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50, // Estimated row height
    overscan: 5, // Render thêm 5 items ngoài viewport
  });

  return (
    <div ref={parentRef} style={{ height: "400px", overflow: "auto" }}>
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: "relative",
        }}
      >
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: "absolute",
              top: 0,
              left: 0,
              width: "100%",
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            {items[virtualItem.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 9. Best Practices

### useEffect Best Practices

```jsx
// ═══════════════════════════════════════════════════════════
// 1. Một effect cho một mục đích
// ═══════════════════════════════════════════════════════════

// ❌ Bad - Nhiều concerns trong một effect
useEffect(() => {
  fetchUser(userId);
  document.title = `User ${userId}`;
  analytics.track("view_user", userId);
  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, [userId]);

// ✅ Good - Tách riêng từng concern
useEffect(() => {
  fetchUser(userId);
}, [userId]);

useEffect(() => {
  document.title = `User ${userId}`;
}, [userId]);

useEffect(() => {
  analytics.track("view_user", userId);
}, [userId]);

useEffect(() => {
  window.addEventListener("resize", handleResize);
  return () => window.removeEventListener("resize", handleResize);
}, []);

// ═══════════════════════════════════════════════════════════
// 2. Luôn cleanup resources
// ═══════════════════════════════════════════════════════════

// ✅ Cleanup subscriptions
useEffect(() => {
  const subscription = eventEmitter.subscribe(handler);
  return () => subscription.unsubscribe();
}, []);

// ✅ Cleanup timers
useEffect(() => {
  const timer = setInterval(tick, 1000);
  return () => clearInterval(timer);
}, []);

// ✅ Cleanup fetch requests
useEffect(() => {
  const controller = new AbortController();

  fetch(url, { signal: controller.signal })
    .then(/* ... */)
    .catch((err) => {
      if (err.name !== "AbortError") {
        setError(err);
      }
    });

  return () => controller.abort();
}, [url]);

// ═══════════════════════════════════════════════════════════
// 3. Tránh unnecessary dependencies
// ═══════════════════════════════════════════════════════════

// ❌ Bad - object trong deps
useEffect(() => {
  doSomething(options);
}, [options]); // options là object mới mỗi render

// ✅ Good - primitive values
useEffect(() => {
  doSomething({ page, limit });
}, [page, limit]);

// ✅ Good - useRef cho stable reference
const optionsRef = useRef(options);
optionsRef.current = options;

useEffect(() => {
  doSomething(optionsRef.current);
}, []); // Không cần deps

// ═══════════════════════════════════════════════════════════
// 4. Sử dụng functional updates cho setState
// ═══════════════════════════════════════════════════════════

// ❌ Bad - count trong deps
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1); // Stale closure
  }, 1000);
  return () => clearInterval(timer);
}, [count]); // Timer reset mỗi khi count thay đổi

// ✅ Good - functional update
useEffect(() => {
  const timer = setInterval(() => {
    setCount((c) => c + 1); // Luôn có giá trị mới nhất
  }, 1000);
  return () => clearInterval(timer);
}, []); // Không cần count trong deps
```

### State Management Best Practices

```jsx
// ═══════════════════════════════════════════════════════════
// 1. Colocate state gần nơi sử dụng
// ═══════════════════════════════════════════════════════════

// ❌ Bad - State ở quá cao
function App() {
  const [searchTerm, setSearchTerm] = useState("");

  return (
    <div>
      <Header />
      <SearchPage searchTerm={searchTerm} setSearchTerm={setSearchTerm} />
      <Footer />
    </div>
  );
}

// ✅ Good - State ở component cần nó
function SearchPage() {
  const [searchTerm, setSearchTerm] = useState("");

  return (
    <div>
      <SearchInput value={searchTerm} onChange={setSearchTerm} />
      <SearchResults term={searchTerm} />
    </div>
  );
}

// ═══════════════════════════════════════════════════════════
// 2. Derive state thay vì sync state
// ═══════════════════════════════════════════════════════════

// ❌ Bad - Sync state
function ProductList({ products }) {
  const [filteredProducts, setFilteredProducts] = useState(products);
  const [filter, setFilter] = useState("");

  useEffect(() => {
    setFilteredProducts(products.filter((p) => p.name.includes(filter)));
  }, [products, filter]);

  return /* ... */;
}

// ✅ Good - Derive trong render
function ProductList({ products }) {
  const [filter, setFilter] = useState("");

  // Tính toán trực tiếp, không cần state riêng
  const filteredProducts = useMemo(() => products.filter((p) => p.name.includes(filter)), [products, filter]);

  return /* ... */;
}

// ═══════════════════════════════════════════════════════════
// 3. Batch related state updates
// ═══════════════════════════════════════════════════════════

// ❌ Bad - Multiple state updates
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
const [data, setData] = useState(null);

const fetchData = async () => {
  setLoading(true);
  setError(null);
  try {
    const result = await fetch(url);
    setData(result);
    setLoading(false);
  } catch (err) {
    setError(err);
    setLoading(false);
  }
};

// ✅ Good - useReducer cho related state
const initialState = { loading: false, error: null, data: null };

function reducer(state, action) {
  switch (action.type) {
    case "FETCH_START":
      return { ...state, loading: true, error: null };
    case "FETCH_SUCCESS":
      return { loading: false, error: null, data: action.payload };
    case "FETCH_ERROR":
      return { loading: false, error: action.payload, data: null };
    default:
      return state;
  }
}

function Component() {
  const [state, dispatch] = useReducer(reducer, initialState);

  const fetchData = async () => {
    dispatch({ type: "FETCH_START" });
    try {
      const result = await fetch(url);
      dispatch({ type: "FETCH_SUCCESS", payload: result });
    } catch (err) {
      dispatch({ type: "FETCH_ERROR", payload: err });
    }
  };
}
```

### Component Design Best Practices

```jsx
// ═══════════════════════════════════════════════════════════
// 1. Single Responsibility
// ═══════════════════════════════════════════════════════════

// ❌ Bad - Component làm quá nhiều việc
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [notifications, setNotifications] = useState([]);
  // ... fetch logic, handlers, etc.

  return (
    <div>
      {/* User info */}
      {/* Posts list */}
      {/* Notifications */}
      {/* Settings */}
    </div>
  );
}

// ✅ Good - Tách thành components nhỏ
function UserDashboard({ userId }) {
  return (
    <div>
      <UserInfo userId={userId} />
      <UserPosts userId={userId} />
      <UserNotifications userId={userId} />
      <UserSettings userId={userId} />
    </div>
  );
}

// ═══════════════════════════════════════════════════════════
// 2. Composition over Props Drilling
// ═══════════════════════════════════════════════════════════

// ❌ Bad - Props drilling
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  return <Header user={user} setUser={setUser} />;
}

function Header({ user, setUser }) {
  return <UserMenu user={user} setUser={setUser} />;
}

// ✅ Good - Composition
function App() {
  const [user, setUser] = useState(null);

  return (
    <Layout header={<Header userMenu={<UserMenu user={user} onLogout={() => setUser(null)} />} />}>
      <MainContent />
    </Layout>
  );
}

// ✅ Better - Context cho global state
const UserContext = createContext();

function App() {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Layout />
    </UserContext.Provider>
  );
}

function UserMenu() {
  const { user, setUser } = useContext(UserContext);
  return /* ... */;
}
```

---

## 10. Debugging Lifecycle

### Console Logging

```jsx
// ═══════════════════════════════════════════════════════════
// Debug hook để track lifecycle
// ═══════════════════════════════════════════════════════════
function useLifecycleLogger(componentName) {
  const renderCount = useRef(0);
  renderCount.current += 1;

  console.log(`[${componentName}] Render #${renderCount.current}`);

  useEffect(() => {
    console.log(`[${componentName}] Mounted`);

    return () => {
      console.log(`[${componentName}] Will Unmount`);
    };
  }, [componentName]);

  useEffect(() => {
    console.log(`[${componentName}] Updated`);
  });
}

// Usage
function MyComponent() {
  useLifecycleLogger("MyComponent");
  // ...
}

// ═══════════════════════════════════════════════════════════
// Track why component re-rendered
// ═══════════════════════════════════════════════════════════
function useWhyDidYouUpdate(name, props) {
  const previousProps = useRef();

  useEffect(() => {
    if (previousProps.current) {
      const allKeys = Object.keys({ ...previousProps.current, ...props });
      const changedProps = {};

      allKeys.forEach((key) => {
        if (previousProps.current[key] !== props[key]) {
          changedProps[key] = {
            from: previousProps.current[key],
            to: props[key],
          };
        }
      });

      if (Object.keys(changedProps).length) {
        console.log(`[${name}] Changed props:`, changedProps);
      }
    }

    previousProps.current = props;
  });
}

// Usage
function MyComponent(props) {
  useWhyDidYouUpdate("MyComponent", props);
  // ...
}
```

### React DevTools

```jsx
// ═══════════════════════════════════════════════════════════
// Profiler API
// ═══════════════════════════════════════════════════════════
import { Profiler } from "react";

function onRenderCallback(
  id, // Profiler id
  phase, // "mount" | "update"
  actualDuration, // Thời gian render
  baseDuration, // Thời gian render không có memoization
  startTime, // Khi React bắt đầu render
  commitTime, // Khi React commit
  interactions // Set of interactions
) {
  console.log(`[Profiler ${id}]`, {
    phase,
    actualDuration: `${actualDuration.toFixed(2)}ms`,
    baseDuration: `${baseDuration.toFixed(2)}ms`,
  });
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <MyComponent />
    </Profiler>
  );
}
```

### Common Issues và Solutions

```jsx
// ═══════════════════════════════════════════════════════════
// Issue 1: Infinite loop
// ═══════════════════════════════════════════════════════════

// ❌ Causes infinite loop
useEffect(() => {
  setCount(count + 1); // setState triggers re-render → effect runs again
}, [count]);

// ✅ Fix: Remove from deps hoặc add condition
useEffect(() => {
  if (count < 10) {
    setCount((c) => c + 1);
  }
}, [count]);

// ═══════════════════════════════════════════════════════════
// Issue 2: Stale closure
// ═══════════════════════════════════════════════════════════

// ❌ Stale value
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // Luôn là 0
    }, 1000);
    return () => clearInterval(timer);
  }, []);
}

// ✅ Fix: useRef hoặc functional update
function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  countRef.current = count;

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(countRef.current); // Giá trị mới nhất
    }, 1000);
    return () => clearInterval(timer);
  }, []);
}

// ═══════════════════════════════════════════════════════════
// Issue 3: Memory leak - setState after unmount
// ═══════════════════════════════════════════════════════════

// ❌ Warning: Can't perform state update on unmounted component
useEffect(() => {
  fetchData().then((data) => {
    setData(data); // Component có thể đã unmount
  });
}, []);

// ✅ Fix: Check mounted state
useEffect(() => {
  let isMounted = true;

  fetchData().then((data) => {
    if (isMounted) {
      setData(data);
    }
  });

  return () => {
    isMounted = false;
  };
}, []);

// ✅ Better: AbortController
useEffect(() => {
  const controller = new AbortController();

  fetchData({ signal: controller.signal })
    .then(setData)
    .catch((err) => {
      if (err.name !== "AbortError") {
        setError(err);
      }
    });

  return () => controller.abort();
}, []);
```

---

## Tổng kết

### Lifecycle Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIFECYCLE CHEAT SHEET                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MOUNTING:                                                       │
│  • useState(() => init)     - Lazy initialization               │
│  • useEffect(() => {}, [])  - Run once after mount              │
│                                                                  │
│  UPDATING:                                                       │
│  • useEffect(() => {}, [deps]) - Run when deps change           │
│  • useMemo/useCallback         - Memoize values/functions       │
│                                                                  │
│  UNMOUNTING:                                                     │
│  • useEffect cleanup function  - Cleanup resources              │
│                                                                  │
│  OPTIMIZATION:                                                   │
│  • React.memo()             - Prevent unnecessary re-renders    │
│  • useMemo()                - Memoize expensive calculations    │
│  • useCallback()            - Memoize functions                 │
│  • lazy() + Suspense        - Code splitting                    │
│                                                                  │
│  COMMON PATTERNS:                                                │
│  • Data fetching            - useEffect + AbortController       │
│  • Subscriptions            - useEffect + cleanup               │
│  • Timers                   - useEffect + clearInterval         │
│  • Event listeners          - useEffect + removeEventListener   │
│  • Previous value           - useRef                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Takeaways

1. **Functional components + Hooks** là cách viết React hiện đại
2. **useEffect** thay thế hầu hết lifecycle methods
3. **Luôn cleanup** resources trong useEffect
4. **Dependencies array** quyết định khi nào effect chạy
5. **Tách effects** theo concern, không gộp chung
6. **Memoization** (memo, useMemo, useCallback) cho performance
7. **Error Boundaries** vẫn cần class components
8. **DevTools** và logging để debug lifecycle issues
