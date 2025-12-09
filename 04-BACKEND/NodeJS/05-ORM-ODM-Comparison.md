# ORM/ODM trong Node.js

## 1. Khái niệm

### ORM là gì?

**ORM (Object-Relational Mapping)** là kỹ thuật cho phép tương tác với database bằng objects thay vì viết SQL trực tiếp.

**Không có ORM:**

```javascript
// Viết SQL trực tiếp
const result = await db.query("SELECT * FROM users WHERE email = $1", ["john@example.com"]);
```

**Với ORM:**

```javascript
// Dùng objects và methods
const user = await User.findOne({ where: { email: "john@example.com" } });
```

### ODM là gì?

**ODM (Object-Document Mapping)** tương tự ORM nhưng cho NoSQL databases (MongoDB).

### Tại sao cần ORM/ODM?

| Lợi ích          | Giải thích                           |
| ---------------- | ------------------------------------ |
| **Abstraction**  | Không cần viết SQL/queries trực tiếp |
| **Type Safety**  | TypeScript support, auto-complete    |
| **Migrations**   | Quản lý schema changes               |
| **Relations**    | Dễ dàng define và query relations    |
| **Security**     | Tự động escape, tránh SQL injection  |
| **Productivity** | Viết code nhanh hơn                  |

### Các ORM/ODM phổ biến

| Tool          | Database      | Đặc điểm              |
| ------------- | ------------- | --------------------- |
| **Prisma**    | SQL + MongoDB | Modern, type-safe     |
| **TypeORM**   | SQL           | Decorator-based       |
| **Sequelize** | SQL           | Mature, feature-rich  |
| **Mongoose**  | MongoDB       | ODM phổ biến nhất     |
| **Drizzle**   | SQL           | Lightweight, SQL-like |
| **Knex.js**   | SQL           | Query builder         |

## 2. Prisma

### Khái niệm

**Prisma** là ORM modern với type-safety tuyệt đối, auto-generate TypeScript types từ schema.

### Đặc điểm

- **Type-safe:** Auto-generate types
- **Schema-first:** Define schema trong file `.prisma`
- **Prisma Studio:** GUI để xem data
- **Migrations:** Built-in migration system
- **Hỗ trợ:** PostgreSQL, MySQL, SQLite, SQL Server, MongoDB

### Cài đặt

```bash
npm install prisma @prisma/client
npx prisma init
```

### Schema Definition

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Models
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  password  String
  role      Role     @default(USER)
  posts     Post[]   // Relation 1-n
  profile   Profile? // Relation 1-1
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  tags      Tag[]    // Relation n-n
  createdAt DateTime @default(now())
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String
  user   User   @relation(fields: [userId], references: [id])
  userId Int    @unique
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}

enum Role {
  USER
  ADMIN
}
```

### CRUD Operations

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// CREATE
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice',
    password: 'hashed_password',
    // Tạo relation cùng lúc
    posts: {
      create: [
        { title: 'Hello World' },
        { title: 'My Second Post' }
      ]
    },
    profile: {
      create: { bio: 'I am Alice' }
    }
  },
  // Include relations trong response
  include: {
    posts: true,
    profile: true
  }
});

// READ - Find many
const users = await prisma.user.findMany({
  where: {
    email: { contains: '@example.com' },
    role: 'USER'
  },
  include: { posts: true },
  orderBy: { createdAt: 'desc' },
  take: 10,  // Limit
  skip: 0    // Offset
});

// READ - Find one
const user = await prisma.user.findUnique({
  where: { id: 1 }
});

const user = await prisma.user.findFirst({
  where: { email: { contains: 'alice' } }
});

// UPDATE
const updated = await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' }
});

// UPDATE many
await prisma.user.updateMany({
  where: { role: 'USER' },
  data: { role: 'ADMIN' }
});

// DELETE
await prisma.user.delete({
  where: { id: 1 }
});

// DELETE many
await prisma.user.deleteMany({
  where: { email: { contains: '@test.com' } }
});

// TRANSACTION
const [user, post] = await prisma.$transaction([
  prisma.user.create({ data: { email: 'bob@example.com', password: '123' } }),
  prisma.post.create({ data: { title: 'Post', authorId: 1 } })
]);

// Interactive transaction
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ data: {...} });
  await tx.post.create({ data: { authorId: user.id, ... } });
});
```

### Migrations

```bash
# Tạo migration từ schema changes
npx prisma migrate dev --name init

# Apply migrations (production)
npx prisma migrate deploy

# Sync schema không tạo migration (dev)
npx prisma db push

# Mở Prisma Studio
npx prisma studio
```

## 3. Mongoose (MongoDB ODM)

### Khái niệm

**Mongoose** là ODM phổ biến nhất cho MongoDB, cung cấp schema validation, middleware, và nhiều features.

### Đặc điểm

- **Schema validation:** Define structure cho documents
- **Middleware (hooks):** Pre/post hooks
- **Population:** Giống JOIN trong SQL
- **Plugins:** Extensible

### Cài đặt

```bash
npm install mongoose
```

### Schema & Model

