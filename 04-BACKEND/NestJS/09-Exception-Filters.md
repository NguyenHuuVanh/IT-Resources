# NestJS - Exception Filters

> **Exception Filter** là lớp chịu trách nhiệm **bắt và xử lý tập trung mọi lỗi (exception)** ném ra trong app, cho phép bạn kiểm soát hoàn toàn nội dung response trả về client khi có lỗi.

## 1. Exception Filter là gì?

Là thành phần trong NestJS **chặn các exception chưa được xử lý** và định dạng lại response lỗi. Thay vì mỗi nơi tự xử lý lỗi rời rạc, bạn gom về **một chỗ xử lý thống nhất**.

> **Ví dụ đời thường:** Exception Filter như **bộ phận chăm sóc khách hàng**. Khi bất cứ phòng ban nào (service) gặp sự cố, thay vì để khách thấy lỗi lộn xộn, mọi sự cố được chuyển về CSKH để trả lời khách theo một **mẫu chuẩn, lịch sự**.

## 2. Cơ chế xử lý lỗi mặc định

NestJS có sẵn một **global exception filter** xử lý mọi lỗi chưa bắt. Khi bạn ném một `HttpException`, nó tự trả response chuẩn:

```typescript
// Ném lỗi
throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);

// Response tự động
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

Các exception dựng sẵn (đều kế thừa `HttpException`):

| Exception | Status |
|-----------|--------|
| `BadRequestException` | 400 |
| `UnauthorizedException` | 401 |
| `ForbiddenException` | 403 |
| `NotFoundException` | 404 |
| `InternalServerErrorException` | 500 |

## 3. Tạo Custom Exception Filter

Dùng khi muốn **tùy chỉnh định dạng response**, ghi log, hoặc xử lý loại lỗi cụ thể.

```typescript
// http-exception.filter.ts
import {
  ExceptionFilter, Catch, ArgumentsHost, HttpException,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException) // 👈 Filter này chỉ bắt HttpException
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();          // chuyển sang context HTTP
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    });
  }
}
```

**Giải thích:**
- `@Catch(HttpException)` — chỉ định loại exception cần bắt. Để trống `@Catch()` = bắt **mọi** loại.
- `ArgumentsHost` — đối tượng truy cập context hiện tại (HTTP / WebSocket / RPC).
- `host.switchToHttp()` — lấy `request` / `response` của context HTTP.

## 4. Áp dụng Filter — 3 cấp

### a) Cấp Method (1 route)

```typescript
@Post()
@UseFilters(new HttpExceptionFilter())
create() {
  throw new ForbiddenException();
}
```

### b) Cấp Controller (toàn controller)

```typescript
@UseFilters(HttpExceptionFilter) // Nest tự khởi tạo + quản lý DI
@Controller('cats')
export class CatsController {}
```

### c) Cấp Global (toàn app)

```typescript
// main.ts
const app = await NestFactory.create(AppModule);
app.useGlobalFilters(new HttpExceptionFilter());
```

Hoặc đăng ký qua module để **hỗ trợ Dependency Injection**:

```typescript
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    { provide: APP_FILTER, useClass: HttpExceptionFilter },
  ],
})
export class AppModule {}
```

> 💡 Nếu filter cần inject service (vd: `LoggerService`), **phải** dùng cách `APP_FILTER` — không dùng `new`.

## 5. Catch-All Filter (bắt mọi lỗi)

Xử lý cả lỗi **không phải `HttpException`** (vd: lỗi database):

```typescript
import {
  ExceptionFilter, Catch, ArgumentsHost, HttpException, HttpStatus,
} from '@nestjs/common';
import { HttpAdapterHost } from '@nestjs/core';

@Catch() // 👈 để trống = bắt TẤT CẢ
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(private readonly httpAdapterHost: HttpAdapterHost) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const { httpAdapter } = this.httpAdapterHost;
    const ctx = host.switchToHttp();

    const httpStatus =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const responseBody = {
      statusCode: httpStatus,
      timestamp: new Date().toISOString(),
      path: httpAdapter.getRequestUrl(ctx.getRequest()),
    };

    httpAdapter.reply(ctx.getResponse(), responseBody, httpStatus);
  }
}
```

## 6. Custom Exception (ném lỗi riêng)

Có thể tự định nghĩa exception kế thừa `HttpException`:

```typescript
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

## 7. Một số lưu ý

| Lưu ý | Chi tiết |
|-------|----------|
| **Thứ tự ưu tiên** | Method > Controller > Global |
| **DI trong global filter** | Chỉ inject được khi đăng ký qua `APP_FILTER` |
| **`@Catch()` trống** | Bắt mọi exception (catch-all) |
| **Kế thừa filter mặc định** | `extends BaseExceptionFilter` để tái dùng logic mặc định |
| **Vị trí trong lifecycle** | Filter chạy **cuối cùng**, sau khi handler/pipe/guard ném lỗi |

## 8. Câu hỏi phỏng vấn thường gặp

**Q: Exception Filter dùng để làm gì?**
> Bắt và xử lý tập trung các exception, kiểm soát định dạng response lỗi trả về client.

**Q: `@Catch()` để trống khác `@Catch(HttpException)` thế nào?**
> Để trống bắt **mọi** loại lỗi; `@Catch(HttpException)` chỉ bắt `HttpException` (và class con).

**Q: Có mấy cấp áp dụng filter?**
> 3 cấp: method (`@UseFilters`), controller (`@UseFilters`), global (`useGlobalFilters` hoặc `APP_FILTER`).

**Q: Làm sao inject service vào global filter?**
> Đăng ký qua `{ provide: APP_FILTER, useClass: ... }` trong module — không dùng `new`.

**Q: `ArgumentsHost` là gì?**
> Đối tượng giúp truy cập context hiện tại (HTTP/WS/RPC) để lấy request/response.

**Q: Filter chạy ở đâu trong request lifecycle?**
> Chạy **cuối cùng** — sau khi Guard/Pipe/Interceptor/Handler ném exception.

## 9. Tổng kết

- **Exception Filter** = lớp xử lý lỗi tập trung, kiểm soát response khi có exception.
- Tạo bằng `implements ExceptionFilter` + decorator `@Catch(...)`.
- Áp dụng 3 cấp: method / controller / global; `APP_FILTER` để hỗ trợ DI.
- `@Catch()` trống = bắt mọi lỗi; kết hợp `HttpAdapterHost` để xử lý cả lỗi ngoài HTTP.
- Chạy ở **cuối** request lifecycle, giúp response lỗi luôn nhất quán.

---

*Nguồn tham khảo: [NestJS Official Documentation — Exception Filters](https://docs.nestjs.com/exception-filters)*
