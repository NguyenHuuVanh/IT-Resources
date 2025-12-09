# Async Patterns trong Node.js

## 1. Tổng quan

### Tại sao cần Async?

Node.js là single-threaded, nếu code chờ đợi (blocking) thì toàn bộ ứng dụng sẽ đứng yên. Async patterns cho phép code tiếp tục chạy trong khi chờ I/O operations hoàn thành.

### Các patterns chính

| Pattern       | Ra đời       | Đặc điểm                         |
| ------------- | ------------ | -------------------------------- |
| Callback      | Đầu tiên     | Đơn giản nhưng dễ callback hell  |
| Promise       | ES6 (2015)   | Chainable, better error handling |
| Async/Await   | ES8 (2017)   | Clean syntax, try/catch          |
| Event Emitter | Node.js core | Pub/sub pattern                  |
| Stream        | Node.js core | Xử lý data theo chunks           |

## 2. Callback Pattern

### Khái niệm

**Callback** là function được truyền vào function khác và được gọi khi operation hoàn thành.

**Error-first Callback Convention:** Trong Node.js, callback luôn có error là argument đầu tiên.

```javascript
// Cấu trúc: callback(error, result)
fs.readFile("file.txt", (err, data) => {
  if (err) {
    console.error("Lỗi:", err);
    return;
  }
  console.log("Data:", data);
});
```

### Tại sao Error-first?

```javascript
// Buộc developer phải handle error trước
function fetchUser(id, callback) {
  db.query("SELECT * FROM users WHERE id = ?", [id], (err, rows) => {
    if (err) {
      return callback(err, null); // Error first
    }
    callback(null, rows[0]); // null = no error
  });
}

// Sử dụng
fetchUser(1, (err, user) => {
  if (err) {
    console.error("Không thể lấy user:", err);
    return;
  }
  console.log("User:", user);
});
```

### Callback Hell (Pyramid of Doom)

**Vấn đề:** Khi có nhiều async operations phụ thuộc nhau, code bị nested sâu.

```javascript
// ❌ Callback Hell
getUser(userId, (err, user) => {
  if (err) return handleError(err);

  getOrders(user.id, (err, orders) => {
    if (err) return handleError(err);

    getOrderDetails(orders[0].id, (err, details) => {
      if (err) return handleError(err);

      getProduct(details.productId, (err, product) => {
        if (err) return handleError(err);

        getReviews(product.id, (err, reviews) => {
          if (err) return handleError(err);

          console.log("Reviews:", reviews);
          // Còn tiếp...
        });
      });
    });
  });
});
```

**Vấn đề:**

- Khó đọc, khó maintain
- Error handling lặp lại
- Khó debug
- Khó thêm logic

## 3. Promise Pattern

### Khái niệm

**Promise** là object đại diện cho kết quả của async operation, có thể ở 3 trạng thái:

- **Pending:** Đang chờ
- **Fulfilled:** Thành công (resolved)
- **Rejected:** Thất bại

### Tạo Promise

```javascript
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    db.query("SELECT * FROM users WHERE id = ?", [id], (err, rows) => {
      if (err) {
        reject(err); // Thất bại
      } else {
        resolve(rows[0]); // Thành công
      }
    });
  });
}

// Sử dụng
fetchUser(1)
  .then((user) => console.log("User:", user))
  .catch((err) => console.error("Error:", err));
```

### Promise Chaining

**Tác dụng:** Giải quyết callback hell bằng cách chain các operations.

```javascript
// ✅ Promise Chain - Flat, dễ đọc
getUser(userId)
  .then((user) => {
    console.log("Got user:", user.name);
    return getOrders(user.id);
  })
  .then((orders) => {
    console.log("Got orders:", orders.length);
    return getOrderDetails(orders[0].id);
  })
  .then((details) => {
    console.log("Got details:", details);
    return getProduct(details.productId);
  })
  .then((product) => {
    console.log("Product:", product.name);
  })
  .catch((err) => {
    // Handle tất cả errors ở một chỗ
    console.error("Error:", err);
  })
  .finally(() => {
    console.log("Done!");
  });
```

### Promise Static Methods

