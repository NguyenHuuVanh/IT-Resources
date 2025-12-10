# Express.js - Hướng dẫn Toàn diện

## Mục lục

1. [Giới thiệu Express.js](#1-giới-thiệu-expressjs)
2. [Cài đặt và Setup](#2-cài-đặt-và-setup)
3. [Routing](#3-routing)
4. [Middleware](#4-middleware)
5. [Request & Response](#5-request--response)
6. [Template Engines](#6-template-engines)
7. [Static Files](#7-static-files)
8. [Error Handling](#8-error-handling)
9. [Database Integration](#9-database-integration)
10. [Authentication & Authorization](#10-authentication--authorization)
11. [Security Best Practices](#11-security-best-practices)
12. [File Upload](#12-file-upload)
13. [API Development](#13-api-development)
14. [Testing](#14-testing)
15. [Deployment](#15-deployment)
16. [Project Structure](#16-project-structure)

---

## 1. Giới thiệu Express.js

### Express.js là gì?

**Định nghĩa:** Express.js là web framework minimal và flexible cho Node.js, cung cấp các tính năng mạnh mẽ để xây dựng web applications và APIs.

### Tại sao dùng Express.js?

- **Minimal & Flexible:** Không ép buộc cấu trúc, tự do thiết kế
- **Middleware ecosystem:** Hàng ngàn middleware có sẵn
- **Routing mạnh mẽ:** Hỗ trợ dynamic routes, route groups
- **Performance:** Lightweight, nhanh
- **Community lớn:** Documentation tốt, nhiều resources

### So sánh với các frameworks khác

| Feature        | Express      | Fastify  | Koa        | NestJS      |
| -------------- | ------------ | -------- | ---------- | ----------- |
| Performance    | Tốt          | Rất tốt  | Tốt        | Tốt         |
| Learning curve | Thấp         | Thấp     | Trung bình | Cao         |
| Middleware     | Có sẵn nhiều | Plugins  | Ít hơn     | Decorators  |
| TypeScript     | Manual       | Built-in | Manual     | Built-in    |
| Structure      | Flexible     | Flexible | Flexible   | Opinionated |

---

## 2. Cài đặt và Setup

### Cài đặt cơ bản

```bash
# Tạo project
mkdir my-express-app
cd my-express-app
npm init -y

# Cài đặt Express
npm install express

# Cài đặt dev dependencies
npm install -D nodemon
```

### Hello World

```javascript
// app.js
const express = require("express");
const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.send("Hello World!");
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### Setup với ES Modules

```javascript
// package.json
{
  "type": "module"
}

// app.js
import express from 'express';
const app = express();
```

### Setup với TypeScript

```bash
npm install -D typescript @types/node @types/express ts-node
npx tsc --init
```

```typescript
// app.ts
import express, { Application, Request, Response } from "express";

const app: Application = express();
const PORT: number = 3000;

app.get("/", (req: Request, res: Response) => {
  res.send("Hello TypeScript!");
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## 3. Routing

### Basic Routing

```javascript
// HTTP Methods
app.get("/users", (req, res) => {
  res.send("GET - Lấy danh sách users");
});

app.post("/users", (req, res) => {
  res.send("POST - Tạo user mới");
});

app.put("/users/:id", (req, res) => {
  res.send(`PUT - Cập nhật user ${req.params.id}`);
});

app.patch("/users/:id", (req, res) => {
  res.send(`PATCH - Cập nhật một phần user ${req.params.id}`);
});

app.delete("/users/:id", (req, res) => {
  res.send(`DELETE - Xóa user ${req.params.id}`);
});

// Xử lý tất cả methods
app.all("/secret", (req, res) => {
  res.send("Accessing the secret section...");
});
```

### Route Parameters

```javascript
// Single parameter
app.get("/users/:id", (req, res) => {
  const userId = req.params.id;
  res.send(`User ID: ${userId}`);
});

// Multiple parameters
app.get("/users/:userId/posts/:postId", (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Optional parameters (sử dụng regex)
app.get("/users/:id?", (req, res) => {
  if (req.params.id) {
    res.send(`User ID: ${req.params.id}`);
  } else {
    res.send("All users");
  }
});

// Parameter với pattern
app.get("/users/:id(\\d+)", (req, res) => {
  // Chỉ match số
  res.send(`User ID (number only): ${req.params.id}`);
});
```

### Query Strings

```javascript
// GET /search?q=express&page=1&limit=10
app.get("/search", (req, res) => {
  const { q, page = 1, limit = 10 } = req.query;
  res.json({
    query: q,
    page: parseInt(page),
    limit: parseInt(limit),
  });
});
```

### Route Handlers

```javascript
// Multiple callbacks
const validateUser = (req, res, next) => {
  if (!req.body.name) {
    return res.status(400).json({ error: "Name is required" });
  }
  next();
};

const createUser = (req, res) => {
  res.json({ message: "User created", data: req.body });
};

app.post("/users", validateUser, createUser);

// Array of callbacks
app.get("/example", [callback1, callback2, callback3]);
```

### Express Router

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();

// Middleware specific to this router
router.use((req, res, next) => {
  console.log("Users Router - Time:", Date.now());
  next();
});

router.get("/", (req, res) => {
  res.json({ users: [] });
});

router.get("/:id", (req, res) => {
  res.json({ user: { id: req.params.id } });
});

router.post("/", (req, res) => {
  res.status(201).json({ message: "User created" });
});

router.put("/:id", (req, res) => {
  res.json({ message: `User ${req.params.id} updated` });
});

router.delete("/:id", (req, res) => {
  res.json({ message: `User ${req.params.id} deleted` });
});

module.exports = router;

// app.js
const usersRouter = require("./routes/users");
app.use("/api/users", usersRouter);
```

### Route Chaining

```javascript
app
  .route("/books")
  .get((req, res) => {
    res.send("Get all books");
  })
  .post((req, res) => {
    res.send("Add a book");
  })
  .put((req, res) => {
    res.send("Update all books");
  });

app
  .route("/books/:id")
  .get((req, res) => {
    res.send(`Get book ${req.params.id}`);
  })
  .put((req, res) => {
    res.send(`Update book ${req.params.id}`);
  })
  .delete((req, res) => {
    res.send(`Delete book ${req.params.id}`);
  });
```

---

## 4. Middleware

### Middleware là gì?

**Định nghĩa:** Functions có quyền truy cập vào request object (req), response object (res), và next middleware function trong request-response cycle.

```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
```

### Application-level Middleware

```javascript
const express = require("express");
const app = express();

// Middleware không có mount path - chạy với mọi request
app.use((req, res, next) => {
  console.log("Time:", Date.now());
  next();
});

// Middleware với mount path
app.use("/api", (req, res, next) => {
  console.log("API Request");
  next();
});

// Multiple middleware functions
app.use(
  "/user/:id",
  (req, res, next) => {
    console.log("Request URL:", req.originalUrl);
    next();
  },
  (req, res, next) => {
    console.log("Request Type:", req.method);
    next();
  }
);
```

### Router-level Middleware

```javascript
const router = express.Router();

// Middleware cho tất cả routes trong router
router.use((req, res, next) => {
  console.log("Router middleware");
  next();
});

// Middleware cho route cụ thể
router.use("/user/:id", (req, res, next) => {
  console.log("User ID:", req.params.id);
  next();
});
```

### Built-in Middleware

```javascript
// Parse JSON body
app.use(express.json());

// Parse URL-encoded body
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static("public"));

// Serve static với virtual path
app.use("/static", express.static("public"));
```

### Third-party Middleware phổ biến

```javascript
const cors = require("cors");
const helmet = require("helmet");
const morgan = require("morgan");
const compression = require("compression");
const cookieParser = require("cookie-parser");

// CORS - Cross-Origin Resource Sharing
app.use(cors());
app.use(
  cors({
    origin: ["http://localhost:3000", "https://myapp.com"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    credentials: true,
  })
);

// Helmet - Security headers
app.use(helmet());

// Morgan - HTTP request logger
app.use(morgan("dev")); // Development
app.use(morgan("combined")); // Production

// Compression - Gzip compression
app.use(compression());

// Cookie Parser
app.use(cookieParser());
```

### Custom Middleware

```javascript
// Logger middleware
const logger = (req, res, next) => {
  const start = Date.now();

  res.on("finish", () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} ${res.statusCode} - ${duration}ms`);
  });

  next();
};

// Auth middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token) {
    return res.status(401).json({ error: "No token provided" });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: "Invalid token" });
  }
};

// Role-based authorization
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
};

// Sử dụng
app.use(logger);
app.get("/profile", authenticate, (req, res) => {
  res.json({ user: req.user });
});
app.delete("/users/:id", authenticate, authorize("admin"), deleteUser);
```

### Async Middleware

```javascript
// Wrapper để handle async errors
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Sử dụng
app.get(
  "/users",
  asyncHandler(async (req, res) => {
    const users = await User.find();
    res.json(users);
  })
);

// Hoặc dùng express-async-handler
const asyncHandler = require("express-async-handler");

app.get(
  "/users",
  asyncHandler(async (req, res) => {
    const users = await User.find();
    res.json(users);
  })
);
```

---

## 5. Request & Response

### Request Object (req)

```javascript
app.post("/api/users/:id", (req, res) => {
  // URL Parameters
  console.log(req.params); // { id: '123' }
  console.log(req.params.id); // '123'

  // Query String: /api/users/123?sort=name&order=asc
  console.log(req.query); // { sort: 'name', order: 'asc' }
  console.log(req.query.sort); // 'name'

  // Request Body (cần express.json() middleware)
  console.log(req.body); // { name: 'John', email: 'john@example.com' }

  // Headers
  console.log(req.headers); // Tất cả headers
  console.log(req.get("Content-Type")); // 'application/json'
  console.log(req.headers.authorization); // 'Bearer token...'

  // HTTP Method
  console.log(req.method); // 'POST'

  // URL Info
  console.log(req.url); // '/api/users/123?sort=name'
  console.log(req.originalUrl); // '/api/users/123?sort=name'
  console.log(req.path); // '/api/users/123'
  console.log(req.baseUrl); // '/api' (nếu dùng router)
  console.log(req.hostname); // 'localhost'
  console.log(req.protocol); // 'http' hoặc 'https'

  // IP Address
  console.log(req.ip); // '127.0.0.1'
  console.log(req.ips); // ['client', 'proxy1', 'proxy2']

  // Cookies (cần cookie-parser)
  console.log(req.cookies); // { sessionId: 'abc123' }
  console.log(req.signedCookies); // Signed cookies

  // Check content type
  console.log(req.is("json")); // 'json' hoặc false
  console.log(req.is("html")); // false

  // Check if AJAX request
  console.log(req.xhr); // true/false
});
```

### Response Object (res)

```javascript
app.get("/api/example", (req, res) => {
  // Send responses
  res.send("Hello World"); // Text
  res.send({ message: "Hello" }); // JSON (auto)
  res.send(Buffer.from("Hello")); // Buffer

  // JSON response
  res.json({ name: "John", age: 30 });

  // Status code
  res.status(201).json({ message: "Created" });
  res.status(404).json({ error: "Not found" });
  res.status(500).json({ error: "Server error" });

  // Chained status
  res.status(400).send("Bad Request");

  // Send status only
  res.sendStatus(200); // Equivalent to res.status(200).send('OK')
  res.sendStatus(404); // res.status(404).send('Not Found')
});

// Headers
app.get("/api/headers", (req, res) => {
  // Set single header
  res.set("Content-Type", "text/html");
  res.set("X-Custom-Header", "MyValue");

  // Set multiple headers
  res.set({
    "Content-Type": "application/json",
    "X-Powered-By": "Express",
  });

  // Append header
  res.append("Set-Cookie", "foo=bar");

  // Get header
  res.get("Content-Type");

  res.send("Headers set!");
});

// Cookies
app.get("/api/cookies", (req, res) => {
  // Set cookie
  res.cookie("name", "John", {
    maxAge: 900000, // 15 minutes
    httpOnly: true, // Không truy cập được từ JS
    secure: true, // Chỉ gửi qua HTTPS
    sameSite: "strict", // CSRF protection
    signed: true, // Signed cookie
  });

  // Clear cookie
  res.clearCookie("name");

  res.send("Cookie set!");
});

// Redirects
app.get("/old-page", (req, res) => {
  res.redirect("/new-page"); // 302 redirect
  res.redirect(301, "/new-page"); // 301 permanent redirect
  res.redirect("back"); // Redirect to referer
  res.redirect("http://google.com"); // External redirect
});

// File responses
app.get("/download", (req, res) => {
  // Send file
  res.sendFile("/path/to/file.pdf");
  res.sendFile("file.pdf", { root: "./public" });

  // Download file
  res.download("/path/to/file.pdf");
  res.download("/path/to/file.pdf", "custom-name.pdf");

  // Attachment
  res.attachment("file.pdf");
});

// Render template
app.get("/page", (req, res) => {
  res.render("index", { title: "Home", user: req.user });
});

// End response
app.get("/end", (req, res) => {
  res.end();
  res.end("Done");
});

// Content type
app.get("/type", (req, res) => {
  res.type("html"); // text/html
  res.type("json"); // application/json
  res.type("application/json");
  res.send("<h1>Hello</h1>");
});

// Format response based on Accept header
app.get("/format", (req, res) => {
  res.format({
    "text/plain": () => {
      res.send("Hello Text");
    },
    "text/html": () => {
      res.send("<h1>Hello HTML</h1>");
    },
    "application/json": () => {
      res.json({ message: "Hello JSON" });
    },
    default: () => {
      res.status(406).send("Not Acceptable");
    },
  });
});
```

### Response Helpers

```javascript
// Success responses
const sendSuccess = (res, data, statusCode = 200) => {
  res.status(statusCode).json({
    success: true,
    data,
  });
};

// Error responses
const sendError = (res, message, statusCode = 500) => {
  res.status(statusCode).json({
    success: false,
    error: message,
  });
};

// Paginated response
const sendPaginated = (res, data, page, limit, total) => {
  res.json({
    success: true,
    data,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
      hasNext: page * limit < total,
      hasPrev: page > 1,
    },
  });
};

// Sử dụng
app.get("/users", async (req, res) => {
  try {
    const users = await User.find();
    sendSuccess(res, users);
  } catch (error) {
    sendError(res, error.message);
  }
});
```

---

## 6. Template Engines

### EJS (Embedded JavaScript)

```bash
npm install ejs
```

```javascript
// Setup
app.set("view engine", "ejs");
app.set("views", "./views");

// Route
app.get("/", (req, res) => {
  res.render("index", {
    title: "Home Page",
    users: [
      { name: "John", email: "john@example.com" },
      { name: "Jane", email: "jane@example.com" },
    ],
  });
});
```

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
  </head>
  <body>
    <h1><%= title %></h1>

    <!-- Loop -->
    <ul>
      <% users.forEach(user => { %>
      <li><%= user.name %> - <%= user.email %></li>
      <% }); %>
    </ul>

    <!-- Conditionals -->
    <% if (users.length > 0) { %>
    <p>Total users: <%= users.length %></p>
    <% } else { %>
    <p>No users found</p>
    <% } %>

    <!-- Include partial -->
    <%- include('partials/header') %>

    <!-- Unescaped HTML -->
    <%- htmlContent %>
  </body>
</html>
```

### Pug (Jade)

```bash
npm install pug
```

```javascript
app.set("view engine", "pug");
app.set("views", "./views");
```

```pug
//- views/index.pug
doctype html
html
  head
    title= title
  body
    h1= title

    ul
      each user in users
        li #{user.name} - #{user.email}

    if users.length > 0
      p Total users: #{users.length}
    else
      p No users found
```

### Handlebars

```bash
npm install express-handlebars
```

```javascript
const { engine } = require("express-handlebars");

app.engine("handlebars", engine());
app.set("view engine", "handlebars");
app.set("views", "./views");
```

```handlebars
<!-- views/index.handlebars -->

<html>
  <head>
    <title>{{title}}</title>
  </head>
  <body>
    <h1>{{title}}</h1>

    <ul>
      {{#each users}}
        <li>{{this.name}} - {{this.email}}</li>
      {{/each}}
    </ul>

    {{#if users.length}}
      <p>Total users: {{users.length}}</p>
    {{else}}
      <p>No users found</p>
    {{/if}}
  </body>
</html>
```

---

## 7. Static Files

### Serving Static Files

```javascript
// Serve từ thư mục 'public'
app.use(express.static("public"));
// Truy cập: http://localhost:3000/images/logo.png
//           http://localhost:3000/css/style.css
//           http://localhost:3000/js/app.js

// Virtual path prefix
app.use("/static", express.static("public"));
// Truy cập: http://localhost:3000/static/images/logo.png

// Multiple directories
app.use(express.static("public"));
app.use(express.static("files"));
app.use(express.static("uploads"));

// Absolute path (recommended)
const path = require("path");
app.use(express.static(path.join(__dirname, "public")));

// Options
app.use(
  express.static("public", {
    dotfiles: "ignore", // Ẩn dotfiles
    etag: true, // Enable ETag
    extensions: ["html", "htm"],
    index: "index.html", // Default file
    maxAge: "1d", // Cache 1 day
    redirect: true, // Redirect to trailing /
    setHeaders: (res, path) => {
      if (path.endsWith(".html")) {
        res.set("Cache-Control", "no-cache");
      }
    },
  })
);
```

### Project Structure với Static Files

```
project/
├── public/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── app.js
│   ├── images/
│   │   └── logo.png
│   └── index.html
├── uploads/
│   └── user-files/
└── app.js
```

---

## 8. Error Handling

### Basic Error Handling

```javascript
// Synchronous errors - tự động catch
app.get("/sync-error", (req, res) => {
  throw new Error("Something went wrong!");
});

// Async errors - phải manually pass to next()
app.get("/async-error", async (req, res, next) => {
  try {
    const data = await someAsyncOperation();
    res.json(data);
  } catch (error) {
    next(error);
  }
});
```

### Error Handling Middleware

```javascript
// Error handling middleware (4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    success: false,
    error: "Something went wrong!",
  });
});

// Multiple error handlers
app.use(logErrors);
app.use(clientErrorHandler);
app.use(errorHandler);

function logErrors(err, req, res, next) {
  console.error(err.stack);
  next(err);
}

function clientErrorHandler(err, req, res, next) {
  if (req.xhr) {
    res.status(500).json({ error: "Something failed!" });
  } else {
    next(err);
  }
}

function errorHandler(err, req, res, next) {
  res.status(500).render("error", { error: err });
}
```

### Custom Error Classes

```javascript
// errors/AppError.js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith("4") ? "fail" : "error";
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

// errors/index.js
class NotFoundError extends AppError {
  constructor(message = "Resource not found") {
    super(message, 404);
  }
}

class BadRequestError extends AppError {
  constructor(message = "Bad request") {
    super(message, 400);
  }
}

class UnauthorizedError extends AppError {
  constructor(message = "Unauthorized") {
    super(message, 401);
  }
}

class ForbiddenError extends AppError {
  constructor(message = "Forbidden") {
    super(message, 403);
  }
}

class ValidationError extends AppError {
  constructor(errors) {
    super("Validation failed", 400);
    this.errors = errors;
  }
}

module.exports = {
  AppError,
  NotFoundError,
  BadRequestError,
  UnauthorizedError,
  ForbiddenError,
  ValidationError,
};
```

### Global Error Handler

```javascript
// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || "error";

  if (process.env.NODE_ENV === "development") {
    sendErrorDev(err, res);
  } else {
    sendErrorProd(err, res);
  }
};

const sendErrorDev = (err, res) => {
  res.status(err.statusCode).json({
    success: false,
    status: err.status,
    error: err,
    message: err.message,
    stack: err.stack,
  });
};

const sendErrorProd = (err, res) => {
  // Operational, trusted error: send message to client
  if (err.isOperational) {
    res.status(err.statusCode).json({
      success: false,
      status: err.status,
      message: err.message,
    });
  }
  // Programming or unknown error: don't leak details
  else {
    console.error("ERROR 💥", err);
    res.status(500).json({
      success: false,
      status: "error",
      message: "Something went wrong!",
    });
  }
};

module.exports = errorHandler;
```

### Async Handler Wrapper

```javascript
// middleware/asyncHandler.js
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

module.exports = asyncHandler;

// Sử dụng
const asyncHandler = require("./middleware/asyncHandler");
const { NotFoundError } = require("./errors");

app.get(
  "/users/:id",
  asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);

    if (!user) {
      throw new NotFoundError("User not found");
    }

    res.json({ success: true, data: user });
  })
);
```

### 404 Handler

```javascript
// Đặt sau tất cả routes
app.use((req, res, next) => {
  res.status(404).json({
    success: false,
    error: `Cannot ${req.method} ${req.url}`,
  });
});

// Hoặc với custom error
app.use((req, res, next) => {
  next(new NotFoundError(`Cannot ${req.method} ${req.url}`));
});
```

### Validation Error Handling

```javascript
const { validationResult } = require("express-validator");

const validate = (req, res, next) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array().map((err) => ({
        field: err.path,
        message: err.msg,
      })),
    });
  }

  next();
};
```

---

## 9. Database Integration

### MongoDB với Mongoose

```bash
npm install mongoose
```

```javascript
// config/database.js
const mongoose = require("mongoose");

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;

// models/User.js
const mongoose = require("mongoose");
const bcrypt = require("bcryptjs");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, "Name is required"],
      trim: true,
      maxlength: [50, "Name cannot exceed 50 characters"],
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      match: [/^\S+@\S+\.\S+$/, "Please use a valid email"],
    },
    password: {
      type: String,
      required: [true, "Password is required"],
      minlength: [6, "Password must be at least 6 characters"],
      select: false, // Không trả về password mặc định
    },
    role: {
      type: String,
      enum: ["user", "admin"],
      default: "user",
    },
    createdAt: {
      type: Date,
      default: Date.now,
    },
  },
  {
    timestamps: true,
    toJSON: { virtuals: true },
    toObject: { virtuals: true },
  }
);

// Hash password before save
userSchema.pre("save", async function (next) {
  if (!this.isModified("password")) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Compare password method
userSchema.methods.comparePassword = async function (candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// Virtual populate
userSchema.virtual("posts", {
  ref: "Post",
  localField: "_id",
  foreignField: "author",
});

module.exports = mongoose.model("User", userSchema);

// controllers/userController.js
const User = require("../models/User");
const asyncHandler = require("../middleware/asyncHandler");

exports.getUsers = asyncHandler(async (req, res) => {
  const users = await User.find().select("-password");
  res.json({ success: true, count: users.length, data: users });
});

exports.getUser = asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id).populate("posts");
  if (!user) {
    throw new NotFoundError("User not found");
  }
  res.json({ success: true, data: user });
});

exports.createUser = asyncHandler(async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json({ success: true, data: user });
});

exports.updateUser = asyncHandler(async (req, res) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, {
    new: true,
    runValidators: true,
  });
  if (!user) {
    throw new NotFoundError("User not found");
  }
  res.json({ success: true, data: user });
});

exports.deleteUser = asyncHandler(async (req, res) => {
  const user = await User.findByIdAndDelete(req.params.id);
  if (!user) {
    throw new NotFoundError("User not found");
  }
  res.json({ success: true, data: {} });
});
```

### PostgreSQL với Sequelize

```bash
npm install sequelize pg pg-hstore
```

```javascript
// config/database.js
const { Sequelize } = require("sequelize");

const sequelize = new Sequelize(process.env.DATABASE_URL, {
  dialect: "postgres",
  logging: process.env.NODE_ENV === "development" ? console.log : false,
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000,
  },
});

module.exports = sequelize;

// models/User.js
const { DataTypes } = require("sequelize");
const sequelize = require("../config/database");
const bcrypt = require("bcryptjs");

const User = sequelize.define(
  "User",
  {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true,
    },
    name: {
      type: DataTypes.STRING,
      allowNull: false,
      validate: {
        len: [2, 50],
      },
    },
    email: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
      validate: {
        isEmail: true,
      },
    },
    password: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    role: {
      type: DataTypes.ENUM("user", "admin"),
      defaultValue: "user",
    },
  },
  {
    timestamps: true,
    hooks: {
      beforeCreate: async (user) => {
        user.password = await bcrypt.hash(user.password, 12);
      },
    },
  }
);

User.prototype.comparePassword = async function (candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

module.exports = User;
```

### Prisma ORM

```bash
npm install prisma @prisma/client
npx prisma init
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  name      String
  email     String   @unique
  password  String
  role      Role     @default(USER)
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
}

enum Role {
  USER
  ADMIN
}
```

```javascript
// Sử dụng Prisma
const { PrismaClient } = require("@prisma/client");
const prisma = new PrismaClient();

// Create
const user = await prisma.user.create({
  data: {
    name: "John",
    email: "john@example.com",
    password: hashedPassword,
  },
});

// Read
const users = await prisma.user.findMany({
  include: { posts: true },
});

// Update
const updated = await prisma.user.update({
  where: { id: userId },
  data: { name: "Jane" },
});

// Delete
await prisma.user.delete({
  where: { id: userId },
});
```

---

## 10. Authentication & Authorization

### JWT Authentication

```bash
npm install jsonwebtoken bcryptjs
```

```javascript
// config/jwt.js
const jwt = require("jsonwebtoken");

const generateToken = (userId) => {
  return jwt.sign({ id: userId }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRE || "7d",
  });
};

