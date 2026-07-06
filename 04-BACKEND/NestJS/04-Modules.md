# Modules

> Module là đơn vị tổ chức code cơ bản nhất trong NestJS — gom nhóm các thành phần liên quan thành một khối tính năng độc lập, giúp ứng dụng rõ ràng, dễ tái sử dụng và dễ mở rộng.

## 1. Module là gì?

**Module** = một "thư mục logic" gom nhóm các thành phần liên quan (Controller, Provider/Service...) thành một **khối tính năng (feature)** độc lập.

Một class trở thành module khi được gắn decorator `@Module()`. Mỗi ứng dụng NestJS luôn có ít nhất **một module gốc (root module)** là `AppModule` — đây là điểm khởi đầu để Nest xây dựng **cây ứng dụng (application graph)**.

## 2. Ví dụ dễ hiểu: Tòa nhà công ty

Hãy tưởng tượng app của bạn là một **tòa nhà**, mỗi module là một **phòng ban**:

- 🧑‍💼 Phòng Nhân sự (`UsersModule`)
- 💰 Phòng Kế toán (`PaymentsModule`)
- 🔐 Phòng Bảo vệ (`AuthModule`)

Mỗi phòng có nhân viên riêng (Service), quầy tiếp khách riêng (Controller). Khi phòng này cần nhờ phòng khác, họ phải "ký giấy hợp tác" — chính là `imports` và `exports`.

## 3. Cấu trúc một Module

```typescript
// todos.module.ts
import { Module } from '@nestjs/common';
import { TodosController } from './todos.controller';
import { TodosService } from './todos.service';

@Module({
  imports: [],                    // các module khác mà module này cần dùng
  controllers: [TodosController], // các controller thuộc module này
  providers: [TodosService],      // các provider (service) thuộc module này
  exports: [TodosService],        // provider cho phép module khác dùng
})
export class TodosModule {}
```

### 4 thuộc tính của `@Module()`

| Thuộc tính | Ý nghĩa |
|------------|---------|
| `controllers` | Khai báo các Controller thuộc module này |
| `providers` | Khai báo các Provider/Service mà module này quản lý & inject nội bộ |
| `imports` | Import các module khác để dùng **provider được export** của chúng |
| `exports` | Công khai provider của module này cho module khác `import` dùng |

## 4. Module dùng provider của module khác

Mặc định, provider chỉ "sống" trong module của nó. Muốn chia sẻ ra ngoài cần 2 bước:

**Bước 1:** Module sở hữu phải `exports` provider.

```typescript
@Module({
  providers: [UsersService],
  exports: [UsersService], // 👈 công khai ra ngoài
})
export class UsersModule {}
```

**Bước 2:** Module cần dùng phải `imports` module đó.

```typescript
@Module({
  imports: [UsersModule],   // 👈 mượn UsersModule
  providers: [AuthService], // AuthService giờ inject được UsersService
})
export class AuthModule {}
```

> Lưu ý: module **không** export provider thì dù import vẫn không inject được provider đó.

## 5. Root Module (AppModule)

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TodosModule } from './todos/todos.module';
import { UsersModule } from './users/users.module';

@Module({
  imports: [TodosModule, UsersModule], // gom các feature module
})
export class AppModule {}
```

```
AppModule (root)
   ├── TodosModule  (TodosController + TodosService)
   ├── UsersModule  (UsersController + UsersService)
   └── AuthModule   (imports UsersModule)
```

## 6. Các loại Module đặc biệt

| Loại | Mô tả |
|------|-------|
| **Feature Module** | Module cho một tính năng (UsersModule, TodosModule) — phổ biến nhất |
| **Shared Module** | Module export provider dùng chung cho nhiều nơi |
| **Global Module** | Gắn `@Global()` → provider dùng được toàn app mà không cần import lại |
| **Dynamic Module** | Module cấu hình động qua method như `forRoot()`, `forFeature()` (ví dụ `ConfigModule.forRoot()`, `TypeOrmModule.forRoot()`) |

**Global module:**

```typescript
import { Global, Module } from '@nestjs/common';

@Global() // 👈 không cần import lại ở mỗi module
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}
```

**Dynamic module (ví dụ ý tưởng):**

```typescript
@Module({})
export class DatabaseModule {
  static forRoot(options: DbOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [{ provide: 'DB_OPTIONS', useValue: options }],
      exports: ['DB_OPTIONS'],
    };
  }
}

// Dùng:
@Module({
  imports: [DatabaseModule.forRoot({ url: '...' })],
})
export class AppModule {}
```

## 7. Dynamic Module & `register()`

Module thường (static) luôn cố định — không nhận tham số cấu hình. **Dynamic Module** cho phép module **nhận cấu hình lúc import**, rất hữu ích khi cùng một module cần chạy khác nhau tùy nơi dùng (vd: `StorageModule` khi thì S3, khi thì local).

### 7.1. Khái niệm

Là module tạo **động** qua một **static method** trả về object kiểu **`DynamicModule`** (chính là `@Module({...})` nhưng dựng lúc runtime). Theo quy ước cộng đồng, method thường đặt tên:

| Method | Ý nghĩa quy ước |
|--------|-----------------|
| `register()` | Cấu hình **riêng cho từng nơi** import (mỗi nơi một config) |
| `forRoot()` | Cấu hình **1 lần toàn app** (thường ở AppModule) |
| `forFeature()` | Cấu hình bổ sung cho **một feature cụ thể** (sau `forRoot`) |
| `*Async()` | Bản async (vd `registerAsync`) — config lấy từ nguồn bất đồng bộ |

> `register` / `forRoot` / `forFeature` chỉ là **tên gọi quy ước** — kỹ thuật đều là static method trả về `DynamicModule`. Nên tuân theo quy ước cho dễ đọc.

### 7.2. `register()` — cấu hình riêng mỗi nơi import

```typescript
// storage.module.ts
import { DynamicModule, Module } from '@nestjs/common';

