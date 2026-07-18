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