```javascript
// Promise.all - Chờ TẤT CẢ resolve, fail nếu 1 reject
const [user, orders, products] = await Promise.all([fetchUser(1), fetchOrders(1), fetchProducts()]);
// Tất cả chạy đồng thời, nhanh hơn sequential

// Promise.allSettled - Chờ tất cả complete (không fail)
const results = await Promise.allSettled([
  fetchUser(1), // success
  fetchOrders(999), // fail
  fetchProducts(), // success
]);
// results = [
//   { status: 'fulfilled', value: user },
//   { status: 'rejected', reason: Error },
//   { status: 'fulfilled', value: products }
// ]

// Promise.race - Trả về kết quả ĐẦU TIÊN (resolve hoặc reject)
const fastest = await Promise.race([
  fetchFromServer1(), // 100ms
  fetchFromServer2(), // 50ms ← Winner
  fetchFromServer3(), // 200ms
]);

// Promise.any - Trả về fulfilled ĐẦU TIÊN (bỏ qua reject)
const firstSuccess = await Promise.any([
  fetchFromServer1(), // reject
  fetchFromServer2(), // resolve ← Winner
  fetchFromServer3(), // resolve (slower)
]);
```

### Promisify - Chuyển Callback thành Promise

```javascript
const { promisify } = require("util");
const fs = require("fs");

// Chuyển callback-based thành Promise-based
const readFileAsync = promisify(fs.readFile);
const writeFileAsync = promisify(fs.writeFile);

// Sử dụng
const data = await readFileAsync("file.txt", "utf8");
await writeFileAsync("output.txt", data);

// Hoặc dùng fs.promises (Node.js 10+)
const fsPromises = require("fs").promises;
const data = await fsPromises.readFile("file.txt", "utf8");
```

## 4. Async/Await Pattern

### Khái niệm

**Async/Await** là syntax sugar trên Promise, cho phép viết async code như synchronous code.

- `async`: Đánh dấu function là asynchronous, luôn return Promise
- `await`: Chờ Promise resolve, chỉ dùng trong async function

### Cú pháp cơ bản

```javascript
// Khai báo async function
async function fetchUserData(userId) {
  // await chờ Promise resolve
  const user = await getUser(userId);
  const orders = await getOrders(user.id);
  const details = await getOrderDetails(orders[0].id);

  return { user, orders, details };
}

// Arrow function
const fetchUserData = async (userId) => {
  const user = await getUser(userId);
  return user;
};

// Sử dụng
const data = await fetchUserData(1);
console.log(data);
```

### Error Handling với try/catch

```javascript
async function fetchUserData(userId) {
  try {
    const user = await getUser(userId);
    const orders = await getOrders(user.id);
    return { user, orders };
  } catch (error) {
    console.error("Error fetching data:", error);
    throw error; // Re-throw nếu cần
  } finally {
    console.log("Cleanup...");
  }
}
```

### Parallel vs Sequential

```javascript
// ❌ Sequential - Chậm (3 giây)
async function fetchAllSequential() {
  const user = await fetchUser(); // 1 giây
  const orders = await fetchOrders(); // 1 giây
  const products = await fetchProducts(); // 1 giây
  return { user, orders, products };
}

// ✅ Parallel - Nhanh (1 giây)
async function fetchAllParallel() {
  const [user, orders, products] = await Promise.all([
    fetchUser(), // 1 giây ┐
    fetchOrders(), // 1 giây ├─ Chạy đồng thời
    fetchProducts(), // 1 giây ┘
  ]);
  return { user, orders, products };
}
```

### Async trong Loops

```javascript
const ids = [1, 2, 3, 4, 5];

// ❌ forEach KHÔNG chờ async
ids.forEach(async (id) => {
  await processItem(id); // Không chờ!
});
console.log("Done"); // Chạy ngay, không chờ forEach

// ✅ for...of - Sequential (lần lượt)
for (const id of ids) {
  await processItem(id);
}
console.log("Done"); // Chờ tất cả xong

// ✅ Promise.all + map - Parallel (đồng thời)
await Promise.all(ids.map((id) => processItem(id)));
console.log("Done");

// ✅ Controlled concurrency (giới hạn số lượng đồng thời)
const pLimit = require("p-limit");
const limit = pLimit(3); // Max 3 concurrent

await Promise.all(ids.map((id) => limit(() => processItem(id))));
```

### Error Handling Patterns

