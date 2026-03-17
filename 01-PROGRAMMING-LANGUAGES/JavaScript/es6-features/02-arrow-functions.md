# Arrow Functions

> Quay lại [Mục lục](./README.md) | Trước: [let & const](./01-let-const.md)

## 📌 Tổng quan

Arrow Functions là một trong những tính năng được sử dụng nhiều nhất của ES6. Chúng cung cấp cú pháp ngắn gọn hơn để viết function expressions và quan trọng nhất — **không có `this` riêng** (lexical `this`).

---

## 1. Cú pháp cơ bản

### 1.1. Từ Regular Function đến Arrow Function

```javascript
// ES5 - Function expression
const add = function(a, b) {
  return a + b;
};

// ES6 - Arrow function (full syntax)
const add = (a, b) => {
  return a + b;
};

// ES6 - Arrow function (concise body - implicit return)
const add = (a, b) => a + b;
```

### 1.2. Các biến thể cú pháp

```javascript
// ═══ ZERO PARAMETERS ═══
const greet = () => "Hello!";
const getTime = () => Date.now();

// ═══ ONE PARAMETER (không cần parentheses) ═══
const double = x => x * 2;
const isPositive = n => n > 0;

// Nhưng nên dùng () cho consistency và readability
const double = (x) => x * 2;

// ═══ MULTIPLE PARAMETERS ═══
const add = (a, b) => a + b;
const fullName = (first, last) => `${first} ${last}`;

// ═══ FUNCTION BODY (nhiều statements) ═══
const calculate = (a, b) => {
  const sum = a + b;
  const product = a * b;
  return { sum, product };
};

// ═══ RETURN OBJECT LITERAL ═══
// ⚠️ Cần wrap trong () vì {} bị hiểu nhầm là function body
const createUser = (name, age) => ({ name, age });

// ❌ Sai - {} được hiểu là function body
const wrong = (name) => { name: name }; // undefined (label syntax)

// ✅ Đúng - () bọc object literal
const right = (name) => ({ name: name });

// ═══ RETURN ARRAY ═══
const pair = (a, b) => [a, b];

// ═══ IIFE (Immediately Invoked) ═══
const result = ((x) => x * 2)(5); // 10
```

### 1.3. Arrow Function với Rest Parameters

```javascript
const sum = (...numbers) => numbers.reduce((a, b) => a + b, 0);
sum(1, 2, 3, 4, 5); // 15

const first = (head, ...tail) => head;
first(1, 2, 3); // 1
```

---

## 2. `this` trong Arrow Functions (Lexical `this`)

### 2.1. Vấn đề với Regular Function

Đây là lý do chính arrow functions được tạo ra.

```javascript
// ❌ Problem: this bị mất trong callback
const timer = {
  seconds: 0,
  start: function() {
    // this ở đây = timer object ✅
    setInterval(function() {
      // this ở đây = window/undefined ❌
      this.seconds++; // ❌ NaN hoặc Error
      console.log(this.seconds);
    }, 1000);
  }
};

// ❌ Problem trong event handler
const counter = {
  count: 0,
  increment: function() {
    document.getElementById("btn").addEventListener("click", function() {
      // this ở đây = button element, KHÔNG phải counter
      this.count++; // ❌ Thao tác trên button.count
    });
  }
};
```

### 2.2. Các cách fix trước ES6

```javascript
// Solution 1: that/self = this
const timer = {
  seconds: 0,
  start: function() {
    const that = this; // Lưu reference
    setInterval(function() {
      that.seconds++; // ✅ Dùng that thay this
    }, 1000);
  }
};

// Solution 2: .bind(this)
const timer = {
  seconds: 0,
  start: function() {
    setInterval(function() {
      this.seconds++; // ✅
    }.bind(this), 1000); // Bind this vào function
  }
};

// Solution 3: Closure qua argument (forEach, map,...)
const obj = {
  values: [1, 2, 3],
  print: function() {
    this.values.forEach(function(v) {
      console.log(this, v);
    }, this); // ✅ thisArg parameter
  }
};
```

### 2.3. Arrow Function Solution (ES6)

```javascript
// ✅ Arrow function kế thừa this từ enclosing scope (lexical this)
const timer = {
  seconds: 0,
  start: function() {
    setInterval(() => {
      this.seconds++; // ✅ this = timer object
      console.log(this.seconds);
    }, 1000);
  }
};

// ✅ Event handler
const counter = {
  count: 0,
  init() {
    document.getElementById("btn").addEventListener("click", () => {
      this.count++; // ✅ this = counter object
      console.log(`Count: ${this.count}`);
    });
  }
};

// ✅ Nested callbacks
const app = {
  data: [],
  fetchAndProcess() {
    fetch("/api/data")
      .then(response => response.json())  // this vẫn = app
      .then(data => {
        this.data = data;                  // ✅ this = app
        return this.data.map(item => ({    // ✅ this = app
          ...item,
          processed: true
        }));
      })
      .catch(error => {
        console.error(this.constructor.name, error); // ✅ this = app
      });
  }
};
```

