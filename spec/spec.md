# Product Specification — Smart Couple Coin

> **Phiên bản**: 1.1\
> **Ngày tạo**: 2026-06-11\
> **Cập nhật**: 2026-06-12\
> **Tác giả**: hado\
> **Trạng thái**: Draft → Đã chốt Phase 1

---

## 1. Tổng quan sản phẩm

### 1.1 Mô tả

Smart Couple Coin là **cặp móc khóa đôi tình yêu** dành cho các cặp đôi ở xa nhau. Mỗi cặp gồm đúng **2 thiết bị** kết nối qua Internet. Khi một người bấm nút trên thiết bị của mình, thiết bị còn lại sẽ **nháy LED** và **rung motor** để thông báo — tạo cảm giác kết nối dù ở khoảng cách xa.

### 1.2 Đối tượng người dùng

- Các cặp đôi yêu xa (long-distance relationship).
- Quà tặng Valentine, kỷ niệm, sinh nhật.

### 1.3 Hình thức sản phẩm

- **Dạng**: Vật trang trí nhỏ, có dây đeo móc khóa.
- **Kích thước**: 50 × 35 × 18mm (chiều cao có thể thay đổi).
- **Vỏ**: In 3D.
- **Bảo vệ**: Chống bụi (IP5X tối thiểu).

### 1.4 Mục tiêu giai đoạn

| Giai đoạn | Mô tả |
|-----------|-------|
| **Prototype** | 1-2 cặp, hàn tay, kiểm tra chức năng đầy đủ |
| **Chi phí mục tiêu** | ≤ 500.000 VNĐ / cặp (2 thiết bị) |

---

## 2. Yêu cầu chức năng (Functional Requirements)

### FR-01: Gửi tín hiệu (Send Signal)

| Thuộc tính | Giá trị |
|------------|---------|
| **Trigger** | Nhấn nút bấm chính (1 lần nhấn ngắn) |
| **Loại tín hiệu** | Duy nhất 1 loại (không phân biệt pattern) |
| **Chống spam** | Chỉ cho phép gửi **1 tín hiệu** trong khoảng **1 phút** (chốt cứng, cấu hình trong firmware) |
| **Bấm giữ 2-3s** | Được tính như 1 lần bấm duy nhất (không khác bấm ngắn) |
| **Bấm giữ 10s** | Factory Reset (xem 5.3.8) |
| **Bấm giữ 20s** | Vào chế độ OTA Update (xem FR-06) |
| **Chống bấm nhầm** | Có — firmware phải debounce và phát hiện bấm trong túi (ví dụ: yêu cầu bấm rõ ràng, kết hợp debounce phần cứng C8 100nF) |

### FR-02: Nhận tín hiệu (Receive Signal)

| Thuộc tính | Giá trị |
|------------|---------|
| **LED** | 2 LED RGB (SK6812MINI-012, **3 kênh RGB**, protocol `NEO_GRB`) nháy trong **3 giây** |
| **Motor rung** | Rung trong **1 giây**, pattern: chạy 80% công suất mỗi 400ms, nghỉ 100ms |
| **Bắt buộc** | Cả LED và motor đều PHẢI hoạt động đồng thời khi nhận tín hiệu |
| **Chế độ im lặng** | Không hỗ trợ (luôn có cả LED + rung) |

### FR-03: Kết nối mạng (Network Connectivity)

| Thuộc tính | Giá trị |
|------------|---------|
| **Giao thức chính** | WiFi → MQTT broker (HiveMQ Cloud) |
| **Giao thức phụ (tương lai)** | BLE qua app điện thoại |
| **Mô hình kết nối** | **Polling** — thiết bị deep sleep, thức dậy định kỳ để kiểm tra tín hiệu |
| **Độ trễ tối đa** | **≤ 5 phút** (ban ngày 7h-23h) hoặc **≤ 30 phút** (ban đêm 23h-7h) |
| **Độ trễ khi bấm nút** | < 3 giây (thời gian wake + kết nối WiFi + MQTT publish) |
| **Mất mạng** | Báo lỗi bằng LED (pattern riêng, ví dụ: nháy đỏ chậm) |
| **Server** | HiveMQ Cloud (dịch vụ MQTT có sẵn) ban đầu, sau này có thể chuyển server riêng |

### FR-04: Cấu hình thiết bị (Device Configuration)

