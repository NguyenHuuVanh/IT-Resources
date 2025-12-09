# Event Loop & Cơ chế Bất đồng bộ trong Node.js

## 1. Khái niệm cơ bản

### Node.js là Single-threaded

**Khái niệm:** Node.js chạy JavaScript trên một thread duy nhất (main thread). Điều này có nghĩa tại một thời điểm, chỉ có một đoạn code được thực thi.

**Câu hỏi:** Vậy làm sao Node.js xử lý được hàng nghìn requests đồng thời?

**Trả lời:** Nhờ cơ chế **Non-blocking I/O** và **Event Loop**.

### Blocking vs Non-blocking I/O

**Blocking I/O (Đồng bộ):**

```javascript
// Code dừng lại chờ đọc file xong mới chạy tiếp
const data = fs.readFileSync("file.txt"); // Chờ ở đây
console.log(data); // Chạy sau khi đọc xong
console.log("Done"); // Chạy cuối cùng
```

**Non-blocking I/O (Bất đồng bộ):**

```javascript
// Code tiếp tục chạy, không chờ đọc file
fs.readFile("file.txt", (err, data) => {
  console.log(data); // Chạy khi đọc xong
});
console.log("Done"); // Chạy ngay lập tức!
```

**Tại sao Non-blocking quan trọng?**

- Blocking: 1 request đọc file 1 giây → 1000 requests = 1000 giây
- Non-blocking: 1000 requests đọc file đồng thời → ~1 giây

## 2. Event Loop là gì?

**Khái niệm:** Event Loop là cơ chế cho phép Node.js thực hiện non-blocking I/O operations mặc dù JavaScript là single-threaded.

**Cách hoạt động đơn giản:**

1. Nhận code JavaScript
2. Thực thi code đồng bộ
3. Khi gặp async operation (đọc file, gọi API...) → đẩy xuống background
4. Tiếp tục thực thi code tiếp theo
5. Khi async operation hoàn thành → callback được đưa vào queue
6. Event Loop kiểm tra: Call Stack trống? → Lấy callback từ queue đưa vào Stack thực thi

## 3. Các thành phần chính

### Call Stack - "Ngăn xếp thực thi"

**Khái niệm:** Call Stack là nơi lưu trữ các function đang được thực thi. Hoạt động theo nguyên tắc LIFO (Last In, First Out).

**Cách hoạt động:**

- Function được gọi → Push vào stack
- Function return → Pop khỏi stack

```javascript
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  const result = square(n);
  console.log(result);
}

printSquare(4);

// Call Stack thay đổi:
// 1. [printSquare]
// 2. [printSquare, square]
// 3. [printSquare, square, multiply]
// 4. [printSquare, square] ← multiply return
// 5. [printSquare] ← square return
// 6. [] ← printSquare return
```

### Web APIs / Node APIs - "Nơi xử lý async"

**Khái niệm:** Các APIs do browser (Web APIs) hoặc Node.js (libuv) cung cấp để xử lý các operations bất đồng bộ.

**Bao gồm:**

- setTimeout, setInterval
- HTTP requests (fetch, axios)
- File system operations
- Database queries

**Cách hoạt động:**

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout");
}, 1000);

console.log("End");

// 1. console.log('Start') → Call Stack → thực thi → "Start"
// 2. setTimeout → Call Stack → đẩy xuống Web API (đếm 1 giây)
// 3. console.log('End') → Call Stack → thực thi → "End"
// 4. Sau 1 giây, callback được đưa vào Callback Queue
// 5. Event Loop: Stack trống? → Lấy callback → thực thi → "Timeout"
```

### Callback Queue (Task Queue / Macrotask Queue)

**Khái niệm:** Hàng đợi chứa các callbacks từ async operations, chờ được đưa vào Call Stack.

**Chứa callbacks từ:**

- setTimeout, setInterval
- setImmediate (Node.js)
- I/O operations
- UI rendering (browser)

### Microtask Queue

**Khái niệm:** Hàng đợi có độ ưu tiên cao hơn Callback Queue, được xử lý ngay sau mỗi task trong Call Stack.

**Chứa callbacks từ:**

- Promise.then(), Promise.catch(), Promise.finally()
- process.nextTick() (Node.js)
- queueMicrotask()
- MutationObserver (browser)

**Thứ tự ưu tiên:**

```
1. Call Stack (Synchronous code)
2. Microtask Queue (process.nextTick > Promise)
3. Callback Queue (setTimeout, I/O...)
```

## 4. Các Phases của Event Loop

```
   ┌───────────────────────────┐
┌─►│         timers            │ ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │ ← I/O callbacks bị defer
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │ ← internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │ ← Lấy I/O events mới
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │ ← setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │ ← socket.on('close')
   └───────────────────────────┘