### 2.4. Lexical `this` chi tiết

```javascript
// Arrow function KHÔNG có this riêng
// Nó "capture" this từ scope bao ngoài tại thời điểm ĐỊNH NGHĨA

const obj = {
  name: "Object",
  
  // Regular method - this = obj khi gọi obj.regular()
  regular: function() {
    console.log("regular:", this.name); // "Object"
    
    // Arrow bên trong - this = this của regular = obj
    const arrow = () => {
      console.log("arrow inside regular:", this.name); // "Object" ✅
    };
    arrow();
  },
  
  // Arrow method - this = this tại thời điểm obj được define
  // Nếu ở global scope thì this = window/undefined
  arrowMethod: () => {
    console.log("arrowMethod:", this.name); // undefined ❌
  }
};

obj.regular();      // "Object" → "Object"
obj.arrowMethod();  // undefined
```

---

## 3. So sánh Regular Function vs Arrow Function

### 3.1. Bảng so sánh

```
┌───────────────────────────────────────────────────────────┐
│         REGULAR FUNCTION vs ARROW FUNCTION                │
├───────────────────┬───────────────┬───────────────────────┤
│ Feature           │ Regular       │ Arrow                 │
├───────────────────┼───────────────┼───────────────────────┤
│ this              │ Dynamic       │ Lexical (inherited)   │
│ arguments         │ ✅ Có         │ ❌ Không              │
│ new (constructor) │ ✅ Có         │ ❌ TypeError          │
│ prototype         │ ✅ Có         │ ❌ Không              │
│ super             │ ❌ Không      │ ✅ Có (từ class)      │
│ new.target        │ ✅ Có         │ ❌ Không              │
│ yield (generator) │ ✅ Có         │ ❌ Không              │
│ Hoisting          │ ✅ (declared) │ ❌ (expression)       │
│ Syntax            │ Dài hơn       │ Ngắn gọn             │
│ Implicit return   │ ❌ Không      │ ✅ Có (concise body)  │
└───────────────────┴───────────────┴───────────────────────┘
```

### 3.2. `arguments` object

```javascript
// Regular function - có arguments object
function regularSum() {
  console.log(arguments); // Arguments [1, 2, 3]
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}
regularSum(1, 2, 3); // 6

// Arrow function - KHÔNG có arguments
const arrowSum = () => {
  console.log(arguments); // ❌ ReferenceError hoặc arguments từ outer scope
};

// ✅ Thay thế: dùng rest parameters
const arrowSum = (...args) => {
  console.log(args); // [1, 2, 3]
  return args.reduce((a, b) => a + b, 0);
};
arrowSum(1, 2, 3); // 6
```

### 3.3. Constructor

```javascript
// Regular function - có thể dùng làm constructor
function Person(name) {
  this.name = name;
}
const john = new Person("John"); // ✅ OK

// Arrow function - KHÔNG thể dùng làm constructor
const Person = (name) => {
  this.name = name;
};
const john = new Person("John"); // ❌ TypeError: Person is not a constructor

// ✅ Dùng class thay thế
class Person {
  constructor(name) {
    this.name = name;
  }
}
```

---

## 4. Khi nào KHÔNG dùng Arrow Functions

### 4.1. Object Methods

```javascript
// ❌ Arrow function làm object method
const calculator = {
  value: 0,
  add: (n) => {
    this.value += n; // ❌ this = window/undefined, KHÔNG phải calculator
    return this;
  },
};

// ✅ Dùng method shorthand
const calculator = {
  value: 0,
  add(n) {
    this.value += n; // ✅ this = calculator
    return this;
  },
  subtract(n) {
    this.value -= n;
    return this;
  },
};

calculator.add(5).subtract(2); // value = 3
```

### 4.2. Event Handlers (khi cần `this` = element)

```javascript
// ❌ Arrow — this không phải button
button.addEventListener("click", () => {
  this.classList.toggle("active"); // ❌ this = enclosing scope
  this.textContent = "Clicked!";
});

// ✅ Regular function — this = button element
button.addEventListener("click", function() {
  this.classList.toggle("active"); // ✅ this = button
  this.textContent = "Clicked!";
});

// ✅ Hoặc dùng event.target / event.currentTarget
button.addEventListener("click", (e) => {
  e.currentTarget.classList.toggle("active"); // ✅
  e.currentTarget.textContent = "Clicked!";
});
```

### 4.3. Prototype Methods

```javascript
// ❌ Arrow function trên prototype
function Person(name) {
  this.name = name;
}

Person.prototype.greet = () => {
  return `Hello, I'm ${this.name}`; // ❌ this = window/undefined
};

// ✅ Regular function trên prototype  
Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`; // ✅ this = instance
};
```

### 4.4. Dynamic Context Required

