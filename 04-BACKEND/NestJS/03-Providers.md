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

NestJS hỗ trợ đúng **4 cách** định nghĩa custom provider. Dưới đây là phần **khái niệm chi tiết**, được viết dành cho việc trả lời phỏng vấn:

### 5.1. `useClass` — Cung cấp instance từ một Class

**Khái niệm chi tiết (dành cho phỏng vấn):**

`useClass` là cách định nghĩa provider **mặc định** trong NestJS. Khi bạn khai báo một class vào mảng `providers`, NestJS sẽ hiểu rằng: **"Bất cứ khi nào ai đó yêu cầu token này, hãy tạo một instance mới của class được chỉ định bằng cách gọi `new ClassName()` và quản lý vòng đời của nó."**

Về bản chất, `useClass` cho phép bạn **tách biệt giữa token (định danh) và implementation (lớp thực thi)**. Điều này cực kỳ mạnh mẽ vì:

- Bạn có thể dễ dàng thay đổi implementation mà không cần sửa code ở nơi inject.
- Hỗ trợ tốt nguyên tắc **Dependency Inversion** (SOLID) khi kết hợp với abstract class hoặc interface.
- NestJS sẽ tự động thực hiện **constructor injection** cho class được chỉ định (nếu class đó có dependencies).

**Đặc điểm quan trọng:**
- Mỗi lần token được inject → NestJS tạo **instance mới** (trừ khi scope là Singleton - mặc định).
- Phù hợp khi bạn muốn kiểm soát **class nào** sẽ được khởi tạo cho một token.

**Ví dụ phỏng vấn hay dùng:**
```ts
export abstract class ConfigService { abstract get(key: string): any; }

@Injectable() class DevConfigService extends ConfigService { ... }
@Injectable() class ProdConfigService extends ConfigService { ... }

// Trong module
{ provide: ConfigService, useClass: process.env.NODE_ENV === 'prod' ? ProdConfigService : DevConfigService }
```

> **Câu trả lời mẫu khi phỏng vấn:**
> "`useClass` cho phép tôi map một token sang một class cụ thể. NestJS sẽ tự động khởi tạo class đó và inject các dependency bên trong. Đây là cách linh hoạt để thay đổi implementation theo môi trường hoặc khi testing."

### 5.2. `useValue` — Cung cấp giá trị tĩnh sẵn có

**Khái niệm chi tiết (dành cho phỏng vấn):**

`useValue` là cách cung cấp một **giá trị tĩnh** (object, primitive, instance đã tạo sẵn) mà **không qua quá trình khởi tạo class** bởi NestJS. Khi bạn dùng `useValue`, bạn đang nói với NestJS: **"Đừng gọi `new` hay chạy bất kỳ logic nào. Chỉ cần trả về đúng giá trị này mỗi khi token được yêu cầu."**

Đây là cách đơn giản nhất trong 4 loại vì:
- Không có quá trình khởi tạo động.
- Giá trị được cung cấp **ngay lập tức** và **không thay đổi** trong suốt vòng đời ứng dụng.
- Thường dùng khi giá trị đã được chuẩn bị sẵn từ bên ngoài (config, connection pool, mock object...).

**Đặc điểm quan trọng:**
- Token thường là **string** hoặc **Symbol** (không phải class).
- Khi inject phải dùng `@Inject('TOKEN')`.
- Rất hữu ích trong **unit testing** để thay thế dependency thật bằng mock.

**Ví dụ phỏng vấn hay dùng:**
```ts
{ provide: 'APP_CONFIG', useValue: { apiKey: '...', timeout: 5000 } }

// Hoặc mock trong test
{ provide: CatsService, useValue: { findAll: () => [...] } }
```

> **Câu trả lời mẫu khi phỏng vấn:**
> "`useValue` dùng khi tôi muốn inject một giá trị tĩnh hoặc một object đã được khởi tạo sẵn. NestJS sẽ không tạo instance mới mà chỉ trả về đúng giá trị tôi cung cấp. Rất tiện khi cung cấp config hoặc mock dependency trong test."

### 5.3. `useFactory` — Tạo provider bằng một hàm (Linh hoạt nhất)

**Khái niệm chi tiết (dành cho phỏng vấn):**