```javascript
// Pattern 1: try/catch
async function getData() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error(error);
    return null;
  }
}

// Pattern 2: .catch() on await
async function getData() {
  const data = await fetchData().catch((err) => {
    console.error(err);
    return null;
  });
  return data;
}

// Pattern 3: Go-style [error, data]
const to = (promise) => {
  return promise.then((data) => [null, data]).catch((err) => [err, null]);
};

async function getData() {
  const [err, data] = await to(fetchData());

  if (err) {
    console.error(err);
    return null;
  }

  return data;
}
```

## 5. Event Emitter Pattern

### Khái niệm

**Event Emitter** là pattern pub/sub (publish/subscribe) cho phép objects emit events và các listeners subscribe để nhận events đó.

**Tác dụng:**

- Decoupling: Emitter không cần biết ai đang listen
- Multiple listeners: Nhiều handlers cho 1 event
- Async communication: Không cần callback trực tiếp

### Cách sử dụng

```javascript
const EventEmitter = require("events");

// Tạo emitter
const emitter = new EventEmitter();

// Subscribe (listen) events
emitter.on("userCreated", (user) => {
  console.log("Send welcome email to:", user.email);
});

emitter.on("userCreated", (user) => {
  console.log("Create default settings for:", user.id);
});

emitter.on("userCreated", (user) => {
  console.log("Log analytics for:", user.id);
});

// Emit event
emitter.emit("userCreated", { id: 1, email: "test@test.com" });

// Output:
// Send welcome email to: test@test.com
// Create default settings for: 1
// Log analytics for: 1
```

### Tạo Custom Event Emitter

```javascript
const EventEmitter = require('events');

class OrderService extends EventEmitter {
  async createOrder(orderData) {
    // Business logic
    const order = await this.saveOrder(orderData);

    // Emit events - các services khác sẽ handle
    this.emit('orderCreated', order);

    return order;
  }

  async cancelOrder(orderId) {
    const order = await this.getOrder(orderId);
    order.status = 'cancelled';
    await this.saveOrder(order);

    this.emit('orderCancelled', order);

    return order;
  }
}

// Sử dụng
const orderService = new OrderService();

// Email service listens
orderService.on('orderCreated', async (order) => {
  await sendOrderConfirmationEmail(order);
});

// Inventory service listens
orderService.on('orderCreated', async (order) => {
  await updateInventory(order.items);
});

// Analytics service listens
orderService.on('orderCreated', (order) => {
  trackEvent('order_created', { orderId: order.id });
});

// Payment service listens
orderService.on('orderCancelled', async (order) => {
  await refundPayment(order.paymentId);
});

// Tạo order - tất cả listeners sẽ được gọi
await orderService.createOrder({ items: [...], userId: 1 });
```

### Event Emitter Methods

```javascript
const emitter = new EventEmitter();

// on - Listen event (nhiều lần)
emitter.on("event", handler);

// once - Listen chỉ 1 lần
emitter.once("event", handler);

// off / removeListener - Bỏ listen
emitter.off("event", handler);

// emit - Phát event
emitter.emit("event", data1, data2);

// removeAllListeners - Xóa tất cả listeners
emitter.removeAllListeners("event");

// listenerCount - Đếm listeners
emitter.listenerCount("event");

// setMaxListeners - Set max (mặc định 10)
emitter.setMaxListeners(20);
```

### Lưu ý Memory Leak

```javascript
// ⚠️ Warning: Possible memory leak
// Nếu add quá nhiều listeners mà không remove

// Tăng limit nếu cần
emitter.setMaxListeners(50);

// Hoặc remove listener khi không cần
const handler = (data) => console.log(data);
emitter.on("event", handler);

// Cleanup
emitter.off("event", handler);
```

## 6. Stream Pattern

### Khái niệm

**Stream** là cách xử lý data theo từng chunks (mảnh) thay vì load toàn bộ vào memory.

**Tại sao cần Stream?**

```javascript
// ❌ Bad: Load toàn bộ file 2GB vào memory
const data = fs.readFileSync("large-file.txt"); // 2GB in memory!

// ✅ Good: Đọc từng chunk 64KB
const stream = fs.createReadStream("large-file.txt");
stream.on("data", (chunk) => {
  // Xử lý chunk 64KB
});
```

### 4 loại Stream

