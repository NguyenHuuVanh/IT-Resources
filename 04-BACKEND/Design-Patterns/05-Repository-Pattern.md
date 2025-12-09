# Repository Pattern

## 1. Khái niệm

**Repository Pattern** là pattern tạo một lớp trừu tượng (abstraction layer) giữa business logic và data access logic. Repository đóng vai trò như một "kho chứa" các đối tượng domain.

### Tại sao cần Repository Pattern?

**Vấn đề khi không có Repository:**

```javascript
// ❌ Service gọi trực tiếp database
class UserService {
  async getUser(id) {
    // Phụ thuộc trực tiếp vào Prisma
    return prisma.user.findUnique({ where: { id } });
  }

  async createUser(data) {
    // Nếu đổi sang MongoDB, phải sửa tất cả
    return prisma.user.create({ data });
  }
}
```

**Vấn đề:**

- Service phụ thuộc trực tiếp vào database (Prisma, Mongoose...)
- Đổi database → sửa tất cả Services
- Khó unit test (phải mock Prisma)
- Data access logic rải rác

**Repository Pattern giải quyết:**

- Tạo interface trừu tượng cho data access
- Service chỉ biết interface, không biết implementation
- Đổi database → chỉ sửa Repository implementation
- Dễ mock khi unit test

## 2. Cấu trúc

```
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│   Service   │────►│   IRepository   │     │  Database   │
│  (Business) │     │   (Interface)   │     │             │
└─────────────┘     └────────┬────────┘     └─────────────┘
                             │                     ▲
                    implements                     │
                             │                     │
                    ┌────────┴────────┐           │
                    │                 │           │
              ┌─────▼─────┐    ┌──────▼─────┐    │
              │  Prisma   │    │  Mongoose  │────┘
              │   Repo    │    │    Repo    │
              └───────────┘    └────────────┘
```

## 3. Triển khai

### Bước 1: Định nghĩa Interface

```typescript
// repositories/IUserRepository.ts

// Generic Repository Interface
interface IRepository<T> {
  findAll(): Promise<T[]>;
  findById(id: string): Promise<T | null>;
  create(entity: T): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// User-specific Repository Interface
interface IUserRepository extends IRepository<User> {
  // Các methods đặc thù cho User
  findByEmail(email: string): Promise<User | null>;
  findByRole(role: string): Promise<User[]>;
  findActive(): Promise<User[]>;
  existsByEmail(email: string): Promise<boolean>;
}

export { IRepository, IUserRepository };
```

**Tại sao dùng Interface?**

- Định nghĩa "hợp đồng" mà implementation phải tuân theo
- Service chỉ biết interface → loose coupling
- Có thể có nhiều implementations khác nhau

### Bước 2: Implement với Prisma

```typescript
// repositories/prisma/PrismaUserRepository.ts
import { PrismaClient } from "@prisma/client";
import { IUserRepository } from "../IUserRepository";
import { User } from "../../entities/User";

export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async findAll(): Promise<User[]> {
    const users = await this.prisma.user.findMany({
      orderBy: { createdAt: "desc" },
    });
    return users.map(this.toEntity);
  }

  async findById(id: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({
      where: { id },
    });
    return user ? this.toEntity(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.prisma.user.findUnique({
      where: { email },
    });
    return user ? this.toEntity(user) : null;
  }

  async findByRole(role: string): Promise<User[]> {
    const users = await this.prisma.user.findMany({
      where: { role },
    });
    return users.map(this.toEntity);
  }

  async findActive(): Promise<User[]> {
    const users = await this.prisma.user.findMany({
      where: { isActive: true },
    });
    return users.map(this.toEntity);
  }

  async existsByEmail(email: string): Promise<boolean> {
    const count = await this.prisma.user.count({
      where: { email },
    });
    return count > 0;
  }

  async create(userData: Omit<User, "id">): Promise<User> {
    const user = await this.prisma.user.create({
      data: userData,
    });
    return this.toEntity(user);
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    const user = await this.prisma.user.update({
      where: { id },
      data,
    });
    return this.toEntity(user);
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }

  // Mapper: Database record → Domain Entity
  private toEntity(data: any): User {
    return new User(data.id, data.email, data.name, data.role, data.isActive, data.createdAt);
  }
}
```

