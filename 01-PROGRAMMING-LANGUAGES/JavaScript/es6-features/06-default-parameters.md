# Default Parameters

> Quay lại [Mục lục](./README.md) | Trước: [Spread & Rest](./05-spread-rest.md)

## 📌 Tổng quan

ES6 cho phép gán **giá trị mặc định** cho function parameters. Khi argument không được truyền (hoặc là `undefined`), giá trị mặc định sẽ được sử dụng. Trước ES6, pattern này phải dùng `||` hoặc `typeof` checks.

---

## 1. So sánh ES5 vs ES6

```javascript
// ES5 — workarounds
function greet(name) {
  name = name || "Guest";         // ⚠️ Bug: "" cũng bị thay thế
  return "Hello " + name;
}

function greetSafe(name) {
  name = typeof name !== "undefined" ? name : "Guest"; // ✅ An toàn hơn
  return "Hello " + name;
}

// ES6 — Clean và chính xác
function greet(name = "Guest") {
  return `Hello ${name}`;
}

greet();         // "Hello Guest"
greet("John");   // "Hello John"
greet(undefined);// "Hello Guest"
```

---

## 2. Các loại Default Values

### 2.1. Primitive Values

```javascript
function example(
  str = "default",
  num = 42,
  bool = true,
  nul = null,     // null là giá trị hợp lệ
  arr = [],
  obj = {}
) {
  console.log(str, num, bool, nul, arr, obj);
}
```

### 2.2. Expressions (biểu thức)

```javascript
// Tham chiếu parameter trước đó
function calculate(a, b = a * 2) {
  return a + b;
}
calculate(5);    // 15 (5 + 10)
calculate(5, 3); // 8

// Biểu thức phức tạp
function createId(prefix = "id", counter = Date.now()) {
  return `${prefix}_${counter}`;
}

// Spread
function merge(defaults = {}, overrides = {}) {
  return { ...defaults, ...overrides };
}
```

### 2.3. Function Calls (Lazy Evaluation)

Default values chỉ được **evaluate khi cần** (lazy), không phải khi function được define.

```javascript
function getDefault() {
  console.log("Computing default...");
  return 42;
}

function process(value = getDefault()) {
  return value;
}

process(100); // 100 — getDefault() KHÔNG được gọi
process();    // "Computing default..." → 42 — getDefault() ĐƯỢC gọi

// ✅ Mỗi lần gọi sẽ evaluate lại
function createArray(item, list = []) {
  list.push(item);
  return list;
}
createArray(1); // [1]
createArray(2); // [2] — list mới, KHÔNG phải [1, 2]
// Khác với Python nơi mutable default bị chia sẻ!
```

### 2.4. Default với Destructuring

```javascript
// Object destructuring với defaults cho properties
function createUser({ name = "Anonymous", age = 0, role = "user" } = {}) {
  return { name, age, role };
}

createUser();                    // { name: "Anonymous", age: 0, role: "user" }
createUser({});                  // { name: "Anonymous", age: 0, role: "user" }
createUser({ name: "John" });   // { name: "John", age: 0, role: "user" }

// Array destructuring với defaults
function getFirstTwo([first = 0, second = 0] = []) {
  return [first, second];
}

getFirstTwo();          // [0, 0]
getFirstTwo([1]);       // [1, 0]
getFirstTwo([1, 2]);    // [1, 2]
```

---

## 3. `undefined` vs `null` vs falsy values

**Quy tắc**: Default values chỉ được sử dụng khi argument là `undefined`.

```javascript
function test(value = "default") {
  return value;
}

// ✅ Trigger default (undefined)
test();            // "default"
test(undefined);   // "default"

// ❌ KHÔNG trigger default (các giá trị khác)
test(null);        // null
test(0);           // 0
test("");          // ""
test(false);       // false
test(NaN);         // NaN

// ⚠️ Đây là khác biệt lớn so với pattern || của ES5
function es5(value) {
  value = value || "default";
  return value;
}
es5(0);     // "default" ← Bug! 0 là falsy
es5("");    // "default" ← Bug! "" là falsy
es5(false); // "default" ← Bug!
```

---

## 4. Patterns nâng cao

### 4.1. Required Parameters Pattern

```javascript
function required(paramName) {
  throw new Error(`Parameter "${paramName}" is required`);
}

function createUser(
  name = required("name"),
  email = required("email")
) {
  return { name, email };
}

createUser("John", "john@example.com"); // ✅ OK
createUser("John");    // ❌ Error: Parameter "email" is required
createUser();          // ❌ Error: Parameter "name" is required
```

### 4.2. Parameter Validation

```javascript
function validateAge(age) {
  if (typeof age !== "number" || age < 0 || age > 150) {
    throw new RangeError("Age must be between 0 and 150");
  }
  return age;
}

function createPerson(name, age = validateAge(0)) {
  return { name, age };
}
```

### 4.3. Previous Parameter Reference

```javascript
// Tham chiếu parameter TRƯỚC (left-to-right)
function greet(name, greeting = `Hello, ${name}!`) {
  return greeting;
}
greet("John"); // "Hello, John!"

// ❌ KHÔNG thể tham chiếu parameter SAU
function bad(a = b, b = 1) {
  // ReferenceError: Cannot access 'b' before initialization
}
```

### 4.4. Kết hợp với Nullish Coalescing (ES2020)

```javascript
// Khi cần xử lý cả null lẫn undefined
function process(config = {}) {
  const timeout = config.timeout ?? 3000;    // null hoặc undefined → 3000
  const retries = config.retries ?? 3;
  const debug = config.debug ?? false;
  
  return { timeout, retries, debug };
}

process({ timeout: 0, retries: null });
// { timeout: 0, retries: 3, debug: false }
// timeout = 0 (giữ nguyên), retries = 3 (null → default)
```

---

## 5. `arguments` object và Default Parameters

```javascript
// ⚠️ arguments KHÔNG phản ánh default values
function example(a = 1, b = 2) {
  console.log(arguments.length); // Số arguments THỰC SỰ được truyền
  console.log(arguments[0]);     // Giá trị THỰC SỰ được truyền
  console.log(a, b);             // Giá trị sau khi apply defaults
}

example();       // arguments.length = 0, arguments[0] = undefined, a = 1, b = 2
example(10);     // arguments.length = 1, arguments[0] = 10, a = 10, b = 2
example(10, 20); // arguments.length = 2, arguments[0] = 10, a = 10, b = 20

// ✅ Nên dùng rest parameters thay vì arguments
function betterExample(a = 1, b = 2, ...rest) {
  console.log(a, b, rest);
}
```

---

## 6. Best Practices

```javascript
// ✅ Đặt parameters có default ở CUỐI
function good(required1, required2, optional = "default") {}

// ❌ Tránh đặt ở giữa (phải truyền undefined để skip)
function bad(optional = "default", required) {}
bad(undefined, "value"); // Awkward

// ✅ Dùng object cho nhiều optional parameters
function createWidget({
  width = 100,
  height = 100,
  color = "blue",
  border = true,
  shadow = false,
} = {}) {
  // ...
}

// ✅ Giữ default values đơn giản
function good(timeout = 3000) {}

// ❌ Tránh side effects trong defaults
function bad(timestamp = console.log("Computing...") || Date.now()) {}
```

---

> Tiếp theo: [Enhanced Object Literals →](./07-enhanced-object-literals.md)
