# ORM/ODM trong Node.js

## 1. Prisma

### Đặc điểm

- Type-safe, modern ORM
- Auto-generate TypeScript types
- Schema-first approach
- Prisma Studio (GUI)
- Hỗ trợ: PostgreSQL, MySQL, SQLite, SQL Server, MongoDB

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

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  profile   Profile?
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
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String
  user   User   @relation(fields: [userId], references: [id])
  userId Int    @unique
}
```

### CRUD Operations

```typescript
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

// Create
const user = await prisma.user.create({
  data: {
    email: "alice@example.com",
    name: "Alice",
    posts: {
      create: { title: "Hello World" },
    },
  },
  include: { posts: true },
});

// Read
const users = await prisma.user.findMany({
  where: { email: { contains: "@example.com" } },
  include: { posts: true },
  orderBy: { createdAt: "desc" },
  take: 10,
  skip: 0,
});

const user = await prisma.user.findUnique({
  where: { id: 1 },
});

// Update
const updated = await prisma.user.update({
  where: { id: 1 },
  data: { name: "Alice Updated" },
});

// Delete
await prisma.user.delete({
  where: { id: 1 },
});

// Transaction
const [user, post] = await prisma.$transaction([
  prisma.user.create({ data: { email: "bob@example.com" } }),
  prisma.post.create({ data: { title: "Post", authorId: 1 } }),
]);
```

### Migration

```bash
npx prisma migrate dev --name init
npx prisma migrate deploy  # production
npx prisma db push         # sync without migration
npx prisma studio          # GUI
```

---

## 2. TypeORM

### Đặc điểm

- Decorator-based (giống Hibernate/JPA)
- Active Record & Data Mapper patterns
- Hỗ trợ nhiều databases
- Migration system

### Cài đặt

```bash
npm install typeorm reflect-metadata
npm install pg  # hoặc mysql2, sqlite3...
```

### Entity Definition

```typescript
// entities/User.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, CreateDateColumn, UpdateDateColumn } from "typeorm";
import { Post } from "./Post";

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

  @UpdateDateColumn()
  updatedAt: Date;
}

// entities/Post.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column({ nullable: true })
  content: string;

  @Column({ default: false })
  published: boolean;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}
```

### CRUD Operations

```typescript
import { AppDataSource } from "./data-source";
import { User } from "./entities/User";

const userRepository = AppDataSource.getRepository(User);

// Create
const user = userRepository.create({
  email: "alice@example.com",
  name: "Alice",
});
await userRepository.save(user);

// Read
const users = await userRepository.find({
  where: { email: Like("%@example.com") },
  relations: ["posts"],
  order: { createdAt: "DESC" },
  take: 10,
  skip: 0,
});

const user = await userRepository.findOne({
  where: { id: 1 },
});

// Update
await userRepository.update(1, { name: "Alice Updated" });

// Delete
await userRepository.delete(1);

// Query Builder
const users = await userRepository
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.posts", "post")
  .where("user.email LIKE :email", { email: "%@example.com" })
  .orderBy("user.createdAt", "DESC")
  .getMany();

// Transaction
await AppDataSource.transaction(async (manager) => {
  await manager.save(user);
  await manager.save(post);
});
```

---

## 3. Sequelize

### Đặc điểm

- Mature, feature-rich
- Promise-based
- Hỗ trợ: PostgreSQL, MySQL, MariaDB, SQLite, SQL Server

### Cài đặt

```bash
npm install sequelize
npm install pg pg-hstore  # PostgreSQL
```

### Model Definition

```javascript
const { Sequelize, DataTypes, Model } = require("sequelize");
const sequelize = new Sequelize("database", "username", "password", {
  host: "localhost",
  dialect: "postgres",
});

class User extends Model {}
User.init(
  {
    email: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
    },
    name: {
      type: DataTypes.STRING,
    },
  },
  {
    sequelize,
    modelName: "User",
    timestamps: true,
  }
);