@Module({}) // 👈 để trống, cấu hình do register() trả về
export class StorageModule {
  static register(options: StorageOptions): DynamicModule {
    return {
      module: StorageModule,          // 👈 BẮT BUỘC — chính module này
      providers: [
        { provide: 'STORAGE_OPTIONS', useValue: options }, // biến options thành provider
        StorageService,
      ],
      exports: [StorageService],
    };
  }
}
```

**Dùng — mỗi nơi truyền config khác nhau:**

```typescript
@Module({ imports: [StorageModule.register({ driver: 's3', bucket: 'my-bucket' })] })
export class PhotoModule {}

@Module({ imports: [StorageModule.register({ driver: 'local', path: '/tmp' })] })
export class ReportModule {}
```

**Nhận config bên trong service:**

```typescript
@Injectable()
export class StorageService {
  constructor(@Inject('STORAGE_OPTIONS') private options: StorageOptions) {}

  save(file: Buffer) {
    if (this.options.driver === 's3') { /* upload S3 */ }
    else { /* lưu local */ }
  }
}
```

> 💡 **Điểm mấu chốt:** `register()` biến **tham số cấu hình** (`options`) thành một **provider** (`useValue`) — nhờ đó service inject và dùng được config đó.

### 7.3. `register()` vs `forRoot()` vs `forFeature()`

| | `register()` | `forRoot()` | `forFeature()` |
|---|-------------|-------------|----------------|
| **Mục đích** | Config riêng mỗi nơi | Config chung toàn app | Config bổ sung cho feature |
| **Số lần gọi** | Nhiều nơi, mỗi nơi 1 config | Thường 1 lần (root) | Nhiều lần theo feature |
| **Ví dụ** | `JwtModule.register()` | `TypeOrmModule.forRoot()` | `TypeOrmModule.forFeature()` |

### 7.4. `registerAsync()` — cấu hình bất đồng bộ

Khi config phải lấy từ nguồn async (vd `ConfigService` đọc `.env`), dùng bản async với `useFactory`:

```typescript
@Module({})
export class StorageModule {
  static registerAsync(options: StorageAsyncOptions): DynamicModule {
    return {
      module: StorageModule,
      imports: options.imports ?? [],
      providers: [
        {
          provide: 'STORAGE_OPTIONS',
          useFactory: options.useFactory, // 👈 hàm tạo config
          inject: options.inject ?? [],   // 👈 dependency của factory
        },
        StorageService,
      ],
      exports: [StorageService],
    };
  }
}

// Dùng — lấy config từ ConfigService
imports: [
  StorageModule.registerAsync({
    imports: [ConfigModule],
    useFactory: (config: ConfigService) => ({
      driver: config.get('STORAGE_DRIVER'),
      bucket: config.get('S3_BUCKET'),
    }),
    inject: [ConfigService],
  }),
]
```

### 7.5. Cấu trúc `DynamicModule`

```typescript
{
  module: StorageModule,   // BẮT BUỘC — class module chính
  imports: [...],          // module phụ thuộc (tùy chọn)
  providers: [...],        // provider (tùy chọn)
  exports: [...],          // provider chia sẻ ra ngoài (tùy chọn)
  global: true,            // biến thành global module (tùy chọn)
}
```

> Chỉ `module` là bắt buộc; còn lại giống hệt các thuộc tính của `@Module()` bình thường.

## 8. Vì sao dùng Module?

- **Tổ chức rõ ràng:** mỗi tính năng một khối → dễ tìm, dễ đọc.
- **Tái sử dụng:** module có thể dùng lại ở nhiều dự án.
- **Đóng gói (encapsulation):** provider không bị "rò rỉ" ra ngoài trừ khi `exports`.
- **Dễ mở rộng (scalable):** thêm tính năng = thêm module, không đụng code cũ.

## 9. Câu hỏi phỏng vấn thường gặp

**Q: Module trong NestJS là gì?**
> Là class gắn `@Module()`, gom nhóm Controller + Provider thành một khối tính năng; là đơn vị tổ chức cơ bản của ứng dụng.

**Q: 4 thuộc tính của `@Module()`?**
> `controllers`, `providers`, `imports`, `exports`.

**Q: Làm sao dùng provider của module khác?**
> Module sở hữu phải `exports` provider, module cần dùng phải `imports` module đó.

**Q: `@Global()` dùng để làm gì?**
> Đánh dấu module là global → provider được export dùng được toàn app mà không cần import lại ở từng module.

**Q: Dynamic Module là gì?**
> Module được cấu hình động khi import qua method tĩnh như `forRoot()`/`forFeature()`, trả về một `DynamicModule`. Dùng cho ConfigModule, TypeOrmModule...

**Q: Root module là gì?**
> `AppModule` — module gốc Nest dùng để bắt đầu dựng application graph.

## 10. Tổng kết

- Module = khối gom Controller + Provider thành một tính năng, khai báo bằng `@Module()`.
- 4 thuộc tính: `controllers`, `providers`, `imports`, `exports`.
- Chia sẻ provider giữa module: **export** ở nơi sở hữu, **import** ở nơi cần dùng.
- `AppModule` là gốc; có các loại đặc biệt: Feature, Shared, Global, Dynamic.
