# Node.js Interview Questions

## 1. Core Node.js

### Q: Event Loop là gì và hoạt động như thế nào?

**Trả lời:**

> Event Loop là cơ chế cho phép Node.js thực hiện non-blocking I/O mặc dù JavaScript single-threaded.
>
> Khi gặp async operation (đọc file, gọi API), Node.js đẩy xuống libuv xử lý. Code tiếp tục chạy không chờ. Khi operation hoàn thành, callback được đưa vào queue.
>
> Event Loop liên tục kiểm tra: Call Stack trống? → Lấy callback từ queue đưa vào thực thi.
>
> Có 6 phases: timers → pending callbacks → poll → check → close callbacks. Giữa mỗi phase, xử lý Microtask Queue.

---

### Q: process.nextTick() vs setImmediate()?

**Trả lời:**

> `process.nextTick()`: Chạy ngay sau operation hiện tại, trước khi Event Loop tiếp tục. Ưu tiên cao nhất trong Microtask Queue.
>
> `setImmediate()`: Chạy trong check phase của Event Loop iteration tiếp theo.
>
> Dùng nextTick khi cần chạy ngay lập tức. Dùng setImmediate cho recursive operations để tránh starve I/O.

---

### Q: Blocking vs Non-blocking I/O?

**Trả lời:**

> **Blocking:** Code dừng lại chờ operation hoàn thành. Ví dụ `fs.readFileSync()` - đọc xong mới chạy tiếp. Block Event Loop, các requests khác phải chờ.
>
> **Non-blocking:** Code tiếp tục chạy, callback được gọi khi xong. Ví dụ `fs.readFile()`. Đây là lý do Node.js handle được nhiều concurrent connections.

---

### Q: Libuv là gì?

**Trả lời:**

> Libuv là thư viện C++ cung cấp:
>
> - Event Loop implementation
> - Thread Pool (mặc định 4 threads) cho file system, DNS, crypto
> - Async I/O operations
> - Cross-platform support

---

## 2. Async Programming

### Q: Callback Hell là gì và cách giải quyết?

**Trả lời:**

> Callback Hell là tình trạng nested callbacks quá sâu, khó đọc và maintain.
>
> Giải quyết:
>
> 1. **Promises:** Chain `.then()` thay vì nest
> 2. **Async/Await:** Viết code async như sync
> 3. **Modularize:** Tách thành functions nhỏ

---

### Q: Promise.all vs Promise.allSettled vs Promise.race?

**Trả lời:**

> - `Promise.all`: Chờ TẤT CẢ resolve, reject ngay nếu 1 reject
> - `Promise.allSettled`: Chờ tất cả complete (không fail)
> - `Promise.race`: Trả về kết quả ĐẦU TIÊN (resolve hoặc reject)
> - `Promise.any`: Trả về fulfilled ĐẦU TIÊN (bỏ qua reject)

---

### Q: Xử lý error trong async/await?

**Trả lời:**

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