```javascript
const mongoose = require("mongoose");

// Connect
mongoose.connect("mongodb://localhost:27017/mydb");

// Schema definition
const userSchema = new mongoose.Schema(
  {
    email: {
      type: String,
      required: [true, "Email là bắt buộc"],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, "Email không hợp lệ"],
    },
    name: {
      type: String,
      required: true,
      minlength: 2,
      maxlength: 100,
    },
    password: {
      type: String,
      required: true,
      minlength: 6,
      select: false, // Không trả về khi query
    },
    role: {
      type: String,
      enum: ["user", "admin"],
      default: "user",
    },
    age: {
      type: Number,
      min: 0,
      max: 120,
    },
    posts: [
      {
        type: mongoose.Schema.Types.ObjectId,
        ref: "Post", // Reference đến Post model
      },
    ],
  },
  {
    timestamps: true, // createdAt, updatedAt tự động
  }
);

// Instance methods
userSchema.methods.isAdmin = function () {
  return this.role === "admin";
};

userSchema.methods.comparePassword = async function (candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// Static methods
userSchema.statics.findByEmail = function (email) {
  return this.findOne({ email: email.toLowerCase() });
};

// Middleware (hooks)
userSchema.pre("save", async function (next) {
  // Hash password trước khi save
  if (this.isModified("password")) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

userSchema.post("save", function (doc) {
  console.log("User saved:", doc._id);
});

// Virtual (computed field)
userSchema.virtual("fullName").get(function () {
  return `${this.firstName} ${this.lastName}`;
});

// Create model
const User = mongoose.model("User", userSchema);

module.exports = User;
```

### CRUD Operations

```javascript
// CREATE
const user = new User({
  email: "alice@example.com",
  name: "Alice",
  password: "123456",
});
await user.save();

// Hoặc
const user = await User.create({
  email: "alice@example.com",
  name: "Alice",
  password: "123456",
});

// READ
const users = await User.find({ role: "user" })
  .select("name email") // Chọn fields
  .populate("posts") // Join với posts
  .sort({ createdAt: -1 }) // Sắp xếp
  .limit(10) // Giới hạn
  .skip(0); // Offset

const user = await User.findById(id);
const user = await User.findOne({ email: "alice@example.com" });

// UPDATE
await User.findByIdAndUpdate(
  id,
  { name: "Alice Updated" },
  { new: true, runValidators: true } // Trả về doc mới, chạy validation
);

await User.updateOne({ _id: id }, { $set: { name: "Alice Updated" } });

await User.updateMany({ role: "user" }, { $set: { role: "admin" } });

// DELETE
await User.findByIdAndDelete(id);
await User.deleteOne({ _id: id });
await User.deleteMany({ role: "user" });

// AGGREGATION
const stats = await User.aggregate([
  { $match: { role: "user" } },
  {
    $group: {
      _id: "$country",
      count: { $sum: 1 },
      avgAge: { $avg: "$age" },
    },
  },
  { $sort: { count: -1 } },
]);

// TRANSACTION
const session = await mongoose.startSession();
session.startTransaction();

try {
  await User.create([{ email: "bob@example.com" }], { session });
  await Post.create([{ title: "Post", author: userId }], { session });
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

## 4. TypeORM

### Khái niệm

**TypeORM** là ORM decorator-based, giống Hibernate (Java) hoặc Entity Framework (.NET).

### Ví dụ

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, ManyToOne } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column({ nullable: true })
  name: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}

// Usage
const userRepository = AppDataSource.getRepository(User);

// Create
const user = userRepository.create({ email: "alice@example.com" });
await userRepository.save(user);

// Read
const users = await userRepository.find({
  where: { email: Like("%@example.com") },
  relations: ["posts"],
  order: { createdAt: "DESC" },
});

// Query Builder
const users = await userRepository
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.posts", "post")
  .where("user.email LIKE :email", { email: "%@example.com" })
  .getMany();
```

## 5. So sánh

| Feature        | Prisma      | Mongoose | TypeORM  |
| -------------- | ----------- | -------- | -------- |
| Type Safety    | ⭐⭐⭐⭐⭐  | ⭐⭐⭐   | ⭐⭐⭐⭐ |
| Learning Curve | Easy        | Easy     | Medium   |
| Performance    | ⭐⭐⭐⭐    | ⭐⭐⭐⭐ | ⭐⭐⭐   |
| Migrations     | ⭐⭐⭐⭐⭐  | N/A      | ⭐⭐⭐⭐ |
| Database       | SQL + Mongo | MongoDB  | SQL      |
| Flexibility    | Medium      | High     | High     |

## 6. Khi nào dùng ORM nào?

| ORM           | Dùng khi                                     |
| ------------- | -------------------------------------------- |
| **Prisma**    | TypeScript projects, cần type-safety, modern |
| **Mongoose**  | MongoDB projects                             |
| **TypeORM**   | Quen với Java/C# patterns, cần flexibility   |
| **Sequelize** | Legacy projects, cần mature solution         |
| **Drizzle**   | Cần lightweight, SQL-like syntax             |

## 7. Tổng kết

**ORM/ODM là gì?** Abstraction layer để tương tác với database bằng objects.

**Prisma:** Modern, type-safe, schema-first, auto-generate types.

**Mongoose:** ODM cho MongoDB, schema validation, middleware.

**TypeORM:** Decorator-based, giống Hibernate.

**Chọn:**

- TypeScript + SQL → Prisma
- MongoDB → Mongoose
- Quen Java/C# → TypeORM