const generateRefreshToken = (userId) => {
  return jwt.sign({ id: userId }, process.env.JWT_REFRESH_SECRET, {
    expiresIn: "30d",
  });
};

module.exports = { generateToken, generateRefreshToken };

// controllers/authController.js
const User = require("../models/User");
const { generateToken, generateRefreshToken } = require("../config/jwt");
const asyncHandler = require("../middleware/asyncHandler");

// Register
exports.register = asyncHandler(async (req, res) => {
  const { name, email, password } = req.body;

  // Check if user exists
  const existingUser = await User.findOne({ email });
  if (existingUser) {
    return res.status(400).json({ error: "Email already registered" });
  }

  // Create user
  const user = await User.create({ name, email, password });

  // Generate tokens
  const token = generateToken(user._id);
  const refreshToken = generateRefreshToken(user._id);

  res.status(201).json({
    success: true,
    data: {
      user: { id: user._id, name: user.name, email: user.email },
      token,
      refreshToken,
    },
  });
});

// Login
exports.login = asyncHandler(async (req, res) => {
  const { email, password } = req.body;

  // Validate input
  if (!email || !password) {
    return res.status(400).json({ error: "Please provide email and password" });
  }

  // Check user
  const user = await User.findOne({ email }).select("+password");
  if (!user || !(await user.comparePassword(password))) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  // Generate tokens
  const token = generateToken(user._id);
  const refreshToken = generateRefreshToken(user._id);

  // Set cookie
  res.cookie("token", token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "strict",
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
  });

  res.json({
    success: true,
    data: {
      user: { id: user._id, name: user.name, email: user.email, role: user.role },
      token,
      refreshToken,
    },
  });
});