| Thuộc tính | Giá trị |
|------------|---------|
| **Pairing** | Cấu hình sẵn trong firmware (mỗi cặp dùng chung 1 MQTT topic duy nhất) |
| **Cấu hình WiFi** | Thiết bị phát AP WiFi → người dùng kết nối và truy cập trang web cấu hình (Captive Portal) |
| **Chọn màu LED** | Trên trang web cấu hình WiFi (Captive Portal), mỗi người chọn màu LED riêng |
| **Reset pairing** | Bấm giữ nút **10 giây** → reset về trạng thái mặc định |

### FR-05: Quản lý nguồn (Power Management)

| Thuộc tính | Giá trị |
|------------|---------|
| **Bật/tắt** | Công tắc cơ học bên ngoài (điều khiển LDO enable) |
| **Cắm USB khi OFF** | Thiết bị **KHÔNG tự bật** — vẫn phải gạt công tắc ON. USB chỉ cấp nguồn sạc qua LTC4054 |
| **Sạc** | USB Type-C, dòng sạc ~140-150mA (R_prog = 6.8kΩ) |
| **Sử dụng khi sạc** | Có — thiết bị hoạt động bình thường khi đang sạc |
| **LED báo sạc** | LED SMD riêng (CHG_LED), sáng khi đang sạc, tắt khi đầy. Nhìn thấy được qua vỏ |
| **Pin yếu** | LED nháy đỏ, motor không rung. Ngưỡng cảnh báo: ~3.3V (firmware đọc ADC) |
| **Cut-off** | Firmware tắt thiết bị (disable LDO hoặc deep sleep) khi VBAT ≤ 3.0V để bảo vệ pin |

### FR-06: Cập nhật firmware OTA

| Thuộc tính | Giá trị |
|------------|---------|
| **OTA update** | Có — hỗ trợ cập nhật firmware qua WiFi sau khi giao sản phẩm |
| **Trigger OTA** | **Bấm giữ nút 20 giây** → thiết bị vào chế độ OTA, kết nối WiFi và chờ firmware mới |
| **Nạp ban đầu** | Qua cổng USB-C trên ESP32-C3 SuperMini |
| **Log/debug sau lắp vỏ** | Không cần |
| **Bảo mật** | Không yêu cầu mã hóa/xác thực thiết bị (phase 1) |
| **Firmware versioning** | Semantic versioning (ví dụ: 1.0.0). Hiển thị trên Captive Portal |

---

## 3. Yêu cầu phi chức năng (Non-Functional Requirements)

### NFR-01: Thời lượng pin

| Thuộc tính | Giá trị |
|------------|---------|
| **Dung lượng pin** | 220mAh Li-Po, 1 cell, 3.7V (chốt cứng) |
| **Mục tiêu** | ≥ 5 ngày |
| **Tần suất sử dụng** | ~15 lần gửi/nhận tín hiệu mỗi ngày |
| **Chế độ tiết kiệm pin** | Deep sleep khi idle, wake theo timer định kỳ, LED switch tắt riêng từng LED |

#### Lịch thức dậy (Wake Schedule)

| Khung giờ | Interval wake | Số lần wake/khung | Ghi chú |
|-----------|---------------|-------------------|----------|
| 07:00 – 23:00 (16h) | Mỗi **5 phút** | ~192 lần | Ban ngày, polling thường xuyên |
| 23:00 – 07:00 (8h) | Mỗi **30 phút** | ~16 lần | Ban đêm, tiết kiệm pin tối đa |
| **Tổng/ngày** | | **~208 lần** | + thêm wake khi bấm nút |

#### Ước tính pin (Power Budget)

| Trạng thái | Dòng tiêu thụ | Thời gian/ngày | Năng lượng/ngày |
|------------|--------------|----------------|-----------------|
| Deep Sleep (ESP32-C3) | ~5 µA | ~23.5 h | ~0.12 mAh |
| Wake polling (WiFi connect + MQTT check) | ~120 mA | 208 lần × ~3s = ~624s | **~20.8 mAh** |
| Bấm nút gửi (WiFi + publish + check) | ~120 mA | 15 lần × ~3s = ~45s | ~1.5 mAh |
| LED nháy nhận tín hiệu (2 LED, 50%) | ~20 mA | 15 × 3s = 45s | ~0.25 mAh |
| Motor rung (80% PWM) | ~60 mA | 15 × 1s = 15s | ~0.25 mAh |
| **Tổng/ngày** | | | **~22.9 mAh** |
| **Thời lượng lý thuyết** | | | **~9.6 ngày** |

> **Ghi chú**: Ước tính lý thuyết. Yếu tố ảnh hưởng thực tế: thời gian kết nối WiFi (1-5s tùy router), số lần bấm nút, dòng rò mạch. Kỳ vọng thực tế: **5-8 ngày**. Nếu WiFi connect nhanh (~1.5s), có thể đạt >10 ngày.

