# Các file đặc biệt trong Next.js App Router: layout.tsx, loading.tsx, error.tsx và not-found.tsx

> Câu hỏi: Trong Next.js App Router, các file `layout.tsx`, `loading.tsx`, `error.tsx` và `not-found.tsx` có vai trò gì? Khi nào chúng được sử dụng?

## Câu trả lời

Đây là các **file đặc biệt** trong Next.js App Router, được dùng để xử lý bố cục, trạng thái tải dữ liệu, lỗi và trường hợp không tìm thấy tài nguyên.

---

### 1. `layout.tsx`

`layout.tsx` được dùng để tạo **giao diện dùng chung** cho một route và các route con bên trong nó (header, sidebar, menu, footer...).

Layout nhận nội dung của route con thông qua prop `children`:

```tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <aside>Sidebar</aside>
      <main>{children}</main>
    </div>
  );
}
```

**Ví dụ cấu trúc:**

```text
app/
└── dashboard/
    ├── layout.tsx
    ├── page.tsx
    └── users/
        └── page.tsx
```

`dashboard/layout.tsx` sẽ bao quanh cả `/dashboard` và `/dashboard/users`.  
Layout có thể được tái sử dụng khi chuyển giữa các trang con → giúp tránh render lại những phần giao diện dùng chung.

**Root Layout** (bắt buộc):  
File `app/layout.tsx` phải chứa thẻ `<html>` và `<body>`:

```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="vi">
      <body>{children}</body>
    </html>
  );
}
```

---

### 2. `loading.tsx`

`loading.tsx` được dùng để hiển thị **giao diện chờ** trong lúc nội dung của route đang được tải hoặc Server Component đang xử lý dữ liệu.

```tsx
export default function Loading() {
  return <div>Đang tải dữ liệu...</div>;
}
```

Thường dùng để hiển thị spinner hoặc skeleton:

```text
app/
└── products/
    ├── loading.tsx
    └── page.tsx
```

Khi người dùng truy cập `/products`, Next.js có thể hiển thị `loading.tsx` ngay trong lúc chờ `page.tsx` hoàn thành.  
Next.js sử dụng **React Suspense** và **streaming** để thay thế giao diện loading bằng nội dung thật khi dữ liệu đã sẵn sàng.

---

### 3. `error.tsx`

`error.tsx` được dùng để xử lý những **lỗi runtime không mong muốn** xảy ra trong một route segment hoặc các route con của nó.  
File này hoạt động như một **React Error Boundary** và hiển thị giao diện thay thế thay vì làm hỏng toàn bộ ứng dụng.

**Lưu ý quan trọng:** `error.tsx` **phải là Client Component**.

```tsx
"use client";

export default function ErrorPage({
  error,
  reset, // hoặc unstable_retry tùy phiên bản
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Đã xảy ra lỗi.</h2>
      <button onClick={() => reset()}>
        Thử lại
      </button>
    </div>
  );
}
```

- `error` chứa thông tin lỗi.
- `reset()` (hoặc `unstable_retry()`) cho phép thử tải và render lại phần nội dung bị lỗi.

Lỗi sẽ được chuyển lên error boundary gần nhất.  
Có thể đặt nhiều file `error.tsx` tại các cấp route khác nhau.

**Lưu ý:**
- Lỗi xảy ra ngay trong `layout.tsx` của cùng segment **không** được `error.tsx` đó bắt.
- Lỗi ở Root Layout có thể được xử lý bằng `global-error.tsx`.

---

### 4. `not-found.tsx`

`not-found.tsx` dùng để hiển thị giao diện **404** khi:
- Tài nguyên không tồn tại
- Hoặc khi chương trình gọi hàm `notFound()`

```tsx
export default function NotFound() {
  return (
    <div>
      <h2>Không tìm thấy sản phẩm</h2>
      <p>Sản phẩm bạn yêu cầu không tồn tại.</p>
    </div>
  );
}
```

**Ví dụ sử dụng trong trang chi tiết sản phẩm:**

```tsx
import { notFound } from "next/navigation";

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const product = await getProduct(id);

  if (!product) {
    notFound(); // → hiển thị not-found.tsx gần nhất
  }

  return <h1>{product.name}</h1>;
}
```

Khi `notFound()` được gọi, Next.js dừng render route segment hiện tại và hiển thị file `not-found.tsx` gần nhất.  
File `app/not-found.tsx` cũng có thể xử lý những URL không khớp với bất kỳ route nào trong ứng dụng.

---

## Cách trả lời ngắn khi phỏng vấn

- **`layout.tsx`**: Tạo giao diện dùng chung cho route và các route con.
- **`loading.tsx`**: Hiển thị trạng thái chờ khi route đang tải dữ liệu.
- **`error.tsx`**: Bắt lỗi runtime không mong muốn và hiển thị fallback UI (phải là Client Component).
- **`not-found.tsx`**: Hiển thị giao diện 404 khi URL hoặc tài nguyên không tồn tại.

**Điểm khác biệt quan trọng:**  
`error.tsx` xử lý **lỗi bất thường** của chương trình, còn `not-found.tsx` xử lý trường hợp **hợp lệ nhưng không tìm thấy tài nguyên**.

---

*Tài liệu tham khảo nhanh – Next.js App Router*