// Logout
exports.logout = (req, res) => {
  res.cookie("token", "none", {
    expires: new Date(Date.now() + 10 * 1000),
    httpOnly: true,
  });

  res.json({ success: true, message: "Logged out successfully" });
};

// Refresh Token
exports.refreshToken = asyncHandler(async (req, res) => {
  const { refreshToken } = req.body;

  if (!refreshToken) {
    return res.status(401).json({ error: "Refresh token required" });
  }

  try {
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    const user = await User.findById(decoded.id);

    if (!user) {
      return res.status(401).json({ error: "User not found" });
    }

    const newToken = generateToken(user._id);
    res.json({ success: true, token: newToken });
  } catch (error) {
    res.status(401).json({ error: "Invalid refresh token" });
  }
});

// Get current user
exports.getMe = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user.id);
  res.json({ success: true, data: user });
});
```

### Auth Middleware

```javascript
// middleware/auth.js
const jwt = require("jsonwebtoken");
const User = require("../models/User");
const asyncHandler = require("./asyncHandler");

// Protect routes
exports.protect = asyncHandler(async (req, res, next) => {
  let token;

  // Get token from header or cookie
  if (req.headers.authorization?.startsWith("Bearer")) {
    token = req.headers.authorization.split(" ")[1];
  } else if (req.cookies.token) {
    token = req.cookies.token;
  }

  if (!token) {
    return res.status(401).json({ error: "Not authorized to access this route" });
  }

  try {
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Get user from token
    req.user = await User.findById(decoded.id);

    if (!req.user) {
      return res.status(401).json({ error: "User not found" });
    }

    next();
  } catch (error) {
    return res.status(401).json({ error: "Not authorized to access this route" });
  }
});

