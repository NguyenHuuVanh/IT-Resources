# Tổng hợp Thuật ngữ IT

## Mục lục

1. [Thuật ngữ Lập trình cơ bản](#1-thuật-ngữ-lập-trình-cơ-bản)
2. [Thuật ngữ Web Development](#2-thuật-ngữ-web-development)
3. [Thuật ngữ Database](#3-thuật-ngữ-database)
4. [Thuật ngữ DevOps & Infrastructure](#4-thuật-ngữ-devops--infrastructure)
5. [Thuật ngữ Security](#5-thuật-ngữ-security)
6. [Thuật ngữ Architecture & Design](#6-thuật-ngữ-architecture--design)
7. [Thuật ngữ Testing](#7-thuật-ngữ-testing)
8. [Thuật ngữ Agile & Project Management](#8-thuật-ngữ-agile--project-management)
9. [Viết tắt thường gặp](#9-viết-tắt-thường-gặp)

---

## 1. Thuật ngữ Lập trình cơ bản

### Algorithm (Thuật toán)

**Định nghĩa:** Tập hợp các bước hướng dẫn để giải quyết một vấn đề cụ thể.

**Ví dụ đời thường:** Công thức nấu ăn là một algorithm - các bước tuần tự để tạo ra món ăn.

**Ví dụ code:**

```javascript
// Algorithm tìm số lớn nhất trong mảng
function findMax(arr) {
  let max = arr[0];
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > max) {
      max = arr[i];
    }
  }
  return max;
}
```

---

### Variable (Biến)

**Định nghĩa:** Vùng nhớ được đặt tên để lưu trữ dữ liệu, giá trị có thể thay đổi.

**Ví dụ đời thường:** Như một chiếc hộp có dán nhãn, bạn có thể bỏ đồ vào và lấy ra.

```javascript
let age = 25; // Biến có thể thay đổi
const name = "John"; // Hằng số, không thể thay đổi
var score = 100; // Cách khai báo cũ
```

---

### Function (Hàm)

**Định nghĩa:** Khối code được đặt tên, có thể tái sử dụng, nhận input và trả về output.

**Ví dụ đời thường:** Như máy pha cà phê - bỏ cà phê vào (input), nhấn nút, ra cà phê (output).

```javascript
// Khai báo function
function greet(name) {
  return `Hello, ${name}!`;
}

// Gọi function
greet("Alice"); // "Hello, Alice!"
```

---

### Loop (Vòng lặp)

**Định nghĩa:** Cấu trúc cho phép thực thi một đoạn code nhiều lần.

**Các loại:**

- `for`: Biết trước số lần lặp
- `while`: Lặp khi điều kiện đúng
- `do-while`: Lặp ít nhất 1 lần
- `for...of`: Lặp qua các phần tử
- `for...in`: Lặp qua các keys

```javascript
// for loop
for (let i = 0; i < 5; i++) {
  console.log(i); // 0, 1, 2, 3, 4
}

// while loop
let count = 0;
while (count < 3) {
  console.log(count);
  count++;
}

// for...of
const fruits = ["apple", "banana"];
for (const fruit of fruits) {
  console.log(fruit);
}
```

---

### Condition (Điều kiện)

**Định nghĩa:** Cấu trúc cho phép thực thi code dựa trên điều kiện đúng/sai.

```javascript
const age = 18;

if (age >= 18) {
  console.log("Đủ tuổi");
} else if (age >= 16) {
  console.log("Gần đủ tuổi");
} else {
  console.log("Chưa đủ tuổi");
}

// Ternary operator
const status = age >= 18 ? "Adult" : "Minor";
```

---

### Array (Mảng)

**Định nghĩa:** Cấu trúc dữ liệu lưu trữ nhiều giá trị trong một biến, truy cập qua index.

**Ví dụ đời thường:** Như dãy tủ đồ có đánh số, mỗi tủ chứa một món đồ.

```javascript
const fruits = ["apple", "banana", "orange"];

fruits[0]; // 'apple' (index bắt đầu từ 0)
fruits.length; // 3
fruits.push("grape"); // Thêm vào cuối
fruits.pop(); // Xóa phần tử cuối
```

---

### Object (Đối tượng)

**Định nghĩa:** Cấu trúc dữ liệu lưu trữ dữ liệu dạng key-value pairs.

**Ví dụ đời thường:** Như hồ sơ cá nhân - có các trường thông tin (tên, tuổi, địa chỉ).

```javascript
const person = {
  name: "John",
  age: 30,
  email: "john@example.com",
  greet: function () {
    return `Hi, I'm ${this.name}`;
  },
};

person.name; // "John"
person.age; // 30
person.greet(); // "Hi, I'm John"
```

---

### Class (Lớp)

**Định nghĩa:** Bản thiết kế (blueprint) để tạo ra các objects có cùng cấu trúc và hành vi.

**Ví dụ đời thường:** Như bản vẽ thiết kế nhà - từ 1 bản vẽ có thể xây nhiều nhà giống nhau.

```javascript
class Car {
  constructor(brand, model) {
    this.brand = brand;
    this.model = model;
  }

  start() {
    return `${this.brand} ${this.model} is starting...`;
  }
}

const myCar = new Car("Toyota", "Camry");
myCar.start(); // "Toyota Camry is starting..."
```

---

### Inheritance (Kế thừa)

**Định nghĩa:** Cơ chế cho phép class con thừa hưởng properties và methods từ class cha.

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  speak() {
    return `${this.name} barks`;
  }
}

const dog = new Dog("Buddy");
dog.speak(); // "Buddy barks"
```

---

### Scope (Phạm vi)

**Định nghĩa:** Vùng trong code mà một biến có thể được truy cập.

**Các loại:**

- **Global scope:** Truy cập được ở mọi nơi
- **Function scope:** Chỉ trong function
- **Block scope:** Chỉ trong block `{}`

```javascript
const globalVar = "I'm global"; // Global scope

function example() {
  const functionVar = "I'm in function"; // Function scope

  if (true) {
    const blockVar = "I'm in block"; // Block scope
    console.log(blockVar); // OK
  }
  // console.log(blockVar); // Error! Không truy cập được
}
```

---

### Closure (Bao đóng)

**Định nghĩa:** Function có khả năng "nhớ" và truy cập biến từ scope bên ngoài, ngay cả khi function đó được thực thi ở nơi khác.

```javascript
function createCounter() {
  let count = 0; // Biến private

  return function () {
    count++;
    return count;
  };
}

const counter = createCounter();
counter(); // 1
counter(); // 2
counter(); // 3
// count không thể truy cập trực tiếp từ bên ngoài
```

---

### Callback

**Định nghĩa:** Function được truyền vào function khác như một argument và được gọi sau khi một operation hoàn thành.

```javascript
function fetchData(callback) {
  setTimeout(() => {
    const data = { name: "John" };
    callback(data); // Gọi callback với data
  }, 1000);
}

fetchData((result) => {
  console.log(result); // { name: "John" }
});
```

---

### Promise

**Định nghĩa:** Object đại diện cho kết quả của một async operation, có thể ở trạng thái pending, fulfilled, hoặc rejected.

```javascript
const promise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("Operation successful!");
  } else {
    reject("Operation failed!");
  }
});

promise.then((result) => console.log(result)).catch((error) => console.error(error));
```

---

### Async/Await

**Định nghĩa:** Syntax sugar trên Promise, cho phép viết async code như synchronous code.

```javascript
async function fetchUser() {
  try {
    const response = await fetch("/api/user");
    const user = await response.json();
    return user;
  } catch (error) {
    console.error("Error:", error);
  }
}
```

---

### Recursion (Đệ quy)

**Định nghĩa:** Kỹ thuật trong đó function gọi chính nó để giải quyết vấn đề.

```javascript
function factorial(n) {
  if (n <= 1) return 1; // Base case
  return n * factorial(n - 1); // Recursive case
}

factorial(5); // 5 * 4 * 3 * 2 * 1 = 120
```

---

### Hoisting

**Định nghĩa:** Cơ chế JavaScript di chuyển declarations lên đầu scope trước khi code được thực thi.

```javascript
console.log(x); // undefined (không error)
var x = 5;

// Tương đương với:
var x;
console.log(x);
x = 5;

// let và const không được hoist
console.log(y); // ReferenceError
let y = 5;
```

---

### This

**Định nghĩa:** Keyword tham chiếu đến object mà function đang được gọi từ đó.

```javascript
const person = {
  name: "John",
  greet: function () {
    console.log(this.name); // "John" - this = person
  },
};

// Arrow function không có this riêng
const obj = {
  name: "Alice",
  greet: () => {
    console.log(this.name); // undefined - this = global/window
  },
};
```

---

### Destructuring

**Định nghĩa:** Syntax cho phép "unpack" values từ arrays hoặc properties từ objects vào các biến riêng biệt.

```javascript
// Array destructuring
const [a, b, c] = [1, 2, 3];
console.log(a); // 1

// Object destructuring
const { name, age } = { name: "John", age: 30 };
console.log(name); // "John"

// Với default values
const { city = "Unknown" } = {};
console.log(city); // "Unknown"
```

---

### Spread Operator (...)

**Định nghĩa:** Operator cho phép "spread" (trải ra) các elements của array hoặc properties của object.

```javascript
// Array
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

// Object
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // { a: 1, b: 2, c: 3 }

// Function arguments
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10
```

---

### Type Coercion (Ép kiểu)

**Định nghĩa:** Quá trình tự động hoặc thủ công chuyển đổi giá trị từ kiểu dữ liệu này sang kiểu khác.

```javascript
// Implicit (tự động)
"5" + 3; // "53" (number → string)
"5" - 3; // 2 (string → number)
!!"hello"; // true (string → boolean)

// Explicit (thủ công)
Number("5"); // 5
String(123); // "123"
Boolean(0); // false
```

---

### Immutability (Bất biến)

**Định nghĩa:** Nguyên tắc không thay đổi data sau khi đã tạo, thay vào đó tạo bản copy mới.

```javascript
// ❌ Mutable
const arr = [1, 2, 3];
arr.push(4); // Thay đổi array gốc

// ✅ Immutable
const arr = [1, 2, 3];
const newArr = [...arr, 4]; // Tạo array mới

// ❌ Mutable
const obj = { name: "John" };
obj.name = "Jane"; // Thay đổi object gốc

// ✅ Immutable
const obj = { name: "John" };
const newObj = { ...obj, name: "Jane" }; // Tạo object mới
```

---

## 2. Thuật ngữ Web Development

### API (Application Programming Interface)

**Định nghĩa:** Giao diện cho phép các ứng dụng giao tiếp với nhau thông qua các quy tắc và giao thức đã định nghĩa.

**Ví dụ đời thường:** Như menu nhà hàng - bạn không cần biết bếp nấu thế nào, chỉ cần gọi món (request) và nhận đồ ăn (response).

```javascript
// Gọi API
fetch('https://api.example.com/users')
  .then(response => response.json())
  .then(data => console.log(data));

// Tạo API endpoint (Express.js)
app.get('/api/users', (req, res) => {
  res.json({ users: [...] });
});
```

---

### REST (Representational State Transfer)

**Định nghĩa:** Kiến trúc thiết kế API sử dụng HTTP methods để thao tác với resources.

**Các HTTP Methods:**
| Method | Mục đích | Ví dụ |
|--------|----------|-------|
| GET | Lấy dữ liệu | GET /users |
| POST | Tạo mới | POST /users |
| PUT | Cập nhật toàn bộ | PUT /users/1 |
| PATCH | Cập nhật một phần | PATCH /users/1 |
| DELETE | Xóa | DELETE /users/1 |

```javascript
// RESTful API endpoints
GET / api / products; // Lấy tất cả products
GET / api / products / 1; // Lấy product có id = 1
POST / api / products; // Tạo product mới
PUT / api / products / 1; // Cập nhật product id = 1
DELETE / api / products / 1; // Xóa product id = 1
```

---

### HTTP (HyperText Transfer Protocol)

**Định nghĩa:** Giao thức truyền tải dữ liệu trên web, định nghĩa cách client và server giao tiếp.

**HTTP Status Codes:**
| Code | Ý nghĩa |
|------|---------|
| 200 | OK - Thành công |
| 201 | Created - Tạo thành công |
| 400 | Bad Request - Request sai |
| 401 | Unauthorized - Chưa xác thực |
| 403 | Forbidden - Không có quyền |
| 404 | Not Found - Không tìm thấy |
| 500 | Internal Server Error - Lỗi server |

```javascript
// HTTP Request structure
{
  method: 'POST',
  url: '/api/users',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: JSON.stringify({ name: 'John' })
}
```

---

### HTTPS

**Định nghĩa:** HTTP Secure - phiên bản bảo mật của HTTP, mã hóa dữ liệu truyền tải bằng SSL/TLS.

**Tại sao cần:**

- Mã hóa dữ liệu (passwords, credit cards)
- Xác thực server
- Bảo vệ khỏi man-in-the-middle attacks
- SEO ranking tốt hơn

---

### DOM (Document Object Model)

**Định nghĩa:** Cấu trúc cây đại diện cho HTML document, cho phép JavaScript truy cập và thao tác với các elements.

**Ví dụ đời thường:** Như sơ đồ tổ chức công ty - từ CEO (root) xuống các phòng ban (branches) đến nhân viên (leaves).

```javascript
// Truy cập DOM elements
document.getElementById("myId");
document.querySelector(".myClass");
document.querySelectorAll("div");

// Thao tác DOM
const element = document.getElementById("title");
element.textContent = "New Title";
element.style.color = "red";
element.classList.add("active");

// Tạo element mới
const newDiv = document.createElement("div");
newDiv.innerHTML = "<p>Hello</p>";
document.body.appendChild(newDiv);
```

---

### SPA (Single Page Application)

**Định nghĩa:** Ứng dụng web chỉ load một trang HTML duy nhất, nội dung được cập nhật động mà không reload trang.

**Ví dụ:** Gmail, Facebook, Twitter

**Ưu điểm:**

- Trải nghiệm mượt mà như app native
- Không reload trang
- Tách biệt frontend và backend

**Nhược điểm:**

- SEO khó hơn
- Initial load chậm hơn
- Cần JavaScript để hoạt động

```javascript
// React SPA routing
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/:id" element={<User />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

### SSR (Server-Side Rendering)

**Định nghĩa:** Kỹ thuật render HTML trên server trước khi gửi về client, thay vì render trên browser.

**Ưu điểm:**

- SEO tốt hơn
- First Contentful Paint nhanh hơn
- Hoạt động khi JavaScript bị disable

**Nhược điểm:**

- Server load cao hơn
- TTFB (Time To First Byte) chậm hơn
- Phức tạp hơn để implement

```javascript
// Next.js SSR
export async function getServerSideProps(context) {
  const res = await fetch("https://api.example.com/data");
  const data = await res.json();

  return {
    props: { data }, // Truyền data vào component
  };
}

function Page({ data }) {
  return <div>{data.title}</div>;
}
```

---

### CSR (Client-Side Rendering)

**Định nghĩa:** Kỹ thuật render HTML trên browser bằng JavaScript, server chỉ gửi HTML trống và JS bundle.

**Flow:**

1. Browser request trang
2. Server trả về HTML trống + JS bundle
3. Browser download và execute JS
4. JS fetch data và render UI

```javascript
// React CSR (mặc định)
function App() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch("/api/data")
      .then((res) => res.json())
      .then(setData);
  }, []);

  if (!data) return <Loading />;
  return <div>{data.title}</div>;
}
```

---

### SSG (Static Site Generation)

**Định nghĩa:** Kỹ thuật generate HTML tĩnh tại build time, không cần server runtime.

**Ưu điểm:**

- Cực kỳ nhanh (serve từ CDN)
- Bảo mật cao (không có server)
- Chi phí hosting thấp

**Phù hợp cho:** Blog, documentation, landing pages

```javascript
// Next.js SSG
export async function getStaticProps() {
  const posts = await fetchPosts();
  return {
    props: { posts },
    revalidate: 3600, // ISR: regenerate mỗi 1 giờ
  };
}

export async function getStaticPaths() {
  const posts = await fetchPosts();
  return {
    paths: posts.map((post) => ({ params: { id: post.id } })),
    fallback: false,
  };
}
```

---

### Routing

**Định nghĩa:** Cơ chế điều hướng giữa các trang/views trong ứng dụng dựa trên URL.

**Các loại:**

- **Server-side routing:** Server xử lý mỗi URL request
- **Client-side routing:** JavaScript xử lý navigation mà không reload

```javascript
// Express.js (Server-side)
app.get("/", (req, res) => res.render("home"));
app.get("/users/:id", (req, res) => {
  const userId = req.params.id;
  res.render("user", { id: userId });
});

// Vue Router (Client-side)
const routes = [
  { path: "/", component: Home },
  { path: "/user/:id", component: User },
  { path: "/:pathMatch(.*)*", component: NotFound }, // 404
];
```

---

### Middleware

**Định nghĩa:** Function nằm giữa request và response, có thể xử lý, modify, hoặc chặn request.

**Ví dụ đời thường:** Như bảo vệ tòa nhà - kiểm tra thẻ trước khi cho vào.

```javascript
// Express.js middleware
// Logger middleware
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // Chuyển sang middleware tiếp theo
};

// Auth middleware
const auth = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: "Unauthorized" });
  }
  // Verify token...
  next();
};

