# Spread & Rest Operators

> Quay lại [Mục lục](./README.md) | Trước: [Destructuring](./04-destructuring.md)

## 📌 Tổng quan

Cùng sử dụng cú pháp `...` nhưng **Spread** và **Rest** có vai trò ngược nhau:
- **Spread** (trải ra): "Mở rộng" một iterable thành các phần tử riêng lẻ
- **Rest** (gom lại): "Thu gom" các phần tử riêng lẻ thành một array/object

---

## 1. Spread Operator (...)

### 1.1. Array Spread

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// Nối arrays
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Copy array (shallow)
const copy = [...arr1]; // [1, 2, 3]
copy.push(4);
console.log(arr1); // [1, 2, 3] (không bị ảnh hưởng)

// Thêm phần tử
const withMore = [0, ...arr1, 4, 5]; // [0, 1, 2, 3, 4, 5]

// String → Array
const chars = [..."hello"]; // ["h", "e", "l", "l", "o"]

// Set → Array (loại bỏ trùng lặp)
const unique = [...new Set([1, 2, 2, 3, 3, 3])]; // [1, 2, 3]

// NodeList → Array
const divs = [...document.querySelectorAll("div")];
```

### 1.2. Object Spread (ES2018)

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// Nối objects
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2, c: 3, d: 4 }

// Copy object (shallow)
const copy = { ...obj1 };

// Override properties (property sau ghi đè property trước)
const updated = { ...obj1, b: 10 }; // { a: 1, b: 10 }

// Thêm properties
const extended = { ...obj1, e: 5, f: 6 }; // { a: 1, b: 2, e: 5, f: 6 }

// ⚠️ Thứ tự quan trọng!
const a = { x: 1, y: 2 };
const b = { y: 3, z: 4 };
const ab = { ...a, ...b }; // { x: 1, y: 3, z: 4 } — y bị ghi đè bởi b
const ba = { ...b, ...a }; // { y: 2, z: 4, x: 1 } — y bị ghi đè bởi a
```

### 1.3. Function Arguments

```javascript
// Math.max không nhận array
const numbers = [5, 2, 8, 1, 9];
// Math.max(numbers); // NaN

// ✅ Spread array thành arguments
console.log(Math.max(...numbers)); // 9
console.log(Math.min(...numbers)); // 1

// ES5 equivalent
console.log(Math.max.apply(null, numbers)); // 9

// Truyền vào function
function sum(a, b, c) {
  return a + b + c;
}
const args = [1, 2, 3];
sum(...args); // 6
```

---

## 2. Rest Operator (...)

### 2.1. Function Parameters

```javascript
// Gom tất cả arguments thành array
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4, 5); // 15

// Mixed: named params + rest
function greet(greeting, ...names) {
  return `${greeting} ${names.join(", ")}!`;
}
greet("Hello", "John", "Jane", "Bob");
// "Hello John, Jane, Bob!"

// ⚠️ Rest phải là parameter cuối cùng
function bad(...rest, last) {} // ❌ SyntaxError
function good(first, ...rest) {} // ✅ OK
```

### 2.2. Array Destructuring với Rest

```javascript
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]

// Head / Tail pattern
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// Init / Last pattern
const numbers = [1, 2, 3, 4, 5];
const last = numbers[numbers.length - 1];
const init = numbers.slice(0, -1);
```

### 2.3. Object Destructuring với Rest

```javascript
const { a, b, ...others } = { a: 1, b: 2, c: 3, d: 4, e: 5 };
console.log(a);      // 1
console.log(b);      // 2
console.log(others); // { c: 3, d: 4, e: 5 }

// ✅ Loại bỏ properties (omit pattern)
const { password, __v, ...safeUser } = {
  name: "John",
  email: "j@x.com",
  password: "secret",
  __v: 0,
};
console.log(safeUser); // { name: "John", email: "j@x.com" }
```

---

## 3. Spread vs Rest — So sánh rõ ràng

```
┌─────────────────────────────────────────────────────────────┐
│              SPREAD vs REST                                  │
├──────────────────────────┬──────────────────────────────────┤
│   SPREAD (Expanding)     │   REST (Collecting)              │
│   "Trải ra"              │   "Gom lại"                      │
├──────────────────────────┼──────────────────────────────────┤
│                          │                                  │
│ // Trải array            │ // Gom vào array                 │
│ const b = [...a, 4];     │ const [x, ...rest] = arr;       │
│                          │                                  │
│ // Trải object           │ // Gom vào object                │
│ const b = { ...a, c: 3 };│ const { x, ...rest } = obj;    │
│                          │                                  │
│ // Trải vào function     │ // Gom từ function               │
│ Math.max(...[1,2,3])     │ function fn(...args) {}          │
│                          │                                  │
│ Vị trí: bên PHẢI = hoặc │ Vị trí: bên TRÁI = hoặc         │
│ trong function call      │ trong function declaration       │
│                          │                                  │
└──────────────────────────┴──────────────────────────────────┘
```

