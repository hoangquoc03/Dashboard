# Prometheus va Grafana

Project mẫu này chạy Prometheus và Grafana bằng Docker Compose. Grafana tự động nạp Prometheus làm data source mặc định qua provisioning.

## Khởi chạy

Từ thư mục chứa `docker-compose.yml`, chạy:

```powershell
docker compose up -d
docker compose ps
```

Mở [http://localhost:3000](http://localhost:3000) và đăng nhập bằng `admin` / `admin`. Hãy đổi mật khẩu nếu Grafana yêu cầu.

Thông tin `admin/admin` chỉ dành cho học tập và local development. Không dùng nguyên trạng trên production.

## Kiểm tra Prometheus data source

Mở **Connections → Data sources** (hoặc **Data sources**) trong Grafana. Data source **Prometheus** sẽ xuất hiện tự động nhờ file `grafana/provisioning/datasources/prometheus.yml`; URL của nó là `http://prometheus:9090` và nó được đặt làm data source mặc định.

Nếu cần kiểm tra thủ công một data source trong giao diện, xác nhận URL `http://prometheus:9090` rồi chọn **Save & test** và kiểm tra thông báo kết nối thành công. Với data source đã provisioning, hãy mở data source **Prometheus** để kiểm tra URL/trạng thái kết nối; cấu hình được provision từ file trong Compose.

## Vì sao không dùng localhost trong URL data source?

Không đặt URL data source thành `http://localhost:9090` khi Grafana chạy trong container riêng. `localhost` bên trong container Grafana trỏ về chính container Grafana, không phải container Prometheus. Docker Compose cung cấp DNS nội bộ để các service cùng network gọi nhau bằng tên service; vì service Prometheus ở đây tên là `prometheus`, URL đúng là `http://prometheus:9090`.

Port `9090` của Prometheus không được publish ra máy host trong cấu hình này; Grafana vẫn truy cập được qua Docker network. Việc publish port không bắt buộc để hai service giao tiếp, đồng thời tránh xung đột với dịch vụ khác đang dùng `9090`.

## Dừng các service

```powershell
docker compose down
```

Lệnh trên không xóa named volumes, vì vậy dữ liệu Prometheus và Grafana được giữ lại. Không dùng `docker compose down -v` nếu muốn giữ dữ liệu.

## VPS Health Dashboard

Compose chạy thêm `node-exporter` và Prometheus scrape metric tại `node-exporter:9100` trên mạng nội bộ. Grafana tự động provision dashboard **VPS Health Dashboard** từ `grafana/dashboards/vps-health.json`; file provider nằm tại `grafana/provisioning/dashboards/dashboards.yml`.

Sau khi cập nhật/chạy stack, xác nhận container bằng `docker compose ps`. Trong Prometheus, kiểm tra trang **Status → Targets**: target `node-exporter` phải ở trạng thái **UP**. Trong Grafana, mở **Dashboards → VPS Health Dashboard** (hoặc `http://localhost:3000/d/vps-health/vps-health-dashboard`) và xác nhận hai panel `CPU Usage (%)` và `Free Memory` có dữ liệu. Dashboard mặc định hiển thị **Last 1 hour** và tự refresh mỗi **30s**.

Panel CPU dùng truy vấn:

```promql
100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
```

`rate(...[5m])` tính tốc độ CPU idle trong 5 phút; `avg by (instance)` lấy trung bình trên các core của từng máy; `1 - idle` là tỷ lệ bận và nhân `100` để thành phần trăm. Giữ label `instance` giúp phân biệt nhiều máy.

Panel RAM dùng truy vấn:

```promql
node_memory_MemFree_bytes
```

Metric vẫn ở Bytes; panel dùng Unit `bytes` (Bytes IEC), để Grafana tự định dạng thành KiB/MiB/GiB. Không chia metric trong PromQL chỉ để đổi đơn vị; `MiB` là đơn vị nhị phân, khác `MB` hệ thập phân. Mỗi instance được giữ riêng, không cộng RAM của nhiều VPS.

> Node Exporter được cấu hình với host PID và bind mount `/proc`, `/sys`, `/` theo mô hình Linux host. Trên Docker Desktop for Windows, các metric này có thể phản ánh Linux VM của Docker thay vì Windows host; cấu hình không phải bộ thu thập Windows Exporter.