// Sử dụng
app.use(logger); // Áp dụng cho tất cả routes
app.get("/protected", auth, (req, res) => {
  res.json({ data: "secret" });
});
```

---

### Cookie

**Định nghĩa:** Dữ liệu nhỏ được lưu trên browser, tự động gửi kèm mỗi request đến server.

**Đặc điểm:**

- Giới hạn ~4KB
- Có thể set expiration
- Tự động gửi với mỗi request
- Có thể bị chặn bởi browser

```javascript
// Set cookie (Server - Express)
res.cookie("userId", "123", {
  httpOnly: true, // Không truy cập được từ JS
  secure: true, // Chỉ gửi qua HTTPS
  maxAge: 3600000, // 1 giờ
  sameSite: "strict", // Chống CSRF
});

// Set cookie (Client)
document.cookie = "theme=dark; max-age=86400";

// Đọc cookie
const cookies = document.cookie; // "theme=dark; userId=123"
```

---

### Session

**Định nghĩa:** Cơ chế lưu trữ thông tin user trên server, client chỉ giữ session ID.

**Flow:**

1. User login → Server tạo session, lưu vào memory/database
2. Server gửi session ID về client (thường qua cookie)
3. Client gửi session ID với mỗi request
4. Server lookup session để xác định user

```javascript
// Express-session
const session = require("express-session");

app.use(
  session({
    secret: "my-secret-key",
    resave: false,
    saveUninitialized: false,
    cookie: { secure: true, maxAge: 3600000 },
  })
);

// Sử dụng session
app.post("/login", (req, res) => {
  req.session.userId = user.id;
  req.session.role = user.role;
});