### NFR-02: Kích thước và cơ khí

| Thuộc tính | Giá trị |
|------------|---------|
| **Kích thước vỏ** | 50 × 35 × 18mm (chiều cao có thể điều chỉnh) |
| **PCB** | 44 × 28mm, 2 lớp |
| **Vỏ** | In 3D (FDM/SLA) |
| **USB-C** | Lộ ra ngoài, có nắp đậy silicon/nhựa |
| **LED** | Mặt top PCB, cần diffuser (nhựa trong/mờ) trên vỏ |
| **Dây đeo** | Có lỗ xỏ dây móc khóa |
| **Chống bụi** | Thiết kế vỏ kín, khe hở tối thiểu |

### NFR-03: Độ tin cậy

| Thuộc tính | Giá trị |
|------------|---------|
| **Watchdog Timer** | Firmware PHẢI có WDT, timeout **15 giây** (chốt). Tự reset khi bị treo |
| **Auto-reconnect** | Tự kết nối lại WiFi/MQTT khi mất kết nối |
| **Thời gian khởi động** | < 5 giây từ bật công tắc đến sẵn sàng |

---

## 4. Đặc tả phần cứng (Hardware Specification)

### 4.1 Sơ đồ khối

```
┌─────────────────────────────────────────────────────────┐
│                    Smart Couple Coin                     │
│                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐   │
│  │ USB-C    │───▶│ LTC4054  │───▶│ Li-Po 220mAh     │   │
│  │ + TVS    │    │ Charger  │    │ (VBAT 3.0-4.2V)  │   │
│  └──────────┘    └──────────┘    └────────┬──────────┘   │
│                                           │              │
│                        ┌──────────────────┤              │
│                        │ SW (công tắc)    │              │
│                        ▼                  │              │
│                  ┌───────────┐             │              │
│                  │ AP2112K   │             │              │
│                  │ LDO 3V3   │             │              │
│                  └─────┬─────┘             │              │
│                        │ 3V3              │ VBAT          │
│                        ▼                  │              │
│  ┌─────────────────────────────┐          │              │
│  │      ESP32-C3 SuperMini     │          │              │
│  │                             │          │              │
│  │  GPIO0 ◄── Nút bấm (R6 47k+C8)│       │              │
│  │  GPIO3 ◄── ADC_BAT (1M/1M) ┤──────────┘              │
│  │  GPIO1 ──▶ SW_LED1 (PMOS)──▶ SK6812 #1              │
│  │  GPIO4 ──▶ LED1 data       ─▶ SK6812 #1              │
│  │  GPIO10──▶ SW_LED2 (PMOS)──▶ SK6812 #2              │
│  │  GPIO6 ──▶ LED2 data       ─▶ SK6812 #2              │
│  │  GPIO7 ──▶ MOTOR (NMOS)   ─▶ Motor rung             │
│  │  GPIO20◄──▶ UART RX (TP)  │                          │
│  │  GPIO21──▶  UART TX (TP)  │                          │
│  └─────────────────────────────┘                         │
│                                                          │
│  ┌──────────┐                                            │
│  │ CHG_LED  │ ◄── LTC4054 CHRG pin                      │
│  └──────────┘                                            │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Danh sách linh kiện chính

| # | Linh kiện | Mã | Số lượng | Vai trò |
|---|-----------|-----|----------|---------|
| 1 | MCU | ESP32-C3 SuperMini | 1 | Điều khiển, WiFi/BLE |
| 2 | IC sạc | LTC4054ES5-4.2 | 1 | Sạc pin 1 cell |
| 3 | LDO | AP2112K-3.3 | 1 | Nguồn 3V3 |
| 4 | LED RGB | SK6812MINI-012 | 2 | Hiệu ứng ánh sáng |
| 5 | Motor rung | 3V coin motor | 1 | Phản hồi rung |
| 6 | Pin | Li-Po 220mAh | 1 | Nguồn chính |
| 7 | NMOS | AO3400 | 3 | Switch motor + LED |
| 8 | PMOS | AO3401A | 2 | Switch nguồn LED |

> **BOM đầy đủ**: Xem [bom.md](../hardware/bom.md)

### 4.3 Phân bổ GPIO

| GPIO | Chức năng | Chiều | Ghi chú |
|------|-----------|-------|---------|
| GPIO0 | Nút bấm | Input | Active LOW, pull-up 47kΩ lên 3V3 |
| GPIO1 | Switch LED 1 | Output | HIGH = bật LED1 |
| GPIO3 | ADC pin | Analog In | VBAT/2 qua cầu chia áp |
| GPIO4 | Data LED 1 | Output | SK6812 protocol |
| GPIO6 | Data LED 2 | Output | SK6812 protocol |
| GPIO7 | Motor rung | Output | HIGH = rung |
| GPIO10 | Switch LED 2 | Output | HIGH = bật LED2 |
| GPIO20 | UART RX | Input | Test pad debug |
| GPIO21 | UART TX | Output | Test pad debug |

> **Chi tiết**: Xem [hardware.md](../hardware/hardware.md)

---

## 5. Đặc tả firmware (Firmware Specification)

### 5.1 Tổng quan

| Thuộc tính | Giá trị |
|------------|---------|
| **Framework** | Arduino (ESP32 Arduino Core) |
| **Ngôn ngữ** | C/C++ |
| **Giao thức mạng** | MQTT over WiFi (TLS nếu HiveMQ yêu cầu) |
| **OTA** | ArduinoOTA hoặc HTTP OTA |

### 5.2 State Machine

```
┌─────────────┐
│  POWER ON   │
└──────┬──────┘
       ▼
