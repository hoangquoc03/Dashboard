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