**Cách nhớ đơn giản:**
- **Spread**: Xuất hiện ở **vế phải** (nơi cung cấp giá trị) → "trải ra, mở rộng"
- **Rest**: Xuất hiện ở **vế trái** (nơi nhận giá trị) → "thu gom, còn lại"

---

## 4. Shallow Copy vs Deep Copy

### 4.1. Spread chỉ tạo Shallow Copy

```javascript
// ⚠️ Spread chỉ copy 1 cấp (shallow)
const original = {
  name: "John",
  address: {
    city: "NYC",     // ← Đây là object reference
    zip: "10001",
  },
  hobbies: ["reading", "coding"], // ← Đây là array reference
};

const copy = { ...original };

// Thay đổi property cấp 1 — ĐỘC LẬP
copy.name = "Jane";
console.log(original.name); // "John" ✅ (không bị ảnh hưởng)

// Thay đổi property cấp 2 — BỊ LIÊN KẾT
copy.address.city = "LA";
console.log(original.address.city); // "LA" ❌ (bị thay đổi!)

copy.hobbies.push("gaming");
console.log(original.hobbies); // ["reading", "coding", "gaming"] ❌
```

### 4.2. Deep Copy

```javascript
// ✅ structuredClone (modern - recommended)
const deepCopy = structuredClone(original);

// ✅ JSON.parse/JSON.stringify (cũ hơn)
// ⚠️ Không hoạt động với: functions, undefined, Symbol, Date, RegExp, Map, Set...
const deepCopy = JSON.parse(JSON.stringify(original));

// ✅ Custom deep clone
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (Array.isArray(obj)) return obj.map(deepClone);
  
  return Object.fromEntries(
    Object.entries(obj).map(([key, val]) => [key, deepClone(val)])
  );
}
```

---

## 5. Practical Examples

### 5.1. Immutable Array Operations

```javascript
const arr = [1, 2, 3, 4, 5];

// Thêm phần tử (không mutate)
const add = (arr, item) => [...arr, item];
const prepend = (arr, item) => [item, ...arr];
const insertAt = (arr, index, item) => [
  ...arr.slice(0, index), 
  item, 
  ...arr.slice(index)
];

// Xóa phần tử
const removeAt = (arr, index) => [
  ...arr.slice(0, index), 
  ...arr.slice(index + 1)
];
const removeItem = (arr, item) => arr.filter(x => x !== item);

// Cập nhật phần tử
const updateAt = (arr, index, newItem) => [
  ...arr.slice(0, index), 
  newItem, 
  ...arr.slice(index + 1)
];

// Example
const original = [1, 2, 3];
const added = add(original, 4);      // [1, 2, 3, 4]
const removed = removeAt(original, 1); // [1, 3]
const updated = updateAt(original, 1, 20); // [1, 20, 3]
console.log(original); // [1, 2, 3] — không bị thay đổi!
```

### 5.2. Immutable Object Operations

```javascript
// Thêm/Update property
const addProp = (obj, key, value) => ({ ...obj, [key]: value });
const update = (obj, updates) => ({ ...obj, ...updates });

// Remove property
const removeProp = (obj, key) => {
  const { [key]: _, ...rest } = obj;
  return rest;
};

// Rename key
const renameKey = (obj, oldKey, newKey) => {
  const { [oldKey]: value, ...rest } = obj;
  return { ...rest, [newKey]: value };
};
```

### 5.3. Config Merging

```javascript
const defaults = {
  theme: "light",
  language: "en",
  notifications: true,
  fontSize: 14,
};

const userPrefs = {
  theme: "dark",
  language: "vi",
};

const settings = { ...defaults, ...userPrefs };
// { theme: "dark", language: "vi", notifications: true, fontSize: 14 }
```

### 5.4. Conditional Spread

```javascript
// Spread có điều kiện
const user = {
  name: "John",
  ...(isAdmin && { role: "admin", permissions: ["all"] }),
  ...(email && { email }),
  ...(phone ? { phone } : { phone: "N/A" }),
};

// Array conditional spread
const features = [
  "basic",
  ...(isPremium ? ["premium", "support"] : []),
  ...(isBeta ? ["experimental"] : []),
];
```

### 5.5. Function Options Pattern

```javascript
function fetchData({ url, method = "GET", ...options }) {
  return fetch(url, {
    method,
    ...options,  // Spread remaining options (headers, body, etc.)
  });
}

fetchData({
  url: "/api/users",
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "John" }),
});
```

---

## 6. Câu hỏi phỏng vấn thường gặp

### Q1: Spread với Rest khác gì nhau?
**A**: Cùng cú pháp `...` nhưng Spread "trải ra" (ở vế phải/function call), Rest "gom lại" (ở vế trái/function declaration).

### Q2: Spread copy là shallow hay deep?
**A**: Shallow copy. Chỉ copy được cấp 1, nested objects/arrays vẫn chia sẻ reference. Dùng `structuredClone()` cho deep copy.

### Q3: Cách loại bỏ property khỏi object mà không mutate?
```javascript
const { unwanted, ...rest } = obj;
```

---

> Tiếp theo: [Default Parameters →](./06-default-parameters.md)
