# Giai đoạn 1: Cơ bản về Container và Kiến trúc Docker

## 1. Mục tiêu giai đoạn

Sau giai đoạn này, bạn cần:

- Hiểu bản chất của container và so sánh đúng với virtual machine.
- Nắm kiến trúc Docker theo mô hình client-server.
- Sử dụng thành thạo lệnh Docker cơ bản để pull/build/run/inspect container.
- Biết vòng đời image và container trong thực tế phát triển.

---

## 2. Tư duy nền tảng: VM vs Container

## 2.1 Virtual Machine (VM)

- Ảo hóa ở mức phần cứng.
- Mỗi VM có guest OS riêng.
- Tốn tài nguyên hơn (RAM/CPU/disk), boot chậm hơn.
- Cách ly mạnh, phù hợp bài toán legacy hoặc nhiều hệ điều hành khác nhau.

## 2.2 Container

- Ảo hóa ở mức hệ điều hành.
- Chia sẻ kernel của host.
- Nhẹ, start nhanh, mật độ deploy cao.
- Phù hợp microservices, CI/CD, scale ngang nhanh.

## 2.3 Khi nào dùng cái nào?

- Dùng VM khi cần tách OS hoàn toàn hoặc workload đặc thù.
- Dùng Container khi cần chuẩn hóa môi trường và tự động hóa pipeline.

---

## 3. Kiến trúc Docker

## 3.1 Docker Client

- Là CLI (`docker ...`) hoặc API client.
- Gửi request đến Docker Daemon.

## 3.2 Docker Daemon (`dockerd`)

- Build image.
- Create/run/stop/remove container.
- Quản lý network, volume, cache.

## 3.3 Docker Registry

- Kho lưu image: Docker Hub, GHCR, ECR, ACR...
- Hỗ trợ push/pull image qua tag.

## 3.4 Luồng hoạt động

1. Viết `Dockerfile`.
2. `docker build` tạo image.
3. `docker run` tạo container từ image.
4. `docker push` đẩy image lên registry.
5. Môi trường khác `docker pull` và chạy đúng image đó.

---

## 4. Đối tượng cốt lõi trong Docker

## 4.1 Image

- Read-only template nhiều layer.
- Có tag để version hóa.
- Không phải process đang chạy.

## 4.2 Container

- Runtime instance của image.
- Có state chạy/dừng.
- Có filesystem tạm thời theo cơ chế copy-on-write.

## 4.3 Repository và Tag

- `myorg/myapp:1.4.2`
- `myorg/myapp:latest` chỉ là tag mặc định, không phải version đáng tin cậy cho production.

## 4.4 Volumes và Networks (nhìn nhanh)

- Volume giữ dữ liệu bền vững.
- Network cho giao tiếp giữa containers.

---

## 5. Bộ lệnh Docker bắt buộc phải thuộc

```bash
# Xem version Docker
docker version
docker info

# Image
docker pull nginx:alpine
docker images
docker image inspect nginx:alpine

# Container
docker run -d --name web -p 8080:80 nginx:alpine
docker ps
docker ps -a
docker logs -f web
docker exec -it web sh
docker stop web
docker rm web

# Cleanup cẩn thận
docker system df
docker image prune
docker container prune
```

---

## 6. Cấp độ trung cấp: quan sát và debug

## 6.1 Inspect container

```bash
docker inspect web
```

Bạn cần đọc được:

- Mounts
- NetworkSettings
- Entrypoint/Cmd
- Env

## 6.2 Theo dõi tài nguyên

```bash
docker stats
```

Hiểu cột CPU %, MEM USAGE, NET I/O, BLOCK I/O để phát hiện container bất thường.

## 6.3 Exit code

```bash
docker ps -a
```

Các mã phổ biến:

- `0`: kết thúc bình thường.
- `1`: lỗi runtime chung.
- `137`: bị kill (thường do OOM hoặc SIGKILL).

---

## 7. Cấp độ nâng cao: Linux kernel phía sau container

- **Namespaces:** cô lập PID, NET, MNT, UTS, IPC, USER.
- **cgroups:** giới hạn CPU/RAM/I/O.
- **Union FS:** ghép các layer của image thành filesystem nhìn thấy trong container.

Bạn chưa cần đi quá sâu kernel code, nhưng phải nắm 3 ý này để hiểu performance và security.

---

## 8. Best practices ngay từ đầu

- Không chạy production bằng tag `latest`.
- Luôn đặt tên container có nghĩa.
- Không nhúng secret trực tiếp vào image.
- Dùng `docker logs`, `docker inspect`, `docker stats` trước khi kết luận bug.
- Cleanup có chủ đích, tránh xóa hàng loạt khi chưa kiểm tra.

---

## 9. Thực hành bắt buộc

1. Chạy Nginx container, map cổng `8080:80`.
2. Vào shell container, kiểm tra file cấu hình Nginx.
3. Dừng rồi xóa container.
4. Chạy lại container từ cùng image để thấy việc tái sử dụng image.
5. Dùng `docker inspect` đọc network và mounts.
6. Dùng `docker stats` ghi lại mức RAM/CPU trung bình.

---

## 10. Tiêu chí hoàn thành giai đoạn

- [ ] Giải thích được khác biệt VM và Container bằng ngôn ngữ kỹ thuật.
- [ ] Mô tả đúng kiến trúc Docker client-daemon-registry.
- [ ] Chạy được container, xem log, exec vào shell, stop/rm không lỗi.
- [ ] Đọc được output `docker inspect` mức cơ bản.
- [ ] Biết ít nhất 3 nguyên nhân thường gặp khiến container thoát sớm.
