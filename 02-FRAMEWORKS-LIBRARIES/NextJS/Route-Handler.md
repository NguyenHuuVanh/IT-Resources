# Route Handler trong Next.js App Router

> Câu hỏi: Trong Next.js App Router, Route Handler là gì? Hãy giải thích cách tạo một API endpoint bằng file `route.ts` và nêu sự khác nhau cơ bản giữa GET và POST.

## Câu trả lời

**Route Handler** là tính năng của Next.js App Router cho phép tạo các **API endpoint** ngay trong thư mục `app`.  

Route Handler sử dụng các **Web API chuẩn** như `Request` và `Response`. Ngoài ra, Next.js còn cung cấp `NextRequest` và `NextResponse` để hỗ trợ thêm các tiện ích như xử lý cookie, URL và redirect.

---

### Cách tạo API Endpoint

Để tạo một API endpoint, tạo file `route.ts` bên trong một thư mục thuộc `app`.

**Ví dụ cấu trúc:**

```text
app/
└── api/
    └── products/
        └── route.ts
```

Endpoint tương ứng sẽ là:

```text
/api/products
```

---

### 1. Xử lý phương thức GET

`GET` thường được dùng để **lấy dữ liệu** và **không nên** làm thay đổi trạng thái của hệ thống.

```ts
// app/api/products/route.ts

const products = [
  { id: 1, name: "Laptop" },
  { id: 2, name: "Keyboard" },
];

export async function GET() {
  return Response.json(
    {
      data: products,
    },
    {
      status: 200,
    },
  );
}
```

Khi client gửi request:

```http
GET /api/products
```

API sẽ trả về danh sách sản phẩm dưới dạng JSON.

**Lưu ý về cache:**  
Trong các phiên bản Next.js hiện tại, Route Handler không được cache mặc định. Một số GET handler có thể được cấu hình để cache khi phù hợp. Các phương thức làm thay đổi dữ liệu như `POST` không được cache theo cơ chế này.

---

### 2. Xử lý phương thức POST

`POST` thường được dùng để **gửi dữ liệu** lên server và **tạo một tài nguyên mới**.

```ts
// app/api/products/route.ts

type CreateProductBody = {
  name?: string;
  price?: number;
};

export async function POST(request: Request) {
  const body = (await request.json()) as CreateProductBody;

  if (!body.name || typeof body.price !== "number") {
    return Response.json(
      {
        message: "Dữ liệu không hợp lệ",
      },
      {
        status: 400,
      },
    );
  }

  const newProduct = {
    id: Date.now(),
    name: body.name,
    price: body.price,
  };

  // Trong dự án thực tế, dữ liệu sẽ được lưu vào database.

  return Response.json(
    {
      message: "Tạo sản phẩm thành công",
      data: newProduct,
    },
    {
      status: 201,
    },
  );
}
```

**Cách client gọi API:**

```ts
const response = await fetch("/api/products", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Mouse",
    price: 500000,
  }),
});

if (!response.ok) {
  throw new Error("Không thể tạo sản phẩm");
}

const result = await response.json();
```

Trong `POST`, dùng `request.json()` để đọc dữ liệu JSON được gửi trong request body.

---

### Sự khác nhau cơ bản giữa GET và POST

| Tiêu chí                    | GET                                      | POST                                      |
|----------------------------|------------------------------------------|-------------------------------------------|
| Mục đích chính             | Lấy dữ liệu                              | Gửi dữ liệu / Tạo tài nguyên mới          |
| Có làm thay đổi dữ liệu?   | Không (nên không)                        | Có thể                                    |
| Vị trí dữ liệu             | Thường nằm trong URL (query parameter)   | Nằm trong request body                    |
| Có thể cache không?        | Có thể được cấu hình cache               | Không được cache                          |
| Ví dụ sử dụng              | Lấy danh sách sản phẩm, chi tiết bài viết| Tạo sản phẩm, đăng ký tài khoản, gửi form |

Next.js hỗ trợ các phương thức: **GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS**.  
Nếu client gọi một phương thức không được hỗ trợ → Next.js trả về trạng thái **405 Method Not Allowed**.

---

### Lưu ý quan trọng

**Không thể** đặt `page.tsx` và `route.ts` trong cùng một route segment vì cả hai sẽ cùng xử lý một URL và gây xung đột.

**Ví dụ không hợp lệ:**

```text
app/
└── products/
    ├── page.tsx
    └── route.ts
```

**Cách tổ chức hợp lệ:**

```text
app/
├── products/
│   └── page.tsx
└── api/
    └── products/
        └── route.ts
```

Route Handler có thể được đặt ở nhiều vị trí trong cây thư mục `app`, nhưng `route.ts` **không được** nằm cùng cấp với `page.tsx`.

---

## Cách trả lời ngắn khi phỏng vấn

Route Handler là cách tạo API endpoint trong Next.js App Router bằng file `route.ts`.  
Bên trong file, em export các hàm mang tên phương thức HTTP như `GET`, `POST`, `PUT` hoặc `DELETE`.  

- `GET` thường dùng để lấy dữ liệu.  
- `POST` dùng để nhận dữ liệu từ request body và tạo tài nguyên mới.  

Handler có thể trả về JSON bằng `Response.json()` hoặc `NextResponse.json()`.

---

*Tài liệu tham khảo nhanh – Next.js App Router*