// Authorize roles
exports.authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        error: `User role '${req.user.role}' is not authorized to access this route`,
      });
    }
    next();
  };
};

// Sử dụng
const { protect, authorize } = require("./middleware/auth");

app.get("/api/profile", protect, getProfile);
app.delete("/api/users/:id", protect, authorize("admin"), deleteUser);
```

### Passport.js

```bash
npm install passport passport-local passport-jwt
```

```javascript
// config/passport.js
const passport = require("passport");
const LocalStrategy = require("passport-local").Strategy;
const JwtStrategy = require("passport-jwt").Strategy;
const ExtractJwt = require("passport-jwt").ExtractJwt;
const User = require("../models/User");

// Local Strategy
passport.use(
  new LocalStrategy({ usernameField: "email" }, async (email, password, done) => {
    try {
      const user = await User.findOne({ email }).select("+password");

      if (!user) {
        return done(null, false, { message: "Invalid email or password" });
      }

      const isMatch = await user.comparePassword(password);

      if (!isMatch) {
        return done(null, false, { message: "Invalid email or password" });
      }

      return done(null, user);
    } catch (error) {
      return done(error);
    }
  })
);

// JWT Strategy
const jwtOptions = {
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET,
};

passport.use(
  new JwtStrategy(jwtOptions, async (payload, done) => {
    try {
      const user = await User.findById(payload.id);

      if (!user) {
        return done(null, false);
      }

      return done(null, user);
    } catch (error) {
      return done(error, false);
    }
  })
);