┌─────────────┐     Chưa có WiFi config    ┌──────────────┐
│ INIT / BOOT │ ──────────────────────────▶ │ AP MODE      │
│             │                             │ Captive Portal│
└──────┬──────┘                             │ (config WiFi, │
       │ Đã có config                       │  chọn màu LED)│
       ▼                                    └──────┬───────┘
┌─────────────────────────────┐                    │ Lưu config
│         DEEP SLEEP          │◀───────────────────┘
│  (Timer: 5min / 30min)      │
│  (GPIO0: nút bấm interrupt) │
└──────┬──────────────┬───────┘
       │ Timer hết    │ Nút bấm
       ▼              ▼
┌──────────────────────────────┐
│          WAKE UP             │
│  1. Kết nối WiFi (~1-3s)     │
│  2. Kết nối MQTT             │
└──────┬──────────────┬────────┘
       │              │
       │ Chỉ timer    │ Có bấm nút
       ▼              ▼
┌────────────┐  ┌──────────────────────┐
│ CHECK ONLY │  │ SEND + CHECK         │
│ Subscribe  │  │ 1. Publish tín hiệu  │
│ & đợi msg  │  │ 2. Subscribe & đợi   │
│ (~500ms)   │  │    msg (~500ms)       │
└──────┬─────┘  └──────────┬───────────┘
       │                   │
       ▼                   ▼
┌──────────────────────────────┐
│     Có tín hiệu đến?        │
│  CÓ → LED nháy 3s + Motor 1s│
│  KHÔNG → bỏ qua             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│  Đo ADC pin → kiểm tra VBAT │
│  Disconnect WiFi + MQTT      │
│  Tính timer tiếp theo        │
│  → Quay lại DEEP SLEEP       │
└──────────────────────────────┘