### Bước 3: Implement với MongoDB/Mongoose

```typescript
// repositories/mongoose/MongoUserRepository.ts
import { Model } from "mongoose";
import { IUserRepository } from "../IUserRepository";
import { User } from "../../entities/User";
import { UserDocument } from "../../models/UserModel";

export class MongoUserRepository implements IUserRepository {
  constructor(private model: Model<UserDocument>) {}

  async findAll(): Promise<User[]> {
    const users = await this.model.find().sort({ createdAt: -1 }).lean();
    return users.map(this.toEntity);
  }

  async findById(id: string): Promise<User | null> {
    const user = await this.model.findById(id).lean();
    return user ? this.toEntity(user) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const user = await this.model.findOne({ email }).lean();
    return user ? this.toEntity(user) : null;
  }

  async findByRole(role: string): Promise<User[]> {
    const users = await this.model.find({ role }).lean();
    return users.map(this.toEntity);
  }

  async findActive(): Promise<User[]> {
    const users = await this.model.find({ isActive: true }).lean();
    return users.map(this.toEntity);
  }

  async existsByEmail(email: string): Promise<boolean> {
    const count = await this.model.countDocuments({ email });
    return count > 0;
  }

  async create(userData: Omit<User, "id">): Promise<User> {
    const doc = await this.model.create(userData);
    return this.toEntity(doc.toObject());
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    const doc = await this.model.findByIdAndUpdate(id, data, { new: true });
    if (!doc) throw new Error("User not found");
    return this.toEntity(doc.toObject());
  }

  async delete(id: string): Promise<void> {
    await this.model.findByIdAndDelete(id);
  }

  private toEntity(data: any): User {
    return new User(data._id.toString(), data.email, data.name, data.role, data.isActive, data.createdAt);
  }
}
```

### Bước 4: Service sử dụng Repository

```typescript
// services/UserService.ts
import { IUserRepository } from "../repositories/IUserRepository";
import { User } from "../entities/User";

export class UserService {
  // Inject interface, không phải implementation cụ thể
  constructor(private userRepository: IUserRepository) {}

  async getAllUsers(): Promise<User[]> {
    return this.userRepository.findAll();
  }

  async getUserById(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new UserNotFoundError(id);
    }
    return user;
  }

  async createUser(data: CreateUserDTO): Promise<User> {
    // Check email tồn tại
    const exists = await this.userRepository.existsByEmail(data.email);
    if (exists) {
      throw new EmailExistsError(data.email);
    }

    // Hash password
    const hashedPassword = await hashPassword(data.password);

    // Tạo user qua repository
    return this.userRepository.create({
      email: data.email,
      name: data.name,
      password: hashedPassword,
      role: "user",
      isActive: true,
      createdAt: new Date(),
    });
  }

  async updateUser(id: string, data: UpdateUserDTO): Promise<User> {
    // Check user tồn tại
    await this.getUserById(id);

    // Check email mới có bị trùng không
    if (data.email) {
      const existing = await this.userRepository.findByEmail(data.email);
      if (existing && existing.id !== id) {
        throw new EmailExistsError(data.email);
      }
    }

    return this.userRepository.update(id, data);
  }

  async deleteUser(id: string): Promise<void> {
    await this.getUserById(id); // Check exists
    await this.userRepository.delete(id);
  }

  async getActiveUsers(): Promise<User[]> {
    return this.userRepository.findActive();
  }

  async getUsersByRole(role: string): Promise<User[]> {
    return this.userRepository.findByRole(role);
  }
}
```

### Bước 5: Dependency Injection

```typescript
// container.ts

// Với Prisma
const prisma = new PrismaClient();
const userRepository = new PrismaUserRepository(prisma);
const userService = new UserService(userRepository);

// Với MongoDB - chỉ cần đổi repository
// const userRepository = new MongoUserRepository(UserModel);
// const userService = new UserService(userRepository);

// Service không cần thay đổi gì!
```

