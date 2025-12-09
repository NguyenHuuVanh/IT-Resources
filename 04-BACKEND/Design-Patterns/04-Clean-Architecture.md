# Clean Architecture / Layered Architecture

## 1. Tổng quan

Clean Architecture tách ứng dụng thành các layers, dependency hướng vào trong.

```
┌─────────────────────────────────────────────────────────┐
│                   Presentation Layer                    │
│              (Controllers, Routes, Views)               │
├─────────────────────────────────────────────────────────┤
│                   Application Layer                     │
│                (Use Cases, Services)                    │
├─────────────────────────────────────────────────────────┤
│                     Domain Layer                        │
│           (Entities, Business Rules, Interfaces)        │
├─────────────────────────────────────────────────────────┤
│                  Infrastructure Layer                   │
│          (Database, External APIs, Frameworks)          │
└─────────────────────────────────────────────────────────┘

Dependency Rule: Outer layers depend on inner layers
                 Inner layers know nothing about outer layers
```

## 2. Cấu trúc thư mục

```
src/
├── presentation/          # UI Layer
│   ├── controllers/
│   ├── routes/
│   ├── middleware/
│   └── validators/
├── application/           # Application Layer
│   ├── use-cases/
│   ├── services/
│   └── dtos/
├── domain/                # Domain Layer (Core)
│   ├── entities/
│   ├── repositories/      # Interfaces only
│   ├── value-objects/
│   └── errors/
└── infrastructure/        # Infrastructure Layer
    ├── database/
    │   ├── repositories/  # Implementations
    │   └── models/
    ├── external/
    └── config/
```

## 3. Domain Layer (Core)

### Entities

```typescript
// domain/entities/User.ts
export class User {
  constructor(
    public readonly id: string,
    public email: string,
    public name: string,
    private password: string,
    public readonly createdAt: Date
  ) {
    this.validateEmail(email);
  }

  private validateEmail(email: string): void {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      throw new InvalidEmailError(email);
    }
  }

  updateEmail(newEmail: string): void {
    this.validateEmail(newEmail);
    this.email = newEmail;
  }

  verifyPassword(plainPassword: string): boolean {
    // Business logic here
    return this.password === plainPassword; // Simplified
  }
}
```

### Repository Interface

```typescript
// domain/repositories/IUserRepository.ts
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(): Promise<User[]>;
  save(user: User): Promise<User>;
  update(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}
```

### Domain Errors

```typescript
// domain/errors/DomainError.ts
export abstract class DomainError extends Error {
  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

// domain/errors/InvalidEmailError.ts
export class InvalidEmailError extends DomainError {
  constructor(email: string) {
    super(`Invalid email format: ${email}`);
  }
}

// domain/errors/UserNotFoundError.ts
export class UserNotFoundError extends DomainError {
  constructor(id: string) {
    super(`User not found: ${id}`);
  }
}
```

## 4. Application Layer

### Use Cases

```typescript
// application/use-cases/CreateUserUseCase.ts
export class CreateUserUseCase {
  constructor(private userRepository: IUserRepository, private hashService: IHashService) {}

  async execute(input: CreateUserDTO): Promise<UserResponseDTO> {
    // Check if email exists
    const existingUser = await this.userRepository.findByEmail(input.email);
    if (existingUser) {
      throw new EmailAlreadyExistsError(input.email);
    }

    // Hash password
    const hashedPassword = await this.hashService.hash(input.password);

    // Create user entity
    const user = new User(generateId(), input.email, input.name, hashedPassword, new Date());

    // Save to repository
    const savedUser = await this.userRepository.save(user);

    // Return DTO
    return UserResponseDTO.fromEntity(savedUser);
  }
}
```

### DTOs (Data Transfer Objects)

```typescript
// application/dtos/CreateUserDTO.ts
export class CreateUserDTO {
  email: string;
  name: string;
  password: string;
}

// application/dtos/UserResponseDTO.ts
export class UserResponseDTO {
  id: string;
  email: string;
  name: string;
  createdAt: Date;

  static fromEntity(user: User): UserResponseDTO {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }
}
```

