# Async Patterns trong Node.js

## 1. Callback Pattern

### Cơ bản

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback(null, { data: "Hello" });
  }, 1000);
}

fetchData((err, result) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(result);
});
```

### Error-first Callback Convention

```javascript
// Node.js convention: error là argument đầu tiên
fs.readFile("file.txt", (err, data) => {
  if (err) {
    // Handle error
    return;
  }
  // Use data
});
```

### Callback Hell (Pyramid of Doom)

```javascript
// ❌ Bad - Callback Hell
getUser(userId, (err, user) => {
  getOrders(user.id, (err, orders) => {
    getOrderDetails(orders[0].id, (err, details) => {
      getProduct(details.productId, (err, product) => {
        console.log(product);
      });
    });
  });
});
```

## 2. Promise Pattern

### Tạo Promise

```javascript
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;
      if (success) {
        resolve({ data: "Hello" });
      } else {
        reject(new Error("Failed"));
      }
    }, 1000);
  });
};
```

### Promise Chaining

```javascript
// ✅ Good - Promise chain
getUser(userId)
  .then((user) => getOrders(user.id))
  .then((orders) => getOrderDetails(orders[0].id))
  .then((details) => getProduct(details.productId))
  .then((product) => console.log(product))
  .catch((err) => console.error(err));
```

### Promise Static Methods

```javascript
// Promise.all - Chờ tất cả resolve, fail nếu 1 reject
const results = await Promise.all([fetchUser(), fetchOrders(), fetchProducts()]);
// results = [user, orders, products]

// Promise.allSettled - Chờ tất cả complete (không fail)
const results = await Promise.allSettled([
  fetchUser(),
  fetchOrders(), // có thể reject
  fetchProducts(),
]);
// results = [
//   { status: 'fulfilled', value: user },
//   { status: 'rejected', reason: Error },
//   { status: 'fulfilled', value: products }
// ]

// Promise.race - Trả về kết quả đầu tiên (resolve hoặc reject)
const fastest = await Promise.race([fetchFromServer1(), fetchFromServer2()]);

// Promise.any - Trả về fulfilled đầu tiên (ES2021)
const firstSuccess = await Promise.any([
  fetchFromServer1(), // reject
  fetchFromServer2(), // resolve first
  fetchFromServer3(),
]);
```

### Promisify

```javascript
const { promisify } = require("util");
const fs = require("fs");

// Convert callback-based to Promise-based
const readFileAsync = promisify(fs.readFile);

const data = await readFileAsync("file.txt", "utf8");
```

## 3. Async/Await Pattern

### Cơ bản

```javascript
async function fetchUserData(userId) {
  try {
    const user = await getUser(userId);
    const orders = await getOrders(user.id);
    const details = await getOrderDetails(orders[0].id);
    return details;
  } catch (error) {
    console.error("Error:", error.message);
    throw error;
  }
}
```

### Parallel Execution

```javascript
// ❌ Sequential - chậm
async function fetchAll() {
  const user = await fetchUser(); // 1s
  const orders = await fetchOrders(); // 1s
  const products = await fetchProducts(); // 1s
  // Total: 3s
}

// ✅ Parallel - nhanh
async function fetchAll() {
  const [user, orders, products] = await Promise.all([
    fetchUser(), // 1s
    fetchOrders(), // 1s  } chạy đồng thời
    fetchProducts(), // 1s
  ]);
  // Total: 1s
}
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

// Pattern 2: .catch() on promise
async function getData() {
  const data = await fetchData().catch((err) => {
    console.error(err);
    return null;
  });
  return data;
}

// Pattern 3: Wrapper function (Go-style)
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

### Async trong loops

```javascript
const ids = [1, 2, 3, 4, 5];

// ❌ forEach không chờ async
ids.forEach(async (id) => {
  await processItem(id); // Không chờ!
});

// ✅ for...of - sequential
for (const id of ids) {
  await processItem(id);
}

// ✅ Promise.all + map - parallel
await Promise.all(ids.map((id) => processItem(id)));

// ✅ Controlled concurrency với p-limit
const pLimit = require("p-limit");
const limit = pLimit(3); // Max 3 concurrent

await Promise.all(ids.map((id) => limit(() => processItem(id))));
```

## 4. Event Emitter Pattern

```javascript
const EventEmitter = require("events");

class OrderService extends EventEmitter {
  createOrder(orderData) {
    // Create order logic
    const order = { id: 1, ...orderData };

    // Emit events
    this.emit("orderCreated", order);
    return order;
  }
}

const orderService = new OrderService();

// Subscribe to events
orderService.on("orderCreated", (order) => {
  console.log("Send email for order:", order.id);
});

orderService.on("orderCreated", (order) => {
  console.log("Update inventory for order:", order.id);
});

// Once - chỉ listen 1 lần
orderService.once("orderCreated", (order) => {
  console.log("First order celebration!");
});

// Create order
orderService.createOrder({ product: "iPhone" });
```

### Event Emitter Methods

```javascript
emitter.on(event, listener); // Add listener
emitter.once(event, listener); // Add one-time listener
emitter.off(event, listener); // Remove listener
emitter.emit(event, ...args); // Trigger event
emitter.removeAllListeners(); // Remove all
emitter.listenerCount(event); // Count listeners
emitter.setMaxListeners(n); // Set max (default 10)
```

## 5. Stream Pattern

```javascript
const fs = require("fs");

// Readable Stream
const readable = fs.createReadStream("large-file.txt");

readable.on("data", (chunk) => {
  console.log("Received chunk:", chunk.length);
});

readable.on("end", () => {
  console.log("Done reading");
});

// Pipe - kết nối streams
const writable = fs.createWriteStream("output.txt");
readable.pipe(writable);

// Pipeline (recommended) - với error handling
const { pipeline } = require("stream/promises");
const zlib = require("zlib");

await pipeline(fs.createReadStream("input.txt"), zlib.createGzip(), fs.createWriteStream("output.txt.gz"));
```

## 6. So sánh các patterns

| Pattern       | Ưu điểm                          | Nhược điểm                        | Khi nào dùng                   |
| ------------- | -------------------------------- | --------------------------------- | ------------------------------ |
| Callback      | Simple, native                   | Callback hell, error handling khó | Legacy code, simple operations |
| Promise       | Chainable, better error handling | Verbose với nhiều operations      | Khi cần chain operations       |
| Async/Await   | Clean, readable, try/catch       | Cần transpile cho old Node        | Default choice                 |
| Event Emitter | Decoupled, multiple listeners    | Memory leaks nếu không cleanup    | Pub/sub, real-time             |
| Stream        | Memory efficient                 | Complex                           | Large files, real-time data    |

## 7. Best Practices

1. **Luôn handle errors** - Unhandled rejections sẽ crash app
2. **Tránh mixing patterns** - Chọn 1 pattern và consistent
3. **Dùng async/await** làm default
4. **Parallel khi có thể** - Dùng Promise.all cho independent operations
5. **Cleanup event listeners** - Tránh memory leaks
6. **Set timeout cho async operations** - Tránh hanging forever

```javascript
// Timeout wrapper
const withTimeout = (promise, ms) => {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error("Timeout")), ms);
  });
  return Promise.race([promise, timeout]);
};

const data = await withTimeout(fetchData(), 5000);
```
