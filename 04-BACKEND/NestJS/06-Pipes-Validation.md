# Pipes, class-transformer & class-validator

> Bộ ba làm việc cùng nhau để **validate** (kiểm tra) và **transform** (biến đổi) dữ liệu đầu vào trước khi vào Controller.

## 1. Pipe là gì?

**Pipe** là class gắn `@Injectable()`, implement interface `PipeTransform`, chạy ngay **trước Route Handler** trong vòng đời request. Pipe có 2 nhiệm vụ:

- **Transformation:** biến đổi dữ liệu (vd: chuỗi `"5"` → số `5`).
- **Validation:** kiểm tra dữ liệu hợp lệ; nếu sai thì ném exception.

```
... → Guards → Interceptors → [Pipes] → Route Handler → ...
```

### Pipe dựng sẵn của NestJS

| Pipe | Công dụng |
|------|-----------|
| `ParseIntPipe` | Ép kiểu sang số nguyên |
| `ParseBoolPipe` | Ép kiểu sang boolean |
| `ParseUUIDPipe` | Kiểm tra UUID hợp lệ |
| `ParseArrayPipe` | Ép kiểu mảng |
| `DefaultValuePipe` | Gán giá trị mặc định |
| `ValidationPipe` | Validate object theo DTO (dùng với class-validator) |

```typescript
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  // id chắc chắn là number, nếu không sẽ trả 400
  return this.service.findOne(id);
}
```

### Custom Pipe

```typescript
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class PositiveIntPipe implements PipeTransform {
  transform(value: any) {
    const val = parseInt(value, 10);
    if (isNaN(val) || val <= 0) {
      throw new BadRequestException('Giá trị phải là số nguyên dương');
    }
    return val;
  }
}
```

## 2. class-validator

Thư viện cung cấp **decorator** để khai báo luật validate ngay trên DTO.

```bash
npm install class-validator class-transformer
```

```typescript
// dto/create-user.dto.ts
import { IsString, IsEmail, IsInt, Min, Max, IsOptional, Length } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @Length(2, 50)
  name: string;

  @IsEmail({}, { message: 'Email không hợp lệ' })
  email: string;

  @IsInt()
  @Min(18)
  @Max(100)
  age: number;

  @IsOptional()
  @IsString()
  bio?: string;
}
```

### Các decorator validate hay dùng

| Decorator | Kiểm tra |
|-----------|----------|
| `@IsString()` `@IsInt()` `@IsBoolean()` | Kiểu dữ liệu |
| `@IsEmail()` `@IsUrl()` `@IsUUID()` | Định dạng |
| `@IsNotEmpty()` `@IsOptional()` | Bắt buộc / tùy chọn |
| `@Min()` `@Max()` `@Length()` | Giới hạn giá trị/độ dài |
| `@IsEnum()` `@IsArray()` | Enum / mảng |
| `@IsDateString()` | Chuỗi ngày tháng |

## 3. class-transformer

Thư viện **biến đổi** giữa plain object (JSON) và instance của class, và chuyển kiểu dữ liệu.

```typescript
import { Type, Transform, Exclude, Expose } from 'class-transformer';

export class PaginationDto {
  @Type(() => Number)     // "10" -> 10
  @IsInt()
  @Min(1)
  page: number;

  @Transform(({ value }) => value.trim()) // cắt khoảng trắng
  @IsString()
  search: string;
}
```

- `@Type()`: ép kiểu (quan trọng vì query/param luôn là string).
- `@Transform()`: biến đổi tùy ý.
- `@Exclude()` / `@Expose()`: ẩn/hiện field khi serialize response (vd ẩn `password`).

## 4. Ghép chung: ValidationPipe + DTO

`ValidationPipe` là "chất keo" — nó dùng class-transformer để biến body thành instance DTO, rồi dùng class-validator để kiểm tra.

### Bật global (khuyên dùng) trong `main.ts`

```typescript
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,            // loại bỏ field không khai báo trong DTO
      forbidNonWhitelisted: true, // báo lỗi nếu có field thừa
      transform: true,            // tự ép kiểu theo DTO (cần class-transformer)
      transformOptions: { enableImplicitConversion: true },
    }),
  );
  await app.listen(3000);
}
```

### Controller chỉ cần khai báo DTO

```typescript
@Post()
create(@Body() dto: CreateUserDto) {
  // dto đã được validate + transform xong, sạch sẽ
  return this.usersService.create(dto);
}
```

### Khi dữ liệu sai → tự động trả 400

```json
{
  "statusCode": 400,
  "message": ["Email không hợp lệ", "age must not be less than 18"],
  "error": "Bad Request"
}
```

## 5. Luồng hoạt động tổng quát

```
Request body (JSON)
      │
      ▼
ValidationPipe
      │  ① class-transformer: plain object → instance DTO (+ ép kiểu)
      │  ② class-validator: kiểm tra theo các decorator
      ▼
  Hợp lệ? ── Không ──→ ném BadRequestException (400)
      │
     Có
      ▼
Route Handler (Controller) nhận DTO sạch
```

## 6. Phạm vi áp dụng Pipe

| Cấp | Cách dùng |
|-----|-----------|
| **Param** | `@Param('id', ParseIntPipe)` |
| **Handler** | `@UsePipes(new ValidationPipe())` trên method |
| **Controller** | `@UsePipes(...)` trên class |
| **Global** | `app.useGlobalPipes(...)` trong `main.ts` |

## 7. Câu hỏi phỏng vấn thường gặp

**Q: Pipe trong NestJS là gì?**
> Class implement `PipeTransform`, chạy trước handler, dùng để transform và/hoặc validate dữ liệu đầu vào.

**Q: Phân biệt class-validator và class-transformer?**
> class-validator khai báo & kiểm tra luật validate bằng decorator; class-transformer biến đổi/ép kiểu giữa plain object và class instance.

**Q: ValidationPipe hoạt động thế nào?**
> Dùng class-transformer chuyển body thành instance DTO (kèm ép kiểu nếu bật `transform`), rồi dùng class-validator kiểm tra; sai thì ném `BadRequestException` (400).

**Q: `whitelist` và `forbidNonWhitelisted` để làm gì?**
> `whitelist`: loại bỏ field không có trong DTO. `forbidNonWhitelisted`: báo lỗi nếu request có field lạ.

**Q: Vì sao cần `@Type()` cho query/param?**
> Vì query/param luôn là string; `@Type(() => Number)` (hoặc `transform: true`) giúp ép về đúng kiểu để validate chính xác.

**Q: Làm sao ẩn field nhạy cảm (password) khi trả response?**
> Dùng `@Exclude()` của class-transformer kết hợp `ClassSerializerInterceptor`.

## 8. Tổng kết

| Thành phần | Vai trò |
|------------|---------|
| **Pipe** | Transform & validate dữ liệu đầu vào, chạy trước handler |
| **class-validator** | Khai báo luật validate bằng decorator trên DTO |
| **class-transformer** | Biến đổi & ép kiểu giữa plain object ↔ class instance |
| **ValidationPipe** | Kết hợp cả hai: tự validate + transform DTO |

3 option quan trọng của `ValidationPipe`: `whitelist`, `forbidNonWhitelisted`, `transform`.