app.get("/profile", (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: "Not logged in" });
  }
  // ...
});
```

---

### LocalStorage & SessionStorage

**Định nghĩa:** Web Storage API cho phép lưu trữ dữ liệu trên browser.

| Đặc điểm          | LocalStorage            | SessionStorage   |
| ----------------- | ----------------------- | ---------------- |
| Dung lượng        | ~5-10MB                 | ~5-10MB          |
| Thời gian tồn tại | Vĩnh viễn               | Đến khi đóng tab |
| Phạm vi           | Tất cả tabs cùng origin | Chỉ tab hiện tại |

```javascript
// LocalStorage
localStorage.setItem("theme", "dark");
localStorage.getItem("theme"); // 'dark'
localStorage.removeItem("theme");
localStorage.clear(); // Xóa tất cả

// SessionStorage
sessionStorage.setItem("tempData", JSON.stringify({ id: 1 }));
const data = JSON.parse(sessionStorage.getItem("tempData"));
```

---

### CORS (Cross-Origin Resource Sharing)

**Định nghĩa:** Cơ chế bảo mật cho phép/chặn requests từ domain khác.

**Ví dụ:** Frontend ở `localhost:3000` gọi API ở `localhost:5000` → Cần CORS.

```javascript
// Express.js - Enable CORS
const cors = require("cors");

// Cho phép tất cả origins
app.use(cors());

// Cấu hình chi tiết
app.use(
  cors({
    origin: ["http://localhost:3000", "https://myapp.com"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    credentials: true, // Cho phép gửi cookies
    allowedHeaders: ["Content-Type", "Authorization"],
  })
);
```

---

### WebSocket

**Định nghĩa:** Giao thức cho phép giao tiếp hai chiều (bidirectional) giữa client và server qua một connection duy nhất.

**So sánh với HTTP:**
| HTTP | WebSocket |
|------|-----------|
| Request-Response | Bidirectional |
| Mỗi request tạo connection mới | Một connection duy trì |
| Client khởi tạo | Cả hai bên có thể gửi |

**Use cases:** Chat, live notifications, real-time games, stock prices

```javascript
// Server (ws library)
const WebSocket = require("ws");
const wss = new WebSocket.Server({ port: 8080 });

wss.on("connection", (ws) => {
  ws.on("message", (message) => {
    console.log("Received:", message);
    ws.send("Hello from server!");
  });
});

// Client
const socket = new WebSocket("ws://localhost:8080");

socket.onopen = () => {
  socket.send("Hello from client!");
};

socket.onmessage = (event) => {
  console.log("Received:", event.data);
};
```

---

### GraphQL

**Định nghĩa:** Query language cho API, cho phép client yêu cầu chính xác dữ liệu cần thiết.

**So sánh với REST:**
| REST | GraphQL |
|------|---------|
| Multiple endpoints | Single endpoint |
| Over-fetching/Under-fetching | Lấy đúng data cần |
| Versioning (v1, v2) | Schema evolution |

```graphql
# Schema definition
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Query {
  user(id: ID!): User
  users: [User!]!
}

# Client query - chỉ lấy name và email
query {
  user(id: "1") {
    name
    email
  }
}
```

---

### Webpack

**Định nghĩa:** Module bundler - đóng gói nhiều files JavaScript, CSS, images thành các bundles tối ưu.

**Chức năng:**

- Bundle modules
- Transpile code (Babel)
- Minify và optimize
- Code splitting
- Hot Module Replacement (HMR)

```javascript
// webpack.config.js
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      { test: /\.js$/, use: "babel-loader" },
      { test: /\.css$/, use: ["style-loader", "css-loader"] },
    ],
  },
  plugins: [new HtmlWebpackPlugin({ template: "./src/index.html" })],
};
```

---

### NPM (Node Package Manager)

**Định nghĩa:** Package manager mặc định của Node.js, quản lý dependencies và scripts.

```bash
# Cài đặt package
npm install express           # Cài vào dependencies
npm install -D jest           # Cài vào devDependencies
npm install -g typescript     # Cài global

# Các lệnh thường dùng
npm init                      # Tạo package.json
npm install                   # Cài tất cả dependencies
npm run dev                   # Chạy script "dev"
npm update                    # Cập nhật packages
npm audit                     # Kiểm tra security
```

```json
// package.json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

---

## 3. Thuật ngữ Database

### Database (Cơ sở dữ liệu)

**Định nghĩa:** Hệ thống tổ chức và lưu trữ dữ liệu có cấu trúc, cho phép truy vấn và quản lý hiệu quả.

**Ví dụ đời thường:** Như tủ hồ sơ văn phòng - có ngăn, có nhãn, dễ tìm kiếm.

**Các loại:**

- **Relational (SQL):** MySQL, PostgreSQL, SQL Server
- **NoSQL:** MongoDB, Redis, Cassandra
- **In-memory:** Redis, Memcached
- **Graph:** Neo4j, Amazon Neptune

---

### SQL (Structured Query Language)

**Định nghĩa:** Ngôn ngữ chuẩn để truy vấn và quản lý cơ sở dữ liệu quan hệ.

```sql
-- Tạo bảng
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- CRUD operations
INSERT INTO users (name, email) VALUES ('John', 'john@example.com');
SELECT * FROM users WHERE id = 1;
UPDATE users SET name = 'Jane' WHERE id = 1;
DELETE FROM users WHERE id = 1;

-- JOIN
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

---

### NoSQL

**Định nghĩa:** Cơ sở dữ liệu không sử dụng cấu trúc bảng quan hệ truyền thống, linh hoạt hơn với dữ liệu phi cấu trúc.

**Các loại NoSQL:**
| Loại | Ví dụ | Use case |
|------|-------|----------|
| Document | MongoDB | Dữ liệu JSON-like |
| Key-Value | Redis | Cache, sessions |
| Column-family | Cassandra | Big data, time-series |
| Graph | Neo4j | Social networks, recommendations |

```javascript
// MongoDB
db.users.insertOne({
  name: "John",
  email: "john@example.com",
  address: {
    city: "Hanoi",
    country: "Vietnam",
  },
  hobbies: ["coding", "gaming"],
});

db.users.find({ "address.city": "Hanoi" });
```

---

### CRUD

**Định nghĩa:** Bốn thao tác cơ bản với dữ liệu: Create, Read, Update, Delete.

| Operation | SQL    | MongoDB     | HTTP Method |
| --------- | ------ | ----------- | ----------- |
| Create    | INSERT | insertOne() | POST        |
| Read      | SELECT | find()      | GET         |
| Update    | UPDATE | updateOne() | PUT/PATCH   |
| Delete    | DELETE | deleteOne() | DELETE      |

---

### Primary Key

**Định nghĩa:** Cột (hoặc tổ hợp cột) xác định duy nhất mỗi row trong bảng.

**Đặc điểm:**

- Unique (không trùng lặp)
- Not null (không được rỗng)
- Immutable (không nên thay đổi)

```sql
-- Auto-increment ID
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100)
);

-- UUID
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  total DECIMAL(10, 2)
);

-- Composite key
CREATE TABLE order_items (
  order_id INT,
  product_id INT,
  quantity INT,
  PRIMARY KEY (order_id, product_id)
);
```

---

### Foreign Key

**Định nghĩa:** Cột tham chiếu đến Primary Key của bảng khác, tạo mối quan hệ giữa các bảng.

```sql
CREATE TABLE orders (
  id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT,
  total DECIMAL(10, 2),
  FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE    -- Xóa user → xóa orders
    ON UPDATE CASCADE    -- Update user.id → update orders.user_id
);
```

---

### Index

**Định nghĩa:** Cấu trúc dữ liệu giúp tăng tốc độ truy vấn bằng cách tạo "mục lục" cho cột.

**Ví dụ đời thường:** Như mục lục sách - thay vì đọc từng trang, tra mục lục để tìm nhanh.

**Khi nào dùng:**

- Cột thường xuyên dùng trong WHERE, JOIN, ORDER BY
- Cột có nhiều giá trị khác nhau (high cardinality)

**Khi nào KHÔNG dùng:**

- Bảng nhỏ
- Cột ít giá trị khác nhau (low cardinality)
- Cột thường xuyên UPDATE

```sql
-- Tạo index
CREATE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Composite index
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Xem execution plan
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';
```

---

### Transaction

**Định nghĩa:** Nhóm các operations được thực thi như một đơn vị - tất cả thành công hoặc tất cả rollback.

**Ví dụ đời thường:** Chuyển tiền ngân hàng - trừ tiền A và cộng tiền B phải cùng thành công hoặc cùng thất bại.

```sql
-- SQL Transaction
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Nếu có lỗi
ROLLBACK;

-- Nếu thành công
COMMIT;
```

```javascript
// Sequelize transaction
const t = await sequelize.transaction();

try {
  await Account.update({ balance: sequelize.literal("balance - 100") }, { where: { id: 1 }, transaction: t });
  await Account.update({ balance: sequelize.literal("balance + 100") }, { where: { id: 2 }, transaction: t });
  await t.commit();
} catch (error) {
  await t.rollback();
}
```

---

### ACID

**Định nghĩa:** Bốn thuộc tính đảm bảo tính toàn vẹn của database transactions.

| Thuộc tính      | Ý nghĩa                    | Ví dụ                                       |
| --------------- | -------------------------- | ------------------------------------------- |
| **A**tomicity   | Tất cả hoặc không gì cả    | Chuyển tiền: trừ A + cộng B cùng thành công |
| **C**onsistency | Dữ liệu luôn hợp lệ        | Balance không âm                            |
| **I**solation   | Transactions độc lập       | 2 người cùng mua 1 sản phẩm                 |
| **D**urability  | Dữ liệu được lưu vĩnh viễn | Mất điện không mất data                     |

---

### ORM (Object-Relational Mapping)

**Định nghĩa:** Kỹ thuật ánh xạ giữa objects trong code và tables trong database, cho phép thao tác database bằng code thay vì SQL thuần.

**Ưu điểm:**

- Code dễ đọc, dễ maintain
- Tránh SQL injection
- Database-agnostic

**Nhược điểm:**

- Performance overhead
- Complex queries khó viết
- Learning curve

```javascript
// Sequelize ORM
const User = sequelize.define("User", {
  name: DataTypes.STRING,
  email: DataTypes.STRING,
});