module.exports = passport;

// Sử dụng
const passport = require("./config/passport");

app.use(passport.initialize());

// Login route
app.post("/login", (req, res, next) => {
  passport.authenticate("local", { session: false }, (err, user, info) => {
    if (err || !user) {
      return res.status(401).json({ error: info?.message || "Login failed" });
    }

    const token = generateToken(user._id);
    res.json({ success: true, token });
  })(req, res, next);
});

// Protected route
app.get("/profile", passport.authenticate("jwt", { session: false }), (req, res) => {
  res.json({ user: req.user });
});
```

### OAuth với Passport (Google)

```bash
npm install passport-google-oauth20
```

```javascript
const GoogleStrategy = require("passport-google-oauth20").Strategy;

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: "/auth/google/callback",
    },
    async (accessToken, refreshToken, profile, done) => {
      try {
        let user = await User.findOne({ googleId: profile.id });

        if (!user) {
          user = await User.create({
            googleId: profile.id,
            name: profile.displayName,
            email: profile.emails[0].value,
            avatar: profile.photos[0].value,
          });
        }

        done(null, user);
      } catch (error) {
        done(error, null);
      }
    }
  )
);

// Routes
app.get("/auth/google", passport.authenticate("google", { scope: ["profile", "email"] }));

app.get("/auth/google/callback", passport.authenticate("google", { session: false }), (req, res) => {
  const token = generateToken(req.user._id);
  res.redirect(`${process.env.CLIENT_URL}/auth?token=${token}`);
});
```

---

## 11. Security Best Practices

### Helmet - Security Headers

```bash
npm install helmet
```

```javascript
const helmet = require("helmet");

// Basic usage
app.use(helmet());

// Custom configuration
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        imgSrc: ["'self'", "data:", "https:"],
        scriptSrc: ["'self'"],
      },
    },
    crossOriginEmbedderPolicy: false,
  })
);
```

### Rate Limiting

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require("express-rate-limit");

// General rate limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per windowMs
  message: {
    error: "Too many requests, please try again later.",
  },
  standardHeaders: true,
  legacyHeaders: false,
});

app.use(limiter);

// Stricter limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts
  message: {
    error: "Too many login attempts, please try again after an hour",
  },
});

app.use("/api/auth/login", authLimiter);
```

### Input Validation & Sanitization

```bash
npm install express-validator
```

```javascript
const { body, param, query, validationResult } = require("express-validator");

// Validation rules
const userValidation = [
  body("name")
    .trim()
    .notEmpty()
    .withMessage("Name is required")
    .isLength({ min: 2, max: 50 })
    .withMessage("Name must be 2-50 characters"),

  body("email")
    .trim()
    .notEmpty()
    .withMessage("Email is required")
    .isEmail()
    .withMessage("Invalid email format")
    .normalizeEmail(),

  body("password")
    .notEmpty()
    .withMessage("Password is required")
    .isLength({ min: 8 })
    .withMessage("Password must be at least 8 characters")
    .matches(/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])/)
    .withMessage("Password must contain uppercase, lowercase, and number"),

  body("age").optional().isInt({ min: 18, max: 120 }).withMessage("Age must be between 18 and 120"),
];

// Validation middleware
const validate = (req, res, next) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array().map((err) => ({
        field: err.path,
        message: err.msg,
      })),
    });
  }

  next();
};

// Sử dụng
app.post("/api/users", userValidation, validate, createUser);
```

### XSS Protection

```bash
npm install xss-clean
```

```javascript
const xss = require("xss-clean");

// Sanitize user input
app.use(xss());

// Manual sanitization
const sanitizeHtml = require("sanitize-html");

const cleanInput = (dirty) => {
  return sanitizeHtml(dirty, {
    allowedTags: [],
    allowedAttributes: {},
  });
};
```

### SQL Injection Prevention

```javascript
// ❌ Vulnerable
const query = `SELECT * FROM users WHERE email = '${email}'`;

// ✅ Safe - Parameterized queries
const query = "SELECT * FROM users WHERE email = $1";
const result = await pool.query(query, [email]);

// ✅ Safe - ORM (Sequelize, Prisma, Mongoose)
const user = await User.findOne({ where: { email } });
```

### CORS Configuration

```javascript
const cors = require("cors");

// Development
app.use(cors());

// Production
const corsOptions = {
  origin: (origin, callback) => {
    const whitelist = ["https://myapp.com", "https://www.myapp.com", "https://admin.myapp.com"];

    if (!origin || whitelist.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS"));
    }
  },
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization"],
  credentials: true,
  maxAge: 86400, // 24 hours
};

app.use(cors(corsOptions));
```

