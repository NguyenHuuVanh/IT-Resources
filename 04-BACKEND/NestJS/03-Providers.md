# NestJS - Providers

> Tổng hợp kiến thức về **Providers** trong NestJS: khái niệm, cơ chế Dependency Injection, các cách định nghĩa provider, scope, ví dụ minh họa dễ hiểu và bộ câu hỏi phỏng vấn thường gặp. (Bám sát tài liệu chính thức [docs.nestjs.com/providers](https://docs.nestjs.com/providers).)

## 1. Providers là gì?

**Provider** là các class được đánh dấu bằng decorator `@Injectable()`, có thể được **tiêm (inject)** vào các class khác thông qua cơ chế **Dependency Injection (DI)** của NestJS.

Provider thường dùng để chứa **logic nghiệp vụ (business logic)**, tách biệt khỏi tầng Controller. Việc "đấu nối" (wiring up) các object này phần lớn do **chính hệ thống runtime của NestJS đảm nhiệm** — bạn không phải tự làm. Hầu hết provider là **Service**, nhưng cũng có thể là:

- **Service** — chứa business logic (phổ biến nhất)
- **Repository** — truy cập dữ liệu
- **Factory** — tạo object động
- **Helper / Utility** — hàm tiện ích dùng chung
- **Value** — giá trị cấu hình, hằng số

> Ý tưởng cốt lõi: bạn **không tự `new` object**, NestJS tự tạo và "đưa" object vào nơi cần dùng. Controller chỉ nên xử lý HTTP request và **giao phó các tác vụ phức tạp cho provider**.

> 💡 **Mẹo:** Nên tuân theo nguyên tắc **SOLID** khi thiết kế provider (gợi ý chính thức từ NestJS).

## 2. Dependency Injection & IoC

NestJS dùng **IoC Container** (Inversion of Control) để quản lý vòng đời của các provider:

- **Inversion of Control (IoC):** thay vì class tự tạo dependency của nó, việc tạo và cấp phát được "đảo ngược" cho framework quản lý.
- **Dependency Injection (DI):** cơ chế cụ thể để IoC Container "tiêm" các dependency vào class khi khởi tạo.

Nhờ khả năng của **TypeScript**, dependency được **phân giải dựa trên kiểu (type)**. Bạn chỉ cần khai báo kiểu ở constructor, NestJS sẽ tạo (hoặc tái sử dụng) instance tương ứng và tiêm vào:

```
Bạn khai báo dependency -> NestJS Container tạo instance -> Tiêm vào constructor
```

## 3. Ví dụ minh họa dễ hiểu: Quán phở

Hãy tưởng tượng một quán phở:

- **Controller** = anh phục vụ (nhận order, mang đồ ra)
- **Provider/Service** = đầu bếp (người thực sự nấu phở)

Anh phục vụ **không tự nấu**, anh ấy gọi đầu bếp. NestJS chính là "ông chủ quán" — tự thuê đầu bếp và giao cho anh phục vụ dùng.

## 4. Cách dùng — 3 bước

### Bước 1: Tạo Service (gắn `@Injectable()`)

```typescript
// pho.service.ts
import { Injectable } from '@nestjs/common';

@Injectable() // 👈 Decorator này biến class thành Provider
export class PhoService {
  private menu = ['Phở bò', 'Phở gà', 'Phở tái'];

  getAllPho(): string[] {
    return this.menu;
  }

  nauPho(loai: string): string {
    return `Đang nấu một tô ${loai} 🍜`;
  }
}
```

> **Mẹo:** Tạo service nhanh bằng CLI: `$ nest g service pho`

### Bước 2: Inject vào Controller (khai báo trong constructor)

```typescript
// pho.controller.ts
import { Controller, Get, Param } from '@nestjs/common';
import { PhoService } from './pho.service';

@Controller('pho')
export class PhoController {
  // 👇 NestJS TỰ ĐỘNG tạo PhoService và tiêm vào đây
  constructor(private readonly phoService: PhoService) {}

  @Get()
  layMenu() {
    return this.phoService.getAllPho();
  }

  @Get(':loai')
  goiMon(@Param('loai') loai: string) {
    return this.phoService.nauPho(loai);
  }
}
```

> Lưu ý từ khóa `private`: cú pháp rút gọn này vừa **khai báo** vừa **khởi tạo** thành viên `phoService` trên cùng một dòng. Bạn **không viết** `new PhoService()` — chỉ cần khai báo trong `constructor`, NestJS lo phần còn lại.

### Bước 3: Đăng ký trong Module (Provider Registration)

```typescript
// pho.module.ts
import { Module } from '@nestjs/common';
import { PhoController } from './pho.controller';
import { PhoService } from './pho.service';

@Module({
  controllers: [PhoController],
  providers: [PhoService], // 👈 Phải đăng ký provider ở đây
})
export class PhoModule {}
```

> ⚠️ Bước này **bắt buộc**. Chỉ khi service nằm trong mảng `providers`, NestJS mới có thể **phân giải dependency** cho `PhoController`. Thiếu bước này → lỗi `Nest can't resolve dependencies`.

## 5. Các loại Provider (Custom Providers) — Chi tiết

IoC Container của NestJS rất mạnh. Mọi provider đều có **2 phần chính**:

```typescript
{
  provide: TOKEN,   // ĐỊNH DANH (token) — "tên" để nơi khác yêu cầu
  useClass | useValue | useFactory | useExisting: ...  // CÁCH TẠO
}
```

- **`provide` (token)**: Có thể là **class**, **string**, hoặc **Symbol**.
- **`use___`**: Quyết định NestJS sẽ **tạo ra cái gì** và **bằng cách nào**.

NestJS hỗ trợ đúng **4 cách** định nghĩa custom provider:

| Loại          | Tạo instance mới? | Dùng khi nào                                      | Có thể inject dependency? | Độ linh hoạt | Phổ biến |
|---------------|-------------------|---------------------------------------------------|---------------------------|--------------|----------|
| **useClass**  | Có                | Override implementation, testing, interface       | Không trực tiếp           | Trung bình   | Rất cao  |
| **useValue**  | Không             | Config tĩnh, constant, mock đơn giản              | Không                     | Thấp         | Cao      |
| **useFactory**| Có                | Dynamic, async, phụ thuộc config/runtime          | Có (`inject`)             | **Cao nhất** | Rất cao  |
| **useExisting**| Không            | Alias, backward compatibility, nhiều tên          | Không                     | Thấp         | Trung bình |

### 5.1. `useClass` — Cung cấp instance từ một Class

**Khái niệm:**
Bảo NestJS: *"Khi ai đó yêu cầu token này, hãy `new` một instance của class chỉ định."*
Đây là cách **mặc định** khi bạn chỉ viết `providers: [MyService]`.

**Cú pháp:**
```ts
// Cách ngắn gọn (khuyến khích khi token = class)
providers: [CatsService]

// Cách đầy đủ (tường minh)
providers: [
  { provide: CatsService, useClass: CatsService }
]
```

**Khi nào nên dùng?**
- Service/repository thông thường.
- Muốn **thay đổi implementation** theo môi trường (dev/prod/test).
- Lập trình theo **interface / abstract class** (Dependency Inversion).
- Override provider trong **Testing** (dùng mock class).

**Tác dụng / Lợi ích:**
- NestJS tự động khởi tạo và inject các dependency của class đó.
- Cho phép **tách biệt token và implementation** → loose coupling, dễ test và bảo trì.

**Ví dụ thực tế — Đổi implementation theo môi trường:**
```ts
export abstract class ConfigService {
  abstract get(key: string): string;
}

@Injectable()
export class DevConfigService extends ConfigService {
  get(key: string) { return `[DEV] ${key}`; }
}

@Injectable()
export class ProdConfigService extends ConfigService {
  get(key: string) { return `[PROD] ${key}`; }
}

// app.module.ts
@Module({
  providers: [
    {
      provide: ConfigService, // token không đổi
      useClass: process.env.NODE_ENV === 'production'
        ? ProdConfigService
        : DevConfigService,
    },
  ],
})
export class AppModule {}
```

> 💡 `ReportService` chỉ phụ thuộc `ConfigService` (abstract) → không cần sửa code khi đổi môi trường.

### 5.2. `useValue` — Cung cấp giá trị tĩnh (không khởi tạo class)

**Khái niệm:**
Bảo NestJS: *"Đừng tạo gì cả, cứ đưa thẳng giá trị/object này cho ai yêu cầu."*
Rất hữu ích khi bạn đã có sẵn giá trị hoặc muốn inject config, constant, third-party instance.

**Cú pháp:**
```ts
providers: [
  {
    provide: 'APP_CONFIG', // thường dùng string/symbol làm token
    useValue: {
      apiKey: 'abc-123',
      timeout: 5000,
      environment: 'production',
    },
  },
]
```

**Khi nào nên dùng?**
- Cung cấp **configuration object** tĩnh.
- Inject **third-party library/instance** đã khởi tạo sẵn (Redis client, DB pool...).
- **Mock** dependency đơn giản khi viết unit test.
- Giá trị không thay đổi theo runtime.

**Tác dụng:**
- Nhanh, không tốn chi phí khởi tạo class.
- Dễ dàng thay thế implementation thật bằng mock trong test.

**Ví dụ 1 — Config tĩnh:**
```ts
@Injectable()
export class ApiService {
  constructor(@Inject('APP_CONFIG') private config) {}

  getApiKey() {
    return this.config.apiKey;
  }
}
```

**Ví dụ 2 — Mock trong Testing:**
```ts
const mockCatsService = {
  findAll: () => [{ id: 1, name: 'Mock Cat' }],
};

const moduleRef = await Test.createTestingModule({
  providers: [
    { provide: CatsService, useValue: mockCatsService },
  ],
}).compile();
```

> ⚠️ Khi token là **string** hoặc **Symbol**, bạn **bắt buộc** phải dùng `@Inject('TOKEN')` khi inject.

### 5.3. `useFactory` — Linh hoạt nhất (Khuyến nghị dùng nhiều)

**Khái niệm:**
Sử dụng một **hàm factory** để tạo provider. Hàm này có thể:
- Chứa logic phức tạp
- Gọi **async/await**
- **Inject các provider khác** qua mảng `inject`

Đây là cách **linh hoạt và mạnh mẽ nhất** trong 4 loại.

**Cú pháp cơ bản:**
```ts
{
  provide: 'DB_CONNECTION',
  useFactory: (configService: ConfigService) => {
    const options = configService.get('database');
    return createConnection(options);
  },
  inject: [ConfigService], // Nest sẽ phân giải và truyền vào
}
```

**Khi nào nên dùng? (Rất phổ biến)**
- Tạo **kết nối database** (TypeORM, Mongoose, Prisma...)
- Provider cần **config động** từ `ConfigService`
- Cần **khởi tạo bất đồng bộ** (async factory)
- Logic tạo instance phức tạp (conditional, multiple instances)
- Kết hợp với `ConfigModule.forRootAsync()`

**Tác dụng:**
- Hỗ trợ async hoàn toàn
- Có thể inject nhiều dependencies
- Rất phù hợp với pattern **Dynamic Module**

**Ví dụ thực tế — Kết nối Database với ConfigService (TypeORM style):**
```ts
@Module({
  providers: [
    ConfigService,
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        const dataSource = new DataSource({
          type: 'postgres',
          host: configService.get('DB_HOST'),
          port: configService.get<number>('DB_PORT'),
          username: configService.get('DB_USERNAME'),
          password: configService.get('DB_PASSWORD'),
          database: configService.get('DB_NAME'),
          entities: [__dirname + '/../**/*.entity{.ts,.js}'],
          synchronize: process.env.NODE_ENV !== 'production',
        });
        return await dataSource.initialize();
      },
      inject: [ConfigService],
    },
  ],
  exports: ['DATABASE_CONNECTION'],
})
export class DatabaseModule {}
```

**Ví dụ có nhiều dependency + optional:**
```ts
{
  provide: 'COMPLEX_SERVICE',
  useFactory: (
    config: ConfigService,
    logger: LoggerService,
    optionalDep?: SomeOptionalService
  ) => {
    logger.log('Creating complex service...');
    return new ComplexService(config, logger, optionalDep);
  },
  inject: [
    ConfigService,
    LoggerService,
    { token: 'OPTIONAL_DEP', optional: true },
  ],
}
```

> 💡 **Quy tắc quan trọng với `inject`:**
> Các phần tử trong mảng `inject` được truyền vào hàm `useFactory` **theo đúng thứ tự** (tên tham số chỉ để đọc code, không ảnh hưởng).

### 5.4. `useExisting` — Tạo Alias (Bí danh)

**Khái niệm:**
Tạo một **tên gọi khác** (alias) cho một provider đã tồn tại. **Cùng một instance** được chia sẻ dưới nhiều token.

**Cú pháp:**
```ts
providers: [
  DatabaseService,
  { provide: 'LEGACY_DB', useExisting: DatabaseService },
  { provide: 'DB_CONNECTION', useExisting: DatabaseService },
]
```

**Khi nào nên dùng?**
- Giữ **backward compatibility** khi refactor đổi tên token.
- Cung cấp **nhiều tên** cho cùng một service (hỗ trợ nhiều module).
- Tạo "public API" cho implementation nội bộ.

**Tác dụng:**
- **Không tạo instance mới** (khác với `useClass`).
- Giúp migrate code dễ dàng mà không phá code cũ.

**Ví dụ:**
```ts
@Module({
  providers: [
    LoggerService,
    { provide: 'APP_LOGGER', useExisting: LoggerService },
  ],
})
export class AppModule {}

@Injectable()
export class UserService {
  constructor(@Inject('APP_LOGGER') private logger: LoggerService) {}
}
```

> 🔑 **Phân biệt rõ:**
> - `useClass` → tạo **instance mới**
> - `useExisting` → **tái sử dụng** instance đã có (alias)

### 5.5. Tổng hợp so sánh 4 loại Custom Provider

| Tiêu chí                    | useClass              | useValue                  | useFactory                    | useExisting             |
|-----------------------------|-----------------------|---------------------------|-------------------------------|-------------------------|
| Tạo instance mới            | Có                    | Không                     | Có                            | Không                   |
| Hỗ trợ async                | Không                 | Không                     | Có                            | Không                   |
| Inject dependency khác      | Gián tiếp (qua class) | Không                     | Có (qua `inject`)             | Không                   |
| Dùng cho config động        | Khó                   | Không phù hợp             | Rất tốt                       | Không                   |
| Dùng cho testing            | Tốt (mock class)      | Tốt (mock object)         | Tốt                           | Ít dùng                 |
| Backward compatibility      | Không                 | Không                     | Không                         | Rất tốt                 |
| Độ phức tạp                 | Thấp                  | Thấp                      | Cao                           | Thấp                    |

### 5.6. Quy tắc quan trọng khi dùng Token

```ts
// Token là CLASS → inject trực tiếp, KHÔNG cần @Inject()
constructor(private config: ConfigService) {}

// Token là STRING hoặc SYMBOL → BẮT BUỘC dùng @Inject()
constructor(@Inject('APP_CONFIG') private config: any) {}
constructor(@Inject(SOME_SYMBOL) private config: any) {}
```

> Nên dùng `Symbol.for('APP_CONFIG')` hoặc hằng số thay vì chuỗi thô để tránh lỗi gõ sai.

## 6. Optional Providers & Property-based Injection

### Optional Provider

```ts
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService {
  constructor(
    @Optional() @Inject('HTTP_OPTIONS') private options?: any,
  ) {}
}
```

### Property-based Injection
```ts
@Injectable()
export class SomeService {
  @Inject('CONFIG') private config: any;
}
```

> Khuyến nghị: Ưu tiên **constructor-based injection** trừ khi có kế thừa phức tạp.

## 7. Scope của Provider

| Scope          | Ý nghĩa                              | Khi nào dùng                     |
|----------------|--------------------------------------|----------------------------------|
| DEFAULT        | Singleton (mặc định)                 | Hầu hết trường hợp               |
| REQUEST        | Tạo mới cho mỗi HTTP request         | Cần dữ liệu theo request         |
| TRANSIENT      | Tạo mới mỗi lần được inject          | Muốn instance riêng biệt         |
```

```ts
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {}
```

## 8. Câu hỏi phỏng vấn thường gặp

**Q: Provider trong NestJS là gì?**
> Class được đánh dấu `@Injectable()`, có thể inject vào class khác qua DI.

**Q: Có mấy cách định nghĩa provider?**
> 4 cách: `useClass`, `useValue`, `useFactory`, `useExisting`.

**Q: Khi nào dùng `useFactory` thay vì `useClass`?**
> Khi cần tạo provider **động**, **async**, hoặc phụ thuộc vào config/runtime values.

**Q: `useExisting` khác `useClass` ở điểm nào?**
> `useExisting` tạo alias (cùng instance), `useClass` tạo instance mới.

**Q: Token là string thì inject thế nào?**
> Bắt buộc dùng `@Inject('TOKEN')`.

## 9. Tổng kết

- **Provider** là nền tảng cốt lõi của NestJS.
- 4 cách custom provider cho phép bạn linh hoạt kiểm soát cách NestJS tạo và cung cấp dependency.
- `useFactory` là cách mạnh mẽ và được dùng nhiều nhất trong thực tế (đặc biệt với database + config).
- Hiểu rõ 4 cách này giúp bạn viết code **clean, testable và scalable** hơn rất nhiều.

---

*Nguồn tham khảo chính: [NestJS Official Documentation — Custom providers](https://docs.nestjs.com/fundamentals/custom-providers)*
