# for...of Loop

> Quay lại [Mục lục](./README.md) | Trước: [Map & Set](./13-map-set.md)

## 📌 Tổng quan

`for...of` là vòng lặp mới trong ES6 dùng để iterate qua **iterable objects** (objects implement `[Symbol.iterator]`). Nó iterate qua **values** (khác `for...in` iterate qua keys).

---

## 1. Cú pháp cơ bản

```javascript
const arr = ["a", "b", "c"];

// for...of — iterate VALUES
for (const value of arr) {
  console.log(value); // "a", "b", "c"
}

// for...in — iterate KEYS (indices cho array)
for (const index in arr) {
  console.log(index); // "0", "1", "2" (strings!)
}

// traditional for
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]); // "a", "b", "c"
}
```

---

## 2. So sánh `for...of` vs `for...in` vs `forEach`

```
┌───────────────────┬──────────────────┬──────────────────┬─────────────────┐
│ Feature           │ for...of         │ for...in         │ forEach         │
├───────────────────┼──────────────────┼──────────────────┼─────────────────┤
│ Iterates          │ VALUES           │ KEYS (props)     │ VALUES          │
│ Works with        │ Iterables        │ Objects          │ Arrays only     │
│ break/continue    │ ✅               │ ✅               │ ❌              │
│ return            │ ✅ (trong fn)    │ ✅ (trong fn)    │ ❌ (chỉ skip)   │
│ await             │ ✅ (for await)   │ ❌               │ ❌              │
│ Index access      │ ❌ (cần entries) │ ✅ (key = index) │ ✅ (2nd arg)    │
│ Inherited props   │ ❌               │ ✅               │ ❌              │
│ Custom iterables  │ ✅               │ ❌               │ ❌              │
│ Prototype chain   │ Không traverse   │ Traverse         │ N/A             │
└───────────────────┴──────────────────┴──────────────────┴─────────────────┘
```

```javascript
const arr = ["a", "b", "c"];

// for...of — clean, can break
for (const value of arr) {
  if (value === "b") break; // ✅ Can break
  console.log(value); // "a"
}

// forEach — cannot break
arr.forEach(value => {
  if (value === "b") return; // ❌ Only skips current iteration (like continue)
  console.log(value); // "a", "c"
});

// for...in — ⚠️ Problematic with arrays
Array.prototype.customMethod = function() {};

for (const key in arr) {
  console.log(key); // "0", "1", "2", "customMethod" ← includes inherited!
}

for (const value of arr) {
  console.log(value); // "a", "b", "c" ← chỉ values ✅
}
```

---

## 3. Iterate với các kiểu dữ liệu

### 3.1. Arrays

```javascript
for (const item of [10, 20, 30]) {
  console.log(item); // 10, 20, 30
}

// Với destructuring
for (const [index, value] of [10, 20, 30].entries()) {
  console.log(`${index}: ${value}`);
  // "0: 10", "1: 20", "2: 30"
}
```

### 3.2. Strings

```javascript
for (const char of "Hello 🌍") {
  console.log(char);
}
// "H", "e", "l", "l", "o", " ", "🌍"
// ✅ Unicode-aware! (emoji là 1 character, không phải 2)

// So sánh với for loop
for (let i = 0; i < "Hello 🌍".length; i++) {
  console.log("Hello 🌍"[i]);
}
// "H", "e", "l", "l", "o", " ", "�", "�" ← emoji bị split!
```

### 3.3. Maps

```javascript
const map = new Map([
  ["name", "John"],
  ["age", 25],
  ["city", "NYC"],
]);

// Default: iterate entries [key, value]
for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// Chỉ keys
for (const key of map.keys()) {
  console.log(key);
}

// Chỉ values
for (const value of map.values()) {
  console.log(value);
}
```

### 3.4. Sets

```javascript
const set = new Set([1, 2, 3, 4, 5]);
for (const value of set) {
  console.log(value); // 1, 2, 3, 4, 5
}
```

### 3.5. NodeList (DOM)

