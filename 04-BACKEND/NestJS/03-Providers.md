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

IoC Container mạnh hơn nhiều so với ví dụ cơ bản. Mọi provider đều có **2 phần**:

```typescript
{
  provide: TOKEN,   // 👈 ĐỊNH DANH — cái "tên" để nơi khác yêu cầu
  use___: ...,      // 👈 CÁCH TẠO — Nest dùng cái này để tạo ra giá trị
}
```

- **`provide` (token):** có thể là class, abstract class, string, hoặc symbol — "chìa khóa" để inject.
- **`use___`:** quyết định Nest **tạo ra cái gì** và **bằng cách nào**. Có đúng **4 kiểu** dưới đây.

| Cách | Tạo ra gì | Dùng khi |
|------|-----------|----------|
| `useClass` | Instance mới của một class | Trường hợp thông thường, đổi impl theo môi trường |
| `useValue` | Giá trị / object có sẵn | Config, hằng số, mock test |
| `useFactory` | Kết quả trả về từ một hàm (sync/async) | Cần logic / async / phụ thuộc provider khác |
| `useExisting` | Alias trỏ về provider khác | Đặt tên thay thế cho provider có sẵn |

### 5.1. `useClass` — Tạo instance từ một class

**Khái niệm:** Bảo NestJS *"Khi ai yêu cầu token này, hãy `new` ra một instance của class chỉ định."* Đây là cách **mặc định và phổ biến nhất**.

**Cách dùng — dạng rút gọn vs đầy đủ:**

```typescript
// Dạng rút gọn (khi token == class)
providers: [CatsService]

// Dạng đầy đủ — tương đương
providers: [
  { provide: CatsService, useClass: CatsService }
]
```

**Khi nào dùng:** service/repository thông thường; đổi implementation theo môi trường (dev/prod/test); lập trình theo interface (abstract class).

**Tác dụng:** Nest tự khởi tạo & tự phân giải dependency của class đó; cho phép **tách token khỏi implementation** → loose coupling.

**Ví dụ — đổi class theo môi trường:**

```typescript
// Token là abstract class (interface chung)
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
      provide: ConfigService,                 // 👈 token KHÔNG đổi
      useClass:
        process.env.NODE_ENV === 'production'
          ? ProdConfigService                 // prod → class này
          : DevConfigService,                 // dev → class kia
    },
  ],
})
export class AppModule {}

// Nơi dùng — KHÔNG cần biết đang là Dev hay Prod
@Injectable()
export class ReportService {
  constructor(private config: ConfigService) {} // chỉ phụ thuộc token
}
```

> 💡 `ReportService` đổi môi trường mà **không sửa một dòng nào** — đúng tinh thần **Dependency Inversion** (chữ D trong SOLID).

> **📦 Vì sao dùng abstract class làm DI token?**
>
> **Abstract class** là class **không thể `new` trực tiếp**, chỉ làm khuôn mẫu để kế thừa. Nó có thể chứa cả **abstract method** (bắt class con phải triển khai) lẫn **method có sẵn logic** (chia sẻ cho con).
>
> | | Interface | Abstract class |
> |---|-----------|----------------|
> | Chứa logic (implementation)? | ❌ Chỉ khai báo "hình dạng" | ✅ Có thể chứa method có logic |
> | Tồn tại lúc runtime? | ❌ Bị xóa khi compile | ✅ **Tồn tại thật** trong JS |
> | Kế thừa | `implements` (nhiều) | `extends` (một) |
>
> **Lý do cốt lõi:** NestJS phân giải dependency dựa trên token tồn tại lúc **runtime**. Vì **interface bị xóa sau khi compile** → không dùng làm token được. Còn **abstract class tồn tại thật** trong JavaScript → vừa đóng vai trò "hợp đồng" (contract) ép các implementation tuân theo cùng cấu trúc, vừa dùng được làm token. Đó là lý do ví dụ trên chọn `abstract class ConfigService` làm token thay vì `interface`.
>
> ```typescript
> abstract class ConfigService {        // 👈 vừa là contract, vừa là token
>   abstract get(key: string): string;
> }
> // interface ConfigService {...}      // ❌ KHÔNG dùng làm token được
> ```