// Thay vì SQL
const users = await User.findAll({
  where: { status: "active" },
  include: [{ model: Order }],
});

// Prisma ORM
const user = await prisma.user.create({
  data: {
    name: "John",
    email: "john@example.com",
    posts: {
      create: { title: "Hello World" },
    },
  },
});
```

---

### ODM (Object-Document Mapping)

**Định nghĩa:** Tương tự ORM nhưng cho NoSQL databases (document-based như MongoDB).

```javascript
// Mongoose ODM
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, unique: true },
  posts: [{ type: mongoose.Schema.Types.ObjectId, ref: "Post" }],
});

const User = mongoose.model("User", userSchema);

// CRUD
const user = await User.create({ name: "John", email: "john@example.com" });
const users = await User.find({ name: /john/i }).populate("posts");
```

---

### Normalization (Chuẩn hóa)

**Định nghĩa:** Quá trình tổ chức database để giảm redundancy (dư thừa) và dependency.

**Các dạng chuẩn:**

- **1NF:** Mỗi cell chứa một giá trị duy nhất
- **2NF:** 1NF + không có partial dependency
- **3NF:** 2NF + không có transitive dependency

```sql
-- ❌ Chưa chuẩn hóa
CREATE TABLE orders (
  id INT,
  customer_name VARCHAR(100),
  customer_email VARCHAR(255),
  customer_address VARCHAR(255),
  product_name VARCHAR(100),
  product_price DECIMAL
);

-- ✅ Đã chuẩn hóa (3NF)
CREATE TABLE customers (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(255),
  address VARCHAR(255)
);

CREATE TABLE products (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  price DECIMAL
);

CREATE TABLE orders (
  id INT PRIMARY KEY,
  customer_id INT REFERENCES customers(id),
  product_id INT REFERENCES products(id),
  quantity INT
);
```

---

### Denormalization

**Định nghĩa:** Ngược lại với normalization - thêm redundancy để tăng performance đọc.

**Khi nào dùng:**

- Read-heavy applications
- Complex JOINs gây chậm
- Reporting/Analytics

```sql
-- Denormalized: lưu thêm customer_name vào orders
CREATE TABLE orders (
  id INT PRIMARY KEY,
  customer_id INT,
  customer_name VARCHAR(100),  -- Redundant nhưng tránh JOIN
  total DECIMAL
);
```

---

### JOIN

**Định nghĩa:** Kết hợp dữ liệu từ nhiều bảng dựa trên điều kiện liên quan.

**Các loại JOIN:**

```sql
-- INNER JOIN: Chỉ lấy rows khớp ở cả 2 bảng
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;

-- LEFT JOIN: Tất cả rows từ bảng trái + rows khớp từ bảng phải
SELECT users.name, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;

-- RIGHT JOIN: Ngược lại LEFT JOIN
-- FULL OUTER JOIN: Tất cả rows từ cả 2 bảng
```

---

### Query Optimization

**Định nghĩa:** Kỹ thuật cải thiện performance của database queries.

**Các kỹ thuật:**

```sql
-- 1. Sử dụng Index
CREATE INDEX idx_email ON users(email);

-- 2. Chỉ SELECT cột cần thiết
SELECT name, email FROM users;  -- ✅
SELECT * FROM users;            -- ❌

-- 3. Sử dụng LIMIT
SELECT * FROM users LIMIT 10 OFFSET 0;

-- 4. Tránh SELECT trong SELECT
-- ❌ Subquery
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);
-- ✅ JOIN
SELECT DISTINCT users.* FROM users INNER JOIN orders ON users.id = orders.user_id;

-- 5. Sử dụng EXPLAIN để phân tích
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';
```

---

### Connection Pool

**Định nghĩa:** Tập hợp các database connections được tạo sẵn và tái sử dụng, thay vì tạo mới mỗi request.

**Tại sao cần:**

- Tạo connection tốn thời gian
- Giới hạn số connections đến database
- Tái sử dụng connections hiệu quả

```javascript
// Node.js với pg (PostgreSQL)
const { Pool } = require("pg");

const pool = new Pool({
  host: "localhost",
  database: "mydb",
  max: 20, // Tối đa 20 connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Sử dụng
const result = await pool.query("SELECT * FROM users");
```

---

### Migration

**Định nghĩa:** Cách quản lý thay đổi database schema theo version, cho phép upgrade/downgrade.

**Tại sao cần:**

- Track changes theo thời gian
- Đồng bộ schema giữa các môi trường
- Rollback khi có lỗi

```javascript
// Sequelize migration
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable("users", {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },
      name: Sequelize.STRING,
      email: Sequelize.STRING,
      createdAt: Sequelize.DATE,
    });
  },

  down: async (queryInterface) => {
    await queryInterface.dropTable("users");
  },
};
```

```bash
# Chạy migrations
npx sequelize-cli db:migrate
npx sequelize-cli db:migrate:undo  # Rollback
```

---

### Seeding

**Định nghĩa:** Quá trình thêm dữ liệu mẫu/khởi tạo vào database.

```javascript
// Sequelize seeder
module.exports = {
  up: async (queryInterface) => {
    await queryInterface.bulkInsert("users", [
      { name: "Admin", email: "admin@example.com", createdAt: new Date() },
      { name: "User", email: "user@example.com", createdAt: new Date() },
    ]);
  },

  down: async (queryInterface) => {
    await queryInterface.bulkDelete("users", null, {});
  },
};
```

---

## 4. Thuật ngữ DevOps & Infrastructure

### DevOps

**Định nghĩa:** Văn hóa và tập hợp practices kết hợp Development và Operations, tự động hóa quy trình phát triển và triển khai phần mềm.

**Mục tiêu:**

- Rút ngắn development cycle
- Tăng tần suất deployment
- Đảm bảo chất lượng và độ tin cậy

**DevOps Lifecycle:**

```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
  ↑                                                          |
  └──────────────────── Feedback ────────────────────────────┘
```

---

### CI/CD

**Định nghĩa:**

- **CI (Continuous Integration):** Tự động merge và test code thường xuyên
- **CD (Continuous Delivery/Deployment):** Tự động deploy code lên các môi trường

**CI/CD Pipeline:**

```
Code Push → Build → Unit Tests → Integration Tests → Deploy to Staging → Deploy to Production
```

```yaml
# GitHub Actions CI/CD
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
      - name: Build
        run: npm run build

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: ./deploy.sh
```

---

### Docker

**Định nghĩa:** Platform để đóng gói ứng dụng và dependencies vào containers, đảm bảo chạy nhất quán trên mọi môi trường.

**Ví dụ đời thường:** Như container vận chuyển hàng hóa - đóng gói mọi thứ cần thiết, ship đi đâu cũng mở ra dùng được.

**Các khái niệm:**

- **Image:** Template read-only để tạo container
- **Container:** Instance đang chạy của image
- **Dockerfile:** File định nghĩa cách build image
- **Docker Compose:** Tool quản lý multi-container

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

```yaml
# docker-compose.yml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db

  db:
    image: postgres:14
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

```bash
# Docker commands
docker build -t myapp .           # Build image
docker run -p 3000:3000 myapp     # Run container
docker ps                          # List running containers
docker logs <container_id>         # Xem logs
docker exec -it <container_id> sh  # Vào container

# Docker Compose
docker-compose up -d               # Start all services
docker-compose down                # Stop all services
docker-compose logs -f             # Xem logs
```

---

### Container

**Định nghĩa:** Đơn vị phần mềm đóng gói code và dependencies, chạy độc lập và nhất quán trên mọi môi trường.

**So sánh Container vs VM:**
| Container | Virtual Machine |
|-----------|-----------------|
| Chia sẻ OS kernel | Mỗi VM có OS riêng |
| Lightweight (MB) | Heavy (GB) |
| Khởi động nhanh (giây) | Khởi động chậm (phút) |
| Ít tài nguyên hơn | Nhiều tài nguyên hơn |

---

### Kubernetes (K8s)

**Định nghĩa:** Platform orchestration để quản lý, scale, và deploy containers tự động.

**Các khái niệm:**