class Post extends Model {}
Post.init(
  {
    title: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    content: DataTypes.TEXT,
    published: {
      type: DataTypes.BOOLEAN,
      defaultValue: false,
    },
  },
  {
    sequelize,
    modelName: "Post",
  }
);

// Relations
User.hasMany(Post, { foreignKey: "authorId" });
Post.belongsTo(User, { foreignKey: "authorId" });
```

### CRUD Operations

```javascript
// Create
const user = await User.create({
  email: "alice@example.com",
  name: "Alice",
});

// Read
const users = await User.findAll({
  where: {
    email: { [Op.like]: "%@example.com" },
  },
  include: [Post],
  order: [["createdAt", "DESC"]],
  limit: 10,
  offset: 0,
});

const user = await User.findByPk(1);

// Update
await User.update({ name: "Alice Updated" }, { where: { id: 1 } });

// Delete
await User.destroy({ where: { id: 1 } });

// Transaction
const t = await sequelize.transaction();
try {
  await User.create({ email: "bob@example.com" }, { transaction: t });
  await Post.create({ title: "Post", authorId: 1 }, { transaction: t });
  await t.commit();
} catch (error) {
  await t.rollback();
}
```

---

## 4. Mongoose (MongoDB ODM)

### Đặc điểm

- ODM cho MongoDB
- Schema validation
- Middleware (hooks)
- Population (like JOIN)

### Cài đặt

```bash
npm install mongoose
```

### Schema & Model

```javascript
const mongoose = require("mongoose");

// Connect
mongoose.connect("mongodb://localhost:27017/mydb");

// Schema
const userSchema = new mongoose.Schema(
  {
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
    },
    name: String,
    age: {
      type: Number,
      min: 0,
      max: 120,
    },
    posts: [
      {
        type: mongoose.Schema.Types.ObjectId,
        ref: "Post",
      },
    ],
  },
  {
    timestamps: true,
  }
);

// Methods
userSchema.methods.getFullName = function () {
  return this.name;
};

// Statics
userSchema.statics.findByEmail = function (email) {
  return this.findOne({ email });
};

// Middleware
userSchema.pre("save", function (next) {
  console.log("Before save");
  next();
});

const User = mongoose.model("User", userSchema);

// Post Schema
const postSchema = new mongoose.Schema(
  {
    title: { type: String, required: true },
    content: String,
    published: { type: Boolean, default: false },
    author: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
  },
  { timestamps: true }
);

const Post = mongoose.model("Post", postSchema);
```

### CRUD Operations

```javascript
// Create
const user = new User({ email: "alice@example.com", name: "Alice" });
await user.save();
// hoặc
const user = await User.create({ email: "alice@example.com" });

// Read
const users = await User.find({ email: /@example.com/ })
  .populate("posts")
  .sort({ createdAt: -1 })
  .limit(10)
  .skip(0);

const user = await User.findById(id);
const user = await User.findOne({ email: "alice@example.com" });

// Update
await User.findByIdAndUpdate(id, { name: "Alice Updated" }, { new: true });
await User.updateOne({ _id: id }, { name: "Alice Updated" });

// Delete
await User.findByIdAndDelete(id);
await User.deleteOne({ _id: id });

// Aggregation
const stats = await User.aggregate([
  { $match: { age: { $gte: 18 } } },
  { $group: { _id: "$country", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
]);

// Transaction
const session = await mongoose.startSession();
session.startTransaction();
try {
  await User.create([{ email: "bob@example.com" }], { session });
  await Post.create([{ title: "Post", author: userId }], { session });
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
} finally {
  session.endSession();
}
```

---

## 5. Drizzle ORM

### Đặc điểm

- Lightweight, TypeScript-first
- SQL-like syntax
- Zero dependencies
- Serverless-ready

### Cài đặt

```bash
npm install drizzle-orm
npm install drizzle-kit -D
npm install pg  # hoặc mysql2, better-sqlite3
```

### Schema Definition

```typescript
// schema.ts
import { pgTable, serial, text, boolean, timestamp, integer } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  email: text("email").notNull().unique(),
  name: text("name"),
  createdAt: timestamp("created_at").defaultNow(),
});