Bấm giữ 10s (khi wake) ──▶ FACTORY RESET ──▶ POWER ON
VBAT ≤ 3.0V ──▶ DEEP SLEEP vĩnh viễn (chỉ tỉnh khi sạc)
```

### 5.3 Hành vi chi tiết

#### 5.3.1 Khởi động (Boot Sequence)

1. Bật công tắc nguồn → LDO cấp 3V3 → ESP32-C3 khởi động
2. Kiểm tra flash có WiFi config không
3. **Có config** → Kết nối WiFi → Kết nối MQTT → Vào IDLE
4. **Chưa có config** → Vào AP Mode (phát WiFi tên `CoupleKey_XXXX`)
5. Toàn bộ quá trình boot < 5 giây

#### 5.3.2 AP Mode (Captive Portal)

1. ESP32 phát WiFi AP: `CoupleKey_XXXX` (XXXX = 4 ký tự cuối MAC)
2. Người dùng kết nối WiFi → tự động hiện trang web cấu hình
3. Trang web cho phép:
   - Nhập SSID và password WiFi nhà
   - Chọn màu LED (color picker, lưu dạng **RGB** — 3 giá trị r, g, b)
   - Hiển thị Pair ID (cố định trong firmware, không cho người dùng sửa)
   - Hiển thị firmware version (semantic versioning, ví dụ: 1.0.0)
4. Lưu config vào flash → Restart → Kết nối
5. **Timeout**: Nếu không có thao tác cấu hình trong **5 phút** → tự động vào deep sleep để tiết kiệm pin

#### 5.3.3 Gửi tín hiệu (Send — khi bấm nút)

Khi người dùng bấm nút, thiết bị thức dậy và thực hiện **cả gửi lẫn nhận** trong cùng một phiên wake:

1. GPIO0 interrupt → ESP32-C3 wake từ deep sleep
2. Debounce 50ms → xác nhận bấm hợp lệ
3. Kiểm tra anti-spam: nếu chưa đủ **1 phút** kể từ lần gửi cuối (lưu trong RTC memory) → bỏ qua gửi, LED nháy vàng 1 lần
4. Kết nối WiFi → Kết nối MQTT
5. **Publish** tín hiệu lên topic: `couplekey/{pair_id}/signal`
6. LED nháy xanh 1 lần để xác nhận đã gửi
7. **Đồng thời subscribe** topic `couplekey/{pair_id}/signal` với `retained = true`
8. Đợi **~500ms** để nhận message (nếu có tín hiệu chờ từ đối phương)
9. Nếu có tín hiệu đến → thực hiện hiệu ứng nhận (xem 5.3.4)
10. Đo ADC pin → Disconnect WiFi/MQTT → Quay lại deep sleep

#### 5.3.4 Nhận tín hiệu (Receive — polling theo timer)

Khi timer hết (5 phút hoặc 30 phút), thiết bị thức dậy để **kiểm tra** xem có tín hiệu chờ không:

1. Timer wake → ESP32-C3 thức dậy
2. Kết nối WiFi → Kết nối MQTT
3. Subscribe topic `couplekey/{pair_id}/signal` với **retained message**
4. Đợi **~500ms** để nhận message
5. **Nếu có tín hiệu** (message `from` khác ID mình):
   - Bật SW_LED1 + SW_LED2 → cấp nguồn cho LED
   - LED1 + LED2 nháy theo màu đã chọn trong **3 giây**
   - Motor rung **1 giây**: ON 400ms (80% PWM) → OFF 100ms → lặp lại (~1s)
   - Clear retained message trên server (publish empty payload với retain=true)
6. **Nếu không có tín hiệu** → bỏ qua
7. Đo ADC pin → Disconnect WiFi/MQTT → Tính timer tiếp theo → Deep sleep

> **Lưu ý MQTT Retained Message**: Server cần giữ retained message để thiết bị polling có thể nhận được tín hiệu ngay cả khi đang sleep. Sau khi nhận xong, thiết bị clear retained message để không nhận lại lần sau.

#### 5.3.5 Quản lý pin

| Điều kiện | Hành vi |
|-----------|---------|
| VBAT > 3.5V | Hoạt động bình thường |
| 3.0V < VBAT ≤ 3.3V | Cảnh báo: LED nháy đỏ khi nhận tín hiệu, motor KHÔNG rung |
| VBAT ≤ 3.0V | Firmware tắt thiết bị (deep sleep vĩnh viễn, chỉ tỉnh khi sạc) |

- Đo ADC **mỗi lần wake** (cả timer lẫn nút bấm)
- Công thức: `VBAT = ADC_RAW × (3.3 / 4095) × 2`

#### 5.3.6 Deep Sleep và Wake Schedule

Thiết bị sử dụng **deep sleep hoàn toàn** (không giữ kết nối WiFi), chỉ thức dậy khi:

| Nguồn wake | Điều kiện | Hành vi khi wake |
|------------|-----------|------------------|
| **Timer (RTC)** | Hết interval (5 min hoặc 30 min) | Kết nối WiFi → MQTT → Kiểm tra tín hiệu → Sleep |
| **Nút bấm (GPIO0)** | Người dùng nhấn nút | Kết nối WiFi → MQTT → Gửi + Kiểm tra tín hiệu → Sleep |

##### Lịch wake chi tiết

```
07:00 ──────────────────────────── 23:00 ──────────────── 07:00
│          Ban ngày (16h)          │     Ban đêm (8h)     │
│    Wake mỗi 5 phút (~192 lần)   │  Wake mỗi 30 phút    │
│                                  │  (~16 lần)           │
└──────────────────────────────────┴──────────────────────┘
```

##### Cách firmware xác định khung giờ

1. Khi cấu hình WiFi lần đầu, firmware lấy thời gian từ **NTP server** (ví dụ: `pool.ntp.org`)
2. **Timezone**: Lấy tự động từ NTP, không hardcode UTC+7. Firmware dùng timezone từ NTP response hoặc cấu hình mặc định UTC+7 nếu không xác định được
3. Lưu thời gian vào **RTC memory** của ESP32-C3
4. Mỗi lần wake, cộng dồn thời gian sleep để tính giờ hiện tại
5. Dựa vào giờ hiện tại để quyết định interval tiếp theo (5 min hoặc 30 min)
6. Đồng bộ lại NTP mỗi **24 giờ** (hoặc mỗi lần kết nối WiFi thành công) để tránh trôi giờ

##### Chu trình wake điển hình (~3 giây)

```
Wake → WiFi Connect (~1-2s) → MQTT Connect (~0.5s)
     → Subscribe + Check retained msg (~0.5s)
     → [Nếu có: LED 3s + Motor 1s]
     → ADC đo pin → Disconnect → Tính timer → Deep Sleep