- **Pod:** Đơn vị nhỏ nhất, chứa 1+ containers
- **Service:** Expose pods ra network
- **Deployment:** Quản lý replicas của pods
- **Namespace:** Phân chia cluster thành các môi trường

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 3000
          resources:
            limits:
              memory: "128Mi"
              cpu: "500m"

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer
```

```bash
# Kubectl commands
kubectl apply -f deployment.yaml   # Deploy
kubectl get pods                   # List pods
kubectl get services               # List services
kubectl logs <pod-name>            # Xem logs
kubectl scale deployment myapp --replicas=5  # Scale
```

---

### Cloud Computing

**Định nghĩa:** Mô hình cung cấp tài nguyên IT (servers, storage, databases) qua internet, trả tiền theo usage.

**Các mô hình:**
| Mô hình | Bạn quản lý | Provider quản lý | Ví dụ |
|---------|-------------|------------------|-------|
| **IaaS** | OS, Runtime, App | Hardware, Network | AWS EC2, Azure VM |
| **PaaS** | App, Data | OS, Runtime, Hardware | Heroku, AWS Elastic Beanstalk |
| **SaaS** | Không gì cả | Tất cả | Gmail, Slack, Salesforce |

**Cloud Providers:**

- AWS (Amazon Web Services)
- Google Cloud Platform (GCP)
- Microsoft Azure
- DigitalOcean, Vercel, Netlify

---

### VPS (Virtual Private Server)

**Định nghĩa:** Server ảo được phân chia từ server vật lý, có tài nguyên riêng biệt.

**So sánh:**
| Shared Hosting | VPS | Dedicated Server |
|----------------|-----|------------------|
| Chia sẻ tài nguyên | Tài nguyên riêng | Toàn bộ server |
| Rẻ nhất | Trung bình | Đắt nhất |
| Ít control | Full control | Full control |

---

### Load Balancer

**Định nghĩa:** Thiết bị/service phân phối traffic đến nhiều servers để tăng availability và performance.

**Các thuật toán:**

- **Round Robin:** Lần lượt từng server
- **Least Connections:** Server ít connections nhất
- **IP Hash:** Dựa trên IP client
- **Weighted:** Dựa trên capacity của server

```nginx
# Nginx Load Balancer
upstream backend {
    least_conn;
    server backend1.example.com weight=3;
    server backend2.example.com;
    server backend3.example.com;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

---

### Reverse Proxy

**Định nghĩa:** Server đứng trước backend servers, nhận requests từ clients và forward đến backend.

**Chức năng:**

- Load balancing
- SSL termination
- Caching
- Compression
- Security (hide backend)

```nginx
# Nginx Reverse Proxy
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api {
        proxy_pass http://localhost:5000;
    }
}
```

---

### CDN (Content Delivery Network)

**Định nghĩa:** Mạng lưới servers phân tán địa lý, cache và serve static content từ server gần user nhất.

**Ví dụ đời thường:** Như chuỗi cửa hàng tiện lợi - thay vì đi siêu thị xa, mua ở cửa hàng gần nhà.

**Lợi ích:**

- Giảm latency
- Giảm load cho origin server
- Tăng availability
- DDoS protection

**CDN Providers:** Cloudflare, AWS CloudFront, Fastly, Akamai

---

### DNS (Domain Name System)

**Định nghĩa:** Hệ thống chuyển đổi domain names (google.com) thành IP addresses (142.250.190.14).

**Ví dụ đời thường:** Như danh bạ điện thoại - tra tên để tìm số.

**Các loại DNS Records:**
| Record | Mục đích | Ví dụ |
|--------|----------|-------|
| A | Domain → IPv4 | example.com → 93.184.216.34 |
| AAAA | Domain → IPv6 | example.com → 2606:2800:220:1:... |
| CNAME | Alias domain | www.example.com → example.com |
| MX | Mail server | example.com → mail.example.com |
| TXT | Text data | SPF, DKIM records |

---

### SSL/TLS

**Định nghĩa:** Protocols mã hóa dữ liệu truyền tải giữa client và server.

- **SSL (Secure Sockets Layer):** Phiên bản cũ, deprecated
- **TLS (Transport Layer Security):** Phiên bản mới, an toàn hơn

**SSL Certificate:** Chứng chỉ xác thực identity của website.

```bash
# Tạo SSL certificate với Let's Encrypt
sudo certbot --nginx -d example.com -d www.example.com
```

```nginx
# Nginx SSL config
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:...;
}
```

---

### Environment Variables

**Định nghĩa:** Biến cấu hình được set ở môi trường runtime, không hardcode trong code.

**Tại sao cần:**

- Bảo mật (không commit secrets)
- Linh hoạt giữa các môi trường
- Dễ thay đổi không cần rebuild

```bash
# .env file
NODE_ENV=production
DATABASE_URL=postgres://user:pass@localhost:5432/db
JWT_SECRET=my-super-secret-key
API_KEY=abc123
```

```javascript
// Sử dụng trong Node.js
require("dotenv").config();

const dbUrl = process.env.DATABASE_URL;
const jwtSecret = process.env.JWT_SECRET;

// Kiểm tra required env vars
const requiredEnvVars = ["DATABASE_URL", "JWT_SECRET"];
requiredEnvVars.forEach((varName) => {
  if (!process.env[varName]) {
    throw new Error(`Missing required env var: ${varName}`);
  }
});
```

---

### Logging

**Định nghĩa:** Ghi lại các events, errors, và thông tin trong quá trình ứng dụng chạy.

**Log Levels:**
| Level | Mục đích |
|-------|----------|
| ERROR | Lỗi nghiêm trọng |
| WARN | Cảnh báo |
| INFO | Thông tin chung |
| DEBUG | Chi tiết để debug |
| TRACE | Chi tiết nhất |

```javascript
// Winston logger
const winston = require("winston");

const logger = winston.createLogger({
  level: "info",
  format: winston.format.combine(winston.format.timestamp(), winston.format.json()),
  transports: [
    new winston.transports.File({ filename: "error.log", level: "error" }),
    new winston.transports.File({ filename: "combined.log" }),
    new winston.transports.Console(),
  ],
});

logger.info("Server started", { port: 3000 });
logger.error("Database connection failed", { error: err.message });
```

---

### Monitoring

**Định nghĩa:** Theo dõi health, performance, và availability của hệ thống.

**Metrics quan trọng:**

- **CPU/Memory usage**
- **Response time**
- **Error rate**
- **Request rate (RPS)**
- **Uptime**

**Tools:** Prometheus, Grafana, Datadog, New Relic

```javascript
// Prometheus metrics với prom-client
const client = require("prom-client");

const httpRequestDuration = new client.Histogram({
  name: "http_request_duration_seconds",
  help: "Duration of HTTP requests in seconds",
  labelNames: ["method", "route", "status_code"],
  buckets: [0.1, 0.5, 1, 2, 5],
});

// Middleware
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on("finish", () => {
    end({ method: req.method, route: req.path, status_code: res.statusCode });
  });
  next();
});
```

---

### Caching

**Định nghĩa:** Lưu trữ tạm thời dữ liệu để truy cập nhanh hơn, giảm load cho database/API.

**Các loại cache:**

- **Browser cache:** Cache ở client
- **CDN cache:** Cache ở edge servers
- **Application cache:** Redis, Memcached
- **Database cache:** Query cache

```javascript
// Redis caching
const Redis = require("ioredis");
const redis = new Redis();

async function getUser(userId) {
  // Check cache first
  const cached = await redis.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }

  // Query database
  const user = await db.users.findById(userId);

  // Store in cache (expire after 1 hour)
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));

  return user;
}

// Cache invalidation
async function updateUser(userId, data) {
  await db.users.update(userId, data);
  await redis.del(`user:${userId}`); // Xóa cache
}
```

---

## 5. Thuật ngữ Security

### Authentication (Xác thực)

**Định nghĩa:** Quá trình xác minh danh tính của user - "Bạn là ai?"

**Ví dụ đời thường:** Như kiểm tra CMND/CCCD khi vào tòa nhà.

**Các phương thức:**

- Username/Password
- Multi-factor Authentication (MFA)
- Biometrics (vân tay, khuôn mặt)
- OAuth/Social login
- SSO (Single Sign-On)

```javascript
// Basic authentication
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });
  if (!user) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  const isValid = await bcrypt.compare(password, user.password);
  if (!isValid) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET);
  res.json({ token });
});
```

---

### Authorization (Phân quyền)

**Định nghĩa:** Quá trình xác định user có quyền truy cập resource không - "Bạn được phép làm gì?"

**Ví dụ đời thường:** Như thẻ từ văn phòng - có thể vào tầng 1 nhưng không vào phòng server.

**Các mô hình:**

- **RBAC (Role-Based):** Phân quyền theo role (admin, user, editor)
- **ABAC (Attribute-Based):** Phân quyền theo attributes
- **ACL (Access Control List):** Danh sách quyền cụ thể

```javascript
// RBAC middleware
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: "Not authenticated" });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: "Not authorized" });
    }

    next();
  };
};

// Sử dụng
app.delete("/users/:id", authorize("admin"), deleteUser);
app.get("/profile", authorize("admin", "user"), getProfile);
```

---

### JWT (JSON Web Token)

**Định nghĩa:** Token format để truyền thông tin xác thực giữa client và server một cách an toàn.

**Cấu trúc:** `header.payload.signature`

```javascript
// JWT structure
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  "userId": "123",
  "email": "john@example.com",
  "role": "admin",
  "iat": 1516239022,  // Issued at
  "exp": 1516242622   // Expiration
}

// Signature
HMACSHA256(base64(header) + "." + base64(payload), secret)
```

```javascript
// Tạo và verify JWT
const jwt = require("jsonwebtoken");

// Tạo token
const token = jwt.sign({ userId: user.id, role: user.role }, process.env.JWT_SECRET, { expiresIn: "1h" });

// Verify token
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: "Invalid token" });
  }
};
```

---

### OAuth 2.0

**Định nghĩa:** Protocol cho phép ứng dụng truy cập tài nguyên của user từ service khác mà không cần password.

**Ví dụ:** "Đăng nhập bằng Google/Facebook/GitHub"

**Flow (Authorization Code):**

```
1. User click "Login with Google"
2. Redirect đến Google authorization page
3. User đồng ý cho phép
4. Google redirect về app với authorization code
5. App đổi code lấy access token
6. App dùng token để lấy user info từ Google
```

```javascript
// Passport.js với Google OAuth
const GoogleStrategy = require("passport-google-oauth20").Strategy;

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: "/auth/google/callback",
    },
    async (accessToken, refreshToken, profile, done) => {
      let user = await User.findOne({ googleId: profile.id });

      if (!user) {
        user = await User.create({
          googleId: profile.id,
          email: profile.emails[0].value,
          name: profile.displayName,
        });
      }

      done(null, user);
    }
  )
);

// Routes
app.get("/auth/google", passport.authenticate("google", { scope: ["profile", "email"] }));
app.get("/auth/google/callback", passport.authenticate("google"), (req, res) => {
  res.redirect("/dashboard");
});
```

---

### Hashing

**Định nghĩa:** Chuyển đổi data thành chuỗi fixed-length, một chiều (không thể reverse).

**Đặc điểm:**

- Cùng input → cùng output
- Không thể reverse
- Thay đổi nhỏ → output khác hoàn toàn

**Use cases:** Lưu passwords, verify file integrity

```javascript
// Bcrypt - hash passwords
const bcrypt = require("bcrypt");

// Hash password
const saltRounds = 10;
const hashedPassword = await bcrypt.hash("myPassword123", saltRounds);
// $2b$10$N9qo8uLOickgx2ZMRZoMy...

// Verify password
const isMatch = await bcrypt.compare("myPassword123", hashedPassword);
// true
```

---

### Encryption (Mã hóa)

**Định nghĩa:** Chuyển đổi data thành dạng không đọc được, có thể decrypt với key.

**Các loại:**

- **Symmetric:** Cùng key để encrypt và decrypt (AES)
- **Asymmetric:** Public key encrypt, private key decrypt (RSA)

```javascript
// Symmetric encryption với crypto
const crypto = require("crypto");

const algorithm = "aes-256-cbc";
const key = crypto.randomBytes(32);
const iv = crypto.randomBytes(16);

// Encrypt
function encrypt(text) {
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  let encrypted = cipher.update(text, "utf8", "hex");
  encrypted += cipher.final("hex");
  return encrypted;
}

// Decrypt
function decrypt(encrypted) {
  const decipher = crypto.createDecipheriv(algorithm, key, iv);
  let decrypted = decipher.update(encrypted, "hex", "utf8");
  decrypted += decipher.final("utf8");
  return decrypted;
}
```

---

### XSS (Cross-Site Scripting)

**Định nghĩa:** Tấn công inject malicious scripts vào web pages, thực thi trên browser của users khác.

**Các loại:**

- **Stored XSS:** Script lưu trong database
- **Reflected XSS:** Script trong URL parameters
- **DOM-based XSS:** Script thao tác DOM

```javascript
// ❌ Vulnerable
document.innerHTML = userInput;

// ✅ Safe - Escape HTML
function escapeHtml(text) {
  const div = document.createElement("div");
  div.textContent = text;
  return div.innerHTML;
}

// ✅ Safe - Sử dụng textContent
document.getElementById("output").textContent = userInput;

// ✅ React tự động escape
function Comment({ text }) {
  return <div>{text}</div>; // Safe
}

// Server-side: Set CSP header
app.use((req, res, next) => {
  res.setHeader("Content-Security-Policy", "default-src 'self'; script-src 'self'");
  next();
});
```

---

### CSRF (Cross-Site Request Forgery)

**Định nghĩa:** Tấn công lừa user thực hiện actions không mong muốn trên website đã authenticated.

**Ví dụ:** User đang login bank, click link độc hại → chuyển tiền mà không biết.

**Phòng chống:**

```javascript
// CSRF Token
const csrf = require("csurf");
const csrfProtection = csrf({ cookie: true });

app.get("/form", csrfProtection, (req, res) => {
  res.render("form", { csrfToken: req.csrfToken() });
});

app.post("/transfer", csrfProtection, (req, res) => {
  // CSRF token được verify tự động
  // Thực hiện transfer...
});
```

```html
<!-- Form với CSRF token -->
<form action="/transfer" method="POST">
  <input type="hidden" name="_csrf" value="{{csrfToken}}" />
  <input type="text" name="amount" />
  <button type="submit">Transfer</button>
</form>
```

---

### SQL Injection

**Định nghĩa:** Tấn công inject SQL code vào input để thao tác database.

```javascript
// ❌ Vulnerable
const query = `SELECT * FROM users WHERE email = '${email}'`;
// Input: ' OR '1'='1
// Query: SELECT * FROM users WHERE email = '' OR '1'='1'

// ✅ Safe - Parameterized queries
const query = "SELECT * FROM users WHERE email = $1";
const result = await pool.query(query, [email]);

// ✅ Safe - ORM
const user = await User.findOne({ where: { email } });
```

---

### Rate Limiting

**Định nghĩa:** Giới hạn số requests từ một client trong khoảng thời gian, chống abuse và DDoS.

```javascript
const rateLimit = require("express-rate-limit");

// Giới hạn 100 requests / 15 phút
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: "Too many requests, please try again later.",
  standardHeaders: true,
  legacyHeaders: false,
});

app.use(limiter);

// Rate limit riêng cho login
const loginLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts
  message: "Too many login attempts",
});

app.post("/login", loginLimiter, loginHandler);
```

---

### Input Validation

**Định nghĩa:** Kiểm tra và sanitize user input trước khi xử lý.

```javascript
// Joi validation
const Joi = require("joi");

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).max(100).required(),
  age: Joi.number().integer().min(18).max(120),
  role: Joi.string().valid("user", "admin").default("user"),
});

app.post("/register", async (req, res) => {
  const { error, value } = userSchema.validate(req.body);

  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }

  // value đã được validate và sanitize
  const user = await User.create(value);
});
```

---

### HTTPS/TLS

**Định nghĩa:** Mã hóa traffic giữa client và server, bảo vệ data in transit.

**Tại sao cần:**

- Mã hóa sensitive data (passwords, credit cards)
- Xác thực server identity
- Prevent man-in-the-middle attacks
- SEO ranking

```javascript
// Force HTTPS redirect
app.use((req, res, next) => {
  if (req.headers["x-forwarded-proto"] !== "https") {
    return res.redirect(`https://${req.hostname}${req.url}`);
  }
  next();
});