## 4. Unit Testing với Mock Repository

**Đây là lợi ích lớn nhất của Repository Pattern:**

```typescript
// __tests__/UserService.test.ts
describe("UserService", () => {
  let userService: UserService;
  let mockRepository: jest.Mocked<IUserRepository>;

  beforeEach(() => {
    // Tạo mock repository
    mockRepository = {
      findAll: jest.fn(),
      findById: jest.fn(),
      findByEmail: jest.fn(),
      findByRole: jest.fn(),
      findActive: jest.fn(),
      existsByEmail: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };

    // Inject mock vào service
    userService = new UserService(mockRepository);
  });

  describe("getUserById", () => {
    it("should return user when found", async () => {
      // Arrange
      const user = new User("1", "test@test.com", "Test", "user", true, new Date());
      mockRepository.findById.mockResolvedValue(user);

      // Act
      const result = await userService.getUserById("1");

      // Assert
      expect(result).toEqual(user);
      expect(mockRepository.findById).toHaveBeenCalledWith("1");
    });

    it("should throw when user not found", async () => {
      // Arrange
      mockRepository.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(userService.getUserById("1")).rejects.toThrow(UserNotFoundError);
    });
  });

  describe("createUser", () => {
    it("should create user when email not exists", async () => {
      // Arrange
      const createData = { email: "new@test.com", name: "New", password: "123456" };
      const createdUser = new User("1", "new@test.com", "New", "user", true, new Date());

      mockRepository.existsByEmail.mockResolvedValue(false);
      mockRepository.create.mockResolvedValue(createdUser);

      // Act
      const result = await userService.createUser(createData);

      // Assert
      expect(result).toEqual(createdUser);
      expect(mockRepository.existsByEmail).toHaveBeenCalledWith("new@test.com");
      expect(mockRepository.create).toHaveBeenCalled();
    });

    it("should throw when email already exists", async () => {
      // Arrange
      mockRepository.existsByEmail.mockResolvedValue(true);

      // Act & Assert
      await expect(
        userService.createUser({
          email: "existing@test.com",
          name: "Test",
          password: "123456",
        })
      ).rejects.toThrow(EmailExistsError);

      expect(mockRepository.create).not.toHaveBeenCalled();
    });
  });
});
```

## 5. Ưu điểm

| Ưu điểm                   | Giải thích                              |
| ------------------------- | --------------------------------------- |
| **Abstraction**           | Service không biết database cụ thể      |
| **Testability**           | Dễ mock repository khi unit test        |
| **Flexibility**           | Dễ switch database                      |
| **Single Responsibility** | Repository chỉ lo data access           |
| **Reusability**           | Repository có thể dùng ở nhiều services |
| **Maintainability**       | Data access logic tập trung một chỗ     |

## 6. Nhược điểm

| Nhược điểm               | Giải thích                                  |
| ------------------------ | ------------------------------------------- |
| **Abstraction overhead** | Thêm layer, thêm code                       |
| **Overkill**             | Quá phức tạp cho simple CRUD                |
| **Leaky abstraction**    | Khó abstract hết tất cả database features   |
| **Performance**          | Có thể miss database-specific optimizations |

## 7. Khi nào dùng?

### ✅ Phù hợp khi:

- Có thể thay đổi database trong tương lai
- Cần unit test business logic
- Multiple data sources (SQL + NoSQL + API)
- Complex queries cần encapsulate
- Team lớn, cần clear boundaries

### ❌ Không phù hợp khi:

- Simple CRUD với 1 database cố định
- Prototype nhanh
- Solo developer, small project
- Không cần unit test

## 8. Tổng kết

**Repository Pattern là gì?** Pattern tạo abstraction layer giữa business logic và data access.

**Tại sao dùng?** Loose coupling, dễ test, dễ switch database.

**Khi nào dùng?** Khi cần flexibility với data source, cần unit test.

**Nhớ:**

- Interface: Định nghĩa contract
- Implementation: Prisma, Mongoose, API...
- Service: Chỉ biết interface
- Testing: Mock interface để test service
