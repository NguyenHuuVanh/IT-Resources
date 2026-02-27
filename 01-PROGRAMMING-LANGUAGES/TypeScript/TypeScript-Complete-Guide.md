# TypeScript - Hướng Dẫn Toàn Diện

## Mục Lục

1. [Giới Thiệu](#giới-thiệu)
2. [Basic Types](#basic-types)
3. [Advanced Types](#advanced-types)
4. [Interface vs Type](#interface-vs-type)
5. [Generics](#generics)
6. [Utility Types](#utility-types)
7. [Type Guards](#type-guards)
8. [Best Practices](#best-practices)

---

## Giới Thiệu

### TypeScript là gì?

**TypeScript** là superset của JavaScript, thêm static typing (kiểu tĩnh) vào JavaScript.

```
JavaScript (Dynamic Typing)
  ↓ Add Types
TypeScript (Static Typing)
  ↓ Compile (tsc)
JavaScript (Runtime)
```

### Tại Sao Dùng TypeScript?

✅ **Phát hiện lỗi sớm**: Lỗi được catch tại compile time thay vì runtime
✅ **IDE Support tốt**: Autocomplete, refactoring, go to definition
✅ **Self-documenting**: Code tự giải thích qua types
✅ **Refactoring an toàn**: Đổi tên, di chuyển code dễ dàng
✅ **Scalability**: Dễ maintain codebase lớn

### Cách Hoạt Động

```typescript
// TypeScript code
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Compile thành JavaScript
function greet(name) {
  return `Hello, ${name}`;
}

// Types bị xóa, chỉ còn logic
```

**Lưu ý**: Types chỉ tồn tại ở compile time, không có ở runtime.

---

## Basic Types

### 1. String

**Khái niệm**: Kiểu dữ liệu văn bản, chuỗi ký tự.

```typescript
// Khai báo
let name: string = "John";
let greeting: string = "Hello";
let template: string = `Hello ${name}`; // Template literal

// Type inference (tự suy luận)
let city = "Hanoi"; // TypeScript tự biết city là string

// String methods vẫn hoạt động
name.toUpperCase();
name.length;
```

**Khi nào dùng**: Text, messages, IDs dạng string, URLs, emails.

### 2. Number

**Khái niệm**: Kiểu số, bao gồm integer và float. TypeScript không phân biệt int/float như các ngôn ngữ khác.

```typescript
// Các dạng number
let age: number = 25; // Integer
let price: number = 99.99; // Float
let hex: number = 0xf00d; // Hexadecimal
let binary: number = 0b1010; // Binary
let octal: number = 0o744; // Octal

// NaN và Infinity cũng là number
let notANumber: number = NaN;
let infinite: number = Infinity;

// Math operations
let sum = age + 5;
let product = price * 2;
```

**Khi nào dùng**: Số lượng, giá tiền, tuổi, điểm số, measurements.

### 3. Boolean

**Khái niệm**: Kiểu logic, chỉ có 2 giá trị: `true` hoặc `false`.

```typescript
let isActive: boolean = true;
let hasPermission: boolean = false;

// Từ expressions
let isAdult = age >= 18; // boolean
let isEmpty = items.length === 0; // boolean

// Logical operations
let canAccess = isActive && hasPermission;
let shouldShow = isActive || isAdmin;
```

**Khi nào dùng**: Flags, conditions, states (on/off, yes/no, enabled/disabled).

### 4. Array

**Khái niệm**: Danh sách các phần tử cùng kiểu.

```typescript
// Cách 1: Type[]
let numbers: number[] = [1, 2, 3, 4, 5];
let names: string[] = ["John", "Jane", "Bob"];

// Cách 2: Array<Type>
let scores: Array<number> = [90, 85, 95];

// Array methods
numbers.push(6);
numbers.map((n) => n * 2);
numbers.filter((n) => n > 3);

// Mixed types (union)
let mixed: (string | number)[] = [1, "two", 3, "four"];

// Array of objects
let users: { name: string; age: number }[] = [
  { name: "John", age: 25 },
  { name: "Jane", age: 30 },
];

// Readonly array
let readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // Error!
```

**Khi nào dùng**: Danh sách items, collections, lists.

### 5. Tuple

**Khái niệm**: Array với số lượng phần tử cố định và kiểu dữ liệu xác định cho từng vị trí.

```typescript
// Basic tuple
let person: [string, number] = ["John", 25];
//           [name,   age]

// Access
let name = person[0]; // string
let age = person[1]; // number

// Named tuple (TypeScript 4.0+)
type Point = [x: number, y: number];
let point: Point = [10, 20];

// Optional elements
type Response = [status: number, message: string, data?: any];
let res1: Response = [200, "OK"];
let res2: Response = [200, "OK", { id: 1 }];

// Rest elements
type StringNumberBooleans = [string, number, ...boolean[]];
let t: StringNumberBooleans = ["hello", 1, true, false, true];

// Destructuring
let [firstName, userAge] = person;
```

**Khi nào dùng**:

- Coordinates: `[x, y]` hoặc `[lat, lng]`
- Key-value pairs: `[key, value]`
- Function returns: `[data, error]` hoặc `[result, status]`
- RGB colors: `[255, 0, 0]`

**Tuple vs Array**:

```typescript
// Array: Không biết chính xác số lượng và vị trí
let arr: number[] = [1, 2, 3, 4, 5]; // Có thể có bao nhiêu phần tử cũng được

// Tuple: Biết chính xác
let tuple: [number, number] = [1, 2]; // Chỉ 2 phần tử, vị trí cố định
```

### 6. Enum

**Khái niệm**: Tập hợp các hằng số có tên, giúp code dễ đọc hơn.

```typescript
// Numeric enum (mặc định bắt đầu từ 0)
enum Direction {
  Up, // 0
  Down, // 1
  Left, // 2
  Right, // 3
}

let dir: Direction = Direction.Up;
console.log(dir); // 0

// Custom numeric values
enum Status {
  Pending = 1,
  Approved = 2,
  Rejected = 3,
}

// String enum (recommended)
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE",
}

let color: Color = Color.Red;
console.log(color); // "RED"

// Const enum (không generate code, chỉ inline values)
const enum HttpStatus {
  OK = 200,
  NotFound = 404,
  ServerError = 500,
}

let status = HttpStatus.OK; // Compiled to: let status = 200;
```

**Khi nào dùng**:

- Status codes: `Pending`, `Approved`, `Rejected`
- Directions: `Up`, `Down`, `Left`, `Right`
- Roles: `Admin`, `User`, `Guest`
- HTTP methods: `GET`, `POST`, `PUT`, `DELETE`

**Enum vs Union Types**:

```typescript
// Enum
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
}
let status: Status = Status.Active;

// Union (thường tốt hơn)
type Status = "ACTIVE" | "INACTIVE";
let status: Status = "ACTIVE";
```

### 7. Any

**Khái niệm**: Tắt type checking, chấp nhận mọi kiểu dữ liệu. **Nên tránh dùng**.

```typescript
let anything: any = 4;
anything = "hello";
anything = true;
anything = { x: 1 };
anything.foo.bar.baz; // Không có lỗi!

// Khi nào dùng any (hiếm khi)
// 1. Migrate từ JavaScript
// 2. Third-party library không có types
// 3. Thực sự không biết type (nhưng nên dùng unknown)
```

**Vấn đề với any**: Mất hết type safety, dễ gây lỗi runtime.

### 8. Unknown

**Khái niệm**: Type-safe version của `any`. Phải check type trước khi dùng.

```typescript
let value: unknown = 4;
value = "hello";
value = true;

// Không thể dùng trực tiếp
// value.toUpperCase(); // Error!

// Phải check type trước
if (typeof value === "string") {
  value.toUpperCase(); // OK
}

// Type guard
function isString(value: unknown): value is string {
  return typeof value === "string";
}

if (isString(value)) {
  value.toUpperCase(); // OK
}
```

**Khi nào dùng**: API responses, user input, data từ external sources.

**Unknown vs Any**:

```typescript
// Any: Không an toàn
let a: any = "hello";
a.toUpperCase(); // OK (nhưng nếu a là number thì lỗi runtime)

// Unknown: An toàn
let u: unknown = "hello";
// u.toUpperCase(); // Error! Phải check trước
if (typeof u === "string") {
  u.toUpperCase(); // OK
}
```

### 9. Void

**Khái niệm**: Không có giá trị trả về. Dùng cho functions không return gì.

```typescript
// Function không return
function log(message: string): void {
  console.log(message);
  // Không có return statement
}

// Function return undefined cũng là void
function doSomething(): void {
  return undefined; // OK
}

// Variable với type void (hiếm khi dùng)
let unusable: void = undefined;
```

**Void vs Undefined**:

```typescript
// Void: Function không return
function log(): void {
  console.log("Hello");
}

// Undefined: Function return undefined
function getValue(): undefined {
  return undefined;
}
```

### 10. Null và Undefined

**Khái niệm**: Giá trị "không có gì".

```typescript
let n: null = null;
let u: undefined = undefined;

// Với strictNullChecks: true
let name: string = "John";
// name = null; // Error!
// name = undefined; // Error!

// Cho phép null/undefined
let name: string | null = "John";
name = null; // OK

let age: number | undefined = 25;
age = undefined; // OK

// Optional chaining
let user = { name: "John", address: { city: "Hanoi" } };
let city = user?.address?.city; // string | undefined

// Nullish coalescing
let displayName = name ?? "Anonymous"; // Nếu name là null/undefined thì dùng "Anonymous"
```

**Null vs Undefined**:

- `undefined`: Biến chưa được gán giá trị
- `null`: Biến được gán giá trị "không có gì" một cách có chủ đích

### 11. Never

**Khái niệm**: Type không bao giờ xảy ra. Dùng cho functions không bao giờ return.

```typescript
// Function throw error
function throwError(message: string): never {
  throw new Error(message);
  // Không bao giờ return
}

// Infinite loop
function infiniteLoop(): never {
  while (true) {
    // Loop forever
  }
}

// Exhaustive checking
type Shape = Circle | Square;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      const _exhaustive: never = shape; // Error nếu thiếu case
      return _exhaustive;
  }
}
```

**Never vs Void**:

```typescript
// Void: Function kết thúc nhưng không return giá trị
function log(): void {
  console.log("Done");
} // Function ends

// Never: Function không bao giờ kết thúc
function fail(): never {
  throw new Error("Failed");
} // Function never ends
```

### 12. Object

**Khái niệm**: Kiểu dữ liệu phức tạp, chứa properties và methods.

```typescript
// Object type
let user: { name: string; age: number } = {
  name: "John",
  age: 25,
};

// Optional properties
let config: {
  host: string;
  port?: number; // Optional
} = {
  host: "localhost",
};

// Readonly properties
let point: {
  readonly x: number;
  readonly y: number;
} = { x: 10, y: 20 };
// point.x = 5; // Error!

// Index signatures (dynamic keys)
let scores: {
  [studentName: string]: number;
} = {
  John: 90,
  Jane: 95,
};

// Object vs object vs {}
let obj1: object = { x: 1 }; // Bất kỳ object nào (không phải primitive)
let obj2: Object = { x: 1 }; // Tương tự object (ít dùng)
let obj3: {} = { x: 1 }; // Object rỗng (ít dùng)
```
