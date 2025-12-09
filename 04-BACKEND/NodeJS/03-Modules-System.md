# Module System trong Node.js

## 1. CommonJS (CJS)

### Export

```javascript
// math.js

// Named exports
module.exports.add = (a, b) => a + b;
module.exports.subtract = (a, b) => a - b;

// Hoặc
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;

// Hoặc export object
module.exports = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};

// Default export
module.exports = function add(a, b) {
  return a + b;
};
```

### Import

```javascript
// Import all
const math = require("./math");
math.add(1, 2);

// Destructuring
const { add, subtract } = require("./math");
add(1, 2);

// Import default
const add = require("./math");
add(1, 2);
```

### Lưu ý với exports

```javascript
// ❌ Sai - gán lại exports không hoạt động
exports = { add: () => {} };

// ✅ Đúng - dùng module.exports
module.exports = { add: () => {} };

// exports chỉ là reference đến module.exports
// exports === module.exports // true (ban đầu)
```

## 2. ES Modules (ESM)

### Enable ESM

```json
// package.json
{
  "type": "module"
}
```

Hoặc dùng extension `.mjs`

### Export

```javascript
// math.mjs

// Named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Export sau khi define
const multiply = (a, b) => a * b;
export { multiply };

// Default export
export default function divide(a, b) {
  return a / b;
}

// Re-export
export { add, subtract } from "./utils.mjs";
export * from "./helpers.mjs";
```

### Import

```javascript
// Named imports
import { add, subtract } from "./math.mjs";

// Import all as namespace
import * as math from "./math.mjs";
math.add(1, 2);

// Default import
import divide from "./math.mjs";

// Mixed
import divide, { add, subtract } from "./math.mjs";

// Rename
import { add as sum } from "./math.mjs";

// Dynamic import
const math = await import("./math.mjs");
```

## 3. So sánh CJS vs ESM

| Feature                   | CommonJS                       | ES Modules          |
| ------------------------- | ------------------------------ | ------------------- |
| Syntax                    | `require()` / `module.exports` | `import` / `export` |
| Loading                   | Synchronous                    | Asynchronous        |
| Top-level await           | ❌                             | ✅                  |
| Tree shaking              | ❌                             | ✅                  |
| Static analysis           | ❌                             | ✅                  |
| `__dirname`, `__filename` | ✅                             | ❌ (cần workaround) |
| JSON import               | ✅ Direct                      | Cần assert          |
| Default in Node           | ✅                             | Cần config          |

### ESM workarounds

```javascript
// __dirname, __filename trong ESM
import { fileURLToPath } from "url";
import { dirname } from "path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// Import JSON trong ESM
import data from "./data.json" assert { type: "json" };

// Hoặc
import { readFile } from "fs/promises";
const data = JSON.parse(await readFile("./data.json", "utf8"));
```

## 4. Module Resolution

### Node.js tìm module như thế nào?

```javascript
require("express");
```

1. **Core modules**: `fs`, `path`, `http`... (built-in)
2. **File modules**: Bắt đầu với `./`, `../`, `/`
3. **node_modules**: Tìm trong thư mục `node_modules`

```
/project/src/utils/helper.js
require('lodash')

Tìm theo thứ tự:
1. /project/src/utils/node_modules/lodash
2. /project/src/node_modules/lodash
3. /project/node_modules/lodash
4. /node_modules/lodash
```

### File extensions resolution

```javascript
require("./math");

// Node tìm theo thứ tự:
// 1. ./math.js
// 2. ./math.json
// 3. ./math.node
// 4. ./math/index.js
// 5. ./math/index.json
// 6. ./math/index.node
```

## 5. Module Caching

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
```

### Clear cache

```javascript
// Clear specific module
delete require.cache[require.resolve("./counter")];

// Clear all cache
Object.keys(require.cache).forEach((key) => {
  delete require.cache[key];
});
```

## 6. Circular Dependencies

```javascript
// a.js
console.log("a starting");
exports.done = false;
const b = require("./b");
console.log("in a, b.done =", b.done);
exports.done = true;
console.log("a done");

// b.js
console.log("b starting");
exports.done = false;
const a = require("./a");
console.log("in b, a.done =", a.done);
exports.done = true;
console.log("b done");

// main.js
const a = require("./a");
const b = require("./b");

// Output:
// a starting
// b starting
// in b, a.done = false  ← a chưa load xong
// b done
// in a, b.done = true
// a done
```

### Giải quyết circular dependencies

```javascript
// 1. Restructure code - tách common logic ra module riêng
// 2. Lazy require - require bên trong function
// 3. Dependency injection
```

## 7. Built-in Modules quan trọng

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

// OS
const os = require("os");
os.cpus();
os.totalmem();
os.freemem();
os.platform();

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
const { exec, spawn } = require("child_process");

// Worker Threads
const { Worker, isMainThread } = require("worker_threads");

// Cluster
const cluster = require("cluster");

// Util
const util = require("util");
util.promisify(fs.readFile);
util.inspect(obj, { depth: null });
```

## 8. Best Practices

1. **Prefer ESM** cho projects mới
2. **Consistent imports** - đặt tất cả imports ở đầu file
3. **Named exports** over default exports (better tree shaking, IDE support)
4. **Avoid circular dependencies**
5. **Use path.join()** thay vì string concatenation
6. **Destructure** khi chỉ cần một số functions

```javascript
// ✅ Good
import { readFile, writeFile } from "fs/promises";

// ❌ Avoid
import fs from "fs/promises";
fs.readFile();
```
