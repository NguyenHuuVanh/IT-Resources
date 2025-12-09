# So sánh các Architecture Patterns

## 1. Bảng so sánh tổng quan

| Pattern            | Độ phức tạp | Testability | Scalability | Use Case        |
| ------------------ | ----------- | ----------- | ----------- | --------------- |
| MVC                | Thấp        | Trung bình  | Trung bình  | Web apps, APIs  |
| MVVM               | Trung bình  | Cao         | Cao         | Frontend SPAs   |
| MVP                | Trung bình  | Cao         | Trung bình  | Mobile, Desktop |
| Clean Architecture | Cao         | Rất cao     | Rất cao     | Enterprise      |
| Repository         | Thấp        | Cao         | Cao         | Data access     |
| Service Layer      | Thấp-TB     | Cao         | Cao         | Business logic  |

## 2. Khi nào dùng pattern nào?

### MVC

```
✅ Dùng khi:
- Small to medium projects
- CRUD applications
- Team mới với patterns
- Prototypes, MVPs
- Server-rendered apps

❌ Không dùng khi:
- Business logic phức tạp
- Cần high testability
- Enterprise applications
```

### MVVM

```
✅ Dùng khi:
- Single Page Applications
- Vue.js, Angular, React apps
- Complex UI state
- Real-time updates
- Forms với validation

❌ Không dùng khi:
- Server-side rendering
- Static websites
- Simple apps
```

### MVP

```
✅ Dùng khi:
- Android native development
- Desktop applications
- Cần unit test UI logic
- Complex presentation logic

❌ Không dùng khi:
- Web applications
- Simple CRUD
- SPAs với data binding
```

### Clean Architecture

```
✅ Dùng khi:
- Enterprise applications
- Long-term projects (3+ years)
- Team lớn (5+ developers)
- Business logic phức tạp
- Cần thay đổi database/framework

❌ Không dùng khi:
- MVPs, prototypes
- Small projects
- Tight deadlines
- Solo developer
```

### Repository Pattern

```
✅ Dùng khi:
- Có thể thay đổi database
- Cần unit test business logic
- Multiple data sources
- Complex queries

❌ Không dùng khi:
- Simple CRUD với 1 database
- Prototype nhanh
```

### Service Layer

```
✅ Dùng khi:
- Controller đang phình to
- Business logic cần reuse
- Multiple entry points (API, CLI, Queue)
- Cần transaction management

❌ Không dùng khi:
- Simple CRUD operations
- Logic chỉ dùng 1 nơi
```

## 3. Kết hợp các patterns

### Small Project: MVC

```
src/
├── controllers/
├── models/
├── views/
└── routes/
```

### Medium Project: MVC + Service + Repository

```
src/
├── controllers/     # HTTP handling
├── services/        # Business logic
├── repositories/    # Data access
├── models/          # Database models
└── routes/
```

### Large Project: Clean Architecture

```
src/
├── presentation/    # Controllers, Routes
├── application/     # Use Cases, Services
├── domain/          # Entities, Interfaces
└── infrastructure/  # Database, External
```

## 4. Migration Path

```
Simple MVC
    │
    ▼ (Controller phình to)
MVC + Service Layer
    │
    ▼ (Cần switch database)
MVC + Service + Repository
    │
    ▼ (Business logic phức tạp)
Clean Architecture
```

## 5. Decision Tree

```
Start
  │
  ├─ Frontend SPA? ──────────────────► MVVM
  │
  ├─ Mobile/Desktop native? ─────────► MVP
  │
  ├─ Small project (<6 months)? ─────► MVC
  │
  ├─ Medium project?
  │   ├─ Complex business logic? ────► MVC + Service
  │   └─ Multiple databases? ────────► MVC + Service + Repository
  │
  └─ Enterprise/Long-term?
      └─ Team > 5 people? ───────────► Clean Architecture
```

## 6. Code Organization Tips

### Naming Conventions

```
Controllers: UserController, OrderController
Services: UserService, OrderService, PaymentService
Repositories: UserRepository, OrderRepository
Use Cases: CreateUserUseCase, GetOrderByIdUseCase
DTOs: CreateUserDTO, UserResponseDTO
Entities: User, Order, Product
```

### File Structure

```javascript
// Controller - chỉ handle HTTP
class UserController {
  async create(req, res) {} // POST /users
  async getAll(req, res) {} // GET /users
  async getById(req, res) {} // GET /users/:id
  async update(req, res) {} // PUT /users/:id
  async delete(req, res) {} // DELETE /users/:id
}

// Service - business logic
class UserService {
  async createUser(data) {}
  async getAllUsers(filters) {}
  async getUserById(id) {}
  async updateUser(id, data) {}
  async deleteUser(id) {}
}

// Repository - data access
class UserRepository {
  async findAll() {}
  async findById(id) {}
  async findByEmail(email) {}
  async create(data) {}
  async update(id, data) {}
  async delete(id) {}
}
```

## 7. Common Mistakes

### ❌ Over-engineering

```javascript
// Không cần Clean Architecture cho TODO app
// Dùng MVC đơn giản là đủ
```

### ❌ Under-engineering

```javascript
// Fat Controller với 500+ lines
// Nên tách ra Service Layer
```

### ❌ Mixing Concerns

```javascript
// Service gọi trực tiếp req, res
// Service không nên biết về HTTP
```

### ❌ Anemic Domain Model

```javascript
// Entity chỉ có getters/setters
// Nên có business methods
```

## 8. Best Practices

1. **Start simple, refactor later**

   - Bắt đầu với MVC
   - Refactor khi cần

2. **Single Responsibility**

   - Mỗi class làm 1 việc
   - Controller: HTTP
   - Service: Business
   - Repository: Data

3. **Dependency Injection**

   - Inject dependencies
   - Dễ test, dễ thay đổi

4. **Interface Segregation**

   - Define interfaces
   - Code to interface, not implementation

5. **Keep it Consistent**
   - Chọn 1 pattern và stick với nó
   - Document decisions
