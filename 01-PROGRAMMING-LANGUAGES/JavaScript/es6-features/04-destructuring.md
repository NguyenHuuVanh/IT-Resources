# Destructuring

> Quay lại [Mục lục](./README.md) | Trước: [Template Literals](./03-template-literals.md)

## 📌 Tổng quan

**Destructuring** (phân rã) là cú pháp cho phép "trích xuất" giá trị từ arrays hoặc properties từ objects vào các biến riêng lẻ. Nó giúp code ngắn gọn, dễ đọc và giảm thiểu code lặp.

---

## 1. Array Destructuring

### 1.1. Cú pháp cơ bản

```javascript
// ES5
var arr = [1, 2, 3];
var a = arr[0];
var b = arr[1];
var c = arr[2];

// ES6 - Array destructuring
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1, 2, 3

// Từ biến array
const colors = ["red", "green", "blue"];
const [first, second, third] = colors;
console.log(first);  // "red"
console.log(second); // "green"
console.log(third);  // "blue"
```

### 1.2. Bỏ qua phần tử (Skip Elements)

```javascript
// Dùng dấu phẩy để bỏ qua
const [a, , c] = [1, 2, 3];
console.log(a, c); // 1, 3

const [, , third] = ["Jan", "Feb", "Mar"];
console.log(third); // "Mar"

const [first, , , fourth] = [1, 2, 3, 4, 5];
console.log(first, fourth); // 1, 4
```

### 1.3. Default Values

```javascript
// Khi phần tử là undefined, default value được sử dụng
const [x = 10, y = 20, z = 30] = [5];
console.log(x, y, z); // 5, 20, 30

const [a = 1, b = 2] = [undefined, null];
console.log(a); // 1 (undefined → dùng default)
console.log(b); // null (null KHÔNG trigger default)

// Default với expression (lazy evaluation)
function getDefault() {
  console.log("Computing default...");
  return 42;
}
const [val = getDefault()] = [100];
// getDefault() KHÔNG được gọi vì val = 100

const [val2 = getDefault()] = [];
// "Computing default..." — getDefault() ĐƯỢC gọi
// val2 = 42
```

### 1.4. Rest Pattern

```javascript
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

const [first, second, ...rest] = "hello";
console.log(first);  // "h"
console.log(second); // "e"
console.log(rest);   // ["l", "l", "o"]

// ⚠️ Rest phải ở cuối
// const [...rest, last] = [1, 2, 3]; // ❌ SyntaxError
```

### 1.5. Swap Variables (hoán đổi biến)

```javascript
// ES5
var temp = a;
a = b;
b = temp;

// ES6 - Swap bằng destructuring
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // 2, 1

// Swap 3 biến
let x = 1, y = 2, z = 3;
[x, y, z] = [z, x, y];
console.log(x, y, z); // 3, 1, 2
```

### 1.6. Nested Array Destructuring

```javascript
const nested = [1, [2, 3], [4, [5, 6]]];
const [a, [b, c], [d, [e, f]]] = nested;
console.log(a, b, c, d, e, f); // 1, 2, 3, 4, 5, 6

// Matrix
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];
const [[a1, , a3], , [, , c3]] = matrix;
console.log(a1, a3, c3); // 1, 3, 9
```

### 1.7. Destructuring từ Function Return

```javascript
function getCoordinates() {
  return [10, 20];
}
const [x, y] = getCoordinates();
console.log(x, y); // 10, 20

// React Hook pattern
function useState(initial) {
  let state = initial;
  const setState = (newValue) => { state = newValue; };
  return [state, setState];
}
const [count, setCount] = useState(0);
```

---

## 2. Object Destructuring

### 2.1. Cú pháp cơ bản

```javascript
// ES5
var person = { name: "John", age: 25, city: "NYC" };
var name = person.name;
var age = person.age;
var city = person.city;

// ES6 - Object destructuring
const person = { name: "John", age: 25, city: "NYC" };
const { name, age, city } = person;
console.log(name, age, city); // "John", 25, "NYC"

// ⚠️ Thứ tự KHÔNG quan trọng (khác array destructuring)
const { city, name, age } = person;
// Vẫn OK - match theo property name
```

### 2.2. Aliasing (đổi tên biến)

```javascript
// Khi tên property không phù hợp hoặc bị trùng
const { name: userName, age: userAge } = person;
console.log(userName); // "John"
console.log(userAge);  // 25
// console.log(name);  // ❌ ReferenceError (name không tồn tại)

// Useful khi tên property xung đột
const { name: authorName } = author;
const { name: bookName } = book;
```

