# Module System trong Node.js

## 1. Khái niệm

### Module là gì?

**Module** là một file JavaScript chứa code có thể được tái sử dụng. Mỗi file trong Node.js được coi là một module riêng biệt.

**Tại sao cần Module?**

- **Tổ chức code:** Chia nhỏ code thành các file có chức năng riêng
- **Tái sử dụng:** Import module vào nhiều nơi
- **Encapsulation:** Ẩn implementation details, chỉ export những gì cần thiết
- **Tránh xung đột:** Mỗi module có scope riêng

### Hai hệ thống Module

| CommonJS (CJS)                 | ES Modules (ESM)       |
| ------------------------------ | ---------------------- |
| `require()` / `module.exports` | `import` / `export`    |
| Synchronous loading            | Asynchronous loading   |
| Default trong Node.js          | Cần config hoặc `.mjs` |
| Không hỗ trợ top-level await   | Hỗ trợ top-level await |

## 2. CommonJS (CJS)

### Khái niệm

**CommonJS** là hệ thống module mặc định của Node.js, sử dụng `require()` để import và `module.exports` để export.

### Export

```javascript
// ========== math.js ==========

// Cách 1: Export từng function
module.exports.add = (a, b) => a + b;
module.exports.subtract = (a, b) => a - b;
module.exports.multiply = (a, b) => a * b;

// Cách 2: Dùng exports shorthand
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;

// Cách 3: Export object
module.exports = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  PI: 3.14159,
};

// Cách 4: Export single function/class (default export)
module.exports = function add(a, b) {
  return a + b;
};

// Cách 5: Export class
class Calculator {
  add(a, b) {
    return a + b;
  }
  subtract(a, b) {
    return a - b;
  }
}
module.exports = Calculator;
```

### Import

```javascript
// ========== app.js ==========

// Import toàn bộ module
const math = require("./math");
console.log(math.add(1, 2)); // 3
console.log(math.PI); // 3.14159

// Destructuring import
const { add, subtract } = require("./math");
console.log(add(1, 2)); // 3

// Import default export
const Calculator = require("./Calculator");
const calc = new Calculator();

// Import built-in modules
const fs = require("fs");
const path = require("path");

// Import từ node_modules
const express = require("express");
const lodash = require("lodash");
```

### Lưu ý quan trọng: exports vs module.exports

```javascript
// exports là reference đến module.exports
console.log(exports === module.exports); // true (ban đầu)

// ✅ Đúng: Thêm properties vào exports
exports.add = (a, b) => a + b;

// ❌ Sai: Gán lại exports không hoạt động
exports = { add: (a, b) => a + b }; // Không export được!

// ✅ Đúng: Gán lại module.exports
module.exports = { add: (a, b) => a + b };

// Giải thích:
// - exports chỉ là shortcut đến module.exports
// - Khi gán lại exports, nó không còn trỏ đến module.exports
// - Node.js chỉ export module.exports, không phải exports
```

## 3. ES Modules (ESM)

### Khái niệm

**ES Modules** là hệ thống module chuẩn của JavaScript (ES6+), sử dụng `import`/`export` syntax.

### Cách enable ESM

```json
// package.json - Cách 1: Toàn bộ project
{
  "type": "module"
}
```

```javascript
// Cách 2: Dùng extension .mjs
// math.mjs - file này sẽ dùng ESM
```

### Export

```javascript
// ========== math.mjs ==========

// Named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

export const PI = 3.14159;

// Export sau khi define
const multiply = (a, b) => a * b;
const divide = (a, b) => a / b;
export { multiply, divide };

// Export với alias
export { multiply as mul, divide as div };

// Default export (chỉ 1 per file)
export default class Calculator {
  add(a, b) {
    return a + b;
  }
}

// Re-export từ module khác
export { add, subtract } from "./utils.mjs";
export * from "./helpers.mjs"; // Export tất cả
```

### Import

