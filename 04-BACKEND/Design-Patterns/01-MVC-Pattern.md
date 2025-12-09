# MVC Pattern (Model - View - Controller)

## 1. Khái niệm

**MVC (Model-View-Controller)** là một mô hình kiến trúc phần mềm chia ứng dụng thành 3 thành phần riêng biệt, mỗi thành phần đảm nhận một nhiệm vụ cụ thể. Đây là pattern phổ biến nhất trong phát triển web.

### Tại sao cần MVC?

Trước khi có MVC, code thường được viết lẫn lộn:

- Logic xử lý dữ liệu
- Logic hiển thị giao diện
- Logic điều khiển luồng chương trình

Điều này dẫn đến:

- Code khó đọc, khó bảo trì
- Khó test từng phần riêng lẻ
- Thay đổi 1 chỗ ảnh hưởng nhiều chỗ khác
- Khó làm việc nhóm

**MVC giải quyết bằng cách tách biệt các concerns (Separation of Concerns).**

## 2. Các thành phần

```
        User Request
             │
             ▼
      ┌─────────────┐
      │  Controller │ ← Nhận request, điều phối
      └──────┬──────┘
             │
     ┌───────┴───────┐
     ▼               ▼
┌─────────┐    ┌─────────┐
│  Model  │───►│  View   │
│ (Data)  │    │  (UI)   │
└─────────┘    └─────────┘
                    │
                    ▼
              User Response
```

### Model - "Dữ liệu và Logic nghiệp vụ"

**Khái niệm:** Model đại diện cho dữ liệu và các quy tắc nghiệp vụ của ứng dụng.

**Nhiệm vụ:**

- Quản lý dữ liệu (CRUD operations)
- Chứa business logic (validation, calculations)
- Tương tác với database
- Không biết gì về View và Controller

**Tác dụng:**

- Tập trung logic dữ liệu một chỗ
- Dễ thay đổi database mà không ảnh hưởng phần khác
- Có thể reuse ở nhiều nơi

```javascript
// models/User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: [true, "Email là bắt buộc"],
    unique: true,
    lowercase: true,
  },
  name: {
    type: String,
    required: [true, "Tên là bắt buộc"],
  },
  password: {
    type: String,
    required: true,
    minlength: 6,
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
});

// Business logic trong Model
userSchema.methods.isAdmin = function () {
  return this.role === "admin";
};

userSchema.methods.canEdit = function (resource) {
  return this.isAdmin() || resource.userId === this.id;
};

userSchema.statics.findByEmail = function (email) {
  return this.findOne({ email: email.toLowerCase() });
};

// Middleware - hash password trước khi save
userSchema.pre("save", async function (next) {
  if (this.isModified("password")) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

module.exports = mongoose.model("User", userSchema);
```

### View - "Giao diện người dùng"

**Khái niệm:** View chịu trách nhiệm hiển thị dữ liệu cho người dùng.

**Nhiệm vụ:**

- Render giao diện (HTML, JSON, XML...)
- Nhận dữ liệu từ Controller để hiển thị
- Không chứa business logic
- Không truy cập trực tiếp database

**Tác dụng:**

- Tách biệt phần hiển thị
- Dễ thay đổi giao diện mà không ảnh hưởng logic
- Designer có thể làm việc độc lập

```ejs
<!-- views/users/index.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Danh sách Users</title>
</head>
<body>
  <h1>Danh sách người dùng</h1>

  <!-- View chỉ hiển thị dữ liệu được truyền vào -->
  <% if (users.length === 0) { %>
    <p>Chưa có người dùng nào.</p>
  <% } else { %>
    <table>
      <thead>
        <tr>
          <th>Tên</th>
          <th>Email</th>
          <th>Vai trò</th>
          <th>Thao tác</th>
        </tr>
      </thead>
      <tbody>
        <% users.forEach(user => { %>
          <tr>
            <td><%= user.name %></td>
            <td><%= user.email %></td>
            <td><%= user.role %></td>
            <td>
              <a href="/users/<%= user.id %>">Xem</a>
              <a href="/users/<%= user.id %>/edit">Sửa</a>
            </td>
          </tr>
        <% }) %>
      </tbody>
    </table>
  <% } %>

  <a href="/users/new">Thêm người dùng mới</a>

  <!-- Hiển thị thông báo nếu có -->
  <% if (message) { %>
    <div class="alert"><%= message %></div>
  <% } %>
</body>
</html>
```

### Controller - "Điều phối viên"

**Khái niệm:** Controller là cầu nối giữa Model và View, xử lý các request từ người dùng.

**Nhiệm vụ:**

- Nhận request từ user (qua routes)
- Gọi Model để lấy/xử lý dữ liệu
- Chọn View phù hợp để render response
- Xử lý input validation cơ bản
- Điều hướng (redirect)

**Tác dụng:**

- Điều phối luồng xử lý
- Kết nối Model và View
- Tập trung xử lý HTTP request/response

