# Chuẩn bị kiến thức trước khi học NestJS

> Chương này giúp bạn xây nền tảng vững chắc trước khi bước vào NestJS — hiểu rõ kiến thức cần có và vì sao framework này được tin dùng trong các hệ thống production.

## 1. Những khái niệm cần biết khi nhập môn Backend

Backend là phần "hậu trường" của ứng dụng: xử lý logic nghiệp vụ, lưu trữ dữ liệu và trả kết quả cho client (web, mobile...). Trước khi học NestJS, bạn nên nắm các khái niệm sau.

### 1.1. Mô hình Client - Server

```
[Client]  --- Request --->  [Server]  --- Query --->  [Database]
(Browser,                   (NestJS App)               (Postgres,
 Mobile,                                                 MySQL,
 Postman)  <--- Response ---           <--- Data ---     MongoDB)
```

- **Client:** nơi gửi yêu cầu (trình duyệt, app mobile, Postman...).
- **Server:** nơi nhận yêu cầu, xử lý logic, truy vấn database và trả về kết quả.
- **Database:** nơi lưu trữ dữ liệu lâu dài.

### 1.2. HTTP & các HTTP Method

HTTP là giao thức giao tiếp giữa client và server. Mỗi request dùng một **method** thể hiện hành động:

| Method | Mục đích | Ví dụ |
|--------|----------|-------|
| `GET` | Lấy dữ liệu | Lấy danh sách todo |
| `POST` | Tạo mới | Thêm todo mới |
| `PUT` | Cập nhật toàn bộ | Thay thế toàn bộ todo |
| `PATCH` | Cập nhật một phần | Đổi trạng thái todo |
| `DELETE` | Xoá | Xoá một todo |

### 1.3. REST API

**REST (Representational State Transfer)** là kiến trúc thiết kế API phổ biến nhất. Nguyên tắc cơ bản:

- Mỗi tài nguyên (resource) có một **endpoint** rõ ràng: `/todos`, `/users`.
- Dùng HTTP method để thể hiện hành động trên tài nguyên.
- Stateless: server không lưu trạng thái phiên giữa các request.

```
GET    /todos        -> Lấy tất cả todo
GET    /todos/1      -> Lấy todo có id = 1
POST   /todos        -> Tạo todo mới
PATCH  /todos/1      -> Cập nhật todo id = 1
DELETE /todos/1      -> Xoá todo id = 1
```

### 1.4. Request & Response

- **Request** gồm: URL, method, **headers** (metadata như `Authorization`, `Content-Type`), **query params**, **body** (dữ liệu gửi lên, thường là JSON).
- **Response** gồm: **status code**, headers và body (dữ liệu trả về).

### 1.5. HTTP Status Code

| Nhóm | Ý nghĩa | Ví dụ thường gặp |
|------|---------|------------------|
| `2xx` | Thành công | `200 OK`, `201 Created`, `204 No Content` |
| `3xx` | Chuyển hướng | `301 Moved Permanently` |
| `4xx` | Lỗi từ client | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found` |
| `5xx` | Lỗi từ server | `500 Internal Server Error` |

### 1.6. JSON

**JSON (JavaScript Object Notation)** là định dạng dữ liệu phổ biến nhất để client ↔ server trao đổi:

```json
{
  "id": 1,
  "title": "Học NestJS",
  "completed": false
}
```

### 1.7. Nền tảng cần có

- **JavaScript / TypeScript:** NestJS viết bằng TypeScript. Cần biết biến, hàm, arrow function, Promise, async/await, module import/export.
- **Node.js:** môi trường chạy JavaScript phía server.
- **npm / yarn / pnpm:** trình quản lý package.
- **TypeScript cơ bản:** kiểu dữ liệu (`string`, `number`, `boolean`), `interface`, `type`, generics, decorators.

## 2. Ôn tập các khái niệm OOP

NestJS được thiết kế theo hướng **hướng đối tượng (OOP)**. Nắm vững OOP giúp bạn hiểu cách framework tổ chức code.

### 2.1. Class & Object

- **Class:** bản thiết kế (blueprint).
- **Object:** thực thể được tạo ra từ class.

```typescript
class Todo {
  constructor(
    public id: number,
    public title: string,
    public completed: boolean = false,
  ) {}

