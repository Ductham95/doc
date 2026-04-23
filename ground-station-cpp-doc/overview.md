# Tổng quan Repository Ground Station C++

## 1. Mục đích và Giới thiệu
Repository `Ground_Station_cpp` là một hệ thống backend được phát triển bằng ngôn ngữ C++ để làm phần điều khiển và xử lý cho Trạm mặt đất (Ground Station). Hệ thống này chủ yếu phụ trách xử lý Dữ liệu Viễn trắc (Telemetry - TM) và Phát lệnh (Telecommand - TC) theo thời gian thực. Hệ thống sử dụng Event Bus cho giao tiếp nội bộ và mở rộng giao tiếp ra ngoài bằng gRPC và RabbitMQ.

## 2. Cấu trúc thư mục (Codebase Structure)

- **`001.Docs/`**: Thư mục chứa tài liệu định hướng và đánh giá kiến trúc phần mềm, trong đó có File [ARCH_REVIEW_REPORT.md](file:///d:/work/Ground_Station_cpp/001.Docs/ARCH_REVIEW_REPORT.md) chứa kết quả đánh giá toàn diện module `arch/` (chỉ ra các data race, điểm rủi ro liên quan đến threading/Object pooling, và cả những điểm mạnh mà đội dự án đã xây dựng).
- **`002.Code/`**: Thư mục chứa source code chính, cấu trúc thành các component phân rã hợp lý:
  - **`arch/`**: Core architecture framework (Tầng thư viện cơ sở phục vụ cho toàn bộ repo).
    - `helper/`: Các công cụ hỗ trợ như hệ thống log bất đồng bộ bằng `spdlog`, crash assert bắt lỗi bằng `backward-cpp`, và bộ trợ giúp `timer` wrapper.
    - `messaging/`: Tập trung vào Message Queue (cụ thể là `RabbitMQClient` sử dụng `AMQP-CPP`) và gói dữ liệu Payload dựa trên `Protobuf`.
    - `service/`: Triển khai Event Bus theo mô hình Pub/Sub và Rpc request nội bộ. Điểm nhấn là kiến trúc Worker-mỗi-Service để xử lý song song, kết hợp `SharedMessagePool` tối ưu hóa độ trễ sinh từ việc cấp phát bộ nhớ liên tục.
  - **`app_tmtc/`**: Ứng dụng nghiệp vụ chính về TM/TC. Ở đây dựng một `gRPC Server` (thông qua `TMTCProcessCommandService`) để nhận các RPC Commands từ ngoài vào, rồi chuyển giao xuống cho Bus thông qua `TMTCProcessMessageService` xử lý. Ứng dụng cũng đã được tích hợp signal termination xử lý việc huỷ dịch vụ một cách graceful.
  - **`app/`**: Tập hợp các Demo/Test Runner để trình diễn cách sử dụng các feature của Core framework như Bus Bridge demo, Connection demo, Legacy timer...
  - **`config/`**: Nơi cấu hình các địa chỉ Host và Port (ví dụ gRPC config constants).
  - **`cmake/`** & gốc dự án chứa [CMakeLists.txt](file:///d:/work/Ground_Station_cpp/002.Code/CMakeLists.txt): Cấu hình hệ thống build CMake để combine tất cả modules lại.
  - **`tests/`**: Các unit-tests để chạy kiểm định chất lượng codebase.

## 3. Kiến trúc Hệ thống nổi bật
1. **Thiết kế Layered (Phân tầng rõ ràng)**: Đi từ `helper` -> `service` (Broker nội bộ) -> `messaging` (Giao tiếp bên ngoài) -> Tới module nghiệp vụ `app`.
2. **Event Bus (Pub/Sub) và ServiceWorker Model**: Các service kế thừa từ mẫu `ServiceBase`. Mỗi một service chạy một Worker Thread độc lập. Giữa các worker giao tiếp thông qua Bus singleton dạng Publish-Subscribe, hoặc RPC nội bộ.
3. **Memory Optimization**: Sử dụng CRTP `PooledMessage` và `SharedMessagePool` object pool cho hệ thống cấp phát Message, giúp giữ latency ổn định cho hệ thống xử lý real-time.
4. **Interoperability (Khả năng tương tác mạng)**: Việc giao tiếp nhận lệnh từ Client (như các Web Dashboard) dựa trên giao thức C/S hiệu năng cao là gRPC. Quá trình phát tán dữ liệu TM thì thông qua RabbitMQ.

## 4. Công nghệ & Dependencies
Dự án được xây dựng trên hệ thống build **CMake** với một vài open-source libraries phổ biến cho backend C++ hiệu suất cao:
- **Boost** (ASIO / System)
- **gRPC** & **Protocol Buffers**
- **AMQP-CPP** (RabbitMQ Client)
- **Spdlog** (Thread-safe logger) và **Backward-cpp** (Stack trace extractor).
