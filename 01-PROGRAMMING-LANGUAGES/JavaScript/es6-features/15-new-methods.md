# New Built-in Methods (Array, String, Object, Number/Math)

> Quay lại [Mục lục](./README.md) | Trước: [for...of Loop](./14-for-of-loop.md)

## 📌 Tổng quan

ES6 và các phiên bản ES tiếp theo bổ sung nhiều methods mới cho các built-in objects. File này tổng hợp tất cả các methods quan trọng được thêm vào.

---

## 1. Array Methods mới

### 1.1. `Array.from()` — Array-like → Array

```javascript
// Convert array-like objects
const nodeList = document.querySelectorAll("div");
const arr = Array.from(nodeList);

// String → Array
Array.from("hello"); // ["h", "e", "l", "l", "o"]

// Set → Array
Array.from(new Set([1, 2, 3])); // [1, 2, 3]

// Map → Array of entries
Array.from(new Map([["a", 1]])); // [["a", 1]]

// Với map function (2nd argument)
Array.from([1, 2, 3], x => x * 2); // [2, 4, 6]

// Generate sequences
Array.from({ length: 5 }, (_, i) => i);     // [0, 1, 2, 3, 4]
Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]
Array.from({ length: 26 }, (_, i) => String.fromCharCode(65 + i));
// ["A", "B", "C", ..., "Z"]

// 2D array
Array.from({ length: 3 }, () => Array(3).fill(0));
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]

// arguments object
function example() {
  return Array.from(arguments);
}
```

### 1.2. `Array.of()` — Tạo array từ arguments

```javascript
Array.of(1, 2, 3);  // [1, 2, 3]
Array.of(7);         // [7] ← Khác Array(7)!

// So sánh
Array(7);            // [<7 empty items>] (7 empty slots)
Array(1, 2, 3);      // [1, 2, 3]
Array.of(7);         // [7]
```

### 1.3. `find()` và `findIndex()`

```javascript
const users = [
  { id: 1, name: "John", age: 25 },
  { id: 2, name: "Jane", age: 30 },
  { id: 3, name: "Bob", age: 35 },
];

// find() — trả về ELEMENT đầu tiên match
const user = users.find(u => u.age > 25);
// { id: 2, name: "Jane", age: 30 }

const notFound = users.find(u => u.age > 100);
// undefined

// findIndex() — trả về INDEX đầu tiên match
const index = users.findIndex(u => u.name === "Jane");
// 1

const notFoundIndex = users.findIndex(u => u.name === "Unknown");
// -1

// ✅ ES2023: findLast() và findLastIndex() — tìm từ cuối
const lastOver25 = users.findLast(u => u.age > 25);
// { id: 3, name: "Bob", age: 35 }
```

### 1.4. `includes()` (ES2016)

```javascript
const arr = [1, 2, 3, NaN];

arr.includes(2);    // true
arr.includes(4);    // false
arr.includes(NaN);  // true ✅

// vs indexOf (không tìm được NaN)
arr.indexOf(NaN) !== -1; // false ❌

// From index
arr.includes(2, 2); // false (tìm từ index 2 trở đi)
```

### 1.5. `fill()`

```javascript
const arr = [1, 2, 3, 4, 5];

// fill(value, start?, end?)
arr.fill(0);        // [0, 0, 0, 0, 0]
[1,2,3,4,5].fill(9, 2);      // [1, 2, 9, 9, 9]
[1,2,3,4,5].fill(7, 1, 3);   // [1, 7, 7, 4, 5]

// Create array with default
new Array(5).fill(0);  // [0, 0, 0, 0, 0]

// ⚠️ fill với objects — cùng reference!
const bad = new Array(3).fill({});
bad[0].x = 1;
console.log(bad); // [{x:1}, {x:1}, {x:1}] ← Tất cả cùng object!

// ✅ Dùng Array.from cho objects riêng biệt
const good = Array.from({ length: 3 }, () => ({}));
```

### 1.6. `copyWithin()`

```javascript
// copyWithin(target, start, end?) — copy phần tử trong chính array
[1, 2, 3, 4, 5].copyWithin(0, 3);    // [4, 5, 3, 4, 5]
[1, 2, 3, 4, 5].copyWithin(1, 3);    // [1, 4, 5, 4, 5]
[1, 2, 3, 4, 5].copyWithin(0, 3, 4); // [4, 2, 3, 4, 5]
```

