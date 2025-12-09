# Node.js Interview Questions

## 1. Core Node.js

### Q: Event Loop là gì và hoạt động như thế nào?

**A:** Event Loop là cơ chế cho phép Node.js thực hiện non-blocking I/O operations mặc dù JavaScript là single-threaded. Nó liên tục kiểm tra Call Stack và các queues để thực thi code.

Các phases:

1. **Timers**: setTimeout, setInterval
2. **Pending callbacks**: I/O callbacks deferred
3. **Poll**: Lấy I/O events mới
4. **Check**: setImmediate
5. **Close callbacks**: socket.on('close')

Giữa mỗi phase, Node.js xử lý Microtask Queue (process.nextTick, Promise).

---

### Q: Sự khác biệt giữa process.nextTick() và setImmediate()?

**A:**

- `process.nextTick()`: Chạy ngay sau operation hiện tại, trước khi event loop tiếp tục. Thuộc Microtask Queue.
- `setImmediate()`: Chạy trong check phase của event loop iteration tiếp theo. Thuộc Macrotask Queue.

```javascript
setImmediate(() => console.log("setImmediate"));
process.nextTick(() => console.log("nextTick"));
// Output: nextTick, setImmediate
```

---

### Q: Blocking vs Non-blocking I/O?

**A:**

- **Blocking**: Code chờ operation hoàn thành mới chạy tiếp. Block event loop.
- **Non-blocking**: Code tiếp tục chạy, callback được gọi khi operation hoàn thành.

```javascript
// Blocking
const data = fs.readFileSync("file.txt");

// Non-blocking
fs.readFile("file.txt", (err, data) => {});
```

---

### Q: Libuv là gì?

**A:** Libuv là thư viện C++ cung cấp:

- Event loop implementation
- Asynchronous I/O
- Thread pool (mặc định 4 threads) cho file system, DNS, crypto operations

---

## 2. Async Programming

### Q: Callback Hell là gì và cách giải quyết?

**A:** Callback Hell là tình trạng nested callbacks quá sâu, khó đọc và maintain.

Giải quyết:

1. **Promises**: Chain `.then()`
2. **Async/Await**: Viết code async như sync
3. **Modularize**: Tách thành functions nhỏ

---

### Q: Promise.all vs Promise.allSettled vs Promise.race?

**A:**

- `Promise.all`: Chờ tất cả resolve, reject ngay nếu 1 reject
- `Promise.allSettled`: Chờ tất cả complete (không fail)
- `Promise.race`: Trả về kết quả đầu tiên (resolve hoặc reject)

---

### Q: Xử lý error trong async/await?

**A:**

```javascript
// try/catch
async function getData() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error(error);
    throw error;
  }
}

// .catch() on promise
const data = await fetchData().catch((err) => null);
```

---

## 3. Modules

### Q: CommonJS vs ES Modules?

**A:**
| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| Syntax | require/module.exports | import/export |
| Loading | Synchronous | Asynchronous |
| Top-level await | ❌ | ✅ |
| Tree shaking | ❌ | ✅ |

---

### Q: Module caching hoạt động như thế nào?

**A:** Node.js cache modules sau lần require đầu tiên. Các lần require sau trả về cùng instance.

```javascript
const a = require("./module");
const b = require("./module");
console.log(a === b); // true
```

---

## 4. Security

### Q: JWT hoạt động như thế nào?

**A:** JWT gồm 3 phần: Header.Payload.Signature

1. Client gửi credentials
2. Server verify và trả về JWT
3. Client gửi JWT trong header cho các requests sau
4. Server verify signature và extract payload

---

### Q: Access Token vs Refresh Token?

**A:**

- **Access Token**: Short-lived (15m), dùng để access resources
- **Refresh Token**: Long-lived (7d), dùng để lấy access token mới

Refresh token rotation: Mỗi lần refresh, cả 2 tokens đều được tạo mới.

---

### Q: Cách prevent SQL Injection?

**A:**

1. Parameterized queries
2. ORM (auto-escaped)
3. Input validation

```javascript
// ❌ Bad
const query = `SELECT * FROM users WHERE id = '${id}'`;

// ✅ Good
const query = "SELECT * FROM users WHERE id = $1";
await db.query(query, [id]);
```

---

## 5. Performance

### Q: Làm sao scale Node.js app?

**A:**

1. **Clustering**: Tận dụng multi-core CPU
2. **Load Balancing**: Nginx, HAProxy
3. **Horizontal Scaling**: Multiple servers
4. **Caching**: Redis
5. **Database Optimization**: Indexing, connection pooling

---

### Q: N+1 Query Problem là gì?

**A:** Khi fetch N records, mỗi record cần thêm 1 query để lấy related data = N+1 queries.

```javascript
// ❌ N+1
const users = await User.find();
for (const user of users) {
  const posts = await Post.find({ authorId: user.id });
}

// ✅ Eager loading
const users = await User.find().populate("posts");
```

---

### Q: Memory leak trong Node.js?

**A:** Nguyên nhân phổ biến:

1. Global variables
2. Event listeners không remove
3. Closures giữ references
4. Timers không clear

Detect: `process.memoryUsage()`, heap snapshots

---

## 6. Database

### Q: Connection Pooling là gì?

**A:** Tái sử dụng database connections thay vì tạo mới cho mỗi request. Giảm overhead, tăng performance.

```javascript
const pool = new Pool({ max: 20 });
```

---

### Q: Transaction trong database?

**A:** Nhóm các operations thành 1 unit. Tất cả thành công hoặc tất cả rollback (ACID).

```javascript
const session = await mongoose.startSession();
session.startTransaction();
try {
  await User.create([...], { session });
  await Order.create([...], { session });
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
}
```

---

## 7. Testing

### Q: Unit Test vs Integration Test vs E2E Test?

**A:**

- **Unit**: Test individual functions/modules
- **Integration**: Test multiple components together
- **E2E**: Test entire application flow

---

### Q: Mocking là gì?

**A:** Thay thế real dependencies bằng fake implementations để isolate unit being tested.

```javascript
jest.mock("axios");
axios.get.mockResolvedValue({ data: {} });
```

---

## 8. Practical Questions

### Q: Thiết kế REST API cho User CRUD?

**A:**

```
GET    /api/users      - List users
GET    /api/users/:id  - Get user
POST   /api/users      - Create user
PUT    /api/users/:id  - Update user
DELETE /api/users/:id  - Delete user
```

---

### Q: Implement rate limiting?

**A:**

```javascript
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
});

app.use("/api/", limiter);
```

---

### Q: Graceful shutdown?

**A:**

```javascript
process.on("SIGTERM", async () => {
  server.close(async () => {
    await db.end();
    await redis.quit();
    process.exit(0);
  });
});
```

---

## 9. Coding Challenges

### Implement Promise.all

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then((value) => {
          results[index] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

### Implement debounce

```javascript
function debounce(fn, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

### Implement throttle

```javascript
function throttle(fn, limit) {
  let inThrottle;
  return function (...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```

---

## 10. Tips for Interview

1. **Explain your thought process** - Nói ra cách bạn suy nghĩ
2. **Ask clarifying questions** - Hỏi để hiểu rõ yêu cầu
3. **Start simple, then optimize** - Giải pháp đơn giản trước
4. **Discuss trade-offs** - Ưu nhược điểm của các approaches
5. **Know your projects** - Chuẩn bị nói về projects đã làm
6. **Practice coding** - LeetCode, HackerRank
7. **Review fundamentals** - Event loop, async, closures
