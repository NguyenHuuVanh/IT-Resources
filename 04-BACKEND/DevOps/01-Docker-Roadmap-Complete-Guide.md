# Lộ Trình Chinh Phục Docker Từ Cơ Bản Đến Nâng Cao

Tài liệu này tổng hợp lộ trình học Docker theo 6 giai đoạn, đi từ nền tảng đến triển khai hệ thống lớn.

## Bộ tài liệu chi tiết theo từng giai đoạn

1. [Giai đoạn 1 - Container và kiến trúc Docker](./02-Stage-1-Container-Architecture-Fundamentals.md)
2. [Giai đoạn 2 - Dockerfile và tối ưu image](./03-Stage-2-Dockerfile-Image-Optimization.md)
3. [Giai đoạn 3 - Storage và Networking](./04-Stage-3-Storage-Networking.md)
4. [Giai đoạn 4 - Docker Compose đa container](./05-Stage-4-Docker-Compose-Multi-Container.md)
5. [Giai đoạn 5 - CI/CD và bảo mật nâng cao](./06-Stage-5-CI-CD-and-Advanced-Security.md)
6. [Giai đoạn 6 - Orchestration (Swarm/Kubernetes)](./07-Stage-6-Orchestration-Swarm-Kubernetes.md)

## Mục tiêu chung

- Hiểu đúng bản chất container và kiến trúc Docker.
- Tự viết, tối ưu, và bảo mật Docker image cho ứng dụng thực tế.
- Vận hành ứng dụng nhiều container bằng Docker Compose.
- Tích hợp Docker vào CI/CD và mở rộng sang orchestration (Swarm/Kubernetes).

---

## Giai đoạn 1: Cơ bản về Container và Kiến trúc Docker

### 1.1 Container vs Virtual Machine

- **Virtual Machine (VM):** Mỗi VM có guest OS riêng, nặng hơn, tốn RAM/CPU hơn.
- **Container:** Chia sẻ OS kernel của host, nhẹ, start nhanh, phù hợp scale linh hoạt.

### 1.2 Kiến trúc Docker (Client - Server)

- **Docker Client:** CLI (`docker ...`) để gửi lệnh.
- **Docker Daemon (`dockerd`):** Chạy nền, build/run/manage container.
- **Docker Registry:** Nơi lưu image (Docker Hub, GHCR, ECR...).

### 1.3 Đối tượng cốt lõi và lệnh nền tảng

- **Image:** Blueprint chỉ đọc.
- **Container:** Runtime instance của image.
- **Registry/Repository/Tag:** Quản lý phiên bản image.

Lệnh cơ bản:

```bash
docker pull nginx:alpine
docker images
docker run -d --name web -p 8080:80 nginx:alpine
docker ps
docker logs -f web
docker stop web && docker rm web
```

### Kết quả cần đạt

- Phân biệt rõ VM và container.
- Đọc được luồng: pull image -> run container -> quan sát logs -> dừng/xóa.

---

## Giai đoạn 2: Xây dựng và Tối ưu Docker Image (Dockerfile)

### 2.1 Cú pháp Dockerfile cốt lõi

Các chỉ lệnh quan trọng:

- `FROM`
- `WORKDIR`
- `COPY`
- `RUN`
- `ENV`
- `EXPOSE`
- `CMD`
- `ENTRYPOINT`

Ví dụ:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "src/index.js"]
```

### 2.2 `CMD` vs `ENTRYPOINT`

- **ENTRYPOINT:** Tiến trình chính cố định của container.
- **CMD:** Tham số mặc định (hoặc command mặc định), dễ bị override khi `docker run`.

Mẫu kết hợp:

```dockerfile
ENTRYPOINT ["node", "src/index.js"]
CMD ["--port=3000"]
```

### 2.3 Tối ưu kích thước + build cache

- Đặt layer ít thay đổi lên trước (cài dependency trước, copy source sau).
- Dùng base image gọn: `alpine`, `distroless`, hoặc `scratch` (khi phù hợp).
- Dùng `.dockerignore` để giảm build context.

Ví dụ `.dockerignore`:

```text
node_modules
.git
.env
npm-debug.log
dist
coverage
```

### 2.4 Multi-stage build (bắt buộc nắm vững)

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --omit=dev
CMD ["node", "dist/index.js"]
```

### Kết quả cần đạt

- Viết được Dockerfile cho app thật.
- Biết giảm size image đáng kể bằng multi-stage và `.dockerignore`.

---

## Giai đoạn 3: Storage và Networking

### 3.1 Storage: dữ liệu container là tạm thời

Container mặc định **stateless**; xóa container là có thể mất dữ liệu.

3 kiểu mount chính:

- **Volumes:** Docker quản lý, bền vững, tốt cho database.
- **Bind mounts:** Map thư mục host vào container, tiện cho dev.
- **tmpfs mounts:** Dữ liệu trên RAM, nhanh và không ghi xuống disk.

Ví dụ:

```bash
# Volume
docker volume create pg_data
docker run -d --name pg -v pg_data:/var/lib/postgresql/data postgres:15

# Bind mount
docker run --rm -it -v ${PWD}:/app -w /app node:20-alpine npm test
```

### 3.2 Networking

- `bridge`: mặc định cho container cùng host.
- `host`: dùng chung network stack của host.
- `none`: không cấp mạng.
- `overlay`: cho multi-host (Swarm/K8s context).

Port mapping:

```bash
docker run -d -p 8080:3000 myapp:latest
```

### Kết quả cần đạt

- Chọn đúng cơ chế lưu trữ theo bài toán.
- Hiểu cách container giao tiếp nội bộ và public service qua port mapping.

---

## Giai đoạn 4: Điều phối đa container với Docker Compose

### 4.1 Khái niệm

Docker Compose giúp định nghĩa toàn bộ stack nhiều service trong một file `docker-compose.yml`.

### 4.2 Cấu hình cốt lõi

- `services`
- `networks`
- `volumes`
- `depends_on`
- `healthcheck`

Ví dụ:

```yaml
version: "3.9"
services:
  api:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/app

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

volumes:
  pg_data:
```

Lệnh thường dùng:

```bash
docker compose up -d
docker compose ps
docker compose logs -f api
docker compose down
```

### Kết quả cần đạt

- Chạy trọn vẹn stack app + DB + cache bằng 1 lệnh.
- Đọc logs, troubleshoot dependency giữa services.

---

## Giai đoạn 5: CI/CD và Bảo mật nâng cao

### 5.1 Docker trong CI/CD

Nguyên tắc quan trọng:

- **Build once:** Build image tại CI một lần.
- **Test trong container:** Đảm bảo môi trường test đồng nhất.
- **Deploy đúng image đã test:** Không build lại ở production.

Pipeline mẫu:

1. Checkout code
2. Build image + tag theo commit SHA
3. Run unit/integration tests
4. Push image lên registry
5. Deploy image tag đó lên môi trường đích

### 5.2 Bảo mật container

- Linux Namespaces: cô lập process/network/mount/user...
- cgroups: giới hạn tài nguyên CPU/RAM.
- Chạy non-root hoặc rootless Docker khi phù hợp.
- Giảm đặc quyền bằng:
  - Linux Capabilities
  - Seccomp profile
  - AppArmor/SELinux
- Quét lỗ hổng image (ví dụ Trivy, Docker Scout).

Ví dụ giới hạn tài nguyên:

```bash
docker run -d --cpus="1.0" --memory="512m" myapp:latest
```

### Kết quả cần đạt

- Có pipeline CI/CD dùng Docker ổn định.
- Biết các lớp hardening cơ bản trước khi lên production.

---

## Giai đoạn 6: Quản trị hệ thống lớn (Orchestration)

### 6.1 Docker Swarm

- Tích hợp sẵn trong hệ sinh thái Docker.
- Setup nhanh, dễ học, hợp dự án nhỏ/vừa.
- Có service scaling, rolling update, overlay network.

### 6.2 Kubernetes (K8s)

- Chuẩn công nghiệp cho hệ thống phân tán lớn.
- Hỗ trợ sâu về auto-scaling, self-healing, service discovery, policy.
- Đơn vị triển khai chuyển từ container đơn lẻ sang **Pod** (nhóm container).

### 6.3 Khi nào chọn Swarm vs Kubernetes

- Chọn **Swarm** khi cần đơn giản, thời gian triển khai nhanh.
- Chọn **Kubernetes** khi cần mở rộng lớn, đa môi trường, quản trị phức tạp.

### Kết quả cần đạt

- Hiểu rõ ngưỡng chuyển từ Docker/Compose sang orchestration.
- Nắm tư duy triển khai production ở quy mô cluster.

---

## Lộ trình thực hành đề xuất (8-12 tuần)

1. Tuần 1-2: Giai đoạn 1 + lệnh Docker cơ bản.
2. Tuần 3-4: Dockerfile, cache, multi-stage.
3. Tuần 5-6: Storage + networking + debug.
4. Tuần 7-8: Docker Compose cho stack hoàn chỉnh.
5. Tuần 9-10: CI/CD + security hardening.
6. Tuần 11-12: Swarm/K8s và bài toán scale.

## Checklist hoàn thành

- [ ] Viết Dockerfile production-ready cho 1 ứng dụng thật.
- [ ] Giảm kích thước image ít nhất 30-70% bằng tối ưu Dockerfile.
- [ ] Dựng thành công stack app + db bằng Docker Compose.
- [ ] Có CI pipeline build/test/push image theo commit.
- [ ] Áp dụng tối thiểu 3 biện pháp bảo mật container.
- [ ] Triển khai thử lên Swarm hoặc Kubernetes.

## Tổng kết

Docker không chỉ là công cụ đóng gói ứng dụng, mà là nền tảng chuẩn hóa toàn bộ vòng đời build-test-deploy. Nếu đi đúng 6 giai đoạn trên và có project thực hành liên tục, bạn sẽ chuyển từ mức "biết chạy container" sang mức "vận hành hệ thống container trong production".
