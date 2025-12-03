# ES6 (ECMAScript 2015) - Hướng dẫn toàn diện

## 📌 Mục lục

1. [Giới thiệu ES6](#1-giới-thiệu-es6)
2. [let và const](#2-let-và-const)
3. [Arrow Functions](#3-arrow-functions)
4. [Template Literals](#4-template-literals)
5. [Destructuring](#5-destructuring)
6. [Spread và Rest Operators](#6-spread-và-rest-operators)
7. [Default Parameters](#7-default-parameters)
8. [Enhanced Object Literals](#8-enhanced-object-literals)
9. [Classes](#9-classes)
10. [Modules (import/export)](#10-modules-importexport)
11. [Promises](#11-promises)
12. [Symbol](#12-symbol)
13. [Iterators và Generators](#13-iterators-và-generators)
14. [Map và Set](#14-map-và-set)
15. [for...of Loop](#15-forof-loop)
16. [Array Methods mới](#16-array-methods-mới)
17. [String Methods mới](#17-string-methods-mới)
18. [Object Methods mới](#18-object-methods-mới)
19. [Number và Math mới](#19-number-và-math-mới)
20. [Proxy và Reflect](#20-proxy-và-reflect)

---

## 1. Giới thiệu ES6

### ES6 là gì?

**ES6** (ECMAScript 2015) là phiên bản lớn nhất của JavaScript, giới thiệu nhiều tính năng mới giúp code ngắn gọn, dễ đọc và mạnh mẽ hơn.

### Lịch sử ECMAScript

| Version | Năm      | Tên gọi             | Highlights                                      |
| ------- | -------- | ------------------- | ----------------------------------------------- |
| ES1     | 1997     | ECMAScript 1        | First edition                                   |
| ES3     | 1999     | ECMAScript 3        | Regular expressions, try/catch                  |
| ES5     | 2009     | ECMAScript 5        | Strict mode, JSON, Array methods                |
| **ES6** | **2015** | **ECMAScript 2015** | **Classes, Modules, Promises, Arrow functions** |
| ES7     | 2016     | ECMAScript 2016     | includes(), \*\* operator                       |
| ES8     | 2017     | ECMAScript 2017     | async/await, Object.entries()                   |
| ES9+    | 2018+    | ECMAScript 2018+    | Rest/Spread properties, etc.                    |

### Tổng quan các tính năng ES6

```
┌─────────────────────────────────────────────────────────────┐
│                    ES6 NEW FEATURES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Variables:        let, const                              │
│  Functions:        Arrow functions, Default params         │
│  Strings:          Template literals, New methods          │
│  Objects:          Enhanced literals, Destructuring        │
│  Arrays:           Spread, Destructuring, New methods      │
│  Classes:          class, extends, super, static           │
│  Modules:          import, export                          │
│  Async:            Promises, Generators                    │
│  Collections:      Map, Set, WeakMap, WeakSet              │
│  Others:           Symbol, Proxy, Reflect, for...of        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. let và const

### var vs let vs const

| Feature           | var            | let      | const    |
| ----------------- | -------------- | -------- | -------- |
| **Scope**         | Function       | Block    | Block    |
| **Hoisting**      | ✅ (undefined) | ✅ (TDZ) | ✅ (TDZ) |
| **Re-declare**    | ✅             | ❌       | ❌       |
| **Re-assign**     | ✅             | ✅       | ❌       |
| **Global object** | ✅ (window)    | ❌       | ❌       |

### Block Scope

```javascript
// var - Function scope
function varExample() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 ✅ (var không có block scope)
}

// let - Block scope
function letExample() {
  if (true) {
    let y = 10;
  }
  console.log(y); // ❌ ReferenceError: y is not defined
}

// const - Block scope
function constExample() {
  if (true) {
    const z = 10;
  }
  console.log(z); // ❌ ReferenceError: z is not defined
}
```

### Temporal Dead Zone (TDZ)

```javascript
// var - Hoisted với giá trị undefined
console.log(a); // undefined
var a = 5;

// let/const - Hoisted nhưng trong TDZ
console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 5;

// TDZ visualization
// ┌─────────────────────────────┐
// │ TDZ starts                  │
// │ console.log(x); // ❌ Error │
// │ let x = 5; // TDZ ends      │
// │ console.log(x); // 5 ✅     │
// └─────────────────────────────┘
```

### const với Objects và Arrays

```javascript
// const không cho phép re-assign
const PI = 3.14159;
PI = 3.14; // ❌ TypeError: Assignment to constant variable

// NHƯNG có thể modify object/array
const person = { name: "John" };
person.name = "Jane"; // ✅ OK - modify property
person.age = 25; // ✅ OK - add property
person = { name: "Bob" }; // ❌ Error - re-assign

const numbers = [1, 2, 3];
numbers.push(4); // ✅ OK - modify array
numbers[0] = 10; // ✅ OK - modify element
numbers = [5, 6, 7]; // ❌ Error - re-assign

// Để freeze object hoàn toàn
const frozen = Object.freeze({ name: "John" });
frozen.name = "Jane"; // ❌ Silently fails (strict mode: Error)
```

### Best Practices

```javascript
// ✅ Mặc định dùng const
const API_URL = "https://api.example.com";
const config = { debug: true };

// ✅ Dùng let khi cần re-assign
let count = 0;
count++;

let user = null;
user = await fetchUser();

// ❌ Tránh dùng var
var oldWay = "avoid this"; // Không nên
```

---

## 3. Arrow Functions

### Cú pháp cơ bản

```javascript
// ES5 - Function expression
const add = function (a, b) {
  return a + b;
};

// ES6 - Arrow function
const add = (a, b) => {
  return a + b;
};

// Rút gọn - Implicit return (1 expression)
const add = (a, b) => a + b;

// 1 parameter - Không cần ()
const double = (x) => x * 2;

// 0 parameters - Cần ()
const greet = () => "Hello!";

// Return object - Cần ()
const createUser = (name, age) => ({ name, age });
```

### So sánh Function vs Arrow Function

```javascript
// ┌─────────────────────────────────────────────────────────┐
// │              FUNCTION vs ARROW FUNCTION                 │
// ├─────────────────────────────────────────────────────────┤
// │                                                         │
// │  Regular Function:                                      │
// │  • Có this riêng (dynamic)                             │
// │  • Có arguments object                                  │
// │  • Có thể dùng làm constructor (new)                   │
// │  • Có prototype                                         │
// │                                                         │
// │  Arrow Function:                                        │
// │  • Không có this riêng (lexical - từ scope cha)        │
// │  • Không có arguments object                            │
// │  • Không thể dùng làm constructor                       │
// │  • Không có prototype                                   │
// │                                                         │
// └─────────────────────────────────────────────────────────┘
```

### this trong Arrow Functions

```javascript
// ❌ Problem với regular function
const person = {
  name: 'John',
  hobbies: ['reading', 'gaming'],

  showHobbies: function() {
    this.hobbies.forEach(function(hobby) {
      // this ở đây là undefined (strict) hoặc window
      console.log(this.name + ' likes ' + hobby); // ❌ undefined likes reading
    });
  }
};

// ✅ Solution 1: Arrow function (ES6)
const person = {
  name: 'John',
  hobbies: ['reading', 'gaming'],

  showHobbies: function() {
    this.hobbies.forEach(hobby => {
      // Arrow function kế thừa this từ showHobbies
      console.log(this.name + ' likes ' + hobby); // ✅ John likes reading
    });
  }
};

// ✅ Solution 2: bind (ES5)
showHobbies: function() {
  this.hobbies.forEach(function(hobby) {
    console.log(this.name + ' likes ' + hobby);
  }.bind(this));
}

// ✅ Solution 3: that = this (ES5)
showHobbies: function() {
  const that = this;
  this.hobbies.forEach(function(hobby) {
    console.log(that.name + ' likes ' + hobby);
  });
}
```

### Khi nào KHÔNG dùng Arrow Functions

```javascript
// ❌ Object methods (cần this của object)
const calculator = {
  value: 0,
  add: (n) => {
    this.value += n; // ❌ this không phải calculator
  },
};

// ✅ Dùng regular function
const calculator = {
  value: 0,
  add(n) {
    this.value += n; // ✅ this là calculator
  },
};

// ❌ Event handlers (cần this là element)
button.addEventListener("click", () => {
  this.classList.toggle("active"); // ❌ this không phải button
});

// ✅ Dùng regular function
button.addEventListener("click", function () {
  this.classList.toggle("active"); // ✅ this là button
});

// ❌ Constructor functions
const Person = (name) => {
  this.name = name;
};
new Person("John"); // ❌ TypeError: Person is not a constructor

// ✅ Dùng regular function hoặc class
function Person(name) {
  this.name = name;
}
```

### Arrow Functions với Array Methods

```javascript
const numbers = [1, 2, 3, 4, 5];

// map
const doubled = numbers.map((n) => n * 2);
// [2, 4, 6, 8, 10]

// filter
const evens = numbers.filter((n) => n % 2 === 0);
// [2, 4]

// reduce
const sum = numbers.reduce((acc, n) => acc + n, 0);
// 15

// find
const found = numbers.find((n) => n > 3);
// 4

// every / some
const allPositive = numbers.every((n) => n > 0); // true
const hasEven = numbers.some((n) => n % 2 === 0); // true

// Chaining
const result = numbers
  .filter((n) => n % 2 === 0)
  .map((n) => n * 2)
  .reduce((acc, n) => acc + n, 0);
// 12 (2*2 + 4*2)
```

---

## 4. Template Literals

### Cú pháp cơ bản

```javascript
// ES5 - String concatenation
var name = "John";
var greeting = "Hello, " + name + "!";

// ES6 - Template literals (backticks `)
const name = "John";
const greeting = `Hello, ${name}!`;
```

### Multi-line Strings

```javascript
// ES5 - Cần \n hoặc concatenation
var html = "<div>\n" + "  <h1>Title</h1>\n" + "  <p>Content</p>\n" + "</div>";

// ES6 - Multi-line tự nhiên
const html = `
  <div>
    <h1>Title</h1>
    <p>Content</p>
  </div>
`;
```

### Expression Interpolation

```javascript
const a = 10;
const b = 20;

// Expressions
console.log(`Sum: ${a + b}`); // Sum: 30
console.log(`Product: ${a * b}`); // Product: 200

// Function calls
const upper = (str) => str.toUpperCase();
console.log(`Hello ${upper("world")}`); // Hello WORLD

// Ternary operator
const age = 20;
console.log(`Status: ${age >= 18 ? "Adult" : "Minor"}`); // Status: Adult

// Object properties
const user = { name: "John", age: 25 };
console.log(`${user.name} is ${user.age} years old`);

// Array elements
const colors = ["red", "green", "blue"];
console.log(`First color: ${colors[0]}`); // First color: red
```

### Tagged Templates

```javascript
// Tagged template function
function highlight(strings, ...values) {
  console.log(strings); // ['Hello ', ', you have ', ' messages']
  console.log(values); // ['John', 5]

  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<strong>${values[i]}</strong>` : "");
  }, "");
}

const name = "John";
const count = 5;
const result = highlight`Hello ${name}, you have ${count} messages`;
// "Hello <strong>John</strong>, you have <strong>5</strong> messages"

// Practical example: SQL query builder
function sql(strings, ...values) {
  const escaped = values.map((v) => (typeof v === "string" ? `'${v.replace(/'/g, "''")}'` : v));
  return strings.reduce((query, str, i) => query + str + (escaped[i] || ""), "");
}

const username = "John'; DROP TABLE users; --";
const query = sql`SELECT * FROM users WHERE name = ${username}`;
// SELECT * FROM users WHERE name = 'John''; DROP TABLE users; --'

// CSS-in-JS example
function css(strings, ...values) {
  return strings.reduce((result, str, i) => result + str + (values[i] || ""), "");
}

const color = "blue";
const styles = css`
  .button {
    background: ${color};
    padding: 10px;
  }
`;
```

### Raw Strings

```javascript
// String.raw - Giữ nguyên escape sequences
console.log(`Line1\nLine2`);
// Line1
// Line2

console.log(String.raw`Line1\nLine2`);
// Line1\nLine2

// Useful for regex, file paths
const path = String.raw`C:\Users\John\Documents`;
console.log(path); // C:\Users\John\Documents
```

---

## 5. Destructuring

### Array Destructuring

```javascript
// Basic
const colors = ["red", "green", "blue"];
const [first, second, third] = colors;
console.log(first); // 'red'
console.log(second); // 'green'
console.log(third); // 'blue'

// Skip elements
const [a, , c] = [1, 2, 3];
console.log(a, c); // 1, 3

// Default values
const [x = 10, y = 20] = [5];
console.log(x, y); // 5, 20

// Rest pattern
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// Swap variables
let m = 1,
  n = 2;
[m, n] = [n, m];
console.log(m, n); // 2, 1

// Nested destructuring
const nested = [1, [2, 3], 4];
const [one, [two, three], four] = nested;
console.log(two, three); // 2, 3

// From function return
function getCoordinates() {
  return [10, 20];
}
const [x, y] = getCoordinates();
```

### Object Destructuring

```javascript
// Basic
const person = { name: "John", age: 25, city: "NYC" };
const { name, age, city } = person;
console.log(name, age, city); // John 25 NYC

// Different variable names (aliasing)
const { name: userName, age: userAge } = person;
console.log(userName, userAge); // John 25

// Default values
const { name, country = "USA" } = person;
console.log(country); // 'USA'

// Alias + Default
const { name: n, country: c = "USA" } = person;

// Nested destructuring
const user = {
  id: 1,
  profile: {
    firstName: "John",
    lastName: "Doe",
    address: {
      city: "NYC",
      zip: "10001",
    },
  },
};

const {
  profile: {
    firstName,
    address: { city },
  },
} = user;
console.log(firstName, city); // John NYC

// Rest pattern
const { name, ...rest } = { name: "John", age: 25, city: "NYC" };
console.log(name); // 'John'
console.log(rest); // { age: 25, city: 'NYC' }

// Computed property names
const key = "name";
const { [key]: value } = { name: "John" };
console.log(value); // 'John'
```

### Function Parameter Destructuring

```javascript
// Array parameter
function sum([a, b]) {
  return a + b;
}
sum([1, 2]); // 3

// Object parameter
function greet({ name, age }) {
  return `Hello ${name}, you are ${age}`;
}
greet({ name: "John", age: 25 }); // Hello John, you are 25

// With defaults
function createUser({ name = "Anonymous", age = 0, role = "user" } = {}) {
  return { name, age, role };
}

createUser({ name: "John" }); // { name: 'John', age: 0, role: 'user' }
createUser(); // { name: 'Anonymous', age: 0, role: 'user' }

// Mixed destructuring
function processData({ user: { name, email }, items: [firstItem, ...restItems], options = {} }) {
  console.log(name, email, firstItem, restItems, options);
}
```

### Practical Examples

```javascript
// API Response handling
const response = {
  data: {
    users: [
      { id: 1, name: "John" },
      { id: 2, name: "Jane" },
    ],
    total: 2,
  },
  status: 200,
};

const {
  data: { users, total },
  status,
} = response;

// React useState
const [count, setCount] = useState(0);
const [user, setUser] = useState(null);

// Import specific functions
const { map, filter, reduce } = require("lodash");

// Regex match groups
const [, year, month, day] = "2024-01-15".match(/(\d{4})-(\d{2})-(\d{2})/);
console.log(year, month, day); // 2024 01 15
```

---

## 6. Spread và Rest Operators

### Spread Operator (...)

```javascript
// ═══════════════════════════════════════════════════════════
// SPREAD - "Trải ra" elements
// ═══════════════════════════════════════════════════════════

// Array spread
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// Combine arrays
const combined = [...arr1, ...arr2];
// [1, 2, 3, 4, 5, 6]

// Copy array (shallow)
const copy = [...arr1];

// Add elements
const withMore = [0, ...arr1, 4, 5];
// [0, 1, 2, 3, 4, 5]

// Object spread
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// Combine objects
const merged = { ...obj1, ...obj2 };
// { a: 1, b: 2, c: 3, d: 4 }

// Copy object (shallow)
const objCopy = { ...obj1 };

// Override properties
const updated = { ...obj1, b: 10 };
// { a: 1, b: 10 }

// Add properties
const extended = { ...obj1, e: 5 };
// { a: 1, b: 2, e: 5 }

// Function arguments
const numbers = [1, 2, 3];
console.log(Math.max(...numbers)); // 3

function sum(a, b, c) {
  return a + b + c;
}
sum(...numbers); // 6

// String to array
const chars = [..."hello"];
// ['h', 'e', 'l', 'l', 'o']
```

### Rest Operator (...)

```javascript
// ═══════════════════════════════════════════════════════════
// REST - "Gom lại" elements còn lại
// ═══════════════════════════════════════════════════════════

// Function parameters
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4, 5); // 15

// Mixed parameters
function greet(greeting, ...names) {
  return `${greeting} ${names.join(", ")}!`;
}
greet("Hello", "John", "Jane", "Bob");
// "Hello John, Jane, Bob!"

// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(second); // 2
console.log(rest); // [3, 4, 5]

// Object destructuring
const { a, b, ...others } = { a: 1, b: 2, c: 3, d: 4 };
console.log(a); // 1
console.log(b); // 2
console.log(others); // { c: 3, d: 4 }
```

### Spread vs Rest

```javascript
// ┌─────────────────────────────────────────────────────────┐
// │              SPREAD vs REST                             │
// ├─────────────────────────────────────────────────────────┤
// │                                                         │
// │  SPREAD (Expanding)          REST (Collecting)         │
// │  ─────────────────          ─────────────────          │
// │                                                         │
// │  // Trải array ra           // Gom vào array           │
// │  const a = [1, 2];          function sum(...nums) {    │
// │  const b = [...a, 3];       return nums.reduce(...);   │
// │  // [1, 2, 3]               }                          │
// │                                                         │
// │  // Trải object ra          // Gom vào object          │
// │  const obj = {...a, c: 3};  const {x, ...rest} = obj;  │
// │                                                         │
// │  // Trải vào function       // Nhận từ function        │
// │  Math.max(...[1,2,3])       function f(...args) {}     │
// │                                                         │
// └─────────────────────────────────────────────────────────┘
```

### Practical Examples

```javascript
// Clone và modify
const original = { name: "John", age: 25 };
const modified = { ...original, age: 26 };

// Conditional spread
const baseConfig = { debug: false };
const config = {
  ...baseConfig,
  ...(process.env.NODE_ENV === "development" && { debug: true }),
};

// Remove property
const { password, ...userWithoutPassword } = user;

// Merge với override
const defaults = { theme: "light", lang: "en" };
const userPrefs = { theme: "dark" };
const settings = { ...defaults, ...userPrefs };
// { theme: 'dark', lang: 'en' }

// Array unique (với Set)
const unique = [...new Set([1, 2, 2, 3, 3, 3])];
// [1, 2, 3]

// Convert NodeList to Array
const divs = [...document.querySelectorAll("div")];

// Immutable array operations
const addItem = (arr, item) => [...arr, item];
const removeItem = (arr, index) => [...arr.slice(0, index), ...arr.slice(index + 1)];
const updateItem = (arr, index, newItem) => [...arr.slice(0, index), newItem, ...arr.slice(index + 1)];
```

---

## 7. Default Parameters

### Cú pháp cơ bản

```javascript
// ES5
function greet(name) {
  name = name || "Guest";
  return "Hello " + name;
}

// ES6
function greet(name = "Guest") {
  return `Hello ${name}`;
}

greet(); // "Hello Guest"
greet("John"); // "Hello John"
```

### Các loại Default Values

```javascript
// Primitive values
function example(str = "default", num = 42, bool = true, nul = null) {}

// Expressions
function calculate(a, b = a * 2) {
  return a + b;
}
calculate(5); // 15 (5 + 10)
calculate(5, 3); // 8

// Function calls
function getDefault() {
  console.log("Getting default...");
  return "default";
}

function greet(name = getDefault()) {
  return `Hello ${name}`;
}

greet("John"); // "Hello John" (getDefault không được gọi)
greet(); // "Getting default..." → "Hello default"

// Objects và Arrays
function createUser(options = {}) {
  const { name = "Anonymous", age = 0 } = options;
  return { name, age };
}

function processItems(items = []) {
  return items.map((item) => item.toUpperCase());
}
```

### Default với Destructuring

```javascript
// Object destructuring với defaults
function createUser({ name = "Anonymous", age = 0, role = "user" } = {}) {
  return { name, age, role };
}

createUser(); // { name: 'Anonymous', age: 0, role: 'user' }
createUser({}); // { name: 'Anonymous', age: 0, role: 'user' }
createUser({ name: "John" }); // { name: 'John', age: 0, role: 'user' }

// Array destructuring với defaults
function getFirstTwo([first = 0, second = 0] = []) {
  return [first, second];
}

getFirstTwo(); // [0, 0]
getFirstTwo([1]); // [1, 0]
getFirstTwo([1, 2]); // [1, 2]
```

### undefined vs null vs falsy

```javascript
function test(value = "default") {
  return value;
}

// Default chỉ apply khi undefined
test(); // 'default' (undefined)
test(undefined); // 'default'

// Các giá trị khác KHÔNG trigger default
test(null); // null
test(0); // 0
test(""); // ''
test(false); // false
test(NaN); // NaN
```

### Required Parameters Pattern

```javascript
// Tạo required parameter
function required(paramName) {
  throw new Error(`Parameter "${paramName}" is required`);
}

function createUser(name = required("name"), email = required("email")) {
  return { name, email };
}

createUser("John", "john@example.com"); // ✅ OK
createUser("John"); // ❌ Error: Parameter "email" is required
createUser(); // ❌ Error: Parameter "name" is required
```

---

## 8. Enhanced Object Literals

### Property Shorthand

```javascript
// ES5
var name = "John";
var age = 25;
var person = {
  name: name,
  age: age,
};

// ES6 - Property shorthand
const name = "John";
const age = 25;
const person = { name, age };
// { name: 'John', age: 25 }

// Mixed
const city = "NYC";
const user = {
  name: "John", // Regular
  age, // Shorthand
  city, // Shorthand
};
```

### Method Shorthand

```javascript
// ES5
var calculator = {
  add: function (a, b) {
    return a + b;
  },
  subtract: function (a, b) {
    return a - b;
  },
};

// ES6 - Method shorthand
const calculator = {
  add(a, b) {
    return a + b;
  },
  subtract(a, b) {
    return a - b;
  },
};

// Với async
const api = {
  async fetchData() {
    const response = await fetch("/api/data");
    return response.json();
  },
};

// Với generator
const iterator = {
  *values() {
    yield 1;
    yield 2;
    yield 3;
  },
};
```

### Computed Property Names

```javascript
// ES5
var prop = "name";
var obj = {};
obj[prop] = "John";

// ES6 - Computed property names
const prop = "name";
const obj = {
  [prop]: "John",
};
// { name: 'John' }

// Dynamic keys
const prefix = "user";
const user = {
  [`${prefix}Name`]: "John",
  [`${prefix}Age`]: 25,
  [`${prefix}Email`]: "john@example.com",
};
// { userName: 'John', userAge: 25, userEmail: 'john@example.com' }

// With expressions
const i = 0;
const obj = {
  [`item${i}`]: "first",
  [`item${i + 1}`]: "second",
};
// { item0: 'first', item1: 'second' }

// Symbol as key
const id = Symbol("id");
const user = {
  [id]: 12345,
  name: "John",
};
```

### Practical Examples

```javascript
// Redux action creator
const createAction = (type) => (payload) => ({ type, payload });

const addTodo = createAction("ADD_TODO");
addTodo({ text: "Learn ES6" });
// { type: 'ADD_TODO', payload: { text: 'Learn ES6' } }

// Dynamic object creation
function createObject(keys, values) {
  return keys.reduce(
    (obj, key, i) => ({
      ...obj,
      [key]: values[i],
    }),
    {}
  );
}

createObject(["a", "b", "c"], [1, 2, 3]);
// { a: 1, b: 2, c: 3 }

// State management
const setState = (key, value) => ({
  ...state,
  [key]: value,
});
```

---

## 9. Classes

### Class Declaration

```javascript
// ES5 - Constructor function
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function () {
  return `Hello, I'm ${this.name}`;
};

// ES6 - Class
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return `Hello, I'm ${this.name}`;
  }
}

const john = new Person("John", 25);
john.greet(); // "Hello, I'm John"
```

### Getters và Setters

```javascript
class Circle {
  constructor(radius) {
    this._radius = radius;
  }

  // Getter
  get radius() {
    return this._radius;
  }

  // Setter với validation
  set radius(value) {
    if (value <= 0) {
      throw new Error("Radius must be positive");
    }
    this._radius = value;
  }

  // Computed property
  get area() {
    return Math.PI * this._radius ** 2;
  }

  get circumference() {
    return 2 * Math.PI * this._radius;
  }
}

const circle = new Circle(5);
console.log(circle.radius); // 5
console.log(circle.area); // 78.54...
circle.radius = 10; // OK
circle.radius = -5; // Error!
```

### Static Methods và Properties

```javascript
class MathUtils {
  // Static property
  static PI = 3.14159;

  // Static method
  static add(a, b) {
    return a + b;
  }

  static multiply(a, b) {
    return a * b;
  }

  // Factory method
  static createRandom() {
    return Math.random();
  }
}

// Gọi trực tiếp trên class, không cần instance
console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.add(2, 3)); // 5
console.log(MathUtils.createRandom());

// ❌ Không thể gọi từ instance
const utils = new MathUtils();
utils.add(2, 3); // ❌ TypeError
```

### Inheritance (extends)

```javascript
// Parent class
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a sound`);
  }

  eat() {
    console.log(`${this.name} is eating`);
  }
}

// Child class
class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Gọi constructor của parent
    this.breed = breed;
  }

  // Override method
  speak() {
    console.log(`${this.name} barks`);
  }

  // New method
  fetch() {
    console.log(`${this.name} is fetching`);
  }
}

class Cat extends Animal {
  speak() {
    super.speak(); // Gọi method của parent
    console.log(`${this.name} meows`);
  }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.speak(); // "Buddy barks"
dog.eat(); // "Buddy is eating" (inherited)
dog.fetch(); // "Buddy is fetching"

const cat = new Cat("Whiskers");
cat.speak();
// "Whiskers makes a sound"
// "Whiskers meows"
```

### Private Fields (ES2022)

```javascript
class BankAccount {
  // Private fields (bắt đầu bằng #)
  #balance = 0;
  #pin;

  constructor(initialBalance, pin) {
    this.#balance = initialBalance;
    this.#pin = pin;
  }

  // Private method
  #validatePin(inputPin) {
    return this.#pin === inputPin;
  }

  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
    }
  }

  withdraw(amount, pin) {
    if (!this.#validatePin(pin)) {
      throw new Error("Invalid PIN");
    }
    if (amount > this.#balance) {
      throw new Error("Insufficient funds");
    }
    this.#balance -= amount;
    return amount;
  }

  get balance() {
    return this.#balance;
  }
}

const account = new BankAccount(1000, "1234");
console.log(account.balance); // 1000
account.deposit(500);
console.log(account.balance); // 1500

// ❌ Cannot access private fields
console.log(account.#balance); // SyntaxError
console.log(account.#pin); // SyntaxError
```

### Class Expression

```javascript
// Named class expression
const Person = class PersonClass {
  constructor(name) {
    this.name = name;
  }
};

// Anonymous class expression
const Animal = class {
  constructor(name) {
    this.name = name;
  }
};

// Immediately invoked class
const instance = new (class {
  constructor() {
    this.value = 42;
  }
})();
```

---

## 10. Modules (import/export)

### Named Exports

```javascript
// math.js - Named exports
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export class Calculator {
  // ...
}

// Hoặc export cuối file
const multiply = (a, b) => a * b;
const divide = (a, b) => a / b;

export { multiply, divide };

// Export với alias
export { multiply as mul, divide as div };
```

### Named Imports

```javascript
// Import specific exports
import { add, subtract } from "./math.js";

add(2, 3); // 5
subtract(5, 2); // 3

// Import với alias
import { add as sum, subtract as minus } from "./math.js";

sum(2, 3); // 5
minus(5, 2); // 3

// Import tất cả
import * as MathUtils from "./math.js";

MathUtils.add(2, 3);
MathUtils.PI;
```

### Default Export

```javascript
// user.js - Default export
export default class User {
  constructor(name) {
    this.name = name;
  }
}

// Hoặc
class User {
  constructor(name) {
    this.name = name;
  }
}
export default User;

// Hoặc anonymous
export default function(name) {
  return { name };
}

// Import default
import User from './user.js';
import MyUser from './user.js';  // Có thể đặt tên bất kỳ
```

### Mixed Exports

```javascript
// api.js
export default class API {
  // Main class
}

export const BASE_URL = "https://api.example.com";
export function fetchData() {}
export function postData() {}

// Import mixed
import API, { BASE_URL, fetchData } from "./api.js";

// Hoặc
import API, * as helpers from "./api.js";
helpers.BASE_URL;
helpers.fetchData();
```

### Re-exports

```javascript
// index.js - Barrel file
export { add, subtract } from "./math.js";
export { default as User } from "./user.js";
export * from "./utils.js";

// Re-export với rename
export { add as sum } from "./math.js";

// Re-export default as named
export { default as API } from "./api.js";

// Usage
import { add, User, API } from "./index.js";
```

### Dynamic Imports

```javascript
// Static import (top-level)
import { add } from "./math.js";

// Dynamic import (runtime)
async function loadModule() {
  const module = await import("./math.js");
  console.log(module.add(2, 3));
}

// Conditional import
if (condition) {
  const { feature } = await import("./feature.js");
  feature();
}

// Lazy loading
button.addEventListener("click", async () => {
  const { heavyFunction } = await import("./heavy-module.js");
  heavyFunction();
});

// With destructuring
const { default: User, helper } = await import("./user.js");
```

### Module Patterns

```javascript
// ┌─────────────────────────────────────────────────────────┐
// │                  MODULE PATTERNS                        │
// ├─────────────────────────────────────────────────────────┤
// │                                                         │
// │  1. Single Default Export                              │
// │  ─────────────────────────                             │
// │  // Component.js                                       │
// │  export default function Component() {}                │
// │                                                         │
// │  2. Multiple Named Exports                             │
// │  ─────────────────────────                             │
// │  // utils.js                                           │
// │  export const util1 = () => {};                        │
// │  export const util2 = () => {};                        │
// │                                                         │
// │  3. Mixed (Default + Named)                            │
// │  ─────────────────────────                             │
// │  // api.js                                             │
// │  export default class API {}                           │
// │  export const config = {};                             │
// │                                                         │
// │  4. Barrel (Re-export)                                 │
// │  ─────────────────────────                             │
// │  // index.js                                           │
// │  export * from './module1.js';                         │
// │  export * from './module2.js';                         │
// │                                                         │
// └─────────────────────────────────────────────────────────┘
```

---

## 11. Promises

### Promise Basics

```javascript
// Creating a Promise
const promise = new Promise((resolve, reject) => {
  // Async operation
  setTimeout(() => {
    const success = true;

    if (success) {
      resolve("Data loaded successfully");
    } else {
      reject(new Error("Failed to load data"));
    }
  }, 1000);
});

// Consuming a Promise
promise
  .then((result) => {
    console.log(result); // 'Data loaded successfully'
  })
  .catch((error) => {
    console.error(error);
  })
  .finally(() => {
    console.log("Operation completed");
  });
```

### Promise States

```javascript
// ┌─────────────────────────────────────────────────────────┐
// │                   PROMISE STATES                        │
// ├─────────────────────────────────────────────────────────┤
// │                                                         │
// │                    ┌─────────┐                         │
// │                    │ PENDING │                         │
// │                    └────┬────┘                         │
// │                         │                              │
// │           ┌─────────────┴─────────────┐               │
// │           │                           │               │
// │           ▼                           ▼               │
// │    ┌───────────┐              ┌───────────┐          │
// │    │ FULFILLED │              │ REJECTED  │          │
// │    │ (resolved)│              │           │          │
// │    └───────────┘              └───────────┘          │
// │         │                           │                │
// │         ▼                           ▼                │
// │      .then()                    .catch()             │
// │                                                       │
// │  • Pending: Initial state                            │
// │  • Fulfilled: Operation completed successfully       │
// │  • Rejected: Operation failed                        │
// │                                                       │
// └─────────────────────────────────────────────────────────┘
```

### Promise Chaining

```javascript
// Sequential operations
fetchUser(userId)
  .then((user) => fetchPosts(user.id))
  .then((posts) => fetchComments(posts[0].id))
  .then((comments) => {
    console.log(comments);
  })
  .catch((error) => {
    console.error("Error:", error);
  });

// Returning values
Promise.resolve(1)
  .then((x) => x + 1) // 2
  .then((x) => x * 2) // 4
  .then((x) => x + 3) // 7
  .then(console.log); // 7

// Returning promises
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

delay(1000)
  .then(() => {
    console.log("1 second passed");
    return delay(2000);
  })
  .then(() => {
    console.log("3 seconds total");
  });
```

### Promise Static Methods

```javascript
// Promise.all - Tất cả phải thành công
const promises = [fetch("/api/users"), fetch("/api/posts"), fetch("/api/comments")];

Promise.all(promises)
  .then(([users, posts, comments]) => {
    // Tất cả đã hoàn thành
  })
  .catch((error) => {
    // Một trong số đó failed
  });

// Promise.allSettled - Chờ tất cả, không quan tâm kết quả
Promise.allSettled(promises).then((results) => {
  results.forEach((result) => {
    if (result.status === "fulfilled") {
      console.log("Success:", result.value);
    } else {
      console.log("Failed:", result.reason);
    }
  });
});

// Promise.race - Lấy kết quả đầu tiên (thành công hoặc thất bại)
Promise.race([fetch("/api/data"), new Promise((_, reject) => setTimeout(() => reject(new Error("Timeout")), 5000))]);

// Promise.any - Lấy kết quả thành công đầu tiên
Promise.any([fetch("https://api1.example.com"), fetch("https://api2.example.com"), fetch("https://api3.example.com")])
  .then((firstSuccess) => console.log(firstSuccess))
  .catch((error) => console.log("All failed"));

// Promise.resolve / Promise.reject
Promise.resolve("value"); // Tạo fulfilled promise
Promise.reject(new Error("error")); // Tạo rejected promise
```

### Error Handling

```javascript
// Catch ở cuối chain
fetchData()
  .then(processData)
  .then(saveData)
  .catch((error) => {
    // Bắt lỗi từ bất kỳ step nào
    console.error(error);
  });

// Catch và recover
fetchData()
  .then((data) => {
    if (!data) throw new Error("No data");
    return data;
  })
  .catch((error) => {
    console.warn("Using default data");
    return defaultData; // Recover với default
  })
  .then((data) => {
    // Tiếp tục với data hoặc defaultData
  });

// Re-throwing errors
fetchData()
  .catch((error) => {
    logError(error);
    throw error; // Re-throw để catch tiếp theo xử lý
  })
  .catch((error) => {
    showUserError(error);
  });
```

### Practical Examples

```javascript
// Fetch wrapper
function fetchJSON(url) {
  return fetch(url).then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return response.json();
  });
}

// Retry logic
function fetchWithRetry(url, retries = 3) {
  return fetch(url).catch((error) => {
    if (retries > 0) {
      console.log(`Retrying... ${retries} attempts left`);
      return fetchWithRetry(url, retries - 1);
    }
    throw error;
  });
}

// Timeout wrapper
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error("Timeout")), ms);
  });
  return Promise.race([promise, timeout]);
}

// Sequential execution
async function sequential(tasks) {
  const results = [];
  for (const task of tasks) {
    results.push(await task());
  }
  return results;
}

// Parallel with limit
async function parallelLimit(tasks, limit) {
  const results = [];
  const executing = [];

  for (const task of tasks) {
    const p = task().then((result) => {
      executing.splice(executing.indexOf(p), 1);
      return result;
    });
    results.push(p);
    executing.push(p);

    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}
```

---

## 12. Symbol

### Symbol Basics

```javascript
// Creating Symbols
const sym1 = Symbol();
const sym2 = Symbol("description");
const sym3 = Symbol("description");

// Mỗi Symbol là unique
console.log(sym2 === sym3); // false (dù cùng description)

// Symbol không thể convert sang string/number
console.log(sym1.toString()); // "Symbol()"
console.log(sym2.description); // "description"

// ❌ Không thể
// sym1 + '';     // TypeError
// +sym1;         // TypeError
```

### Symbol as Object Keys

```javascript
// Unique property keys
const id = Symbol("id");
const name = Symbol("name");

const user = {
  [id]: 12345,
  [name]: "John",
  age: 25,
};

console.log(user[id]); // 12345
console.log(user[name]); // 'John'

// Symbols không xuất hiện trong for...in
for (let key in user) {
  console.log(key); // Chỉ 'age'
}

// Không xuất hiện trong Object.keys()
console.log(Object.keys(user)); // ['age']

// Lấy Symbol keys
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id), Symbol(name)]

// Lấy tất cả keys
console.log(Reflect.ownKeys(user)); // ['age', Symbol(id), Symbol(name)]
```

### Global Symbol Registry

```javascript
// Symbol.for() - Tạo/lấy Symbol từ global registry
const globalSym1 = Symbol.for("app.id");
const globalSym2 = Symbol.for("app.id");

console.log(globalSym1 === globalSym2); // true (cùng Symbol)

// Symbol.keyFor() - Lấy key của global Symbol
console.log(Symbol.keyFor(globalSym1)); // 'app.id'

// Regular Symbol không có trong registry
const localSym = Symbol("local");
console.log(Symbol.keyFor(localSym)); // undefined
```

### Well-known Symbols

```javascript
// Symbol.iterator - Định nghĩa iterator
const range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;

    return {
      next() {
        if (current <= last) {
          return { value: current++, done: false };
        }
        return { done: true };
      },
    };
  },
};

for (const num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}

// Symbol.toStringTag - Custom toString
class MyClass {
  get [Symbol.toStringTag]() {
    return "MyClass";
  }
}

console.log(Object.prototype.toString.call(new MyClass()));
// "[object MyClass]"

// Symbol.toPrimitive - Custom type conversion
const money = {
  amount: 100,
  currency: "USD",

  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.amount;
    if (hint === "string") return `${this.amount} ${this.currency}`;
    return this.amount;
  },
};

console.log(+money); // 100
console.log(`${money}`); // "100 USD"
console.log(money + 50); // 150
```

---

## 13. Iterators và Generators

### Iterators

```javascript
// Iterator protocol
const iterator = {
  current: 0,
  last: 5,

  next() {
    if (this.current <= this.last) {
      return { value: this.current++, done: false };
    }
    return { value: undefined, done: true };
  },
};

console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
// ...
console.log(iterator.next()); // { value: undefined, done: true }

// Iterable object
const iterable = {
  data: [1, 2, 3],

  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;

    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false };
        }
        return { done: true };
      },
    };
  },
};

for (const item of iterable) {
  console.log(item); // 1, 2, 3
}
```

### Generators

```javascript
// Generator function (function*)
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Generator với loop
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

for (const num of range(1, 5)) {
  console.log(num); // 1, 2, 3, 4, 5
}

// Spread với generator
const numbers = [...range(1, 5)]; // [1, 2, 3, 4, 5]

// Infinite generator
function* infiniteSequence() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

const infinite = infiniteSequence();
console.log(infinite.next().value); // 0
console.log(infinite.next().value); // 1
// ...continues forever
```

### Generator với yield\*

```javascript
// Delegate to another generator
function* gen1() {
  yield 1;
  yield 2;
}

function* gen2() {
  yield* gen1(); // Delegate
  yield 3;
  yield 4;
}

console.log([...gen2()]); // [1, 2, 3, 4]

// Flatten nested arrays
function* flatten(arr) {
  for (const item of arr) {
    if (Array.isArray(item)) {
      yield* flatten(item); // Recursive delegation
    } else {
      yield item;
    }
  }
}

const nested = [1, [2, [3, 4]], 5];
console.log([...flatten(nested)]); // [1, 2, 3, 4, 5]
```

### Two-way Communication

```javascript
// Passing values to generator
function* conversation() {
  const name = yield "What is your name?";
  const age = yield `Hello ${name}! How old are you?`;
  return `${name} is ${age} years old`;
}

const talk = conversation();
console.log(talk.next().value); // "What is your name?"
console.log(talk.next("John").value); // "Hello John! How old are you?"
console.log(talk.next(25).value); // "John is 25 years old"

// Error handling
function* errorGenerator() {
  try {
    yield 1;
    yield 2;
  } catch (e) {
    console.log("Caught:", e);
  }
}

const gen = errorGenerator();
gen.next();
gen.throw(new Error("Something went wrong")); // "Caught: Error: Something went wrong"
```

### Practical Examples

```javascript
// ID generator
function* idGenerator(prefix = "id") {
  let id = 1;
  while (true) {
    yield `${prefix}_${id++}`;
  }
}

const userIdGen = idGenerator("user");
console.log(userIdGen.next().value); // "user_1"
console.log(userIdGen.next().value); // "user_2"

// Pagination
function* paginate(items, pageSize) {
  for (let i = 0; i < items.length; i += pageSize) {
    yield items.slice(i, i + pageSize);
  }
}

const items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const pages = paginate(items, 3);

console.log(pages.next().value); // [1, 2, 3]
console.log(pages.next().value); // [4, 5, 6]
console.log(pages.next().value); // [7, 8, 9]
console.log(pages.next().value); // [10]

// Async-like flow (before async/await)
function* fetchUserFlow() {
  const user = yield fetch("/api/user");
  const posts = yield fetch(`/api/posts/${user.id}`);
  return { user, posts };
}
```

---

## 14. Map và Set

### Map

```javascript
// Creating Map
const map = new Map();

// Set values
map.set("name", "John");
map.set("age", 25);
map.set({ id: 1 }, "object key");

// Get values
console.log(map.get("name")); // 'John'
console.log(map.get("age")); // 25

// Check existence
console.log(map.has("name")); // true

// Size
console.log(map.size); // 3

// Delete
map.delete("age");

// Clear all
map.clear();

// Initialize with entries
const map2 = new Map([
  ["key1", "value1"],
  ["key2", "value2"],
  ["key3", "value3"],
]);

// Iteration
for (const [key, value] of map2) {
  console.log(key, value);
}

map2.forEach((value, key) => {
  console.log(key, value);
});

// Get keys, values, entries
console.log([...map2.keys()]); // ['key1', 'key2', 'key3']
console.log([...map2.values()]); // ['value1', 'value2', 'value3']
console.log([...map2.entries()]); // [['key1', 'value1'], ...]
```

### Map vs Object

```javascript
// ┌─────────────────────────────────────────────────────────┐
// │                   MAP vs OBJECT                         │
// ├─────────────────────────────────────────────────────────┤
// │                                                         │
// │  Feature          │ Map              │ Object           │
// │  ─────────────────┼──────────────────┼─────────────────│
// │  Key types        │ Any type         │ String/Symbol   │
// │  Key order        │ Insertion order  │ Not guaranteed  │
// │  Size             │ map.size         │ Object.keys().length │
// │  Iteration        │ Directly iterable│ Need Object.keys() │
// │  Performance      │ Better for       │ Better for      │
// │                   │ frequent add/del │ static keys     │
// │  Default keys     │ None             │ Has prototype   │
// │  Serialization    │ No native JSON   │ JSON.stringify  │
// │                                                         │
// └─────────────────────────────────────────────────────────┘

// Object as key (only Map can do this)
const objKey = { id: 1 };
const map = new Map();
map.set(objKey, "value");
console.log(map.get(objKey)); // 'value'

// Object cannot use object as key
const obj = {};
obj[objKey] = "value";
console.log(obj["[object Object]"]); // 'value' (key converted to string)
```

### WeakMap

```javascript
// WeakMap - Keys must be objects, garbage collected
const weakMap = new WeakMap();

let obj = { name: 'John' };
weakMap.set(obj, 'metadata');

console.log(weakMap.get(obj)); // 'metadata'

obj = null; // Object can be garbage collected
// weakMap entry is also removed

// Use case: Private data
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    privateData.set(this, { name, age });
  }

  getName() {
    return privateData.get(this).name;
  }
}

// Use case: Caching
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }

  const result = /* expensive computation */;
  cache.set(obj, result);
  return result;
}
```

### Set

```javascript
// Creating Set
const set = new Set();

// Add values
set.add(1);
set.add(2);
set.add(2); // Duplicate ignored
set.add("hello");

console.log(set.size); // 3

// Check existence
console.log(set.has(1)); // true

// Delete
set.delete(1);

// Clear
set.clear();

// Initialize with values
const set2 = new Set([1, 2, 3, 3, 4, 4, 5]);
console.log([...set2]); // [1, 2, 3, 4, 5] - duplicates removed

// Iteration
for (const value of set2) {
  console.log(value);
}

set2.forEach((value) => console.log(value));

// Convert to array
const arr = [...set2];
const arr2 = Array.from(set2);
```

### Set Operations

```javascript
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// Union (A ∪ B)
const union = new Set([...setA, ...setB]);
// Set {1, 2, 3, 4, 5, 6}

// Intersection (A ∩ B)
const intersection = new Set([...setA].filter((x) => setB.has(x)));
// Set {3, 4}

// Difference (A - B)
const difference = new Set([...setA].filter((x) => !setB.has(x)));
// Set {1, 2}

// Symmetric Difference (A △ B)
const symDiff = new Set([...[...setA].filter((x) => !setB.has(x)), ...[...setB].filter((x) => !setA.has(x))]);
// Set {1, 2, 5, 6}

// Subset check
const isSubset = [...setA].every((x) => setB.has(x));
```

### WeakSet

```javascript
// WeakSet - Only objects, garbage collected
const weakSet = new WeakSet();

let obj = { name: "John" };
weakSet.add(obj);

console.log(weakSet.has(obj)); // true

obj = null; // Object can be garbage collected

// Use case: Tracking visited objects
const visited = new WeakSet();

function processNode(node) {
  if (visited.has(node)) {
    return; // Already processed
  }

  visited.add(node);
  // Process node...
}
```

---

## 15. for...of Loop

### Cú pháp cơ bản

```javascript
// for...of - Iterate over values
const arr = ["a", "b", "c"];

for (const value of arr) {
  console.log(value); // 'a', 'b', 'c'
}

// for...in - Iterate over keys/indices
for (const index in arr) {
  console.log(index); // '0', '1', '2'
}
```

### for...of vs for...in vs forEach

```javascript
// ┌─────────────────────────────────────────────────────────┐
// │           for...of vs for...in vs forEach               │
// ├─────────────────────────────────────────────────────────┤
// │                                                         │
// │  for...of                                               │
// │  • Iterates over VALUES                                │
// │  • Works with iterables (Array, String, Map, Set)      │
// │  • Can use break, continue, return                     │
// │  • Cannot iterate plain objects                        │
// │                                                         │
// │  for...in                                               │
// │  • Iterates over KEYS (enumerable properties)          │
// │  • Works with objects                                  │
// │  • Includes inherited properties                       │
// │  • Not recommended for arrays                          │
// │                                                         │
// │  forEach                                                │
// │  • Array method only                                   │
// │  • Cannot break or return early                        │
// │  • Cleaner syntax for simple iterations                │
// │                                                         │
// └─────────────────────────────────────────────────────────┘

const arr = ["a", "b", "c"];

// for...of
for (const value of arr) {
  if (value === "b") break; // ✅ Can break
  console.log(value);
}

// forEach
arr.forEach((value) => {
  if (value === "b") return; // ❌ Only skips current iteration
  console.log(value);
});
```

### Iterating Different Types

```javascript
// Arrays
for (const item of [1, 2, 3]) {
  console.log(item);
}

// Strings
for (const char of "hello") {
  console.log(char); // 'h', 'e', 'l', 'l', 'o'
}

// Maps
const map = new Map([
  ["a", 1],
  ["b", 2],
]);
for (const [key, value] of map) {
  console.log(key, value);
}

// Sets
const set = new Set([1, 2, 3]);
for (const value of set) {
  console.log(value);
}

// NodeList
const divs = document.querySelectorAll("div");
for (const div of divs) {
  console.log(div);
}

// arguments object
function example() {
  for (const arg of arguments) {
    console.log(arg);
  }
}

// Generators
function* gen() {
  yield 1;
  yield 2;
}
for (const value of gen()) {
  console.log(value);
}
```

### With Destructuring

```javascript
// Array of arrays
const pairs = [
  [1, "one"],
  [2, "two"],
  [3, "three"],
];
for (const [num, word] of pairs) {
  console.log(`${num} = ${word}`);
}

// Array of objects
const users = [
  { name: "John", age: 25 },
  { name: "Jane", age: 30 },
];
for (const { name, age } of users) {
  console.log(`${name} is ${age}`);
}

// Map entries
const map = new Map([
  ["a", 1],
  ["b", 2],
]);
for (const [key, value] of map) {
  console.log(key, value);
}

// Object.entries()
const obj = { a: 1, b: 2, c: 3 };
for (const [key, value] of Object.entries(obj)) {
  console.log(key, value);
}
```

---

## 16. Array Methods mới

### Array.from()

```javascript
// Convert array-like to array
const nodeList = document.querySelectorAll("div");
const arr = Array.from(nodeList);

// String to array
Array.from("hello"); // ['h', 'e', 'l', 'l', 'o']

// Set to array
Array.from(new Set([1, 2, 3])); // [1, 2, 3]

// Map function
Array.from([1, 2, 3], (x) => x * 2); // [2, 4, 6]

// Generate sequence
Array.from({ length: 5 }, (_, i) => i); // [0, 1, 2, 3, 4]
Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]

// Create 2D array
Array.from({ length: 3 }, () => Array(3).fill(0));
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

### Array.of()

```javascript
// Array.of() - Tạo array từ arguments
Array.of(1, 2, 3); // [1, 2, 3]
Array.of(7); // [7]

// So sánh với Array()
Array(7); // [empty × 7] (7 empty slots)
Array(1, 2, 3); // [1, 2, 3]
```

### find() và findIndex()

```javascript
const users = [
  { id: 1, name: "John", age: 25 },
  { id: 2, name: "Jane", age: 30 },
  { id: 3, name: "Bob", age: 35 },
];

// find() - Trả về element đầu tiên match
const user = users.find((u) => u.age > 25);
// { id: 2, name: 'Jane', age: 30 }

const notFound = users.find((u) => u.age > 100);
// undefined

// findIndex() - Trả về index đầu tiên match
const index = users.findIndex((u) => u.name === "Jane");
// 1

const notFoundIndex = users.findIndex((u) => u.name === "Unknown");
// -1
```

### includes()

```javascript
const arr = [1, 2, 3, NaN];

// includes() - Kiểm tra element có tồn tại
arr.includes(2); // true
arr.includes(4); // false
arr.includes(NaN); // true ✅

// So sánh với indexOf()
arr.indexOf(2) !== -1; // true
arr.indexOf(NaN) !== -1; // false ❌ (indexOf không tìm được NaN)

// From index
arr.includes(2, 2); // false (tìm từ index 2)
```

### fill()

```javascript
// fill(value, start, end)
const arr = [1, 2, 3, 4, 5];

arr.fill(0); // [0, 0, 0, 0, 0]
arr.fill(9, 2); // [0, 0, 9, 9, 9]
arr.fill(7, 1, 3); // [0, 7, 7, 9, 9]

// Tạo array với giá trị mặc định
new Array(5).fill(0); // [0, 0, 0, 0, 0]

// ⚠️ Cẩn thận với objects
const arr2 = new Array(3).fill({});
arr2[0].name = "John";
console.log(arr2); // [{name: 'John'}, {name: 'John'}, {name: 'John'}]
// Tất cả reference cùng 1 object!

// ✅ Dùng Array.from() thay thế
const arr3 = Array.from({ length: 3 }, () => ({}));
```

### copyWithin()

```javascript
// copyWithin(target, start, end)
const arr = [1, 2, 3, 4, 5];

arr.copyWithin(0, 3); // [4, 5, 3, 4, 5]
arr.copyWithin(1, 3); // [4, 4, 5, 4, 5]

const arr2 = [1, 2, 3, 4, 5];
arr2.copyWithin(0, 3, 4); // [4, 2, 3, 4, 5]
```

### entries(), keys(), values()

```javascript
const arr = ["a", "b", "c"];

// entries() - [index, value] pairs
for (const [index, value] of arr.entries()) {
  console.log(index, value);
}
// 0 'a'
// 1 'b'
// 2 'c'

// keys() - indices
for (const index of arr.keys()) {
  console.log(index);
}
// 0, 1, 2

// values() - values
for (const value of arr.values()) {
  console.log(value);
}
// 'a', 'b', 'c'
```

### flat() và flatMap() (ES2019)

```javascript
// flat() - Flatten nested arrays
const nested = [1, [2, [3, [4]]]];

nested.flat(); // [1, 2, [3, [4]]]
nested.flat(2); // [1, 2, 3, [4]]
nested.flat(Infinity); // [1, 2, 3, 4]

// flatMap() - map + flat(1)
const sentences = ["Hello world", "How are you"];

sentences.flatMap((s) => s.split(" "));
// ['Hello', 'world', 'How', 'are', 'you']

// Equivalent to
sentences.map((s) => s.split(" ")).flat();
```

---

## 17. String Methods mới

### Template Literals (đã covered ở section 4)

### startsWith(), endsWith(), includes()

```javascript
const str = "Hello, World!";

// startsWith()
str.startsWith("Hello"); // true
str.startsWith("World", 7); // true (from position 7)

// endsWith()
str.endsWith("!"); // true
str.endsWith("World", 12); // true (check first 12 chars)

// includes()
str.includes("World"); // true
str.includes("world"); // false (case-sensitive)
str.includes("o", 5); // true (from position 5)
```

### repeat()

```javascript
"abc".repeat(3); // 'abcabcabc'
"abc".repeat(0); // ''
"abc".repeat(1.5); // 'abc' (truncated to 1)

// Practical use
const indent = "  ".repeat(depth);
const separator = "-".repeat(20);
```

### padStart() và padEnd() (ES2017)

```javascript
// padStart(targetLength, padString)
"5".padStart(3, "0"); // '005'
"42".padStart(5, "0"); // '00042'
"hello".padStart(10); // '     hello'

// padEnd(targetLength, padString)
"5".padEnd(3, "0"); // '500'
"hello".padEnd(10, "."); // 'hello.....'

// Practical uses
const price = "9.99".padStart(10); // '      9.99'
const binary = (255).toString(2).padStart(8, "0"); // '11111111'
```

### trimStart() và trimEnd() (ES2019)

```javascript
const str = "   Hello World   ";

str.trim(); // 'Hello World'
str.trimStart(); // 'Hello World   '
str.trimEnd(); // '   Hello World'

// Aliases
str.trimLeft(); // Same as trimStart()
str.trimRight(); // Same as trimEnd()
```

### String.raw

```javascript
// Raw string (escape sequences not processed)
String.raw`Hello\nWorld`; // 'Hello\\nWorld'

// Useful for regex, file paths
const path = String.raw`C:\Users\John\Documents`;
const regex = new RegExp(String.raw`\d+\.\d+`);
```

---

## 18. Object Methods mới

### Object.assign()

```javascript
// Shallow copy
const target = { a: 1 };
const source = { b: 2, c: 3 };
const result = Object.assign(target, source);
// target = { a: 1, b: 2, c: 3 }

// Multiple sources
Object.assign({}, obj1, obj2, obj3);

// Clone object
const clone = Object.assign({}, original);

// Merge with override
const defaults = { theme: "light", lang: "en" };
const userPrefs = { theme: "dark" };
const settings = Object.assign({}, defaults, userPrefs);
// { theme: 'dark', lang: 'en' }

// ⚠️ Shallow copy only
const obj = { nested: { value: 1 } };
const copy = Object.assign({}, obj);
copy.nested.value = 2;
console.log(obj.nested.value); // 2 (affected!)
```

### Object.is()

```javascript
// Object.is() - Strict equality với special cases
Object.is(25, 25); // true
Object.is("foo", "foo"); // true
Object.is(NaN, NaN); // true ✅ (=== returns false)
Object.is(0, -0); // false ✅ (=== returns true)

// So sánh với ===
NaN === NaN; // false
0 === -0; // true
```

### Object.keys(), values(), entries()

```javascript
const obj = { a: 1, b: 2, c: 3 };

// Object.keys() - ES5
Object.keys(obj); // ['a', 'b', 'c']

// Object.values() - ES2017
Object.values(obj); // [1, 2, 3]

// Object.entries() - ES2017
Object.entries(obj); // [['a', 1], ['b', 2], ['c', 3]]

// Object.fromEntries() - ES2019
const entries = [
  ["a", 1],
  ["b", 2],
];
Object.fromEntries(entries); // { a: 1, b: 2 }

// Convert Map to Object
const map = new Map([
  ["a", 1],
  ["b", 2],
]);
Object.fromEntries(map); // { a: 1, b: 2 }
```

### Object.getOwnPropertyDescriptors() (ES2017)

```javascript
const obj = {
  get foo() {
    return "foo";
  },
};

// Get all property descriptors
Object.getOwnPropertyDescriptors(obj);
// {
//   foo: {
//     get: [Function: get foo],
//     set: undefined,
//     enumerable: true,
//     configurable: true
//   }
// }

// Proper cloning with getters/setters
const clone = Object.defineProperties({}, Object.getOwnPropertyDescriptors(obj));
```

---

## 19. Number và Math mới

### Number Methods

```javascript
// Number.isFinite()
Number.isFinite(42); // true
Number.isFinite(Infinity); // false
Number.isFinite(NaN); // false
Number.isFinite("42"); // false (global isFinite('42') = true)

// Number.isNaN()
Number.isNaN(NaN); // true
Number.isNaN("NaN"); // false (global isNaN('NaN') = true)

// Number.isInteger()
Number.isInteger(42); // true
Number.isInteger(42.0); // true
Number.isInteger(42.5); // false

// Number.isSafeInteger()
Number.isSafeInteger(42); // true
Number.isSafeInteger(Math.pow(2, 53)); // false
Number.isSafeInteger(Math.pow(2, 53) - 1); // true

// Number.parseFloat(), Number.parseInt()
Number.parseFloat("3.14"); // 3.14
Number.parseInt("42px"); // 42

// Constants
Number.EPSILON; // ~2.22e-16
Number.MAX_SAFE_INTEGER; // 9007199254740991
Number.MIN_SAFE_INTEGER; // -9007199254740991
```

### Math Methods

```javascript
// Math.sign() - Dấu của số
Math.sign(5); // 1
Math.sign(-5); // -1
Math.sign(0); // 0

// Math.trunc() - Bỏ phần thập phân
Math.trunc(4.7); // 4
Math.trunc(-4.7); // -4

// Math.cbrt() - Căn bậc 3
Math.cbrt(8); // 2
Math.cbrt(27); // 3

// Math.log2(), Math.log10()
Math.log2(8); // 3
Math.log10(100); // 2

// Math.hypot() - Căn bậc 2 của tổng bình phương
Math.hypot(3, 4); // 5
Math.hypot(3, 4, 5); // 7.07...

// Math.expm1(), Math.log1p()
Math.expm1(1); // e^1 - 1 = 1.718...
Math.log1p(1); // ln(1 + 1) = 0.693...

// Math.clz32() - Count leading zeros (32-bit)
Math.clz32(1); // 31
Math.clz32(2); // 30

// Math.imul() - 32-bit integer multiplication
Math.imul(2, 4); // 8

// Math.fround() - Nearest 32-bit float
Math.fround(1.5); // 1.5
Math.fround(1.337); // 1.3370000123977661
```

---

## 20. Proxy và Reflect

### Proxy Basics

```javascript
// Proxy - Intercept object operations
const target = { name: "John", age: 25 };

const handler = {
  get(target, prop) {
    console.log(`Getting ${prop}`);
    return target[prop];
  },

  set(target, prop, value) {
    console.log(`Setting ${prop} to ${value}`);
    target[prop] = value;
    return true;
  },
};

const proxy = new Proxy(target, handler);

proxy.name; // "Getting name" → "John"
proxy.age = 30; // "Setting age to 30"
```

### Proxy Traps

```javascript
const handler = {
  // Property access
  get(target, prop, receiver) {},
  set(target, prop, value, receiver) {},
  has(target, prop) {}, // 'prop' in proxy
  deleteProperty(target, prop) {}, // delete proxy.prop

  // Object operations
  getOwnPropertyDescriptor(target, prop) {},
  defineProperty(target, prop, descriptor) {},
  ownKeys(target) {}, // Object.keys(), for...in

  // Function operations
  apply(target, thisArg, args) {}, // proxy()
  construct(target, args) {}, // new proxy()

  // Prototype
  getPrototypeOf(target) {},
  setPrototypeOf(target, proto) {},

  // Extensibility
  isExtensible(target) {},
  preventExtensions(target) {},
};
```

### Practical Proxy Examples

```javascript
// Validation
const validator = {
  set(target, prop, value) {
    if (prop === "age") {
      if (typeof value !== "number") {
        throw new TypeError("Age must be a number");
      }
      if (value < 0 || value > 150) {
        throw new RangeError("Age must be between 0 and 150");
      }
    }
    target[prop] = value;
    return true;
  },
};

const person = new Proxy({}, validator);
person.age = 25; // OK
person.age = -5; // RangeError
person.age = "old"; // TypeError

// Default values
const withDefaults = (target, defaults) => {
  return new Proxy(target, {
    get(obj, prop) {
      return prop in obj ? obj[prop] : defaults[prop];
    },
  });
};

const config = withDefaults({}, { theme: "dark", lang: "en" });
console.log(config.theme); // 'dark'
console.log(config.lang); // 'en'

// Negative array indices
const negativeArray = (arr) => {
  return new Proxy(arr, {
    get(target, prop) {
      const index = Number(prop);
      if (index < 0) {
        return target[target.length + index];
      }
      return target[prop];
    },
  });
};

const arr = negativeArray([1, 2, 3, 4, 5]);
console.log(arr[-1]); // 5
console.log(arr[-2]); // 4
```

### Reflect API

```javascript
// Reflect - Companion to Proxy
const obj = { name: "John" };

// Property operations
Reflect.get(obj, "name"); // 'John'
Reflect.set(obj, "age", 25); // true
Reflect.has(obj, "name"); // true
Reflect.deleteProperty(obj, "age"); // true

// Object operations
Reflect.ownKeys(obj); // ['name']
Reflect.defineProperty(obj, "id", { value: 1 });

// Function operations
Reflect.apply(Math.max, null, [1, 2, 3]); // 3
Reflect.construct(Date, [2024, 0, 1]); // Date object

// Use in Proxy handlers
const handler = {
  get(target, prop, receiver) {
    console.log(`Accessing ${prop}`);
    return Reflect.get(target, prop, receiver);
  },

  set(target, prop, value, receiver) {
    console.log(`Setting ${prop}`);
    return Reflect.set(target, prop, value, receiver);
  },
};
```

---

## 📚 Tổng kết ES6

### Cheat Sheet

```javascript
// ┌─────────────────────────────────────────────────────────┐
// │                   ES6 CHEAT SHEET                       │
// ├─────────────────────────────────────────────────────────┤
// │                                                         │
// │  Variables                                              │
// │  let x = 1;              // Block-scoped, reassignable │
// │  const y = 2;            // Block-scoped, constant     │
// │                                                         │
// │  Arrow Functions                                        │
// │  const fn = (a, b) => a + b;                           │
// │  const fn = x => x * 2;                                │
// │                                                         │
// │  Template Literals                                      │
// │  `Hello ${name}!`                                      │
// │  `Multi                                                │
// │   line`                                                │
// │                                                         │
// │  Destructuring                                          │
// │  const [a, b] = [1, 2];                                │
// │  const { name, age } = person;                         │
// │                                                         │
// │  Spread/Rest                                            │
// │  const arr = [...arr1, ...arr2];                       │
// │  const obj = { ...obj1, ...obj2 };                     │
// │  function fn(...args) {}                               │
// │                                                         │
// │  Default Parameters                                     │
// │  function fn(x = 10) {}                                │
// │                                                         │
// │  Classes                                                │
// │  class Dog extends Animal {                            │
// │    constructor(name) { super(name); }                  │
// │  }                                                     │
// │                                                         │
// │  Modules                                                │
// │  export const x = 1;                                   │
// │  export default class {}                               │
// │  import { x } from './module';                         │
// │  import Default from './module';                       │
// │                                                         │
// │  Promises                                               │
// │  new Promise((resolve, reject) => {})                  │
// │  promise.then().catch().finally()                      │
// │  Promise.all([p1, p2])                                 │
// │                                                         │
// │  Collections                                            │
// │  new Map(), new Set()                                  │
// │  new WeakMap(), new WeakSet()                          │
// │                                                         │
// └─────────────────────────────────────────────────────────┘
```

### Browser Support

ES6 được hỗ trợ bởi tất cả modern browsers. Với older browsers, sử dụng:

- **Babel** - Transpiler ES6+ → ES5
- **Polyfills** - Thêm features thiếu

---

## 📖 Tài liệu tham khảo

- [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [ECMAScript 6 Features](http://es6-features.org/)
- [Exploring ES6](https://exploringjs.com/es6/)
- [JavaScript.info](https://javascript.info/)
- [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)
