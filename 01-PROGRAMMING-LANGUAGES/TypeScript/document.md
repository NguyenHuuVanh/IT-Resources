# Interface

**Interface** trong TypeScript là cấu trúc định nghĩa hình dạng của một đối tượng (thuộc tính, phương thức). Interface chỉ tồn tại trong quá trình biên dịch TS để kiểm tra kiểu, sẽ bị loại bỏ khi chuyển sang JS.

## Chức năng chính

- **Định nghĩa cấu trúc dữ liệu:**  
  Mô tả cấu trúc mong muốn của object, class, biến.
- **Kiểm tra kiểu (Type Checking):**  
  Đảm bảo đối tượng tuân thủ đúng cấu trúc.
- **Tăng cường tính nhất quán:**  
  Giúp mã nguồn dễ đọc, nhất quán.
- **Hỗ trợ tái cấu trúc (Refactoring):**  
  Tách biệt logic triển khai khỏi cấu trúc.

---

## Các tính năng của Interface

### Tùy chọn, readonly, optional vs undefined

```typescript
interface User {
  email?: string; // Có thể vắng mặt
  phone: string | undefined; // Luôn có property, có thể undefined
}
```

### Index signatures (key động)

```typescript
interface StringMap {
  [key: string]: string;
}

interface Dict<T> {
  [id: string]: T;
}
```

### Kiểu hàm & constructor

```typescript
interface Add {
  (a: number, b: number): number;
}
const add: Add = (x, y) => x + y;

interface Ctor<T> {
  new (...args: any[]): T;
}
```

### Hybrid types (vừa callable, vừa có thuộc tính)

```typescript
interface Counter {
  (start: number): void;
  value: number;
  reset(): void;
}

function makeCounter(): Counter {
  const fn = ((start: number) => { fn.value = start; }) as Counter;
  fn.value = 0;
  fn.reset = () => { fn.value = 0; };
  return fn;
}
```

### Kế thừa (extends) & tổng hợp

```typescript
interface Person { name: string }
interface Employee extends Person { salary: number }

type Address = { city: string; country: string };
interface Staff extends Address { id: number }
```

### Generic interface

```typescript
interface ApiResponse<TData, TMeta = { page: number }> {
  data: TData;
  meta: TMeta;
}
type UserResp = ApiResponse<User>;
```

### Class implements interface

```typescript
interface Repository<T> {
  get(id: string): Promise<T>;
  save(entity: T): Promise<void>;
}

class UserRepo implements Repository<User> {
  async get(id: string) { /* ... */ return { id: 1, name: "A", createdAt: new Date() }; }
  async save(entity: User) { /* ... */ }
}
```

---

# Type

**Type** dùng để tạo type alias (tên gọi mới cho kiểu dữ liệu bất kỳ: nguyên thủy, object, union, tuple...). Giúp code rõ ràng, dễ đọc, tái sử dụng và kiểm tra kiểu tại thời điểm biên dịch.

## Alias kiểu cơ bản & hàm

```typescript
type ID = string | number;
type Add = (a: number, b: number) => number;
const add: Add = (x, y) => x + y;
```

## Union & Intersection

```typescript
type Result = { ok: true; data: unknown } | { ok: false; error: string };
type A = { x: number };
type B = { y: number };
type AB = A & B; // { x: number; y: number }
```

## Tuple

```typescript
type Pair = [number, number];
type Labeled = [id: string, active?: boolean];
```

## Generic type alias

```typescript
type Box<T> = { value: T };
const a: Box<number> = { value: 1 };
```

## Mapped types

```typescript
type ReadonlyAll<T> = { readonly [K in keyof T]: T[K] };
type PartialAll<T>  = { [K in keyof T]?: T[K] };

type User = { id: string; name: string };
type UserRO = ReadonlyAll<User>;
```

## Conditional types

```typescript
type Awaited<T> = T extends Promise<infer U> ? U : T;
type IsString<T> = T extends string ? true : false;

type A1 = Awaited<Promise<number>>; // number
type A2 = IsString<"hi">;           // true
```

## Template literal types

```typescript
type EventName = `on${Capitalize<string>}`;
type Route = `/api/${string}`;
```

## keyof, typeof, satisfies

```typescript
type Keys<T> = keyof T;

const cfg = { baseUrl: "/api", timeout: 1000 } as const;
type Cfg = typeof cfg;           // { readonly baseUrl: "/api"; readonly timeout: 1000 }
type CfgKeys = keyof Cfg;        // "baseUrl" | "timeout"

const user = {
  id: "u1",
  name: "A",
} satisfies { id: string; name: string };
// `satisfies` kiểm tính hợp lệ mà vẫn giữ literal types tốt hơn as-cast.
```

## Utility types (có sẵn trong TS)

```typescript
type U = { id: string; name: string; age?: number };

type Picked   = Pick<U, "id" | "name">;
type Omitted  = Omit<U, "age">;
type PartialU = Partial<U>;
type RequiredU= Required<U>;
type ReadonlyU= Readonly<U>;
type NonNull  = NonNullable<string | null | undefined>;
type RecordU  = Record<string, number>;
type ReturnT  = ReturnType<() => Promise<void>>;
```

---

## Khi nào nên dùng type?

- Cần union/intersection/tuple hoặc kiểu biến đổi → **type**
- Cần conditional/mapped/template-literal types → **type**
- Định nghĩa hàm hay utility generic → **type** tiện hơn
- Mở rộng API của lib qua merging → ưu tiên **interface**

---

# Ví dụ gắn với app email tool

```typescript
// Kiểu bản ghi Gmail
export type Contact = {
  id: string;
  email: string;
  name: string;
};

// Map động: email -> Contact
export type ContactMap = Record<string, Contact>;

// Kết quả API tổng quát
export type ApiResult<T> =
  | { ok: true;  data: T }
  | { ok: false; error: string };

// Hàm gửi mail (callable type)
export type SendMail = (args: {
  to: string;
  subject: string;
  html: string;
}) => Promise<ApiResult<void>>;

// Biến đổi tất cả field thành optional (để partial update)
export type Update<T> = Partial<T>;
```