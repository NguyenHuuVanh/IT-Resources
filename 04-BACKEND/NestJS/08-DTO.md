# DTO (Data Transfer Object)

> DTO định nghĩa hình dạng dữ liệu khi truyền giữa các tầng — phổ biến nhất là dữ liệu client gửi lên server (body, query) hoặc server trả về client.

## 1. DTO là gì?

**DTO (Data Transfer Object)** = "bản hợp đồng dữ liệu" — quy định request/response phải có những field nào, kiểu gì.

Trong NestJS, DTO thường là một **class** (không phải interface) — vì class tồn tại lúc runtime nên có thể gắn decorator để **validate** (class-validator) và **transform** (class-transformer).

## 2. Vì sao cần DTO?

| Không có DTO | Có DTO |
|--------------|--------|
| `@Body() body: any` — không biết có field gì | Rõ ràng field nào, kiểu gì |
| Không validate được | Validate tự động bằng decorator |
| Dễ lỗi, khó bảo trì | Type-safe, IDE gợi ý tốt |
| Code rối | Tách bạch, dễ đọc |

## 3. Cách tạo DTO

### Bước 1: Tạo file DTO (quy ước đặt trong thư mục `dto/`)

```typescript
// dto/create-user.dto.ts
import { IsString, IsEmail, IsInt, Min, IsOptional, Length } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @Length(2, 50)
  name: string;

  @IsEmail({}, { message: 'Email không hợp lệ' })
  email: string;

  @IsInt()
  @Min(18)
  age: number;

  @IsOptional() // không bắt buộc
  @IsString()
  bio?: string;
}
```

### Bước 2: Dùng DTO trong Controller

```typescript
// users.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    // createUserDto đã được validate xong, dữ liệu sạch
    return createUserDto;
  }
}
```

### Bước 3: Bật ValidationPipe (trong `main.ts`)

```typescript
import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,            // bỏ field thừa không khai báo trong DTO
    forbidNonWhitelisted: true, // báo lỗi nếu có field lạ
    transform: true,            // tự ép kiểu theo DTO
  }),
);
```

## 4. Ví dụ minh họa: API tạo User

**Request hợp lệ (`POST /users`):**

```json
{ "name": "Việt Anh", "email": "vietanh@example.com", "age": 25 }
```
✅ Vào Controller bình thường.

**Request sai:**

```json
{ "name": "A", "email": "khong-phai-email", "age": 15 }
```
❌ Tự trả về **400 Bad Request**:

```json
{
  "statusCode": 400,
  "message": [
    "name must be longer than or equal to 2 characters",
    "Email không hợp lệ",
    "age must not be less than 18"
  ],
  "error": "Bad Request"
}
```

## 5. `whitelist: true` có khác `@Expose()` / `@Exclude()` không?

**Khác hoàn toàn** — khác mục đích, hướng dữ liệu lẫn thư viện.

| | `whitelist: true` | `@Expose()` / `@Exclude()` |
|---|---|---|
| **Thuộc về** | `ValidationPipe` (class-validator) | class-transformer |
| **Hướng dữ liệu** | **Input** — dữ liệu client gửi LÊN | Thường cho **Output** — dữ liệu trả VỀ |
| **Mục đích** | Lọc bỏ field **không có decorator validate** trong DTO | Chọn field nào được **hiện/ẩn** khi serialize |

### `whitelist: true` — làm sạch dữ liệu vào

Tự động **xoá field thừa** mà client gửi lên nhưng DTO không khai báo.

```jsonc
// DTO chỉ có name, email
// Client gửi: { "name": "Việt", "email": "v@a.com", "isAdmin": true }
// Sau ValidationPipe(whitelist: true): { "name": "Việt", "email": "v@a.com" }
```

> Tác dụng bảo mật: chặn client "nhét" field nguy hiểm như `isAdmin`, `role`...

### `@Exclude()` / `@Expose()` — kiểm soát dữ liệu ra

```typescript
import { Exclude } from 'class-transformer';

export class UserEntity {
  id: number;
  name: string;

  @Exclude() // 👈 KHÔNG trả password ra client
  password: string;
}
```

Kích hoạt bằng `ClassSerializerInterceptor`:

```typescript
@UseInterceptors(ClassSerializerInterceptor)
@Get(':id')
findOne() { return this.usersService.findOne(1); }
// DB: { id, name, password } -> Response: { id, name }
```

### Tóm gọn

- `whitelist` = **lọc rác đầu VÀO** (chặn field thừa client gửi lên).
- `@Exclude/@Expose` = **kiểm soát đầu RA** (ẩn/hiện field khi trả về).
- Hai cái **bổ trợ nhau**, không thay thế nhau.

## 6. Tái sử dụng DTO — `PartialType`

Khi update, các field thường là **optional**. Dùng `PartialType` để kế thừa và biến mọi field thành tùy chọn:

```typescript
// dto/update-user.dto.ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto';

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

Các tiện ích khác: `PickType` (lấy vài field), `OmitType` (loại vài field), `IntersectionType` (gộp).

### Nếu KHÔNG dùng `PartialType` thì sao?

`PartialType` chỉ là tiện ích, không bắt buộc. Không dùng sẽ có hệ quả:

**Vấn đề 1 — mọi field thành bắt buộc:** nếu dùng lại `CreateUserDto`, client buộc phải gửi đủ mọi field dù chỉ sửa 1 trường.

```jsonc
// PATCH /users/1 — chỉ muốn đổi tên: { "name": "Tên mới" }
// ❌ Lỗi 400: "email must be an email", "age must not be empty"...
```

**Vấn đề 2 — phải viết lại DTO thủ công (lặp code):**

```typescript
export class UpdateUserDto {
  @IsOptional() @IsString() @Length(2, 50)
  name?: string;

  @IsOptional() @IsEmail()
  email?: string;

  @IsOptional() @IsInt() @Min(18)
  age?: number;
}
```

➡️ Dài, lặp luật validate, khi `CreateUserDto` đổi phải sửa cả 2 nơi (dễ lệch).

**Dùng `PartialType`:** tự biến mọi field thành optional, kế thừa toàn bộ luật validate, tự đồng bộ khi DTO gốc đổi.

## 7. Câu hỏi phỏng vấn thường gặp

**Q: DTO là gì? Vì sao dùng class chứ không phải interface?**
> DTO định nghĩa hình dạng dữ liệu truyền giữa các tầng. Dùng class vì class tồn tại lúc runtime nên gắn được decorator để validate/transform; interface bị xoá khi biên dịch.

**Q: `whitelist` khác `@Exclude/@Expose` thế nào?**
> `whitelist` lọc field thừa ở dữ liệu đầu vào (class-validator); `@Exclude/@Expose` ẩn/hiện field khi serialize đầu ra (class-transformer).

**Q: `PartialType` để làm gì?**
> Tạo DTO update bằng cách kế thừa DTO create và biến mọi field thành optional, tránh viết lại và giữ đồng bộ.

**Q: Không dùng PartialType cho update thì sao?**
> Vẫn chạy nhưng phải tự viết lại DTO và thêm `@IsOptional()` cho từng field, dễ lặp code và lệch với DTO gốc.

## 8. Tổng kết

- **DTO** định nghĩa hình dạng dữ liệu truyền giữa client ↔ server.
- Dùng **class** để gắn được decorator validate/transform.
- Tạo trong thư mục `dto/`, dùng với `@Body()`, kết hợp **ValidationPipe**.
- `whitelist` (lọc input) khác `@Expose/@Exclude` (kiểm soát output) — bổ trợ nhau.
- Tái sử dụng bằng `PartialType`, `PickType`, `OmitType`.