`useFactory` là cách **linh hoạt và mạnh mẽ nhất** trong 4 loại custom provider. Thay vì chỉ định class hoặc giá trị tĩnh, bạn cung cấp một **hàm factory**. NestJS sẽ **gọi hàm này** và sử dụng **giá trị trả về** của hàm làm provider.

Điểm then chốt là:
- Hàm factory có thể chứa **bất kỳ logic nào** (sync hoặc async).
- Bạn có thể **inject các provider khác** vào hàm thông qua mảng `inject`.
- NestJS sẽ **chờ Promise resolve** nếu factory trả về Promise (rất mạnh khi kết nối database).

Về bản chất, `useFactory` cho phép bạn **tạo provider một cách động** dựa trên runtime conditions, configuration, hoặc các dependency khác.

**Đặc điểm quan trọng:**
- Mảng `inject` quyết định những gì được truyền vào hàm (theo thứ tự).
- Hỗ trợ optional dependency với `{ token: '...', optional: true }`.
- Đây là cách được dùng nhiều nhất khi làm việc với **ConfigModule** và database connection.

**Ví dụ phỏng vấn hay dùng:**
```ts
{
  provide: 'DATABASE_CONNECTION',
  useFactory: async (config: ConfigService) => {
    const options = config.get('database');
    const dataSource = new DataSource(options);
    return await dataSource.initialize();
  },
  inject: [ConfigService],
}
```

> **Câu trả lời mẫu khi phỏng vấn:**
> "`useFactory` cho phép tôi tạo provider một cách linh hoạt bằng cách cung cấp một hàm. Hàm này có thể nhận các dependency khác qua `inject`, hỗ trợ async, và tôi có thể viết logic phức tạp để quyết định instance nào được tạo. Đây là cách tôi thường dùng để tạo database connection dựa trên config động."

### 5.4. `useExisting` — Tạo bí danh (Alias) cho provider đã có

**Khái niệm chi tiết (dành cho phỏng vấn):**

`useExisting` được dùng để tạo **bí danh (alias)** cho một provider đã tồn tại. Khi bạn dùng `useExisting`, bạn đang nói với NestJS: **"Token mới này không tạo instance mới. Hãy trỏ nó về cùng một instance của provider đã được đăng ký trước đó."**

Điểm khác biệt lớn nhất so với 3 cách còn lại là:
- **Không tạo instance mới**.
- Chỉ tạo thêm một "tên gọi khác" trỏ đến provider đã có.
- Cùng một instance được chia sẻ giữa nhiều token.

Cách này rất hữu ích khi bạn muốn:
- Giữ **backward compatibility** sau khi refactor.
- Cung cấp nhiều tên cho cùng một service.
- Tạo public API cho implementation nội bộ.

**Đặc điểm quan trọng:**
- Phải có provider gốc đã được đăng ký trước.
- Cùng instance được tái sử dụng → tiết kiệm tài nguyên.

**Ví dụ phỏng vấn hay dùng:**
```ts
providers: [
  LoggerService,
  { provide: 'APP_LOGGER', useExisting: LoggerService },
]
```

> **Câu trả lời mẫu khi phỏng vấn:**
> "`useExisting` dùng để tạo alias cho một provider đã tồn tại. Nó không tạo instance mới mà chỉ trỏ token mới về cùng một instance. Tôi dùng cách này khi muốn đổi tên provider hoặc hỗ trợ nhiều module cùng dùng một service mà không tạo duplicate instance."

### 5.5. Bảng so sánh nhanh 4 loại Custom Provider

| Tiêu chí                    | `useClass`            | `useValue`                | `useFactory`                  | `useExisting`             |
|-----------------------------|-----------------------|---------------------------|-------------------------------|---------------------------|
| Tạo instance mới?           | Có                    | Không                     | Có                            | Không                     |
| Hỗ trợ async                | Không                 | Không                     | Có                            | Không                     |
| Inject dependency khác      | Có (qua class)        | Không                     | Có (qua `inject`)             | Không                     |
| Dùng cho config động        | Khó                   | Không phù hợp             | Rất tốt                       | Không                     |
| Dùng cho testing            | Tốt                   | Tốt (mock object)         | Tốt                           | Ít dùng                   |
| Backward compatibility      | Không                 | Không                     | Không                         | Rất tốt                   |
| Độ linh hoạt                | Trung bình            | Thấp                      | **Cao nhất**                  | Thấp                      |

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