### 1.7. `entries()`, `keys()`, `values()`

```javascript
const arr = ["a", "b", "c"];

// entries() — [index, value]
for (const [index, value] of arr.entries()) {
  console.log(index, value); // 0 "a", 1 "b", 2 "c"
}

// keys() — indices
for (const index of arr.keys()) {
  console.log(index); // 0, 1, 2
}

// values() — values
for (const value of arr.values()) {
  console.log(value); // "a", "b", "c"
}
```

### 1.8. `flat()` và `flatMap()` (ES2019)

```javascript
// flat(depth) — flatten nested arrays
[1, [2, [3, [4]]]].flat();         // [1, 2, [3, [4]]]
[1, [2, [3, [4]]]].flat(2);        // [1, 2, 3, [4]]
[1, [2, [3, [4]]]].flat(Infinity);  // [1, 2, 3, 4]

// Remove empty slots
[1, , 3, , 5].flat(); // [1, 3, 5]

// flatMap() — map() + flat(1)
const sentences = ["Hello world", "How are you"];
sentences.flatMap(s => s.split(" "));
// ["Hello", "world", "How", "are", "you"]

// Practical: filter + map in one step
const nums = [1, 2, 3, 4, 5];
nums.flatMap(n => n % 2 === 0 ? [n * 2] : []);
// [4, 8] — chỉ double số chẵn
```

### 1.9. Immutable Methods (ES2023)

```javascript
const arr = [3, 1, 4, 1, 5];

// Mutable (modify original)
arr.sort();       // arr bị thay đổi!
arr.reverse();    // arr bị thay đổi!
arr.splice(1, 1); // arr bị thay đổi!

// ✅ Immutable (ES2023 — return new array)
const sorted = arr.toSorted();          // arr giữ nguyên
const reversed = arr.toReversed();      // arr giữ nguyên
const spliced = arr.toSpliced(1, 1);    // arr giữ nguyên
const replaced = arr.with(0, 99);       // arr giữ nguyên, index 0 → 99
```

---

## 2. String Methods mới

### 2.1. `startsWith()`, `endsWith()`, `includes()`

```javascript
const str = "Hello, World!";

// startsWith(searchString, position?)
str.startsWith("Hello");     // true
str.startsWith("World", 7);  // true

// endsWith(searchString, length?)
str.endsWith("!");            // true
str.endsWith("World", 12);   // true (check first 12 chars)

// includes(searchString, position?)
str.includes("World");       // true
str.includes("world");       // false (case-sensitive)
str.includes("o", 5);        // true (from position 5)
```

### 2.2. `repeat()`

```javascript
"abc".repeat(3);   // "abcabcabc"
"abc".repeat(0);   // ""
"abc".repeat(1.5); // "abc" (truncated to 1)

// Practical
const indent = "  ".repeat(depth);
const separator = "─".repeat(40);
const stars = "⭐".repeat(rating);
```

### 2.3. `padStart()` và `padEnd()` (ES2017)

```javascript
// padStart(targetLength, padString?)
"5".padStart(3, "0");      // "005"
"42".padStart(5, "0");     // "00042"
"hello".padStart(10);      // "     hello"

// padEnd(targetLength, padString?)
"5".padEnd(3, "0");        // "500"
"hello".padEnd(10, ".");   // "hello....."

// Practical
const hours = String(h).padStart(2, "0");
const binary = (255).toString(2).padStart(8, "0"); // "11111111"

// Table alignment
const data = [["Name", "Age"], ["John", "25"], ["Jane", "30"]];
data.forEach(([name, age]) => {
  console.log(`${name.padEnd(10)} ${age.padStart(5)}`);
});
```

### 2.4. `trimStart()` và `trimEnd()` (ES2019)

```javascript
const str = "   Hello World   ";

str.trim();      // "Hello World"
str.trimStart(); // "Hello World   "
str.trimEnd();   // "   Hello World"

// Aliases
str.trimLeft();  // Same as trimStart()
str.trimRight(); // Same as trimEnd()
```

### 2.5. `replaceAll()` (ES2021)