```

> **Ưu điểm**: Deep sleep chỉ tiêu thụ ~5µA, tốt hơn rất nhiều so với light sleep (~1-5mA). Đánh đổi: độ trễ nhận tín hiệu tối đa 5 phút (ban ngày) hoặc 30 phút (ban đêm).

#### 5.3.7 Mất kết nối (khi wake)

1. Wake → cố kết nối WiFi, timeout **10 giây**
2. Nếu WiFi thất bại → LED nháy đỏ 2 lần → quay lại deep sleep (thử lại ở lần wake tiếp theo)
3. Nếu WiFi OK nhưng MQTT thất bại → retry 2 lần (mỗi lần 2 giây) → nếu vẫn fail → LED nháy đỏ 2 lần → deep sleep
4. Nếu bấm nút mà mất mạng → LED nháy đỏ 3 lần (báo gửi thất bại), **lưu tín hiệu vào RTC memory** để gửi lại ở lần wake tiếp theo
5. **Pending signal**: Chỉ lưu tối đa **1 tín hiệu** trong RTC memory. Tín hiệu pending bị xóa sau **24 giờ** (hết hạn trong ngày). Nếu gửi thất bại nhiều lần liên tiếp, chỉ giữ tín hiệu mới nhất
6. Không có exponential backoff — mỗi lần wake đều thử kết nối mới
7. Không giảm tần suất polling khi mất mạng dài ngày — giữ nguyên schedule 5min/30min

#### 5.3.8 Factory Reset

1. Bấm giữ nút **10 giây liên tục**
2. Sau 3 giây giữ: LED nháy nhanh cảnh báo (cho người dùng biết đang trong quá trình reset)
3. Sau 10 giây: Xóa toàn bộ config WiFi, màu LED trong flash (Pair ID không bị xóa vì cố định)
4. LED nháy trắng 3 lần → Restart → Vào AP Mode

#### 5.3.9 OTA Update Mode

1. Bấm giữ nút **20 giây liên tục** (khi thiết bị đang wake)
2. Sau 10 giây: LED nháy nhanh (cảnh báo factory reset — nhưng tiếp tục giữ)
3. Sau 15 giây: LED chuyển sang nháy xanh dương chậm (báo hiệu sắp vào OTA mode)
4. Sau 20 giây: Thiết bị vào chế độ OTA
   - Kết nối WiFi đã lưu
   - Khởi động ArduinoOTA hoặc HTTP OTA server
   - LED nháy xanh dương liên tục (báo đang chờ firmware)
5. Timeout OTA: **5 phút** — nếu không nhận firmware → tự restart về chế độ bình thường
6. Sau khi nhận firmware thành công → LED nháy xanh lá 5 lần → Restart với firmware mới

---

## 6. Giao diện Captive Portal (Web UI)

### 6.1 Yêu cầu

| Thuộc tính | Giá trị |
|------------|---------|
| **Công nghệ** | HTML/CSS/JS inline (lưu trong flash ESP32) |
| **Responsive** | Tối ưu cho màn hình điện thoại |
| **Các trang** | 1 trang duy nhất (single page) |

### 6.2 Nội dung trang cấu hình

1. **Logo / Tên sản phẩm**: Smart Couple Coin
2. **Firmware Version**: Hiển thị version hiện tại (semantic versioning, ví dụ: v1.0.0)
3. **WiFi Setup**:
   - Scan và liệt kê WiFi khả dụng
   - Ô nhập password
4. **Color Picker**: Chọn màu LED hiệu ứng (lưu dạng RGB — r, g, b). Full color wheel, không giới hạn preset
5. **Nút Lưu**: Lưu config và restart thiết bị
6. **Pair ID**: Hiển thị Pair ID (chỉ đọc, cố định trong firmware, không cho sửa)
7. **Device ID**: Hiển thị ID thiết bị ("A" hoặc "B")

---

## 7. Giao thức MQTT

### 7.1 Broker

| Thuộc tính | Giá trị |
|------------|---------|
| **Dịch vụ** | HiveMQ Cloud (free tier) |
| **Account** | **1 account chung** cho tất cả các cặp thiết bị. Phân biệt cặp bằng `pair_id` trong topic |
| **Bảo mật** | TLS (port 8883) |
| **Xác thực** | Username/Password **hardcode trong firmware** (không cho người dùng nhập) |

### 7.2 Topic Structure

```
couplekey/{pair_id}/signal     — Gửi/nhận tín hiệu bấm nút
couplekey/{pair_id}/status     — (Tương lai) Trạng thái online/offline
couplekey/{pair_id}/config     — (Tương lai) Đồng bộ cấu hình
```

> **Pair ID**: Cố định trong firmware, mỗi cặp thiết bị được nạp cùng 1 `pair_id` khi sản xuất. Không thay đổi được bởi người dùng.

### 7.3 Message Format

**Signal Message** (QoS 1):
```json
{
  "from": "A",
  "ts": 1234567890,
  "bat": 85
}
```

| Trường | Mô tả |
|--------|-------|
| `from` | ID thiết bị gửi ("A" hoặc "B"), cố định trong firmware |
| `ts` | Unix timestamp |
| `bat` | Phần trăm pin còn lại — chỉ để logging, thiết bị nhận **không xử lý** giá trị này |

> **Race condition**: Nếu cả 2 thiết bị bấm nút gần đồng thời, mỗi thiết bị vừa publish vừa check retained message. Chấp nhận behavior này — không cần xử lý race condition đặc biệt.

> **Retained message overlap**: Nếu A gửi 2 lần liên tiếp trước khi B wake, B chỉ nhận được retained message mới nhất. **Chấp nhận mất tín hiệu cũ** — đây là giới hạn của polling model.

---

## 8. Cơ khí và vỏ (Mechanical)

### 8.1 Thiết kế vỏ

```
         ┌──────────────────────┐
         │  ○ Lỗ xỏ dây móc    │
         │      khóa            │
         ├──────────────────────┤
         │                      │
         │    ┌────┐  ┌────┐    │
         │    │LED1│  │LED2│    │  ← Diffuser nhựa mờ
         │    └────┘  └────┘    │
         │                      │
         │      [  NÚT  ]      │  ← Nút bấm
         │                      │
         ├──────────────────────┤
  USB-C ─┤  ╔══╗  ← nắp đậy   │
         │  ╚══╝               │
         ├──────────────────────┤
         │    SW ← công tắc     │
         └──────────────────────┘
