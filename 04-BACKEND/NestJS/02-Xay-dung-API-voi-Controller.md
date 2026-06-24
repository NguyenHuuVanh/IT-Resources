# Xây dựng API với Controller

> Chương này giúp bạn hiểu cách xây dựng API trong NestJS thông qua **Controller** và **Decorator** – nền tảng cốt lõi của framework. Bạn sẽ học cách tạo các route CRUD, xử lý request/response và tổ chức logic với **Todo API**. Đồng thời nắm được cách dùng query params, phân trang và trả về HTTP status code đúng chuẩn.

## 1. Controller là gì?

**Controller** chịu trách nhiệm **nhận request** từ client và **trả về response**. Mỗi controller quản lý một nhóm route liên quan (ví dụ tất cả route về `todos`).

```
Client  --->  Controller  --->  Service  --->  Database
        request          gọi              truy vấn
        response  <---           <---            <---
```

> Controller chỉ nên **điều phối** — logic nghiệp vụ đặt trong Service.

## 2. Decorator trong NestJS là gì?

**Decorator** là một hàm đặc biệt, đặt trước class/method/tham số bằng ký hiệu `@`, dùng để **gắn metadata** giúp NestJS hiểu vai trò và cách xử lý của thành phần đó.

NestJS dùng decorator để khai báo mọi thứ — đây là "ngôn ngữ" của framework.

### 2.1. Các nhóm decorator thường gặp

**Decorator khai báo class:**

| Decorator | Ý nghĩa |
|-----------|---------|
| `@Module()` | Khai báo một module |
| `@Controller()` | Khai báo một controller (kèm route prefix) |
| `@Injectable()` | Khai báo một provider có thể inject |

**Decorator định nghĩa route (HTTP method):**

| Decorator | HTTP Method |
|-----------|-------------|
| `@Get()` | GET |
| `@Post()` | POST |
| `@Put()` | PUT |
| `@Patch()` | PATCH |
| `@Delete()` | DELETE |

**Decorator lấy dữ liệu từ request (parameter decorator):**

| Decorator | Lấy gì |
|-----------|--------|
| `@Param()` | Route param (`/todos/:id`) |
| `@Query()` | Query string (`?page=1&limit=10`) |
| `@Body()` | Dữ liệu trong request body |
| `@Headers()` | HTTP headers |
| `@Req()` / `@Res()` | Đối tượng request / response gốc (Express) |

### 2.2. Cách decorator hoạt động (ngắn gọn)

Decorator thực chất là hàm nhận thông tin về thành phần được trang trí và lưu **metadata**. Khi app khởi động, NestJS đọc metadata này để biết: route nào ánh xạ tới method nào, cần inject gì, validate ra sao...

```typescript
@Controller('todos')      // metadata: prefix = 'todos'
export class TodosController {
  @Get(':id')             // metadata: GET /todos/:id
  findOne(@Param('id') id: string) {  // metadata: lấy param 'id'
    return `Todo ${id}`;
  }
}
```

## 3. Tạo Controller đầu tiên

```bash
nest g controller todos
```

```typescript
// todos.controller.ts
import { Controller, Get } from '@nestjs/common';

@Controller('todos') // route prefix: /todos
export class TodosController {
  @Get() // GET /todos
  findAll() {
    return 'Tất cả todo';
  }
}
```

## 4. Xây dựng CRUD với Todo API

Ví dụ hoàn chỉnh: một controller quản lý todo với đầy đủ Create, Read, Update, Delete.

### 4.1. DTO (Data Transfer Object)

DTO định nghĩa **hình dạng dữ liệu** client gửi lên.

```typescript
// dto/create-todo.dto.ts
export class CreateTodoDto {
  title: string;
  completed?: boolean;
}

// dto/update-todo.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateTodoDto } from './create-todo.dto';

// PartialType: tất cả field của CreateTodoDto trở thành optional
export class UpdateTodoDto extends PartialType(CreateTodoDto) {}
```

### 4.2. Service (logic nghiệp vụ)

```typescript
// todos.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { CreateTodoDto } from './dto/create-todo.dto';
import { UpdateTodoDto } from './dto/update-todo.dto';

@Injectable()
export class TodosService {
  private todos = [];
  private nextId = 1;

  findAll(page = 1, limit = 10) {
    const start = (page - 1) * limit;
    const data = this.todos.slice(start, start + limit);
    return {
      data,
      meta: {
        total: this.todos.length,
        page,
        limit,
        totalPages: Math.ceil(this.todos.length / limit),
      },
    };
  }

  findOne(id: number) {
    const todo = this.todos.find((t) => t.id === id);
    if (!todo) {
      throw new NotFoundException(`Không tìm thấy todo có id ${id}`);
    }
    return todo;
  }

  create(dto: CreateTodoDto) {
    const todo = { id: this.nextId++, completed: false, ...dto };
    this.todos.push(todo);
    return todo;
  }

  update(id: number, dto: UpdateTodoDto) {
    const todo = this.findOne(id);
    Object.assign(todo, dto);
    return todo;
  }

  remove(id: number) {
    const todo = this.findOne(id);
    this.todos = this.todos.filter((t) => t.id !== id);
    return todo;
  }
}
```

### 4.3. Controller (đầy đủ CRUD)