```javascript
const elements = document.querySelectorAll("div.card");

for (const element of elements) {
  element.classList.add("active");
}
```

### 3.6. Arguments Object

```javascript
function printArgs() {
  for (const arg of arguments) {
    console.log(arg);
  }
}
printArgs("a", "b", "c"); // "a", "b", "c"
```

### 3.7. Generators

```javascript
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

for (const num of fibonacci()) {
  if (num > 100) break;
  console.log(num);
}
// 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89
```

---

## 4. Với Destructuring

```javascript
// Array of arrays
const pairs = [[1, "one"], [2, "two"], [3, "three"]];
for (const [num, word] of pairs) {
  console.log(`${num} = ${word}`);
}

// Array of objects
const users = [
  { name: "John", age: 25, role: "admin" },
  { name: "Jane", age: 30, role: "user" },
];
for (const { name, age } of users) {
  console.log(`${name} is ${age}`);
}

// Nested destructuring
const data = [
  { user: { name: "John" }, scores: [90, 85, 92] },
  { user: { name: "Jane" }, scores: [88, 95, 91] },
];
for (const { user: { name }, scores: [first] } of data) {
  console.log(`${name}'s first score: ${first}`);
}

// Object.entries() — iterate object
const config = { host: "localhost", port: 3000, debug: true };
for (const [key, value] of Object.entries(config)) {
  console.log(`${key} = ${value}`);
}
```

---

## 5. `for await...of` (ES2018)

Iterate qua async iterables.

```javascript
// Async generator
async function* fetchPages(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    yield await response.json();
  }
}

// for await...of
async function processAll() {
  const urls = ["/api/page1", "/api/page2", "/api/page3"];
  
  for await (const data of fetchPages(urls)) {
    console.log("Received:", data);
    // Xử lý tuần tự, đợi page trước xong mới fetch page sau
  }
}

// Với Streams (Node.js)
const stream = fs.createReadStream("large-file.txt", { encoding: "utf8" });
for await (const chunk of stream) {
  console.log("Chunk:", chunk.length);
}
```

---

## 6. Iterate Objects (not iterable by default)

Objects thông thường **KHÔNG** implement `[Symbol.iterator]`, nên không dùng `for...of` trực tiếp được.

```javascript
const obj = { a: 1, b: 2, c: 3 };

// ❌ KHÔNG hoạt động
// for (const value of obj) {} // TypeError: obj is not iterable

// ✅ Workarounds:
// 1. Object.entries()
for (const [key, value] of Object.entries(obj)) {
  console.log(key, value);
}

// 2. Object.keys()
for (const key of Object.keys(obj)) {
  console.log(key, obj[key]);
}

// 3. Object.values()
for (const value of Object.values(obj)) {
  console.log(value);
}

// 4. Thêm [Symbol.iterator] cho object
const iterableObj = {
  a: 1, b: 2, c: 3,
  [Symbol.iterator]() {
    const entries = Object.entries(this).filter(
      ([key]) => key !== Symbol.iterator.toString()
    );
    let index = 0;
    return {
      next() {
        if (index < entries.length) {
          return { value: entries[index++], done: false };
        }
        return { done: true };
      }
    };
  }
};
```

---

## 7. Best Practices

```javascript
// ✅ Dùng for...of cho iterables (array, string, map, set)
for (const item of items) { }

// ✅ Dùng for...in cho object properties
for (const key in config) {
  if (config.hasOwnProperty(key)) { } // Filter inherited props
}

// ✅ Dùng forEach cho simple array operations (no break needed)
items.forEach(item => processItem(item));

// ✅ Dùng for...of + entries() khi cần index
for (const [i, item] of items.entries()) { }

// ✅ Dùng for...of khi cần break/continue
for (const item of items) {
  if (shouldSkip(item)) continue;
  if (isDone(item)) break;
  process(item);
}

// ❌ Tránh for...in với arrays (vì includes inherited props)
for (const i in array) { } // Bad practice
```

---

> Tiếp theo: [New Built-in Methods →](./15-new-methods.md)
