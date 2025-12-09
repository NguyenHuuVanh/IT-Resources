# Node.js Frameworks Comparison

## 1. Express.js

### Đặc điểm

- Minimal, unopinionated
- Phổ biến nhất, ecosystem lớn
- Middleware-based architecture
- Dễ học, flexible

### Cài đặt

```bash
npm install express
```

### Ví dụ cơ bản

```javascript
const express = require("express");
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get("/users", (req, res) => {
  res.json({ users: [] });
});

app.post("/users", (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ id: 1, name, email });
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: "Something went wrong!" });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

### Middleware Flow

```javascript
// Middleware thực thi theo thứ tự
app.use((req, res, next) => {
  console.log("1. First middleware");
  next();
});

app.use((req, res, next) => {
  console.log("2. Second middleware");
  next();
});

app.get("/test", (req, res) => {
  console.log("3. Route handler");
  res.send("Done");
});

// Request đến /test:
// 1. First middleware
// 2. Second middleware
// 3. Route handler
```

### Ưu/Nhược điểm

| Ưu điểm                  | Nhược điểm                         |
| ------------------------ | ---------------------------------- |
| Dễ học, flexible         | Không có structure chuẩn           |
| Ecosystem lớn            | Thiếu built-in features            |
| Nhiều middleware có sẵn  | Cần setup nhiều thứ manually       |
| Phù hợp mọi project size | Không có TypeScript support native |

---

## 2. NestJS

### Đặc điểm

- TypeScript-first
- Architecture giống Angular (modules, decorators, DI)
- Opinionated, enterprise-ready
- Built-in support cho microservices, GraphQL, WebSocket

### Cài đặt

```bash
npm i -g @nestjs/cli
nest new project-name
```

### Ví dụ cơ bản

```typescript
// users.controller.ts
import { Controller, Get, Post, Body, Param } from "@nestjs/common";
import { UsersService } from "./users.service";
import { CreateUserDto } from "./dto/create-user.dto";

@Controller("users")
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(":id")
  findOne(@Param("id") id: string) {
    return this.usersService.findOne(+id);
  }

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}

// users.service.ts
import { Injectable } from "@nestjs/common";

@Injectable()
export class UsersService {
  private users = [];

  findAll() {
    return this.users;
  }

  findOne(id: number) {
    return this.users.find((user) => user.id === id);
  }

  create(createUserDto: CreateUserDto) {
    const user = { id: Date.now(), ...createUserDto };
    this.users.push(user);
    return user;
  }
}

// users.module.ts
import { Module } from "@nestjs/common";
import { UsersController } from "./users.controller";
import { UsersService } from "./users.service";

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

### Dependency Injection

```typescript
// NestJS tự động inject dependencies
@Injectable()
export class OrdersService {
  constructor(private usersService: UsersService, private productsService: ProductsService) {}
}
```

### Ưu/Nhược điểm

| Ưu điểm             | Nhược điểm                  |
| ------------------- | --------------------------- |
| TypeScript native   | Learning curve cao          |
| Structure rõ ràng   | Boilerplate nhiều           |
| DI, testing dễ      | Overkill cho small projects |
| Enterprise features | Bundle size lớn             |

---

## 3. Fastify

### Đặc điểm

- Hiệu năng cao (gấp 2x Express)
- Schema-based validation (JSON Schema)
- Plugin architecture
- TypeScript support tốt

### Cài đặt

```bash
npm install fastify
```

### Ví dụ cơ bản

```javascript
const fastify = require("fastify")({ logger: true });

// Schema validation
const userSchema = {
  body: {
    type: "object",
    required: ["name", "email"],
    properties: {
      name: { type: "string" },
      email: { type: "string", format: "email" },
    },
  },
  response: {
    201: {
      type: "object",
      properties: {
        id: { type: "number" },
        name: { type: "string" },
        email: { type: "string" },
      },
    },
  },
};

fastify.post("/users", { schema: userSchema }, async (request, reply) => {
  const { name, email } = request.body;
  return reply.code(201).send({ id: 1, name, email });
});

fastify.get("/users", async (request, reply) => {
  return { users: [] };
});

// Start server
const start = async () => {
  try {
    await fastify.listen({ port: 3000 });
  } catch (err) {
    fastify.log.error(err);
    process.exit(1);
  }
};
start();
```

