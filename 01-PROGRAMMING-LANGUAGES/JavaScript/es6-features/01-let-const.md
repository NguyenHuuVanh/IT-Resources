# let và const - Khai báo biến trong ES6

> Quay lại [Mục lục](./README.md)

## 📌 Tổng quan

Trước ES6, JavaScript chỉ có `var` để khai báo biến. ES6 giới thiệu `let` và `const` - hai cách khai báo biến mới giúp code an toàn và dễ quản lý hơn nhờ **block scope** và các ràng buộc chặt chẽ hơn.

---

## 1. Vấn đề với `var`

### 1.1. Function Scope (không có Block Scope)

```javascript
// var chỉ có function scope, không có block scope
function varProblem() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 ✅ — x "tràn" ra ngoài block if
}

// Vấn đề phổ biến trong loop
for (var i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 5, 5, 5, 5, 5 (không phải 0, 1, 2, 3, 4)
// Vì var i là function-scoped, khi setTimeout chạy thì loop đã kết thúc và i = 5
```

### 1.2. Hoisting gây nhầm lẫn

```javascript
// var được hoisted lên đầu function với giá trị undefined
console.log(x); // undefined (không báo lỗi!)
var x = 5;

// JavaScript hiểu như sau:
// var x;           // Declaration được hoisted
// console.log(x);  // undefined
// x = 5;           // Assignment giữ nguyên vị trí
```

### 1.3. Re-declaration (khai báo lại)

```javascript
// var cho phép khai báo lại — dễ gây bug
var name = "John";
var name = "Jane"; // ✅ Không báo lỗi, ghi đè âm thầm!
console.log(name); // "Jane"

// Trong dự án lớn, điều này rất nguy hiểm vì có thể
// vô tình ghi đè biến quan trọng mà không biết
```

### 1.4. Global Object Pollution

```javascript
// var khai báo ở top level tạo property trên window/global
var globalVar = "I'm global";
console.log(window.globalVar); // "I'm global"

// Điều này làm ô nhiễm global scope
// và có thể xung đột với thư viện khác
```

---

## 2. `let` - Block-scoped Variable

### 2.1. Block Scope

```javascript
// let có block scope - chỉ tồn tại trong block {}
function letExample() {
  if (true) {
    let y = 10;
    console.log(y); // 10 ✅
  }
  console.log(y); // ❌ ReferenceError: y is not defined
}

// Giải quyết vấn đề loop
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0, 1, 2, 3, 4 ✅
// Mỗi iteration tạo một scope mới với giá trị i riêng
```

### 2.2. Không cho phép Re-declaration trong cùng scope

```javascript
let name = "John";
let name = "Jane"; // ❌ SyntaxError: Identifier 'name' has already been declared

// Nhưng có thể khai báo lại ở scope khác
let x = 10;
if (true) {
  let x = 20; // ✅ OK - scope khác
  console.log(x); // 20
}
console.log(x); // 10 (x bên ngoài không bị ảnh hưởng)
```

### 2.3. Cho phép Re-assignment

```javascript
let count = 0;
count = 1;    // ✅ OK
count++;      // ✅ OK
count += 10;  // ✅ OK

// let phù hợp khi giá trị biến cần thay đổi
let user = null;
user = await fetchUser();    // Re-assign sau khi fetch

let total = 0;
for (const item of items) {
  total += item.price;       // Accumulator
}
```

### 2.4. Không tạo property trên Global Object

```javascript
let globalLet = "I'm a let";
console.log(window.globalLet); // undefined
// let không làm ô nhiễm global scope
```

---

## 3. `const` - Block-scoped Constant

### 3.1. Phải khởi tạo giá trị khi khai báo

```javascript
const PI = 3.14159;    // ✅ OK
const MAX_SIZE;        // ❌ SyntaxError: Missing initializer in const declaration
```

### 3.2. Không cho phép Re-assignment

```javascript
const API_URL = "https://api.example.com";
API_URL = "https://other-api.com"; // ❌ TypeError: Assignment to constant variable

const MAX_RETRIES = 3;
MAX_RETRIES++;  // ❌ TypeError
```

### 3.3. `const` với Objects và Arrays (Reference vs Value)

⚠️ **Quan trọng**: `const` bảo vệ **reference** (tham chiếu), KHÔNG bảo vệ **value** (giá trị bên trong).

