# Server Component và Client Component trong Next.js App Router

> Câu hỏi: Trong Next.js App Router, Server Component và Client Component khác nhau như thế nào? Khi nào cần dùng `"use client"`?

## Câu trả lời

Trong Next.js **App Router**, các `page` và `layout` **mặc định là Server Component**.

### 1. Server Component

Server Component chủ yếu được xử lý trên **server** và phù hợp với các công việc:

- Gọi API / fetch dữ liệu
- Truy vấn database
- Sử dụng biến môi trường bí mật (secret keys)
- Render nội dung không cần tương tác trực tiếp với người dùng

**Ưu điểm chính:**
- Giảm lượng JavaScript gửi xuống trình duyệt
- Bảo mật tốt hơn (code và secret không bị lộ ra client)
- Hiệu suất cao hơn

### 2. Client Component

Client Component được sử dụng khi giao diện cần các tính năng **chỉ hoạt động ở phía client**, ví dụ:

- Sử dụng state với `useState`
- Sử dụng lifecycle hoặc side effect với `useEffect`
- Xử lý sự kiện như `onClick`, `onChange`
- Truy cập API của trình duyệt (`window`, `document`, `localStorage`...)
- Sử dụng các custom hook phụ thuộc vào những tính năng trên

### 3. Cách khai báo Client Component

Để khai báo một Client Component, đặt `"use client"` ở **đầu file**, trước các câu lệnh `import`:

```tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
