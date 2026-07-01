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

NestJS hỗ trợ đúng **4 cách** định nghĩa custom provider. Dưới đây là phần **khái niệm cốt lõi** của từng loại:

### 5.1. `useClass` — Cung cấp instance từ một Class

**Khái niệm cốt lõi:**

`useClass` bảo NestJS: **"Khi ai đó yêu cầu token này, hãy tạo một instance mới (`new`) của class mà tôi chỉ định."**

Đây là cách **mặc định và phổ biến nhất**. Khi bạn viết đơn giản `providers: [MyService]`, NestJS hiểu ngầm là `{ provide: MyService, useClass: MyService }`.

Nói cách khác, NestJS sẽ **tự động khởi tạo** class bạn cung cấp và quản lý vòng đời của nó.

**Cú pháp:**
```ts
// Cách ngắn gọn (khuyến khích)
providers: [CatsService]

// Cách đầy đủ (tường minh hơn)
providers: [
  { provide: CatsService, useClass: CatsService }
]
```

**Điểm nổi bật:**
- NestJS tự động gọi `new ClassName()` và inject các dependency bên trong class đó.
- Cho phép bạn **thay đổi implementation** mà không ảnh hưởng đến nơi sử dụng (rất mạnh khi dùng với abstract class/interface).

### 5.2. `useValue` — Cung cấp giá trị tĩnh sẵn có

**Khái niệm cốt lõi:**

`useValue` bảo NestJS: **"Đừng tạo instance nào cả. Cứ đưa thẳng giá trị/object này cho ai yêu cầu token."**

NestJS sẽ **không gọi `new`** và cũng không chạy hàm nào. Nó chỉ đơn giản trả về giá trị bạn cung cấp sẵn.

Cách này phù hợp khi bạn đã có sẵn giá trị và không muốn NestJS tự tạo.

**Cú pháp:**
```ts
providers: [
  {
    provide: 'APP_CONFIG',           // Thường dùng string/symbol làm token
    useValue: {
      apiKey: 'abc-123',
      timeout: 5000,
    },
  },
]
```

**Điểm nổi bật:**
- Rất nhanh vì không qua bước khởi tạo.
- Thường dùng để cung cấp **config tĩnh**, hằng số, hoặc **mock object** trong test.
- Token thường là string hoặc Symbol (không phải class).

### 5.3. `useFactory` — Tạo provider bằng một hàm (Linh hoạt nhất)

**Khái niệm cốt lõi:**

`useFactory` bảo NestJS: **"Hãy gọi hàm này, và dùng kết quả trả về của hàm làm provider."**

Đây là cách **linh hoạt và mạnh mẽ nhất**. Vì là hàm nên bạn có thể:
- Viết logic phức tạp
- Sử dụng `async/await`
- Nhận các dependency khác thông qua mảng `inject`

NestJS sẽ **gọi hàm** và dùng giá trị trả về (có thể là Promise) làm provider.

**Cú pháp:**
```ts
{
  provide: 'DB_CONNECTION',
  useFactory: (configService: ConfigService) => {
    const options = configService.get('database');
    return createConnection(options);
  },
  inject: [ConfigService], // NestJS sẽ phân giải và truyền vào hàm
}
```

**Điểm nổi bật:**
- Hỗ trợ **async** hoàn toàn (NestJS sẽ chờ Promise resolve).
- Có thể inject nhiều provider khác vào hàm factory.
- Đây là cách được dùng nhiều nhất khi tạo database connection, Redis client, v.v.

### 5.4. `useExisting` — Tạo bí danh (Alias) cho provider đã có

**Khái niệm cốt lõi:**

`useExisting` bảo NestJS: **"Token này chỉ là một tên gọi khác của provider đã tồn tại trước đó. Hãy trỏ về cùng một instance."**

Nó **không tạo instance mới**. Thay vào đó, nó chỉ tạo thêm một "bí danh" (alias) trỏ đến provider đã được đăng ký.

**Cú pháp:**
```ts
providers: [
  DatabaseService,
  { provide: 'LEGACY_DB_CONNECTION', useExisting: DatabaseService },
]
```

**Điểm nổi bật:**
- Cùng **một instance** được chia sẻ dưới nhiều token khác nhau.
- Rất hữu ích khi muốn giữ **backward compatibility** hoặc cung cấp nhiều tên cho cùng một service.

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