```

### Chi tiết từng phase:

**1. Timers Phase**

- Thực thi callbacks của `setTimeout()` và `setInterval()`
- Callbacks được thực thi khi thời gian đã đến

**2. Pending Callbacks Phase**

- Thực thi I/O callbacks bị deferred từ loop trước
- Ví dụ: TCP errors

**3. Poll Phase**

- Lấy I/O events mới từ hệ điều hành
- Thực thi I/O callbacks (đọc file, network...)
- Nếu không có gì, sẽ chờ ở đây

**4. Check Phase**

- Thực thi `setImmediate()` callbacks
- Chạy ngay sau poll phase

**5. Close Callbacks Phase**

- Thực thi close callbacks
- Ví dụ: `socket.on('close', ...)`

## 5. process.nextTick() vs setImmediate() vs setTimeout()

### Khái niệm

**process.nextTick():**

- Chạy ngay sau operation hiện tại
- Trước khi Event Loop tiếp tục
- Thuộc Microtask Queue
- Ưu tiên cao nhất

**setImmediate():**

- Chạy trong check phase của Event Loop
- Sau poll phase
- Thuộc Macrotask Queue

**setTimeout(fn, 0):**

- Chạy trong timers phase
- Minimum delay ~1ms (không phải 0ms thật)
- Thuộc Macrotask Queue

### Ví dụ so sánh

```javascript
console.log("1: Start");

setTimeout(() => {
  console.log("2: setTimeout");
}, 0);

setImmediate(() => {
  console.log("3: setImmediate");
});

process.nextTick(() => {
  console.log("4: nextTick");
});

Promise.resolve().then(() => {
  console.log("5: Promise");
});

console.log("6: End");

// Output:
// 1: Start
// 6: End
// 4: nextTick      ← Microtask, ưu tiên cao nhất
// 5: Promise       ← Microtask
// 2: setTimeout    ← Macrotask (thứ tự có thể đổi với setImmediate)
// 3: setImmediate  ← Macrotask
```

### Khi nào dùng gì?

| Method               | Khi nào dùng                                    |
| -------------------- | ----------------------------------------------- |
| `process.nextTick()` | Cần chạy ngay sau operation hiện tại, trước I/O |
| `setImmediate()`     | Cần chạy sau I/O events, an toàn cho recursive  |
| `setTimeout(fn, 0)`  | Defer execution, không quan trọng timing        |

### Cảnh báo: nextTick starvation

```javascript
// ❌ Bad: I/O sẽ không bao giờ được xử lý
function recursiveNextTick() {
  process.nextTick(recursiveNextTick);
}
recursiveNextTick();

// ✅ Good: Cho I/O cơ hội chạy
function recursiveImmediate() {
  setImmediate(recursiveImmediate);
}
recursiveImmediate();
```

## 6. Libuv là gì?

**Khái niệm:** Libuv là thư viện C++ cung cấp Event Loop và async I/O cho Node.js.

**Vai trò:**

- Implement Event Loop
- Cung cấp Thread Pool (mặc định 4 threads)
- Handle async I/O operations
- Cross-platform support

**Thread Pool dùng cho:**

- File system operations (fs.readFile, fs.writeFile...)
- DNS lookups (dns.lookup)
- Crypto operations (crypto.pbkdf2, crypto.randomBytes...)
- Zlib compression

```javascript
// Tăng thread pool size nếu cần
process.env.UV_THREADPOOL_SIZE = 8; // Mặc định là 4
```

## 7. Ví dụ thực tế

### Ví dụ 1: Thứ tự thực thi

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve().then(() => console.log("3"));

process.nextTick(() => console.log("4"));

setImmediate(() => console.log("5"));

console.log("6");

// Output: 1, 6, 4, 3, 2, 5
// Giải thích:
// - 1, 6: Synchronous, chạy ngay
// - 4: nextTick, ưu tiên cao nhất trong microtask
// - 3: Promise, microtask
// - 2: setTimeout, macrotask (timers phase)
// - 5: setImmediate, macrotask (check phase)
```

### Ví dụ 2: Nested callbacks

```javascript
setTimeout(() => {
  console.log("timeout 1");

  Promise.resolve().then(() => {
    console.log("promise inside timeout");
  });
}, 0);

Promise.resolve().then(() => {
  console.log("promise 1");

  setTimeout(() => {
    console.log("timeout inside promise");
  }, 0);
});

// Output:
// promise 1
// timeout 1
// promise inside timeout
// timeout inside promise
```

### Ví dụ 3: Blocking Event Loop

```javascript
// ❌ Bad: Blocking Event Loop
app.get("/hash", (req, res) => {
  // Synchronous - block tất cả requests khác
  const hash = crypto.pbkdf2Sync(password, salt, 100000, 64, "sha512");
  res.send(hash);
});

// ✅ Good: Non-blocking
app.get("/hash", (req, res) => {
  // Asynchronous - không block
  crypto.pbkdf2(password, salt, 100000, 64, "sha512", (err, hash) => {
    res.send(hash);
  });
});
```

### Ví dụ 4: Đo Event Loop Lag