```

### 8.2 Yêu cầu cơ khí

| # | Yêu cầu |
|---|---------|
| 1 | Vỏ 2 nửa (top + bottom), gắn bằng **vít M2** (chốt) |
| 2 | Khu vực LED có **diffuser nhựa mờ** để tán sáng đều |
| 3 | Lỗ cho USB-C có nắp đậy silicon/nhựa mềm |
| 4 | Lỗ xỏ dây ở cạnh trên, đường kính ≥ 3mm |
| 5 | Khe cho công tắc nguồn bên cạnh |
| 6 | Chống bụi: gioăng hoặc tolerance chặt giữa 2 nửa vỏ |
| 7 | CHG_LED phải nhìn thấy được qua vỏ (lỗ hoặc vùng trong suốt) |

---

## 9. Quy trình sử dụng (User Flow)

### 9.1 Lần đầu sử dụng (First-Time Setup)

```
1. Bật công tắc nguồn
   └──▶ LED nháy trắng → Thiết bị phát WiFi "CoupleKey_XXXX"

2. Kết nối WiFi "CoupleKey_XXXX" bằng điện thoại
   └──▶ Tự động hiện trang cấu hình

3. Nhập WiFi nhà + Chọn màu LED + Lưu
   └──▶ Thiết bị restart, kết nối WiFi

4. Khi LED nháy xanh 2 lần → Sẵn sàng!

5. Lặp lại bước 1-4 cho thiết bị thứ 2
```

### 9.2 Sử dụng hàng ngày

```
Người A bấm nút
   └──▶ LED xanh nháy 1 lần (xác nhận gửi)
   └──▶ Tín hiệu đi qua Internet (MQTT)
   └──▶ Thiết bị B nhận:
         ├── LED nháy màu đã chọn (3 giây)
         └── Motor rung (1 giây)
```

### 9.3 Sạc pin

```
Cắm USB-C
   └──▶ CHG_LED sáng (đang sạc)
   └──▶ Thiết bị vẫn hoạt động bình thường
   └──▶ CHG_LED tắt (sạc xong)