```javascript
// ========== app.mjs ==========

// Named imports
import { add, subtract } from "./math.mjs";
console.log(add(1, 2)); // 3

// Import với alias
import { add as sum, subtract as sub } from "./math.mjs";
console.log(sum(1, 2)); // 3

// Import tất cả as namespace
import * as math from "./math.mjs";
console.log(math.add(1, 2)); // 3
console.log(math.PI); // 3.14159

// Default import
import Calculator from "./math.mjs";
const calc = new Calculator();

// Mixed import
import Calculator, { add, subtract } from "./math.mjs";

// Dynamic import (lazy loading)
const math = await import("./math.mjs");
console.log(math.add(1, 2));

// Import JSON (cần assert)
import data from "./data.json" assert { type: "json" };
```

### Top-level Await

```javascript
// ✅ ESM hỗ trợ top-level await
const response = await fetch("https://api.example.com/data");
const data = await response.json();

export { data };

// ❌ CommonJS không hỗ trợ
// const data = await fetch(...); // SyntaxError
```

## 4. So sánh CJS vs ESM

| Feature                   | CommonJS                     | ES Modules          |
| ------------------------- | ---------------------------- | ------------------- |
| Syntax                    | `require` / `module.exports` | `import` / `export` |
| Loading                   | Synchronous                  | Asynchronous        |
| Top-level await           | ❌                           | ✅                  |
| Tree shaking              | ❌                           | ✅                  |
| Static analysis           | ❌                           | ✅                  |
| `__dirname`, `__filename` | ✅                           | ❌ (cần workaround) |
| JSON import               | ✅ Direct                    | Cần assert          |
| Default in Node           | ✅                           | Cần config          |
| Browser support           | ❌                           | ✅                  |

### ESM Workarounds

```javascript
// __dirname, __filename trong ESM
import { fileURLToPath } from "url";
import { dirname } from "path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

console.log(__dirname); // /path/to/directory
console.log(__filename); // /path/to/file.mjs

// Import JSON trong ESM
// Cách 1: assert
import data from "./data.json" assert { type: "json" };

// Cách 2: fs
import { readFile } from "fs/promises";
const data = JSON.parse(await readFile(new URL("./data.json", import.meta.url)));
```

## 5. Module Resolution

### Node.js tìm module như thế nào?

```javascript
require("express"); // Tìm ở đâu?
```

**Thứ tự tìm kiếm:**

1. **Core modules:** `fs`, `path`, `http`... (built-in)
2. **File modules:** Bắt đầu với `./`, `../`, `/`
3. **node_modules:** Tìm trong thư mục `node_modules`

```
/project/src/utils/helper.js
require('lodash')

Tìm theo thứ tự:
1. /project/src/utils/node_modules/lodash
2. /project/src/node_modules/lodash
3. /project/node_modules/lodash
4. /node_modules/lodash
5. Global node_modules
```

### File Extension Resolution

```javascript
require("./math");

// Node.js tìm theo thứ tự:
// 1. ./math.js
// 2. ./math.json
// 3. ./math.node
// 4. ./math/index.js
// 5. ./math/index.json
// 6. ./math/index.node
// 7. ./math/package.json → "main" field
```

## 6. Module Caching

### Khái niệm

**Module Caching:** Node.js cache modules sau lần require đầu tiên. Các lần require sau trả về cùng instance.

```javascript
// counter.js
let count = 0;
module.exports = {
  increment: () => ++count,
  getCount: () => count,
};

// app.js
const counter1 = require("./counter");
const counter2 = require("./counter");

counter1.increment(); // 1
counter2.increment(); // 2 (cùng instance!)

console.log(counter1 === counter2); // true
console.log(counter1.getCount()); // 2
console.log(counter2.getCount()); // 2
```

### Tại sao cần Caching?

- **Performance:** Không cần load lại module
- **Singleton pattern:** Đảm bảo chỉ có 1 instance
- **Shared state:** Các file khác nhau dùng chung state

### Clear Cache (khi cần)

```javascript
// Clear cache của 1 module
delete require.cache[require.resolve("./counter")];

// Clear tất cả cache
Object.keys(require.cache).forEach((key) => {
  delete require.cache[key];
});
```

## 7. Circular Dependencies

### Khái niệm

**Circular Dependency** xảy ra khi module A require module B, và module B require module A.