```javascript
// controllers/userController.js
const User = require("../models/User");

const userController = {
  // GET /users - Hiển thị danh sách
  async index(req, res) {
    try {
      // 1. Gọi Model lấy dữ liệu
      const users = await User.find().sort({ createdAt: -1 });

      // 2. Truyền dữ liệu cho View
      res.render("users/index", {
        users,
        message: req.flash("message"),
      });
    } catch (error) {
      res.status(500).render("error", { message: "Lỗi server" });
    }
  },

  // GET /users/:id - Xem chi tiết
  async show(req, res) {
    try {
      const user = await User.findById(req.params.id);

      if (!user) {
        return res.status(404).render("error", {
          message: "Không tìm thấy người dùng",
        });
      }

      res.render("users/show", { user });
    } catch (error) {
      res.status(500).render("error", { message: "Lỗi server" });
    }
  },

  // GET /users/new - Form tạo mới
  new(req, res) {
    res.render("users/new", { errors: [] });
  },

  // POST /users - Xử lý tạo mới
  async create(req, res) {
    try {
      const { email, name, password } = req.body;

      // Tạo user mới qua Model
      const user = new User({ email, name, password });
      await user.save();

      // Redirect sau khi tạo thành công
      req.flash("message", "Tạo người dùng thành công!");
      res.redirect(`/users/${user.id}`);
    } catch (error) {
      // Validation error - render lại form với lỗi
      res.status(400).render("users/new", {
        errors: [error.message],
        data: req.body,
      });
    }
  },

  // GET /users/:id/edit - Form chỉnh sửa
  async edit(req, res) {
    try {
      const user = await User.findById(req.params.id);

      if (!user) {
        return res.status(404).render("error", {
          message: "Không tìm thấy người dùng",
        });
      }

      res.render("users/edit", { user, errors: [] });
    } catch (error) {
      res.status(500).render("error", { message: "Lỗi server" });
    }
  },

  // PUT /users/:id - Xử lý cập nhật
  async update(req, res) {
    try {
      const { name, email } = req.body;

      const user = await User.findByIdAndUpdate(req.params.id, { name, email }, { new: true, runValidators: true });

      if (!user) {
        return res.status(404).render("error", {
          message: "Không tìm thấy người dùng",
        });
      }

      req.flash("message", "Cập nhật thành công!");
      res.redirect(`/users/${user.id}`);
    } catch (error) {
      res.status(400).render("users/edit", {
        errors: [error.message],
      });
    }
  },

  // DELETE /users/:id - Xóa
  async destroy(req, res) {
    try {
      await User.findByIdAndDelete(req.params.id);

      req.flash("message", "Đã xóa người dùng");
      res.redirect("/users");
    } catch (error) {
      res.status(500).render("error", { message: "Lỗi server" });
    }
  },
};

module.exports = userController;
```

## 3. Luồng hoạt động

```
1. User gửi request: GET /users/123

2. Router nhận request, chuyển đến Controller:
   router.get('/users/:id', userController.show)

3. Controller xử lý:
   - Lấy id từ params
   - Gọi Model: User.findById(id)
   - Nhận data từ Model
   - Gọi View: res.render('users/show', { user })

4. View render HTML với data

5. Response trả về cho User
```

## 4. MVC cho REST API

Khi làm API, View được thay bằng JSON response:

```javascript
// controllers/api/userController.js
const userApiController = {
  // GET /api/users
  async index(req, res) {
    try {
      const { page = 1, limit = 10, search } = req.query;

      // Build query
      const query = {};
      if (search) {
        query.name = { $regex: search, $options: "i" };
      }

      // Lấy data từ Model
      const users = await User.find(query)
        .select("-password") // Không trả về password
        .limit(limit)
        .skip((page - 1) * limit)
        .sort({ createdAt: -1 });

      const total = await User.countDocuments(query);

      // Trả về JSON thay vì render View
      res.json({
        success: true,
        data: users,
        pagination: {
          page: parseInt(page),
          limit: parseInt(limit),
          total,
          totalPages: Math.ceil(total / limit),
        },
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        error: error.message,
      });
    }
  },

  // POST /api/users
  async create(req, res) {
    try {
      const user = new User(req.body);
      await user.save();

      // Không trả về password
      const userResponse = user.toObject();
      delete userResponse.password;

      res.status(201).json({
        success: true,
        data: userResponse,
      });
    } catch (error) {
      res.status(400).json({
        success: false,
        error: error.message,
      });
    }
  },
};
```

## 5. Cấu trúc thư mục MVC

