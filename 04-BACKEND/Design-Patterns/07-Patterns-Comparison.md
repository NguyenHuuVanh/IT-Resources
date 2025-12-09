# So sánh các Architecture Patterns

## 1. Bảng so sánh tổng quan

| Pattern                | Mục đích             | Độ phức tạp | Testability | Use Case        |
| ---------------------- | -------------------- | ----------- | ----------- | --------------- |
| **MVC**                | Tách UI, Logic, Data | Thấp        | Trung bình  | Web apps, APIs  |
| **MVVM**               | Two-way binding      | Trung bình  | Cao         | Frontend SPAs   |
| **MVP**                | View passive         | Trung bình  | Cao         | Mobile, Desktop |
| **Clean Architecture** | Layers độc lập       | Cao         | Rất cao     | Enterprise      |
| **Repository**         | Abstract data access | Thấp        | Cao         | Data layer      |
| **Service Layer**      | Tách business logic  | Thấp-TB     | Cao         | Business layer  |

## 2. So sánh chi tiết

### MVC vs MVVM vs MVP

```
┌─────────────────────────────────────────────────────────────────┐
│                           MVC                                   │
│  User → Controller → Model → View → User                        │
│  - Controller xử lý request                                     │
│  - View có thể có logic                                         │
│  - One-way data flow                                            │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                          MVVM                                   │
│  User ↔ View ↔ ViewModel ↔ Model                                │
│  - Two-way data binding                                         │
│  - View passive (chỉ bind)                                      │
│  - ViewModel quản lý state                                      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                           MVP                                   │
│  User → View → Presenter → Model                                │
│         ↑         │                                             │
│         └─────────┘ (update via interface)                      │
│  - View hoàn toàn passive                                       │
│  - Presenter điều khiển View qua interface                      │
│  - Manual updates                                               │
└─────────────────────────────────────────────────────────────────┘
```

| Khía cạnh        | MVC             | MVVM                | MVP                   |
| ---------------- | --------------- | ------------------- | --------------------- |
| **View**         | Active          | Passive + Binding   | Passive               |
| **Mediator**     | Controller      | ViewModel           | Presenter             |
| **Data binding** | Manual          | Automatic           | Manual                |
| **View-Model**   | View biết Model | View bind ViewModel | View không biết Model |
| **Testing**      | Test Controller | Test ViewModel      | Test Presenter        |
| **Platform**     | Server-side     | Frontend            | Mobile/Desktop        |

### Khi nào dùng pattern nào?

```
Bạn đang làm gì?
│
├─ Server-side web app / REST API?
│   └─► MVC
│
├─ Frontend SPA (Vue, React, Angular)?
│   └─► MVVM
│
├─ Mobile native (Android, iOS)?
│   └─► MVP
│
├─ Enterprise app phức tạp?
│   └─► Clean Architecture
│
├─ Cần switch database?
│   └─► Repository Pattern
│
└─ Controller quá dài?
    └─► Service Layer
```

## 3. Kết hợp các Patterns

### Level 1: MVC đơn giản

```
Controller → Model → View
```

**Dùng khi:** Small projects, CRUD đơn giản

### Level 2: MVC + Service Layer

```
Controller → Service → Model → View
```

**Dùng khi:** Business logic phức tạp, controller phình to

### Level 3: MVC + Service + Repository

```
Controller → Service → Repository → Database
```

**Dùng khi:** Cần abstract database, unit test

### Level 4: Clean Architecture

```
Presentation → Application → Domain ← Infrastructure
```

**Dùng khi:** Enterprise, long-term projects

## 4. Cấu trúc thư mục theo Level

### Level 1: MVC

```
src/
├── controllers/
├── models/
├── views/
└── routes/
```

### Level 2: MVC + Service

```
src/
├── controllers/
├── services/
├── models/
├── views/
└── routes/
```

### Level 3: MVC + Service + Repository

```
src/
├── controllers/
├── services/
├── repositories/
├── models/
└── routes/
```

### Level 4: Clean Architecture

