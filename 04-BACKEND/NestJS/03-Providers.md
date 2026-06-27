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

## 5. Các loại Provider (Custom Providers)

IoC Container mạnh hơn nhiều so với ví dụ cơ bản. Có **4 cách** định nghĩa provider:

| Cách | Mô tả | Dùng khi |
|------|-------|----------|
| `useClass` | Mặc định, cung cấp một class | Trường hợp thông thường |
| `useValue` | Inject một giá trị / object có sẵn | Config, hằng số, mock test |
| `useFactory` | Tạo instance động qua hàm (sync/async) | Cần khởi tạo theo điều kiện / async |
| `useExisting` | Tạo alias cho provider khác | Đặt tên thay thế cho provider |

```typescript
// useValue
{ provide: 'API_KEY', useValue: 'abc-123' }

// useFactory
{
  provide: 'DB_CONNECTION',
  useFactory: async (config: ConfigService) => {
    return await createConnection(config.get('DB_URL'));
  },
  inject: [ConfigService], // dependency mà factory cần
}

// useClass
{ provide: Logger, useClass: ProductionLogger }

// useExisting (alias)
{ provide: 'Logger', useExisting: ProductionLogger }
```

Với provider không phải class (token là string/symbol), inject bằng `@Inject()`:

```typescript
constructor(@Inject('API_KEY') private apiKey: string) {}
```

> 💡 Nên dùng `Symbol` hoặc hằng số thay cho chuỗi thô làm token để tránh gõ sai/trùng.

## 6. Optional Providers (Provider không bắt buộc)

Đôi khi một dependency **không nhất thiết phải tồn tại** — ví dụ class phụ thuộc vào một object cấu hình, nhưng nếu không có thì dùng giá trị mặc định. Khi đó việc thiếu provider **không nên gây lỗi**.

Đánh dấu bằng decorator `@Optional()`:

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(
    @Optional() @Inject('HTTP_OPTIONS') private httpClient: T,
  ) {}
}
```

## 7. Property-based Injection (Inject qua property)

Cách dùng ở trên gọi là **constructor-based injection**. Trong vài trường hợp đặc biệt — ví dụ class cấp cao phải truyền dependency lên qua `super()` ở các sub-class quá rườm rà — ta có thể dùng `@Inject()` trực tiếp ở cấp property:

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

> ⚠️ **Cảnh báo:** Nếu class **không kế thừa** class khác, hãy **ưu tiên constructor-based injection**. Constructor thể hiện rõ ràng các dependency bắt buộc, dễ đọc và dễ bảo trì hơn.

## 8. Scope của Provider

Mặc định, provider có **vòng đời gắn với vòng đời ứng dụng**: khi app bootstrap → mọi provider được khởi tạo; khi app tắt → mọi provider bị hủy.

| Scope | Ý nghĩa |
|-------|---------|
| `DEFAULT` (Singleton) | 1 instance dùng chung toàn app — **mặc định** |
| `REQUEST` | Tạo instance mới cho mỗi request |
| `TRANSIENT` | Tạo instance mới mỗi lần được inject |

```typescript
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {}
```

> ⚠️ REQUEST scope "lây lan" ngược lên: nếu A (request-scoped) được B inject thì B cũng thành request-scoped → ảnh hưởng hiệu năng. Chỉ dùng khi thật cần.

## 9. Manual Instantiation (Khởi tạo thủ công)

NestJS tự động xử lý hầu hết việc phân giải dependency. Tuy nhiên đôi khi bạn cần **bước ra ngoài** hệ thống DI để tự lấy/tự khởi tạo provider:

- **Lấy instance đang tồn tại hoặc khởi tạo động** → dùng **Module reference** (`ModuleRef`).
- **Lấy provider bên trong hàm `bootstrap()`** (vd: standalone app, dùng config service khi đang bootstrap) → xem **Standalone applications**.

## 10. Lợi ích của Provider + DI

| Không dùng DI | Dùng Provider + DI |
|---------------|--------------------|
| `new PhoService()` ở mọi nơi | NestJS tạo **1 instance dùng chung** (singleton) |
| Khó test (phải tạo object thật) | Dễ test (thay bằng mock) |
| Code rối, lặp lại | Tách bạch rõ ràng, tái sử dụng |
| Coupling chặt | **Loose coupling** |

## 11. Câu hỏi phỏng vấn thường gặp

**Q: Provider trong NestJS là gì?**
> Là class được đánh dấu `@Injectable()`, có thể được inject vào class khác qua DI. Thường chứa business logic.

**Q: Phân biệt Controller và Provider?**
> Controller xử lý request/response (tầng giao tiếp HTTP). Provider chứa logic nghiệp vụ. Controller gọi Provider để xử lý.

**Q: `@Injectable()` để làm gì?**
> Gắn metadata, đánh dấu class là provider để NestJS (IoC container) có thể quản lý và inject nó vào nơi khác.

**Q: Dependency Injection và IoC là gì?**
> IoC: việc tạo dependency được đảo ngược cho framework quản lý thay vì class tự tạo. DI: cơ chế cụ thể để container tiêm dependency vào class.

**Q: NestJS phân giải dependency dựa vào đâu?**
> Dựa trên **kiểu (type)** khai báo ở constructor, nhờ khả năng metadata của TypeScript (`reflect-metadata`).

**Q: Có mấy cách định nghĩa provider?**
> 4 cách: `useClass` (mặc định), `useValue`, `useFactory`, `useExisting`.

**Q: Custom provider dùng khi nào?**
> Khi cần inject giá trị không phải class (config, connection string), hoặc cần tạo instance theo điều kiện / async.

**Q: Khi nào dùng `@Optional()`?**
> Khi dependency không bắt buộc — thiếu cũng không gây lỗi, dùng giá trị mặc định.

**Q: Constructor-based vs Property-based injection?**
> Constructor-based là mặc định, rõ ràng dependency. Property-based (`@Inject()` ở property) chỉ nên dùng khi có kế thừa class khiến truyền qua `super()` rườm rà.

**Q: Provider có những scope nào?**
> DEFAULT (singleton — mặc định), REQUEST (mỗi request một instance), TRANSIENT (mỗi lần inject một instance).

**Q: Làm sao một module dùng được provider của module khác?**
> Module sở hữu provider phải `exports` nó, module cần dùng phải `imports` module đó.

## 12. Tổng kết

- **Provider** là xương sống của NestJS, được đánh dấu bằng `@Injectable()`.
- Nhờ **Dependency Injection**, code trở nên **dễ test, dễ bảo trì và loose coupling**.
- Quy trình dùng: gắn `@Injectable()` → khai báo trong `constructor` → đăng ký trong `providers` của Module.
- 4 cách định nghĩa provider: `useClass` / `useValue` / `useFactory` / `useExisting`.
- Bổ trợ: `@Optional()` (dependency không bắt buộc), property-based injection (khi có kế thừa), 3 scope vòng đời, và `ModuleRef` khi cần khởi tạo thủ công.
- Đây là nền tảng giúp NestJS được thiết kế theo hướng **modular và scalable**.

---

*Nguồn tham khảo: [NestJS Official Documentation — Providers](https://docs.nestjs.com/providers#dependency-injection)*