```javascript
// Đo độ trễ của Event Loop
function measureEventLoopLag() {
  const start = Date.now();

  setImmediate(() => {
    const lag = Date.now() - start;
    console.log(`Event Loop Lag: ${lag}ms`);

    if (lag > 100) {
      console.warn("⚠️ Event Loop bị block!");
    }
  });
}

// Chạy định kỳ
setInterval(measureEventLoopLag, 1000);
```

## 8. Best Practices

### 1. Không block Event Loop

```javascript
// ❌ Bad: Synchronous operations
const data = fs.readFileSync("large-file.txt");
const hash = crypto.pbkdf2Sync(password, salt, 100000, 64, "sha512");

// ✅ Good: Asynchronous operations
const data = await fs.promises.readFile("large-file.txt");
const hash = await util.promisify(crypto.pbkdf2)(password, salt, 100000, 64, "sha512");
```

### 2. Chia nhỏ CPU-intensive tasks

```javascript
// ❌ Bad: Block Event Loop
function processLargeArray(array) {
  for (let i = 0; i < array.length; i++) {
    heavyComputation(array[i]);
  }
}

// ✅ Good: Chia nhỏ với setImmediate
async function processLargeArray(array) {
  const chunkSize = 100;

  for (let i = 0; i < array.length; i += chunkSize) {
    const chunk = array.slice(i, i + chunkSize);

    for (const item of chunk) {
      heavyComputation(item);
    }

    // Cho Event Loop cơ hội xử lý I/O
    await new Promise((resolve) => setImmediate(resolve));
  }
}
```

### 3. Dùng Worker Threads cho CPU-intensive

```javascript
const { Worker, isMainThread, parentPort } = require("worker_threads");

if (isMainThread) {
  // Main thread - không bị block
  const worker = new Worker(__filename);

  worker.on("message", (result) => {
    console.log("Result:", result);
  });

  worker.postMessage({ data: "heavy computation" });
} else {
  // Worker thread - xử lý heavy computation
  parentPort.on("message", (data) => {
    const result = heavyComputation(data);
    parentPort.postMessage(result);
  });
}
```

### 4. Tránh nextTick recursion

```javascript
// ❌ Bad: Starve I/O
function bad() {
  process.nextTick(bad);
}

// ✅ Good: Dùng setImmediate
function good() {
  setImmediate(good);
}
```

## 9. Câu hỏi phỏng vấn thường gặp

### Q1: Event Loop là gì và hoạt động như thế nào?

**Trả lời:**

> Event Loop là cơ chế cho phép Node.js thực hiện non-blocking I/O mặc dù JavaScript single-threaded. Nó liên tục kiểm tra Call Stack và các queues để thực thi code.
>
> Khi gặp async operation, Node.js đẩy xuống libuv xử lý. Code tiếp tục chạy. Khi operation hoàn thành, callback được đưa vào queue. Event Loop kiểm tra Stack trống thì lấy callback từ queue đưa vào thực thi.

### Q2: Sự khác biệt giữa process.nextTick() và setImmediate()?

**Trả lời:**

> `process.nextTick()` chạy ngay sau operation hiện tại, trước khi Event Loop tiếp tục - ưu tiên cao nhất.
>
> `setImmediate()` chạy trong check phase của Event Loop iteration tiếp theo - sau I/O events.
>
> Dùng nextTick khi cần chạy ngay lập tức, dùng setImmediate cho recursive operations để tránh starve I/O.

### Q3: Tại sao Node.js single-threaded nhưng handle được nhiều requests?

**Trả lời:**

> Vì Node.js dùng non-blocking I/O. Khi có request đọc database, Node.js không chờ mà tiếp tục xử lý request khác. Khi database trả về, callback được đưa vào queue và xử lý sau.
>
> Thực tế, libuv có thread pool (4 threads mặc định) để xử lý file system và một số operations khác.

### Q4: Làm sao tránh blocking Event Loop?

**Trả lời:**

> 1. Dùng async methods thay vì sync (readFile thay vì readFileSync)
> 2. Chia nhỏ CPU-intensive tasks với setImmediate
> 3. Dùng Worker Threads cho heavy computation
> 4. Tránh JSON.parse/stringify với data lớn
> 5. Monitor Event Loop lag

## 10. Tổng kết

**Event Loop là gì?** Cơ chế cho phép Node.js xử lý async operations trên single thread.

**Tại sao quan trọng?** Cho phép handle nhiều concurrent connections mà không cần multi-threading.

**Thứ tự ưu tiên:**

1. Synchronous code (Call Stack)
2. process.nextTick() (Microtask)
3. Promise callbacks (Microtask)
4. setTimeout/setInterval (Macrotask - timers)
5. setImmediate (Macrotask - check)
6. I/O callbacks (Macrotask - poll)

**Nhớ:**

- Không block Event Loop
- Dùng async operations
- Chia nhỏ heavy tasks
- Worker Threads cho CPU-intensive
