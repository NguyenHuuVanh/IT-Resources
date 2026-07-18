# Next.js khác React thuần ở những điểm nào?

> Câu hỏi thường gặp khi bắt đầu học Next.js

## Câu trả lời

Next.js khác React thuần ở một số điểm chính như sau:

### 1. Bản chất: Thư viện vs Framework

- **React** là một **thư viện** (library) dùng để xây dựng giao diện người dùng (UI).
- **Next.js** là một **framework** được phát triển dựa trên React.  
  Next.js cung cấp sẵn nhiều tính năng để xây dựng một ứng dụng web hoàn chỉnh (routing, rendering, API, optimization...).

### 2. Routing (Định tuyến)

- **Next.js** hỗ trợ **routing dựa trên cấu trúc thư mục** (File-system based routing).  
  Lập trình viên có thể sử dụng **App Router** hoặc **Pages Router** để tạo các trang và đường dẫn một cách tự động.
- **React thuần**: Phải cài thêm thư viện bên thứ ba như **React Router** và tự cấu hình routing.

### 3. Cơ chế Render

Next.js hỗ trợ nhiều cơ chế render mạnh mẽ:

- **Server-Side Rendering (SSR)**
- **Static Site Generation (SSG)**
- **Incremental Static Regeneration (ISR)**
- **Client-Side Rendering (CSR)**
- **React Server Components** (mặc định trong App Router)

Trong khi đó, **React thuần** thường chỉ render ở phía client (CSR) nếu không cấu hình thêm các giải pháp phức tạp.

### 4. SEO (Tối ưu công cụ tìm kiếm)

- Next.js hỗ trợ SEO tốt hơn vì nội dung có thể được **render sẵn trên server** hoặc tại thời điểm build.
- Điều này giúp công cụ tìm kiếm (Google, Bing...) dễ đọc và lập chỉ mục nội dung hơn.
- React thuần (CSR) thường gặp khó khăn với SEO vì nội dung chỉ được tạo sau khi JavaScript chạy trên trình duyệt.

### 5. Các tính năng bổ sung của Next.js

Ngoài những điểm trên, Next.js còn cung cấp sẵn nhiều tính năng hữu ích:

- Tối ưu hình ảnh (`next/image`)
- Tối ưu font (`next/font`)
- **Server Components** & **Client Components**
- **API Routes** / Route Handlers
- **Server Actions**
- Caching mạnh mẽ
- Middleware
- Turbopack (bundler nhanh)
- Tự động code splitting & prefetching

---

## Tóm tắt

| Tiêu chí              | React thuần                  | Next.js                              |
|-----------------------|------------------------------|--------------------------------------|
| Bản chất              | Thư viện UI                  | Full-stack Framework                 |
| Routing               | Cần cài React Router         | File-based routing (sẵn có)          |
| Rendering             | Chủ yếu CSR                  | SSR, SSG, ISR, CSR, Server Components|
| SEO                   | Yếu (nếu chỉ CSR)            | Tốt hơn nhiều                        |
| Tối ưu & Tính năng    | Phải tự cấu hình             | Có sẵn nhiều tính năng tối ưu        |

**Kết luận:**  
React tập trung vào việc xây dựng giao diện người dùng.  
Next.js mở rộng React thành một **framework đầy đủ** hơn, hỗ trợ routing, nhiều phương thức render, SEO và tối ưu hiệu năng — giúp developer xây dựng ứng dụng production nhanh và hiệu quả hơn.

---

*Tài liệu tham khảo nhanh – cập nhật cho Next.js (App Router)*
