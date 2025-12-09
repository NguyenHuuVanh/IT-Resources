# Repository Pattern

## 1. Tổng quan

Repository Pattern tách data access logic khỏi business logic, tạo abstraction layer giữa domain và data source.

```
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│   Service   │────►│   Repository    │────►│  Database   │
│  (Business) │     │   (Interface)   │     │  (Storage)  │
└─────────────┘     └─────────────────┘     └─────────────┘
                            │
                    ┌───────┴───────┐
                    ▼               ▼
              ┌──────────┐   ┌──────────┐
              │  MySQL   │   │ MongoDB  │
              │   Impl   │   │   Impl   │
              └──────────┘   └──────────┘
```

## 2. Repository Interface

```typescript
// repositories/IRepository.ts (Generic)
interface IRepository<T> {
  findAll(): Promise<T[]>;
  findById(id: string): Promise<T | null>;
  create(entity: T): Promise<T>;
  update(id: string, entity: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// repositories/IUserRepository.ts (Specific)
interface IUserRepository extends IRepository<User> {
  findByEmail(email: string): Promise<User | null>;
  findByRole(role: string): Promise<User[]>;
  findActive(): Promise<User[]>;
}
```

## 3. Implementation với Prisma

```typescript
// repositories/prisma/PrismaUserRepository.ts
import { PrismaClient } from "@prisma/client";
import { IUserRepository } from "../IUserRepository";
import { User } from "../../entities/User";

export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async findAll(): Promise<User[]> {
    return this.prisma.user.findMany();
  }

  async findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  async findByRole(role: string): Promise<User[]> {
    return this.prisma.user.findMany({ where: { role } });
  }

  async findActive(): Promise<User[]> {
    return this.prisma.user.findMany({ where: { isActive: true } });
  }

  async create(data: Omit<User, "id">): Promise<User> {
    return this.prisma.user.create({ data });
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    return this.prisma.user.update({ where: { id }, data });
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }
}
```

## 4. Implementation với MongoDB/Mongoose

```typescript
// repositories/mongoose/MongoUserRepository.ts
import { Model } from "mongoose";
import { IUserRepository } from "../IUserRepository";
import { User, UserDocument } from "../../models/User";

export class MongoUserRepository implements IUserRepository {
  constructor(private model: Model<UserDocument>) {}

  async findAll(): Promise<User[]> {
    return this.model.find().lean();
  }

  async findById(id: string): Promise<User | null> {
    return this.model.findById(id).lean();
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.model.findOne({ email }).lean();
  }

  async findByRole(role: string): Promise<User[]> {
    return this.model.find({ role }).lean();
  }

  async findActive(): Promise<User[]> {
    return this.model.find({ isActive: true }).lean();
  }

  async create(data: Omit<User, "id">): Promise<User> {
    const doc = await this.model.create(data);
    return doc.toObject();
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    const doc = await this.model.findByIdAndUpdate(id, data, { new: true });
    if (!doc) throw new Error("User not found");
    return doc.toObject();
  }

  async delete(id: string): Promise<void> {
    await this.model.findByIdAndDelete(id);
  }
}
```

## 5. Service sử dụng Repository

```typescript
// services/UserService.ts
export class UserService {
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
    // Check email exists
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) {
      throw new EmailExistsError(data.email);
    }

    // Business logic
    const hashedPassword = await hashPassword(data.password);

    return this.userRepository.create({
      ...data,
      password: hashedPassword,
      isActive: true,
      createdAt: new Date(),
    });
  }

  async updateUser(id: string, data: UpdateUserDTO): Promise<User> {
    await this.getUserById(id); // Check exists
    return this.userRepository.update(id, data);
  }

  async deleteUser(id: string): Promise<void> {
    await this.getUserById(id); // Check exists
    await this.userRepository.delete(id);
  }
}
```

## 6. Dependency Injection

```typescript
// Với Prisma
const prisma = new PrismaClient();
const userRepository = new PrismaUserRepository(prisma);
const userService = new UserService(userRepository);

// Với MongoDB
const userRepository = new MongoUserRepository(UserModel);
const userService = new UserService(userRepository);

// Switch database dễ dàng - Service không cần thay đổi!
```

## 7. Unit Testing với Mock Repository

```typescript
// __tests__/UserService.test.ts
describe("UserService", () => {
  let userService: UserService;
  let mockRepository: jest.Mocked<IUserRepository>;

  beforeEach(() => {
    mockRepository = {
      findAll: jest.fn(),
      findById: jest.fn(),
      findByEmail: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };
    userService = new UserService(mockRepository);
  });

  describe("getUserById", () => {
    it("should return user when found", async () => {
      const user = { id: "1", name: "John", email: "john@test.com" };
      mockRepository.findById.mockResolvedValue(user);

      const result = await userService.getUserById("1");

      expect(result).toEqual(user);
      expect(mockRepository.findById).toHaveBeenCalledWith("1");
    });

    it("should throw when user not found", async () => {
      mockRepository.findById.mockResolvedValue(null);

      await expect(userService.getUserById("1")).rejects.toThrow(UserNotFoundError);
    });
  });

  describe("createUser", () => {
    it("should create user when email not exists", async () => {
      mockRepository.findByEmail.mockResolvedValue(null);
      mockRepository.create.mockResolvedValue({ id: "1", name: "John" });

      const result = await userService.createUser({
        name: "John",
        email: "john@test.com",
        password: "123456",
      });

      expect(result.id).toBe("1");
      expect(mockRepository.create).toHaveBeenCalled();
    });

    it("should throw when email exists", async () => {
      mockRepository.findByEmail.mockResolvedValue({ id: "1" });

      await expect(
        userService.createUser({
          name: "John",
          email: "john@test.com",
          password: "123456",
        })
      ).rejects.toThrow(EmailExistsError);
    });
  });
});
```

## 8. Ưu và Nhược điểm

### Ưu điểm

- ✅ Tách biệt data access và business logic
- ✅ Dễ switch database
- ✅ Dễ unit test với mock
- ✅ Single Responsibility Principle
- ✅ Code reusable

### Nhược điểm

- ❌ Thêm layer abstraction
- ❌ Có thể overkill cho simple apps
- ❌ Cần maintain interface và implementations

## 9. Khi nào dùng?

- ✅ Có thể thay đổi database trong tương lai
- ✅ Cần unit test business logic
- ✅ Multiple data sources
- ✅ Complex queries cần encapsulate
- ❌ Simple CRUD với 1 database cố định
