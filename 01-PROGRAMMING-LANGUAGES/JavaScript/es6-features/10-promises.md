# Promises

> Quay lại [Mục lục](./README.md) | Trước: [Modules](./09-modules.md)

## 📌 Tổng quan

**Promise** là object đại diện cho kết quả (thành công hoặc thất bại) của một **thao tác bất đồng bộ** (asynchronous operation). Promise giải quyết vấn đề **callback hell** và giúp xử lý async code dễ đọc, dễ bảo trì hơn.

### Tại sao cần Promises?

```javascript
// ❌ Callback hell (Pyramid of Doom)
getUser(userId, function(user) {
  getPosts(user.id, function(posts) {
    getComments(posts[0].id, function(comments) {
      getLikes(comments[0].id, function(likes) {
        // ... nested deeper and deeper
      });
    });
  });
});

// ✅ Promises — flat chain
getUser(userId)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => getLikes(comments[0].id))
  .then(likes => console.log(likes))
  .catch(error => console.error(error));
```

---

## 1. Promise Basics

### 1.1. Tạo Promise

```javascript
const promise = new Promise((resolve, reject) => {
  // Executor function — chạy NGAY LẬP TỨC khi tạo promise
  
  // Thao tác bất đồng bộ
  setTimeout(() => {
    const success = true;
    
    if (success) {
      resolve("Data loaded successfully"); // Thành công
    } else {
      reject(new Error("Failed to load"));  // Thất bại
    }
  }, 1000);
});

// Sử dụng promise
promise
  .then(result => console.log(result))    // Xử lý thành công
  .catch(error => console.error(error))   // Xử lý lỗi
  .finally(() => console.log("Done"));    // Luôn chạy
```

### 1.2. Promise States

```
┌─────────────────────────────────────────────────────────┐
│                   PROMISE STATES                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│                    ┌──────────┐                         │
│                    │ PENDING  │ ← Trạng thái ban đầu   │
│                    └────┬─────┘                         │
│                         │                               │
│           ┌─────────────┴─────────────┐                │
│           │                           │                │
│           ▼                           ▼                │
│    ┌───────────┐              ┌───────────┐           │
│    │ FULFILLED │              │ REJECTED  │           │
│    │ (resolved)│              │ (failed)  │           │
│    └─────┬─────┘              └─────┬─────┘           │
│          │                          │                  │
│          ▼                          ▼                  │
│       .then(onFulfilled)       .catch(onRejected)     │
│                                                        │
│          └──────────┬───────────┘                      │
│                     ▼                                  │
│              .finally(onFinally)                       │
│                                                        │
│  • Pending → Fulfilled HOẶC Rejected (một lần duy nhất)│
│  • Sau khi settled, state không thể thay đổi           │
│  • Promise là immutable về state                       │
│                                                        │
└─────────────────────────────────────────────────────────┘
```

### 1.3. Resolve/Reject chỉ gọi 1 lần

```javascript
const promise = new Promise((resolve, reject) => {
  resolve("first");   // ✅ Promise fulfilled
  resolve("second");  // ❌ Bị bỏ qua
  reject("error");    // ❌ Bị bỏ qua
  console.log("Still runs"); // ✅ Code vẫn chạy nhưng promise đã settled
});
// Luôn nên return sau resolve/reject nếu không muốn code tiếp tục
```

---

## 2. Promise Chaining

### 2.1. Sequential Operations

```javascript
fetchUser(userId)
  .then(user => {
    console.log("User:", user);
    return fetchPosts(user.id); // Return promise → chain tiếp
  })
  .then(posts => {
    console.log("Posts:", posts);
    return fetchComments(posts[0].id);
  })
  .then(comments => {
    console.log("Comments:", comments);
  })
  .catch(error => {
    // Bắt lỗi từ BẤT KỲ step nào ở trên
    console.error("Error:", error);
  });
```

### 2.2. Return Values trong Chain

```javascript
// Return value → wrapped trong Promise.resolve()
Promise.resolve(1)
  .then(x => x + 1)    // 2 (return 2 → Promise.resolve(2))
  .then(x => x * 2)    // 4
  .then(x => x + 3)    // 7
  .then(console.log);  // 7

// Return promise → chờ promise resolve
Promise.resolve("start")
  .then(val => {
    return new Promise(resolve => {
      setTimeout(() => resolve(val + " → step1"), 1000);
    });
  })
  .then(val => {
    return new Promise(resolve => {
      setTimeout(() => resolve(val + " → step2"), 1000);
    });
  })
  .then(console.log);
// Sau 2 giây: "start → step1 → step2"

// Không return → undefined
Promise.resolve(1)
  .then(x => { x + 1; })  // ⚠️ Không return → undefined
  .then(x => console.log(x)); // undefined
```