```
project/
├── app.js                 # Entry point
├── config/
│   ├── database.js        # Database connection
│   └── config.js          # App configuration
├── controllers/
│   ├── userController.js
│   ├── productController.js
│   └── api/
│       └── userController.js
├── models/
│   ├── User.js
│   └── Product.js
├── views/
│   ├── layouts/
│   │   └── main.ejs       # Layout chung
│   ├── users/
│   │   ├── index.ejs
│   │   ├── show.ejs
│   │   ├── new.ejs
│   │   └── edit.ejs
│   ├── partials/
│   │   ├── header.ejs
│   │   └── footer.ejs
│   └── error.ejs
├── routes/
│   ├── index.js
│   ├── users.js
│   └── api.js
├── public/                # Static files
│   ├── css/
│   ├── js/
│   └── images/
├── middleware/
│   ├── auth.js
│   └── errorHandler.js
└── package.json
```

## 6. Routes - Kết nối URL với Controller

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();
const userController = require("../controllers/userController");
const { isAuthenticated, isAdmin } = require("../middleware/auth");

// Public routes
router.get("/", userController.index);
router.get("/:id", userController.show);

// Protected routes - cần đăng nhập
router.get("/new", isAuthenticated, userController.new);
router.post("/", isAuthenticated, userController.create);
router.get("/:id/edit", isAuthenticated, userController.edit);
router.put("/:id", isAuthenticated, userController.update);
router.delete("/:id", isAuthenticated, isAdmin, userController.destroy);

module.exports = router;

// app.js
const userRoutes = require("./routes/users");
app.use("/users", userRoutes);
```

## 7. Ưu điểm của MVC

| Ưu điểm                    | Giải thích                                      |
| -------------------------- | ----------------------------------------------- |
| **Separation of Concerns** | Mỗi thành phần có nhiệm vụ riêng, code rõ ràng  |
| **Dễ bảo trì**             | Thay đổi 1 phần không ảnh hưởng phần khác       |
| **Dễ test**                | Có thể test Model độc lập                       |
| **Làm việc nhóm**          | Frontend làm View, Backend làm Model/Controller |
| **Reusable**               | Model có thể dùng ở nhiều Controller            |
| **Phổ biến**               | Nhiều framework hỗ trợ, tài liệu phong phú      |

## 8. Nhược điểm của MVC

| Nhược điểm         | Giải thích                                 |
| ------------------ | ------------------------------------------ |
| **Fat Controller** | Controller dễ phình to khi logic phức tạp  |
| **Fat Model**      | Model có thể chứa quá nhiều business logic |
| **Tight coupling** | View và Model có thể bị phụ thuộc nhau     |
| **Không rõ ràng**  | Business logic phức tạp không biết đặt đâu |

## 9. Khi nào dùng MVC?

### ✅ Phù hợp khi:

- Small to medium projects
- CRUD applications đơn giản
- Team mới làm quen với patterns
- Prototypes, MVPs
- Server-rendered web apps
- REST APIs đơn giản

### ❌ Không phù hợp khi:

- Business logic rất phức tạp → Dùng Clean Architecture
- Cần high testability → Thêm Service Layer
- Enterprise applications lớn
- Microservices

## 10. Tiến hóa từ MVC

Khi project phát triển, MVC có thể được mở rộng:

```
MVC đơn giản
    │
    ▼ (Controller phình to)
MVC + Service Layer
    │
    ▼ (Cần abstract database)
MVC + Service + Repository
    │
    ▼ (Business logic phức tạp)
Clean Architecture
```

## 11. Best Practices

1. **Thin Controller**: Controller chỉ điều phối, không chứa business logic
2. **Fat Model**: Business logic nên ở Model
3. **Không gọi Model từ View**: View chỉ nhận data đã xử lý
4. **Validation**: Validate ở cả Controller (input) và Model (business rules)
5. **Error Handling**: Xử lý lỗi tập trung qua middleware

```javascript
// ❌ Bad: Fat Controller
async create(req, res) {
  // Quá nhiều logic trong controller
  const { email, password } = req.body;

  // Validation
  if (!email || !email.includes('@')) {
    return res.status(400).json({ error: 'Invalid email' });
  }

  // Check exists
  const existing = await User.findOne({ email });
  if (existing) {
    return res.status(400).json({ error: 'Email exists' });
  }

  // Hash password
  const hashedPassword = await bcrypt.hash(password, 10);

  // Create user
  const user = await User.create({ email, password: hashedPassword });

  // Send email
  await sendWelcomeEmail(user.email);

  res.status(201).json(user);
}

// ✅ Good: Thin Controller, logic ở Model/Service
async create(req, res) {
  try {
    const user = await User.createWithValidation(req.body);
    res.status(201).json({ data: user });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
}
```

## 12. Tổng kết

**MVC là gì?** Pattern chia ứng dụng thành 3 phần: Model (data), View (UI), Controller (điều phối).

**Tại sao dùng?** Tách biệt concerns, dễ bảo trì, dễ làm việc nhóm.

**Khi nào dùng?** Small-medium projects, CRUD apps, khi bắt đầu học patterns.

**Nhớ:** MVC là điểm khởi đầu tốt, có thể mở rộng thêm Service Layer, Repository khi cần.
