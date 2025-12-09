# Event Loop & Cơ chế Bất đồng bộ trong Node.js

## 1. Event Loop là gì?

Event Loop là cơ chế cho phép Node.js thực hiện các non-blocking I/O operations mặc dù JavaScript là single-threaded. Nó liên tục kiểm tra Call Stack và Callback Queue để thực thi code.

## 2. Các thành phần chính

### Call Stack

- Nơi lưu trữ các function đang được thực thi
- LIFO (Last In, First Out)
- Khi function được gọi → push vào stack
- Khi function return → pop khỏi stack

### Callback Queue (Task Queue / Macrotask Queue)

- Chứa callbacks từ: `setTimeout`, `setInterval`, `setImmediate`, I/O operations
- Được xử lý sau khi Call Stack trống

### Microtask Queue

- Ưu tiên cao hơn Callback Queue
- Chứa: `Promise.then()`, `process.nextTick()`, `queueMicrotask()`
- Được xử lý ngay sau mỗi task trong Call Stack

## 3. Các phases của Event Loop

```
   ┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ← I/O callbacks deferred
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  ← internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  ← retrieve new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  ← setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  ← socket.on('close')
   └───────────────────────────┘
```

### Chi tiết từng phase:

1. **Timers**: Thực thi callbacks của `setTimeout()` và `setInterval()`
2. **Pending callbacks**: Thực thi I/O callbacks bị deferred từ loop trước
3. **Idle, prepare**: Chỉ dùng nội bộ
4. **Poll**: Lấy I/O events mới, thực thi I/O callbacks
5. **Check**: Thực thi `setImmediate()` callbacks
6. **Close callbacks**: Thực thi close callbacks như `socket.on('close')`

## 4. process.nextTick() vs setImmediate() vs setTimeout()

```javascript
console.log("1: Start");

setTimeout(() => console.log("2: setTimeout"), 0);

setImmediate(() => console.log("3: setImmediate"));

process.nextTick(() => console.log("4: nextTick"));

Promise.resolve().then(() => console.log("5: Promise"));

console.log("6: End");

// Output:
// 1: Start
// 6: End
// 4: nextTick
// 5: Promise
// 2: setTimeout (hoặc 3)
// 3: setImmediate (hoặc 2)
```

### Thứ tự ưu tiên:

1. **Synchronous code** (Call Stack)
2. **process.nextTick()** - Microtask, ưu tiên cao nhất
3. **Promise callbacks** - Microtask
4. **setTimeout/setInterval** - Macrotask (timers phase)
5. **setImmediate** - Macrotask (check phase)

### Khi nào dùng gì?

| Method               | Khi nào dùng                                       |
| -------------------- | -------------------------------------------------- |
| `process.nextTick()` | Cần chạy ngay sau operation hiện tại, trước I/O    |
| `setImmediate()`     | Cần chạy trong iteration tiếp theo của event loop  |
| `setTimeout(fn, 0)`  | Defer execution, không quan trọng timing chính xác |

## 5. Blocking vs Non-blocking I/O

### Blocking (Synchronous)

```javascript
const fs = require("fs");

// Blocking - chờ đọc xong mới chạy tiếp
const data = fs.readFileSync("/file.txt");
console.log(data);
console.log("Done"); // Chạy sau khi đọc xong
```

### Non-blocking (Asynchronous)

```javascript
const fs = require("fs");

// Non-blocking - không chờ
fs.readFile("/file.txt", (err, data) => {
  console.log(data);
});
console.log("Done"); // Chạy ngay lập tức
```

## 6. Libuv

Libuv là thư viện C++ cung cấp:

- Event loop implementation
- Asynchronous I/O
- Thread pool (mặc định 4 threads) cho các operations như:
  - File system operations
  - DNS lookups
  - Crypto operations
  - Compression

```javascript
// Tăng thread pool size
process.env.UV_THREADPOOL_SIZE = 8;
```

## 7. Ví dụ thực tế

### Vấn đề: Blocking Event Loop

```javascript
// ❌ Bad - blocks event loop
app.get("/hash", (req, res) => {
  const hash = crypto.pbkdf2Sync(password, salt, 100000, 64, "sha512");
  res.send(hash);
});

// ✅ Good - non-blocking
app.get("/hash", (req, res) => {
  crypto.pbkdf2(password, salt, 100000, 64, "sha512", (err, hash) => {
    res.send(hash);
  });
});
```

### Vấn đề: nextTick starvation

```javascript
// ❌ Bad - I/O sẽ không bao giờ được xử lý
function recursiveNextTick() {
  process.nextTick(recursiveNextTick);
}
recursiveNextTick();

// ✅ Good - dùng setImmediate để cho I/O cơ hội chạy
function recursiveImmediate() {
  setImmediate(recursiveImmediate);
}
```

## 8. Câu hỏi phỏng vấn thường gặp

1. **Event loop là gì và hoạt động như thế nào?**
2. **Sự khác biệt giữa process.nextTick() và setImmediate()?**
3. **Microtask và Macrotask khác nhau như thế nào?**
4. **Tại sao Node.js là single-threaded nhưng vẫn handle được nhiều requests?**
5. **Libuv là gì và vai trò của nó?**
6. **Làm sao để tránh blocking event loop?**

## 9. Tips

- Tránh synchronous operations trong production
- Dùng `setImmediate()` thay vì `process.nextTick()` cho recursive operations
- Monitor event loop lag để phát hiện blocking
- Offload CPU-intensive tasks sang Worker Threads
