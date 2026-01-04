# 📘 Kiến thức về thiết kế RESTful API

## 📖 Mục lục

- [Giới thiệu RESTful API](#giới-thiệu-restful-api)
- [Nguyên tắc REST](#nguyên-tắc-rest)
- [SOLID trong API Design](#solid-trong-api-design)
- [Best Practices](#best-practices)
- [API Versioning](#api-versioning)
- [Security](#security)
- [Documentation](#documentation)
- [Performance Optimization](#performance-optimization)
- [Monitoring & Logging](#monitoring--logging)

---

## 🎯 Giới thiệu RESTful API

### **REST là gì?**

- **REST** (Representational State Transfer) - kiến trúc phần mềm cho hệ thống phân tán
- **RESTful API** là API tuân theo các nguyên tắc REST
- Sử dụng HTTP methods để thực hiện các thao tác CRUD

### **Tại sao dùng REST?**

✅ **Stateless**: Mỗi request chứa đầy đủ thông tin  
✅ **Scalable**: Dễ dàng mở rộng  
✅ **Simple**: Sử dụng các chuẩn HTTP quen thuộc  
✅ **Flexible**: Hỗ trợ nhiều định dạng dữ liệu

---

## 🔧 Nguyên tắc REST

### **1. Client-Server Architecture**

```
Client ↔ REST API ↔ Server
```

- **Tách biệt** giao diện và xử lý dữ liệu
- Server không lưu trữ trạng thái client

### **2. Stateless**

```javascript
// ❌ KHÔNG tốt - Session-based
app.use(session({ secret: "keyboard cat" }));

// ✅ TỐT - Token-based
app.post("/login", (req, res) => {
  const token = jwt.sign({ userId: user.id }, secret);
  res.json({ token });
});
```

### **3. Cacheable**

```http
GET /api/products
Cache-Control: max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

### **4. Uniform Interface**

- **Resource identification**: URI định danh tài nguyên
- **Resource manipulation**: Sử dụng HTTP methods
- **Self-descriptive messages**: Mỗi response chứa đủ thông tin
- **HATEOAS**: Hypermedia as the Engine of Application State

### **5. Layered System**

```
Client → Load Balancer → API Gateway → Microservice → Database
```

### **6. Code on Demand (optional)**

- Server có thể gửi executable code (JavaScript) cho client

---

## 🏗️ SOLID trong API Design

### **S - Single Responsibility Principle**

```javascript
// ❌ VI PHẠM - Nhiều trách nhiệm
app.get("/user/:id", (req, res) => {
  // 1. Xác thực
  // 2. Lấy user từ DB
  // 3. Format response
  // 4. Gửi email thông báo
});

// ✅ TUÂN THỦ - Tách biệt concerns
const authMiddleware = require("./middleware/auth");
const userController = require("./controllers/user");
const emailService = require("./services/email");

app.get("/user/:id", authMiddleware.validateToken, userController.getUser, emailService.notifyAccess);
```

### **O - Open/Closed Principle**

```javascript
// ✅ API có thể mở rộng, không sửa code cũ
class PaymentProcessor {
  process(payment) {
    throw new Error("Must be implemented by subclass");
  }
}

class CreditCardProcessor extends PaymentProcessor {
  process(payment) {
    // Xử lý thẻ tín dụng
  }
}

class PayPalProcessor extends PaymentProcessor {
  process(payment) {
    // Xử lý PayPal
  }
}

// Route handler
app.post("/payment", (req, res) => {
  const processor = PaymentFactory.create(req.body.type);
  processor.process(req.body);
});
```

### **L - Liskov Substitution Principle**

```javascript
// ✅ Các resource con có thể thay thế resource cha
class BaseResource {
  validate() {
    /* logic chung */
  }
  serialize() {
    /* logic chung */
  }
}

class UserResource extends BaseResource {
  validate() {
    super.validate();
    // Thêm validation riêng
  }

  serialize() {
    const base = super.serialize();
    return { ...base, customField: "value" };
  }
}
```

### **I - Interface Segregation Principle**

```javascript
// ❌ VI PHẠM - Interface quá lớn
interface IUserService {
  createUser();
  updateUser();
  deleteUser();
  sendEmail();
  generateReport();
  backupData();
}

// ✅ TUÂN THỦ - Interface nhỏ, tập trung
interface IUserCRUD {
  createUser();
  updateUser();
  deleteUser();
}

interface IUserNotification {
  sendEmail();
}

interface IUserAnalytics {
  generateReport();
}
```

### **D - Dependency Inversion Principle**

```javascript
// ✅ Phụ thuộc vào abstraction, không phải implementation
class UserController {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async getUsers() {
    return await this.userRepository.findAll();
  }
}

// Có thể dễ dàng thay đổi repository
const sqlRepo = new SQLUserRepository();
const mongoRepo = new MongoUserRepository();
const userController = new UserController(mongoRepo);
```

---

## 🚀 Best Practices

### **1. Naming Conventions**

| Resource    | Method    | Endpoint          | Status Code |
| ----------- | --------- | ----------------- | ----------- |
| Users       | GET       | `/api/users`      | 200         |
| Single User | GET       | `/api/users/{id}` | 200, 404    |
| Create User | POST      | `/api/users`      | 201         |
| Update User | PUT/PATCH | `/api/users/{id}` | 200, 404    |
| Delete User | DELETE    | `/api/users/{id}` | 204         |

### **2. HTTP Methods đúng cách**

```http
GET    /api/users           # Lấy danh sách
GET    /api/users/{id}      # Lấy chi tiết
POST   /api/users           # Tạo mới
PUT    /api/users/{id}      # Cập nhật toàn bộ
PATCH  /api/users/{id}      # Cập nhật một phần
DELETE /api/users/{id}      # Xóa
```

### **3. Response Format chuẩn**

```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "metadata": {
    "total": 100,
    "page": 1,
    "limit": 10
  },
  "errors": null,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### **4. Error Handling**

```javascript
// Error handler middleware
app.use((err, req, res, next) => {
  const status = err.status || 500;

  res.status(status).json({
    success: false,
    message: err.message || "Internal Server Error",
    errors: err.errors || null,
    stack: process.env.NODE_ENV === "development" ? err.stack : undefined,
  });
});

// Sử dụng
app.get("/api/users/:id", async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      throw new NotFoundError("User not found");
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

### **5. Status Codes**

| Code | Category     | Meaning            |
| ---- | ------------ | ------------------ |
| 2xx  | Success      | Request thành công |
| 3xx  | Redirection  | Cần thao tác thêm  |
| 4xx  | Client Error | Lỗi từ phía client |
| 5xx  | Server Error | Lỗi từ phía server |

**Chi tiết:**

- `200 OK` - GET, PUT, PATCH thành công
- `201 Created` - POST thành công
- `204 No Content` - DELETE thành công
- `400 Bad Request` - Request không hợp lệ
- `401 Unauthorized` - Chưa xác thực
- `403 Forbidden` - Không có quyền
- `404 Not Found` - Resource không tồn tại
- `429 Too Many Requests` - Rate limit
- `500 Internal Server Error` - Lỗi server

---

## 🔄 API Versioning

### **Các phương pháp versioning**

```http
# 1. URI Versioning (Phổ biến nhất)
GET /api/v1/users
GET /api/v2/users

# 2. Header Versioning
GET /api/users
Accept: application/vnd.company.v1+json

# 3. Query Parameter Versioning
GET /api/users?version=1

# 4. Media Type Versioning
GET /api/users
Accept: application/json; version=1
```

### **Implementation**

```javascript
// Versioning middleware
const versioning = (req, res, next) => {
  const version = req.headers["api-version"] || "v1";
  req.version = version;
  next();
};

// Route với versioning
app.use("/api", versioning, (req, res, next) => {
  if (req.version === "v1") {
    require("./routes/v1")(req, res, next);
  } else if (req.version === "v2") {
    require("./routes/v2")(req, res, next);
  }
});
```

---

## 🔒 Security

### **1. Authentication & Authorization**

```javascript
// JWT Implementation
const jwt = require("jsonwebtoken");

const generateToken = (user) => {
  return jwt.sign(
    {
      userId: user.id,
      role: user.role,
    },
    process.env.JWT_SECRET,
    { expiresIn: "1h" }
  );
};

// Middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: "Unauthorized" });
  }
};
```

### **2. Rate Limiting**

```javascript
const rateLimit = require("express-rate-limit");

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 100, // Giới hạn mỗi IP
  message: {
    error: "Too many requests",
    retryAfter: "15 minutes",
  },
});

app.use("/api/", apiLimiter);
```

### **3. Input Validation**

```javascript
const { body, validationResult } = require("express-validator");

app.post(
  "/api/users",
  [body("email").isEmail().normalizeEmail(), body("password").isLength({ min: 8 }), body("name").trim().notEmpty()],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Process data
  }
);
```

### **4. CORS Configuration**

```javascript
const cors = require("cors");

app.use(
  cors({
    origin: process.env.ALLOWED_ORIGINS.split(","),
    methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,
  })
);
```

---

## 📚 Documentation

### **OpenAPI/Swagger**

```yaml
# swagger.yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /users:
    get:
      summary: Get all users
      responses:
        "200":
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/User"

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
```

### **Auto-generated Documentation**

```javascript
const swaggerJsDoc = require("swagger-jsdoc");
const swaggerUi = require("swagger-ui-express");

const swaggerOptions = {
  definition: {
    openapi: "3.0.0",
    info: {
      title: "API Documentation",
      version: "1.0.0",
    },
  },
  apis: ["./routes/*.js"],
};

const swaggerSpec = swaggerJsDoc(swaggerOptions);
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

---

## ⚡ Performance Optimization

### **1. Pagination**

```javascript
// Query parameters
GET /api/users?page=1&limit=10&sort=createdAt&order=desc

// Implementation
app.get('/api/users', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const [users, total] = await Promise.all([
    User.find().skip(skip).limit(limit),
    User.countDocuments()
  ]);

  res.json({
    data: users,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit)
    }
  });
});
```

### **2. Caching Strategy**

```javascript
const NodeCache = require("node-cache");
const cache = new NodeCache({ stdTTL: 300 });

app.get("/api/products/:id", async (req, res) => {
  const cacheKey = `product_${req.params.id}`;
  const cached = cache.get(cacheKey);

  if (cached) {
    return res.json(cached);
  }

  const product = await Product.findById(req.params.id);
  cache.set(cacheKey, product);
  res.json(product);
});
```

### **3. Compression**

```javascript
const compression = require("compression");
app.use(compression());
```

### **4. Database Optimization**

```javascript
// Use indexes
UserSchema.index({ email: 1 }, { unique: true });
UserSchema.index({ createdAt: -1 });

// Select only needed fields
User.find().select("name email -_id");

// Use lean() for read-only operations
User.find().lean();
```

---

## 📊 Monitoring & Logging

### **1. Structured Logging**

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  format: winston.format.combine(winston.format.timestamp(), winston.format.json()),
  transports: [
    new winston.transports.File({ filename: "error.log", level: "error" }),
    new winston.transports.File({ filename: "combined.log" }),
  ],
});

// Log middleware
app.use((req, res, next) => {
  logger.info({
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.get("User-Agent"),
  });
  next();
});
```

### **2. Health Check**

```javascript
app.get("/health", (req, res) => {
  const health = {
    status: "UP",
    timestamp: new Date().toISOString(),
    services: {
      database: checkDatabase(),
      redis: checkRedis(),
      api: true,
    },
  };

  res.json(health);
});
```

### **3. Metrics & Analytics**

```javascript
const client = require("prom-client");

// Collect default metrics
const collectDefaultMetrics = promClient.collectDefaultMetrics;
collectDefaultMetrics({ timeout: 5000 });

// Custom metric
const httpRequestDurationMicroseconds = new promClient.Histogram({
  name: "http_request_duration_ms",
  help: "Duration of HTTP requests in ms",
  labelNames: ["method", "route", "code"],
});

// Expose metrics
app.get("/metrics", async (req, res) => {
  res.set("Content-Type", promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

---

## 🎯 Checklist khi thiết kế API

### **✅ Phải có**

- [ ] Sử dụng HTTPS
- [ ] Authentication/Authorization
- [ ] Input validation
- [ ] Error handling
- [ ] Rate limiting
- [ ] CORS configuration
- [ ] API versioning
- [ ] Pagination cho list endpoints
- [ ] Structured logging
- [ ] API documentation

### **✅ Nên có**

- [ ] Caching strategy
- [ ] Request/Response compression
- [ ] Health check endpoint
- [ ] Metrics endpoint
- [ ] Request ID tracking
- [ ] API gateway
- [ ] Circuit breaker pattern
- [ ] Retry logic với exponential backoff

### **✅ Tốt để có**

- [ ] HATEOAS
- [ ] GraphQL alternative
- [ ] Webhook support
- [ ] Batch operations
- [ ] Async operations với webhook/callback
- [ ] API testing suite
- [ ] Performance monitoring

---

## 📦 Sample Project Structure

```
src/
├── api/
│   ├── v1/
│   │   ├── controllers/
│   │   ├── routes/
│   │   ├── services/
│   │   └── validators/
│   └── v2/
├── middleware/
│   ├── auth.js
│   ├── errorHandler.js
│   └── validator.js
├── models/
├── utils/
│   ├── logger.js
│   └── constants.js
├── config/
├── tests/
└── app.js
```

---

## 📚 Resources

- [REST API Tutorial](https://restfulapi.net/)
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)
- [Google API Design Guide](https://cloud.google.com/apis/design)
- [OpenAPI Specification](https://swagger.io/specification/)

---

**💡 Lời khuyên cuối cùng:**
_"Thiết kế API không chỉ là technical decision mà còn là product decision. Hãy nghĩ về developer experience khi sử dụng API của bạn."_

---

_Document created: 2024-01-15_  
_Last updated: 2024-01-15_