### Data Sanitization

```bash
npm install express-mongo-sanitize hpp
```

```javascript
const mongoSanitize = require("express-mongo-sanitize");
const hpp = require("hpp");

// Prevent NoSQL injection
app.use(mongoSanitize());

// Prevent HTTP Parameter Pollution
app.use(
  hpp({
    whitelist: ["sort", "fields", "page", "limit"],
  })
);
```

---

## 12. File Upload

### Multer - File Upload

```bash
npm install multer
```

```javascript
const multer = require("multer");
const path = require("path");

// Disk storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + "-" + Math.round(Math.random() * 1e9);
    cb(null, file.fieldname + "-" + uniqueSuffix + path.extname(file.originalname));
  },
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpeg|jpg|png|gif|pdf/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);

  if (extname && mimetype) {
    cb(null, true);
  } else {
    cb(new Error("Only images and PDFs are allowed!"), false);
  }
};

// Multer config
const upload = multer({
  storage: storage,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB
  },
  fileFilter: fileFilter,
});

// Single file upload
app.post("/upload", upload.single("file"), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: "No file uploaded" });
  }

  res.json({
    success: true,
    file: {
      filename: req.file.filename,
      path: req.file.path,
      size: req.file.size,
    },
  });
});

// Multiple files upload
app.post("/upload-multiple", upload.array("files", 5), (req, res) => {
  res.json({
    success: true,
    files: req.files.map((file) => ({
      filename: file.filename,
      path: file.path,
    })),
  });
});

// Multiple fields
const uploadFields = upload.fields([
  { name: "avatar", maxCount: 1 },
  { name: "gallery", maxCount: 8 },
]);

app.post("/upload-fields", uploadFields, (req, res) => {
  res.json({
    avatar: req.files["avatar"],
    gallery: req.files["gallery"],
  });
});
```

### Memory Storage (for cloud upload)

```javascript
const storage = multer.memoryStorage();
const upload = multer({ storage: storage });

// Upload to AWS S3
const AWS = require("aws-sdk");
const s3 = new AWS.S3();

app.post("/upload-s3", upload.single("file"), async (req, res) => {
  const params = {
    Bucket: process.env.AWS_BUCKET_NAME,
    Key: `uploads/${Date.now()}-${req.file.originalname}`,
    Body: req.file.buffer,
    ContentType: req.file.mimetype,
    ACL: "public-read",
  };

  try {
    const result = await s3.upload(params).promise();
    res.json({ success: true, url: result.Location });
  } catch (error) {
    res.status(500).json({ error: "Upload failed" });
  }
});
```

### Image Processing với Sharp

```bash
npm install sharp
```

```javascript
const sharp = require("sharp");

app.post("/upload-image", upload.single("image"), async (req, res) => {
  try {
    const filename = `image-${Date.now()}.webp`;

    await sharp(req.file.buffer)
      .resize(800, 600, { fit: "inside" })
      .webp({ quality: 80 })
      .toFile(`uploads/${filename}`);

    // Create thumbnail
    await sharp(req.file.buffer)
      .resize(200, 200, { fit: "cover" })
      .webp({ quality: 60 })
      .toFile(`uploads/thumb-${filename}`);

    res.json({
      success: true,
      image: filename,
      thumbnail: `thumb-${filename}`,
    });
  } catch (error) {
    res.status(500).json({ error: "Image processing failed" });
  }
});
```

---

## 13. API Development

### RESTful API Structure

```javascript
// routes/api/v1/users.js
const express = require("express");
const router = express.Router();
const { getUsers, getUser, createUser, updateUser, deleteUser } = require("../../../controllers/userController");
const { protect, authorize } = require("../../../middleware/auth");

router.route("/").get(getUsers).post(protect, authorize("admin"), createUser);

router.route("/:id").get(getUser).put(protect, updateUser).delete(protect, authorize("admin"), deleteUser);

module.exports = router;

// app.js
app.use("/api/v1/users", require("./routes/api/v1/users"));
```

### Advanced Filtering, Sorting, Pagination

```javascript
// middleware/advancedResults.js
const advancedResults = (model, populate) => async (req, res, next) => {
  let query;

  // Copy req.query
  const reqQuery = { ...req.query };

  // Fields to exclude
  const removeFields = ["select", "sort", "page", "limit", "search"];
  removeFields.forEach((param) => delete reqQuery[param]);

  // Create query string
  let queryStr = JSON.stringify(reqQuery);

  // Create operators ($gt, $gte, $lt, $lte, $in)
  queryStr = queryStr.replace(/\b(gt|gte|lt|lte|in)\b/g, (match) => `$${match}`);

  // Finding resource
  query = model.find(JSON.parse(queryStr));

  // Search
  if (req.query.search) {
    query = query.find({
      $or: [
        { name: { $regex: req.query.search, $options: "i" } },
        { email: { $regex: req.query.search, $options: "i" } },
      ],
    });
  }

  // Select fields
  if (req.query.select) {
    const fields = req.query.select.split(",").join(" ");
    query = query.select(fields);
  }

  // Sort
  if (req.query.sort) {
    const sortBy = req.query.sort.split(",").join(" ");
    query = query.sort(sortBy);
  } else {
    query = query.sort("-createdAt");
  }

  // Pagination
  const page = parseInt(req.query.page, 10) || 1;
  const limit = parseInt(req.query.limit, 10) || 10;
  const startIndex = (page - 1) * limit;
  const endIndex = page * limit;
  const total = await model.countDocuments(JSON.parse(queryStr));

  query = query.skip(startIndex).limit(limit);

  // Populate
  if (populate) {
    query = query.populate(populate);
  }

  // Execute query
  const results = await query;

  // Pagination result
  const pagination = {
    page,
    limit,
    total,
    totalPages: Math.ceil(total / limit),
  };

  if (endIndex < total) {
    pagination.next = { page: page + 1, limit };
  }

  if (startIndex > 0) {
    pagination.prev = { page: page - 1, limit };
  }

  res.advancedResults = {
    success: true,
    count: results.length,
    pagination,
    data: results,
  };

  next();
};

module.exports = advancedResults;

// Sử dụng
const advancedResults = require("../middleware/advancedResults");
const User = require("../models/User");

router.get("/", advancedResults(User, "posts"), (req, res) => {
  res.json(res.advancedResults);
});

// API calls:
// GET /api/users?name=John
// GET /api/users?age[gte]=18&age[lte]=30
// GET /api/users?select=name,email
// GET /api/users?sort=-createdAt,name
// GET /api/users?page=2&limit=10
// GET /api/users?search=john
```

### API Versioning

```javascript
// routes/api/v1/index.js
const router = require("express").Router();

router.use("/users", require("./users"));
router.use("/posts", require("./posts"));
router.use("/auth", require("./auth"));

module.exports = router;

// routes/api/v2/index.js
const router = require("express").Router();

router.use("/users", require("./users"));
// New features in v2...

module.exports = router;

// app.js
app.use("/api/v1", require("./routes/api/v1"));
app.use("/api/v2", require("./routes/api/v2"));
```