```typescript
// todos.controller.ts
import {
  Controller, Get, Post, Patch, Delete,
  Param, Query, Body, HttpCode, HttpStatus, ParseIntPipe,
} from '@nestjs/common';
import { TodosService } from './todos.service';
import { CreateTodoDto } from './dto/create-todo.dto';
import { UpdateTodoDto } from './dto/update-todo.dto';

@Controller('todos')
export class TodosController {
  constructor(private readonly todosService: TodosService) {}

  // GET /todos?page=1&limit=10
  @Get()
  findAll(
    @Query('page', new ParseIntPipe({ optional: true })) page = 1,
    @Query('limit', new ParseIntPipe({ optional: true })) limit = 10,
  ) {
    return this.todosService.findAll(page, limit);
  }

  // GET /todos/:id
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.todosService.findOne(id);
  }

  // POST /todos  -> 201 Created
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() dto: CreateTodoDto) {
    return this.todosService.create(dto);
  }

  // PATCH /todos/:id
  @Patch(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() dto: UpdateTodoDto,
  ) {
    return this.todosService.update(id, dto);
  }

  // DELETE /todos/:id  -> 204 No Content
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseIntPipe) id: number) {
    this.todosService.remove(id);
  }
}
```

### 4.4. Đăng ký trong Module

```typescript
// todos.module.ts
import { Module } from '@nestjs/common';
import { TodosController } from './todos.controller';
import { TodosService } from './todos.service';

@Module({
  controllers: [TodosController],
  providers: [TodosService],
})
export class TodosModule {}
```

## 5. Xử lý Request & Response

### 5.1. Lấy dữ liệu đầu vào

```typescript
@Post()
demo(
  @Body() body: CreateTodoDto,        // toàn bộ body
  @Body('title') title: string,       // 1 field trong body
  @Param('id') id: string,            // route param
  @Query('page') page: string,        // query param
  @Headers('authorization') auth: string, // header
) {}
```

### 5.2. Trả về response

NestJS **tự động** serialize giá trị return thành JSON và set status code phù hợp. Bạn chỉ cần `return` object/array — không cần `res.json()`.

```typescript
@Get()
findAll() {
  return [{ id: 1, title: 'Học NestJS' }]; // tự thành JSON
}
```

> Hạn chế dùng `@Res()` để tự trả response (mất các tính năng tự động của Nest), trừ khi thật sự cần.

## 6. Query Params & Phân trang (Pagination)

Query params là dữ liệu sau dấu `?` trong URL, thường dùng cho lọc, sắp xếp, phân trang.

```
GET /todos?page=2&limit=5&search=nest
```

```typescript
@Get()
findAll(
  @Query('page', new ParseIntPipe({ optional: true })) page = 1,
  @Query('limit', new ParseIntPipe({ optional: true })) limit = 10,
  @Query('search') search?: string,
) {
  return this.todosService.findAll(page, limit, search);
}
```

**Kết quả phân trang nên kèm metadata:**

```json
{
  "data": [ { "id": 6, "title": "..." } ],
  "meta": {
    "total": 42,
    "page": 2,
    "limit": 5,
    "totalPages": 9
  }
}
```

> Mẹo: gom các tham số phân trang vào một DTO (`PaginationQueryDto`) để tái sử dụng và validate dễ hơn.

## 7. HTTP Status Code đúng chuẩn

NestJS đặt status code mặc định: `200` cho hầu hết method, `201` cho `@Post()`. Bạn có thể tùy chỉnh.

### 7.1. Đặt status code thủ công

```typescript
import { HttpCode, HttpStatus } from '@nestjs/common';

@Post()
@HttpCode(HttpStatus.CREATED)   // 201
create() {}

@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT) // 204
remove() {}
```

### 7.2. Ném lỗi với status code chuẩn

NestJS cung cấp sẵn các exception tương ứng với status code:

```typescript
import {
  NotFoundException,      // 404
  BadRequestException,    // 400
  UnauthorizedException,  // 401
  ForbiddenException,     // 403
} from '@nestjs/common';

@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  const todo = this.todosService.findOne(id);
  if (!todo) {
    throw new NotFoundException(`Không tìm thấy todo ${id}`); // -> 404
  }
  return todo;
}
```

### 7.3. Bảng status code nên dùng cho REST API

| Hành động | Status code |
|-----------|-------------|
| GET thành công | `200 OK` |
| POST tạo mới thành công | `201 Created` |
| DELETE/PUT không trả body | `204 No Content` |
| Dữ liệu gửi lên sai | `400 Bad Request` |
| Chưa đăng nhập | `401 Unauthorized` |
| Không đủ quyền | `403 Forbidden` |
| Không tìm thấy tài nguyên | `404 Not Found` |
| Lỗi server | `500 Internal Server Error` |

## 8. Tổng kết chương

- **Controller** nhận request & trả response; logic nghiệp vụ đặt trong **Service**.
- **Decorator** là cốt lõi: `@Controller`, `@Get/@Post/@Patch/@Delete`, `@Param`, `@Query`, `@Body`...
- Xây **CRUD Todo API** chuẩn: DTO định nghĩa dữ liệu, Service xử lý, Controller điều phối.
- NestJS **tự serialize** giá trị return thành JSON — chỉ cần `return`.
- Dùng **query params** cho lọc & **phân trang** (kèm metadata).
- Trả **HTTP status code đúng chuẩn** bằng `@HttpCode`, `HttpStatus` và các Exception dựng sẵn.