// HSTS header
app.use((req, res, next) => {
  res.setHeader("Strict-Transport-Security", "max-age=31536000; includeSubDomains");
  next();
});
```

---

### Security Headers

**Định nghĩa:** HTTP headers bảo vệ ứng dụng khỏi các loại tấn công phổ biến.

```javascript
// Helmet.js - set security headers
const helmet = require("helmet");

app.use(helmet());

// Hoặc cấu hình chi tiết
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
    },
    noSniff: true,
    xssFilter: true,
  })
);
```

**Các headers quan trọng:**
| Header | Mục đích |
|--------|----------|
| Content-Security-Policy | Chống XSS, injection |
| X-Frame-Options | Chống clickjacking |
| X-Content-Type-Options | Chống MIME sniffing |
| Strict-Transport-Security | Force HTTPS |
| X-XSS-Protection | Browser XSS filter |

---

### Secrets Management

**Định nghĩa:** Quản lý và bảo vệ sensitive data như API keys, passwords, certificates.

**Best practices:**

- Không commit secrets vào git
- Sử dụng environment variables
- Rotate secrets định kỳ
- Sử dụng secrets manager (AWS Secrets Manager, HashiCorp Vault)

```javascript
// ❌ Bad - hardcoded secrets
const apiKey = 'sk-1234567890abcdef';

// ✅ Good - environment variables
const apiKey = process.env.API_KEY;

// .gitignore
.env
.env.local
*.pem
```

```bash
# .env.example (commit file này)
DATABASE_URL=
JWT_SECRET=
API_KEY=

# .env (KHÔNG commit)
DATABASE_URL=postgres://user:pass@localhost/db
JWT_SECRET=super-secret-key
API_KEY=sk-1234567890
```

---

## 6. Thuật ngữ Architecture & Design

### Monolith Architecture

**Định nghĩa:** Kiến trúc ứng dụng được build và deploy như một đơn vị duy nhất.

**Ưu điểm:**

- Đơn giản để develop và deploy
- Dễ debug và test
- Không có network latency giữa components

**Nhược điểm:**

- Khó scale từng phần
- Codebase lớn khó maintain
- Một lỗi có thể crash toàn bộ app

```
┌─────────────────────────────────────┐
│           Monolith App              │
│  ┌─────────┬─────────┬─────────┐   │
│  │  User   │  Order  │ Product │   │
│  │ Module  │ Module  │ Module  │   │
│  └─────────┴─────────┴─────────┘   │
│  ┌─────────────────────────────┐   │
│  │      Shared Database        │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

---

### Microservices Architecture

**Định nghĩa:** Kiến trúc chia ứng dụng thành các services nhỏ, độc lập, giao tiếp qua APIs.

**Ưu điểm:**

- Scale từng service độc lập
- Tech stack linh hoạt
- Team độc lập
- Fault isolation

**Nhược điểm:**

- Phức tạp hơn
- Network latency
- Data consistency khó hơn
- DevOps overhead

```
┌─────────┐    ┌─────────┐    ┌─────────┐
│  User   │    │  Order  │    │ Product │
│ Service │    │ Service │    │ Service │
└────┬────┘    └────┬────┘    └────┬────┘
     │              │              │
     │    ┌─────────┴─────────┐    │
     └────┤    API Gateway    ├────┘
          └───────────────────┘
```

---

### SOLID Principles

**Định nghĩa:** 5 nguyên tắc thiết kế OOP giúp code dễ maintain và mở rộng.

**S - Single Responsibility:** Mỗi class chỉ có một lý do để thay đổi.

```javascript
// ❌ Bad
class User {
  saveToDatabase() {}
  sendEmail() {}
  generateReport() {}
}

// ✅ Good
class User {}
class UserRepository {
  saveToDatabase() {}
}
class EmailService {
  sendEmail() {}
}
class ReportGenerator {
  generateReport() {}
}
```

**O - Open/Closed:** Open for extension, closed for modification.

```javascript
// ✅ Good - Extend bằng inheritance/composition
class Shape {
  area() {}
}
class Rectangle extends Shape {
  area() {
    return this.width * this.height;
  }
}
class Circle extends Shape {
  area() {
    return Math.PI * this.radius ** 2;
  }
}
```

