# Quy Trình Hàn, Kiểm Tra Phần Cứng & Phát Triển Firmware

> **Dự án**: Smart Couple Coin\
> **Phiên bản**: 1.0\
> **Ngày**: 2026-06-11\
> **Tác giả**: hado

---

## Mục lục

- [Phần A: Quy trình hàn và kiểm tra phần cứng](#phần-a-quy-trình-hàn-và-kiểm-tra-phần-cứng)
- [Phần B: Quy trình phát triển firmware](#phần-b-quy-trình-phát-triển-firmware)

---

# Phần A: Quy Trình Hàn và Kiểm Tra Phần Cứng

## A1. Chuẩn bị

### A1.1 Dụng cụ cần thiết

| # | Dụng cụ | Mục đích | Bắt buộc |
|---|---------|----------|----------|
| 1 | Mỏ hàn chì (đầu nhọn JBC/Hakko) | Hàn SMD 0603/0805, SOT-23 | ✅ |
| 2 | Chì hàn 0.3-0.5mm (Sn63/Pb37 hoặc lead-free) | Hàn linh kiện | ✅ |
| 3 | Flux (dạng bút hoặc kem) | Hỗ trợ hàn SMD, chống bám dính | ✅ |
| 4 | Kính lúp / Kính hiển vi SMD | Kiểm tra mối hàn 0603, SK6812MINI | ✅ |
| 5 | Nhíp chống tĩnh điện (ESD tweezers) | Gắp linh kiện SMD | ✅ |
| 6 | Đồng hồ vạn năng (multimeter) | Đo thông mạch, điện áp, điện trở | ✅ |
| 7 | Nguồn lập trình (bench PSU) hoặc cáp USB-C | Cấp nguồn test | ✅ |
| 8 | Dây hút chì / bơm hút chì | Sửa lỗi hàn | ✅ |
| 9 | Cồn IPA + bàn chải | Vệ sinh flux sau hàn | ✅ |
| 10 | Keo tản nhiệt / keo dán (tùy chọn) | Cố định motor, pin | Tùy chọn |

### A1.2 Kiểm tra PCB trước khi hàn

- [ ] Kiểm tra bề mặt PCB: không trầy xước, không đứt trace
- [ ] Kiểm tra thông mạch GND giữa các vùng (dùng multimeter mode continuity)
- [ ] Kiểm tra pad USB-C, pad pin, pad motor không bị ngắn mạch
- [ ] Đối chiếu footprint trên PCB với linh kiện thực tế (đặc biệt SK6812MINI 1.2×1.2mm)

---

## A2. Thứ tự hàn linh kiện

> **Nguyên tắc**: Hàn từ **thấp → cao**, **nhỏ → lớn**, **khó → dễ**. Hàn IC và linh kiện nhỏ trước, module lớn sau cùng.

### Bước 1: Mặt BOTTOM (mặt dưới PCB)

Hàn mặt bottom trước vì linh kiện ở đây ít và nhỏ, tránh lật ngược khi đã gắn module ESP32 lớn.

| Thứ tự | Linh kiện | Giá trị | Vị trí | Lưu ý |
|--------|-----------|---------|--------|-------|
| 1.1 | R2, R4 | 1MΩ (0805) | Cầu chia áp ADC | Chú ý hướng (không phân cực) |
| 1.2 | R17, R15 | 100Ω (0603) | Gate MOSFET LED1 | |
| 1.3 | R13, R16 | 100Ω (0603) | Gate MOSFET LED2 | R16 là 0805 |
| 1.4 | R18, R19 | 100kΩ (0603) | Pull-down gate PMOS | |
| 1.5 | C5 | 100nF (0603) | Lọc ADC | |
| 1.6 | C10, C11 | 10µF | Tụ bypass LED | |
| 1.7 | C12 | 47µF | Tụ nguồn motor | Chú ý phân cực nếu dùng tụ điện phân |
| 1.8 | Q3 (AO3401A) | PMOS SOT-23 | Switch nguồn LED1 | **Chú ý hướng chân G/S/D** |
| 1.9 | Q5 (AO3400) | NMOS SOT-23 | Driver LED1 | **Chú ý hướng chân G/S/D** |
| 1.10 | Q2 (AO3401A) | PMOS SOT-23 | Switch nguồn LED2 | |
| 1.11 | Q4 (AO3400) | NMOS SOT-23 | Driver LED2 | |

### Bước 2: Mặt TOP — Nguồn và sạc (ưu tiên cao nhất)

| Thứ tự | Linh kiện | Giá trị | Vị trí | Lưu ý |
|--------|-----------|---------|--------|-------|
| 2.1 | U1 (LTC4054) | SOT-23-5 | IC sạc | **Cực kỳ chú ý hướng chân 1** (chấm tròn). Hàn 1 chân trước, kiểm tra, rồi hàn các chân còn lại |
| 2.2 | R5 | 6.8kΩ (0603) | Set dòng sạc | Xác nhận đúng giá trị bằng multimeter |
| 2.3 | C1, C4 | 10µF | Tụ input/output IC sạc | Đặt gần U1 |
| 2.4 | R12 | 2.2kΩ (0603) | LED sạc | |
| 2.5 | CHG_LED | LED 0805 | LED báo sạc | **Chú ý phân cực anode/cathode** |
| 2.6 | D3 (SMF5.0A) | TVS diode | Bảo vệ USB | **Cathode (vạch) nối V_CHARGE** |
| 2.7 | R1, R3 | 5.1kΩ (0603) | CC1/CC2 USB | |

### Bước 3: Mặt TOP — LDO và nguồn 3V3

| Thứ tự | Linh kiện | Giá trị | Vị trí | Lưu ý |
|--------|-----------|---------|--------|-------|
| 3.1 | U2 (AP2112K) | SOT-23-5 | LDO 3V3 | **Chú ý hướng chân**. Hàn 1 chân trước, kiểm tra alignment |
| 3.2 | C2 | 10µF | Tụ input LDO | Đặt gần VIN |
| 3.3 | C3 | 10µF | Tụ output LDO | Đặt gần VOUT |
| 3.4 | R21 | 100kΩ (0805) | Pull-down LDO EN | |
| 3.5 | R11 | 0Ω / DNP | Jumper | **Mặc định KHÔNG hàn** |

### Bước 4: Mặt TOP — Motor driver

| Thứ tự | Linh kiện | Giá trị | Vị trí | Lưu ý |
|--------|-----------|---------|--------|-------|
| 4.1 | Q1 (AO3400) | NMOS SOT-23 | Motor driver | Chú ý hướng chân |
| 4.2 | R9 | 100Ω (0603) | Gate resistor | |
| 4.3 | R10 | 100kΩ (0603) | Pull-down gate | |
| 4.4 | SS14 | Schottky diode | Dập xung motor | **Cathode (vạch) nối VBAT, Anode nối MOTOR-** |
| 4.5 | C9 | 100nF | Lọc nhiễu motor | |

### Bước 5: Mặt TOP — LED RGB và nút bấm

| Thứ tự | Linh kiện | Giá trị | Vị trí | Lưu ý |
|--------|-----------|---------|--------|-------|
| 5.1 | R7, R8 | 330Ω (0603) | Series data LED | |
| 5.2 | R20, R14 | 100kΩ | Pull-down data LED | |
| 5.3 | D1 (SK6812MINI) | LED RGB | LED 1 | ⚠️ **Rất nhỏ 1.2×1.2mm**. Chú ý hướng chân (GND mark). Nhiệt hàn ≤ 260°C, ≤ 5 giây |
| 5.4 | D2 (SK6812MINI) | LED RGB | LED 2 | Tương tự D1 |
| 5.5 | R6 | 100kΩ (0603) | Pull-up nút bấm | *(Khuyến nghị đổi sang 10-47kΩ theo review)* |
| 5.6 | C8 | 100nF | Debounce nút | |

### Bước 6: Mặt TOP — Tụ nguồn MCU

| Thứ tự | Linh kiện | Giá trị | Vị trí | Lưu ý |
|--------|-----------|---------|--------|-------|
| 6.1 | C7 | 100nF | Lọc nhiễu cao tần | Đặt gần chân 3V3/GND module |
| 6.2 | C6 | 47µF | Tụ bulk nguồn MCU | Đặt gần module ESP32 |

### Bước 7: Mặt TOP — Connector và module (hàn cuối cùng)

| Thứ tự | Linh kiện | Vị trí | Lưu ý |
|--------|-----------|--------|-------|
| 7.1 | J1 (USB-C) | Cổng USB | Hàn 4 chân cơ khí trước, sau đó hàn chân tín hiệu. Kiểm tra không ngắn mạch CC1/CC2/VBUS/GND |
| 7.2 | U3 (ESP32-C3 SuperMini) | Module MCU | **Hàn cuối cùng**. Đặt module lên header pad, hàn 2 chân góc đối diện trước, kiểm tra alignment, sau đó hàn hết. **KHÔNG cấp nguồn vào chân 5V** nếu đã cấp qua 3V3 |

### Bước 8: Kết nối ngoại vi (dây/pad)

| Thứ tự | Kết nối | Lưu ý |
|--------|---------|-------|
| 8.1 | Pad SW (công tắc nguồn) | Hàn dây đến công tắc ngoài. Kiểm tra: gạt ON → LDO_EN = VBAT |
| 8.2 | Pad BAT+/BAT- | Hàn dây pin Li-Po. ⚠️ **CHÚ Ý ĐÚNG CỰC +/-**. Kiểm tra kỹ 2 lần trước khi hàn |
| 8.3 | Pad MOTOR+/MOTOR- | Hàn dây motor rung |

---

## A3. Kiểm tra phần cứng (Hardware Test)

### A3.1 Test Level 0 — Kiểm tra ngoại quan (trước khi cấp nguồn)

> ⚠️ **KHÔNG CẤP NGUỒN** cho đến khi hoàn thành bước này.

- [ ] **Kiểm tra mối hàn bằng kính lúp**: Tìm bridge (ngắn mạch), cold joint (hàn lạnh), missing pad
- [ ] **Kiểm tra ngắn mạch nguồn**: Multimeter đo giữa VBAT và GND → phải > 10kΩ
- [ ] **Kiểm tra ngắn mạch 3V3**: Đo giữa 3V3 và GND → phải > 10kΩ
- [ ] **Kiểm tra ngắn mạch USB**: Đo giữa VBUS (V_CHARGE) và GND → phải > 1kΩ
- [ ] **Kiểm tra hướng IC**: LTC4054 chân 1 đúng vị trí, AP2112K chân 1 đúng vị trí
- [ ] **Kiểm tra hướng diode**: SS14, SMF5.0A, CHG_LED — cathode/anode đúng
- [ ] **Kiểm tra hướng LED RGB**: SK6812MINI chân GND mark đúng

### A3.2 Test Level 1 — Mạch sạc và nguồn (KHÔNG cần ESP32)

> Có thể test mà **chưa hàn module ESP32** để cô lập lỗi.

**Test 1.1: USB input**

1. Cắm cáp USB-C vào J1
2. Đo điện áp V_CHARGE: **kỳ vọng ~5V ±0.25V**
3. Đo dòng tiêu thụ không tải: **kỳ vọng < 1mA** (chỉ IC sạc standby)

```
✅ PASS: V_CHARGE = 4.75-5.25V
❌ FAIL: Kiểm tra R1/R3 (5.1kΩ), D3 (TVS), J1 (USB-C)
```

**Test 1.2: IC sạc LTC4054**

1. Cắm USB-C + Kết nối pin Li-Po (hoặc dùng bench PSU set 3.7V/100mA limit thay pin)
2. Đo điện áp chân BAT (VBAT): **kỳ vọng tăng dần từ 3.7V → 4.2V**
3. Đo dòng sạc: **kỳ vọng ~140-150mA** (R5 = 6.8kΩ)
4. CHG_LED: **kỳ vọng sáng** khi đang sạc

```
✅ PASS: Dòng sạc 130-170mA, CHG_LED sáng
❌ FAIL: Kiểm tra R5, U1 hướng chân, CHG_LED phân cực
```

**Test 1.3: LDO 3V3**

1. Gạt công tắc nguồn ON (hoặc nối trực tiếp VBAT → LDO_EN)
2. Đo điện áp ngõ ra 3V3: **kỳ vọng 3.3V ±0.1V**
3. Đo dòng không tải: **kỳ vọng < 100µA** (LDO quiescent)

```
✅ PASS: 3V3 = 3.2-3.4V
❌ FAIL: Kiểm tra U2 hướng chân, R21 (pull-down EN), công tắc
```

**Test 1.4: Công tắc nguồn**

1. Gạt OFF: đo LDO_EN → **kỳ vọng ~0V** (R21 kéo xuống GND)
2. Gạt ON: đo LDO_EN → **kỳ vọng = VBAT** (3.7-4.2V)
3. Đo 3V3 tương ứng: OFF → 0V, ON → 3.3V

```
✅ PASS: Công tắc hoạt động đúng
❌ FAIL: Kiểm tra pad SW, R21, R11
```

### A3.3 Test Level 2 — ESP32-C3 cơ bản

> Hàn module ESP32-C3 SuperMini (nếu chưa hàn). Cắm USB-C trực tiếp vào module để nạp firmware test.

**Test 2.1: Module boot**

1. Cắm USB-C vào **module ESP32-C3** (không phải USB-C sạc pin)
2. Mở Arduino IDE / PlatformIO → chọn board ESP32-C3
3. Upload sketch `Blink` (nháy LED onboard nếu có)
4. Mở Serial Monitor 115200 baud → **kỳ vọng thấy log boot**

```
✅ PASS: Module boot, Serial hoạt động
❌ FAIL: Kiểm tra hàn module, nguồn 3V3
```

**Test 2.2: GPIO test — Nút bấm**

```cpp
// test_button.ino
#define BTN_PIN 0  // GPIO0

void setup() {
  Serial.begin(115200);
  pinMode(BTN_PIN, INPUT_PULLUP);
}

void loop() {
  Serial.println(digitalRead(BTN_PIN)); // 1=thả, 0=nhấn
  delay(100);
}
```

```
✅ PASS: Thả = 1, Nhấn = 0, ổn định không rung
❌ FAIL: Kiểm tra R6 (pull-up), C8 (debounce), hàn nút bấm
```

**Test 2.3: GPIO test — ADC pin**

```cpp
// test_adc.ino
#define ADC_PIN 3  // GPIO3 = ADC_BAT

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
}

void loop() {
  int raw = analogRead(ADC_PIN);
  float vbat = raw * (3.3 / 4095.0) * 2.0;
  Serial.printf("ADC: %d, VBAT: %.2fV\n", raw, vbat);
  delay(1000);
}
```

```
✅ PASS: VBAT hiển thị đúng ±0.1V so với multimeter
❌ FAIL: Kiểm tra R2/R4 (1MΩ), C5 (100nF)
```

### A3.4 Test Level 3 — Ngoại vi

**Test 3.1: LED RGB SK6812**

```cpp
// test_led.ino
#include <Adafruit_NeoPixel.h>

#define SW_LED1  1   // GPIO1 - switch nguồn LED1
#define SW_LED2  10  // GPIO10 - switch nguồn LED2
#define LED1_PIN 4   // GPIO4 - data LED1
#define LED2_PIN 6   // GPIO6 - data LED2

Adafruit_NeoPixel led1(1, LED1_PIN, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel led2(1, LED2_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  // Enable LED power switches
  pinMode(SW_LED1, OUTPUT);
  pinMode(SW_LED2, OUTPUT);
  digitalWrite(SW_LED1, HIGH); // ON
  digitalWrite(SW_LED2, HIGH); // ON
  delay(10);

  led1.begin();
  led2.begin();
}

void loop() {
  // Red
  led1.setPixelColor(0, led1.Color(255, 0, 0));
  led2.setPixelColor(0, led2.Color(255, 0, 0));
  led1.show(); led2.show();
  delay(1000);

  // Green
  led1.setPixelColor(0, led1.Color(0, 255, 0));
  led2.setPixelColor(0, led2.Color(0, 255, 0));
  led1.show(); led2.show();
  delay(1000);

  // Blue
  led1.setPixelColor(0, led1.Color(0, 0, 255));
  led2.setPixelColor(0, led2.Color(0, 0, 255));
  led1.show(); led2.show();
  delay(1000);

  // Off
  led1.setPixelColor(0, 0);
  led2.setPixelColor(0, 0);
  led1.show(); led2.show();
  delay(1000);
}
```

```
✅ PASS: Cả 2 LED lần lượt Đỏ → Xanh lá → Xanh dương → Tắt
❌ FAIL nếu LED không sáng:
  → Kiểm tra SW_LEDx (Q2/Q3/Q4/Q5 hướng chân, R15-R19)
  → Kiểm tra data (R7/R8 330Ω)
  → Kiểm tra SK6812 hướng chân (GND mark)
  → Đo V_LED1 / V_LED2 khi switch ON: kỳ vọng ≈ VBAT
```

**Test 3.2: Motor rung**

```cpp
// test_motor.ino
#define MOTOR_PIN 7  // GPIO7

void setup() {
  pinMode(MOTOR_PIN, OUTPUT);
  // PWM setup
  ledcAttach(MOTOR_PIN, 1000, 8); // 1kHz, 8-bit
}

void loop() {
  // 80% duty cycle
  ledcWrite(MOTOR_PIN, 204); // 204/255 ≈ 80%
  delay(1000);

  // Off
  ledcWrite(MOTOR_PIN, 0);
  delay(2000);
}
```

```
✅ PASS: Motor rung 1 giây, nghỉ 2 giây, lặp lại
❌ FAIL: Kiểm tra Q1 (AO3400 hướng chân), R9, R10, SS14 hướng, dây motor
```

**Test 3.3: Deep Sleep + Wake by button**

```cpp
// test_deepsleep.ino
#define BTN_PIN 0

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("Awake! Going to sleep in 3s...");
  delay(3000);

  // Configure GPIO0 as wake source (LOW to wake)
  esp_deep_sleep_enable_gpio_wakeup(1 << BTN_PIN, ESP_GPIO_WAKEUP_GPIO_LOW);
  Serial.println("Entering deep sleep...");
  esp_deep_sleep_start();
}

void loop() {
  // Never reaches here
}
```

```
✅ PASS: Module vào sleep (dòng < 50µA), nhấn nút → wake → in "Awake!"
❌ FAIL: Kiểm tra GPIO0 wakeup support, nút bấm, R6/C8
```

**Test 3.4: Deep Sleep + Wake by timer**

```cpp
// test_timer_wake.ino
#define uS_TO_S 1000000ULL

void setup() {
  Serial.begin(115200);
  delay(1000);

  // Print wake reason
  esp_sleep_wakeup_cause_t cause = esp_sleep_get_wakeup_cause();
  Serial.printf("Wake cause: %d\n", cause);

  // Set timer for 10 seconds (test with short interval)
  esp_sleep_enable_timer_wakeup(10 * uS_TO_S);

  // Also enable GPIO0 button wake
  esp_deep_sleep_enable_gpio_wakeup(1 << 0, ESP_GPIO_WAKEUP_GPIO_LOW);

  Serial.println("Sleeping 10s...");
  esp_deep_sleep_start();
}

void loop() {}
```

```
✅ PASS: Wake mỗi 10 giây (cause=4) VÀ wake khi bấm nút (cause=7)
❌ FAIL: Kiểm tra RTC timer ESP32-C3
```

### A3.5 Test Level 4 — Tích hợp toàn bộ

**Test 4.1: WiFi connect**

```cpp
// test_wifi.ino
#include <WiFi.h>

void setup() {
  Serial.begin(115200);
  WiFi.begin("YOUR_SSID", "YOUR_PASSWORD");

  unsigned long start = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - start < 10000) {
    delay(100);
    Serial.print(".");
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.printf("\nConnected! IP: %s, Time: %lums\n",
      WiFi.localIP().toString().c_str(), millis() - start);
  } else {
    Serial.println("\nFailed to connect!");
  }
}

void loop() {}
```

```
✅ PASS: Kết nối WiFi < 5 giây
Ghi chú: Ghi lại thời gian kết nối thực tế → ảnh hưởng trực tiếp đến power budget
```

**Test 4.2: MQTT HiveMQ**

```cpp
// test_mqtt.ino (pseudocode — cần thư viện PubSubClient hoặc AsyncMqttClient)
#include <WiFi.h>
#include <PubSubClient.h>

// ... WiFi connect ...
// ... MQTT connect to HiveMQ Cloud (TLS port 8883) ...
// ... Publish test message ...
// ... Subscribe and wait for retained message ...
```

```
✅ PASS: Publish và nhận message thành công
Ghi chú: Đo tổng thời gian WiFi + MQTT connect + publish → target < 3 giây
```

**Test 4.3: Đo dòng deep sleep thực tế**

1. Ngắt USB, chỉ dùng pin
2. Bật công tắc nguồn
3. Nạp firmware deep sleep (không WiFi)
4. Đo dòng bằng multimeter (thang µA): **kỳ vọng 5-20µA**

```
✅ PASS: Deep sleep < 50µA
⚠️ WARNING nếu > 50µA: Kiểm tra LED switch có tắt không, R2/R4 dòng rò
❌ FAIL nếu > 500µA: Có linh kiện tiêu thụ dòng → kiểm tra từng khối
```

### A3.6 Bảng tổng kết kiểm tra

| Test | Mô tả | Kết quả | Ghi chú |
|------|-------|---------|---------|
| L0 | Kiểm tra ngoại quan | ☐ | |
| L1.1 | USB input 5V | ☐ | V = |
| L1.2 | IC sạc + dòng sạc | ☐ | I = |
| L1.3 | LDO 3V3 | ☐ | V = |
| L1.4 | Công tắc nguồn | ☐ | |
| L2.1 | ESP32 boot | ☐ | |
| L2.2 | Nút bấm GPIO0 | ☐ | |
| L2.3 | ADC pin | ☐ | VBAT đo = |
| L3.1 | LED RGB (2 LED) | ☐ | |
| L3.2 | Motor rung | ☐ | |
| L3.3 | Deep sleep + button wake | ☐ | I_sleep = |
| L3.4 | Deep sleep + timer wake | ☐ | |
| L4.1 | WiFi connect | ☐ | T_connect = |
| L4.2 | MQTT HiveMQ | ☐ | |
| L4.3 | Dòng deep sleep thực tế | ☐ | I = |

---

# Phần B: Quy Trình Phát Triển Firmware

## B1. Môi trường phát triển

### B1.1 Công cụ

| Công cụ | Phiên bản | Mục đích |
|---------|-----------|----------|
| Arduino IDE 2.x hoặc PlatformIO | Latest | IDE chính |
| ESP32 Arduino Core | ≥ 3.0 | Board support |
| Board | ESP32-C3 Dev Module | Target |
| USB Driver | CH340/CP210x (tùy module) | Giao tiếp serial |

### B1.2 Thư viện cần thiết

| Thư viện | Mục đích |
|----------|----------|
| `WiFi.h` | Kết nối WiFi (built-in) |
| `WiFiClientSecure.h` | TLS cho MQTT (built-in) |
| `PubSubClient` hoặc `AsyncMqttClient` | Giao tiếp MQTT |
| `Adafruit_NeoPixel` | Điều khiển SK6812 |
| `WebServer.h` / `DNSServer.h` | Captive Portal |
| `Preferences.h` | Lưu config vào flash (built-in) |
| `esp_sntp.h` | Đồng bộ thời gian NTP (built-in) |
| `ArduinoOTA` | OTA update |

### B1.3 Cấu hình board Arduino IDE

```
Board: ESP32C3 Dev Module
Upload Speed: 921600
CPU Frequency: 160MHz
Flash Mode: DIO
Flash Size: 4MB (32Mb)
Partition Scheme: Default 4MB with spiffs (hoặc OTA)
```

---

## B2. Kiến trúc firmware

### B2.1 Cấu trúc thư mục

```
firmware/
├── src/
│   ├── main.cpp              # Entry point, state machine chính
│   ├── config.h              # Hằng số, pin definitions
│   ├── wifi_manager.h/cpp    # WiFi connect, AP mode, Captive Portal
│   ├── mqtt_manager.h/cpp    # MQTT connect, publish, subscribe
│   ├── led_controller.h/cpp  # LED RGB effects
│   ├── motor_controller.h/cpp # Motor rung patterns
│   ├── power_manager.h/cpp   # ADC pin, deep sleep, wake schedule
│   ├── button_handler.h/cpp  # Nút bấm, debounce, long press
│   ├── ntp_time.h/cpp        # NTP sync, RTC time tracking
│   └── ota_updater.h/cpp     # OTA firmware update
├── data/                     # SPIFFS files
│   └── index.html            # Captive Portal web page
├── platformio.ini            # PlatformIO config (nếu dùng)
└── README.md
```

### B2.2 Sơ đồ module

```
┌──────────────────────────────────────────────────┐
│                   main.cpp                        │
│              (State Machine chính)                │
│                                                    │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ power_      │  │ wifi_        │  │ mqtt_    │ │
│  │ manager     │  │ manager      │  │ manager  │ │
│  │             │  │              │  │          │ │
│  │ • sleep()   │  │ • connect()  │  │ •publish│ │
│  │ • readADC() │  │ • startAP()  │  │ •subscribe│ │
│  │ • setTimer()│  │ • portal()   │  │ •check() │ │
│  └─────────────┘  └──────────────┘  └──────────┘ │
│                                                    │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ button_     │  │ led_         │  │ motor_   │ │
│  │ handler     │  │ controller   │  │controller│ │
│  │             │  │              │  │          │ │
│  │ • check()   │  │ • blink()    │  │ • buzz() │ │
│  │ • isLong()  │  │ • setColor() │  │ • stop() │ │
│  │ • debounce()│  │ • off()      │  │          │ │
│  └─────────────┘  └──────────────┘  └──────────┘ │
│                                                    │
│  ┌─────────────┐  ┌──────────────┐                │
│  │ ntp_time    │  │ ota_updater  │                │
│  │ • sync()    │  │ • check()    │                │
│  │ • getHour() │  │ • update()   │                │
│  └─────────────┘  └──────────────┘                │
└──────────────────────────────────────────────────┘
```

---

## B3. Lộ trình phát triển (Development Milestones)

### Phase 1: Foundation (Nền tảng)

| Bước | Mô tả | Tiêu chí hoàn thành | Ưu tiên |
|------|-------|---------------------|---------|
| F1.1 | **Pin definitions + config.h** | Tất cả GPIO, hằng số MQTT, thời gian được định nghĩa | 🔴 Cao |
| F1.2 | **Deep sleep + timer wake** | ESP32 sleep/wake đúng interval, phân biệt wake source | 🔴 Cao |
| F1.3 | **Deep sleep + button wake** | GPIO0 interrupt wake hoạt động, debounce đúng | 🔴 Cao |
| F1.4 | **LED controller** | Bật/tắt nguồn LED, nháy màu tuỳ chọn, off sau khi xong | 🔴 Cao |
| F1.5 | **Motor controller** | PWM 80%, pattern ON 400ms / OFF 100ms | 🔴 Cao |
| F1.6 | **ADC đo pin** | Đọc VBAT chính xác, phát hiện pin yếu | 🟡 Trung bình |

### Phase 2: Connectivity (Kết nối)

| Bước | Mô tả | Tiêu chí hoàn thành | Ưu tiên |
|------|-------|---------------------|---------|
| F2.1 | **WiFi connect** | Kết nối WiFi từ config đã lưu, timeout 10s | 🔴 Cao |
| F2.2 | **MQTT connect + publish** | Kết nối HiveMQ TLS, publish message thành công | 🔴 Cao |
| F2.3 | **MQTT subscribe + retained** | Subscribe topic, nhận retained message, clear sau khi nhận | 🔴 Cao |
| F2.4 | **NTP time sync** | Lấy thời gian từ NTP, lưu RTC, tính interval đúng | 🟡 Trung bình |

### Phase 3: Configuration (Cấu hình)

| Bước | Mô tả | Tiêu chí hoàn thành | Ưu tiên |
|------|-------|---------------------|---------|
| F3.1 | **Captive Portal AP Mode** | ESP32 phát WiFi AP, hiện trang web cấu hình | 🔴 Cao |
| F3.2 | **Web UI** | Trang HTML: nhập WiFi, color picker, nút lưu | 🔴 Cao |
| F3.3 | **Lưu/đọc config** | Preferences lưu SSID, password, màu LED, pair ID | 🔴 Cao |
| F3.4 | **Factory reset** | Bấm giữ 10s → xóa config → restart vào AP Mode | 🟡 Trung bình |

### Phase 4: Integration (Tích hợp)

| Bước | Mô tả | Tiêu chí hoàn thành | Ưu tiên |
|------|-------|---------------------|---------|
| F4.1 | **State machine hoàn chỉnh** | Tất cả state chuyển đổi đúng theo spec | 🔴 Cao |
| F4.2 | **Wake schedule (5min/30min)** | Phân biệt ban ngày/đêm, interval đúng | 🔴 Cao |
| F4.3 | **Send + Check đồng thời** | Bấm nút → publish + subscribe + check retained | 🔴 Cao |
| F4.4 | **Anti-spam** | Chỉ gửi 1 lần/X phút, lưu timestamp trong RTC memory | 🟡 Trung bình |
| F4.5 | **Lỗi mạng → lưu RTC** | Gửi thất bại → lưu pending signal → gửi lại lần wake sau | 🟡 Trung bình |
| F4.6 | **Chống bấm nhầm trong túi** | Debounce + logic phát hiện bấm ngắn vs bấm dài | 🟡 Trung bình |

### Phase 5: Polish (Hoàn thiện)

| Bước | Mô tả | Tiêu chí hoàn thành | Ưu tiên |
|------|-------|---------------------|---------|
| F5.1 | **OTA update** | Cập nhật firmware qua WiFi (ArduinoOTA hoặc HTTP OTA) | 🟡 Trung bình |
| F5.2 | **LED status patterns** | Phân biệt rõ: đang kết nối, mất mạng, pin yếu, đã gửi, nhận tín hiệu | 🟡 Trung bình |
| F5.3 | **Watchdog timer** | WDT reset nếu firmware treo > 15 giây khi wake | 🟡 Trung bình |
| F5.4 | **Power optimization** | Tối ưu thời gian wake, giảm dòng rò, test thời lượng pin thực tế | 🟢 Thấp |
| F5.5 | **Test 2 thiết bị giao tiếp** | Thiết bị A bấm → B nhận (và ngược lại), nhiều lần, ổn định | 🔴 Cao |

---

## B4. Quy trình phát triển từng bước

### B4.1 Mỗi module firmware

```
1. VIẾT CODE module (ví dụ: led_controller)
   │
2. UPLOAD lên ESP32-C3 qua USB-C
   │
3. TEST bằng Serial Monitor
   │   ├── OK → Chuyển module tiếp theo
   │   └── FAIL → Debug → Sửa → Quay lại bước 2
   │
4. GHI NHẬN kết quả test vào bảng A3.6
```

### B4.2 Tích hợp

```
1. GHÉP các module đã test riêng lẻ vào main.cpp
   │
2. TEST state machine: Boot → Sleep → Wake → Check/Send → Sleep
   │
3. TEST với 2 thiết bị thật:
   │   ├── Thiết bị A bấm → B nhận?
   │   ├── Thiết bị B bấm → A nhận?
   │   ├── Cả 2 bấm gần nhau → xử lý đúng?
   │   └── Mất WiFi → báo lỗi → reconnect?
   │
4. ĐO thời lượng pin thực tế (chạy 24-48h liên tục)
   │
5. OTA update test: push firmware mới qua WiFi
```

### B4.3 Lưu ý quan trọng khi phát triển

| # | Lưu ý |
|---|-------|
| 1 | **Luôn test deep sleep riêng trước** — nếu sleep không đúng, mọi thứ khác vô nghĩa |
| 2 | **Đo dòng thực tế** sau mỗi lần thêm module — phát hiện sớm linh kiện tiêu thụ bất thường |
| 3 | **WiFi connect time khác nhau tuỳ router** — test trên router thực tế của người dùng |
| 4 | **RTC memory mất khi reset** — chỉ giữ khi deep sleep, không giữ khi power off |
| 5 | **SK6812 cần tín hiệu chính xác** — tắt interrupt khi gửi data LED (NeoPixel tự làm) |
| 6 | **Chân 5V module ESP32-C3 SuperMini** — KHÔNG cấp nguồn vào đây nếu đã cấp 3V3 bên ngoài |
| 7 | **HiveMQ Cloud free tier** có giới hạn connection/message — kiểm tra quota |
| 8 | **Retained message MQTT** — phải clear sau khi nhận, nếu không sẽ nhận lại mỗi lần wake |

---

## B5. Checklist trước khi đóng vỏ

- [ ] Tất cả test Level 0-4 đã PASS
- [ ] 2 thiết bị giao tiếp qua MQTT ổn định (test > 50 lần gửi/nhận)
- [ ] Captive Portal hoạt động, chọn WiFi + màu LED OK
- [ ] Factory reset hoạt động (bấm giữ 10s)
- [ ] OTA update thành công ít nhất 1 lần
- [ ] Thời lượng pin thực tế ≥ 5 ngày
- [ ] Dòng deep sleep ≤ 50µA
- [ ] Thời gian wake + connect + check ≤ 5 giây
- [ ] Pin yếu: LED đỏ hoạt động đúng, auto shutdown ở 3.0V
- [ ] Hàn dây pin, motor, công tắc chắc chắn, có keo cố định
- [ ] Vệ sinh PCB sạch flux