```javascript
// a.js
console.log("a.js starting");
exports.done = false;
const b = require("./b");
console.log("in a.js, b.done =", b.done);
exports.done = true;
console.log("a.js done");

// b.js
console.log("b.js starting");
exports.done = false;
const a = require("./a");
console.log("in b.js, a.done =", a.done);
exports.done = true;
console.log("b.js done");

// main.js
const a = require("./a");
const b = require("./b");

// Output:
// a.js starting
// b.js starting
// in b.js, a.done = false  ← a chưa load xong!
// b.js done
// in a.js, b.done = true
// a.js done
```

### Cách giải quyết

```javascript
// 1. Restructure code - Tách common logic ra module riêng
// common.js
module.exports = { sharedFunction };

// a.js
const common = require("./common");

// b.js
const common = require("./common");

// 2. Lazy require - Require bên trong function
// a.js
module.exports = {
  doSomething() {
    const b = require("./b"); // Lazy require
    return b.getValue();
  },
};

// 3. Dependency Injection
// a.js
class A {
  setB(b) {
    this.b = b;
  }
}
```

## 8. Built-in Modules quan trọng

```javascript
// File System
const fs = require("fs");
const fsPromises = require("fs/promises");

// Path
const path = require("path");
path.join(__dirname, "file.txt");
path.resolve("src", "utils");
path.basename("/foo/bar/file.txt"); // 'file.txt'
path.extname("file.txt"); // '.txt'
path.dirname("/foo/bar/file.txt"); // '/foo/bar'

// OS
const os = require("os");
os.cpus(); // CPU info
os.totalmem(); // Total memory
os.freemem(); // Free memory
os.platform(); // 'darwin', 'win32', 'linux'
os.homedir(); // Home directory

// Crypto
const crypto = require("crypto");
crypto.randomBytes(16).toString("hex");
crypto.createHash("sha256").update("data").digest("hex");

// HTTP
const http = require("http");
const https = require("https");

// URL
const { URL, URLSearchParams } = require("url");

// Events
const EventEmitter = require("events");

// Stream
const { Readable, Writable, Transform } = require("stream");

// Child Process
const { exec, spawn, fork } = require("child_process");

// Worker Threads
const { Worker, isMainThread } = require("worker_threads");

// Cluster
const cluster = require("cluster");

// Util
const util = require("util");
util.promisify(fs.readFile);
util.inspect(obj, { depth: null });
```

## 9. Best Practices

### 1. Prefer ESM cho projects mới

```javascript
// package.json
{
  "type": "module"
}
```

### 2. Named exports over default exports

```javascript
// ✅ Good: Named exports - IDE support tốt hơn
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// ❌ Avoid: Default export
export default { add, subtract };
```

### 3. Consistent imports

```javascript
// ✅ Good: Tất cả imports ở đầu file
import fs from "fs";
import path from "path";
import { add } from "./math.js";

// Code...

// ❌ Bad: Import rải rác
import fs from "fs";
// Code...
import path from "path"; // Import ở giữa file
```

### 4. Destructure khi chỉ cần một số functions

```javascript
// ✅ Good: Chỉ import những gì cần
import { readFile, writeFile } from "fs/promises";

// ❌ Avoid: Import toàn bộ
import fs from "fs/promises";
fs.readFile();
```

### 5. Dùng path.join() cho paths

```javascript
// ✅ Good: Cross-platform
const filePath = path.join(__dirname, "data", "file.txt");

// ❌ Bad: Hardcoded separator
const filePath = __dirname + "/data/file.txt";
```

## 10. Tổng kết

**CommonJS:** Default trong Node.js, `require`/`module.exports`, synchronous.

**ES Modules:** Modern standard, `import`/`export`, async, tree shaking.

**Module Resolution:** Core → File → node_modules.

**Caching:** Modules được cache, cùng instance khi require nhiều lần.

**Circular Dependencies:** Tránh bằng cách restructure hoặc lazy require.

**Nhớ:**

- ESM cho projects mới
- Named exports preferred
- Imports ở đầu file
- path.join() cho paths
- Tránh circular dependencies