### 2.3. Default Values

```javascript
const person = { name: "John", age: 25 };

// Default khi property undefined
const { name, country = "USA" } = person;
console.log(country); // "USA" (không tồn tại trong person)

// Alias + Default cùng lúc
const { name: n, country: c = "USA" } = person;
console.log(n); // "John"
console.log(c); // "USA"

// null KHÔNG trigger default (giống array)
const { value = 10 } = { value: null };
console.log(value); // null (KHÔNG phải 10)

const { value2 = 10 } = { value2: undefined };
console.log(value2); // 10 (undefined → dùng default)
```

### 2.4. Nested Object Destructuring

```javascript
const user = {
  id: 1,
  profile: {
    firstName: "John",
    lastName: "Doe",
    address: {
      street: "123 Main St",
      city: "NYC",
      zip: "10001",
    },
  },
  settings: {
    theme: "dark",
    notifications: true,
  },
};

// Destructure nested properties
const {
  profile: {
    firstName,
    lastName,
    address: { city, zip },
  },
  settings: { theme },
} = user;

console.log(firstName); // "John"
console.log(city);      // "NYC"
console.log(theme);     // "dark"

// ⚠️ profile, address, settings KHÔNG phải biến
// console.log(profile); // ❌ ReferenceError

// Nếu muốn cả parent và nested
const { profile, profile: { firstName: fn } } = user;
console.log(profile);  // { firstName: "John", ... }
console.log(fn);       // "John"
```

### 2.5. Rest Pattern với Object

```javascript
const { name, ...rest } = { name: "John", age: 25, city: "NYC", role: "admin" };
console.log(name); // "John"
console.log(rest); // { age: 25, city: "NYC", role: "admin" }

// ✅ Practical: Loại bỏ property (giống omit)
const { password, __v, ...safeUser } = userFromDB;
// safeUser không chứa password và __v

// ✅ Practical: Tách props
const { className, style, ...htmlProps } = props;
```

### 2.6. Computed Property Names

```javascript
const key = "name";
const { [key]: value } = { name: "John" };
console.log(value); // "John"

// Dynamic key
const field = "email";
const { [field]: fieldValue = "N/A" } = formData;
```

---

## 3. Function Parameter Destructuring

### 3.1. Array Parameter

```javascript
function sum([a, b]) {
  return a + b;
}
sum([1, 2]); // 3

function getFirst([first, ...rest]) {
  return { first, rest };
}
getFirst([1, 2, 3]); // { first: 1, rest: [2, 3] }
```

### 3.2. Object Parameter

```javascript
// ❌ Trước ES6 - nhiều parameters khó nhớ thứ tự
function createUser(name, age, email, role, active) {
  // Gọi: createUser("John", 25, "j@x.com", "admin", true)
  // Phải nhớ thứ tự!
}

// ✅ ES6 - Object parameter (không cần nhớ thứ tự)
function createUser({ name, age, email, role, active }) {
  return { name, age, email, role, active };
}

createUser({
  name: "John",
  email: "j@x.com",
  age: 25,     // Thứ tự tùy ý
  role: "admin",
  active: true,
});
```

### 3.3. Với Default Values

```javascript
// Default cho properties + default cho toàn bộ object
function createUser({
  name = "Anonymous",
  age = 0,
  role = "user",
  active = true,
} = {}) {
  return { name, age, role, active };
}

createUser({ name: "John" });
// { name: "John", age: 0, role: "user", active: true }

createUser();
// { name: "Anonymous", age: 0, role: "user", active: true }

// ⚠️ Nếu không có = {} ở cuối
function badCreate({ name = "Anonymous" }) {
  return name;
}
badCreate();       // ❌ TypeError: Cannot destructure property 'name' of undefined
badCreate({});     // ✅ "Anonymous"
```

### 3.4. Mixed Destructuring

```javascript
function processData({
  user: { name, email },
  items: [firstItem, ...restItems],
  options: { sort = "asc", limit = 10 } = {},
}) {
  console.log(name, email);
  console.log(firstItem, restItems);
  console.log(sort, limit);
}

processData({
  user: { name: "John", email: "j@x.com" },
  items: ["apple", "banana", "cherry"],
  options: { sort: "desc" },
});
```

---

## 4. Practical Examples

### 4.1. API Response Handling