---

## 3. Error Handling

### 3.1. `.catch()` vị trí

```javascript
// Catch ở CUỐI — bắt tất cả lỗi
fetchData()
  .then(processData)
  .then(saveData)
  .then(notify)
  .catch(error => {
    // Bắt lỗi từ fetchData, processData, saveData, hoặc notify
    console.error("Something failed:", error);
  });
```

### 3.2. Catch và Recover

```javascript
fetchData()
  .then(data => {
    if (!data) throw new Error("No data");
    return data;
  })
  .catch(error => {
    console.warn("Using fallback data");
    return fallbackData; // Recover — chain TIẾP TỤC bình thường
  })
  .then(data => {
    // Nhận data hoặc fallbackData
    renderUI(data);
  });
```

### 3.3. Re-throwing Errors

```javascript
fetchData()
  .catch(error => {
    logError(error);     // Log lỗi
    throw error;         // Re-throw → catch tiếp theo xử lý
  })
  .then(data => {
    // KHÔNG chạy nếu catch ở trên throw
  })
  .catch(error => {
    showUserError(error); // Hiển thị lỗi cho user
  });
```

### 3.4. Error Handling Best Practices

```javascript
// ✅ Luôn catch ở cuối chain
promise.then(onSuccess).catch(onError);

// ❌ Unhandled rejection (no catch)
promise.then(onSuccess); // Nếu reject → UnhandledPromiseRejection

// ✅ Throw Error objects (không throw strings)
reject(new Error("Something failed")); // ✅ Có stack trace
reject("Something failed");             // ❌ Không có stack trace

// ✅ Specific error handling
fetchData()
  .catch(error => {
    if (error instanceof NetworkError) {
      return retryFetch();
    }
    if (error instanceof AuthError) {
      return redirectToLogin();
    }
    throw error; // Re-throw unknown errors
  });
```

---

## 4. Promise Static Methods

### 4.1. `Promise.all()`

Chờ **TẤT CẢ** promises fulfill. Nếu **MỘT** reject → toàn bộ reject.

```javascript
const promises = [
  fetch("/api/users"),
  fetch("/api/posts"),
  fetch("/api/comments"),
];

Promise.all(promises)
  .then(([usersRes, postsRes, commentsRes]) => {
    // Tất cả đã hoàn thành thành công
    return Promise.all([
      usersRes.json(),
      postsRes.json(),
      commentsRes.json(),
    ]);
  })
  .then(([users, posts, comments]) => {
    console.log(users, posts, comments);
  })
  .catch(error => {
    // MỘT trong số đó failed → vào đây
    console.error("At least one request failed:", error);
  });

// Empty array → resolve ngay với []
Promise.all([]).then(results => console.log(results)); // []
```

### 4.2. `Promise.allSettled()` (ES2020)

Chờ **TẤT CẢ** promises settle (dù success hay fail).

```javascript
const promises = [
  fetch("/api/users"),
  fetch("/api/broken-endpoint"),
  fetch("/api/posts"),
];

Promise.allSettled(promises).then(results => {
  results.forEach((result, index) => {
    if (result.status === "fulfilled") {
      console.log(`#${index} Success:`, result.value);
    } else {
      console.log(`#${index} Failed:`, result.reason);
    }
  });
  
  // Lọc chỉ lấy thành công
  const successful = results
    .filter(r => r.status === "fulfilled")
    .map(r => r.value);
});
```

### 4.3. `Promise.race()`

Lấy kết quả **ĐẦU TIÊN** settle (dù success hay fail).

```javascript
// Timeout pattern
function fetchWithTimeout(url, timeoutMs) {
  return Promise.race([
    fetch(url),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error("Timeout")), timeoutMs)
    ),
  ]);
}

fetchWithTimeout("/api/data", 5000)
  .then(response => response.json())
  .catch(error => {
    if (error.message === "Timeout") {
      console.log("Request timed out");
    }
  });
```

### 4.4. `Promise.any()` (ES2021)

Lấy kết quả **THÀNH CÔNG ĐẦU TIÊN**. Chỉ reject khi TẤT CẢ fail.

```javascript
// Fastest mirror / fallback
Promise.any([
  fetch("https://api1.example.com/data"),
  fetch("https://api2.example.com/data"),
  fetch("https://api3.example.com/data"),
])
  .then(response => {
    console.log("First success:", response.url);
  })
  .catch(error => {
    // AggregateError — tất cả đều failed
    console.log("All failed:", error.errors);
  });