export const posts = pgTable("posts", {
  id: serial("id").primaryKey(),
  title: text("title").notNull(),
  content: text("content"),
  published: boolean("published").default(false),
  authorId: integer("author_id").references(() => users.id),
});
```

### CRUD Operations

```typescript
import { drizzle } from "drizzle-orm/node-postgres";
import { eq, like, desc } from "drizzle-orm";
import { users, posts } from "./schema";

const db = drizzle(pool);

// Create
const [user] = await db.insert(users).values({ email: "alice@example.com", name: "Alice" }).returning();

// Read
const allUsers = await db
  .select()
  .from(users)
  .where(like(users.email, "%@example.com"))
  .orderBy(desc(users.createdAt))
  .limit(10);

// Join
const usersWithPosts = await db.select().from(users).leftJoin(posts, eq(users.id, posts.authorId));

// Update
await db.update(users).set({ name: "Alice Updated" }).where(eq(users.id, 1));

// Delete
await db.delete(users).where(eq(users.id, 1));

// Transaction
await db.transaction(async (tx) => {
  await tx.insert(users).values({ email: "bob@example.com" });
  await tx.insert(posts).values({ title: "Post", authorId: 1 });
});
```

---

## 6. Knex.js (Query Builder)

### Đặc điểm

- Query builder (không phải full ORM)
- Flexible, SQL-like
- Migration & seeding
- Dùng làm base cho nhiều ORMs

```javascript
const knex = require("knex")({
  client: "pg",
  connection: process.env.DATABASE_URL,
});

// Select
const users = await knex("users")
  .select("*")
  .where("email", "like", "%@example.com")
  .orderBy("created_at", "desc")
  .limit(10);

// Insert
const [id] = await knex("users").insert({ email: "alice@example.com", name: "Alice" }).returning("id");

// Update
await knex("users").where({ id: 1 }).update({ name: "Alice Updated" });

// Delete
await knex("users").where({ id: 1 }).del();

// Join
const usersWithPosts = await knex("users")
  .leftJoin("posts", "users.id", "posts.author_id")
  .select("users.*", "posts.title");

// Raw query
const result = await knex.raw("SELECT * FROM users WHERE id = ?", [1]);
```

---

## 7. So sánh tổng quan

| Feature        | Prisma      | TypeORM  | Sequelize | Mongoose | Drizzle    |
| -------------- | ----------- | -------- | --------- | -------- | ---------- |
| Type Safety    | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐ | ⭐⭐      | ⭐⭐⭐   | ⭐⭐⭐⭐⭐ |
| Learning Curve | Easy        | Medium   | Medium    | Easy     | Easy       |
| Performance    | ⭐⭐⭐⭐    | ⭐⭐⭐   | ⭐⭐⭐    | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Flexibility    | Medium      | High     | High      | High     | High       |
| Migration      | ⭐⭐⭐⭐⭐  | ⭐⭐⭐⭐ | ⭐⭐⭐⭐  | N/A      | ⭐⭐⭐⭐   |
| Database       | SQL + Mongo | SQL      | SQL       | MongoDB  | SQL        |

## 8. Khi nào dùng ORM nào?

- **Prisma**: TypeScript projects, cần type-safety, developer experience tốt
- **TypeORM**: Quen với Java/C# patterns, cần flexibility
- **Sequelize**: Legacy projects, cần mature solution
- **Mongoose**: MongoDB projects
- **Drizzle**: Cần lightweight, serverless, SQL-like syntax
- **Knex**: Cần query builder thuần, không cần full ORM
