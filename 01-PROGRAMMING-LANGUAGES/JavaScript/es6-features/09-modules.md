# Modules (import/export)

> Quay lại [Mục lục](./README.md) | Trước: [Classes](./08-classes.md)

## 📌 Tổng quan

ES6 Modules là hệ thống module **chính thức** của JavaScript, cho phép chia code thành các file riêng biệt với **scope riêng**. Trước ES6, phải dùng patterns như IIFE, CommonJS (Node.js), hoặc AMD (RequireJS).

### Module vs Script

| Feature | Script | Module |
|---------|--------|--------|
| **Scope** | Global | File-level (tự động) |
| **Strict mode** | Tùy chọn | Tự động |
| **Top-level `this`** | `window` | `undefined` |
| **`import`/`export`** | ❌ | ✅ |
| **Loading** | Synchronous | Asynchronous |
| **Duplicate execution** | Mỗi lần include | Chỉ 1 lần (cached) |

```html
<!-- Script thường -->
<script src="app.js"></script>

<!-- Module -->
<script type="module" src="app.js"></script>
```

---

## 1. Named Exports

### 1.1. Inline Export

```javascript
// math.js — export ngay khi khai báo
export const PI = 3.14159;
export const E = 2.71828;

export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export class Calculator {
  constructor() {
    this.result = 0;
  }
  add(n) {
    this.result += n;
    return this;
  }
}
```

### 1.2. Bottom Export (export list)

```javascript
// utils.js — khai báo trước, export ở cuối file
const formatDate = (date) => date.toISOString().split("T")[0];
const formatCurrency = (amount) => `$${amount.toFixed(2)}`;

function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

function throttle(fn, limit) {
  let lastCall = 0;
  return (...args) => {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      return fn(...args);
    }
  };
}

// Export ở cuối — dễ quản lý
export { formatDate, formatCurrency, debounce, throttle };
```

### 1.3. Export với Alias

```javascript
// Export với tên khác
export { 
  add as sum, 
  subtract as minus,
  multiply as mul 
};

// Import sẽ dùng tên mới
// import { sum, minus, mul } from './math.js';
```

---

## 2. Named Imports

```javascript
// Import cụ thể
import { add, subtract } from "./math.js";
add(2, 3);      // 5
subtract(5, 2); // 3

// Import với alias (đổi tên)
import { add as sum, subtract as minus } from "./math.js";
sum(2, 3);   // 5
minus(5, 2); // 3

// Import tất cả (namespace import)
import * as MathUtils from "./math.js";
MathUtils.add(2, 3);  // 5
MathUtils.PI;          // 3.14159
MathUtils.Calculator;  // class

// ⚠️ Namespace import tạo READ-ONLY object
// MathUtils.add = null; // ❌ TypeError in strict mode
```

---

## 3. Default Export

Mỗi module chỉ có **MỘT** default export.

### 3.1. Các cách export default

```javascript
// Cách 1: Inline
export default class User {
  constructor(name) {
    this.name = name;
  }
}

// Cách 2: Khai báo trước
class User {
  constructor(name) {
    this.name = name;
  }
}
export default User;

// Cách 3: Anonymous
export default function(x) {
  return x * 2;
}

// Cách 4: Expression
export default {
  theme: "dark",
  language: "vi",
};
```

### 3.2. Import Default

```javascript
// Import default — có thể đặt tên BẤT KỲ
import User from "./user.js";
import MyUser from "./user.js";     // Cùng file, tên khác
import Whatever from "./user.js";   // Vẫn OK

// ⚠️ Không cần {} cho default import
import User from "./user.js";       // ✅ Default
import { User } from "./user.js";   // ❌ Named import (khác!)
```

---

## 4. Mixed Exports (Default + Named)

```javascript
// api.js — vừa default vừa named
export default class API {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }
  async get(path) {
    return fetch(`${this.baseURL}${path}`);
  }
}

export const BASE_URL = "https://api.example.com";
export function createAPI(config) {
  return new API(config.baseURL || BASE_URL);
}
export const VERSION = "1.0.0";
```

```javascript
// Import mixed
import API, { BASE_URL, createAPI, VERSION } from "./api.js";

// Hoặc: default + namespace cho named
import API, * as helpers from "./api.js";
helpers.BASE_URL;      // "https://api.example.com"
helpers.createAPI({});  // API instance
```

---

## 5. Re-exports (Barrel File)

Tạo "barrel file" (`index.js`) để re-export từ nhiều modules.

```javascript
// components/Button.js
export default function Button() { /* ... */ }

// components/Input.js
export default function Input() { /* ... */ }

// components/Modal.js
export default function Modal() { /* ... */ }
export const MODAL_SIZES = { sm: 300, md: 500, lg: 800 };

// ═══ components/index.js — Barrel file ═══

// Re-export default as named
export { default as Button } from "./Button.js";
export { default as Input } from "./Input.js";
export { default as Modal } from "./Modal.js";

// Re-export named
export { MODAL_SIZES } from "./Modal.js";

// Re-export tất cả named exports
export * from "./utils.js";

// Re-export với rename
export { helper as componentHelper } from "./helpers.js";
```