**L - Liskov Substitution:** Subclass phải thay thế được parent class.

**I - Interface Segregation:** Nhiều interfaces nhỏ tốt hơn một interface lớn.

**D - Dependency Inversion:** Depend on abstractions, not concretions.

```javascript
// ✅ Good - Inject dependency
class UserService {
  constructor(repository) {
    this.repository = repository;
  }
}
```

---

### DRY (Don't Repeat Yourself)

**Định nghĩa:** Tránh duplicate code, mỗi logic chỉ nên tồn tại ở một nơi.

```javascript
// ❌ Bad - Duplicate
function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
function checkEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// ✅ Good - Single source of truth
const validators = {
  email: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  phone: (value) => /^\d{10}$/.test(value),
};
```

---

### KISS (Keep It Simple, Stupid)

**Định nghĩa:** Giữ code đơn giản nhất có thể, tránh over-engineering.

```javascript
// ❌ Over-engineered
class StringReverserFactory {
  createReverser() {
    return new StringReverser();
  }
}

// ✅ Simple
const reverse = (str) => str.split("").reverse().join("");
```

---

### Design Patterns

**Định nghĩa:** Giải pháp tái sử dụng cho các vấn đề phổ biến trong thiết kế phần mềm.

**Creational Patterns:**

```javascript
// Singleton - Chỉ một instance
class Database {
  static instance;
  static getInstance() {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }
}

// Factory - Tạo objects
class AnimalFactory {
  create(type) {
    switch (type) {
      case "dog":
        return new Dog();
      case "cat":
        return new Cat();
    }
  }
}
```

**Structural Patterns:**

```javascript
// Adapter - Chuyển đổi interface
class OldAPI {
  oldMethod() {}
}
class Adapter {
  constructor(oldApi) {
    this.oldApi = oldApi;
  }
  newMethod() {
    return this.oldApi.oldMethod();
  }
}
```

**Behavioral Patterns:**

```javascript
// Observer - Pub/Sub
class EventEmitter {
  constructor() {
    this.events = {};
  }
  on(event, callback) {
    this.events[event] = this.events[event] || [];
    this.events[event].push(callback);
  }
  emit(event, data) {
    this.events[event]?.forEach((cb) => cb(data));
  }
}
```

---

### API Design

**Định nghĩa:** Thiết kế giao diện lập trình ứng dụng rõ ràng, nhất quán, dễ sử dụng.

**RESTful API Best Practices:**

```
GET    /api/users          # Lấy danh sách
GET    /api/users/123      # Lấy một user
POST   /api/users          # Tạo mới
PUT    /api/users/123      # Cập nhật toàn bộ
PATCH  /api/users/123      # Cập nhật một phần
DELETE /api/users/123      # Xóa

# Filtering, Sorting, Pagination
GET /api/users?status=active&sort=-createdAt&page=1&limit=10
```

---

### Scalability

**Định nghĩa:** Khả năng hệ thống xử lý tăng trưởng workload.

**Vertical Scaling (Scale Up):** Tăng resources của server (CPU, RAM).
**Horizontal Scaling (Scale Out):** Thêm nhiều servers.

| Vertical                | Horizontal             |
| ----------------------- | ---------------------- |
| Đơn giản                | Phức tạp hơn           |
| Có giới hạn             | Gần như không giới hạn |
| Single point of failure | High availability      |

---

### High Availability (HA)

**Định nghĩa:** Hệ thống luôn sẵn sàng hoạt động, minimize downtime.

**Kỹ thuật:**

- Load balancing
- Redundancy (multiple servers)
- Failover
- Health checks
- Geographic distribution

---

### Event-Driven Architecture

**Định nghĩa:** Kiến trúc các components giao tiếp qua events, loose coupling.

```javascript
// Event Producer
eventBus.emit("order.created", { orderId: 123, userId: 456 });

// Event Consumers
eventBus.on("order.created", async (data) => {
  await sendConfirmationEmail(data);
});

eventBus.on("order.created", async (data) => {
  await updateInventory(data);
});
```

---

## 7. Thuật ngữ Testing

### Unit Test

**Định nghĩa:** Test từng đơn vị code nhỏ nhất (function, method) một cách độc lập.

**Đặc điểm:**

- Nhanh
- Isolated (không phụ thuộc external services)
- Deterministic (cùng input → cùng output)

```javascript
// Jest unit test
describe("Calculator", () => {
  test("add should return sum of two numbers", () => {
    expect(add(2, 3)).toBe(5);
  });

  test("divide should throw error when dividing by zero", () => {
    expect(() => divide(10, 0)).toThrow("Cannot divide by zero");
  });
});
```

---

### Integration Test

**Định nghĩa:** Test sự tương tác giữa nhiều components/modules.

```javascript
// Test API endpoint với database
describe("POST /api/users", () => {
  beforeEach(async () => {
    await db.users.deleteMany({});
  });

  test("should create a new user", async () => {
    const response = await request(app).post("/api/users").send({ name: "John", email: "john@example.com" });

    expect(response.status).toBe(201);
    expect(response.body.name).toBe("John");

    const user = await db.users.findOne({ email: "john@example.com" });
    expect(user).toBeTruthy();
  });
});
```

---

### E2E Test (End-to-End)

**Định nghĩa:** Test toàn bộ flow từ UI đến database, mô phỏng user thực.

```javascript
// Playwright E2E test
test("user can login and see dashboard", async ({ page }) => {
  await page.goto("/login");
  await page.fill('[name="email"]', "user@example.com");
  await page.fill('[name="password"]', "password123");
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL("/dashboard");
  await expect(page.locator("h1")).toContainText("Welcome");
});
```

---

### TDD (Test-Driven Development)

**Định nghĩa:** Phương pháp viết test trước, code sau.

**Quy trình Red-Green-Refactor:**

1. **Red:** Viết test → test fail
2. **Green:** Viết code tối thiểu để test pass
3. **Refactor:** Cải thiện code, giữ test pass

```javascript
// 1. Red - Viết test trước
test("should return fizz for multiples of 3", () => {
  expect(fizzBuzz(3)).toBe("Fizz");
  expect(fizzBuzz(9)).toBe("Fizz");
});

// 2. Green - Viết code để pass
function fizzBuzz(n) {
  if (n % 3 === 0) return "Fizz";
  return n;
}

// 3. Refactor - Cải thiện
```

---

### Mocking

**Định nghĩa:** Tạo fake objects thay thế real dependencies trong tests.

**Các loại:**

- **Mock:** Fake object với expectations
- **Stub:** Fake object trả về giá trị cố định
- **Spy:** Wrap real object, track calls

```javascript
// Jest mocking
jest.mock("./emailService");

test("should send welcome email after registration", async () => {
  const mockSendEmail = jest.fn().mockResolvedValue(true);
  emailService.sendEmail = mockSendEmail;

  await registerUser({ email: "test@example.com" });

  expect(mockSendEmail).toHaveBeenCalledWith("test@example.com", "Welcome!");
});
```

---

### Code Coverage

**Định nghĩa:** Phần trăm code được thực thi bởi tests.

**Các metrics:**

- **Line coverage:** % dòng code được chạy
- **Branch coverage:** % nhánh if/else được test
- **Function coverage:** % functions được gọi

```bash
# Jest với coverage
npm test -- --coverage

# Output
----------|---------|----------|---------|---------|
File      | % Stmts | % Branch | % Funcs | % Lines |
----------|---------|----------|---------|---------|
All files |   85.71 |    75.00 |   100   |   85.71 |
----------|---------|----------|---------|---------|
```

---

### Test Pyramid

**Định nghĩa:** Chiến lược phân bổ số lượng tests theo loại.

```
        /\
       /  \      E2E Tests (ít nhất)
      /----\
     /      \    Integration Tests
    /--------\
   /          \  Unit Tests (nhiều nhất)
  /------------\
```

**Tỷ lệ khuyến nghị:** 70% Unit, 20% Integration, 10% E2E

---

## 8. Thuật ngữ Agile & Project Management

### Agile

**Định nghĩa:** Phương pháp phát triển phần mềm linh hoạt, chia nhỏ công việc thành các iterations ngắn.

**Agile Manifesto:**

- Individuals and interactions > processes and tools
- Working software > comprehensive documentation
- Customer collaboration > contract negotiation
- Responding to change > following a plan

---

### Scrum

**Định nghĩa:** Framework Agile phổ biến nhất, chia công việc thành Sprints.

**Các roles:**

- **Product Owner:** Định nghĩa requirements, prioritize backlog
- **Scrum Master:** Facilitate process, remove blockers
- **Development Team:** Build product

**Các events:**

- **Sprint Planning:** Lên kế hoạch sprint
- **Daily Standup:** Meeting 15 phút mỗi ngày
- **Sprint Review:** Demo kết quả
- **Sprint Retrospective:** Đánh giá và cải thiện

---

### Sprint

**Định nghĩa:** Khoảng thời gian cố định (1-4 tuần) để hoàn thành một set công việc.

```
Sprint 1 (2 weeks)
├── Sprint Planning (Day 1)
├── Daily Standups (Every day)
├── Development
├── Sprint Review (Day 14)
└── Retrospective (Day 14)
```

---

### Kanban

**Định nghĩa:** Phương pháp quản lý công việc visual, focus vào flow và WIP limits.

**Kanban Board:**

```
┌──────────┬──────────┬──────────┬──────────┐
│ Backlog  │   To Do  │  Doing   │   Done   │
├──────────┼──────────┼──────────┼──────────┤
│ Task 5   │ Task 3   │ Task 1   │ Task 0   │
│ Task 6   │ Task 4   │ Task 2   │          │
│ Task 7   │          │          │          │
└──────────┴──────────┴──────────┴──────────┘
         WIP Limit: 3
```