## 5. Infrastructure Layer

### Repository Implementation

```typescript
// infrastructure/database/repositories/UserRepository.ts
import { PrismaClient } from "@prisma/client";
import { IUserRepository } from "../../../domain/repositories/IUserRepository";
import { User } from "../../../domain/entities/User";

export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const data = await this.prisma.user.findUnique({ where: { id } });
    return data ? this.toDomain(data) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const data = await this.prisma.user.findUnique({ where: { email } });
    return data ? this.toDomain(data) : null;
  }

  async findAll(): Promise<User[]> {
    const data = await this.prisma.user.findMany();
    return data.map(this.toDomain);
  }

  async save(user: User): Promise<User> {
    const data = await this.prisma.user.create({
      data: {
        id: user.id,
        email: user.email,
        name: user.name,
        password: user["password"],
        createdAt: user.createdAt,
      },
    });
    return this.toDomain(data);
  }

  async update(user: User): Promise<User> {
    const data = await this.prisma.user.update({
      where: { id: user.id },
      data: { email: user.email, name: user.name },
    });
    return this.toDomain(data);
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }

  private toDomain(data: any): User {
    return new User(data.id, data.email, data.name, data.password, data.createdAt);
  }
}
```

## 6. Presentation Layer

### Controller

```typescript
// presentation/controllers/UserController.ts
import { Request, Response } from "express";
import { CreateUserUseCase } from "../../application/use-cases/CreateUserUseCase";
import { GetUserUseCase } from "../../application/use-cases/GetUserUseCase";

export class UserController {
  constructor(private createUserUseCase: CreateUserUseCase, private getUserUseCase: GetUserUseCase) {}

  async create(req: Request, res: Response): Promise<void> {
    try {
      const user = await this.createUserUseCase.execute(req.body);
      res.status(201).json({ data: user });
    } catch (error) {
      if (error instanceof DomainError) {
        res.status(400).json({ error: error.message });
      } else {
        res.status(500).json({ error: "Internal server error" });
      }
    }
  }

  async getById(req: Request, res: Response): Promise<void> {
    try {
      const user = await this.getUserUseCase.execute(req.params.id);
      res.json({ data: user });
    } catch (error) {
      if (error instanceof UserNotFoundError) {
        res.status(404).json({ error: error.message });
      } else {
        res.status(500).json({ error: "Internal server error" });
      }
    }
  }
}
```

## 7. Dependency Injection

```typescript
// infrastructure/config/container.ts
import { PrismaClient } from "@prisma/client";
import { PrismaUserRepository } from "../database/repositories/UserRepository";
import { BcryptHashService } from "../services/BcryptHashService";
import { CreateUserUseCase } from "../../application/use-cases/CreateUserUseCase";
import { UserController } from "../../presentation/controllers/UserController";

// Create instances
const prisma = new PrismaClient();
const userRepository = new PrismaUserRepository(prisma);
const hashService = new BcryptHashService();

// Use Cases
const createUserUseCase = new CreateUserUseCase(userRepository, hashService);
const getUserUseCase = new GetUserUseCase(userRepository);

// Controllers
export const userController = new UserController(createUserUseCase, getUserUseCase);
```

## 8. Ưu và Nhược điểm

### Ưu điểm

- ✅ Độc lập với frameworks
- ✅ Highly testable
- ✅ Dễ maintain và scale
- ✅ Business logic được bảo vệ
- ✅ Dễ thay đổi database, UI

### Nhược điểm

- ❌ Phức tạp, nhiều files
- ❌ Overkill cho small projects
- ❌ Learning curve cao
- ❌ Cần thời gian setup

## 9. Khi nào dùng?

- ✅ Enterprise applications
- ✅ Long-term projects
- ✅ Team lớn
- ✅ Cần testability cao
- ✅ Business logic phức tạp
- ❌ MVPs, prototypes
- ❌ Small CRUD apps