```javascript
// ✅ Clean imports từ barrel file
import { Button, Input, Modal, MODAL_SIZES } from "./components";
// Thay vì:
// import Button from "./components/Button.js";
// import Input from "./components/Input.js";
// import Modal from "./components/Modal.js";
```

---

## 6. Dynamic Imports

### 6.1. Lazy Loading

```javascript
// Dynamic import trả về Promise
async function loadModule() {
  const module = await import("./math.js");
  console.log(module.add(2, 3)); // 5
  console.log(module.default);    // Default export (nếu có)
}

// Conditional import
async function loadFeature(featureName) {
  if (featureName === "chart") {
    const { Chart } = await import("./features/chart.js");
    return new Chart();
  }
  if (featureName === "editor") {
    const { Editor } = await import("./features/editor.js");
    return new Editor();
  }
}

// Event-driven loading
document.getElementById("btn").addEventListener("click", async () => {
  const { heavyFunction } = await import("./heavy-module.js");
  heavyFunction();
});
```

### 6.2. Import với Destructuring

```javascript
// Destructure dynamic import
const { default: API, createAPI, VERSION } = await import("./api.js");

// Pattern matching
const locale = navigator.language;
const messages = await import(`./locales/${locale}.js`);
```

### 6.3. Code Splitting (React)

```javascript
// React.lazy với dynamic import
const Dashboard = React.lazy(() => import("./pages/Dashboard"));
const Settings = React.lazy(() => import("./pages/Settings"));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

---

## 7. Module Patterns

```
┌─────────────────────────────────────────────────────────┐
│                  MODULE PATTERNS                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Single Default Export (Component pattern)          │
│     // Button.js                                       │
│     export default function Button() {}                │
│                                                         │
│  2. Multiple Named Exports (Utility pattern)           │
│     // utils.js                                        │
│     export const util1 = () => {};                     │
│     export const util2 = () => {};                     │
│                                                         │
│  3. Mixed (Default + Named)                            │
│     // api.js                                          │
│     export default class API {}                        │
│     export const config = {};                          │
│                                                         │
│  4. Barrel (Re-export pattern)                         │
│     // index.js                                        │
│     export * from './module1.js';                      │
│     export { default as X } from './x.js';             │
│                                                         │
│  5. Namespace (Group related exports)                  │
│     // constants.js                                    │
│     export const COLORS = { ... };                     │
│     export const SIZES = { ... };                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 8. CommonJS vs ES Modules

```
┌──────────────────────┬─────────────────────┬──────────────────┐
│ Feature              │ CommonJS (CJS)      │ ES Modules (ESM) │
├──────────────────────┼─────────────────────┼──────────────────┤
│ Syntax               │ require/module.exports │ import/export │
│ Loading              │ Synchronous         │ Asynchronous     │
│ Evaluation           │ Runtime             │ Pre-parsed       │
│ Binding              │ Copy of value       │ Live binding     │
│ Tree-shaking         │ ❌ Khó              │ ✅ Tốt           │
│ Top-level await      │ ❌                  │ ✅ (ES2022)      │
│ Environment          │ Node.js             │ Browser + Node   │
│ File extension       │ .js, .cjs           │ .js, .mjs        │
│ Circular dependency  │ Partial exports     │ Live bindings    │
└──────────────────────┴─────────────────────┴──────────────────┘
```

```javascript
// CommonJS
const { add } = require("./math");
module.exports = { add };

// ES Modules
import { add } from "./math.js";
export { add };

// Live binding (ESM)
// counter.js
export let count = 0;
export function increment() { count++; }

// main.js
import { count, increment } from "./counter.js";
console.log(count); // 0
increment();
console.log(count); // 1 (live binding — thấy thay đổi!)
```

---

## 9. Best Practices

```javascript
// ✅ 1 default export per file cho components/classes
// User.js
export default class User { }

// ✅ Named exports cho utilities, constants
// utils.js
export const formatDate = () => {};
export const formatNumber = () => {};

// ✅ Barrel files cho clean imports
// components/index.js
export { default as Button } from "./Button";

// ✅ Consistent naming — file name = default export name
// UserProfile.js → export default UserProfile

// ✅ Import ở đầu file, group theo loại
import React, { useState, useEffect } from "react"; // External
import { Button, Input } from "@/components";         // Internal absolute
import { formatDate } from "./utils";                  // Relative
import styles from "./styles.module.css";              // Styles

// ❌ Tránh wildcard re-export khi barrel file lớn (tree-shaking issues)
// export * from "./everything.js"; // Có thể include code không dùng

// ❌ Tránh circular imports
// a.js imports from b.js, b.js imports from a.js
```

---

> Tiếp theo: [Promises →](./10-promises.md)