  toggle() {
    this.completed = !this.completed;
  }
}

const todo = new Todo(1, 'Học NestJS'); // object
todo.toggle();
```

### 2.2. Bốn tính chất của OOP

| Tính chất | Ý nghĩa | Trong NestJS |
|-----------|---------|--------------|
| **Encapsulation** (Đóng gói) | Ẩn chi tiết bên trong, chỉ lộ ra interface cần thiết | `private`/`public` trong Service |
| **Inheritance** (Kế thừa) | Class con kế thừa class cha | Kế thừa base entity, base service |
| **Polymorphism** (Đa hình) | Cùng một method, hành vi khác nhau | Override method, interface |
| **Abstraction** (Trừu tượng) | Định nghĩa khung, ẩn cài đặt cụ thể | `abstract class`, `interface` |

### 2.3. Access Modifier

```typescript
class UserService {
  public name: string;       // truy cập mọi nơi (mặc định)
  private password: string;  // chỉ trong class
  protected role: string;    // trong class & class con
  readonly id: number;       // chỉ đọc, gán 1 lần
}
```

### 2.4. Interface & Abstract Class

```typescript
interface ITodo {
  id: number;
  title: string;
  completed: boolean;
}

abstract class BaseRepository<T> {
  abstract findAll(): T[];   // class con bắt buộc cài đặt
}
```

### 2.5. Decorator (cực kỳ quan trọng với NestJS)

**Decorator** là một hàm đặc biệt gắn vào class, method, property hoặc tham số để **thêm metadata / hành vi**. NestJS dùng decorator ở khắp nơi (`@Module`, `@Controller`, `@Injectable`, `@Get`...).

```typescript
@Injectable() // decorator gắn vào class
export class TodoService {}
```

> Cần bật `experimentalDecorators` và `emitDecoratorMetadata` trong `tsconfig.json` (Nest CLI đã cấu hình sẵn).

## 3. NestJS có gì đặc biệt?

NestJS là framework Node.js để xây dựng ứng dụng server-side **hiệu quả, dễ mở rộng**. Điểm mạnh:

- **TypeScript-first:** type-safe, gợi ý code tốt, ít bug.
- **Kiến trúc Modular:** chia ứng dụng thành các **module** độc lập, dễ bảo trì và mở rộng.
- **Dependency Injection (DI):** quản lý dependency tự động → loose coupling, dễ test.
- **Dựa trên nền tảng vững chắc:** mặc định dùng **Express** (có thể đổi sang **Fastify** để tăng hiệu năng).
- **Sử dụng Decorator:** cú pháp khai báo gọn gàng, rõ ràng (lấy cảm hứng từ Angular).
- **Hệ sinh thái phong phú:** hỗ trợ sẵn validation, config, ORM (TypeORM/Prisma), GraphQL, WebSocket, microservices, testing...
- **Có cấu trúc rõ ràng:** áp đặt convention giúp team lớn làm việc nhất quán → lý do NestJS được tin dùng trong **production / enterprise**.

```
                 ┌─────────────────────────┐
                 │        NestJS           │
                 │  (Modular + DI + TS)    │
                 ├─────────────────────────┤
                 │   Express / Fastify     │  <- HTTP platform
                 ├─────────────────────────┤
                 │        Node.js          │
                 └─────────────────────────┘
```

## 4. Vòng đời của một request trong NestJS

Khi một request đi vào ứng dụng NestJS, nó đi qua các tầng theo thứ tự sau (Request lifecycle):

```
   Incoming Request
        │
        ▼
   ① Middleware            (logging, cors, body parser...)
        │
        ▼
   ② Guards                (xác thực/ủy quyền - AuthGuard, RolesGuard)
        │
        ▼
   ③ Interceptors (before) (biến đổi/đo thời gian trước handler)
        │
        ▼
   ④ Pipes                 (validation & transform dữ liệu đầu vào)
        │
        ▼
   ⑤ Route Handler         (Controller -> gọi Service xử lý logic)
        │
        ▼
   ⑥ Interceptors (after)  (biến đổi response trả về)
        │
        ▼
   ⑦ Exception Filters     (bắt & xử lý lỗi nếu có)
        │
        ▼
   Outgoing Response
