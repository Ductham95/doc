# Messaging and Communication Architecture

Kiến trúc luân chuyển dòng dữ liệu (Dataflow) tại Ground Station `C++` đóng vai trò quan trọng trong việc giữ đảm bảo tính chất thời gian thực và bất đồng bộ khi khối lượng bản tin cực kỳ lớn. Hệ thống kết hợp ba cơ chế giao tiếp chính:

## 1. Internal Event Bus (Pub/Sub Singleton)
Được đặt tại `002.Code/arch/service/bus.hpp`, `Bus` hoạt động với tư cách là Broker trung tâm trong phạm vi tiến trình (In-process).
- **Topic-based Routing**: Việc định tuyến route một message được xác định bằng một khoá đôi `(type_index, topic)`. Khi có Service publish, Dispatcher sẽ map kiểu Type object cộng chuỗi Topic để đẩy callback về hàm lắng nghe.
- **Worker Dispatcher Loop**: Mỗi bus chạy ngầm một thread gọi là `dispatcher_loop`. Thay vì khoá mutex liên tục để gọi callback, Queue sẽ đẩy tin vào một Envelope và đợi dispatcher xử lý `pump()` bất đồng bộ, qua đó phân phối ngược lại các Service worker.
- **Collision Protection**: Thuật toán xử lý Hash table kết hợp thư viện `Boost hash_combine` loại trừ tối đa xác suất xung đột con trỏ của event registry (Collision) khi hàng trăm Topics cùng đăng ký.

## 2. RabbitMQ Client
Thiết kế tập trung tại `arch/messaging/rabbitmq`, cho phép Ground Station publish luồng TM giải mã theo giao thức AMQP ra ngoài.
- **AMQP-CPP**: Project sử dụng thư viện network AMQP-CPP được wrapped lại kết hợp với Boost Asio `io_context` (Cung cấp Non-blocking I/O).
- **Auto Reconnect**: Implement kỹ thuật `scheduleReconnect()` nếu rớt mạng đột ngột giữa chừng.
- **Builder Pattern Config**: Module này sử dụng builder pattern `RabbitMQConfigBuilder` tạo sự thuận tiện khi gán các tuỳ chọn (Exchange, RoutingKey, Host, Pass...).

## 3. Quản lý Payload & Protobuf
- Module `arch/messaging/protobuf/proto_payload_message.hpp` wrapper các payload tuỳ biến sinh ra từ mã dịch `Protobuf`, đóng gói thành dạng `MessageBase` để có thể thoải mái luân chuyển trên `Event Bus`.
- Cấu trúc protobuf nằm ở `tm_tc_service.proto` cho phép gRPC định hình rõ MDB Service, Control Request.

## 4. gRPC Communication
Cài đặt tại `arch/messaging/grpc/`. Framework giao diện RPC Remote của Google:
- Ground Station dựng Host qua `GrpcServer::start()` và binding Server Builder.
- Các Service (như `TMTCProcessCommandService`) kế thừa các Stub callbacks của GRPC đồng thời kế thừa `GrpcServiceBase` để quản lý life cycle một cách đồng bộ cùng app C++. 
