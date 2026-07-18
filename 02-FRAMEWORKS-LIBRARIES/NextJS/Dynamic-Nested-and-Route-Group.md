# Dynamic Route, Nested Route và Route Group trong Next.js App Router

> Câu hỏi: Trong Next.js App Router, dynamic route, nested route và route group là gì? Hãy giải thích ngắn gọn và cho ví dụ cấu trúc thư mục cho từng loại.

## Câu trả lời

Trong Next.js **App Router**, routing được xây dựng dựa trên cấu trúc thư mục bên trong thư mục `app`.  
Mỗi thư mục thường đại diện cho một phần của URL và file `page.tsx` tạo giao diện cho đường dẫn đó.

---

### 1. Dynamic Route

**Dynamic route** là đường dẫn có một phần giá trị **không được xác định trước**, ví dụ: ID sản phẩm, tên người dùng hoặc slug bài viết.

Để tạo dynamic route, đặt tên thư mục trong **dấu ngoặc vuông**, ví dụ `[id]` hoặc `[slug]`.

```text
app/
└── products/
    └── [id]/
        └── page.tsx
```

Cấu trúc trên có thể xử lý các URL như:

```text
/products/1
/products/25
/products/iphone-15
```

Giá trị nằm sau `/products/` sẽ được lấy thông qua tham số `id` (hoặc `params.id`).

**Ngoài ra, Next.js còn hỗ trợ:**

```text
[...slug]      // Catch-all segment
[[...slug]]    // Optional catch-all segment
```

Ví dụ: `app/docs/[...slug]/page.tsx` có thể xử lý các đường dẫn như:
- `/docs/react`
- `/docs/react/hooks/use-state`

---

### 2. Nested Route

**Nested route** là route được tạo bằng cách **lồng các thư mục** bên trong nhau.  
Mỗi thư mục tương ứng với một segment trong URL.

**Ví dụ cấu trúc:**

```text
app/
└── dashboard/
    ├── page.tsx
    └── users/
        ├── page.tsx
        └── [id]/
            └── page.tsx
```

**Các URL tương ứng:**

```text
/dashboard
/dashboard/users
/dashboard/users/123
```

Nested route giúp thể hiện rõ cấu trúc phân cấp của ứng dụng.  
Các `layout` cũng có thể được lồng nhau, nghĩa là layout của route cha có thể bao bọc giao diện của các route con.

---

### 3. Route Group

**Route group** được sử dụng để **nhóm và tổ chức** các route mà **không làm thay đổi URL**.

Để tạo route group, đặt tên thư mục trong **dấu ngoặc tròn**, ví dụ `(marketing)` hoặc `(dashboard)`.

```text
app/
├── (marketing)/
│   ├── about/
│   │   └── page.tsx
│   └── contact/
│       └── page.tsx
└── (dashboard)/
    ├── layout.tsx
    └── users/
        └── page.tsx
```

**URL thực tế sẽ là:**

```text
/about
/contact
/users
```

Tên `(marketing)` và `(dashboard)` **không xuất hiện** trong URL.

#### Route group thường được dùng để:

- Sắp xếp các route theo từng khu vực chức năng
- Áp dụng layout khác nhau cho từng nhóm route
- Tách khu vực public, authentication và dashboard
- Giữ cấu trúc dự án rõ ràng mà không làm URL dài hơn

#### Ví dụ thực tế:

```text
app/
├── (public)/
│   ├── page.tsx
│   └── products/
│       └── page.tsx
├── (auth)/
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
└── (admin)/
    ├── layout.tsx
    └── dashboard/
        └── page.tsx
```

---

## Cách trả lời ngắn khi phỏng vấn

- **Dynamic route**: Dùng để xử lý các URL có tham số thay đổi, được khai báo bằng thư mục như `[id]`.
- **Nested route**: Được tạo bằng cách lồng các thư mục để thể hiện cấu trúc URL phân cấp.
- **Route group**: Sử dụng thư mục có dấu ngoặc tròn như `(admin)` để tổ chức route hoặc áp dụng layout riêng, nhưng **không làm tên thư mục xuất hiện trong URL**.

---

*Tài liệu tham khảo nhanh – Next.js App Router*
