# MVC Pattern (Model - View - Controller)

## 1. Tổng quan

MVC là mô hình kiến trúc phổ biến nhất, tách ứng dụng thành 3 thành phần độc lập.

```
        User Request
             │
             ▼
      ┌─────────────┐
      │  Controller │ ◄── Nhận request, điều phối
      └──────┬──────┘
             │
     ┌───────┴───────┐
     ▼               ▼
┌─────────┐    ┌─────────┐
│  Model  │    │  View   │
│ (Data)  │───►│  (UI)   │
└─────────┘    └─────────┘
                    │
                    ▼
              User Response
```

## 2. Các thành phần

### Model

- Quản lý data và business logic
- Tương tác với database
- Không biết về View và Controller

```javascript
// models/User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  password: { type: String, required: true },
  role: { type: String, enum: ["user", "admin"], default: "user" },
  createdAt: { type: Date, default: Date.now },
});

// Business logic trong Model
userSchema.methods.isAdmin = function () {
  return this.role === "admin";
};

userSchema.statics.findByEmail = function (email) {
  return this.findOne({ email: email.toLowerCase() });
};

module.exports = mongoose.model("User", userSchema);
```

### View

- Hiển thị data cho user
- Nhận data từ Controller
- Không chứa business logic

```ejs
<!-- views/users/index.ejs -->
<!DOCTYPE html>
<html>
<head>
  <title>Users List</title>
</head>
<body>
  <h1>Users</h1>

  <% if (users.length === 0) { %>
    <p>No users found.</p>
  <% } else { %>
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Email</th>
          <th>Role</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <% users.forEach(user => { %>
          <tr>
            <td><%= user.name %></td>
            <td><%= user.email %></td>
            <td><%= user.role %></td>
            <td>
              <a href="/users/<%= user.id %>">View</a>
              <a href="/users/<%= user.id %>/edit">Edit</a>
            </td>
          </tr>
        <% }) %>
      </tbody>
    </table>
  <% } %>

  <a href="/users/new">Create New User</a>
</body>
</html>
```

### Controller

- Nhận request từ user
- Gọi Model để lấy/xử lý data
- Chọn View để render response

```javascript
// controllers/userController.js
const User = require("../models/User");

const userController = {
  // GET /users
  async index(req, res) {
    try {
      const users = await User.find().sort({ createdAt: -1 });
      res.render("users/index", { users });
    } catch (error) {
      res.status(500).render("error", { message: error.message });
    }
  },

  // GET /users/:id
  async show(req, res) {
    try {
      const user = await User.findById(req.params.id);
      if (!user) {
        return res.status(404).render("error", { message: "User not found" });
      }
      res.render("users/show", { user });
    } catch (error) {
      res.status(500).render("error", { message: error.message });
    }
  },

  // GET /users/new
  new(req, res) {
    res.render("users/new");
  },

  // POST /users
  async create(req, res) {
    try {
      const { email, name, password } = req.body;
      const user = new User({ email, name, password });
      await user.save();
      res.redirect(`/users/${user.id}`);
    } catch (error) {
      res.status(400).render("users/new", { error: error.message });
    }
  },

  // GET /users/:id/edit
  async edit(req, res) {
    try {
      const user = await User.findById(req.params.id);
      if (!user) {
        return res.status(404).render("error", { message: "User not found" });
      }
      res.render("users/edit", { user });
    } catch (error) {
      res.status(500).render("error", { message: error.message });
    }
  },

  // PUT /users/:id
  async update(req, res) {
    try {
      const { name, email } = req.body;
      const user = await User.findByIdAndUpdate(req.params.id, { name, email }, { new: true, runValidators: true });
      if (!user) {
        return res.status(404).render("error", { message: "User not found" });
      }
      res.redirect(`/users/${user.id}`);
    } catch (error) {
      res.status(400).render("users/edit", { error: error.message });
    }
  },

  // DELETE /users/:id
  async destroy(req, res) {
    try {
      await User.findByIdAndDelete(req.params.id);
      res.redirect("/users");
    } catch (error) {
      res.status(500).render("error", { message: error.message });
    }
  },
};

module.exports = userController;
```

## 3. Routes

```javascript
// routes/users.js
const express = require("express");
const router = express.Router();
const userController = require("../controllers/userController");

router.get("/", userController.index);
router.get("/new", userController.new);
router.post("/", userController.create);
router.get("/:id", userController.show);
router.get("/:id/edit", userController.edit);
router.put("/:id", userController.update);
router.delete("/:id", userController.destroy);

module.exports = router;
```

## 4. Cấu trúc thư mục

```
project/
├── app.js
├── config/
│   └── database.js
├── controllers/
│   ├── userController.js
│   ├── productController.js
│   └── orderController.js
├── models/
│   ├── User.js
│   ├── Product.js
│   └── Order.js
├── views/
│   ├── layouts/
│   │   └── main.ejs
│   ├── users/
│   │   ├── index.ejs
│   │   ├── show.ejs
│   │   ├── new.ejs
│   │   └── edit.ejs
│   └── error.ejs
├── routes/
│   ├── index.js
│   └── users.js
├── public/
│   ├── css/
│   └── js/
└── package.json
```

## 5. MVC cho REST API

Khi làm API, View được thay bằng JSON response:

```javascript
// controllers/api/userController.js
const User = require("../../models/User");

const userController = {
  // GET /api/users
  async index(req, res) {
    try {
      const { page = 1, limit = 10 } = req.query;
      const users = await User.find()
        .select("-password")
        .limit(limit)
        .skip((page - 1) * limit)
        .sort({ createdAt: -1 });

      const total = await User.countDocuments();

      res.json({
        data: users,
        pagination: {
          page: parseInt(page),
          limit: parseInt(limit),
          total,
          pages: Math.ceil(total / limit),
        },
      });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  },

  // GET /api/users/:id
  async show(req, res) {
    try {
      const user = await User.findById(req.params.id).select("-password");
      if (!user) {
        return res.status(404).json({ error: "User not found" });
      }
      res.json({ data: user });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  },

  // POST /api/users
  async create(req, res) {
    try {
      const user = new User(req.body);
      await user.save();
      res.status(201).json({ data: user });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  },

  // PUT /api/users/:id
  async update(req, res) {
    try {
      const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true, runValidators: true }).select(
        "-password"
      );

      if (!user) {
        return res.status(404).json({ error: "User not found" });
      }
      res.json({ data: user });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  },

  // DELETE /api/users/:id
  async destroy(req, res) {
    try {
      const user = await User.findByIdAndDelete(req.params.id);
      if (!user) {
        return res.status(404).json({ error: "User not found" });
      }
      res.status(204).send();
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  },
};

module.exports = userController;
```

## 6. Ưu và Nhược điểm

### Ưu điểm

- ✅ Dễ hiểu, dễ học
- ✅ Phân tách rõ ràng responsibilities
- ✅ Phổ biến, nhiều tài liệu
- ✅ Phù hợp với hầu hết web frameworks

### Nhược điểm

- ❌ Controller có thể phình to (Fat Controller)
- ❌ Model có thể chứa quá nhiều logic
- ❌ Khó test khi logic phức tạp
- ❌ Không rõ ràng nơi đặt business logic phức tạp

## 7. Khi nào dùng MVC?

- ✅ Small to medium projects
- ✅ CRUD applications
- ✅ Team mới làm quen với patterns
- ✅ Prototypes, MVPs
- ❌ Không phù hợp cho enterprise apps phức tạp (nên dùng Clean Architecture)