```

---

## 10. Rủi ro và giới hạn đã biết

| # | Rủi ro | Mức độ | Giải pháp |
|---|--------|--------|-----------|
| 1 | Pin không có PCM → có thể xả quá sâu | Trung bình | Firmware cut-off ở 3.0V + ưu tiên pin có PCM |
| 2 | Light sleep WiFi tiêu thụ 1-5mA → pin có thể không đạt 5 ngày | Cao | Cần test thực tế, cân nhắc wake theo interval thay vì giữ kết nối |
| 3 | Mất Internet = thiết bị không hoạt động | Cao | Đây là giới hạn thiết kế (cần WiFi), báo lỗi rõ ràng cho người dùng |
| 4 | Không mã hóa/xác thực → ai biết topic có thể gửi tín hiệu giả | Thấp | Chấp nhận cho prototype, thêm auth ở phase sau |
| 5 | ESP32-C3 SuperMini không có reset button → khó debug | Thấp | Dùng USB-C để nạp lại, có test pad UART |

---

## 11. Phạm vi Phase 1 vs Phase 2

| Tính năng | Phase 1 (Prototype) | Phase 2 (Phát triển) |
|-----------|---------------------|----------------------|
| Gửi/nhận tín hiệu | ✅ | ✅ |
| LED + Motor | ✅ | ✅ |
| WiFi + MQTT (HiveMQ) | ✅ | ✅ |
| Captive Portal config | ✅ | ✅ |
| OTA Update | ✅ | ✅ |
| BLE qua App | ❌ | ✅ |
| Server MQTT riêng | ❌ | ✅ |
| Mã hóa/xác thực | ❌ | ✅ |
| Chế độ im lặng | ❌ | ❌ (Không yêu cầu) |
| Nhiều pattern tín hiệu | ❌ | Xem xét |

---

## 12. Tiêu chí nghiệm thu (Acceptance Criteria)

### Phase 1 — Prototype

#### Chức năng cốt lõi
- [ ] Bấm nút thiết bị A → tín hiệu publish thành công trong **< 3 giây**
- [ ] Bấm nút thiết bị B → tín hiệu publish thành công trong **< 3 giây**
- [ ] Thiết bị B nhận tín hiệu từ A trong **≤ 5 phút** (ban ngày) hoặc **≤ 30 phút** (ban đêm)
- [ ] Thiết bị A nhận tín hiệu từ B trong **≤ 5 phút** (ban ngày) hoặc **≤ 30 phút** (ban đêm)
- [ ] Khi nhận tín hiệu: LED nháy đúng màu đã chọn trong 3 giây + Motor rung 1 giây

#### Cấu hình
- [ ] Cấu hình WiFi qua Captive Portal thành công
- [ ] Chọn màu LED (RGB color picker) trên web và LED hiển thị đúng màu
- [ ] Captive Portal hiển thị firmware version và Pair ID
- [ ] AP Mode tự tắt sau 5 phút nếu không có thao tác

#### Quản lý tín hiệu
- [ ] Chống spam: bấm liên tục chỉ gửi 1 lần trong 1 phút
- [ ] Gửi thất bại → lưu pending signal, gửi lại lần wake sau
- [ ] Pending signal hết hạn sau 24 giờ

#### Trạng thái & Thông báo
- [ ] Báo mất kết nối bằng LED nháy đỏ
- [ ] Báo pin yếu bằng LED đỏ (VBAT ≤ 3.3V), motor không rung
- [ ] Tự tắt (deep sleep vĩnh viễn) khi VBAT ≤ 3.0V

#### Nguồn & Sạc
- [ ] Sạc USB-C hoạt động, CHG_LED sáng đúng
- [ ] Thiết bị hoạt động bình thường khi đang sạc
- [ ] Cắm USB khi công tắc OFF → chỉ sạc, không tự bật

#### Nút bấm
- [ ] Bấm giữ 10 giây → Factory Reset thành công
- [ ] Bấm giữ 20 giây → Vào chế độ OTA Update

#### Firmware & OTA
- [ ] OTA update firmware thành công qua WiFi
- [ ] Watchdog timer hoạt động (reset sau 15 giây treo)
- [ ] NTP sync hoạt động, wake schedule chuyển đổi đúng ban ngày/đêm

#### Phần cứng
- [ ] Thời lượng pin ≥ 5 ngày (đo thực tế)
- [ ] Dòng deep sleep ≤ 50µA
- [ ] Thiết bị vừa trong vỏ 50×35×18mm
- [ ] Vỏ gắn bằng vít M2
