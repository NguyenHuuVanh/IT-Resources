# Giai đoạn 5: Tích hợp CI/CD và Bảo mật nâng cao

## 1. Mục tiêu giai đoạn

Bạn cần:

- Tích hợp Docker vào pipeline CI/CD chuẩn.
- Áp dụng mô hình build once, deploy many.
- Thiết lập kiểm thử trong container.
- Hardening bảo mật container theo nhiều lớp.

---

## 2. Mô hình CI/CD với Docker

Luồng chuẩn:

1. Checkout source.
2. Build image từ commit.
3. Chạy test/lint/security scan.
4. Push image lên registry.
5. Deploy đúng image tag đã qua kiểm thử.

Nguyên tắc:

- Artifact duy nhất là image.
- Không build lại ở production.
- Rollback bằng cách quay về image tag trước.

---

## 3. Tagging strategy cho image

Nên dùng:

- `myapp:<git-sha>`
- `myapp:1.8.0`
- `myapp:main` (chỉ để tham chiếu luồng, không dùng deploy trực tiếp)

Tránh phụ thuộc `latest` trong môi trường production.

---

## 4. Pipeline tham chiếu (GitHub Actions)

```yaml
name: ci
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-test-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - name: Login registry
        run: echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login ghcr.io -u "${{ github.actor }}" --password-stdin

      - name: Build image
        run: docker build -t ghcr.io/org/myapp:${{ github.sha }} .

      - name: Run tests in container
        run: docker run --rm ghcr.io/org/myapp:${{ github.sha }} npm test

      - name: Push image
        run: docker push ghcr.io/org/myapp:${{ github.sha }}
```

---

## 5. Security scanning trong pipeline

Mục tiêu:

- Phát hiện CVE từ base image và dependencies.
- Chặn release khi vượt ngưỡng rủi ro.

Cách làm phổ biến:

- Trivy/Grype scan image.
- SCA cho package manager.
- Policy gate cho severity `HIGH/CRITICAL`.

---

## 6. Hardening runtime container

## 6.1 Không chạy root

- Chạy bằng user riêng.
- Giảm rủi ro privilege escalation.

## 6.2 Giới hạn tài nguyên

```bash
docker run -d --cpus=1 --memory=512m --pids-limit=200 myapp:sha
```

## 6.3 Giới hạn quyền Linux capabilities

Ví dụ giảm quyền:

```bash
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp:sha
```

## 6.4 `no-new-privileges`

```bash
docker run --security-opt no-new-privileges:true myapp:sha
```

## 6.5 Read-only filesystem

```bash
docker run --read-only --tmpfs /tmp myapp:sha
```

---

## 7. Rootless Docker

Khái niệm:

- Docker daemon và container chạy không cần root trực tiếp trên host.

Lợi ích:

- Giảm blast radius khi daemon/container bị compromise.

Lưu ý:

- Có giới hạn với một số tính năng networking/storage nâng cao.

---

## 8. Seccomp, AppArmor, SELinux

- **Seccomp:** lọc syscall nguy hiểm.
- **AppArmor/SELinux:** MAC policy để giới hạn truy cập tài nguyên hệ thống.

Trong production, nên dùng profile mặc định hoặc profile hardened theo chuẩn tổ chức.

---

## 9. Quản lý secrets đúng cách

Không lưu secrets trong:

- Dockerfile
- Git repo
- ENV hardcoded trong compose production

Nên dùng:

- Secret manager (Vault, AWS Secrets Manager, GCP Secret Manager...)
- Cơ chế secret của orchestrator.

---

## 10. Supply chain security

Các điểm quan trọng:

- Pin digest cho base image.
- Ký image (cosign/sigstore).
- Sinh SBOM (Software Bill of Materials).
- Theo dõi provenance của artifact.

---

## 11. Thực hành bắt buộc

1. Viết pipeline build/test/push image theo commit SHA.
2. Thêm bước scan image và fail pipeline khi có CVE nghiêm trọng.
3. Chạy container non-root + drop capabilities.
4. Bật read-only filesystem và tmpfs cho thư mục tạm.
5. Chứng minh rollback bằng cách redeploy tag cũ.

---

## 12. Tiêu chí hoàn thành giai đoạn

- [ ] Pipeline CI/CD chạy ổn định theo mô hình build once.
- [ ] Có image tagging strategy rõ ràng.
- [ ] Có bước security scan bắt buộc trước deploy.
- [ ] Runtime hardening gồm ít nhất 4 biện pháp.
- [ ] Có quy trình rollback bằng tag cụ thể.
