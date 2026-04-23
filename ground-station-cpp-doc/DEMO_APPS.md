# Các ứng dụng Demo (Sandbox & Testing Application)

Để giúp cho các nhà phát triển mới làm quen với kiến trúc Ground Station C++, thư mục `002.Code/app/iml` cung cấp sẵn một chuỗi các ứng dụng mô phỏng (Demos) cô lập, giúp dễ dàng kiểm thử các chức năng độc lập mà không cần phải chạy toàn bộ khối lượng hệ thống TM/TC. Cụ thể có các Demo sau thường được thiết lập rẽ nhánh từ hàm khởi chạy `main.cpp` trong thư mục `app`:

## 1. Connection Demo (`connection_demo.hpp`)
Mô phỏng cách khởi tạo một kết nối mạng căn bản, chủ yếu là khả năng sử dụng giao thức UDP:
- Gửi gói tin qua cấu trúc `UdpService`.
- Lắng nghe mảng bytes tới, thiết lập callback xử lý sự kiện khi có network message (ví dụ hàm callback `on_receive`).
- Kiểm tra tính ổn định của vòng lặp Asio IO Context kết hợp `ServiceBase`.

## 2. Bus Bridge Demo (`bus_bridge_demo.hpp`)
Trình diễn tính toán xử lý cầu nối giữa cơ chế nội vi Event Bus:
- Subscribe một loại event Type vào `Bus::instance()`.
- Dispatch liên hoàn (publish) messages vào Topic và xem các callback thread worker hứng event bất đồng bộ, khẳng định tính toàn vẹn (Memory safe) trên môi trường đa luồng.

## 3. gRPC Demos (`grpc_demo.hpp` & `app_data_demo.hpp`)
Việc gọi từ xa rất quan trọng cho việc Dashboard hoặc Mission Control Center ra lệnh.
- **`grpc_demo.hpp`**: Khởi phát cục bộ một `GrpcServer` đính kèm một service mẫu (`GrpcDemoService`) để thấy cách giao tiếp 2 chiều bằng Protobuf service diễn ra.
- **`app_data_demo.hpp`**: Một biến thể Demo nâng cao làm việc chủ yếu về App Data lấy từ hệ thống trả về phía Requestor.

## 4. Legacy Demo (`legacy_demo.hpp`)
Trình diễn các bài test ban đầu của kiến trúc Ground Station cũ:
- Viết thử nghiệm Logger output.
- Khởi tạo Timer service (`gs_timer` timer timeout callbacks định kỳ thực hiện công việc giống như heartbeat timer).
- Ping pong thông điệp.

---

> **Tip**: Quản trị bằng tính năng 주석 comment out trong `app/main.cpp` để chọn phương án Demo nào muốn chạy build test trong lúc lập trình gỡ lỗi kiến trúc hạ tầng tầng dưới.