```javascript
const str = "foo-bar-baz";

// replace() — chỉ thay thế đầu tiên
str.replace("-", "_");    // "foo_bar-baz"

// replaceAll() — thay thế tất cả
str.replaceAll("-", "_"); // "foo_bar_baz"

// Trước ES2021
str.replace(/-/g, "_");   // "foo_bar_baz" (regex with g flag)
str.split("-").join("_"); // "foo_bar_baz"
```

### 2.6. `matchAll()` (ES2020)

```javascript
const str = "Hello 123 World 456";
const regex = /\d+/g;

for (const match of str.matchAll(regex)) {
  console.log(match[0], match.index);
}
// "123" 6
// "456" 16
```

### 2.7. `at()` (ES2022)

```javascript
const str = "Hello";

str.at(0);   // "H"
str.at(-1);  // "o" ← Negative index!
str.at(-2);  // "l"

// Works on arrays too
[1, 2, 3].at(-1); // 3
```

---

## 3. Object Methods mới

### 3.1. `Object.assign()` (ES6)

```javascript
// Shallow copy / merge
const target = { a: 1 };
const source = { b: 2, c: 3 };
Object.assign(target, source);
// target = { a: 1, b: 2, c: 3 }

// Clone
const clone = Object.assign({}, original);

// Multiple sources (later sources override)
const defaults = { theme: "light", lang: "en" };
const userPrefs = { theme: "dark" };
const settings = Object.assign({}, defaults, userPrefs);
// { theme: "dark", lang: "en" }

// ⚠️ Shallow only
const obj = { nested: { value: 1 } };
const copy = Object.assign({}, obj);
copy.nested.value = 2;
console.log(obj.nested.value); // 2 ← affected!

// ✅ Modern alternative: spread
const clone = { ...original };
const merged = { ...defaults, ...userPrefs };
```

### 3.2. `Object.is()` (ES6)

```javascript
// Strict equality với special cases
Object.is(25, 25);     // true
Object.is("foo", "foo"); // true
Object.is(NaN, NaN);   // true ✅ (=== returns false)
Object.is(0, -0);      // false ✅ (=== returns true)
Object.is(null, null);  // true

// vs ===
NaN === NaN;  // false
0 === -0;     // true
```

### 3.3. `Object.keys()`, `values()`, `entries()`, `fromEntries()`

```javascript
const obj = { a: 1, b: 2, c: 3 };

// Object.keys() (ES5)
Object.keys(obj);    // ["a", "b", "c"]

// Object.values() (ES2017)
Object.values(obj);  // [1, 2, 3]

// Object.entries() (ES2017)
Object.entries(obj); // [["a", 1], ["b", 2], ["c", 3]]

// Object.fromEntries() (ES2019) — reverse of entries()
Object.fromEntries([["a", 1], ["b", 2]]); // { a: 1, b: 2 }

// Map → Object
const map = new Map([["x", 10], ["y", 20]]);
Object.fromEntries(map); // { x: 10, y: 20 }

// Transform object
const doubled = Object.fromEntries(
  Object.entries(obj).map(([key, value]) => [key, value * 2])
);
// { a: 2, b: 4, c: 6 }

// Filter object
const filtered = Object.fromEntries(
  Object.entries(obj).filter(([key, value]) => value > 1)
);
// { b: 2, c: 3 }
```

### 3.4. `Object.getOwnPropertyDescriptors()` (ES2017)

```javascript
const obj = {
  name: "John",
  get fullName() { return this.name; },
};

// Get all property descriptors
Object.getOwnPropertyDescriptors(obj);
// {
//   name: { value: "John", writable: true, enumerable: true, configurable: true },
//   fullName: { get: [Function], set: undefined, enumerable: true, configurable: true }
// }

// ✅ Proper cloning (preserves getters/setters)
const clone = Object.defineProperties(
  {},
  Object.getOwnPropertyDescriptors(obj)
);
```

### 3.5. `Object.hasOwn()` (ES2022)

```javascript
const obj = { a: 1 };

// ✅ Modern way
Object.hasOwn(obj, "a");           // true
Object.hasOwn(obj, "toString");    // false (inherited)

// ❌ Old way (has issues)
obj.hasOwnProperty("a");           // true
// Nhưng fails nếu obj = Object.create(null)
```

### 3.6. `Object.groupBy()` (ES2024)

