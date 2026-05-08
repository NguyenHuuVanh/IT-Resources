# Giai đoạn 6: Quản trị hệ thống lớn (Container Orchestration)

## 1. Mục tiêu giai đoạn

Bạn cần:

- Hiểu vì sao cần orchestration khi hệ thống vượt quy mô một host.
- Nắm Docker Swarm ở mức vận hành cụm nhỏ-vừa.
- Nắm Kubernetes ở mức kiến trúc và triển khai production.
- Biết tiêu chí chọn Swarm hay Kubernetes theo bối cảnh thực tế.

---

## 2. Vì sao Compose chưa đủ cho production lớn?

Compose mạnh ở single-host hoặc môi trường dev/staging. Khi hệ thống lớn dần, bạn cần:

- Scheduling workload trên nhiều node.
- Tự phục hồi khi container/node lỗi.
- Rolling update có kiểm soát.
- Service discovery và load balancing đa host.
- Chính sách bảo mật và quản trị tập trung.

Đây là lý do orchestration xuất hiện.

---

## 3. Docker Swarm

## 3.1 Thành phần

- **Manager node:** điều phối cluster state.
- **Worker node:** chạy workload.
- **Service:** khai báo desired state.
- **Task:** instance thực tế của service.

## 3.2 Ưu điểm

- Dễ học, tích hợp sẵn Docker CLI.
- Setup nhanh cho team nhỏ/vừa.
- Phù hợp workload không quá phức tạp.

## 3.3 Lệnh nền tảng

```bash
docker swarm init
docker swarm join-token worker
docker service create --name web --replicas 3 -p 80:80 nginx:alpine
docker service ls
docker service ps web
docker service scale web=5
docker service rm web
```

---

## 4. Kubernetes (K8s)

## 4.1 Khái niệm cốt lõi

- **Pod:** đơn vị triển khai nhỏ nhất, gồm 1 hoặc nhiều container.
- **Deployment:** quản lý replica và rolling update cho pod.
- **Service:** cung cấp endpoint ổn định để truy cập pod.
- **ConfigMap/Secret:** cấu hình và dữ liệu nhạy cảm.
- **Ingress:** routing HTTP/HTTPS từ bên ngoài vào cluster.

## 4.2 Khả năng mạnh của K8s

- Auto-scaling (HPA).
- Self-healing (reschedule pod khi node lỗi).
- Rolling update/rollback rõ ràng.
- Hệ sinh thái mở rộng rất lớn (observability, policy, service mesh).

---

## 5. So sánh Swarm vs Kubernetes

| Tiêu chí             | Docker Swarm                  | Kubernetes                          |
| -------------------- | ----------------------------- | ----------------------------------- |
| Độ phức tạp          | Thấp hơn                      | Cao hơn                             |
| Setup ban đầu        | Nhanh                         | Cần đầu tư hơn                      |
| Tính năng mở rộng    | Đủ dùng cho cụm nhỏ/vừa       | Rất mạnh cho hệ thống lớn           |
| Hệ sinh thái         | Gọn, ít plugin hơn            | Rộng, nhiều chuẩn công nghiệp       |
| Learning curve       | Dễ tiếp cận                   | Dốc hơn                             |

Nguyên tắc chọn:

- Chọn Swarm khi cần nhanh, đơn giản, nguồn lực vận hành ít.
- Chọn Kubernetes khi cần tiêu chuẩn hóa hạ tầng dài hạn và scale lớn.

---

## 6. Tư duy triển khai production trên orchestration

Bạn cần quan tâm:

- **Resource requests/limits:** tránh tranh chấp tài nguyên.
- **Readiness/Liveness probes:** điều phối traffic an toàn.
- **Rolling strategy:** giảm downtime khi deploy.
- **Observability:** metrics/logs/traces tập trung.
- **Disaster recovery:** backup dữ liệu stateful và phục hồi cluster.

---

## 7. Mẫu Deployment + Service tối thiểu (K8s)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: ghcr.io/org/myapp:1.0.0
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
          livenessProbe:
            httpGet:
              path: /live
              port: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 3000
```

---

## 8. CI/CD cho orchestration

Luồng khuyến nghị:

1. CI build/test/scan image.
2. Push image với tag bất biến.
3. CD cập nhật manifest chart/yaml theo tag mới.
4. Triển khai rolling update.
5. Tự động rollback nếu health check fail.

---

## 9. Bảo mật cụm ở cấp orchestration

- RBAC tối thiểu quyền.
- Network policy giữa namespace/service.
- Secret encryption.
- Admission policy (kiểm soát image, security context).
- Quét cấu hình và audit định kỳ.

---

## 10. Lộ trình chuyển đổi thực tế

1. Dockerfile chuẩn hóa từng service.
2. Compose ổn định trên local/staging.
3. Đóng gói artifact + CI/CD chuẩn.
4. POC orchestration với 1-2 service.
5. Migrate dần theo domain, không big-bang.

---

## 11. Thực hành bắt buộc

1. Tạo cụm Swarm nhỏ (1 manager + 1 worker), deploy service replicas.
2. Thực hiện rolling update image và rollback.
3. Dựng cluster K8s local (kind/minikube), deploy app 3 replicas.
4. Thiết lập readiness/liveness probes.
5. Mô phỏng pod lỗi và quan sát self-healing.

---

## 12. Tiêu chí hoàn thành giai đoạn

- [ ] Giải thích được ngưỡng cần chuyển sang orchestration.
- [ ] Vận hành được service scale/rolling update ở Swarm.
- [ ] Deploy được ứng dụng cơ bản lên Kubernetes.
- [ ] Hiểu rõ Pod/Deployment/Service/Probe.
- [ ] Có tiêu chí chọn Swarm hoặc K8s theo yêu cầu hệ thống.