// .catch() on await
const data = await fetchData().catch((err) => null);
```

---

## 3. Modules

### Q: CommonJS vs ES Modules?

**Trả lời:**

> | Feature         | CommonJS                   | ES Modules        |
> | --------------- | -------------------------- | ----------------- |
> | Syntax          | `require`/`module.exports` | `import`/`export` |
> | Loading         | Synchronous                | Asynchronous      |
> | Top-level await | ❌                         | ✅                |
> | Tree shaking    | ❌                         | ✅                |
>
> ESM là standard mới, hỗ trợ static analysis và tree shaking.

---

### Q: Module caching hoạt động như thế nào?

**Trả lời:**

> Node.js cache modules sau lần require đầu tiên. Các lần require sau trả về cùng instance.
>
> ```javascript
> const a = require("./module");
> const b = require("./module");
> console.log(a === b); // true
> ```
>
> Lợi ích: Performance, singleton pattern, shared state.

---

## 4. Security

### Q: JWT hoạt động như thế nào?

**Trả lời:**

> JWT gồm 3 phần: Header.Payload.Signature
>
> Flow:
>
> 1. User login với credentials
> 2. Server verify và tạo JWT
> 3. Client lưu JWT và gửi trong header mỗi request
> 4. Server verify signature và extract payload
>
> JWT là stateless - server không cần lưu session.

---

### Q: Access Token vs Refresh Token?

**Trả lời:**

> - **Access Token:** Short-lived (15m), dùng để access resources
> - **Refresh Token:** Long-lived (7d), dùng để lấy access token mới
>
> Tại sao cần cả hai? Access token ngắn hạn giảm risk nếu bị leak. Refresh token cho phép user không cần login lại liên tục.
>
> Best practice: Token rotation - mỗi lần refresh, cả 2 tokens đều đổi mới.

---

### Q: Cách prevent SQL Injection?

**Trả lời:**

> 1. **Parameterized queries:**
>
> ```javascript
> const query = "SELECT * FROM users WHERE id = $1";
> await db.query(query, [id]);
> ```
>
> 2. **ORM:** Tự động escape
> 3. **Input validation:** Validate và sanitize input

---

## 5. Performance

### Q: Làm sao scale Node.js app?

**Trả lời:**

> 1. **Clustering:** Tận dụng multi-core CPU với PM2 cluster mode
> 2. **Load Balancing:** Nginx phân phối requests
> 3. **Horizontal Scaling:** Multiple servers
> 4. **Caching:** Redis để giảm database load
> 5. **Database:** Connection pooling, indexing, read replicas

---

### Q: N+1 Query Problem là gì?

**Trả lời:**

> Khi fetch N records, mỗi record cần thêm 1 query để lấy related data = N+1 queries.
>
> ```javascript
> // ❌ N+1: 101 queries cho 100 users
> const users = await User.find();
> for (const user of users) {
>   const posts = await Post.find({ authorId: user.id });
> }
>
> // ✅ Eager loading: 2 queries
> const users = await User.find().populate("posts");
> ```

---

### Q: Memory leak trong Node.js?

**Trả lời:**

> Nguyên nhân phổ biến:
>
> 1. Global variables tích tụ data
> 2. Event listeners không remove
> 3. Closures giữ references
> 4. Timers không clear
>
> Detect: `process.memoryUsage()`, heap snapshots
>
> Fix: Remove listeners, clear timers, avoid globals

---

## 6. Database

### Q: Connection Pooling là gì?

**Trả lời:**

> Tái sử dụng database connections thay vì tạo mới cho mỗi request.
>
> Lợi ích:
>
> - Giảm overhead tạo connection
> - Giới hạn số connections
> - Tăng performance
>
> ```javascript
> const pool = new Pool({ max: 20 });
> ```

---

### Q: Transaction trong database?

**Trả lời:**

> Transaction nhóm các operations thành 1 unit. Tất cả thành công hoặc tất cả rollback (ACID).
>
> ```javascript
> const session = await mongoose.startSession();
> session.startTransaction();
> try {
>   await User.create([...], { session });
>   await Order.create([...], { session });
>   await session.commitTransaction();
> } catch (error) {
>   await session.abortTransaction();
> }
> ```

---

## 7. Testing

### Q: Unit Test vs Integration Test vs E2E Test?

**Trả lời:**

> - **Unit:** Test từng function riêng lẻ, nhanh, nhiều
> - **Integration:** Test nhiều components kết hợp
> - **E2E:** Test toàn bộ flow từ đầu đến cuối
>
> Testing Pyramid: Nhiều unit tests, ít E2E tests.

---

### Q: Mocking là gì?

**Trả lời:**

> Thay thế real dependencies bằng fake implementations để isolate unit đang test.
>
> ```javascript
> jest.mock("axios");
> axios.get.mockResolvedValue({ data: {} });
> ```
>
> Lợi ích: Test nhanh, không phụ thuộc external services.

---

## 8. Practical Questions

### Q: Thiết kế REST API cho User CRUD?

**Trả lời:**

```
GET    /api/users      - List users (với pagination)
GET    /api/users/:id  - Get user by ID
POST   /api/users      - Create user
PUT    /api/users/:id  - Update user
DELETE /api/users/:id  - Delete user

Status codes:
- 200: Success
- 201: Created
- 400: Bad Request
- 401: Unauthorized
- 404: Not Found
- 500: Server Error
```

---

### Q: Implement rate limiting?

**Trả lời:**

```javascript
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 100, // 100 requests per window
  message: "Too many requests",
});

app.use("/api/", limiter);
```

---

### Q: Graceful shutdown?

**Trả lời:**

```javascript
process.on("SIGTERM", async () => {
  console.log("SIGTERM received");

  server.close(async () => {
    await db.end();
    await redis.quit();
    process.exit(0);
  });

  // Force shutdown after timeout
  setTimeout(() => process.exit(1), 30000);
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

    if (promises.length === 0) {
      resolve([]);
      return;
    }

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

---

## 11. Tổng kết

**Core:** Event Loop, Async, Modules

**Security:** JWT, Password hashing, SQL Injection

**Performance:** Clustering, Caching, N+1

**Database:** Connection pooling, Transactions

**Testing:** Unit, Integration, Mocking

**DevOps:** Docker, CI/CD, Logging