```javascript
const people = [
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 25 },
  { name: "Diana", age: 30 },
];

const grouped = Object.groupBy(people, person => person.age);
// {
//   25: [{ name: "Alice", ... }, { name: "Charlie", ... }],
//   30: [{ name: "Bob", ... }, { name: "Diana", ... }]
// }
```

---

## 4. Number & Math Methods mới

### 4.1. Number Methods

```javascript
// Number.isFinite() — stricter than global isFinite()
Number.isFinite(42);        // true
Number.isFinite(Infinity);  // false
Number.isFinite(NaN);       // false
Number.isFinite("42");      // false (global isFinite("42") = true!)

// Number.isNaN() — stricter than global isNaN()
Number.isNaN(NaN);          // true
Number.isNaN("NaN");        // false (global isNaN("NaN") = true!)
Number.isNaN(undefined);    // false

// Number.isInteger()
Number.isInteger(42);       // true
Number.isInteger(42.0);     // true
Number.isInteger(42.5);     // false

// Number.isSafeInteger()
Number.isSafeInteger(42);                    // true
Number.isSafeInteger(Math.pow(2, 53));       // false
Number.isSafeInteger(Math.pow(2, 53) - 1);   // true (MAX_SAFE_INTEGER)

// Number.parseFloat(), Number.parseInt() — giống global nhưng "cleaner"
Number.parseFloat("3.14");  // 3.14
Number.parseInt("42px");    // 42

// Constants
Number.EPSILON;           // ~2.22e-16 (smallest float difference)
Number.MAX_SAFE_INTEGER;  // 9007199254740991 (2^53 - 1)
Number.MIN_SAFE_INTEGER;  // -9007199254740991
```

### 4.2. Math Methods

```javascript
// Math.sign() — dấu của số
Math.sign(5);   // 1
Math.sign(-5);  // -1
Math.sign(0);   // 0

// Math.trunc() — bỏ phần thập phân (không làm tròn)
Math.trunc(4.7);   // 4
Math.trunc(-4.7);  // -4 (khác Math.floor(-4.7) = -5)

// Math.cbrt() — căn bậc 3
Math.cbrt(8);   // 2
Math.cbrt(27);  // 3

// Math.log2(), Math.log10()
Math.log2(8);    // 3
Math.log10(100); // 2

// Math.hypot() — √(a² + b² + ...)
Math.hypot(3, 4);    // 5
Math.hypot(3, 4, 5); // 7.07...

// Math.clz32() — count leading zeros (32-bit)
Math.clz32(1);   // 31
Math.clz32(256); // 23

// Math.imul() — C-like 32-bit integer multiplication
Math.imul(2, 4); // 8

// Math.fround() — nearest 32-bit float
Math.fround(1.337); // 1.3370000123977661
```

---

## 5. Tổng hợp nhanh

| Method | ES Version | Mục đích |
|--------|-----------|----------|
| `Array.from()` | ES6 | Array-like → Array |
| `Array.of()` | ES6 | Tạo array từ arguments |
| `find() / findIndex()` | ES6 | Tìm element/index |
| `includes()` | ES2016 | Kiểm tra tồn tại |
| `flat() / flatMap()` | ES2019 | Flatten arrays |
| `at()` | ES2022 | Negative indexing |
| `toSorted() / toReversed()` | ES2023 | Immutable sort/reverse |
| `String.includes()` | ES6 | Substring check |
| `startsWith() / endsWith()` | ES6 | Position check |
| `repeat()` | ES6 | Repeat string |
| `padStart() / padEnd()` | ES2017 | Padding |
| `replaceAll()` | ES2021 | Replace all occurrences |
| `Object.assign()` | ES6 | Merge objects |
| `Object.is()` | ES6 | Strict comparison |
| `Object.entries()` | ES2017 | Key-value pairs |
| `Object.fromEntries()` | ES2019 | Entries → Object |
| `Object.hasOwn()` | ES2022 | Own property check |
| `Object.groupBy()` | ES2024 | Group by key |
| `Number.isFinite()` | ES6 | Strict finite check |
| `Number.isNaN()` | ES6 | Strict NaN check |
| `Math.trunc()` | ES6 | Truncate decimal |
| `Math.sign()` | ES6 | Sign of number |

---

> Tiếp theo: [Proxy & Reflect →](./16-proxy-reflect.md)