| Loại          | Mô tả              | Ví dụ                               |
| ------------- | ------------------ | ----------------------------------- |
| **Readable**  | Đọc data           | fs.createReadStream, http request   |
| **Writable**  | Ghi data           | fs.createWriteStream, http response |
| **Duplex**    | Đọc và ghi         | TCP socket                          |
| **Transform** | Đọc, biến đổi, ghi | zlib.createGzip                     |

### Readable Stream

```javascript
const fs = require("fs");

const readable = fs.createReadStream("large-file.txt", {
  encoding: "utf8",
  highWaterMark: 64 * 1024, // 64KB per chunk
});

readable.on("data", (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

readable.on("end", () => {
  console.log("Done reading");
});

readable.on("error", (err) => {
  console.error("Error:", err);
});
```

### Writable Stream

```javascript
const fs = require("fs");

const writable = fs.createWriteStream("output.txt");

writable.write("Hello ");
writable.write("World!");
writable.end(); // Kết thúc stream

writable.on("finish", () => {
  console.log("Done writing");
});
```

### Pipe - Kết nối Streams

```javascript
const fs = require("fs");
const zlib = require("zlib");

// Đọc file → Nén → Ghi file
fs.createReadStream("input.txt").pipe(zlib.createGzip()).pipe(fs.createWriteStream("input.txt.gz"));

// HTTP response với stream
app.get("/video", (req, res) => {
  const videoStream = fs.createReadStream("video.mp4");
  videoStream.pipe(res);
});
```

### Pipeline (Recommended)

```javascript
const { pipeline } = require("stream/promises");
const fs = require("fs");
const zlib = require("zlib");

async function compressFile(input, output) {
  await pipeline(fs.createReadStream(input), zlib.createGzip(), fs.createWriteStream(output));
  console.log("Compression complete");
}

// Tự động handle errors
compressFile("input.txt", "input.txt.gz").catch(console.error);
```

## 7. So sánh các Patterns

| Pattern           | Ưu điểm                       | Nhược điểm         | Khi nào dùng       |
| ----------------- | ----------------------------- | ------------------ | ------------------ |
| **Callback**      | Đơn giản, native              | Callback hell      | Legacy code        |
| **Promise**       | Chainable, error handling     | Verbose            | Chain operations   |
| **Async/Await**   | Clean, try/catch              | Cần transpile (cũ) | Default choice     |
| **Event Emitter** | Decoupled, multiple listeners | Memory leaks       | Pub/sub, real-time |
| **Stream**        | Memory efficient              | Complex            | Large files        |

## 8. Best Practices

### 1. Luôn handle errors

```javascript
// ❌ Bad: Unhandled rejection
async function bad() {
  const data = await fetchData(); // Nếu fail → crash
}

// ✅ Good: Handle error
async function good() {
  try {
    const data = await fetchData();
  } catch (error) {
    console.error("Error:", error);
  }
}
```

### 2. Dùng Async/Await làm default

```javascript
// ✅ Preferred
async function getData() {
  const user = await getUser();
  const orders = await getOrders(user.id);
  return { user, orders };
}
```

### 3. Parallel khi có thể

```javascript
// ✅ Parallel cho independent operations
const [user, products, settings] = await Promise.all([getUser(), getProducts(), getSettings()]);
```

### 4. Cleanup Event Listeners

```javascript
// ✅ Remove listener khi không cần
const handler = (data) => console.log(data);
emitter.on("event", handler);

// Cleanup
emitter.off("event", handler);
```

### 5. Timeout cho async operations

```javascript
// ✅ Tránh chờ vô hạn
const withTimeout = (promise, ms) => {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error("Timeout")), ms);
  });
  return Promise.race([promise, timeout]);
};

const data = await withTimeout(fetchData(), 5000);
```

## 9. Tổng kết

**Callback:** Pattern cơ bản, error-first convention, dễ callback hell.

**Promise:** Chainable, better error handling, Promise.all cho parallel.

**Async/Await:** Clean syntax, try/catch, default choice cho modern code.

**Event Emitter:** Pub/sub pattern, decoupled, multiple listeners.

**Stream:** Memory efficient, xử lý large data theo chunks.

**Nhớ:**

- Async/Await là default choice
- Promise.all cho parallel operations
- Handle errors luôn
- Cleanup listeners
- Stream cho large files