```javascript
// Fetch API response
const response = {
  data: {
    users: [
      { id: 1, name: "John", email: "j@x.com" },
      { id: 2, name: "Jane", email: "jane@x.com" },
    ],
    pagination: {
      total: 100,
      page: 1,
      perPage: 10,
    },
  },
  status: 200,
  headers: { "content-type": "application/json" },
};

const {
  data: { users, pagination: { total, page } },
  status,
} = response;

console.log(users.length); // 2
console.log(total, page);  // 100, 1
console.log(status);       // 200
```

### 4.2. Import Pattern

```javascript
// Named imports sử dụng destructuring syntax
const { map, filter, reduce } = require("lodash");

// React hooks
const { useState, useEffect, useCallback, useMemo } = React;
```

### 4.3. Regex Match Groups

```javascript
const dateStr = "2024-01-15";
const [, year, month, day] = dateStr.match(/(\d{4})-(\d{2})-(\d{2})/);
console.log(year, month, day); // "2024", "01", "15"

// Named groups (ES2018)
const { groups: { y, m, d } } = dateStr.match(
  /(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/
);
```

### 4.4. React Patterns

```javascript
// useState
const [count, setCount] = useState(0);
const [user, setUser] = useState(null);
const [items, setItems] = useState([]);

// Props destructuring
function UserCard({ name, age, avatar, onEdit, onDelete }) {
  return (
    <div className="card">
      <img src={avatar} alt={name} />
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <button onClick={onEdit}>Edit</button>
      <button onClick={onDelete}>Delete</button>
    </div>
  );
}

// Context
const { theme, toggleTheme } = useContext(ThemeContext);
```

### 4.5. Config / Options Pattern

```javascript
function connect({
  host = "localhost",
  port = 3306,
  database,
  user = "root",
  password = "",
  ssl = false,
  pool = { min: 2, max: 10 },
  timeout = 30000,
} = {}) {
  if (!database) throw new Error("Database name is required");
  
  console.log(`Connecting to ${user}@${host}:${port}/${database}`);
  console.log(`SSL: ${ssl}, Pool: ${pool.min}-${pool.max}`);
  // ...
}

connect({
  host: "production-db.example.com",
  database: "myapp",
  password: "secret",
  ssl: true,
});
```

### 4.6. Loop Destructuring

```javascript
// Array of objects
const users = [
  { name: "John", age: 25 },
  { name: "Jane", age: 30 },
];

for (const { name, age } of users) {
  console.log(`${name} is ${age} years old`);
}

// Map entries
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);
for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// Object.entries()
const config = { host: "localhost", port: 3000, debug: true };
for (const [key, value] of Object.entries(config)) {
  console.log(`${key} = ${value}`);
}
```

---

## 5. Edge Cases và Lưu ý

### 5.1. Destructuring null/undefined

```javascript
// ❌ Không thể destructure null hoặc undefined
const { name } = null;      // TypeError
const { name } = undefined; // TypeError
const [a] = null;            // TypeError

// ✅ Dùng nullish coalescing hoặc default
const { name } = response?.data ?? {};
```

### 5.2. Khai báo vs Assignment

```javascript
// Khai báo (declaration) - không cần ()
const { a, b } = obj;
let { x, y } = obj;

// Assignment - CẦN () vì {} bị hiểu nhầm là block
let a, b;
({ a, b } = { a: 1, b: 2 }); // ✅ Cần ()
// { a, b } = { a: 1, b: 2 }; // ❌ SyntaxError
```

### 5.3. Nested Destructuring với Optional Properties

```javascript
// ⚠️ Nested destructuring sẽ lỗi nếu parent là undefined
const user = { name: "John" };
// const { address: { city } } = user; // ❌ TypeError

// ✅ Safe alternatives
const { address: { city } = {} } = user; // city = undefined
const city = user?.address?.city;         // Optional chaining
```

---

## 6. Câu hỏi phỏng vấn thường gặp

### Q1: Destructuring là gì? Có mấy loại?
**A**: Destructuring là cú pháp trích xuất giá trị từ arrays/objects vào biến. Có 2 loại: Array destructuring (dựa trên vị trí) và Object destructuring (dựa trên tên property).

### Q2: Output là gì?

```javascript
const { a: { b } = {} } = { a: { b: 5 } };
console.log(b); // 5 (a.b = 5, default {} không được dùng)

const { a: { b } = {} } = {};
console.log(b); // undefined (a = undefined → dùng default {} → b = undefined)
```

### Q3: Swap 2 biến không dùng biến phụ?
```javascript
let a = 1, b = 2;
[a, b] = [b, a]; // ✅ Array destructuring
```

---

> Tiếp theo: [Spread & Rest Operators →](./05-spread-rest.md)
