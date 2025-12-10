# Node.js Frameworks Comparison

## 1. Tổng quan

### Framework là gì?

**Framework** là bộ công cụ và quy tắc giúp xây dựng ứng dụng nhanh hơn. Thay vì viết mọi thứ từ đầu, framework cung cấp sẵn các tính năng như routing, middleware, error handling...

### Tại sao cần Framework?

**Không có framework:**
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // Tự parse URL
  const url = new URL(req.url, `http://${req.headers.host}`);
  
  // Tự routing
  if (req.method === 'GET' && url.pathname === '/users') {
    // Tự parse query params
    // Tự set headers
    // Tự handle errors
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ users: [] }));
  } else if (req.method === 'POST' && url.pathname === '/users') {
    // Tự parse body
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const data = JSON.parse(body);
      // ...
    });
  }
  // ... rất nhiều code
});
```

**Với framework (Express):**
```javascript
const express = require('express');
const app = express();

app.use(express.json()); // Parse JSON body

app.get('/users', (req, res) => {
  res.json({ users: [] });
});

app.post('/users', (req, res) => {
  const data = req.body; // Đã được parse
  res.status(201).json(data);
});
```

### Các Framework phổ biến

| Framework | Đặc điểm | Stars (GitHub) |
|-----------|----------|----------------|
| **Express.js** | Minimal, flexible, phổ biến nhất | 60k+ |
| **NestJS** | TypeScript, architecture giống Angular | 55k+ |
| **Fastify** | Hiệu năng cao, schema validation | 27k+ |
| **Koa.js** | Modern, lightweight, từ team Express | 34k+ |
| **Hapi.js** | Configuration-centric, enterprise | 14k+ |

## 2. Express.js

### Khái niệm

**Express.js** là framework minimal và flexible, phổ biến nhất cho Node.js. Nó cung cấp các tính năng cơ bản và cho phép tự do lựa chọn cách tổ chức code.

### Đặc điểm

- **Minimal:** Chỉ cung cấp core features
- **Flexible:** Không ép buộc cấu trúc
- **Middleware-based:** Mọi thứ là middleware
- **Ecosystem lớn:** Nhiều middleware có sẵn

### Cài đặt và sử dụng

```bash
npm install express
```

```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json()); // Parse JSON body
app.use(express.urlencoded({ extended: true })); // Parse form data

// Logging middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// Routes
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.get('/users', (req, res) => {
  res.json({ users: [] });
});

app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ id, name: 'John' });
});

app.post('/users', (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({ id: 1, name, email });
});

// Error handling middleware (4 params)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Middleware Flow

```javascript
// Middleware thực thi theo thứ tự định nghĩa
app.use((req, res, next) => {
  console.log('1. First middleware');
  next(); // Chuyển sang middleware tiếp theo
});

app.use((req, res, next) => {
  console.log('2. Second middleware');
  next();
});

app.get('/test', (req, res) => {
  console.log('3. Route handler');
  res.send('Done');
});

// Request đến GET /test:
// Output: 1, 2, 3
```

### Router

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ users: [] });
});

router.get('/:id', (req, res) => {
  res.json({ id: req.params.id });
});

router.post('/', (req, res) => {
  res.status(201).json(req.body);
});

module.exports = router;

// app.js
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);
// GET /api/users → users.js router.get('/')
```

### Ưu/Nhược điểm

| Ưu điểm | Nhược điểm |
|---------|------------|
| Dễ học, dễ bắt đầu | Không có structure chuẩn |
| Ecosystem lớn nhất | Thiếu built-in features |
| Flexible, không ép buộc | Cần setup nhiều thứ manually |
| Nhiều tài liệu, tutorials | Không có TypeScript native |

## 3. NestJS

### Khái niệm

**NestJS** là framework TypeScript-first với architecture giống Angular. Nó cung cấp structure rõ ràng, dependency injection, và nhiều built-in features cho enterprise applications.

### Đặc điểm

- **TypeScript native:** Built with TypeScript
- **Opinionated:** Có structure và conventions rõ ràng
- **Dependency Injection:** Giống Angular/Spring
- **Modular:** Tổ chức theo modules
- **Built-in features:** Validation, caching, microservices...

### Cài đặt

```bash
npm i -g @nestjs/cli
nest new project-name
```

### Cấu trúc

```
src/
├── app.module.ts        # Root module
├── app.controller.ts    # Root controller
├── app.service.ts       # Root service
├── main.ts              # Entry point
└── users/
    ├── users.module.ts
    ├── users.controller.ts
    ├── users.service.ts
    └── dto/
        ├── create-user.dto.ts
        └── update-user.dto.ts
```

### Ví dụ

```typescript
// users/dto/create-user.dto.ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;
}

// users/users.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  private users = [];

  findAll() {
    return this.users;
  }

  findOne(id: number) {
    return this.users.find(user => user.id === id);
  }

  create(createUserDto: CreateUserDto) {
    const user = {
      id: Date.now(),
      ...createUserDto
    };
    this.users.push(user);
    return user;
  }
}