### API Documentation với Swagger

```bash
npm install swagger-jsdoc swagger-ui-express
```

```javascript
const swaggerJsdoc = require("swagger-jsdoc");
const swaggerUi = require("swagger-ui-express");

const swaggerOptions = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "My API",
      version: "1.0.0",
      description: "API Documentation",
    },
    servers: [{ url: "http://localhost:3000/api/v1" }],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: "http",
          scheme: "bearer",
          bearerFormat: "JWT",
        },
      },
    },
  },
  apis: ["./routes/*.js", "./models/*.js"],
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);

app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerSpec));

// Route documentation
/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/User'
 */
router.get("/", getUsers);
```

---

## 14. Testing

### Jest Setup

```bash
npm install -D jest supertest
```

```javascript
// package.json
{
  "scripts": {
    "test": "jest --coverage",
    "test:watch": "jest --watch"
  },
  "jest": {
    "testEnvironment": "node",
    "coveragePathIgnorePatterns": ["/node_modules/"],
    "setupFilesAfterEnv": ["./tests/setup.js"]
  }
}

// tests/setup.js
const mongoose = require('mongoose');

beforeAll(async () => {
  await mongoose.connect(process.env.MONGODB_TEST_URI);
});

afterAll(async () => {
  await mongoose.connection.close();
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});
```

### Unit Tests

```javascript
// tests/unit/user.test.js
const User = require("../../models/User");

describe("User Model", () => {
  it("should hash password before saving", async () => {
    const user = new User({
      name: "Test User",
      email: "test@example.com",
      password: "password123",
    });

    await user.save();

    expect(user.password).not.toBe("password123");
    expect(user.password).toMatch(/^\$2[ab]\$/);
  });

  it("should compare password correctly", async () => {
    const user = await User.create({
      name: "Test User",
      email: "test@example.com",
      password: "password123",
    });

    const isMatch = await user.comparePassword("password123");
    const isNotMatch = await user.comparePassword("wrongpassword");

    expect(isMatch).toBe(true);
    expect(isNotMatch).toBe(false);
  });
});
```

### Integration Tests

```javascript
// tests/integration/auth.test.js
const request = require("supertest");
const app = require("../../app");
const User = require("../../models/User");

describe("Auth Endpoints", () => {
  describe("POST /api/auth/register", () => {
    it("should register a new user", async () => {
      const res = await request(app).post("/api/auth/register").send({
        name: "Test User",
        email: "test@example.com",
        password: "password123",
      });

      expect(res.statusCode).toBe(201);
      expect(res.body.success).toBe(true);
      expect(res.body.data).toHaveProperty("token");
      expect(res.body.data.user.email).toBe("test@example.com");
    });

    it("should not register with existing email", async () => {
      await User.create({
        name: "Existing User",
        email: "test@example.com",
        password: "password123",
      });

      const res = await request(app).post("/api/auth/register").send({
        name: "Test User",
        email: "test@example.com",
        password: "password123",
      });

      expect(res.statusCode).toBe(400);
      expect(res.body.error).toBe("Email already registered");
    });
  });

  describe("POST /api/auth/login", () => {
    beforeEach(async () => {
      await User.create({
        name: "Test User",
        email: "test@example.com",
        password: "password123",
      });
    });

    it("should login with valid credentials", async () => {
      const res = await request(app).post("/api/auth/login").send({
        email: "test@example.com",
        password: "password123",
      });

      expect(res.statusCode).toBe(200);
      expect(res.body.success).toBe(true);
      expect(res.body.data).toHaveProperty("token");
    });

    it("should not login with invalid password", async () => {
      const res = await request(app).post("/api/auth/login").send({
        email: "test@example.com",
        password: "wrongpassword",
      });

      expect(res.statusCode).toBe(401);
    });
  });
});
```

### Testing Protected Routes

```javascript
// tests/integration/users.test.js
const request = require("supertest");
const app = require("../../app");
const User = require("../../models/User");
const { generateToken } = require("../../config/jwt");

describe("User Endpoints", () => {
  let token;
  let adminToken;

  beforeEach(async () => {
    const user = await User.create({
      name: "User",
      email: "user@example.com",
      password: "password123",
      role: "user",
    });

    const admin = await User.create({
      name: "Admin",
      email: "admin@example.com",
      password: "password123",
      role: "admin",
    });

    token = generateToken(user._id);
    adminToken = generateToken(admin._id);
  });

  describe("GET /api/users", () => {
    it("should get all users", async () => {
      const res = await request(app).get("/api/users").set("Authorization", `Bearer ${token}`);

      expect(res.statusCode).toBe(200);
      expect(res.body.data).toHaveLength(2);
    });
  });

  describe("DELETE /api/users/:id", () => {
    it("should delete user if admin", async () => {
      const user = await User.findOne({ email: "user@example.com" });

      const res = await request(app).delete(`/api/users/${user._id}`).set("Authorization", `Bearer ${adminToken}`);

      expect(res.statusCode).toBe(200);
    });

    it("should not delete user if not admin", async () => {
      const user = await User.findOne({ email: "user@example.com" });

      const res = await request(app).delete(`/api/users/${user._id}`).set("Authorization", `Bearer ${token}`);

      expect(res.statusCode).toBe(403);
    });
  });
});
```

---

## 15. Deployment

### Environment Configuration

```javascript
// config/config.js
require("dotenv").config();

module.exports = {
  env: process.env.NODE_ENV || "development",
  port: process.env.PORT || 3000,

  db: {
    uri: process.env.MONGODB_URI,
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    },
  },

  jwt: {
    secret: process.env.JWT_SECRET,
    expire: process.env.JWT_EXPIRE || "7d",
    refreshSecret: process.env.JWT_REFRESH_SECRET,
    refreshExpire: "30d",
  },

  cors: {
    origin: process.env.CORS_ORIGIN?.split(",") || "*",
  },

  rateLimit: {
    windowMs: 15 * 60 * 1000,
    max: process.env.RATE_LIMIT_MAX || 100,
  },
};
```

```bash
# .env.example
NODE_ENV=development
PORT=3000

# Database
MONGODB_URI=mongodb://localhost:27017/myapp

# JWT
JWT_SECRET=your-super-secret-key
JWT_EXPIRE=7d
JWT_REFRESH_SECRET=your-refresh-secret

# CORS
CORS_ORIGIN=http://localhost:3000,https://myapp.com

# Rate Limiting
RATE_LIMIT_MAX=100
```

### Production Setup

