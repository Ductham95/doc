# Application TM/TC (Telemetry & Telecommand)

`app_tmtc` là module ứng dụng trung tâm chịu trách nhiệm xử lý các luồng dữ liệu Viễn trắc (Telemetry - TM) và điều khiển trạm phát Lệnh (Telecommand - TC). Kiến trúc của nó được thiết kế tuân theo mô hình **Event-driven** và Micro-service thu nhỏ bằng việc ứng dụng Event Bus nội bộ.

## 1. Thành phần chính trong TM/TC
Ứng dụng TM/TC được phân giải qua 2 Service chủ đạo đan xen tương tác với nhau và với thế giới bên ngoài.
### A. TMTCProcessCommandService
Service này chịu trách nhiệm phơi bày giao thức (expose interface) cho các Client bên ngoài hệ thống. Được kế thừa từ `GrpcServiceBase` và triển khai API gRPC (Protobuf gRPC Service). Nhờ đó, Client (ví dụ Web Dashboard) có thể triệu gọi các Remote Procedure Call (RPC) đến Ground Station một cách bất đồng bộ với độ trễ thấp.
- **PushCommand**: Xử lý yêu cầu gửi lệnh kích hoạt thiết bị từ xa. Forward lệnh này qua Event Bus tới `TMTCProcessMessageService`.
- **UpdateMdb**: Chức năng cho phép cập nhật cấu trúc cơ sở dữ liệu xử lý (Dynamic MDB Config) nóng ngay trong thời gian chạy (Runtime Dynamic Schema).

### B. TMTCProcessMessageService
Service xương sống của tiến trình điều khiển. Kế thừa `ServiceBase` và chạy ngầm một Worker Loop Thread độc lập.
Thực hiện các công việc sau:
1. Giao tiếp UDP với các hệ thống tầng dưới chuẩn (`UdpService`).
2. Nhúng `TmTcEngine` (thư viện xử lý logic giải mã/mã hoá core).
3. Sử dụng `RabbitMQClient` (`AMQP-CPP`) để push bản tin giải mã Telemetry lên hệ thống Data Pipeline ngoài để lưu trữ/ hiển thị.
4. Quản trị vòng đời Heartbeat và Status, cập nhật Sequence counter theo Timer định kỳ.
5. Mutex Lock bảo vệ `mdb_mutex_` chống các cạnh tranh dữ liệu trên tài nguyên cấu hình MDB được load tự động.

## 2. Tiêu chuẩn khởi động hệ thống (Graceful Shutdown)
Tại `main.cpp` của `app_tmtc`, ứng dụng cung cấp logic xử lý vòng đời một cách rõ ràng:
1. Signal Interrupts `SIGINT` (Ctrl+C) và `SIGTERM` được đăng ký và uỷ quyền cho một Condition Variable `shutdown_cv`.
2. Trình khơi chạy tuần tự `message_service -> command_service -> gRPC Server`.
3. Khi nhận tín hiệu Terminate, block thread sẽ mở, tuần tự `stop()` gRPC host và `stop()` các services đi kèm để giải phóng triệt để các kết nối mạng và bộ nhớ tài nguyên mà không làm sập ứng dụng cưỡng chế.
