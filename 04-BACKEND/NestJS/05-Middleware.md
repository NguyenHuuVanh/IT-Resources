# Middleware

> Middleware là hàm chạy **trước khi request đến Route Handler** — đứng ở "cửa ngõ" của ứng dụng, là tầng đầu tiên trong vòng đời request của NestJS.

## 1. Middleware là gì?

**Middleware** là hàm nhận đối tượng `request`, `response` và hàm `next()`, chạy **trước** khi request đến Controller.

```
Request → [Middleware] → Guards → Interceptors → Pipes → Handler → ...
```

NestJS middleware về bản chất tương đương **middleware của Express**.

> Middleware = trạm kiểm soát đầu tiên mà mọi request đi qua trước khi vào controller.

## 2. Ví dụ dễ hiểu: Bảo vệ ở cổng

Tòa nhà có **bảo vệ ở cổng**:
- Ghi lại ai vào (logging)
- Kiểm tra thẻ ra vào (check token)
- Cho qua (`next()`) hoặc chặn lại

Bảo vệ xử lý **trước khi** khách gặp được nhân viên (Controller).

## 3. Middleware có thể làm gì?

- Ghi log request (method, URL, thời gian)
- Chỉnh sửa `request` / `response`
- Kiểm tra, xác thực sơ bộ
- Kết thúc request sớm (không gọi `next()`)
- Gọi `next()` để chuyển sang tầng tiếp theo

## 4. Class Middleware (khuyên dùng)

```typescript
// logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`[${req.method}] ${req.originalUrl}`);
    next(); // 👈 BẮT BUỘC gọi để request đi tiếp
  }
}
```

> Nếu **không gọi `next()`** mà cũng không trả response → request bị "treo".

## 5. Đăng ký Middleware trong Module

Middleware không khai báo trong `@Module()` mà qua method `configure()` (module phải implement `NestModule`):

```typescript
// app.module.ts
import { MiddlewareConsumer, Module, NestModule, RequestMethod } from '@nestjs/common';
import { LoggerMiddleware } from './logger.middleware';

@Module({ /* ... */ })
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('todos'); // áp dụng cho route /todos
  }
}
```

### Tùy chọn áp dụng route

```typescript
// Cho mọi route
.forRoutes('*');

// Chỉ một HTTP method cụ thể
.forRoutes({ path: 'todos', method: RequestMethod.GET });

// Cho một controller
.forRoutes(TodosController);

// Loại trừ route
.apply(LoggerMiddleware)
.exclude({ path: 'todos', method: RequestMethod.POST })
.forRoutes(TodosController);
```

## 6. Functional Middleware (cho logic đơn giản)

```typescript
// logger.middleware.ts
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`[${req.method}] ${req.originalUrl}`);
  next();
}
```

```typescript
consumer.apply(logger).forRoutes('*');
```

> Dùng functional middleware khi **không cần inject dependency**.

## 7. Global Middleware

Áp dụng cho **toàn bộ app**, đăng ký trong `main.ts` (chỉ dùng được functional middleware):

```typescript
// main.ts
const app = await NestFactory.create(AppModule);
app.use(logger); // áp cho mọi request
await app.listen(3000);
```

## 8. Middleware vs Guard vs Interceptor

| Tầng | Chạy khi nào | Dùng để |
|------|--------------|---------|
| **Middleware** | Sớm nhất, trước guard | Log, parse, chỉnh request thô |
| **Guard** | Sau middleware | Quyết định cho phép truy cập (auth/role) |
| **Interceptor** | Bọc quanh handler | Biến đổi response, đo thời gian, cache |

> Middleware **không truy cập được `ExecutionContext`** nên không biết handler nào sẽ xử lý request; Guard/Interceptor thì biết.

## 9. Câu hỏi phỏng vấn thường gặp

**Q: Middleware trong NestJS là gì?**
> Hàm chạy trước Route Handler, nhận `req`, `res`, `next`; tương đương middleware Express; là tầng đầu tiên trong request lifecycle.

**Q: Có mấy loại middleware?**
> 2 loại: class middleware (`implements NestMiddleware`, inject được dependency) và functional middleware (hàm đơn giản).

**Q: Middleware đăng ký ở đâu?**
> Trong `configure(consumer: MiddlewareConsumer)` của module (implement `NestModule`), hoặc `app.use()` cho global.

**Q: Vì sao phải gọi `next()`?**
> Để chuyển request sang tầng tiếp theo. Không gọi và không trả response → request treo.

**Q: Phân biệt Middleware và Guard?**
> Middleware chạy sớm hơn, xử lý request thô, không biết handler. Guard chạy sau, quyết định quyền truy cập, truy cập được `ExecutionContext`.

## 10. Tổng kết

- Middleware chạy **đầu tiên**, trước Controller; nhận `req`, `res`, `next`.
- 2 dạng: **class** (inject được) và **functional** (đơn giản).
- Phải gọi `next()` để request đi tiếp.
- Đăng ký qua `configure(consumer)` trong module, hoặc `app.use()` cho global.
- Dùng cho: logging, CORS, parse body, xác thực sơ bộ...
