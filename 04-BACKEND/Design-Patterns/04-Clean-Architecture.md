# Clean Architecture / Layered Architecture

## 1. Khái niệm

**Clean Architecture** là mô hình kiến trúc do Robert C. Martin (Uncle Bob) đề xuất, tổ chức code thành các layers đồng tâm với nguyên tắc: **dependency hướng vào trong**.

### Tại sao cần Clean Architecture?

**Vấn đề với MVC khi project lớn:**

- Controller phình to (Fat Controller)
- Business logic rải rác khắp nơi
- Khó thay đổi database hoặc framework
- Khó test business logic
- Code coupling cao

**Clean Architecture giải quyết:**

- Tách biệt business logic khỏi framework
- Dễ thay đổi database, UI, external services
- Business logic dễ test
- Code maintainable, scalable

### Nguyên tắc cốt lõi: Dependency Rule

```
"Source code dependencies must point only inward,
toward higher-level policies."

Outer layers phụ thuộc vào Inner layers.
Inner layers KHÔNG biết gì về Outer layers.
```

## 2. Các Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                       │
│              (Controllers, Routes, Views, DTOs)             │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                 APPLICATION LAYER                    │   │
│  │            (Use Cases, Application Services)         │   │
│  │                                                      │   │
│  │  ┌─────────────────────────────────────────────┐    │   │
│  │  │              DOMAIN LAYER                    │    │   │
│  │  │    (Entities, Business Rules, Interfaces)   │    │   │
│  │  │                                              │    │   │
│  │  │         ← CORE (không phụ thuộc gì) →       │    │   │
│  │  │                                              │    │   │
│  │  └─────────────────────────────────────────────┘    │   │
│  │                                                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                   INFRASTRUCTURE LAYER                      │
│         (Database, External APIs, Frameworks, Config)       │
└─────────────────────────────────────────────────────────────┘
```

### Domain Layer - "Trái tim của ứng dụng"

**Khái niệm:** Layer trong cùng, chứa business logic thuần túy, không phụ thuộc bất kỳ framework hay library nào.

**Chứa gì:**

- **Entities**: Đối tượng nghiệp vụ với data và behavior
- **Value Objects**: Đối tượng immutable, so sánh bằng value
- **Domain Services**: Logic không thuộc về entity cụ thể
- **Repository Interfaces**: Chỉ định nghĩa interface, không implementation
- **Domain Events**: Events trong domain
- **Domain Errors**: Custom errors

**Tác dụng:**

- Business logic tập trung, dễ hiểu
- Không phụ thuộc framework → dễ test
- Có thể reuse ở nhiều projects
- Thay đổi database không ảnh hưởng

```typescript
// domain/entities/User.ts
export class User {
  private constructor(
    public readonly id: string,
    private _email: string,
    private _name: string,
    private _password: string,
    public readonly createdAt: Date
  ) {
    // Validation trong constructor
    this.validateEmail(_email);
    this.validateName(_name);
  }

  // Factory method
  static create(props: CreateUserProps): User {
    return new User(generateId(), props.email, props.name, props.password, new Date());
  }

  // Getters
  get email(): string {
    return this._email;
  }

  get name(): string {
    return this._name;
  }

  // Business methods
  updateEmail(newEmail: string): void {
    this.validateEmail(newEmail);
    this._email = newEmail;
  }

  updateName(newName: string): void {
    this.validateName(newName);
    this._name = newName;
  }

  verifyPassword(plainPassword: string): boolean {
    // Business logic: verify password
    return comparePassword(plainPassword, this._password);
  }

  // Private validation methods
  private validateEmail(email: string): void {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      throw new InvalidEmailError(email);
    }
  }

  private validateName(name: string): void {
    if (!name || name.trim().length < 2) {
      throw new InvalidNameError(name);
    }
  }
}
```

```typescript
// domain/repositories/IUserRepository.ts
// CHỈ ĐỊNH NGHĨA INTERFACE - không implementation
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(): Promise<User[]>;
  save(user: User): Promise<User>;
  update(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}
