# Authentication & Security trong Node.js

## 1. Khái niệm cơ bản

### Authentication vs Authorization

| Khái niệm          | Ý nghĩa                       | Ví dụ                    |
| ------------------ | ----------------------------- | ------------------------ |
| **Authentication** | Xác thực "Bạn là ai?"         | Login với email/password |
| **Authorization**  | Phân quyền "Bạn được làm gì?" | Admin mới được xóa user  |

### Các phương thức Authentication

| Phương thức           | Mô tả                   | Use case              |
| --------------------- | ----------------------- | --------------------- |
| **Session-based**     | Lưu session trên server | Web apps truyền thống |
| **Token-based (JWT)** | Token lưu ở client      | APIs, SPAs, Mobile    |
| **OAuth 2.0**         | Đăng nhập qua bên thứ 3 | "Login with Google"   |
| **API Keys**          | Key cố định             | Server-to-server      |

## 2. JWT (JSON Web Token)

### Khái niệm

**JWT** là chuẩn mở (RFC 7519) để truyền thông tin an toàn giữa các bên dưới dạng JSON object được ký số.

### Cấu trúc JWT

```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Header:** Algorithm và token type

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:** Data (claims)

```json
{
  "sub": "1234567890",
  "name": "John",
  "iat": 1516239022,
  "exp": 1516242622
}
```

**Signature:** Verify token không bị sửa đổi

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

### Tại sao dùng JWT?

- **Stateless:** Server không cần lưu session
- **Scalable:** Dễ scale vì không cần shared session store
- **Cross-domain:** Hoạt động tốt với CORS
- **Mobile-friendly:** Phù hợp cho mobile apps

### Implementation

```javascript
const jwt = require("jsonwebtoken");

const SECRET = process.env.JWT_SECRET; // Phải đủ dài và random
const REFRESH_SECRET = process.env.JWT_REFRESH_SECRET;

// Tạo tokens
function generateTokens(user) {
  // Access token - ngắn hạn (15 phút)
  const accessToken = jwt.sign(
    {
      userId: user.id,
      email: user.email,
      role: user.role,
    },
    SECRET,
    { expiresIn: "15m" }
  );

  // Refresh token - dài hạn (7 ngày)
  const refreshToken = jwt.sign({ userId: user.id }, REFRESH_SECRET, { expiresIn: "7d" });

  return { accessToken, refreshToken };
}

// Verify token
function verifyToken(token) {
  try {
    return jwt.verify(token, SECRET);
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      throw new Error("Token đã hết hạn");
    }
    throw new Error("Token không hợp lệ");
  }
}

// Auth middleware
function authMiddleware(req, res, next) {
  // Lấy token từ header
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Không có token" });
  }

  const token = authHeader.split(" ")[1];

  try {
    const decoded = verifyToken(token);
    req.user = decoded; // Gắn user info vào request
    next();
  } catch (error) {
    return res.status(401).json({ error: error.message });
  }
}

// Login endpoint
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  // Tìm user
  const user = await User.findOne({ email });
  if (!user) {
    return res.status(401).json({ error: "Email không tồn tại" });
  }

  // Verify password
  const isValid = await bcrypt.compare(password, user.password);
  if (!isValid) {
    return res.status(401).json({ error: "Mật khẩu không đúng" });
  }

  // Tạo tokens
  const tokens = generateTokens(user);

  // Lưu refresh token vào database (optional nhưng recommended)
  await RefreshToken.create({
    token: tokens.refreshToken,
    userId: user.id,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
  });

  res.json(tokens);
});

// Refresh token endpoint
app.post("/refresh", async (req, res) => {
  const { refreshToken } = req.body;

  if (!refreshToken) {
    return res.status(401).json({ error: "Refresh token required" });
  }

  try {
    // Verify refresh token
    const decoded = jwt.verify(refreshToken, REFRESH_SECRET);

    // Check token trong database
    const storedToken = await RefreshToken.findOne({
      token: refreshToken,
      userId: decoded.userId,
    });

    if (!storedToken) {
      return res.status(401).json({ error: "Invalid refresh token" });
    }

    // Lấy user
    const user = await User.findById(decoded.userId);

    // Tạo tokens mới
    const tokens = generateTokens(user);

    // Xóa refresh token cũ, lưu mới (Token Rotation)
    await RefreshToken.deleteOne({ token: refreshToken });
    await RefreshToken.create({
      token: tokens.refreshToken,
      userId: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
    });

    res.json(tokens);
  } catch (error) {
    return res.status(401).json({ error: "Invalid refresh token" });
  }
});

// Protected route
app.get("/profile", authMiddleware, async (req, res) => {
  const user = await User.findById(req.user.userId);
  res.json({ user });
});