```javascript
// Object - có thể thay đổi properties
const person = { name: "John", age: 25 };
person.name = "Jane";     // ✅ OK - modify property
person.email = "j@x.com"; // ✅ OK - add property
delete person.age;         // ✅ OK - delete property
person = { name: "Bob" }; // ❌ TypeError - re-assign reference

// Array - có thể thay đổi elements
const numbers = [1, 2, 3];
numbers.push(4);      // ✅ OK - modify array
numbers[0] = 10;      // ✅ OK - modify element
numbers.pop();         // ✅ OK - remove element
numbers.sort();        // ✅ OK - sort in place
numbers = [5, 6, 7];  // ❌ TypeError - re-assign reference
```

### 3.4. Làm Object/Array thực sự bất biến

```javascript
// Object.freeze() - Freeze 1 cấp (shallow freeze)
const frozen = Object.freeze({ name: "John", age: 25 });
frozen.name = "Jane";   // ❌ Silently fails (strict mode: TypeError)
frozen.email = "j@x.com"; // ❌ Silently fails
delete frozen.age;       // ❌ Silently fails

// ⚠️ Object.freeze() chỉ freeze shallow (cấp 1)
const obj = Object.freeze({
  name: "John",
  address: { city: "NYC" }
});
obj.address.city = "LA"; // ✅ Vẫn thay đổi được! (nested object)

// Deep freeze function
function deepFreeze(obj) {
  Object.freeze(obj);
  Object.getOwnPropertyNames(obj).forEach(prop => {
    if (
      obj[prop] !== null &&
      typeof obj[prop] === "object" &&
      !Object.isFrozen(obj[prop])
    ) {
      deepFreeze(obj[prop]);
    }
  });
  return obj;
}

const deepFrozen = deepFreeze({
  name: "John",
  address: { city: "NYC" }
});
deepFrozen.address.city = "LA"; // ❌ Không thay đổi được

// Object.seal() - Không thêm/xóa property, nhưng có thể modify
const sealed = Object.seal({ name: "John", age: 25 });
sealed.name = "Jane";    // ✅ OK - modify existing
sealed.email = "j@x.com"; // ❌ Silently fails - cannot add
delete sealed.age;        // ❌ Silently fails - cannot delete
```

---

## 4. Temporal Dead Zone (TDZ)

### 4.1. TDZ là gì?

TDZ (Vùng chết tạm thời) là khoảng thời gian từ khi block bắt đầu cho đến khi biến được khai báo. Trong TDZ, biến tồn tại nhưng **không thể truy cập**.

```javascript
// var - Hoisted với giá trị undefined (KHÔNG có TDZ)
console.log(a); // undefined (không lỗi)
var a = 5;
console.log(a); // 5

// let - Hoisted nhưng NẰM TRONG TDZ
console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 5;
console.log(b); // 5

// const - Tương tự let
console.log(c); // ❌ ReferenceError
const c = 10;
```

### 4.2. TDZ Visualization

```
┌───────────────────────────────────────┐
│ {                                     │
│   // ← TDZ cho x bắt đầu ở đây      │
│   console.log(x); // ❌ ReferenceError │
│   console.log(x); // ❌ ReferenceError │
│                                       │
│   let x = 5; // ← TDZ cho x kết thúc │
│                                       │
│   console.log(x); // ✅ 5             │
│ }                                     │
└───────────────────────────────────────┘
```

### 4.3. TDZ trong thực tế

```javascript
// TDZ với default parameters
function example(a = b, b = 1) {
  // ❌ ReferenceError: Cannot access 'b' before initialization
  // Vì khi evaluate a = b, b vẫn trong TDZ
}

function correct(a = 1, b = a) {
  // ✅ OK - a đã được khởi tạo trước b
  console.log(a, b); // 1, 1
}

// TDZ với typeof
typeof undeclared; // "undefined" (biến chưa khai báo - OK)
typeof declared;   // ❌ ReferenceError (biến trong TDZ)
let declared = 10;

// TDZ trong class
class MyClass {
  // ❌ Không thể dùng this trước super() trong constructor
  constructor() {
    // TDZ cho this khi extends
  }
}
```

---

## 5. So sánh chi tiết var vs let vs const

### 5.1. Bảng so sánh

| Feature              | `var`           | `let`     | `const`   |
| -------------------- | --------------- | --------- | --------- |
| **Scope**            | Function        | Block     | Block     |
| **Hoisting**         | ✅ (undefined)  | ✅ (TDZ)  | ✅ (TDZ)  |
| **Re-declare**       | ✅              | ❌        | ❌        |
| **Re-assign**        | ✅              | ✅        | ❌        |
| **Global object**    | ✅ (window)     | ❌        | ❌        |
| **Phải khởi tạo**   | ❌              | ❌        | ✅        |
| **TDZ**              | ❌              | ✅        | ✅        |