```

### 4.5. `Promise.resolve()` / `Promise.reject()`

```javascript
// Tạo fulfilled promise
Promise.resolve("value");
Promise.resolve(42);
Promise.resolve({ data: "test" });

// Tạo rejected promise
Promise.reject(new Error("failed"));

// Wrap value thành promise (useful cho consistent API)
function getData(useCache) {
  if (useCache) {
    return Promise.resolve(cachedData); // Sync → Promise
  }
  return fetch("/api/data").then(r => r.json()); // Async → Promise
}
// Caller luôn dùng .then() regardless
```

---

## 5. Practical Patterns

### 5.1. Fetch Wrapper

```javascript
async function fetchJSON(url, options = {}) {
  const response = await fetch(url, {
    headers: { "Content-Type": "application/json", ...options.headers },
    ...options,
  });
  
  if (!response.ok) {
    const error = new Error(`HTTP ${response.status}: ${response.statusText}`);
    error.status = response.status;
    error.response = response;
    throw error;
  }
  
  return response.json();
}
```

### 5.2. Retry Logic

```javascript
function fetchWithRetry(url, options = {}, retries = 3, delay = 1000) {
  return fetch(url, options).catch(error => {
    if (retries <= 0) throw error;
    
    console.log(`Retrying... ${retries} attempts left`);
    return new Promise(resolve => setTimeout(resolve, delay))
      .then(() => fetchWithRetry(url, options, retries - 1, delay * 2));
  });
}
```

### 5.3. Parallel with Limit

```javascript
async function parallelLimit(tasks, limit) {
  const results = [];
  const executing = new Set();

  for (const [index, task] of tasks.entries()) {
    const promise = task().then(result => {
      executing.delete(promise);
      return result;
    });
    
    results[index] = promise;
    executing.add(promise);

    if (executing.size >= limit) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}

// Usage: Max 3 concurrent requests
const tasks = urls.map(url => () => fetch(url));
const results = await parallelLimit(tasks, 3);
```

### 5.4. Promise Queue

```javascript
class PromiseQueue {
  #queue = [];
  #processing = false;

  add(promiseFn) {
    return new Promise((resolve, reject) => {
      this.#queue.push({ promiseFn, resolve, reject });
      this.#process();
    });
  }

  async #process() {
    if (this.#processing) return;
    this.#processing = true;

    while (this.#queue.length > 0) {
      const { promiseFn, resolve, reject } = this.#queue.shift();
      try {
        const result = await promiseFn();
        resolve(result);
      } catch (error) {
        reject(error);
      }
    }

    this.#processing = false;
  }
}
```

---

## 6. Promise vs async/await

```javascript
// Promise chain
function getUserData(userId) {
  return getUser(userId)
    .then(user => getPosts(user.id))
    .then(posts => ({ user, posts })) // ⚠️ user không accessible
    .catch(error => console.error(error));
}

// async/await — cleaner, easier to reason about
async function getUserData(userId) {
  try {
    const user = await getUser(userId);
    const posts = await getPosts(user.id); // user accessible ✅
    return { user, posts };
  } catch (error) {
    console.error(error);
  }
}

// async/await VẪN dùng Promises bên dưới
// async function luôn return Promise
// await unwrap Promise value
```

---

## 7. Anti-patterns

```javascript
// ❌ Promise constructor anti-pattern (không cần new Promise khi đã có promise)
function bad() {
  return new Promise((resolve, reject) => {
    fetch("/api/data")
      .then(response => resolve(response.json()))
      .catch(error => reject(error));
  });
}

// ✅ Just return the promise
function good() {
  return fetch("/api/data").then(response => response.json());
}

// ❌ Chaining không đúng (nested .then)
fetch("/api/user")
  .then(user => {
    fetch(`/api/posts/${user.id}`)
      .then(posts => {
        // Nested — callback hell pattern!
      });
  });

// ✅ Flat chain
fetch("/api/user")
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(posts => { /* ... */ });

// ❌ Forget to return in .then
promise
  .then(value => {
    doSomethingAsync(value); // ⚠️ No return — next .then gets undefined
  })
  .then(result => { /* result is undefined */ });
```

---

> Tiếp theo: [Symbol →](./11-symbol.md)
