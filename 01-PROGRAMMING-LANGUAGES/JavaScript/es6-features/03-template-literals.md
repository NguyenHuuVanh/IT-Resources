# Template Literals

> Quay lại [Mục lục](./README.md) | Trước: [Arrow Functions](./02-arrow-functions.md)

## 📌 Tổng quan

Template Literals (hay Template Strings) là cách viết chuỗi mới trong ES6 sử dụng dấu backtick (`` ` ``) thay vì dấu nháy đơn/đôi. Chúng hỗ trợ **string interpolation**, **multi-line strings**, và **tagged templates** — những tính năng mà string truyền thống không có.

---

## 1. Cú pháp cơ bản

### 1.1. String Interpolation (Nội suy chuỗi)

```javascript
// ES5 - String concatenation (nối chuỗi)
var name = "John";
var age = 25;
var greeting = "Hello, " + name + "! You are " + age + " years old.";

// ES6 - Template literal với ${expression}
const name = "John";
const age = 25;
const greeting = `Hello, ${name}! You are ${age} years old.`;
```

### 1.2. Expressions trong `${}`

Bất kỳ **JavaScript expression** nào đều có thể đặt trong `${}`.

```javascript
const a = 10;
const b = 20;

// Arithmetic
console.log(`Sum: ${a + b}`);         // Sum: 30
console.log(`Product: ${a * b}`);     // Product: 200
console.log(`Power: ${a ** 2}`);      // Power: 100

// Function calls
const upper = (str) => str.toUpperCase();
console.log(`Hello ${upper("world")}`); // Hello WORLD

// Ternary operator
const age = 20;
console.log(`Status: ${age >= 18 ? "Adult" : "Minor"}`); // Status: Adult

// Object properties
const user = { name: "John", age: 25 };
console.log(`${user.name} is ${user.age} years old`);

// Array elements & methods
const colors = ["red", "green", "blue"];
console.log(`First: ${colors[0]}`);              // First: red
console.log(`All: ${colors.join(", ")}`);         // All: red, green, blue

// Nested template literals
const isAdmin = true;
const message = `User is ${isAdmin ? `an <strong>admin</strong>` : `a regular user`}`;

// Method calls
const price = 9.99;
console.log(`Price: $${price.toFixed(2)}`);       // Price: $9.99

// Immediately Invoked
console.log(`Result: ${(() => 42)()}`);           // Result: 42
```

---

## 2. Multi-line Strings

### 2.1. So sánh ES5 vs ES6

```javascript
// ES5 - Cần \n hoặc concatenation
var html = '<div>\n' +
           '  <h1>Title</h1>\n' +
           '  <p>Content</p>\n' +
           '</div>';

// Hoặc dùng \
var html = '<div>\
  <h1>Title</h1>\
  <p>Content</p>\
</div>';
// ⚠️ \ line continuation không thêm \n

// ES6 - Multi-line tự nhiên
const html = `
  <div>
    <h1>Title</h1>
    <p>Content</p>
  </div>
`;
// Giữ nguyên whitespace và line breaks
```

### 2.2. Lưu ý về Whitespace

```javascript
// ⚠️ Template literal giữ nguyên tất cả whitespace (kể cả indentation)
function render() {
  return `
    <div>
      <p>Hello</p>
    </div>
  `;
}
// String sẽ chứa leading/trailing newlines và spaces

// ✅ Trim whitespace
function render() {
  return `
    <div>
      <p>Hello</p>
    </div>
  `.trim();
}

// ✅ Hoặc bắt đầu ngay sau backtick
function render() {
  return `<div>
  <p>Hello</p>
</div>`;
}
```

---

## 3. Tagged Templates

### 3.1. Khái niệm

Tagged Templates cho phép bạn xử lý template literal bằng một function. Function nhận các phần string tĩnh và các giá trị interpolated riêng biệt.

```javascript
// Syntax: tagFunction`template literal`
function tag(strings, ...values) {
  // strings: Array các phần string tĩnh
  // values: Array các giá trị interpolated
  console.log(strings); // ["Hello ", ", you are ", " years old"]
  console.log(values);  // ["John", 25]
}

tag`Hello ${"John"}, you are ${25} years old`;
```

### 3.2. Ví dụ: Highlight

```javascript
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] !== undefined 
      ? `<mark>${values[i]}</mark>` 
      : "");
  }, "");
}

const name = "John";
const count = 5;
const result = highlight`Hello ${name}, you have ${count} messages`;
// "Hello <mark>John</mark>, you have <mark>5</mark> messages"
```

### 3.3. Ví dụ thực tế: SQL Query Builder (chống SQL Injection)

```javascript
function sql(strings, ...values) {
  // Escape tất cả values để chống SQL injection
  const escaped = values.map(v => {
    if (typeof v === "string") {
      return `'${v.replace(/'/g, "''")}'`; // Escape single quotes
    }
    if (v === null) return "NULL";
    if (typeof v === "boolean") return v ? "TRUE" : "FALSE";
    return String(v);
  });
  
  return strings.reduce(
    (query, str, i) => query + str + (escaped[i] || ""), 
    ""
  );
}

const username = "John'; DROP TABLE users; --";
const age = 25;
const query = sql`SELECT * FROM users WHERE name = ${username} AND age = ${age}`;
// SELECT * FROM users WHERE name = 'John''; DROP TABLE users; --' AND age = 25
```

### 3.4. Ví dụ: CSS-in-JS (styled-components pattern)

```javascript
function css(strings, ...values) {
  return strings.reduce(
    (result, str, i) => result + str + (values[i] || ""), 
    ""
  );
}

const primaryColor = "#007bff";
const fontSize = 16;