### Plugin System

```javascript
// Tạo plugin
async function myPlugin(fastify, options) {
  fastify.decorate("utility", () => "useful");

  fastify.addHook("onRequest", async (request, reply) => {
    // Run on every request
  });
}

// Register plugin
fastify.register(myPlugin, { option: "value" });
```

### Ưu/Nhược điểm

| Ưu điểm                    | Nhược điểm                |
| -------------------------- | ------------------------- |
| Performance cao nhất       | Ecosystem nhỏ hơn Express |
| Schema validation built-in | Ít middleware có sẵn      |
| Async/await native         | Cần migrate từ Express    |
| Logging built-in           |                           |

---

## 4. Koa.js

### Đặc điểm

- Tạo bởi team Express
- Lightweight, modern
- Async/await native
- Không có built-in middleware

### Cài đặt

```bash
npm install koa
npm install koa-router koa-bodyparser
```

### Ví dụ cơ bản

```javascript
const Koa = require("koa");
const Router = require("koa-router");
const bodyParser = require("koa-bodyparser");

const app = new Koa();
const router = new Router();

app.use(bodyParser());

router.get("/users", async (ctx) => {
  ctx.body = { users: [] };
});

router.post("/users", async (ctx) => {
  const { name, email } = ctx.request.body;
  ctx.status = 201;
  ctx.body = { id: 1, name, email };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);
```

### Middleware (Onion Model)

```javascript
app.use(async (ctx, next) => {
  console.log("1. Before");
  await next();
  console.log("5. After");
});

app.use(async (ctx, next) => {
  console.log("2. Before");
  await next();
  console.log("4. After");
});

app.use(async (ctx, next) => {
  console.log("3. Handler");
});

// Output: 1, 2, 3, 4, 5
```

### Ưu/Nhược điểm

| Ưu điểm                | Nhược điểm           |
| ---------------------- | -------------------- |
| Lightweight            | Cần nhiều middleware |
| Modern async/await     | Ecosystem nhỏ        |
| Clean middleware model | Ít tài liệu          |
| Flexible               |                      |

---

## 5. Hapi.js

### Đặc điểm

- Configuration-centric
- Built-in features (validation, caching, auth)
- Enterprise-focused
- Plugin system mạnh

### Ví dụ cơ bản

```javascript
const Hapi = require("@hapi/hapi");
const Joi = require("joi");

const init = async () => {
  const server = Hapi.server({
    port: 3000,
    host: "localhost",
  });

  server.route({
    method: "GET",
    path: "/users",
    handler: (request, h) => {
      return { users: [] };
    },
  });

  server.route({
    method: "POST",
    path: "/users",
    options: {
      validate: {
        payload: Joi.object({
          name: Joi.string().required(),
          email: Joi.string().email().required(),
        }),
      },
    },
    handler: (request, h) => {
      const { name, email } = request.payload;
      return h.response({ id: 1, name, email }).code(201);
    },
  });

  await server.start();
  console.log("Server running on %s", server.info.uri);
};

init();
```

---

## 6. So sánh tổng quan

| Feature        | Express    | NestJS     | Fastify       | Koa         | Hapi         |
| -------------- | ---------- | ---------- | ------------- | ----------- | ------------ |
| Performance    | ⭐⭐⭐     | ⭐⭐       | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐    | ⭐⭐⭐       |
| Learning Curve | Easy       | Hard       | Medium        | Easy        | Medium       |
| TypeScript     | Manual     | Native     | Good          | Manual      | Good         |
| Structure      | Flexible   | Strict     | Flexible      | Flexible    | Config-based |
| Ecosystem      | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐   | ⭐⭐⭐        | ⭐⭐        | ⭐⭐⭐       |
| Best For       | General    | Enterprise | High-perf API | Modern apps | Enterprise   |

## 7. Khi nào dùng framework nào?

- **Express**: Default choice, prototypes, small-medium projects
- **NestJS**: Enterprise apps, microservices, team lớn cần structure
- **Fastify**: APIs cần performance cao, real-time apps
- **Koa**: Khi muốn control hoàn toàn, modern codebase
- **Hapi**: Enterprise apps cần built-in features