### 5.2. `useValue` — Cung cấp một giá trị có sẵn

**Khái niệm:** Bảo NestJS *"Đừng tạo gì cả, cứ đưa thẳng giá trị/object này cho ai yêu cầu token."*

**Cách dùng:**

```typescript
providers: [
  { provide: 'APP_CONFIG', useValue: { apiKey: 'abc-123', timeout: 5000 } }
]
```

**Khi nào dùng:** hằng số / object cấu hình tĩnh; inject một thư viện bên ngoài đã khởi tạo sẵn; **mock** dependency khi test.

**Tác dụng:** không qua bước khởi tạo — đưa nguyên giá trị; cực tiện để **thay đồ thật bằng đồ giả** lúc test.

**Ví dụ — config + mock test:**

```typescript
// 1. Inject object cấu hình tĩnh
const configProvider = {
  provide: 'APP_CONFIG',
  useValue: { siteName: 'My Shop', currency: 'VND' },
};

@Injectable()
export class ShopService {
  constructor(@Inject('APP_CONFIG') private config: any) {}
  getCurrency() { return this.config.currency; } // 'VND'
}

// 2. Mock khi test — thay CatsService thật bằng object giả
const mockCatsService = { findAll: () => [{ name: 'Mèo giả', age: 1 }] };

const moduleRef = await Test.createTestingModule({
  providers: [
    { provide: CatsService, useValue: mockCatsService }, // 👈 đồ giả
  ],
}).compile();
```

> ⚠️ Token là **string/symbol** thì nơi inject phải dùng `@Inject('TOKEN')`. Token là class thì không cần.

### 5.3. `useFactory` — Tạo bằng một hàm (linh hoạt nhất)

**Khái niệm:** Bảo NestJS *"Hãy gọi hàm này, lấy kết quả nó trả về làm provider."* Vì là hàm nên được viết **logic tùy ý**, kể cả **async** và **phụ thuộc provider khác**.

**Cách dùng:**

```typescript
providers: [
  {
    provide: 'DB_CONNECTION',
    useFactory: async (config: ConfigService) => {
      return await createConnection(config.get('DB_URL'));
    },
    inject: [ConfigService], // 👈 dependency factory cần, Nest tự đưa vào hàm
  },
]
```

> Mảng `inject` rất quan trọng: liệt kê các provider Nest phải phân giải **trước**, rồi truyền vào tham số hàm `useFactory` theo đúng thứ tự.

**Khi nào dùng:** việc tạo cần logic/điều kiện; cần async (kết nối DB, lấy token...); giá trị phụ thuộc provider khác.

**Tác dụng:** linh hoạt nhất trong 4 cách; hỗ trợ khởi tạo bất đồng bộ (Nest **chờ** Promise resolve xong mới bootstrap tiếp).

**Ví dụ — kết nối DB async phụ thuộc config:**

```typescript
@Module({
  providers: [
    ConfigService,
    {
      provide: 'DB_CONNECTION',
      useFactory: async (config: ConfigService) => {
        const url = config.get('DB_URL');
        console.log('Đang kết nối tới', url, '...');
        return await createConnection(url); // async
      },
      inject: [ConfigService], // factory nhận ConfigService
    },
  ],
  exports: ['DB_CONNECTION'],
})
export class DatabaseModule {}

@Injectable()
export class UserRepository {
  constructor(@Inject('DB_CONNECTION') private db: Connection) {}
}
```

**Nếu hàm factory có tham số thì sao? → dùng mảng `inject`:**

Các tham số của hàm factory được NestJS "bơm" vào thông qua mảng `inject`. Bạn liệt kê provider nào trong `inject`, Nest phân giải chúng và truyền vào hàm **theo đúng thứ tự** (tên tham số không quan trọng, **vị trí** mới quan trọng):

