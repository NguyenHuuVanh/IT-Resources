# NestJS - Providers

> Tổng hợp kiến thức về **Providers** trong NestJS: khái niệm, cơ chế Dependency Injection, ví dụ minh họa dễ hiểu và bộ câu hỏi phỏng vấn thường gặp.

## 1. Providers là gì?

**Provider** là các class được đánh dấu bằng decorator `@Injectable()`, có thể được **tiêm (inject)** vào các class khác thông qua cơ chế **Dependency Injection (DI)** của NestJS.

Provider thường dùng để chứa **logic nghiệp vụ (business logic)**, tách biệt khỏi tầng Controller. Hầu hết provider là **Service**, nhưng cũng có thể là:

- **Service** — chứa business logic (phổ biến nhất)
- **Repository** — truy cập dữ liệu
- **Factory** — tạo object động
- **Helper / Utility** — hàm tiện ích dùng chung
- **Value** — giá trị cấu hình, hằng số

> Ý tưởng cốt lõi: bạn **không tự `new` object**, NestJS tự tạo và "đưa" object vào nơi cần dùng.

## 2. Dependency Injection & IoC

NestJS dùng **IoC Container** (Inversion of Control) để quản lý vòng đời của các provider:

- **Inversion of Control (IoC):** thay vì class tự tạo dependency của nó, việc tạo và cấp phát được "đảo ngược" cho framework quản lý.
- **Dependency Injection (DI):** cơ chế cụ thể để IoC Container "tiêm" các dependency vào class khi khởi tạo.

```
Bạn khai báo dependency  ->  NestJS Container tạo instance  ->  Tiêm vào constructor
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

> Lưu ý: bạn **không viết** `new PhoService()`. Chỉ cần khai báo trong `constructor`, NestJS lo phần còn lại.

### Bước 3: Đăng ký trong Module

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

## 5. Các loại Provider (Custom Providers)

Có 4 cách định nghĩa provider:

| Cách | Mô tả | Dùng khi |
|------|-------|----------|
| `useClass` | Mặc định, cung cấp một class | Trường hợp thông thường |
| `useValue` | Inject một giá trị / object có sẵn | Config, hằng số, mock test |
| `useFactory` | Tạo instance động qua hàm | Cần khởi tạo theo điều kiện / async |
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
  inject: [ConfigService],
}

// useClass
{ provide: Logger, useClass: ProductionLogger }
```

Inject custom provider qua token bằng `@Inject()`:

```typescript
constructor(@Inject('API_KEY') private apiKey: string) {}
```

## 6. Scope của Provider

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

## 7. Lợi ích của Provider + DI

| Không dùng DI | Dùng Provider + DI |
|---------------|--------------------|
| `new PhoService()` ở mọi nơi | NestJS tạo **1 instance dùng chung** (singleton) |
| Khó test (phải tạo object thật) | Dễ test (thay bằng mock) |
| Code rối, lặp lại | Tách bạch rõ ràng, tái sử dụng |
| Coupling chặt | **Loose coupling** |

## 8. Câu hỏi phỏng vấn thường gặp

**Q: Provider trong NestJS là gì?**
> Là class được đánh dấu `@Injectable()`, có thể được inject vào class khác qua DI. Thường chứa business logic.

**Q: Phân biệt Controller và Provider?**
> Controller xử lý request/response (tầng giao tiếp HTTP). Provider chứa logic nghiệp vụ. Controller gọi Provider để xử lý.

**Q: `@Injectable()` để làm gì?**
> Đánh dấu class là provider để NestJS có thể quản lý và inject nó vào nơi khác.

**Q: Dependency Injection và IoC là gì?**
> IoC: việc tạo dependency được đảo ngược cho framework quản lý thay vì class tự tạo. DI: cơ chế cụ thể để container tiêm dependency vào class.

**Q: Có mấy cách định nghĩa provider?**
> 4 cách: `useClass` (mặc định), `useValue`, `useFactory`, `useExisting`.

**Q: Custom provider dùng khi nào?**
> Khi cần inject giá trị không phải class (config, connection string), hoặc cần tạo instance theo điều kiện / async.

**Q: Provider có những scope nào?**
> DEFAULT (singleton — mặc định), REQUEST (mỗi request một instance), TRANSIENT (mỗi lần inject một instance).

**Q: Làm sao một module dùng được provider của module khác?**
> Module sở hữu provider phải `exports` nó, module cần dùng phải `imports` module đó.

## 9. Tổng kết

- **Provider** là xương sống của NestJS, được đánh dấu bằng `@Injectable()`.
- Nhờ **Dependency Injection**, code trở nên **dễ test, dễ bảo trì và loose coupling**.
- Quy trình dùng: gắn `@Injectable()` → khai báo trong `constructor` → đăng ký trong `providers` của Module.
- Đây là nền tảng giúp NestJS được thiết kế theo hướng **modular và scalable**.