// users/users.controller.ts
import { Controller, Get, Post, Body, Param, ParseIntPipe } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id);
  }

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}

// users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Export để module khác dùng
})
export class UsersModule {}
```

### Dependency Injection

```typescript
// NestJS tự động inject dependencies
@Injectable()
export class OrdersService {
  constructor(
    private usersService: UsersService,      // Auto-injected
    private productsService: ProductsService, // Auto-injected
  ) {}

  async createOrder(userId: number, items: any[]) {
    const user = await this.usersService.findOne(userId);
    // ...
  }
}
```

### Ưu/Nhược điểm

| Ưu điểm | Nhược điểm |
|---------|------------|
| TypeScript native | Learning curve cao |
| Structure rõ ràng | Boilerplate nhiều |
| DI, testing dễ | Overkill cho small projects |
| Enterprise features | Bundle size lớn |
| Microservices support | Cần hiểu decorators |

## 4. Fastify

### Khái niệm

**Fastify** là framework tập trung vào hiệu năng cao, với schema-based validation và plugin architecture.

### Đặc điểm

- **Performance:** Nhanh gấp 2x Express
- **Schema validation:** JSON Schema built-in
- **Plugin system:** Encapsulated plugins
- **TypeScript support:** Tốt
- **Logging:** Pino logger built-in

### Ví dụ

```javascript
const fastify = require('fastify')({ 
  logger: true // Built-in logging
});

// Schema validation
const userSchema = {
  body: {
    type: 'object',
    required: ['name', 'email'],
    properties: {
      name: { type: 'string', minLength: 2 },
      email: { type: 'string', format: 'email' }
    }
  },
  response: {
    201: {
      type: 'object',
      properties: {
        id: { type: 'number' },
        name: { type: 'string' },
        email: { type: 'string' }
      }
    }
  }
};

fastify.post('/users', { schema: userSchema }, async (request, reply) => {
  const { name, email } = request.body;
  return reply.code(201).send({ id: 1, name, email });
});

fastify.get('/users', async (request, reply) => {
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

### Ưu/Nhược điểm

| Ưu điểm | Nhược điểm |
|---------|------------|
| Performance cao nhất | Ecosystem nhỏ hơn Express |
| Schema validation built-in | Ít middleware có sẵn |
| Async/await native | Cần migrate từ Express |
| Logging built-in | |

## 5. Koa.js

### Khái niệm

**Koa.js** được tạo bởi team Express, là framework lightweight và modern với async/await native.

### Đặc điểm

- **Lightweight:** Không có built-in middleware
- **Modern:** Async/await từ đầu
- **Onion model:** Middleware chạy theo mô hình củ hành

### Ví dụ

```javascript
const Koa = require('koa');
const Router = require('@koa/router');
const bodyParser = require('koa-bodyparser');

const app = new Koa();
const router = new Router();

app.use(bodyParser());

// Middleware (Onion model)
app.use(async (ctx, next) => {
  console.log('1. Before');
  await next();
  console.log('5. After');
});

app.use(async (ctx, next) => {
  console.log('2. Before');
  await next();
  console.log('4. After');
});

router.get('/users', async (ctx) => {
  console.log('3. Handler');
  ctx.body = { users: [] };
});

router.post('/users', async (ctx) => {
  const { name, email } = ctx.request.body;
  ctx.status = 201;
  ctx.body = { id: 1, name, email };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);

// Request đến GET /users:
// Output: 1, 2, 3, 4, 5 (Onion model)
```

## 6. So sánh tổng quan

| Feature | Express | NestJS | Fastify | Koa |
|---------|---------|--------|---------|-----|
| Performance | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Learning Curve | Easy | Hard | Medium | Easy |
| TypeScript | Manual | Native | Good | Manual |
| Structure | Flexible | Strict | Flexible | Flexible |
| Ecosystem | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Best For | General | Enterprise | High-perf | Modern |

## 7. Khi nào dùng Framework nào?

| Framework | Dùng khi |
|-----------|----------|
| **Express** | Default choice, prototypes, small-medium projects |
| **NestJS** | Enterprise apps, microservices, team lớn cần structure |
| **Fastify** | APIs cần performance cao, real-time apps |
| **Koa** | Muốn control hoàn toàn, modern codebase |

## 8. Tổng kết

**Express:** Phổ biến nhất, dễ học, flexible, ecosystem lớn.

**NestJS:** TypeScript, enterprise, DI, structure rõ ràng.

**Fastify:** Performance cao, schema validation, modern.

**Koa:** Lightweight, modern, từ team Express.

**Chọn:**
- Mới bắt đầu → Express
- Enterprise/Team lớn → NestJS
- Cần performance → Fastify
- Muốn modern/lightweight → Koa
