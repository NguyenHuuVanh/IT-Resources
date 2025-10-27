Interface

Interface trong ts là một cấu trúc định nghĩa hình dạng của một đối tượng, bao gồm các thuộc tính, phương thức mà đối tượng đó cần. Nó hoạt động như một hợp đồng để kiểm tra tính tương thích kiểu, dữ liệu theo cấu trúc, giúp có thể phát hiện lỗi sớm trong quá trình phát triển mà không cần triển khai bất kì chức năng nào. Interface chỉ tồn tại trong quá trình biên dịch TS để đảm bảo tính đúng đắn của mã nguồn, và sẽ bị xoá bỏ khi mã được biên dịch sang JS

Chức năng chính của Interface trong TypeScript:

• Định nghĩa cấu trúc dữ liệu: 

Interface cho phép bạn mô tả cấu trúc mong muốn của các đối tượng, class, hoặc biến một cách rõ ràng và có tổ chức. 

• Kiểm tra kiểu (Type Checking): 

TypeScript sử dụng Interface để xác minh rằng các đối tượng được tạo ra tuân thủ đúng cấu trúc đã định nghĩa. Nếu không tuân thủ, trình biên dịch sẽ báo lỗi ngay lập tức. 

• Tăng cường tính nhất quán: 

Bằng cách đặt ra một quy chuẩn cho cấu trúc của các đối tượng, Interface giúp duy trì sự nhất quán và dễ đọc trong mã nguồn, đặc biệt là trong các dự án lớn. 

• Hỗ trợ tái cấu trúc (Refactoring): 

Giao diện giúp tách biệt logic triển khai khỏi cấu trúc, cho phép bạn thay đổi cách triển khai bên trong mà không ảnh hưởng đến phần sử dụng. 

Tùy chọn, readonly, và “optional vs undefined”

• email?: string nghĩa là property có thể vắng mặt. Khi có mặt, phải là string.

• Nếu muốn “luôn có property nhưng có thể là undefined”, dùng email: string | undefined.

Index signatures (định nghĩa key động)

interface StringMap {

  \[key: string]: string; // mọi key chuỗi phải có giá trị string

}

interface Dict\<T> {

  \[id: string]: T;}

Kiểu hàm (callable) & constructor

interface Add {

  (a: number, b: number): number;

}

const add: Add = (x, y) => x + y;

interface Ctor\<T> {

  new (...args: any\[]): T; // construct signature

}

“Hybrid types” (vừa callable, vừa có thuộc tính)

interface Counter {

  (start: number): void; // callable

  value: number;         // thuộc tính

  reset(): void;         // phương thức

}

function makeCounter(): Counter {

  const fn = ((start: number) => { fn.value = start; }) as Counter;

  fn.value = 0;

  fn.reset = () => { fn.value = 0; };

  return fn;

}

Kế thừa (extends) & tổng hợp

• Interface có thể extends nhiều interface.

• Interface có thể extends một type alias nếu type đó là object type.

interface Person { name: string }

interface Employee extends Person { salary: number }

type Address = { city: string; country: string };

interface Staff extends Address { id: number }  // OK vì Address là object type

Generic interface

interface ApiResponse\<TData, TMeta = { page: number }> {

  data: TData;

  meta: TMeta;

}

type UserResp = ApiResponse\<User>;

Class implements interface

interface Repository\<T> {

  get(id: string): Promise\<T>;

  save(entity: T): Promise\<void>;

}

class UserRepo implements Repository\<User> {

  async get(id: string) { /\* ... \*/ return { id: 1, name: "A", createdAt: new Date() }; }

  async save(entity: User) { /\* ... \*/ }

}

Type

Trong TS, từ khoá type dùng để tạo type alias, tức là một tên gọi mới cho bất kì kiểu dữ liệu nào, bao gồm các kiểu nguyên thuỷ (như string, number,…), các đối tượng phức tạp, union type (kết hợp nhiều kiểu), hoặc tuple. Nó giúp định nghĩa các kiểu dữ liệu tuỳ chỉnh để làm cho code rõ ràng, dễ đọc, tái sử đụng được và tránh lỗi bằng cách kiểm tra kiểu tại thời điểm biên dịch.

Alias kiểu cơ bản & hàm

type ID = string | number;

type Add = (a: number, b: number) => number;

const add: Add = (x, y) => x + y;

Union & Intersection

type Result = { ok: true; data: unknown } | { ok: false; error: string };

type A = { x: number };

type B = { y: number };

type AB = A & B; // { x: number; y: number }

Tuple

type Pair = \[number, number];

type Labeled = \[id: string, active?: boolean];

Generic type alias

type Box\<T> = { value: T };

const a: Box\<number> = { value: 1 };

Mapped types (biến đổi thuộc tính)

type ReadonlyAll\<T> = { readonly \[K in keyof T]: T\[K] };

type PartialAll\<T>  = { \[K in keyof T]?: T\[K] };

type User = { id: string; name: string };

type UserRO = ReadonlyAll\<User>;

Conditional types

type Awaited\<T> = T extends Promise\<infer U> ? U : T;

type IsString\<T> = T extends string ? true : false;

type A1 = Awaited\<Promise\<number>>; // number

type A2 = IsString<"hi">;            // true

Template literal types

type EventName = \`on${Capitalize\<string>}\`;

type Route = \`/api/${string}\`;

keyof, typeof, satisfies

type Keys\<T> = keyof T;

const cfg = { baseUrl: "/api", timeout: 1000 } as const;

type Cfg = typeof cfg;           // { readonly baseUrl: "/api"; readonly timeout: 1000 }

type CfgKeys = keyof Cfg;        // "baseUrl" | "timeout"

const user = {

  id: "u1",

  name: "A",

} satisfies { id: string; name: string };

// \`satisfies\` kiểm tính hợp lệ mà vẫn giữ literal types tốt hơn as-cast.

Utility types (có sẵn trong TS – đa số dựa trên type)

type U = { id: string; name: string; age?: number };

type Picked   = Pick\<U, "id" | "name">;

type Omitted  = Omit\<U, "age">;

type PartialU = Partial\<U>;

type RequiredU= Required\<U>;

type ReadonlyU= Readonly\<U>;

type NonNull  = NonNullable\<string | null | undefined>;

type RecordU  = Record\<string, number>;

type ReturnT  = ReturnType<() => Promise\<void>>;

Khi nào nên dùng type?

• Cần union/intersection/tuple hoặc kiểu biến đổi → type.

• Cần conditional/mapped/template-literal types → type.

• Định nghĩa hàm hay utility generic → type tiện hơn.

• Mở rộng API của lib qua merging → ưu tiên interface.

Ví dụ nhỏ gắn với app của bạn (email tool)

// Kiểu bản ghi Gmail

export type Contact = {

  id: string;

  email: string;

  name: string;

};

// Map động: email -> Contact

export type ContactMap = Record\<string, Contact>;

// Kết quả API tổng quát

export type ApiResult\<T> =

  | { ok: true;  data: T }

  | { ok: false; error: string };

// Hàm gửi mail (callable type)

export type SendMail = (args: {

  to: string;

  subject: string;

  html: string;

}) => Promise\<ApiResult\<void>>;

// Biến đổi tất cả field thành optional (để partial update)

export type Update\<T> = Partial\<T>;