```javascript
// app.js
const express = require("express");
const helmet = require("helmet");
const cors = require("cors");
const compression = require("compression");
const morgan = require("morgan");
const rateLimit = require("express-rate-limit");
const config = require("./config/config");

const app = express();

// Trust proxy (for rate limiting behind reverse proxy)
if (config.env === "production") {
  app.set("trust proxy", 1);
}

// Security middleware
app.use(helmet());
app.use(cors(config.cors));

// Compression
app.use(compression());

// Rate limiting
app.use(rateLimit(config.rateLimit));

// Logging
if (config.env === "development") {
  app.use(morgan("dev"));
} else {
  app.use(morgan("combined"));
}

// Body parser
app.use(express.json({ limit: "10kb" }));
app.use(express.urlencoded({ extended: true, limit: "10kb" }));

// Routes
app.use("/api/v1", require("./routes/api/v1"));

// Error handling
app.use(require("./middleware/errorHandler"));

module.exports = app;

// server.js
const app = require("./app");
const config = require("./config/config");
const connectDB = require("./config/database");

// Handle uncaught exceptions
process.on("uncaughtException", (err) => {
  console.error("UNCAUGHT EXCEPTION! Shutting down...");
  console.error(err.name, err.message);
  process.exit(1);
});

// Connect to database
connectDB();

const server = app.listen(config.port, () => {
  console.log(`Server running in ${config.env} mode on port ${config.port}`);
});

// Handle unhandled promise rejections
process.on("unhandledRejection", (err) => {
  console.error("UNHANDLED REJECTION! Shutting down...");
  console.error(err.name, err.message);
  server.close(() => {
    process.exit(1);
  });
});

// Graceful shutdown
process.on("SIGTERM", () => {
  console.log("SIGTERM received. Shutting down gracefully...");
  server.close(() => {
    console.log("Process terminated!");
  });
});
```

### Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy source
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000

CMD ["node", "server.js"]
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
      - MONGODB_URI=mongodb://mongo:27017/myapp
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - mongo
    restart: unless-stopped

  mongo:
    image: mongo:6
    volumes:
      - mongo_data:/data/db
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    restart: unless-stopped

volumes:
  mongo_data:
```

### PM2 Process Manager

```bash
npm install -g pm2
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "my-express-app",
      script: "server.js",
      instances: "max",
      exec_mode: "cluster",
      autorestart: true,
      watch: false,
      max_memory_restart: "1G",
      env: {
        NODE_ENV: "development",
      },
      env_production: {
        NODE_ENV: "production",
      },
    },
  ],
};
```

```bash
# Start
pm2 start ecosystem.config.js --env production

# Commands
pm2 list                    # List processes
pm2 logs                    # View logs
pm2 monit                   # Monitor
pm2 restart all             # Restart all
pm2 reload all              # Zero-downtime reload
pm2 stop all                # Stop all
pm2 delete all              # Delete all

# Startup script
pm2 startup
pm2 save
```

### Nginx Reverse Proxy

```nginx
# /etc/nginx/sites-available/myapp
upstream nodejs {
    server 127.0.0.1:3000;
    keepalive 64;
}

server {
    listen 80;
    server_name myapp.com www.myapp.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name myapp.com www.myapp.com;

    ssl_certificate /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;

    # SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Gzip
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    location / {
        proxy_pass http://nodejs;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Static files
    location /static {
        alias /var/www/myapp/public;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## 16. Project Structure

### Basic Structure

```
my-express-app/
├── config/
│   ├── config.js
│   ├── database.js
│   └── passport.js
├── controllers/
│   ├── authController.js
│   └── userController.js
├── middleware/
│   ├── auth.js
│   ├── errorHandler.js
│   ├── asyncHandler.js
│   └── validate.js
├── models/
│   ├── User.js
│   └── Post.js
├── routes/
│   ├── api/
│   │   └── v1/
│   │       ├── index.js
│   │       ├── auth.js
│   │       └── users.js
│   └── index.js
├── services/
│   ├── emailService.js
│   └── uploadService.js
├── utils/
│   ├── helpers.js
│   └── validators.js
├── errors/
│   └── AppError.js
├── tests/
│   ├── unit/
│   ├── integration/
│   └── setup.js
├── public/
│   ├── css/
│   ├── js/
│   └── images/
├── uploads/
├── .env
├── .env.example
├── .gitignore
├── app.js
├── server.js
├── package.json
└── README.md
```

### Advanced Structure (Feature-based)

```
my-express-app/
├── src/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── auth.controller.js
│   │   │   ├── auth.routes.js
│   │   │   ├── auth.service.js
│   │   │   ├── auth.validation.js
│   │   │   └── auth.test.js
│   │   ├── users/
│   │   │   ├── user.controller.js
│   │   │   ├── user.model.js
│   │   │   ├── user.routes.js
│   │   │   ├── user.service.js
│   │   │   ├── user.validation.js
│   │   │   └── user.test.js
│   │   └── posts/
│   │       └── ...
│   ├── common/
│   │   ├── middleware/
│   │   ├── utils/
│   │   └── errors/
│   ├── config/
│   ├── app.js
│   └── server.js
├── tests/
├── docs/
├── scripts/
└── ...
```

### Complete Example App

```javascript
// app.js
const express = require("express");
const helmet = require("helmet");
const cors = require("cors");
const compression = require("compression");
const morgan = require("morgan");
const rateLimit = require("express-rate-limit");
const mongoSanitize = require("express-mongo-sanitize");
const hpp = require("hpp");
const cookieParser = require("cookie-parser");

const config = require("./config/config");
const errorHandler = require("./middleware/errorHandler");
const { NotFoundError } = require("./errors");

const app = express();

// Trust proxy
if (config.env === "production") {
  app.set("trust proxy", 1);
}

// Security
app.use(helmet());
app.use(cors(config.cors));
app.use(mongoSanitize());
app.use(hpp());

// Rate limiting
app.use(
  "/api",
  rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    message: { error: "Too many requests" },
  })
);

// Compression
app.use(compression());

// Logging
app.use(morgan(config.env === "development" ? "dev" : "combined"));

// Body parser
app.use(express.json({ limit: "10kb" }));
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());

// Static files
app.use(express.static("public"));

// Health check
app.get("/health", (req, res) => {
  res.json({ status: "ok", timestamp: new Date().toISOString() });
});

// API routes
app.use("/api/v1", require("./routes/api/v1"));

// 404 handler
app.use((req, res, next) => {
  next(new NotFoundError(`Cannot ${req.method} ${req.url}`));
});

// Error handler
app.use(errorHandler);

module.exports = app;
```

---

## Tổng kết

Express.js là framework mạnh mẽ và linh hoạt cho Node.js. Các điểm chính cần nhớ:

1. **Routing:** Sử dụng Router để tổ chức routes theo modules
2. **Middleware:** Hiểu rõ middleware chain và thứ tự thực thi
3. **Error Handling:** Luôn có global error handler và async wrapper
4. **Security:** Sử dụng helmet, cors, rate limiting, input validation
5. **Authentication:** JWT hoặc session-based, kết hợp với Passport.js
6. **Database:** Sử dụng ORM/ODM (Mongoose, Sequelize, Prisma)
7. **Testing:** Unit tests và integration tests với Jest + Supertest
8. **Deployment:** Docker, PM2, Nginx reverse proxy

### Tài liệu tham khảo

- [Express.js Official Documentation](https://expressjs.com/)
- [Express.js GitHub](https://github.com/expressjs/express)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
