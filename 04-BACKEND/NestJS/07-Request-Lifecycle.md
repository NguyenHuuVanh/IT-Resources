# Vòng đời của một Request (Request Lifecycle)

> Hiểu rõ thứ tự các tầng xử lý một request giúp bạn biết đặt logic ở đâu cho đúng — và là câu hỏi phỏng vấn rất hay gặp.

## 1. Thứ tự tổng quát

```
Request đến
   │
   ▼
① Middleware              (sớm nhất - "cổng" ứng dụng)
   │
   ▼
② Guards                  (cho phép truy cập hay không?)
   │
   ▼
③ Interceptors (before)   (bọc quanh handler - trước)
   │
   ▼
④ Pipes                   (validate & transform input)
   │
   ▼
⑤ Route Handler           (Controller → Service)
   │
   ▼
⑥ Interceptors (after)    (bọc quanh handler - sau)
   │
   ▼
⑦ Exception Filters       (bắt lỗi nếu có)
   │
   ▼
Response trả về
```

> **Middleware luôn vào TRƯỚC Pipe.**

## 2. Vai trò từng tầng

| # | Tầng | Nhiệm vụ | Biết handler? |
|---|------|----------|---------------|
| 1 | **Middleware** | Xử lý request thô: log, CORS, parse body | ❌ Không |
| 2 | **Guards** | Quyết định cho phép truy cập (auth/role) | ✅ Có |
| 3 | **Interceptors (before)** | Đo thời gian, logging, biến đổi trước handler | ✅ Có |
| 4 | **Pipes** | Validate & transform dữ liệu đầu vào (DTO, ép kiểu) | ✅ Có |
| 5 | **Route Handler** | Nhận request, điều phối tới Service xử lý logic | — |
| 6 | **Interceptors (after)** | Biến đổi/định dạng response trả về, cache | ✅ Có |
| 7 | **Exception Filters** | Bắt & định dạng lỗi | ✅ Có |

## 3. Middleware vs Pipe — cái nào trước?

**Middleware chạy trước Pipe.**

| | Middleware | Pipe |
|---|------------|------|
| **Vị trí** | Sớm nhất, ở "cổng" ứng dụng | Ngay sát Route Handler |
| **Nhiệm vụ** | Xử lý request thô: log, CORS, parse | Validate & transform dữ liệu đầu vào |
| **Biết handler nào xử lý?** | ❌ Không (không có `ExecutionContext`) | ✅ Có |
| **Làm việc trên** | `req`, `res`, `next` (Express) | Giá trị tham số (`@Body`, `@Param`, `@Query`) |

> Mẹo nhớ: Middleware = **bảo vệ ở cổng** (gặp đầu tiên); Pipe = **kiểm tra giấy tờ ngay trước cửa phòng** (gặp cuối trước khi vào handler).

## 4. Thứ tự khi có nhiều cấp (global / controller / method)

Với Guard, Interceptor, Pipe, Filter — có thể đăng ký ở nhiều cấp. Thứ tự thực thi:

```
Global  →  Controller  →  Method (route handler)
```

Ví dụ với Guards: global guard chạy trước, rồi controller guard, rồi method guard.

> Riêng **Interceptors** có tính "bọc": phần *before* chạy theo thứ tự global → controller → method, còn phần *after* chạy ngược lại method → controller → global.

## 5. Khi có lỗi xảy ra

Bất kỳ tầng nào ném exception (Guard từ chối, Pipe validate fail, Service lỗi...) → request **nhảy thẳng** tới **Exception Filters** để định dạng response lỗi, bỏ qua các bước còn lại.

```
... → Pipe ném BadRequestException ──┐
                                      ▼
                              Exception Filter → Response 400
```

## 6. Câu hỏi phỏng vấn thường gặp

**Q: Khi nhận một request, Middleware hay Pipe chạy trước?**
> Middleware chạy trước. Thứ tự: Middleware → Guards → Interceptors → Pipes → Handler.

**Q: Kể thứ tự vòng đời request trong NestJS.**
> Middleware → Guards → Interceptors (before) → Pipes → Route Handler → Interceptors (after) → Exception Filters → Response.

**Q: Vì sao Middleware không thay được Guard?**
> Middleware không truy cập `ExecutionContext` nên không biết handler/metadata; Guard biết, phù hợp cho auth/role.

**Q: Thứ tự giữa global / controller / method?**
> Global → Controller → Method. Riêng phần *after* của Interceptor chạy ngược lại.

**Q: Khi Pipe validate thất bại thì sao?**
> Ném exception → nhảy tới Exception Filter, trả response lỗi (mặc định 400), không vào handler.

## 7. Tổng kết

- Thứ tự: **Middleware → Guards → Interceptors → Pipes → Handler → Interceptors → Exception Filters**.
- **Middleware vào trước Pipe**.
- Middleware không biết handler; Guards/Interceptors/Pipes/Filters thì biết (có `ExecutionContext`).
- Đa cấp chạy theo: Global → Controller → Method (Interceptor *after* chạy ngược).
- Có lỗi → nhảy tới Exception Filters.
