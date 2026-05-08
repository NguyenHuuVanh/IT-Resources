# Giai đoạn 2: Xây dựng và Tối ưu hóa Docker Image (Dockerfile)

## 1. Mục tiêu giai đoạn

Bạn cần đạt được:

- Viết Dockerfile chuẩn cho ứng dụng thật.
- Phân biệt rõ `CMD` và `ENTRYPOINT`.
- Tối ưu kích thước image, thời gian build, và mức độ bảo mật.
- Áp dụng thành thạo multi-stage build.

---

## 2. Cấu trúc Dockerfile từ cơ bản đến chuẩn production

Ví dụ tối thiểu:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["node", "src/index.js"]
```

Ý nghĩa chỉ lệnh chính:

- `FROM`: base image.
- `WORKDIR`: thư mục làm việc mặc định.
- `COPY`: đưa file vào image.
- `RUN`: chạy lệnh khi build.
- `ENV`: biến môi trường trong image/container.
- `EXPOSE`: metadata về cổng dịch vụ.
- `CMD`: command mặc định khi start container.

---

## 3. `CMD` và `ENTRYPOINT`

## 3.1 Quy tắc vận hành

- `ENTRYPOINT`: bắt buộc chạy, khó bị thay thế.
- `CMD`: đối số mặc định hoặc command mặc định, dễ override.

## 3.2 Mẫu thực tế

```dockerfile
ENTRYPOINT ["node", "dist/index.js"]
CMD ["--port=3000"]
```

Khi chạy:

```bash
docker run myapp --port=4000
```

Container vẫn chạy entrypoint `node dist/index.js`, chỉ thay đối số.

---

## 4. Tối ưu build cache

Docker cache theo layer; layer đổi thì các layer sau bị build lại.

Thứ tự tốt:

1. Copy file phụ thuộc (`package.json`, lock file) trước.
2. Cài dependencies.
3. Copy source code sau cùng.

Ví dụ:

```dockerfile
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
```

Nếu chỉ đổi code mà không đổi dependency, build sẽ nhanh hơn rõ rệt.

---

## 5. Giảm kích thước image

## 5.1 Chọn base image

- `alpine`: nhỏ, phổ biến.
- `distroless`: rất gọn, giảm bề mặt tấn công.
- `scratch`: siêu tối giản cho binary tĩnh.

## 5.2 Dùng `.dockerignore`

```text
node_modules
.git
.env
coverage
dist
*.log
```

Lợi ích:

- Giảm build context.
- Giảm rủi ro lộ file nhạy cảm.

## 5.3 Không cài thừa package

- Node.js: `npm ci --omit=dev` ở runtime stage.
- Python: dùng `--no-cache-dir`.
- Ubuntu: `apt-get clean` và xóa apt lists.

---

## 6. Multi-stage build (trọng tâm)

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runtime
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

Lợi ích:

- Tách môi trường build và runtime.
- Runtime image gọn, ít công cụ dư thừa.
- Giảm attack surface.

---

## 7. Bảo mật image ở mức Dockerfile

- Không chạy root nếu không cần.
- Pin version base image (vd: `node:20.12-alpine3.19`).
- Không hardcode secrets trong Dockerfile.
- Ưu tiên copy file đúng phạm vi cần thiết, tránh `COPY . .` bừa bãi ở giai đoạn runtime.

Ví dụ non-root:

```dockerfile
RUN addgroup -S app && adduser -S app -G app
USER app
```

---

## 8. Build nâng cao với BuildKit

Khi dùng BuildKit bạn có thể:

- Mount cache để tăng tốc.
- Mount secret trong build mà không lưu vào layer.

Ví dụ cache:

```dockerfile
RUN --mount=type=cache,target=/root/.npm npm ci
```

Ví dụ secret:

```dockerfile
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm ci
```

---

## 9. Gắn nhãn image và metadata

```dockerfile
LABEL org.opencontainers.image.title="myapp" \
      org.opencontainers.image.source="https://example.com/repo" \
      org.opencontainers.image.version="1.0.0"
```

Giúp trace image trong CI/CD, audit và governance.

---

## 10. Anti-pattern thường gặp

- Dùng `latest` cho production.
- Cài dependencies trước khi copy lock file.
- Để secrets trong `ENV` hoặc commit `.env` vào image.
- Chạy root mặc định.
- Dockerfile quá dài, không tách stage.

---

## 11. Thực hành bắt buộc

1. Viết 1 Dockerfile đơn giản cho API.
2. Đo size image ban đầu.
3. Thêm `.dockerignore`, đo lại.
4. Chuyển sang multi-stage build, đo lại lần nữa.
5. Chạy container bằng non-root user.
6. Validate app vẫn chạy đúng endpoint healthcheck.

---

## 12. Tiêu chí hoàn thành giai đoạn

- [ ] Viết được Dockerfile rõ ràng cho môi trường dev/prod.
- [ ] Giải thích đúng vai trò `CMD` và `ENTRYPOINT`.
- [ ] Cắt giảm size image đáng kể bằng multi-stage.
- [ ] Build tận dụng cache tốt (rebuild nhanh khi đổi code).
- [ ] Runtime container chạy non-root và không chứa secret trong image.