---

### User Story

**Định nghĩa:** Mô tả feature từ góc nhìn user, format chuẩn.

**Format:** As a [user type], I want [goal] so that [benefit].

```
As a registered user,
I want to reset my password
so that I can regain access to my account.

Acceptance Criteria:
- User can request password reset via email
- Reset link expires after 24 hours
- User must create password with min 8 characters
```

---

### Epic

**Định nghĩa:** User story lớn, cần chia thành nhiều stories nhỏ hơn.

```
Epic: User Authentication
├── Story: User Registration
├── Story: User Login
├── Story: Password Reset
├── Story: Social Login
└── Story: Two-Factor Authentication
```

---

### Backlog

**Định nghĩa:** Danh sách tất cả công việc cần làm, được prioritize.

**Product Backlog:** Tất cả features, bugs, technical debt
**Sprint Backlog:** Công việc được chọn cho sprint hiện tại

---

### Story Points

**Định nghĩa:** Đơn vị ước lượng độ phức tạp của task, không phải thời gian.

**Fibonacci sequence:** 1, 2, 3, 5, 8, 13, 21...

| Points | Complexity                 |
| ------ | -------------------------- |
| 1      | Rất đơn giản               |
| 2-3    | Đơn giản                   |
| 5      | Trung bình                 |
| 8      | Phức tạp                   |
| 13+    | Rất phức tạp, nên chia nhỏ |

---

### Velocity

**Định nghĩa:** Số story points team hoàn thành trong một sprint.

```
Sprint 1: 20 points
Sprint 2: 25 points
Sprint 3: 22 points
Average Velocity: 22.3 points/sprint
```

---

### Definition of Done (DoD)

**Định nghĩa:** Checklist criteria để xác định task hoàn thành.

```
✅ Code written and reviewed
✅ Unit tests written and passing
✅ Integration tests passing
✅ Documentation updated
✅ Deployed to staging
✅ QA approved
```

---

### Technical Debt

**Định nghĩa:** "Nợ kỹ thuật" - code shortcuts cần refactor sau.

**Ví dụ:**

- Hardcoded values
- Missing tests
- Outdated dependencies
- Poor documentation
- Code duplication

---

### MVP (Minimum Viable Product)

**Định nghĩa:** Phiên bản sản phẩm với features tối thiểu để test với users thực.

**Mục đích:**

- Validate idea nhanh
- Thu thập feedback sớm
- Giảm risk và cost

---

## 9. Viết tắt thường gặp

| Viết tắt   | Đầy đủ                                                                                  | Ý nghĩa                                            |
| ---------- | --------------------------------------------------------------------------------------- | -------------------------------------------------- |
| **API**    | Application Programming Interface                                                       | Giao diện lập trình ứng dụng                       |
| **SDK**    | Software Development Kit                                                                | Bộ công cụ phát triển phần mềm                     |
| **IDE**    | Integrated Development Environment                                                      | Môi trường phát triển tích hợp (VS Code, IntelliJ) |
| **CLI**    | Command Line Interface                                                                  | Giao diện dòng lệnh                                |
| **GUI**    | Graphical User Interface                                                                | Giao diện đồ họa                                   |
| **URL**    | Uniform Resource Locator                                                                | Địa chỉ web                                        |
| **URI**    | Uniform Resource Identifier                                                             | Định danh tài nguyên                               |
| **JSON**   | JavaScript Object Notation                                                              | Format dữ liệu                                     |
| **XML**    | eXtensible Markup Language                                                              | Ngôn ngữ đánh dấu                                  |
| **HTML**   | HyperText Markup Language                                                               | Ngôn ngữ đánh dấu siêu văn bản                     |
| **CSS**    | Cascading Style Sheets                                                                  | Ngôn ngữ định dạng                                 |
| **DOM**    | Document Object Model                                                                   | Mô hình đối tượng tài liệu                         |
| **SQL**    | Structured Query Language                                                               | Ngôn ngữ truy vấn                                  |
| **ORM**    | Object-Relational Mapping                                                               | Ánh xạ đối tượng-quan hệ                           |
| **CRUD**   | Create, Read, Update, Delete                                                            | 4 thao tác cơ bản                                  |
| **REST**   | Representational State Transfer                                                         | Kiến trúc API                                      |
| **HTTP**   | HyperText Transfer Protocol                                                             | Giao thức truyền tải                               |
| **HTTPS**  | HTTP Secure                                                                             | HTTP bảo mật                                       |
| **SSL**    | Secure Sockets Layer                                                                    | Lớp socket bảo mật                                 |
| **TLS**    | Transport Layer Security                                                                | Bảo mật tầng vận chuyển                            |
| **JWT**    | JSON Web Token                                                                          | Token xác thực                                     |
| **OAuth**  | Open Authorization                                                                      | Chuẩn ủy quyền                                     |
| **SSO**    | Single Sign-On                                                                          | Đăng nhập một lần                                  |
| **MFA**    | Multi-Factor Authentication                                                             | Xác thực đa yếu tố                                 |
| **XSS**    | Cross-Site Scripting                                                                    | Tấn công script                                    |
| **CSRF**   | Cross-Site Request Forgery                                                              | Giả mạo request                                    |
| **CORS**   | Cross-Origin Resource Sharing                                                           | Chia sẻ tài nguyên cross-origin                    |
| **CDN**    | Content Delivery Network                                                                | Mạng phân phối nội dung                            |
| **DNS**    | Domain Name System                                                                      | Hệ thống tên miền                                  |
| **VPS**    | Virtual Private Server                                                                  | Server ảo riêng                                    |
| **VM**     | Virtual Machine                                                                         | Máy ảo                                             |
| **CI/CD**  | Continuous Integration/Deployment                                                       | Tích hợp/triển khai liên tục                       |
| **DevOps** | Development + Operations                                                                | Kết hợp dev và ops                                 |
| **SPA**    | Single Page Application                                                                 | Ứng dụng một trang                                 |
| **SSR**    | Server-Side Rendering                                                                   | Render phía server                                 |
| **CSR**    | Client-Side Rendering                                                                   | Render phía client                                 |
| **SSG**    | Static Site Generation                                                                  | Tạo trang tĩnh                                     |
| **PWA**    | Progressive Web App                                                                     | Ứng dụng web tiến bộ                               |
| **SEO**    | Search Engine Optimization                                                              | Tối ưu công cụ tìm kiếm                            |
| **UX**     | User Experience                                                                         | Trải nghiệm người dùng                             |
| **UI**     | User Interface                                                                          | Giao diện người dùng                               |
| **MVP**    | Minimum Viable Product                                                                  | Sản phẩm khả thi tối thiểu                         |
| **POC**    | Proof of Concept                                                                        | Chứng minh khái niệm                               |
| **SaaS**   | Software as a Service                                                                   | Phần mềm dạng dịch vụ                              |
| **PaaS**   | Platform as a Service                                                                   | Nền tảng dạng dịch vụ                              |
| **IaaS**   | Infrastructure as a Service                                                             | Hạ tầng dạng dịch vụ                               |
| **ACID**   | Atomicity, Consistency, Isolation, Durability                                           | Thuộc tính database                                |
| **SOLID**  | Single responsibility, Open-closed, Liskov, Interface segregation, Dependency inversion | Nguyên tắc OOP                                     |
| **DRY**    | Don't Repeat Yourself                                                                   | Không lặp lại code                                 |
| **KISS**   | Keep It Simple, Stupid                                                                  | Giữ đơn giản                                       |
| **YAGNI**  | You Aren't Gonna Need It                                                                | Không làm thừa                                     |
| **TDD**    | Test-Driven Development                                                                 | Phát triển hướng test                              |
| **BDD**    | Behavior-Driven Development                                                             | Phát triển hướng hành vi                           |
| **OOP**    | Object-Oriented Programming                                                             | Lập trình hướng đối tượng                          |
| **FP**     | Functional Programming                                                                  | Lập trình hàm                                      |
| **MVC**    | Model-View-Controller                                                                   | Mô hình MVC                                        |
| **MVVM**   | Model-View-ViewModel                                                                    | Mô hình MVVM                                       |
| **PR**     | Pull Request                                                                            | Yêu cầu merge code                                 |
| **WIP**    | Work In Progress                                                                        | Đang thực hiện                                     |
| **ETA**    | Estimated Time of Arrival                                                               | Thời gian dự kiến hoàn thành                       |
| **ASAP**   | As Soon As Possible                                                                     | Càng sớm càng tốt                                  |
| **FYI**    | For Your Information                                                                    | Để bạn biết                                        |
| **LGTM**   | Looks Good To Me                                                                        | Code review OK                                     |
| **WFH**    | Work From Home                                                                          | Làm việc từ xa                                     |

---

## Tổng kết

Tài liệu này tổng hợp các thuật ngữ IT quan trọng nhất, được phân loại theo chủ đề:

1. **Lập trình cơ bản:** Variables, Functions, Loops, OOP concepts
2. **Web Development:** API, REST, HTTP, DOM, SPA/SSR/SSG
3. **Database:** SQL, NoSQL, CRUD, ACID, ORM
4. **DevOps:** Docker, Kubernetes, CI/CD, Cloud
5. **Security:** Authentication, Authorization, JWT, XSS, CSRF
6. **Architecture:** Microservices, SOLID, Design Patterns
7. **Testing:** Unit Test, Integration Test, E2E, TDD
8. **Agile:** Scrum, Sprint, Kanban, User Story
9. **Viết tắt:** Các từ viết tắt thường gặp trong IT

Hãy bookmark và sử dụng như reference khi cần!