```

**Tóm tắt vai trò:**

| Tầng | Nhiệm vụ |
|------|----------|
| **Middleware** | Chạy trước mọi thứ, xử lý request thô (log, cors) |
| **Guards** | Quyết định request có được phép đi tiếp không (auth) |
| **Interceptors** | Bọc quanh handler — biến đổi input/output, logging, cache |
| **Pipes** | Validate & transform dữ liệu đầu vào |
| **Controller/Handler** | Nhận request, điều phối tới Service |
| **Exception Filters** | Bắt lỗi và định dạng response lỗi |

> Hiểu rõ thứ tự này giúp bạn biết đặt logic ở đâu cho đúng.

## 5. Cài đặt dự án với Nest CLI

**Nest CLI** là công cụ dòng lệnh giúp khởi tạo, phát triển và quản lý dự án NestJS.

### 5.1. Cài đặt CLI

```bash
npm install -g @nestjs/cli
nest --version
```

### 5.2. Tạo dự án mới

```bash
nest new my-project
# chọn package manager: npm / yarn / pnpm
cd my-project
```

### 5.3. Chạy dự án

```bash
npm run start         # chạy thường
npm run start:dev     # chạy watch mode (tự reload khi sửa code)
npm run start:prod    # chạy bản build production
```

Mặc định app chạy ở `http://localhost:3000`.

### 5.4. Sinh code tự động (generate)

```bash
nest generate module todos      # hoặc: nest g module todos
nest g controller todos
nest g service todos
nest g resource todos           # tạo full CRUD (module + controller + service + DTO)
```

> `nest g resource` là cách nhanh nhất để scaffold một REST resource hoàn chỉnh.

## 6. Cấu trúc thư mục & Triết lý thiết kế của Nest

### 6.1. Cấu trúc thư mục mặc định

```
my-project/
├── src/
│   ├── main.ts             # Điểm khởi động (bootstrap) ứng dụng
│   ├── app.module.ts       # Module gốc (root module)
│   ├── app.controller.ts   # Controller mẫu
│   ├── app.service.ts      # Service mẫu
│   └── todos/              # Một feature module
│       ├── todos.module.ts
│       ├── todos.controller.ts
│       ├── todos.service.ts
│       └── dto/
│           ├── create-todo.dto.ts
│           └── update-todo.dto.ts
├── test/                   # Test e2e
├── nest-cli.json
├── tsconfig.json
└── package.json
```

### 6.2. `main.ts` — điểm khởi động

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

### 6.3. Triết lý thiết kế

NestJS xoay quanh **3 khối xây dựng (building blocks)** chính:

| Khối | Vai trò |
|------|---------|
| **Module** | Gom nhóm các thành phần liên quan thành một đơn vị (feature) |
| **Controller** | Nhận request, định nghĩa route, trả response |
| **Provider/Service** | Chứa business logic, được inject vào nơi cần dùng |

**Nguyên tắc thiết kế:**

- **Modular:** mỗi tính năng là một module độc lập → dễ tái sử dụng, dễ mở rộng.
- **Separation of Concerns:** tách bạch trách nhiệm — Controller chỉ điều phối, Service xử lý logic, DTO định nghĩa dữ liệu.
- **Dependency Injection:** giảm phụ thuộc giữa các thành phần.
- **Convention over configuration:** tuân theo quy ước chung → code nhất quán giữa các dự án và thành viên.

```
AppModule (root)
   ├── TodosModule
   │     ├── TodosController
   │     └── TodosService
   ├── UsersModule
   └── AuthModule
```

## 7. Tổng kết chương

- Backend cần nắm: Client-Server, HTTP, REST API, request/response, status code, JSON.
- OOP là nền tảng tư duy của NestJS: class, 4 tính chất OOP, access modifier, **decorator**.
- NestJS đặc biệt nhờ: **TypeScript, kiến trúc modular, DI, decorator** → phù hợp hệ thống production.
- Một request đi qua: **Middleware → Guards → Interceptors → Pipes → Handler → Interceptors → Exception Filters**.
- Dùng **Nest CLI** để khởi tạo & sinh code; cấu trúc xoay quanh **Module – Controller – Provider**.