```
src/
├── presentation/
│   ├── controllers/
│   └── routes/
├── application/
│   ├── use-cases/
│   └── dtos/
├── domain/
│   ├── entities/
│   └── repositories/
└── infrastructure/
    ├── database/
    └── services/
```

## 5. Decision Matrix

| Yếu tố        | MVC        | +Service    | +Repository  | Clean       |
| ------------- | ---------- | ----------- | ------------ | ----------- |
| Project size  | Small      | Medium      | Medium-Large | Large       |
| Team size     | 1-3        | 3-5         | 5-10         | 10+         |
| Duration      | < 6 months | 6-12 months | 1-2 years    | 2+ years    |
| Complexity    | Low        | Medium      | Medium-High  | High        |
| Test coverage | < 50%      | 50-70%      | 70-85%       | 85%+        |
| Change DB     | Unlikely   | Possible    | Likely       | Very likely |

## 6. Migration Path

```
MVC đơn giản
    │
    │ Controller > 100 lines?
    ▼
MVC + Service Layer
    │
    │ Cần mock database?
    ▼
MVC + Service + Repository
    │
    │ Business logic phức tạp?
    │ Team > 5 người?
    ▼
Clean Architecture
```

## 7. Common Mistakes

### ❌ Over-engineering

```javascript
// Không cần Clean Architecture cho TODO app
// MVC đơn giản là đủ
```

### ❌ Under-engineering

```javascript
// Controller 500+ lines
// Nên tách ra Service Layer
```

### ❌ Mixing concerns

```javascript
// ❌ Service biết về HTTP
class UserService {
  async createUser(req, res) {
    // Sai!
    // ...
  }
}

// ✅ Service chỉ biết business logic
class UserService {
  async createUser(userData) {
    // ...
  }
}
```

### ❌ Anemic Domain Model

```javascript
// ❌ Entity chỉ có data, không có behavior
class User {
  id;
  email;
  name;
}

// ✅ Entity có business methods
class User {
  id;
  email;
  name;

  updateEmail(newEmail) {
    this.validateEmail(newEmail);
    this.email = newEmail;
  }

  isAdmin() {
    return this.role === "admin";
  }
}
```

## 8. Best Practices

### 1. Start Simple, Refactor Later

```
Bắt đầu với MVC
→ Khi cần, thêm Service Layer
→ Khi cần, thêm Repository
→ Không cần Clean Architecture từ đầu
```

### 2. Single Responsibility

```
Controller: HTTP handling
Service: Business logic
Repository: Data access
Entity: Business rules
```

### 3. Dependency Injection

```javascript
// ✅ Inject dependencies
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }
}

// ❌ Tạo dependency bên trong
class UserService {
  constructor() {
    this.userRepository = new UserRepository(); // Khó test
  }
}
```

### 4. Code to Interface

```typescript
// ✅ Depend on interface
class UserService {
  constructor(private repo: IUserRepository) {}
}

// ❌ Depend on implementation
class UserService {
  constructor(private repo: PrismaUserRepository) {}
}
```

### 5. Keep it Consistent

```
- Chọn 1 pattern và stick với nó
- Document decisions
- Team phải hiểu và follow
```

## 9. Quick Reference

### Chọn pattern theo câu hỏi:

| Câu hỏi                  | Nếu Yes            | Nếu No     |
| ------------------------ | ------------------ | ---------- |
| Project < 6 tháng?       | MVC                | Xem tiếp   |
| Controller > 100 lines?  | + Service          | Giữ nguyên |
| Cần unit test business?  | + Service          | Giữ nguyên |
| Có thể đổi database?     | + Repository       | Giữ nguyên |
| Team > 5 người?          | Clean Architecture | Giữ nguyên |
| Business logic phức tạp? | Clean Architecture | Giữ nguyên |

## 10. Tổng kết

**MVC:** Default choice, simple projects
**MVVM:** Frontend SPAs với data binding
**MVP:** Mobile/Desktop cần testability
**Clean Architecture:** Enterprise, long-term
**Repository:** Abstract data access
**Service Layer:** Tách business logic

**Nguyên tắc:**

1. Start simple
2. Refactor when needed
3. Don't over-engineer
4. Keep it consistent
5. Document decisions