```

```typescript
// domain/errors/DomainErrors.ts
export abstract class DomainError extends Error {
  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class InvalidEmailError extends DomainError {
  constructor(email: string) {
    super(`Email không hợp lệ: ${email}`);
  }
}

export class UserNotFoundError extends DomainError {
  constructor(identifier: string) {
    super(`Không tìm thấy user: ${identifier}`);
  }
}

export class EmailAlreadyExistsError extends DomainError {
  constructor(email: string) {
    super(`Email đã tồn tại: ${email}`);
  }
}
```

### Application Layer - "Điều phối Use Cases"

**Khái niệm:** Layer chứa application-specific business logic, điều phối các entities và services để thực hiện use cases.

**Chứa gì:**

- **Use Cases**: Mỗi use case = 1 hành động của user
- **Application Services**: Orchestrate multiple use cases
- **DTOs**: Data Transfer Objects cho input/output
- **Interfaces**: Cho external services (email, payment...)

**Tác dụng:**

- Mỗi use case độc lập, dễ hiểu
- Dễ test từng use case
- Tách biệt với presentation layer

```typescript
// application/dtos/UserDTOs.ts
export class CreateUserDTO {
  email: string;
  name: string;
  password: string;
}

export class UpdateUserDTO {
  name?: string;
  email?: string;
}

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

```typescript
// application/use-cases/CreateUserUseCase.ts
export class CreateUserUseCase {
  constructor(
    private userRepository: IUserRepository,
    private hashService: IHashService,
    private emailService: IEmailService
  ) {}

  async execute(input: CreateUserDTO): Promise<UserResponseDTO> {
    // 1. Check email đã tồn tại chưa
    const existingUser = await this.userRepository.findByEmail(input.email);
    if (existingUser) {
      throw new EmailAlreadyExistsError(input.email);
    }

    // 2. Hash password
    const hashedPassword = await this.hashService.hash(input.password);

    // 3. Tạo User entity
    const user = User.create({
      email: input.email,
      name: input.name,
      password: hashedPassword,
    });

    // 4. Lưu vào repository
    const savedUser = await this.userRepository.save(user);

    // 5. Gửi welcome email (async, không chờ)
    this.emailService
      .sendWelcomeEmail(savedUser.email, savedUser.name)
      .catch((err) => console.error("Send email failed:", err));

    // 6. Trả về DTO
    return UserResponseDTO.fromEntity(savedUser);
  }
}
```

```typescript
// application/use-cases/GetUserByIdUseCase.ts
export class GetUserByIdUseCase {
  constructor(private userRepository: IUserRepository) {}

  async execute(id: string): Promise<UserResponseDTO> {
    const user = await this.userRepository.findById(id);

    if (!user) {
      throw new UserNotFoundError(id);
    }

    return UserResponseDTO.fromEntity(user);
  }
}
```

```typescript
// application/use-cases/UpdateUserUseCase.ts
export class UpdateUserUseCase {
  constructor(private userRepository: IUserRepository) {}

  async execute(id: string, input: UpdateUserDTO): Promise<UserResponseDTO> {
    // 1. Tìm user
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new UserNotFoundError(id);
    }

    // 2. Check email mới có bị trùng không
    if (input.email && input.email !== user.email) {
      const existingUser = await this.userRepository.findByEmail(input.email);
      if (existingUser) {
        throw new EmailAlreadyExistsError(input.email);
      }
      user.updateEmail(input.email);
    }

    // 3. Update name nếu có
    if (input.name) {
      user.updateName(input.name);
    }

    // 4. Lưu và trả về
    const updatedUser = await this.userRepository.update(user);
    return UserResponseDTO.fromEntity(updatedUser);
  }
}
```

### Infrastructure Layer - "Kết nối với thế giới bên ngoài"

**Khái niệm:** Layer implement các interfaces từ Domain/Application, kết nối với database, external APIs, frameworks.

**Chứa gì:**

- **Repository Implementations**: Implement IRepository interfaces
- **External Service Implementations**: Email, Payment, Storage...
- **Database Models**: ORM models
- **Framework Config**: Express, NestJS setup

**Tác dụng:**

- Tách biệt implementation details
- Dễ thay đổi database (MySQL → PostgreSQL)
- Dễ mock khi testing

```typescript
// infrastructure/repositories/PrismaUserRepository.ts
import { PrismaClient } from "@prisma/client";
import { IUserRepository } from "../../domain/repositories/IUserRepository";
import { User } from "../../domain/entities/User";

export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    const data = await this.prisma.user.findUnique({
      where: { id },
    });
    return data ? this.toDomain(data) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const data = await this.prisma.user.findUnique({
      where: { email },
    });
    return data ? this.toDomain(data) : null;
  }

  async findAll(): Promise<User[]> {
    const data = await this.prisma.user.findMany({
      orderBy: { createdAt: "desc" },
    });
    return data.map(this.toDomain);
  }

  async save(user: User): Promise<User> {
    const data = await this.prisma.user.create({
      data: this.toPersistence(user),
    });
    return this.toDomain(data);
  }

  async update(user: User): Promise<User> {
    const data = await this.prisma.user.update({
      where: { id: user.id },
      data: {
        email: user.email,
        name: user.name,
      },
    });
    return this.toDomain(data);
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }

  // Mapper: Database → Domain
  private toDomain(data: any): User {
    return User.reconstitute({
      id: data.id,
      email: data.email,
      name: data.name,
      password: data.password,
      createdAt: data.createdAt,
    });
  }

  // Mapper: Domain → Database
  private toPersistence(user: User): any {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      password: user["_password"],
      createdAt: user.createdAt,
    };
  }
}
```

```typescript
// infrastructure/services/BcryptHashService.ts
import bcrypt from "bcrypt";
import { IHashService } from "../../application/interfaces/IHashService";

export class BcryptHashService implements IHashService {
  private readonly saltRounds = 12;

  async hash(password: string): Promise<string> {
    return bcrypt.hash(password, this.saltRounds);
  }

  async compare(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}
```

```typescript
// infrastructure/services/NodemailerEmailService.ts
import nodemailer from "nodemailer";
import { IEmailService } from "../../application/interfaces/IEmailService";

export class NodemailerEmailService implements IEmailService {
  private transporter: nodemailer.Transporter;

  constructor(config: EmailConfig) {
    this.transporter = nodemailer.createTransport(config);
  }

  async sendWelcomeEmail(email: string, name: string): Promise<void> {
    await this.transporter.sendMail({
      to: email,
      subject: "Chào mừng bạn!",
      html: `<h1>Xin chào ${name}!</h1><p>Cảm ơn bạn đã đăng ký.</p>`,
    });
  }
}
```

### Presentation Layer - "Giao tiếp với User"

**Khái niệm:** Layer xử lý HTTP requests, validate input, format output.

**Chứa gì:**

- **Controllers**: Handle HTTP requests
- **Routes**: Define API endpoints
- **Middleware**: Auth, validation, error handling
- **Request/Response DTOs**: Validate và transform data

```typescript
// presentation/controllers/UserController.ts
import { Request, Response, NextFunction } from "express";
import { CreateUserUseCase } from "../../application/use-cases/CreateUserUseCase";
import { GetUserByIdUseCase } from "../../application/use-cases/GetUserByIdUseCase";
import { DomainError } from "../../domain/errors/DomainErrors";

export class UserController {
  constructor(
    private createUserUseCase: CreateUserUseCase,
    private getUserByIdUseCase: GetUserByIdUseCase,
    private updateUserUseCase: UpdateUserUseCase,
    private deleteUserUseCase: DeleteUserUseCase
  ) {}

  async create(req: Request, res: Response, next: NextFunction) {
    try {
      const user = await this.createUserUseCase.execute(req.body);

      res.status(201).json({
        success: true,
        data: user,
      });
    } catch (error) {
      next(error);
    }
  }

  async getById(req: Request, res: Response, next: NextFunction) {
    try {
      const user = await this.getUserByIdUseCase.execute(req.params.id);

      res.json({
        success: true,
        data: user,
      });
    } catch (error) {
      next(error);
    }
  }

  async update(req: Request, res: Response, next: NextFunction) {
    try {
      const user = await this.updateUserUseCase.execute(req.params.id, req.body);

      res.json({
        success: true,
        data: user,
      });
    } catch (error) {
      next(error);
    }
  }

  async delete(req: Request, res: Response, next: NextFunction) {
    try {
      await this.deleteUserUseCase.execute(req.params.id);

      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}
```

```typescript
// presentation/middleware/errorHandler.ts
import { Request, Response, NextFunction } from "express";
import { DomainError, UserNotFoundError, EmailAlreadyExistsError } from "../../domain/errors/DomainErrors";

export function errorHandler(error: Error, req: Request, res: Response, next: NextFunction) {
  console.error("Error:", error);

  // Domain errors → Client errors (4xx)
  if (error instanceof UserNotFoundError) {
    return res.status(404).json({
      success: false,
      error: error.message,
    });
  }

  if (error instanceof EmailAlreadyExistsError) {
    return res.status(409).json({
      success: false,
      error: error.message,
    });
  }

  if (error instanceof DomainError) {
    return res.status(400).json({
      success: false,
      error: error.message,
    });
  }

  // Unknown errors → Server error (500)
  res.status(500).json({
    success: false,
    error: "Lỗi server",
  });
}
```

## 3. Dependency Injection

**Khái niệm:** Kỹ thuật inject dependencies từ bên ngoài thay vì tạo bên trong class.

**Tại sao cần?**

- Loose coupling
- Dễ test (inject mock)
- Dễ thay đổi implementation

```typescript
// infrastructure/container.ts
import { PrismaClient } from "@prisma/client";

// Infrastructure
const prisma = new PrismaClient();
const userRepository = new PrismaUserRepository(prisma);
const hashService = new BcryptHashService();
const emailService = new NodemailerEmailService(emailConfig);

// Use Cases
const createUserUseCase = new CreateUserUseCase(userRepository, hashService, emailService);
const getUserByIdUseCase = new GetUserByIdUseCase(userRepository);
const updateUserUseCase = new UpdateUserUseCase(userRepository);
const deleteUserUseCase = new DeleteUserUseCase(userRepository);

// Controllers
export const userController = new UserController(
  createUserUseCase,
  getUserByIdUseCase,
  updateUserUseCase,
  deleteUserUseCase
);
```

## 4. Cấu trúc thư mục

```
src/
├── domain/                      # CORE - không phụ thuộc gì
│   ├── entities/
│   │   ├── User.ts
│   │   └── Order.ts
│   ├── value-objects/
│   │   ├── Email.ts
│   │   └── Money.ts
│   ├── repositories/            # Chỉ interfaces
│   │   ├── IUserRepository.ts
│   │   └── IOrderRepository.ts
│   ├── services/
│   │   └── PricingService.ts
│   └── errors/
│       └── DomainErrors.ts
│
├── application/                 # Use Cases
│   ├── use-cases/
│   │   ├── user/
│   │   │   ├── CreateUserUseCase.ts
│   │   │   ├── GetUserByIdUseCase.ts
│   │   │   └── UpdateUserUseCase.ts
│   │   └── order/
│   │       └── CreateOrderUseCase.ts
│   ├── dtos/
│   │   ├── UserDTOs.ts
│   │   └── OrderDTOs.ts
│   └── interfaces/              # External service interfaces
│       ├── IHashService.ts
│       └── IEmailService.ts
│
├── infrastructure/              # Implementations
│   ├── database/
│   │   ├── prisma/
│   │   │   └── schema.prisma
│   │   └── repositories/
│   │       ├── PrismaUserRepository.ts
│   │       └── PrismaOrderRepository.ts
│   ├── services/
│   │   ├── BcryptHashService.ts
│   │   └── NodemailerEmailService.ts
│   ├── config/
│   │   └── index.ts
│   └── container.ts             # Dependency Injection
│
├── presentation/                # HTTP Layer
│   ├── controllers/
│   │   ├── UserController.ts
│   │   └── OrderController.ts
│   ├── routes/
│   │   ├── index.ts
│   │   ├── userRoutes.ts
│   │   └── orderRoutes.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── validate.ts
│   │   └── errorHandler.ts
│   └── validators/
│       └── userValidators.ts
│
└── app.ts                       # Entry point
```

## 5. Ưu điểm

| Ưu điểm                   | Giải thích                                        |
| ------------------------- | ------------------------------------------------- |
| **Framework Independent** | Business logic không phụ thuộc Express, NestJS... |
| **Database Independent**  | Dễ đổi từ MySQL sang PostgreSQL, MongoDB          |
| **Testable**              | Domain và Use Cases dễ unit test                  |
| **Maintainable**          | Code tổ chức rõ ràng, dễ tìm                      |
| **Scalable**              | Dễ thêm features mới                              |

## 6. Nhược điểm

| Nhược điểm           | Giải thích                         |
| -------------------- | ---------------------------------- |
| **Complexity**       | Nhiều files, folders, abstractions |
| **Boilerplate**      | Cần viết nhiều interfaces, DTOs    |
| **Learning curve**   | Cần thời gian để hiểu              |
| **Overkill**         | Quá phức tạp cho small projects    |
| **Over-engineering** | Dễ bị cuốn vào abstractions        |

## 7. Khi nào dùng?

### ✅ Phù hợp khi:

- Enterprise applications
- Long-term projects (3+ năm)
- Team lớn (5+ developers)
- Business logic phức tạp
- Cần thay đổi database/framework
- Cần high testability

### ❌ Không phù hợp khi:

- MVPs, prototypes
- Small projects
- Tight deadlines
- Solo developer
- Simple CRUD apps

## 8. Tổng kết

**Clean Architecture là gì?** Kiến trúc layers với dependency hướng vào trong, business logic ở core.

**Tại sao dùng?** Tách biệt concerns, dễ test, dễ thay đổi database/framework.

**Khi nào dùng?** Enterprise apps, long-term projects, team lớn.

**Nhớ:**

- Domain Layer: Business logic thuần túy
- Application Layer: Use Cases điều phối
- Infrastructure Layer: Database, external services
- Presentation Layer: HTTP handling
- Dependency Rule: Outer → Inner
