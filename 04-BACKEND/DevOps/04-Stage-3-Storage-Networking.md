# Giai đoạn 3: Quản lý Lưu trữ (Storage) và Mạng (Networking)

## 1. Mục tiêu giai đoạn

Bạn cần:

- Hiểu vì sao container mặc định là stateless.
- Chọn đúng giữa volume, bind mount, tmpfs.
- Nắm mô hình mạng Docker: bridge, host, none, overlay.
- Thiết kế được kết nối service nội bộ và public qua port mapping.

---

## 2. Storage trong Docker: bản chất và hệ quả

Container dùng writable layer tạm thời. Khi xóa container, dữ liệu layer đó mất.

Kết luận:

- Không đặt dữ liệu quan trọng trong filesystem tạm của container.
- Luôn quyết định chiến lược lưu trữ ngay từ thiết kế.

---

## 3. Volumes

## 3.1 Khi dùng

- Database (PostgreSQL, MySQL, MongoDB).
- Dữ liệu cần bền vững qua vòng đời container.

## 3.2 Ưu điểm

- Docker quản lý đường dẫn và quyền tốt hơn bind mount.
- Hiệu năng ổn định trong môi trường production.
- Dễ backup/restore.

## 3.3 Lệnh cốt lõi

```bash
docker volume create pg_data
docker volume ls
docker volume inspect pg_data
docker run -d --name pg -v pg_data:/var/lib/postgresql/data postgres:15-alpine
```

---

## 4. Bind mounts

## 4.1 Khi dùng

- Development: đồng bộ code trực tiếp từ host vào container.
- Cần truy cập file cụ thể trên host.

## 4.2 Rủi ro

- Phụ thuộc cấu trúc host path.
- Sai quyền file dễ gây lỗi runtime.
- Ít portable hơn volume.

Ví dụ:

```bash
docker run --rm -it -v ${PWD}:/app -w /app node:20-alpine npm test
```

---

## 5. tmpfs mounts

## 5.1 Khi dùng

- Dữ liệu tạm thời, nhạy cảm, cần tốc độ cao.
- Không muốn ghi xuống disk host.

## 5.2 Ví dụ

```bash
docker run -d --tmpfs /run:rw,noexec,nosuid,size=64m myapp:latest
```

---

## 6. So sánh nhanh 3 cơ chế mount

| Cơ chế       | Bền vững | Hiệu năng | Phù hợp          |
| ------------ | -------- | --------- | ---------------- |
| Volume       | Cao      | Tốt       | Production DB    |
| Bind mount   | Theo host| Tốt       | Dev, local sync  |
| tmpfs mount  | Không    | Rất cao   | Cache/secret tạm |

---

## 7. Networking trong Docker

## 7.1 bridge network (mặc định)

- Phổ biến nhất cho single-host.
- Container cùng bridge gọi nhau qua tên container/service.

```bash
docker network ls
docker network create app_net
docker run -d --name api --network app_net myapi:latest
docker run -d --name db --network app_net postgres:15-alpine
```

## 7.2 host network

- Container dùng trực tiếp network stack của host.
- Ít overhead nhưng giảm cách ly.
- Cần cẩn thận xung đột cổng.

## 7.3 none network

- Không có kết nối mạng.
- Dùng cho workload không cần network hoặc yêu cầu hardening.

## 7.4 overlay network

- Dùng cho multi-host (Swarm).
- Cho container trên nhiều node giao tiếp như cùng mạng logic.

---

## 8. Port mapping và service exposure

```bash
docker run -d -p 8080:3000 myapp:latest
```

- Bên trái `8080`: cổng host.
- Bên phải `3000`: cổng trong container.

Best practice:

- Chỉ expose cổng thật sự cần public.
- DB/Redis thường chỉ nên mở nội bộ network.

---

## 9. DNS nội bộ của Docker

Trong cùng network tùy chỉnh, container có thể gọi nhau theo tên:

- `postgres://user:pass@db:5432/app`
- `redis://redis:6379`

Không nên hardcode IP container vì IP có thể đổi.

---

## 10. Debug storage/network thực chiến

Lệnh hữu ích:

```bash
docker inspect <container>
docker network inspect <network>
docker exec -it <container> sh
```

Cần kiểm tra:

- Mount path đúng chưa?
- Container có join đúng network chưa?
- DNS resolve theo tên service có chạy không?
- Port mapping có đúng phía host/container không?

---

## 11. Bảo mật trong storage/network

- Mount read-only khi có thể: `:ro`.
- Không bind mount thư mục nhạy cảm của host.
- Tách network theo từng nhóm service.
- Hạn chế publish cổng DB ra ngoài internet.

---

## 12. Thực hành bắt buộc

1. Dựng PostgreSQL dùng named volume.
2. Tạo API container kết nối DB qua tên service.
3. Dùng bind mount để hot-reload code ở môi trường dev.
4. Tách 2 network: `frontend_net` và `backend_net`.
5. Chỉ publish cổng API, không publish cổng DB.

---

## 13. Tiêu chí hoàn thành giai đoạn

- [ ] Giải thích đúng stateless và persistence trong Docker.
- [ ] Chọn đúng volume/bind/tmpfs cho từng bài toán.
- [ ] Thiết lập được network cho app nhiều service.
- [ ] Debug được lỗi kết nối bằng `inspect` và `exec`.
- [ ] Áp dụng tối thiểu 2 nguyên tắc hardening storage/network.