### 5.2. Scope trong các tình huống

```javascript
// === SWITCH statement ===
// Cần chú ý: tất cả case chia sẻ cùng scope
switch (action) {
  case "increment": {
    // ✅ Dùng block {} để tạo scope riêng
    let count = 1;
    break;
  }
  case "decrement": {
    let count = -1; // ✅ OK - scope khác
    break;
  }
}

// === TRY-CATCH ===
try {
  let result = riskyOperation();
  console.log(result);
} catch (error) {
  // result không accessible ở đây
  console.error(error);
}
// result và error đều không accessible ở đây

// === FOR loop variants ===
// for...in
for (let key in object) {
  // key chỉ tồn tại trong loop body
}

// for...of
for (const item of array) {
  // const OK vì mỗi iteration tạo binding mới
}
```

---

## 6. Best Practices

### 6.1. Quy tắc chung

```javascript
// ✅ RULE 1: Mặc định dùng const
const API_URL = "https://api.example.com";
const config = { debug: true, version: "1.0" };
const users = [];

// ✅ RULE 2: Dùng let khi cần re-assign
let count = 0;
count++;

let user = null;
user = await fetchUser();

let total = 0;
for (const item of items) {
  total += item.price;
}

// ❌ RULE 3: KHÔNG dùng var
// var oldWay = "avoid this";
```

### 6.2. Naming Conventions

```javascript
// UPPER_SNAKE_CASE cho constant values (không thay đổi)
const MAX_RETRIES = 3;
const API_BASE_URL = "https://api.example.com";
const DEFAULT_TIMEOUT_MS = 5000;
const COLORS = Object.freeze({
  PRIMARY: "#007bff",
  SECONDARY: "#6c757d",
});

// camelCase cho const references (object/array có thể thay đổi nội dung)
const userList = [];
const configOptions = { theme: "dark" };

// camelCase cho let
let currentPage = 1;
let isLoading = false;
```

### 6.3. Khi nào dùng `let`?

```javascript
// ✅ Accumulators
let sum = 0;
numbers.forEach(n => { sum += n; });

// ✅ Conditional initialization
let connection;
if (useSSL) {
  connection = createSSLConnection();
} else {
  connection = createConnection();
}

// ✅ Loop counters (traditional for loop)
for (let i = 0; i < 10; i++) {
  // ...
}

// ✅ State that changes over time
let retryCount = 0;
while (retryCount < MAX_RETRIES) {
  try {
    await fetchData();
    break;
  } catch (e) {
    retryCount++;
  }
}

// ✅ Swap variables
let a = 1, b = 2;
[a, b] = [b, a]; // Sử dụng destructuring
```

### 6.4. Anti-patterns cần tránh

```javascript
// ❌ Dùng var
var x = 10;

// ❌ Dùng let khi không cần re-assign
let name = "John"; // Nên dùng const

// ❌ Khai báo biến cách xa nơi sử dụng
let result;
// ... 50 dòng code khác ...
result = compute(); // Khó theo dõi

// ✅ Khai báo gần nơi sử dụng
// ... code khác ...
const result = compute();

// ❌ Nhiều let trên cùng dòng
let x = 1, y = 2, z = 3;

// ✅ Mỗi biến một dòng (dễ đọc, dễ debug)
let x = 1;
let y = 2;
let z = 3;
```

---

## 7. Câu hỏi phỏng vấn thường gặp

### Q1: Sự khác biệt giữa var, let, const?
**A**: `var` là function-scoped, `let`/`const` là block-scoped. `var` hoisted với `undefined`, `let`/`const` hoisted nhưng nằm trong TDZ. `const` không cho phép re-assignment.

### Q2: TDZ là gì?
**A**: Temporal Dead Zone là khoảng thời gian từ đầu block đến khi biến let/const được khai báo. Truy cập biến trong TDZ sẽ throw ReferenceError.

### Q3: const có thực sự immutable không?
**A**: Không. `const` chỉ ngăn re-assignment reference, không ngăn mutation (thay đổi nội dung). Object/Array được khai báo với const vẫn có thể thay đổi properties/elements. Dùng `Object.freeze()` để tạo immutability thực sự.

### Q4: Output là gì?

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Output: 3, 3, 3

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
// Output: 0, 1, 2
```

**A**: `var i` là function-scoped nên chỉ có 1 biến `i` chung, khi timeout chạy thì `i = 3`. `let j` là block-scoped, mỗi iteration tạo một `j` mới.

---

> Tiếp theo: [Arrow Functions →](./02-arrow-functions.md)