```
inject: [ ConfigService , LoggerService ]
             │                 │
             ▼                 ▼
useFactory: ( config    ,     logger    ) => { ... }
```

```typescript
{
  provide: 'DB_CONNECTION',
  useFactory: (config: ConfigService, logger: LoggerService) => { // 👈 2 tham số
    logger.log('Đang kết nối...');
    return createConnection(config.get('DB_URL'));
  },
  inject: [ConfigService, LoggerService], // 👈 đúng thứ tự tham số
}
```

- ⚠️ Nếu hàm có tham số mà **quên khai báo trong `inject`** → tham số đó là `undefined`. Nest **không tự đoán** dependency của factory qua kiểu (khác với constructor thông thường).
- Hàm **không có tham số** → bỏ luôn `inject`:
  ```typescript
  { provide: 'RANDOM_ID', useFactory: () => Math.random().toString(36) }
  ```
- Dependency có token **string/symbol** cũng đặt thẳng vào `inject`:
  ```typescript
  { provide: 'CONNECTION', useFactory: (opts) => new Conn(opts), inject: ['CONFIG_OPTIONS'] }
  ```
- Dependency **optional** (có thể thiếu) → bọc `{ token, optional: true }`:
  ```typescript
  inject: [
    ConfigService,                             // bắt buộc
    { token: 'HTTP_OPTIONS', optional: true }, // 👈 thiếu = undefined
  ]
  ```

| Câu hỏi | Trả lời |
|---------|---------|
| Tham số của factory lấy từ đâu? | Từ mảng **`inject`** |
| Mapping thế nào? | Theo **thứ tự** phần tử trong `inject` ↔ vị trí tham số |
| Quên khai báo trong `inject`? | Tham số → `undefined` |
| Hàm không tham số? | Bỏ `inject` |
| Dependency có thể thiếu? | `{ token: 'X', optional: true }` |

### 5.4. `useExisting` — Tạo alias (bí danh)

**Khái niệm:** Bảo NestJS *"Token này chỉ là **tên gọi khác** của một provider đã tồn tại — cùng trỏ về **một instance**."*

**Cách dùng:**

```typescript
providers: [
  LoggerService,
  { provide: 'AliasedLogger', useExisting: LoggerService }, // 👈 trỏ về provider có sẵn
]
```

**Khi nào dùng:** muốn 2 token cùng trỏ về 1 instance; đổi/thêm tên token mới cho provider cũ mà không phá vỡ code đang dùng; cấp một "mặt tiền" công khai cho implementation nội bộ.

**Tác dụng:** **không tạo instance mới** (khác hẳn `useClass`) — chỉ là con trỏ tới instance đã có.

**Ví dụ — cùng một logger, hai cái tên:**

```typescript
@Injectable()
export class LoggerService {
  log(msg: string) { console.log(msg); }
}

@Module({
  providers: [
    LoggerService,
    { provide: 'AliasedLogger', useExisting: LoggerService },
  ],
})
export class AppModule {}

@Injectable()
export class ServiceA {
  constructor(private logger: LoggerService) {}
}

@Injectable()
export class ServiceB {
  constructor(@Inject('AliasedLogger') private logger: LoggerService) {}
}
// ServiceA.logger === ServiceB.logger  ✅ (cùng 1 object)
```

> 🔑 **Phân biệt với `useClass`:** `useClass` tạo **instance mới**; `useExisting` **tái sử dụng** instance đã có (alias).

### 5.5. Quy tắc token & `@Inject()`

```typescript
// Token là CLASS / abstract class → inject trực tiếp, KHÔNG cần @Inject
constructor(private config: ConfigService) {}

// Token là STRING / SYMBOL → BẮT BUỘC dùng @Inject('TOKEN')
constructor(@Inject('APP_CONFIG') private config: any) {}
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