// Logout
app.post("/logout", authMiddleware, async (req, res) => {
  // Xóa refresh token
  await RefreshToken.deleteMany({ userId: req.user.userId });
  res.json({ message: "Logged out" });
});
```

### Access Token vs Refresh Token

| Aspect       | Access Token        | Refresh Token        |
| ------------ | ------------------- | -------------------- |
| **Thời hạn** | Ngắn (15m - 1h)     | Dài (7d - 30d)       |
| **Mục đích** | Access resources    | Lấy access token mới |
| **Lưu trữ**  | Memory/localStorage | httpOnly cookie      |
| **Gửi đi**   | Mỗi request         | Chỉ khi refresh      |

## 3. Password Hashing

### Khái niệm

**Hashing** là quá trình chuyển đổi password thành chuỗi không thể đảo ngược. Không bao giờ lưu password dạng plain text!

### Bcrypt

```javascript
const bcrypt = require("bcrypt");

const SALT_ROUNDS = 12; // Càng cao càng an toàn nhưng chậm hơn

// Hash password
async function hashPassword(password) {
  return bcrypt.hash(password, SALT_ROUNDS);
}

// Verify password
async function verifyPassword(password, hash) {
  return bcrypt.compare(password, hash);
}

// Sử dụng
const hash = await hashPassword("mypassword123");
// $2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/X4.VTtYA...

const isValid = await verifyPassword("mypassword123", hash);
// true
```

### Argon2 (Recommended)

```javascript
const argon2 = require("argon2");

// Hash password
async function hashPassword(password) {
  return argon2.hash(password, {
    type: argon2.argon2id, // Recommended type
    memoryCost: 65536, // 64 MB
    timeCost: 3, // 3 iterations
    parallelism: 4, // 4 threads
  });
}

// Verify password
async function verifyPassword(password, hash) {
  return argon2.verify(hash, password);
}
```

## 4. Security Best Practices

### Helmet.js - Security Headers

```javascript
const helmet = require("helmet");

app.use(helmet()); // Thêm nhiều security headers

// Hoặc configure riêng
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  })
);
```

### Rate Limiting

```javascript
const rateLimit = require("express-rate-limit");

// General rate limit
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 100, // 100 requests per window
  message: "Quá nhiều requests, vui lòng thử lại sau",
});

app.use("/api/", limiter);

// Stricter limit cho auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 giờ
  max: 5, // 5 attempts
  message: "Quá nhiều lần đăng nhập thất bại",
});

app.use("/api/login", authLimiter);
```

### Input Validation

```javascript
const Joi = require("joi");

// Define schema
const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/) // Ít nhất 1 chữ thường, 1 chữ hoa, 1 số
    .required(),
  name: Joi.string().max(100),
});

// Validate
app.post("/register", async (req, res) => {
  const { error, value } = userSchema.validate(req.body);

  if (error) {
    return res.status(400).json({
      error: error.details[0].message,
    });
  }

  // Tiếp tục với validated data
});
```

### SQL Injection Prevention

```javascript
// ❌ NGUY HIỂM: SQL Injection
const query = `SELECT * FROM users WHERE email = '${email}'`;
// Attacker có thể nhập: ' OR '1'='1

// ✅ AN TOÀN: Parameterized query
const query = "SELECT * FROM users WHERE email = $1";
const result = await pool.query(query, [email]);

// ✅ AN TOÀN: ORM (tự động escape)
const user = await User.findOne({ where: { email } });
```

### XSS Prevention

```javascript
const xss = require("xss");

// Sanitize user input
const sanitizedInput = xss(userInput);

// Trong template engines - dùng auto-escaping
// EJS: <%= userInput %> (escaped)
// EJS: <%- userInput %> (unescaped - TRÁNH!)
```

### CORS Configuration

```javascript
const cors = require("cors");

app.use(
  cors({
    origin: ["https://myapp.com", "https://admin.myapp.com"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true, // Cho phép cookies
    maxAge: 86400, // Cache preflight 24h
  })
);
```

## 5. Security Checklist

- [ ] HTTPS everywhere
- [ ] Password hashing (bcrypt/argon2)
- [ ] JWT với expiration ngắn + refresh token
- [ ] Rate limiting
- [ ] Input validation
- [ ] Parameterized queries (tránh SQL injection)
- [ ] Security headers (Helmet)
- [ ] CORS configuration
- [ ] Secure cookie settings
- [ ] Environment variables cho secrets
- [ ] Regular dependency updates

## 6. Tổng kết

**JWT:** Token-based auth, stateless, gồm access + refresh token.

**Password Hashing:** Dùng bcrypt hoặc argon2, không bao giờ lưu plain text.

**Security Headers:** Dùng Helmet.js.

**Rate Limiting:** Giới hạn requests để chống brute force.

**Input Validation:** Validate tất cả user input.

**SQL Injection:** Dùng parameterized queries hoặc ORM.
