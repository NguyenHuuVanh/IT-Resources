# Giai đoạn 4: Điều phối đa Container với Docker Compose

## 1. Mục tiêu giai đoạn

Bạn cần:

- Dựng và vận hành ứng dụng nhiều service bằng `docker-compose.yml`.
- Quản lý dependencies giữa API, DB, cache.
- Thiết lập healthcheck, network, volume đúng chuẩn.
- Biết cấu trúc compose cho dev và production.

---

## 2. Docker Compose là gì?

Compose là công cụ định nghĩa toàn bộ stack container bằng file YAML:

- Dễ tái sử dụng.
- Dễ version hóa cùng source code.
- Giảm sai sót khi khởi động nhiều service thủ công.

---

## 3. Thành phần chính của `docker-compose.yml`

- `services`: danh sách service.
- `networks`: mạng logic.
- `volumes`: lưu trữ bền vững.
- `configs` / `secrets` (khi cần môi trường nâng cao).

Ví dụ nền tảng:

```yaml
version: "3.9"
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://user:pass@db:5432/app
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app_net

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    volumes:
      - pg_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app_net

networks:
  app_net:

volumes:
  pg_data:
```

---

## 4. `depends_on` và `healthcheck`

Lưu ý quan trọng:

- `depends_on` chỉ đảm bảo thứ tự start.
- Không tự đảm bảo service đã sẵn sàng nhận kết nối.
- Cần `healthcheck` để tăng độ ổn định khởi động.

Nên thêm retry logic ở ứng dụng thay vì phụ thuộc hoàn toàn vào `depends_on`.

---

## 5. Profiles và tách môi trường

Bạn có thể dùng profile để bật/tắt service theo ngữ cảnh:

```yaml
services:
  api:
    profiles: ["default"]
  adminer:
    image: adminer
    ports:
      - "8081:8080"
    profiles: ["dev"]
```

Chạy:

```bash
docker compose --profile dev up -d
```

---

## 6. Override file theo môi trường

- `docker-compose.yml`: base.
- `docker-compose.override.yml`: local dev.
- `docker-compose.prod.yml`: production tuning.

Ví dụ chạy:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## 7. Build và image strategy trong Compose

Hai hướng:

- `build:` dùng cho local/dev.
- `image:` dùng image đã build sẵn từ CI cho staging/prod.

Production nên dùng image tag bất biến (commit SHA/semver), tránh build trực tiếp trên server production.

---

## 8. Vận hành và quan sát

Lệnh quan trọng:

```bash
docker compose up -d
docker compose ps
docker compose logs -f api
docker compose exec api sh
docker compose config
docker compose down
```

Lưu ý:

- `docker compose config` giúp validate và render file sau merge override.

---

## 9. Scaling với Compose

```bash
docker compose up -d --scale api=3
```

Compose scale được cho nhu cầu local/staging; production lớn vẫn nên chuyển orchestration chuyên dụng (Swarm/K8s).

---

## 10. Best practices cho Compose

- Dùng network riêng thay vì mặc định cho toàn bộ hệ thống.
- DB/Redis không public port nếu không cần.
- Đặt healthcheck cho service quan trọng.
- Quản lý biến môi trường qua `.env` (không commit secret thật).
- Dùng named volumes cho stateful data.

---

## 11. Anti-pattern thường gặp

- Nhét mọi service vào một network không phân tách.
- Không healthcheck nhưng kỳ vọng startup ổn định.
- Publish cổng database ra internet.
- Dùng chung file compose cho mọi môi trường mà không có override/profile.

---

## 12. Thực hành bắt buộc

1. Dựng stack `frontend + api + postgres + redis`.
2. Thiết lập healthcheck cho API và DB.
3. Dùng `.env` để cấu hình.
4. Tách file compose dev/prod.
5. Scale API lên 2-3 replicas ở local.
6. Ghi nhận log và xử lý một lỗi kết nối DB mô phỏng.

---

## 13. Tiêu chí hoàn thành giai đoạn

- [ ] Viết được file compose hoàn chỉnh cho hệ thống nhiều service.
- [ ] Hiểu đúng hành vi `depends_on` và `healthcheck`.
- [ ] Vận hành thành thạo stack qua `docker compose`.
- [ ] Tổ chức compose cho dev/prod rõ ràng.
- [ ] Xử lý được lỗi startup và networking cơ bản.