```javascript
// ❌ Vue.js methods/computed (dùng arrow)
export default {
  data() {
    return { count: 0 };
  },
  methods: {
    increment: () => {
      this.count++; // ❌ this không phải Vue instance
    }
  }
};

// ✅ Regular function
export default {
  methods: {
    increment() {
      this.count++; // ✅ this = Vue instance
    }
  }
};
```

---

## 5. Arrow Functions với Array Methods

Arrow functions tỏa sáng nhất khi kết hợp với array methods.

```javascript
const users = [
  { name: "Alice", age: 25, role: "admin" },
  { name: "Bob", age: 30, role: "user" },
  { name: "Charlie", age: 35, role: "admin" },
  { name: "Diana", age: 28, role: "user" },
];

// ═══ map — transform ═══
const names = users.map(u => u.name);
// ["Alice", "Bob", "Charlie", "Diana"]

const greetings = users.map(u => `Hello, ${u.name}!`);
// ["Hello, Alice!", "Hello, Bob!", ...]

// ═══ filter — lọc ═══
const admins = users.filter(u => u.role === "admin");
const over30 = users.filter(u => u.age > 30);

// ═══ find — tìm phần tử đầu tiên ═══
const bob = users.find(u => u.name === "Bob");

// ═══ reduce — tích lũy ═══
const totalAge = users.reduce((sum, u) => sum + u.age, 0);
// 118

const byRole = users.reduce((groups, u) => {
  (groups[u.role] = groups[u.role] || []).push(u);
  return groups;
}, {});
// { admin: [...], user: [...] }

// ═══ every / some ═══
const allAdults = users.every(u => u.age >= 18); // true
const hasAdmin = users.some(u => u.role === "admin"); // true

// ═══ sort ═══
const byAge = [...users].sort((a, b) => a.age - b.age);
const byName = [...users].sort((a, b) => a.name.localeCompare(b.name));

// ═══ Chaining ═══
const result = users
  .filter(u => u.role === "admin")
  .map(u => u.name.toUpperCase())
  .sort()
  .join(", ");
// "ALICE, CHARLIE"
```

---

## 6. Advanced Patterns

### 6.1. Currying

```javascript
// Currying với arrow functions
const multiply = (a) => (b) => a * b;

const double = multiply(2);
const triple = multiply(3);

double(5);  // 10
triple(5);  // 15

// Practical: Event handler factory
const handleChange = (field) => (event) => {
  setState({ [field]: event.target.value });
};

// Sử dụng trong React
<input onChange={handleChange("username")} />
<input onChange={handleChange("email")} />
```

### 6.2. Composition

```javascript
// Function composition
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

const addOne = (x) => x + 1;
const double = (x) => x * 2;
const square = (x) => x * x;

const transform = pipe(addOne, double, square);
transform(3); // ((3 + 1) * 2)² = 64
```

### 6.3. Conditional Arrow Functions

```javascript
// Short-circuit evaluation
const greet = (name) => name ? `Hello, ${name}!` : "Hello, stranger!";

// Nullish coalescing
const display = (value) => value ?? "N/A";

// Multiple conditions
const getDiscount = (role) =>
  role === "admin" ? 0.5 :
  role === "member" ? 0.2 :
  role === "guest" ? 0.05 :
  0;
```

---

## 7. Best Practices

```javascript
// ✅ Dùng arrow function cho callbacks
array.map(item => item.value);
promise.then(data => processData(data));

// ✅ Dùng arrow function khi cần lexical this
class Component {
  handleClick = () => {
    // this luôn là Component instance
  };
}

// ✅ Dùng () cho implicit return object
const toUser = (name, age) => ({ name, age });

// ✅ Dùng explicit return cho logic phức tạp
const process = (data) => {
  if (!data) return null;
  const transformed = transform(data);
  return validate(transformed) ? transformed : null;
};

// ❌ Tránh nested arrow quá sâu (khó đọc)
const bad = (a) => (b) => (c) => (d) => a + b + c + d;

// ✅ Tốt hơn
const add = (a, b, c, d) => a + b + c + d;

// ❌ Tránh arrow function quá dài (> 3 dòng implicit)
// Nên dùng explicit return block cho readability
```

---

## 8. Câu hỏi phỏng vấn thường gặp

### Q1: Arrow function khác gì regular function?
**A**: 4 điểm chính: (1) Không có `this` riêng (lexical), (2) Không có `arguments`, (3) Không thể làm constructor, (4) Không có `prototype`.

### Q2: Output là gì?

```javascript
const obj = {
  name: "Object",
  getName: () => this.name,
  getNameRegular() { return this.name; }
};

console.log(obj.getName());        // undefined (this = window/global)
console.log(obj.getNameRegular()); // "Object" (this = obj)
```

### Q3: Khi nào KHÔNG nên dùng arrow function?
**A**: Object methods, event handlers cần `this` = element, prototype methods, constructor functions, generator functions.

---

> Tiếp theo: [Template Literals →](./03-template-literals.md)