const styles = css`
  .button {
    background-color: ${primaryColor};
    font-size: ${fontSize}px;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
  }
`;
```

### 3.5. Ví dụ: Internationalization (i18n)

```javascript
const translations = {
  en: { greeting: "Hello", farewell: "Goodbye" },
  vi: { greeting: "Xin chào", farewell: "Tạm biệt" },
};

function i18n(lang) {
  return function(strings, ...keys) {
    return strings.reduce((result, str, i) => {
      const translation = translations[lang]?.[keys[i]] || keys[i] || "";
      return result + str + translation;
    }, "");
  };
}

const t = i18n("vi");
console.log(t`${"greeting"}, world! ${"farewell"}!`);
// "Xin chào, world! Tạm biệt!"
```

### 3.6. Ví dụ: HTML Sanitization

```javascript
function safeHtml(strings, ...values) {
  const escapeHtml = (str) =>
    String(str)
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#39;");

  return strings.reduce(
    (result, str, i) => result + str + (values[i] !== undefined ? escapeHtml(values[i]) : ""),
    ""
  );
}

const userInput = '<script>alert("XSS")</script>';
const html = safeHtml`<div class="content">${userInput}</div>`;
// '<div class="content">&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</div>'
```

---

## 4. Raw Strings

### 4.1. `String.raw`

`String.raw` là built-in tagged template function giữ nguyên escape sequences.

```javascript
// Bình thường - escape sequences được xử lý
console.log(`Line1\nLine2`);
// Line1
// Line2

console.log(`Tab\there`);
// Tab    here

// String.raw - giữ nguyên escape sequences (raw form)
console.log(String.raw`Line1\nLine2`);
// Line1\nLine2

console.log(String.raw`Tab\there`);
// Tab\there
```

### 4.2. Use Cases

```javascript
// ✅ File paths (Windows)
const path = String.raw`C:\Users\John\Documents\file.txt`;
console.log(path); // C:\Users\John\Documents\file.txt

// ✅ Regular expressions
const pattern = String.raw`\d+\.\d+`;
const regex = new RegExp(pattern);
regex.test("3.14"); // true

// ✅ LaTeX
const formula = String.raw`\frac{n!}{k!(n-k)!}`;

// ✅ Unicode escape sequences
console.log(String.raw`\u0041`); // \u0041 (không phải "A")
```

### 4.3. `strings.raw` trong Tagged Templates

```javascript
function showRaw(strings, ...values) {
  console.log("Cooked:", strings);     // Escape sequences ĐÃ xử lý
  console.log("Raw:", strings.raw);    // Escape sequences CHƯA xử lý
}

showRaw`Line1\nLine2`;
// Cooked: ["Line1\nLine2"]  (có newline character)
// Raw: ["Line1\\nLine2"]    (literal \n)
```

---

## 5. Practical Patterns

### 5.1. HTML Templates

```javascript
const renderCard = ({ title, description, imageUrl, tags }) => `
  <div class="card">
    <img src="${imageUrl}" alt="${title}" class="card__image" />
    <div class="card__body">
      <h2 class="card__title">${title}</h2>
      <p class="card__description">${description}</p>
      <div class="card__tags">
        ${tags.map(tag => `<span class="tag">${tag}</span>`).join("")}
      </div>
    </div>
  </div>
`.trim();

// Sử dụng
const html = renderCard({
  title: "ES6 Guide",
  description: "Learn modern JavaScript",
  imageUrl: "/images/es6.jpg",
  tags: ["javascript", "es6", "tutorial"],
});
```

### 5.2. Logging với Colors

```javascript
const log = {
  info: (msg) => console.log(`ℹ️  [INFO] ${msg}`),
  warn: (msg) => console.warn(`⚠️  [WARN] ${msg}`),
  error: (msg) => console.error(`❌ [ERROR] ${msg}`),
  success: (msg) => console.log(`✅ [SUCCESS] ${msg}`),
  debug: (msg) => console.log(`🐛 [DEBUG] ${new Date().toISOString()} ${msg}`),
};

log.info(`User ${user.name} logged in`);
log.error(`Failed to fetch ${url}: ${error.message}`);
```

### 5.3. String Builder Pattern

```javascript
const buildQuery = (table, conditions, options = {}) => {
  const { orderBy, limit, offset } = options;
  
  let query = `SELECT * FROM ${table}`;
  
  if (conditions.length > 0) {
    query += ` WHERE ${conditions.join(" AND ")}`;
  }
  if (orderBy) query += ` ORDER BY ${orderBy}`;
  if (limit) query += ` LIMIT ${limit}`;
  if (offset) query += ` OFFSET ${offset}`;
  
  return query;
};

buildQuery("users", ["age > 18", "status = 'active'"], {
  orderBy: "name ASC",
  limit: 10,
});
// "SELECT * FROM users WHERE age > 18 AND status = 'active' ORDER BY name ASC LIMIT 10"
```

---

## 6. Best Practices

```javascript
// ✅ Dùng template literals cho string có interpolation
const greeting = `Hello, ${name}!`;

// ✅ Dùng cho multi-line strings
const html = `
  <div>
    <p>${content}</p>
  </div>
`;

// ℹ️ String đơn giản không cần interpolation — dùng '' hoặc ""
const simple = "Hello, World!";

// ❌ Tránh template literal quá phức tạp
const bad = `${condition ? `${a > b ? `${x}` : `${y}`}` : `${z}`}`;

// ✅ Tách ra cho dễ đọc
const value = condition 
  ? (a > b ? x : y) 
  : z;
const good = `Result: ${value}`;

// ✅ Dùng tagged template cho security-sensitive contexts
const query = sql`SELECT * FROM users WHERE id = ${userId}`;
const html = safeHtml`<p>${userInput}</p>`;
```

---

> Tiếp theo: [Destructuring →](./04-destructuring.md)
