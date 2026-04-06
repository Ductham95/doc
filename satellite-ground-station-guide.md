# Hướng Dẫn Toàn Diện: Hệ Thống Trạm Mặt Đất Giám Sát Vệ Tinh

> **Đối tượng:** Người mới bắt đầu, chưa có kiến thức về vệ tinh.
> **Mục tiêu:** Hiểu được vệ tinh hoạt động thế nào, trạm mặt đất làm gì, và ý nghĩa của từng thông số trong hệ thống.

---

## Mục lục

1. [Tổng Quan — Vệ Tinh và Trạm Mặt Đất Là Gì?](#1-tổng-quan)
2. [Các Thông Số Vị Trí & Quỹ Đạo](#2-các-thông-số-vị-trí--quỹ-đạo)
3. [Các Thông Số Tư Thế (Attitude)](#3-các-thông-số-tư-thế-attitude)
4. [Hệ Thống Năng Lượng (EPS)](#4-hệ-thống-năng-lượng-eps)
5. [Hệ Thống Thông Tin Liên Lạc (COMM)](#5-hệ-thống-thông-tin-liên-lạc-comm)
6. [Hệ Thống Nhiệt (Thermal)](#6-hệ-thống-nhiệt-thermal)
7. [Telemetry — Nhận Dữ Liệu Từ Vệ Tinh](#7-telemetry--nhận-dữ-liệu-từ-vệ-tinh)
8. [Telecommand — Gửi Lệnh Lên Vệ Tinh](#8-telecommand--gửi-lệnh-lên-vệ-tinh)
9. [Contact Session — Phiên Liên Lạc](#9-contact-session--phiên-liên-lạc)
10. [Packet Log — Nhật Ký Gói Tin](#10-packet-log--nhật-ký-gói-tin)
11. [Giao Thức CCSDS](#11-giao-thức-ccsds)
12. [Các Chức Năng Chính Của Trạm Mặt Đất](#12-các-chức-năng-chính-của-trạm-mặt-đất)
13. [Bảng Tổng Hợp Tất Cả Thông Số](#13-bảng-tổng-hợp-tất-cả-thông-số)
14. [Thuật Ngữ](#14-thuật-ngữ)

---

## 1. Tổng Quan

### Vệ tinh là gì?

Vệ tinh nhân tạo (satellite) là một thiết bị do con người chế tạo, được phóng lên không gian và bay quanh Trái Đất theo một quỹ đạo (orbit) nhất định. Hãy tưởng tượng vệ tinh như một **"máy tính bay"** ở trên cao — nó có pin, bộ xử lý, cảm biến, anten liên lạc, và các tấm pin mặt trời.

**Ví dụ thực tế dễ hiểu:**
- Khi bạn xem dự báo thời tiết → ảnh mây chụp từ **vệ tinh khí tượng**
- Khi bạn dùng Google Maps → vị trí xác định bởi **vệ tinh GPS**
- Khi bạn xem truyền hình cáp → tín hiệu từ **vệ tinh viễn thông**

### Trạm mặt đất (Ground Station) là gì?

Trạm mặt đất là **trung tâm điều khiển** đặt trên mặt đất, có nhiệm vụ:

```
    🛰️ Vệ tinh (bay trên quỹ đạo, cách mặt đất 350-680 km)
        │
        │  📡 Sóng radio (tín hiệu vô tuyến)
        │     ↕ Lên: gửi lệnh (Telecommand)
        │     ↕ Xuống: nhận dữ liệu (Telemetry)
        │
    📡 Trạm mặt đất (Ground Station)
        │
    🖥️ Phần mềm giám sát (chính là ứng dụng này!)
```

**Trạm mặt đất giống như "phòng khám cho vệ tinh"** — bạn theo dõi "sức khỏe" (pin, nhiệt độ, hướng bay...) và "ra y lệnh" (bật/tắt thiết bị, thay đổi chế độ...) cho vệ tinh từ xa.

### Phân loại quỹ đạo trong hệ thống này

| Viết tắt | Tên đầy đủ | Độ cao | Đặc điểm |
|----------|-----------|--------|-----------|
| **LEO** | Low Earth Orbit | 200–2000 km | Bay nhanh, quanh Trái Đất ~90 phút/vòng. Dùng cho quan sát, IoT |
| **SSO** | Sun-Synchronous Orbit | 600–800 km | Luôn đi qua cùng vị trí vào cùng giờ mỗi ngày. Dùng chụp ảnh Trái Đất |

---

## 2. Các Thông Số Vị Trí & Quỹ Đạo

Để biết vệ tinh đang ở đâu, bay nhanh bao nhiêu, và có thể "nhìn thấy" từ trạm mặt đất không, ta cần các thông số sau:

### 2.1 Latitude (Vĩ độ) — `orbit.latitude`

| | |
|---|---|
| **Đơn vị** | Độ (°) |
| **Phạm vi** | -90° đến +90° |
| **Ý nghĩa** | Vị trí Bắc-Nam của vệ tinh trên bề mặt Trái Đất |

**Giải thích đơn giản:** Hãy tưởng tượng Trái Đất là quả cam. Vĩ độ cho biết vệ tinh đang **ở trên hay dưới đường xích đạo**:
- `+90°` = Bắc Cực
- `0°` = Đường xích đạo
- `-90°` = Nam Cực
- Ví dụ: TP.HCM ở khoảng `+10.8°`, Hà Nội `+21°`

**Dùng để làm gì:**
- Hiển thị vệ tinh trên bản đồ
- Tính toán khi nào vệ tinh bay qua vùng phủ sóng của trạm

### 2.2 Longitude (Kinh độ) — `orbit.longitude`

| | |
|---|---|
| **Đơn vị** | Độ (°) |
| **Phạm vi** | -180° đến +180° |
| **Ý nghĩa** | Vị trí Đông-Tây của vệ tinh |

**Giải thích đơn giản:** Kinh độ cho biết vệ tinh **nằm bên trái hay phải đường kinh tuyến gốc** (chạy qua London, Anh):
- `+180°` = Phía Đông xa nhất
- `0°` = Kinh tuyến gốc (London)
- `-180°` = Phía Tây xa nhất
- Ví dụ: Việt Nam nằm ở khoảng `+105°` đến `+109°`

**Dùng để làm gì:**
- Kết hợp với Latitude → xác định chính xác "điểm chiếu" của vệ tinh trên mặt đất
- Vẽ **ground track** (vết chiếu quỹ đạo trên bản đồ)

### 2.3 Altitude (Độ cao) — `orbit.altitude`

| | |
|---|---|
| **Đơn vị** | km (kilômét) |
| **Phạm vi điển hình** | 350–680 km (trong hệ thống này) |
| **Ý nghĩa** | Khoảng cách từ vệ tinh đến bề mặt Trái Đất |

**Giải thích đơn giản:** Máy bay thương mại bay ở ~10 km. Vệ tinh LEO bay cao hơn gấp **35-70 lần** so với máy bay! Độ cao không cố định hoàn toàn — nó dao động nhẹ do lực hấp dẫn và môi trường không gian.

**Dùng để làm gì:**
- Tính vùng phủ sóng (vệ tinh càng cao → "nhìn" được diện tích mặt đất càng rộng)
- Xác định tuổi thọ quỹ đạo (quá thấp → bị lực cản khí quyển kéo xuống dần)
- Tính toán link budget (quỹ liên lạc)

### 2.4 Velocity (Vận tốc) — `orbit.velocity`

| | |
|---|---|
| **Đơn vị** | km/h |
| **Giá trị điển hình** | ~27.000–28.000 km/h |
| **Ý nghĩa** | Tốc độ di chuyển của vệ tinh trên quỹ đạo |

**Giải thích đơn giản:** Vệ tinh LEO bay với tốc độ khoảng **28.000 km/h** — nhanh gấp ~35 lần tốc độ âm thanh! Ở tốc độ này, vệ tinh bay quanh Trái Đất chỉ trong ~90 phút. Đây là lý do bạn chỉ có **cửa sổ liên lạc** ngắn (5-20 phút) mỗi khi vệ tinh bay qua trạm.

**Dùng để làm gì:**
- Giám sát sức khỏe quỹ đạo (vận tốc giảm bất thường = có vấn đề)
- Tính thời gian vệ tinh bay qua trạm

### 2.5 Azimuth (Góc phương vị) — `orbit.azimuth`

| | |
|---|---|
| **Đơn vị** | Độ (°) |
| **Phạm vi** | 0° đến 360° |
| **Ý nghĩa** | Hướng cần chĩa anten (tính từ hướng Bắc, theo chiều kim đồng hồ) |

**Giải thích đơn giản:** Hãy đứng giữa sân và muốn chỉ cho ai đó biết vệ tinh ở hướng nào:
- `0°` = hướng Bắc
- `90°` = hướng Đông
- `180°` = hướng Nam
- `270°` = hướng Tây

**Dùng để làm gì:**
- **Điều khiển anten quay theo hướng vệ tinh** — anten trạm mặt đất phải luôn chĩa đúng hướng vệ tinh
- Tự động tracking (bám bắt) vệ tinh

### 2.6 Elevation Angle (Góc ngẩng) — `orbit.elevation_angle`

| | |
|---|---|
| **Đơn vị** | Độ (°) |
| **Phạm vi** | -10° đến 90° |
| **Ý nghĩa** | Góc nhìn lên/xuống từ trạm mặt đất đến vệ tinh |

**Giải thích đơn giản:** Elevation Angle kết hợp với Azimuth giúp "lock" (bám) vệ tinh:
- `0°` = vệ tinh nằm ngay đường chân trời
- `90°` = vệ tinh ở ngay trên đỉnh đầu bạn
- Giá trị âm = vệ tinh ở phía dưới đường chân trời (không liên lạc được!)

> **Quy tắc:** Thông thường, liên lạc chỉ khả thi khi elevation > 5°–10°. Dưới ngưỡng đó, tín hiệu bị cản bởi địa hình, tòa nhà, hoặc tầng khí quyển dày.

**Dùng để làm gì:**
- Xác định vệ tinh có đang trong tầm liên lạc hay không
- Điều khiển anten ngẩng lên/hạ xuống bám theo vệ tinh

### 2.7 Các thông số quỹ đạo bổ sung

Những thông số này được cấu hình sẵn cho mỗi vệ tinh (không thay đổi theo thời gian thực):

| Thông số | Đơn vị | Ý nghĩa |
|----------|--------|---------|
| **Inclination** (Góc nghiêng) | ° | Góc giữa mặt phẳng quỹ đạo và mặt phẳng xích đạo. VD: 51.6° = vệ tinh có thể bay từ vĩ độ -51.6° đến +51.6° |
| **Period** (Chu kỳ) | phút | Thời gian bay hết 1 vòng quanh Trái Đất. LEO thường 90-98 phút |
| **NORAD ID** | số | Mã định danh quốc tế duy nhất, do NORAD (Bộ Tư lệnh Phòng thủ Không gian Bắc Mỹ) cấp |

---

## 3. Các Thông Số Tư Thế (Attitude)

### Tư thế vệ tinh là gì?

Tư thế (Attitude) là **hướng quay** của vệ tinh trong không gian — tức là anten nó đang chĩa về đâu, tấm pin mặt trời có hướng về Mặt Trời không, camera có chụp đúng mục tiêu không.

**Ví dụ thực tế:** Hãy tưởng tượng bạn đang cầm điện thoại chụp ảnh. Bạn có thể xoay điện thoại theo 3 cách — đó chính là 3 trục tư thế:

### 3.1 Roll (Cuộn) — `adcs.roll`

| | |
|---|---|
| **Đơn vị** | Độ (°) |
| **Giá trị lý tưởng** | Gần 0° |
| **Ý nghĩa** | Xoay quanh trục dọc (hướng bay) |

**Hình dung:** Như khi bạn **nghiêng đầu sang trái/phải**. Nếu vệ tinh roll quá lớn → anten lệch hướng → mất liên lạc.

### 3.2 Pitch (Ngáng) — `adcs.pitch`

| | |
|---|---|
| **Đơn vị** | Độ (°) |
| **Giá trị lý tưởng** | Gần 0° |
| **Ý nghĩa** | Xoay quanh trục ngang (vuông góc hướng bay) |

**Hình dung:** Như khi bạn **ngước lên nhìn trời hoặc cúi xuống nhìn đất**. Pitch kiểm soát camera/anten chĩa lên hay xuống.

### 3.3 Yaw (Hướng) — `adcs.yaw`

| | |
|---|---|
| **Đơn vị** | Độ (°) |
| **Giá trị lý tưởng** | Gần 0° |
| **Ý nghĩa** | Xoay quanh trục thẳng đứng |

**Hình dung:** Như khi bạn **quay người sang trái/phải**. Yaw quyết định "mũi" vệ tinh hướng về đâu.

### Hệ thống ADCS

**ADCS** (Attitude Determination and Control System) = "Hệ thống xác định và điều khiển tư thế". Đây là bộ phận "giữ thăng bằng" cho vệ tinh, gồm:
- **Cảm biến:** Sun sensor (cảm biến Mặt Trời), Star tracker (cảm biến sao), IMU (cảm biến quán tính)
- **Bộ chấp hành:** Reaction wheels (bánh xe quán tính), Magnetorquers (cuộn từ), Thrusters (động cơ đẩy)

> **Tại sao quan trọng?** Trong không gian không có trọng lực giữ vệ tinh ổn định. Nếu không có ADCS, vệ tinh sẽ **quay vô hướng** (tumble) và mất liên lạc hoàn toàn. Lệnh `ADCS_DETUMBLE` trong hệ thống chính là để "cứu" vệ tinh khỏi tình trạng này.

---

## 4. Hệ Thống Năng Lượng (EPS)

**EPS** (Electrical Power System) = "Hệ thống nguồn điện". Vệ tinh như một thiết bị điện tử — không có điện thì mọi thứ dừng lại.

### 4.1 Battery Voltage — `eps.battery_voltage`

| | |
|---|---|
| **Đơn vị** | V (Volt) |
| **Phạm vi bình thường** | 50–60 V |
| **Ý nghĩa** | Điện áp pin — cho biết pin "khỏe" hay "yếu" |

**Giải thích:** Giống như kiểm tra pin điện thoại bằng vôn kế. Điện áp giảm thấp → pin sắp hết → cần giảm tải hoặc chuyển sang chế độ tiết kiệm (SAFE mode).

### 4.2 Solar Panel Current — `eps.solar_current`

| | |
|---|---|
| **Đơn vị** | A (Ampere) |
| **Phạm vi bình thường** | 6–10 A |
| **Ý nghĩa** | Dòng điện từ tấm pin mặt trời |

**Giải thích:** Khi vệ tinh ở phía có ánh sáng Mặt Trời → pin mặt trời phát điện → dòng điện cao. Khi vệ tinh đi vào **vùng bóng tối** (eclipse) của Trái Đất → dòng = 0 → phải dùng pin dự trữ.

### 4.3 Total Power — `eps.power`

| | |
|---|---|
| **Đơn vị** | W (Watt) |
| **Phạm vi bình thường** | 50–65 W |
| **Ý nghĩa** | Tổng công suất tiêu thụ |

### 4.4 Battery SOC — `eps.battery_soc`

| | |
|---|---|
| **Đơn vị** | % |
| **Phạm vi** | 0–100% |
| **Ý nghĩa** | State of Charge — mức pin còn lại (giống % pin điện thoại) |

> **Cảnh báo:** SOC giảm dưới 20% → trạm mặt đất cần ra lệnh giảm tải (tắt payload, giảm tốc độ truyền dữ liệu).

---

## 5. Hệ Thống Thông Tin Liên Lạc (COMM)

### 5.1 Signal Strength — `comm.signal_strength`

| | |
|---|---|
| **Đơn vị** | dBm (decibel-milliwatt) |
| **Phạm vi điển hình** | -85 đến -70 dBm |
| **Ý nghĩa** | Cường độ tín hiệu nhận được từ vệ tinh |

**Giải thích:** dBm là đơn vị đo trên thang logarit, giá trị càng gần 0 thì tín hiệu càng mạnh:
- `-70 dBm` = tín hiệu tốt
- `-85 dBm` = tín hiệu yếu
- Dưới `-90 dBm` = có thể mất liên lạc

### 5.2 Uplink Rate — `comm.uplink`

| | |
|---|---|
| **Đơn vị** | kbps (kilobit/giây) |
| **Phạm vi** | 8–12 kbps |
| **Ý nghĩa** | Tốc độ truyền dữ liệu **từ mặt đất LÊN vệ tinh** |

**Dùng cho:** Gửi lệnh (telecommand), upload phần mềm mới.

### 5.3 Downlink Rate — `comm.downlink`

| | |
|---|---|
| **Đơn vị** | kbps |
| **Phạm vi** | 250–300 kbps |
| **Ý nghĩa** | Tốc độ truyền dữ liệu **từ vệ tinh XUỐNG mặt đất** |

**Dùng cho:** Nhận telemetry, tải ảnh chụp, dữ liệu khoa học. Downlink thường nhanh hơn uplink nhiều lần vì lượng dữ liệu cần tải xuống lớn hơn.

### 5.4 Link Margin — `comm.link_margin`

| | |
|---|---|
| **Đơn vị** | dB |
| **Phạm vi an toàn** | > 3 dB |
| **Ý nghĩa** | "Dự trữ" chất lượng liên lạc |

**Giải thích:** Link margin = cường độ tín hiệu thực tế − ngưỡng tối thiểu cần có. Nếu link margin > 3 dB → liên lạc ổn định. Dưới 3 dB → có nguy cơ mất gói tin.

### Băng tần liên lạc

| Băng tần | Tần số | Dùng cho |
|----------|--------|----------|
| **UHF** | 300–3000 MHz | Telemetry cơ bản, command |
| **VHF** | 30–300 MHz | Beacon, tín hiệu nhận dạng |
| **S-band** | 2–4 GHz | Truyền dữ liệu tốc độ cao |

---

## 6. Hệ Thống Nhiệt (Thermal)

### Temperature — `thermal.temperature`

| | |
|---|---|
| **Đơn vị** | °C |
| **Phạm vi bình thường** | 28–38 °C |
| **Ý nghĩa** | Nhiệt độ bên trong vệ tinh |

**Tại sao quan trọng?** Trong không gian:
- Phía hướng Mặt Trời: có thể lên tới **+120°C**
- Phía bóng tối: có thể xuống **-160°C**

Nếu linh kiện điện tử quá nóng hoặc quá lạnh → hỏng hóc. Hệ thống có lệnh `EPS_SET_HEATER` để bật/tắt bộ sưởi pin khi nhiệt độ quá thấp.

---

## 7. Telemetry — Nhận Dữ Liệu Từ Vệ Tinh

### Telemetry là gì?

**Telemetry (TM)** = dữ liệu mà vệ tinh tự động gửi xuống mặt đất, báo cáo "sức khỏe" của nó. Giống như bệnh nhân gắn máy monitor trong bệnh viện — liên tục phát dữ liệu nhịp tim, huyết áp... về trung tâm giám sát.

### Cách hoạt động trong hệ thống này

```
Vệ tinh (cảm biến đo liên tục)
    │
    │ gửi mỗi 1 giây qua sóng radio
    ▼
Backend Server (port 8081)
    │
    │ WebSocket broadcast
    ▼
OpenMCT Data Engine (xử lý, lưu cache)
    │
    │ React Hooks (useTelemetry)
    ▼
Giao diện người dùng (Dashboard, Charts, Gauges)
```

### Format dữ liệu Telemetry

Mỗi điểm dữ liệu (datum) có cấu trúc:

```json
{
    "id": "eps.battery_voltage",
    "timestamp": 1711900000000,
    "value": 55.23
}
```

| Trường | Ý nghĩa |
|--------|---------|
| `id` | Định danh thông số (VD: `eps.battery_voltage`) |
| `timestamp` | Thời điểm đo, tính bằng Unix milliseconds (UTC) |
| `value` | Giá trị đo được (số thực) |

### Phân loại Telemetry

| Loại | Cách lấy | Dùng cho |
|------|----------|----------|
| **Realtime** | WebSocket (cập nhật mỗi giây) | Dashboard live, cảnh báo tức thời |
| **Historical** | REST API (query theo time range) | Xem biểu đồ quá khứ, phân tích xu hướng |

---

## 8. Telecommand — Gửi Lệnh Lên Vệ Tinh

### Telecommand là gì?

**Telecommand (TC)** = lệnh điều khiển gửi từ mặt đất lên vệ tinh. Nếu telemetry là "nghe", thì telecommand là "nói".

### Quy trình gửi lệnh

```
1. Kỹ sư chọn lệnh từ Command Dictionary
2. Thiết lập tham số (nếu cần)
3. Review → Nhấn "Send" hoặc thêm vào Queue
4. Hệ thống đóng gói thành CCSDS packet
5. Gửi qua uplink lên vệ tinh
6. Vệ tinh nhận → giải mã → thực thi
7. Vệ tinh gửi ACK (xác nhận) về
8. Trạm mặt đất ghi nhận vào Command History
```

### Danh mục lệnh trong hệ thống

#### CI (Command Ingest) — Bộ tiếp nhận lệnh
| Lệnh | Ý nghĩa |
|-------|---------|
| `NOOP` | Lệnh rỗng — chỉ kiểm tra xem đường truyền có thông không |
| `ENABLE_TO` | Bật ứng dụng Telemetry Output |
| `RESET_CTRS` | Reset bộ đếm telemetry |

#### TO (Telemetry Output) — Bộ xuất telemetry
| Lệnh | Ý nghĩa |
|-------|---------|
| `ENABLE` | Bật truyền telemetry xuống trạm mặt đất |
| `DISABLE` | Tắt truyền telemetry |

#### SCH (Scheduler) — Bộ lập lịch
| Lệnh | Ý nghĩa |
|-------|---------|
| `ENABLE` | Bật thực thi lệnh theo lịch |
| `DISABLE` | Tắt bộ lập lịch |

#### EPS (Power System) — Hệ thống nguồn
| Lệnh | Nguy hiểm? | Ý nghĩa |
|-------|-----------|---------|
| `SET_MODE` | ⚠️ CÓ | Chuyển chế độ: SAFE (tiết kiệm), NOMINAL (bình thường), HIGH_POWER |
| `DEPLOY_PANEL` | ⚠️ CÓ | Mở tấm pin mặt trời (chỉ làm 1 lần sau phóng!) |
| `SET_HEATER` | Không | Bật/tắt bộ sưởi pin |

#### ADCS (Attitude Control) — Điều khiển tư thế
| Lệnh | Nguy hiểm? | Ý nghĩa |
|-------|-----------|---------|
| `SET_ATTITUDE` | ⚠️ CÓ | Đặt hướng quay mới cho vệ tinh (sử dụng quaternion) |
| `DETUMBLE` | ⚠️ CÓ | Kích hoạt chế độ ổn định khi vệ tinh đang quay mất kiểm soát |

> **Lệnh nguy hiểm (danger):** Các lệnh đánh dấu ⚠️ có thể gây hậu quả nghiêm trọng nếu sai. Ví dụ `DEPLOY_PANEL` — tấm pin mặt trời chỉ mở được **một lần** duy nhất trong vũ trụ, không thể hoàn tác.

---

## 9. Contact Session — Phiên Liên Lạc

### Contact Session là gì?

Vì vệ tinh LEO bay rất nhanh, nó **không phải lúc nào cũng nằm trong tầm nhìn** của trạm mặt đất. Mỗi lần vệ tinh bay qua và có thể liên lạc được gọi là một **Contact Session** (phiên liên lạc) hay **Pass**.

```
   Vệ tinh bay qua                    Vệ tinh bay qua tiếp
   ◄─────────────────►                ◄─────────────────►
   │  Contact Session │                │  Contact Session │
   │    5-20 phút     │                │    5-20 phút     │
───┼──────────────────┼────────────────┼──────────────────┼───→ thời gian
   AOS              LOS              AOS              LOS
                    (không liên lạc được ~70 phút)
```

| Thuật ngữ | Ý nghĩa |
|-----------|---------|
| **AOS** (Acquisition of Signal) | Thời điểm vệ tinh "mọc" trên đường chân trời, bắt đầu nhận tín hiệu |
| **LOS** (Loss of Signal) | Thời điểm vệ tinh "lặn" dưới đường chân trời, mất tín hiệu |
| **Max Elevation** | Góc ngẩng lớn nhất trong pass (elevation càng cao → liên lạc càng tốt) |

### Trạng thái Contact

| Trạng thái | Ý nghĩa |
|-----------|---------|
| `scheduled` | Đã lên lịch, chưa xảy ra |
| `active` | Đang diễn ra — vệ tinh đang trong tầm liên lạc |
| `completed` | Đã hoàn thành |

---

## 10. Packet Log — Nhật Ký Gói Tin

### Packet là gì?

Mọi dữ liệu trao đổi giữa vệ tinh và trạm mặt đất đều được đóng gói thành **packets** (gói tin) — giống như thư bỏ vào phong bì trước khi gửi.

Mỗi packet bao gồm:
- **Header** (phần đầu): chứa thông tin nhận dạng, độ dài, sequence number
- **Payload** (phần dữ liệu): chứa giá trị telemetry hoặc lệnh thực sự
- **Binary data**: dữ liệu thô dạng hex (VD: `0a 1f 3c 89...`)

### Các trường quan trọng của Packet

| Trường | Ý nghĩa |
|--------|---------|
| `packetName` | Tên/loại gói tin (VD: `/CCSDS/CCSDS_TM`) |
| `generationTime` | Thời điểm vệ tinh tạo ra packet |
| `receptionTime` | Thời điểm trạm mặt đất nhận được |
| `link` | Kênh nhận (`udp-in`, `tcp-in`, `debug-in`) |
| `size` | Kích thước gói tin (bytes) |
| `containers` | Cấu trúc chi tiết các trường dữ liệu bên trong |

---

## 11. Giao Thức CCSDS

### CCSDS là gì?

**CCSDS** (Consultative Committee for Space Data Systems) là tổ chức quốc tế định nghĩa **chuẩn truyền thông** cho ngành vũ trụ — giống như HTTP là chuẩn cho web.

Mọi gói tin trong hệ thống đều có **CCSDS header** với các trường:

| Trường | Loại | Ý nghĩa |
|--------|------|---------|
| `CCSDS_STREAMID` | uint16 | Mã nhận dạng gói tin (mỗi loại lệnh/telemetry có mã riêng) |
| `CCSDS_SEQUENCE` | uint16 | Số thứ tự gói tin (phát hiện mất gói) |
| `CCSDS_LENGTH` | uint16 | Độ dài phần dữ liệu (bytes) |
| `CCSDS_CHECKSUM` | uint8 | Mã kiểm tra lỗi |
| `CCSDS_FUNCCODE` | uint8 | Mã chức năng lệnh (phân biệt NOOP, ENABLE, DISABLE...) |

---

## 12. Các Chức Năng Chính Của Trạm Mặt Đất

### 12.1 Giám sát vệ tinh theo thời gian thực (Satellite Overview)

Hiển thị tất cả vệ tinh đang quản lý trên bản đồ thế giới, bao gồm:
- Vị trí hiện tại (lat, lon) cập nhật mỗi 2 giây
- Ground track (vệt quỹ đạo trên bản đồ)
- Trạng thái hoạt động
- Phiên liên lạc sắp tới

### 12.2 Dashboard giám sát sức khỏe

Hiển thị tất cả thông số quan trọng dưới dạng:
- **Biểu đồ đường** (Line chart): theo dõi xu hướng theo thời gian
- **Đồng hồ đo** (Gauge): giá trị hiện tại với ngưỡng cảnh báo
- **Bảng giá trị** (Value table): giá trị mới nhất

### 12.3 TM/TC Monitor (Giám sát Telemetry & Telecommand)

Giao diện linh hoạt cho phép người dùng tự tạo layout với các widget:
- Chart widget, Value widget, Alarm widget...
- Kéo thả, thay đổi kích thước

### 12.4 Command Sender (Gửi lệnh)

- Chọn target (CI, TO, EPS, ADCS...)
- Chọn lệnh từ Command Dictionary
- Thiết lập tham số
- Gửi trực tiếp hoặc thêm vào hàng đợi (queue)
- Xem lịch sử lệnh (thành công/thất bại)

### 12.5 Packet Logs (Nhật ký gói tin)

- Danh sách gói tin nhận được theo thời gian thực
- Lọc theo tên packet, link, thời gian
- Xem chi tiết: cấu trúc container, giá trị từng trường
- Xem hex dump (dữ liệu thô)

### 12.6 Orbit Visualization

Hiển thị quỹ đạo 3D và thông số quỹ đạo chi tiết.

---

## 13. Bảng Tổng Hợp Tất Cả Thông Số

| Hệ thống | Key | Tên | Đơn vị | Mô tả ngắn |
|----------|-----|-----|--------|-------------|
| **Orbit** | `orbit.latitude` | Latitude | ° | Vĩ độ (Bắc-Nam) |
| | `orbit.longitude` | Longitude | ° | Kinh độ (Đông-Tây) |
| | `orbit.altitude` | Altitude | km | Độ cao so với mặt đất |
| | `orbit.velocity` | Velocity | km/h | Tốc độ bay |
| | `orbit.azimuth` | Azimuth | ° | Hướng chĩa anten (0°=Bắc) |
| | `orbit.elevation_angle` | Elevation | ° | Góc ngẩng nhìn vệ tinh |
| **ADCS** | `adcs.roll` | Roll | ° | Xoay trục dọc |
| | `adcs.pitch` | Pitch | ° | Xoay trục ngang |
| | `adcs.yaw` | Yaw | ° | Xoay trục đứng |
| **EPS** | `eps.battery_voltage` | Battery Voltage | V | Điện áp pin |
| | `eps.solar_current` | Solar Current | A | Dòng điện pin mặt trời |
| | `eps.power` | Total Power | W | Tổng công suất tiêu thụ |
| | `eps.battery_soc` | Battery SOC | % | Mức pin còn lại |
| **COMM** | `comm.signal_strength` | Signal Strength | dBm | Cường độ tín hiệu |
| | `comm.uplink` | Uplink Rate | kbps | Tốc độ truyền lên |
| | `comm.downlink` | Downlink Rate | kbps | Tốc độ truyền xuống |
| | `comm.link_margin` | Link Margin | dB | Dự trữ chất lượng liên lạc |
| **Thermal** | `thermal.temperature` | Temperature | °C | Nhiệt độ bên trong vệ tinh |

---

## 14. Thuật Ngữ

| Thuật ngữ | Viết tắt | Giải thích |
|-----------|----------|------------|
| Telemetry | TM | Dữ liệu vệ tinh gửi xuống mặt đất |
| Telecommand | TC | Lệnh điều khiển gửi từ mặt đất lên vệ tinh |
| Ground Station | GS | Trạm mặt đất |
| LEO | — | Quỹ đạo Trái Đất thấp (200-2000 km) |
| SSO | — | Quỹ đạo đồng bộ Mặt Trời |
| ADCS | — | Hệ thống xác định và điều khiển tư thế |
| EPS | — | Hệ thống nguồn điện |
| COMM | — | Hệ thống thông tin liên lạc |
| CCSDS | — | Chuẩn truyền thông ngành vũ trụ |
| AOS | — | Thời điểm bắt đầu nhận tín hiệu |
| LOS | — | Thời điểm mất tín hiệu |
| Pass | — | Một lần vệ tinh bay qua trạm mặt đất |
| Downlink | — | Đường truyền từ vệ tinh xuống mặt đất |
| Uplink | — | Đường truyền từ mặt đất lên vệ tinh |
| Link Budget | — | Tính toán quỹ công suất liên lạc |
| Beacon | — | Tín hiệu nhận dạng liên tục từ vệ tinh |
| Eclipse | — | Khoảng thời gian vệ tinh nằm trong bóng Trái Đất |
| Quaternion | — | Cách biểu diễn toán học cho hướng quay 3D |
| Detumble | — | Ổn định vệ tinh khi đang quay mất kiểm soát |
| Ground Track | — | Vệt quỹ đạo chiếu xuống bản đồ Trái Đất |
| Packet | — | Gói tin dữ liệu đã đóng gói theo chuẩn |
| Hex Dump | — | Hiển thị dữ liệu thô dạng hệ thập lục phân |

---

> **Tài liệu này được viết cho dự án Satellite Ground Station.**
> Cập nhật lần cuối: Tháng 4, 2026.
