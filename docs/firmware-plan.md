# 🚀 PLAN — Firmware Development & Testing

> **Dự án**: Smart Couple Coin\
> **Spec tham chiếu**: [spec/spec.md](../spec/spec.md) v1.1\
> **Ngày tạo**: 2026-06-12\
> **Tác giả**: hado\
> **Framework**: Arduino (ESP32 Arduino Core ≥ 3.0)\
> **Board**: ESP32-C3 SuperMini

---

## Mục lục

- [Phase 0: Setup môi trường](#phase-0-setup-môi-trường)
- [Phase 1: Foundation — Phần cứng cơ bản](#phase-1-foundation--phần-cứng-cơ-bản)
- [Phase 2: Connectivity — Kết nối mạng](#phase-2-connectivity--kết-nối-mạng)
- [Phase 3: Configuration — Captive Portal](#phase-3-configuration--captive-portal)
- [Phase 4: Integration — State Machine](#phase-4-integration--state-machine)
- [Phase 5: Polish — Hoàn thiện](#phase-5-polish--hoàn-thiện)
- [Phase 6: System Test — Kiểm thử hệ thống](#phase-6-system-test--kiểm-thử-hệ-thống)
- [Phụ lục: Bảng LED Pattern](#phụ-lục-a-bảng-led-pattern)
- [Phụ lục: RTC Memory Layout](#phụ-lục-b-rtc-memory-layout)

---

## Phase 0: Setup môi trường

### 0.1 Cài đặt công cụ

| Bước | Hành động | Ghi chú |
|------|-----------|---------|
| 0.1.1 | Cài PlatformIO (VSCode extension) hoặc Arduino IDE 2.x | PlatformIO khuyến nghị |
| 0.1.2 | Cài ESP32 Arduino Core ≥ 3.0 | Board Manager URL: `https://espressif.github.io/arduino-esp32/package_esp32_index.json` |
| 0.1.3 | Cài USB driver CH340 hoặc CP210x | Tùy module SuperMini |
| 0.1.4 | Kiểm tra kết nối: cắm USB → Serial port xuất hiện | |

### 0.2 Tạo cấu trúc project

```
firmware/
├── src/
│   ├── main.cpp                # Entry point, state machine
│   ├── config.h                # Hằng số, GPIO, timing
│   ├── wifi_manager.h
│   ├── wifi_manager.cpp
│   ├── mqtt_manager.h
│   ├── mqtt_manager.cpp
│   ├── led_controller.h
│   ├── led_controller.cpp
│   ├── motor_controller.h
│   ├── motor_controller.cpp
│   ├── power_manager.h
│   ├── power_manager.cpp
│   ├── button_handler.h
│   ├── button_handler.cpp
│   ├── ntp_time.h
│   ├── ntp_time.cpp
│   └── ota_updater.h
│   └── ota_updater.cpp
├── data/
│   └── index.html              # Captive Portal HTML
├── platformio.ini
└── README.md
```

### 0.3 Cấu hình PlatformIO (`platformio.ini`)

```ini
[env:esp32c3]
platform = espressif32
board = esp32-c3-devkitm-1
framework = arduino
monitor_speed = 115200
upload_speed = 921600
board_build.partitions = min_spiffs.csv  ; OTA support
lib_deps =
    knolleary/PubSubClient@^2.8
    adafruit/Adafruit NeoPixel@^1.12
build_flags =
    -DARDUINO_USB_MODE=1
    -DARDUINO_USB_CDC_ON_BOOT=1
```

### 0.4 Test board cơ bản

```cpp
// Sketch: Hello World — xác nhận board hoạt động
void setup() {
    Serial.begin(115200);
    delay(2000);
    Serial.println("Smart Couple Coin - Board OK!");
    Serial.printf("Chip: %s, Rev: %d\n", ESP.getChipModel(), ESP.getChipRevision());
    Serial.printf("Flash: %d MB\n", ESP.getFlashChipSize() / 1024 / 1024);
}
void loop() { delay(1000); }
```

**✅ PASS**: Serial monitor hiển thị thông tin chip\
**❌ FAIL**: Kiểm tra USB driver, board selection, upload speed

---

## Phase 1: Foundation — Phần cứng cơ bản

> **Mục tiêu**: Mỗi ngoại vi (LED, motor, nút, ADC, sleep) hoạt động đúng riêng lẻ.\
> **Không cần WiFi/MQTT ở phase này.**

### 1.1 `config.h` — Pin Definitions & Constants

**File**: `src/config.h`

```cpp
#ifndef CONFIG_H
#define CONFIG_H

// ====== FIRMWARE VERSION ======
#define FW_VERSION      "1.0.0"

// ====== GPIO PINS ======
#define PIN_BUTTON      0       // GPIO0 - Nút bấm (Active LOW, pull-up 47kΩ)
#define PIN_SW_LED1     1       // GPIO1 - Switch nguồn LED 1 (HIGH = ON)
#define PIN_ADC_BAT     3       // GPIO3 - ADC đo pin (VBAT/2)
#define PIN_LED1_DATA   4       // GPIO4 - Data SK6812 LED 1
#define PIN_LED2_DATA   6       // GPIO6 - Data SK6812 LED 2
#define PIN_MOTOR       7       // GPIO7 - Motor rung (HIGH = ON)
#define PIN_SW_LED2     10      // GPIO10 - Switch nguồn LED 2 (HIGH = ON)

// ====== TIMING (ms) ======
#define DEBOUNCE_MS             50
#define ANTI_SPAM_MS            60000       // 1 phút
#define FACTORY_RESET_MS        10000       // 10 giây
#define OTA_TRIGGER_MS          20000       // 20 giây
#define AP_TIMEOUT_MS           300000      // 5 phút
#define OTA_TIMEOUT_MS          300000      // 5 phút
#define WIFI_TIMEOUT_MS         10000       // 10 giây
#define MQTT_RETRY_MS           2000        // 2 giây/retry
#define MQTT_MAX_RETRIES        2
#define MQTT_WAIT_MSG_MS        500         // Đợi retained message
#define WDT_TIMEOUT_S           15          // Watchdog 15 giây

// ====== WAKE SCHEDULE ======
#define WAKE_INTERVAL_DAY_US    (5ULL * 60 * 1000000)   // 5 phút (µs)
#define WAKE_INTERVAL_NIGHT_US  (30ULL * 60 * 1000000)  // 30 phút (µs)
#define DAY_START_HOUR          7
#define DAY_END_HOUR            23

// ====== LED EFFECTS (ms) ======
#define LED_SIGNAL_DURATION_MS  3000        // Nháy LED 3 giây khi nhận
#define MOTOR_DURATION_MS       1000        // Motor rung 1 giây
#define MOTOR_ON_MS             400         // PWM ON 400ms
#define MOTOR_OFF_MS            100         // PWM OFF 100ms
#define MOTOR_PWM_DUTY          204         // 80% of 255

// ====== BATTERY THRESHOLDS ======
#define VBAT_FULL               4.2f
#define VBAT_WARNING            3.3f
#define VBAT_CUTOFF             3.0f

// ====== MQTT CONFIG ======
#define MQTT_BROKER     "xxxxxxx.s1.eu.hivemq.cloud"
#define MQTT_PORT       8883
#define MQTT_USER       "your_username"
#define MQTT_PASS       "your_password"

// ====== DEVICE CONFIG (chốt khi nạp firmware) ======
#define PAIR_ID         "couple001"
#define DEVICE_ID       "A"         // "A" hoặc "B"

// ====== NTP ======
#define NTP_SERVER      "pool.ntp.org"
#define NTP_TIMEZONE    "ICT-7"     // UTC+7 mặc định

// ====== PENDING SIGNAL ======
#define PENDING_EXPIRY_HOURS    24

#endif
```

**Test**: Biên dịch thành công, không lỗi\
**Tiêu chí**: ✅ File include không lỗi, tất cả constant đúng giá trị theo spec

---

### 1.2 `button_handler` — Nút bấm + Debounce + Long Press

**File**: `src/button_handler.h` + `src/button_handler.cpp`

**API**:
```cpp
void     buttonInit();
bool     buttonIsPressed();          // Trả về true nếu bấm hợp lệ (sau debounce)
uint32_t buttonGetHoldTime();        // Thời gian giữ nút (ms)
bool     buttonIsFactoryReset();     // Giữ ≥ 10s
bool     buttonIsOTATrigger();       // Giữ ≥ 20s
```

**Logic chi tiết**:
1. `pinMode(PIN_BUTTON, INPUT)` — pull-up 47kΩ bên ngoài
2. Đọc `digitalRead(PIN_BUTTON)` — LOW = đang bấm
3. Debounce: chỉ xác nhận sau 50ms liên tục ở mức LOW
4. Đo hold time: từ lúc bắt đầu bấm đến lúc thả (hoặc timeout)
5. Phân biệt 4 mức:
   - **Bấm ngắn** (< 3s) → Gửi tín hiệu
   - **Bấm giữ 2-3s** → Vẫn tính là bấm ngắn (1 lần gửi)
   - **Bấm giữ 10s** → Factory Reset
   - **Bấm giữ 20s** → OTA Mode

**Test**:
```
Upload → Mở Serial Monitor → Bấm nút:
  - Bấm ngắn → "SHORT_PRESS"
  - Giữ 3s  → "SHORT_PRESS" (không khác)
  - Giữ 10s → "FACTORY_RESET"  + LED nháy nhanh từ 3s
  - Giữ 20s → "OTA_MODE" + LED xanh dương chậm từ 15s
```

**✅ PASS**: Phân biệt đúng 4 mức, debounce ổn định\
**❌ FAIL**: Kiểm tra R6 (47kΩ), C8 (100nF), hàn nút bấm

---

### 1.3 `led_controller` — LED RGB SK6812

**File**: `src/led_controller.h` + `src/led_controller.cpp`

**API**:
```cpp
void ledInit();
void ledPowerOn();                      // Bật SW_LED1 + SW_LED2
void ledPowerOff();                     // Tắt SW_LED1 + SW_LED2
void ledBlink(uint8_t r, uint8_t g, uint8_t b, int durationMs, int intervalMs);
void ledBlinkOnce(uint8_t r, uint8_t g, uint8_t b, int durationMs);  // 1 lần
void ledOff();                          // Tắt pixel data
void ledSetBrightness(uint8_t b);       // 0-255
```

**Logic chi tiết**:
1. Dùng `Adafruit_NeoPixel` với `NEO_GRB + NEO_KHZ800` (SK6812 RGB 3 kênh)
2. **Trước khi sáng**: bật `PIN_SW_LED1` + `PIN_SW_LED2` → HIGH, delay 10ms
3. **Sau khi xong**: `ledOff()` → set pixel = 0 → `ledPowerOff()` → LOW
4. Brightness mặc định: 50% (128) — "sáng vừa phải"
5. Hai LED instance riêng biệt (mỗi LED 1 pin data, 1 pixel)

**Test** (giống assembly-and-testing.md Test 3.1):
```
Đỏ 1s → Xanh lá 1s → Xanh dương 1s → Tắt 1s → Lặp
Cả 2 LED phải sáng đồng thời đúng màu.
```

**✅ PASS**: Cả 2 LED R/G/B đúng, sáng vừa phải\
**❌ FAIL**: Kiểm tra SW_LEDx, Q2-Q5, R7/R8, SK6812 hướng chân

---

### 1.4 `motor_controller` — Motor rung

**File**: `src/motor_controller.h` + `src/motor_controller.cpp`

**API**:
```cpp
void motorInit();
void motorBuzz(int durationMs);     // Pattern: ON 400ms (80%) → OFF 100ms → lặp
void motorStop();
```

**Logic chi tiết**:
1. Dùng `ledcAttach(PIN_MOTOR, 1000, 8)` — 1kHz, 8-bit PWM
2. Pattern: `ledcWrite(PIN_MOTOR, 204)` (80% duty) → delay 400ms → `ledcWrite(PIN_MOTOR, 0)` → delay 100ms
3. Lặp pattern cho đến khi hết `durationMs` (mặc định 1000ms → ~2 chu kỳ)

**Test** (giống assembly-and-testing.md Test 3.2):
```
Motor rung 1 giây → nghỉ 2 giây → lặp
Cảm nhận: rung vừa phải, pattern rõ ràng
```

**✅ PASS**: Motor rung đúng pattern, cường độ vừa\
**❌ FAIL**: Kiểm tra Q1, R9, R10, SS14, dây motor

---

### 1.5 `power_manager` — ADC Pin + Deep Sleep

**File**: `src/power_manager.h` + `src/power_manager.cpp`

**API**:
```cpp
void     powerInit();
float    powerReadVBAT();                   // Đo ADC → trả về VBAT (V)
uint8_t  powerGetBatPercent();              // VBAT → % (0-100)
bool     powerIsBatteryLow();               // VBAT ≤ 3.3V
bool     powerIsBatteryCritical();          // VBAT ≤ 3.0V
void     powerDeepSleep(uint64_t intervalUs); // Sleep với timer
void     powerDeepSleepForever();           // Sleep vĩnh viễn (chỉ wake khi sạc/reset)
bool     powerIsWakeByButton();             // True nếu wake bởi GPIO0
bool     powerIsWakeByTimer();              // True nếu wake bởi RTC timer
```

**Logic chi tiết**:

**ADC đo pin**:
1. `analogReadResolution(12)` — 12-bit (0-4095)
2. `int raw = analogRead(PIN_ADC_BAT)`
3. `float vbat = raw * (3.3 / 4095.0) * 2.0` — cầu chia áp 1M/1M
4. Lọc nhiễu: đọc 16 lần, bỏ min/max, lấy trung bình 14 giá trị

**Deep Sleep**:
1. Tắt LED switch: `digitalWrite(PIN_SW_LED1, LOW); digitalWrite(PIN_SW_LED2, LOW)`
2. Kéo data LED về LOW: `digitalWrite(PIN_LED1_DATA, LOW); digitalWrite(PIN_LED2_DATA, LOW)`
3. `esp_sleep_enable_timer_wakeup(intervalUs)`
4. `esp_deep_sleep_enable_gpio_wakeup(1 << PIN_BUTTON, ESP_GPIO_WAKEUP_GPIO_LOW)`
5. `esp_deep_sleep_start()`

**Wake source detection**:
```cpp
esp_sleep_wakeup_cause_t cause = esp_sleep_get_wakeup_cause();
// cause == ESP_SLEEP_WAKEUP_TIMER  → timer wake
// cause == ESP_SLEEP_WAKEUP_GPIO   → button wake
```

**Test A — ADC** (giống assembly-and-testing.md Test 2.3):
```
Đọc VBAT mỗi 1 giây → so sánh với multimeter → sai số ≤ 0.1V
```

**Test B — Deep Sleep + Timer** (giống assembly-and-testing.md Test 3.4):
```
Sleep 10 giây → tự wake → in wake cause → sleep lại
Đo dòng khi sleep: ≤ 50µA
```

**Test C — Deep Sleep + Button** (giống assembly-and-testing.md Test 3.3):
```
Sleep → bấm nút → wake → in wake cause = GPIO
```

**✅ PASS**: ADC chính xác ±0.1V, sleep current ≤ 50µA, cả 2 wake source đúng\
**❌ FAIL**: Kiểm tra R2/R4, C5, GPIO0 wakeup

---

### 1.6 Milestone 1 — Checkpoint

**Trước khi sang Phase 2, xác nhận:**

- [ ] `config.h` biên dịch không lỗi
- [ ] Nút bấm: debounce ổn định, phân biệt short/10s/20s
- [ ] LED: cả 2 LED sáng R/G/B đúng, tắt sạch
- [ ] Motor: rung đúng pattern 400ms/100ms
- [ ] ADC: VBAT chính xác ±0.1V
- [ ] Deep sleep: dòng ≤ 50µA
- [ ] Wake by timer: OK
- [ ] Wake by button: OK

---

## Phase 2: Connectivity — Kết nối mạng

> **Mục tiêu**: WiFi, MQTT, NTP hoạt động đúng. Gửi/nhận message qua HiveMQ.\
> **Cần WiFi nhà + account HiveMQ Cloud.**

### 2.1 `wifi_manager` — WiFi Connect (STA mode)

**File**: `src/wifi_manager.h` + `src/wifi_manager.cpp`

**API**:
```cpp
bool wifiConnect(const char* ssid, const char* password, uint32_t timeoutMs);
void wifiDisconnect();
bool wifiIsConnected();
```

**Logic chi tiết**:
1. `WiFi.mode(WIFI_STA)`
2. `WiFi.begin(ssid, password)`
3. Loop chờ `WiFi.status() == WL_CONNECTED` với timeout 10s
4. Nếu timeout → return false
5. Ghi lại thời gian kết nối (cho power budget)

**Test** (giống assembly-and-testing.md Test 4.1):
```
Hardcode SSID/pass → connect → in IP + thời gian kết nối
Target: < 5 giây
```

**✅ PASS**: Kết nối WiFi < 5 giây\
**❌ FAIL**: Sai SSID/pass, router không 2.4GHz

---

### 2.2 `mqtt_manager` — MQTT TLS + Publish/Subscribe

**File**: `src/mqtt_manager.h` + `src/mqtt_manager.cpp`

**API**:
```cpp
bool     mqttConnect();
void     mqttDisconnect();
bool     mqttPublishSignal();                    // Publish tín hiệu bấm nút
bool     mqttCheckRetainedSignal();              // Subscribe + đợi retained msg
void     mqttClearRetained();                    // Publish empty payload với retain=true
void     mqttSetCallback(MQTT_CALLBACK_FUNC);    // Đặt callback nhận message
String   mqttGetTopic();                         // "couplekey/{pair_id}/signal"
```

**Logic chi tiết**:

**Kết nối HiveMQ Cloud (TLS)**:
```cpp
WiFiClientSecure espClient;
PubSubClient mqtt(espClient);
espClient.setInsecure();  // Skip cert verify (Phase 1)
mqtt.setServer(MQTT_BROKER, MQTT_PORT);
mqtt.setKeepAlive(15);    // Ngắn vì chỉ kết nối tạm
```

**Publish signal message**:
```json
{
  "from": "A",
  "ts": 1234567890,
  "bat": 85
}
```
- QoS 1, retain = true
- Dùng `ArduinoJson` hoặc `snprintf` để tạo JSON

**Subscribe + Check retained**:
1. `mqtt.subscribe(topic, 1)` — QoS 1
2. `mqtt.loop()` trong 500ms
3. Callback: kiểm tra `from` ≠ DEVICE_ID → có tín hiệu mới
4. Clear retained: `mqtt.publish(topic, "", true)` — payload rỗng, retain=true

**Test A — Publish**:
```
Kết nối WiFi → MQTT → Publish 1 message → Kiểm tra trên HiveMQ web console
```

**Test B — Subscribe + Retained**:
```
Từ HiveMQ console: publish retained message {"from":"B","ts":0,"bat":100}
ESP32 subscribe → nhận → in ra Serial → clear retained
```

**✅ PASS**: Publish thành công, nhận retained đúng, clear OK\
**❌ FAIL**: Kiểm tra broker URL, port 8883, username/password, TLS

---

### 2.3 `ntp_time` — NTP Sync + Wake Schedule

**File**: `src/ntp_time.h` + `src/ntp_time.cpp`

**API**:
```cpp
bool     ntpSync();                     // Đồng bộ NTP (cần WiFi)
int      ntpGetCurrentHour();           // Giờ hiện tại (0-23)
bool     ntpIsDaytime();                // 7:00-23:00 = true
uint64_t ntpGetNextWakeInterval();      // 5 min hoặc 30 min (µs)
void     ntpSaveToRTC();                // Lưu time vào RTC memory
void     ntpRestoreFromRTC();           // Khôi phục time từ RTC memory
bool     ntpNeedResync();               // True nếu > 24h chưa sync
```

**Logic chi tiết**:
1. `configTzTime(NTP_TIMEZONE, NTP_SERVER)` — ESP32 built-in SNTP
2. Chờ `getLocalTime(&timeinfo)` thành công (timeout 5s)
3. Lưu epoch timestamp vào RTC memory: `RTC_DATA_ATTR uint32_t rtcEpoch`
4. Mỗi lần wake: `rtcEpoch += sleepDurationSeconds`
5. Nếu `ntpNeedResync()` (>24h) → sync lại qua NTP

**RTC Memory cho NTP**:
```cpp
RTC_DATA_ATTR uint32_t  rtcLastNtpSync = 0;     // Epoch lần sync cuối
RTC_DATA_ATTR uint32_t  rtcCurrentEpoch = 0;     // Epoch hiện tại (ước tính)
RTC_DATA_ATTR uint32_t  rtcSleepDuration = 0;    // Thời gian sleep lần này (s)
```

**Test**:
```
Kết nối WiFi → NTP sync → In giờ hiện tại → Sleep 10s → Wake → In giờ (phải +10s)
Kiểm tra: isDaytime() đúng theo giờ thực
```

**✅ PASS**: Giờ chính xác ±2 giây, isDaytime() đúng\
**❌ FAIL**: NTP server blocked, timezone sai

---

### 2.4 Milestone 2 — Checkpoint

- [ ] WiFi connect < 5 giây
- [ ] MQTT TLS kết nối HiveMQ Cloud thành công
- [ ] Publish message → thấy trên HiveMQ console
- [ ] Subscribe + nhận retained message đúng
- [ ] Clear retained message thành công
- [ ] NTP sync đúng giờ
- [ ] isDaytime() trả về đúng
- [ ] Wake interval chuyển đổi 5min ↔ 30min đúng

---

## Phase 3: Configuration — Captive Portal

> **Mục tiêu**: Người dùng cấu hình WiFi + màu LED qua web browser.

### 3.1 `wifi_manager` (bổ sung) — AP Mode + Captive Portal

**API bổ sung**:
```cpp
void wifiStartAP();                                 // Phát WiFi AP
void wifiStartCaptivePortal();                      // DNS + WebServer
void wifiHandleCaptivePortalClient();               // Loop xử lý request
bool wifiPortalHasConfig();                         // True nếu user đã submit
void wifiStopAP();                                  // Tắt AP
```

**Logic chi tiết**:

**AP Mode**:
1. `WiFi.mode(WIFI_AP)`
2. `WiFi.softAP("CoupleKey_XXXX")` — XXXX = 4 ký tự cuối MAC
3. `WiFi.softAPIP()` → thường là `192.168.4.1`

**Captive Portal**:
1. `DNSServer` chuyển mọi DNS request → `192.168.4.1`
2. `WebServer` phục vụ trang HTML trên port 80
3. Routes:
   - `GET /` → trả file `index.html`
   - `GET /scan` → JSON danh sách WiFi
   - `POST /save` → nhận SSID, password, r, g, b → lưu Preferences → restart
4. **Timeout 5 phút**: nếu không có request → deep sleep

### 3.2 `data/index.html` — Web UI

**Nội dung trang**:
1. **Header**: "Smart Couple Coin" + firmware version (v1.0.0)
2. **WiFi Selection**: Dropdown scan WiFi + ô nhập password
3. **Color Picker**: Input type="color" hoặc 3 slider R/G/B (0-255)
4. **Info (chỉ đọc)**: Pair ID, Device ID
5. **Nút Lưu**: POST → `/save`
6. **Style**: Responsive, tối ưu mobile, gọn gàng

> **Lưu ý**: HTML/CSS/JS inline trong 1 file (lưu SPIFFS), tối đa ~10KB.

**Test**:
```
1. Flash firmware AP mode
2. Kết nối WiFi "CoupleKey_XXXX" trên điện thoại
3. Trang web tự hiện → Nhập WiFi + chọn màu → Lưu
4. Thiết bị restart → kết nối WiFi nhà thành công
5. LED nháy xanh 2 lần → sẵn sàng
```

**✅ PASS**: Captive Portal hiện đúng, lưu config, restart kết nối WiFi OK\
**❌ FAIL**: DNS redirect không hoạt động, HTML không load

---

### 3.3 Config Storage — `Preferences`

**API** (tích hợp trong wifi_manager hoặc module riêng):
```cpp
void     configSave(String ssid, String pass, uint8_t r, uint8_t g, uint8_t b);
bool     configLoad();           // Đọc config từ flash
bool     configHasWiFi();        // True nếu đã có SSID
void     configClear();          // Xóa toàn bộ (factory reset)
String   configGetSSID();
String   configGetPass();
uint8_t  configGetColorR();
uint8_t  configGetColorG();
uint8_t  configGetColorB();
```

**Keys trong Preferences**:
| Key | Type | Mô tả |
|-----|------|-------|
| `ssid` | String | WiFi SSID |
| `pass` | String | WiFi password |
| `led_r` | uint8_t | Màu LED Red (0-255) |
| `led_g` | uint8_t | Màu LED Green (0-255) |
| `led_b` | uint8_t | Màu LED Blue (0-255) |

> **Pair ID** và **Device ID** KHÔNG lưu Preferences — hardcode trong `config.h`.

**Test**:
```
Lưu config → restart → đọc lại → đúng giá trị
Factory reset → đọc → config trống → vào AP Mode
```

---

### 3.4 Factory Reset

**Logic** (tích hợp trong `button_handler` + `main.cpp`):

```
Khi wake (bất kỳ nguồn nào):
  1. Đọc nút bấm
  2. Nếu vẫn đang giữ:
     - 3s → LED nháy nhanh (cảnh báo)
     - 10s → configClear() + LED trắng 3 lần + restart
     - 15s → LED xanh dương chậm (cảnh báo OTA)
     - 20s → vào OTA Mode
  3. Nếu thả trước 3s → bấm ngắn (gửi tín hiệu)
```

**Test**:
```
1. Cấu hình WiFi qua Portal → hoạt động bình thường
2. Bấm giữ 10s → LED nháy trắng 3 lần → restart vào AP Mode
3. Bấm giữ 20s → LED xanh dương → vào OTA (đợi firmware)
```

---

### 3.5 Milestone 3 — Checkpoint

- [ ] AP Mode phát WiFi "CoupleKey_XXXX"
- [ ] Captive Portal hiện trang web trên điện thoại
- [ ] Scan WiFi + hiển thị danh sách
- [ ] Nhập password + chọn màu + Lưu → restart thành công
- [ ] Config persist qua restart
- [ ] Factory Reset (10s) xóa config, vào AP Mode
- [ ] AP Mode timeout 5 phút → deep sleep

---

## Phase 4: Integration — State Machine

> **Mục tiêu**: Ghép tất cả module thành firmware hoàn chỉnh theo spec state machine.

### 4.1 `main.cpp` — State Machine

```
┌─────────────────────────────────────────────────────────────┐
│                        setup()                              │
│  1. Init tất cả module                                      │
│  2. Enable WDT (15s)                                        │
│  3. Đọc wake source                                         │
│  4. Kiểm tra config WiFi                                    │
│     ├── Không có config → AP MODE (goto 5)                  │
│     └── Có config → WAKE FLOW (goto 6)                      │
│                                                              │
│  5. AP MODE:                                                 │
│     └── startCaptivePortal() → timeout 5min → deep sleep    │
│                                                              │
│  6. WAKE FLOW:                                               │
│     a. Kiểm tra nút bấm (long press?)                       │
│        ├── 10s → Factory Reset                               │
│        ├── 20s → OTA Mode                                    │
│        └── < 3s → tiếp tục                                   │
│     b. Đo ADC → kiểm tra pin                                │
│        ├── ≤ 3.0V → deep sleep vĩnh viễn                    │
│        └── > 3.0V → tiếp tục                                │
│     c. Kết nối WiFi (timeout 10s)                            │
│        ├── Fail → LED đỏ 2 lần → deep sleep                 │
│        └── OK → tiếp tục                                     │
│     d. NTP sync (nếu cần)                                    │
│     e. Kết nối MQTT (retry 2 lần)                            │
│        ├── Fail → LED đỏ 2 lần → deep sleep                 │
│        └── OK → tiếp tục                                     │
│     f. Nếu wake bởi BUTTON:                                  │
│        ├── Kiểm tra anti-spam (1 phút)                       │
│        │   ├── Spam → LED vàng 1 lần                         │
│        │   └── OK → Publish signal + LED xanh 1 lần          │
│        └── Gửi pending signal nếu có                         │
│     g. Subscribe + check retained message (500ms)            │
│        ├── Có signal (from ≠ mình):                          │
│        │   ├── VBAT > 3.3V → LED 3s + Motor 1s              │
│        │   └── VBAT ≤ 3.3V → LED đỏ 3s (không motor)        │
│        │   └── Clear retained message                        │
│        └── Không có → bỏ qua                                │
│     h. Disconnect WiFi + MQTT                                │
│     i. Tính timer tiếp theo (5min/30min)                     │
│     j. Deep sleep                                            │
│                                                              │
│  loop() → KHÔNG BAO GIỜ CHẠY (deep sleep reset từ setup)    │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 RTC Memory — Anti-spam + Pending Signal

```cpp
RTC_DATA_ATTR uint32_t rtcLastSendEpoch = 0;      // Epoch lần gửi cuối
RTC_DATA_ATTR bool     rtcPendingSignal = false;   // Có tín hiệu chờ gửi
RTC_DATA_ATTR uint32_t rtcPendingEpoch = 0;        // Epoch tạo pending signal
RTC_DATA_ATTR uint32_t rtcLastNtpSync = 0;         // Epoch lần NTP sync cuối
RTC_DATA_ATTR uint32_t rtcCurrentEpoch = 0;        // Epoch ước tính hiện tại
RTC_DATA_ATTR uint32_t rtcBootCount = 0;            // Đếm số lần wake
```

**Anti-spam logic**:
```cpp
bool canSend = (rtcCurrentEpoch - rtcLastSendEpoch) >= 60;  // 1 phút
if (canSend) {
    mqttPublishSignal();
    rtcLastSendEpoch = rtcCurrentEpoch;
}
```

**Pending signal logic**:
```cpp
// Khi gửi thất bại:
rtcPendingSignal = true;
rtcPendingEpoch = rtcCurrentEpoch;

// Khi wake thành công:
if (rtcPendingSignal) {
    if (rtcCurrentEpoch - rtcPendingEpoch > 24 * 3600) {
        rtcPendingSignal = false;  // Hết hạn 24h
    } else {
        mqttPublishSignal();       // Gửi lại
        rtcPendingSignal = false;
    }
}
```

### 4.3 LED Status Patterns

| Sự kiện | Pattern LED | Thời gian |
|---------|-------------|-----------|
| Đã gửi tín hiệu | Xanh lá 1 lần | 300ms |
| Nhận tín hiệu | Màu user chọn, nháy | 3000ms |
| Anti-spam (bấm quá sớm) | Vàng 1 lần | 300ms |
| WiFi/MQTT thất bại | Đỏ 2 lần | 200ms × 2 |
| Gửi thất bại (mất mạng) | Đỏ 3 lần | 200ms × 3 |
| Pin yếu (3.0-3.3V) | Đỏ chậm | 3000ms |
| Factory reset (đang giữ) | Trắng nháy nhanh | 100ms interval |
| Factory reset (hoàn thành) | Trắng 3 lần | 500ms × 3 |
| OTA mode (sắp vào) | Xanh dương chậm | 500ms interval |
| OTA mode (đang chờ) | Xanh dương liên tục | — |
| OTA thành công | Xanh lá 5 lần | 300ms × 5 |
| AP Mode (đang chờ config) | Trắng nháy | 1000ms interval |

---

### 4.4 Milestone 4 — Checkpoint

- [ ] State machine chạy đúng flow: Boot → Check config → WiFi → MQTT → Sleep
- [ ] Wake by timer → check retained → sleep (không gửi)
- [ ] Wake by button → publish + check → sleep
- [ ] Anti-spam: bấm 2 lần trong < 1 phút → lần 2 bị chặn (LED vàng)
- [ ] Pending signal: mất mạng → lưu RTC → wake sau gửi lại
- [ ] Pending signal: hết hạn 24h → xóa
- [ ] Wake schedule: 5 phút ban ngày, 30 phút ban đêm
- [ ] NTP resync mỗi 24h
- [ ] Pin yếu (3.0-3.3V): LED đỏ, không motor
- [ ] Pin cạn (≤3.0V): deep sleep vĩnh viễn
- [ ] LED patterns đúng cho mọi trạng thái

---

## Phase 5: Polish — Hoàn thiện

### 5.1 `ota_updater` — OTA Update Mode

**API**:
```cpp
void otaStart();     // Kết nối WiFi + ArduinoOTA + chờ firmware
```

**Logic**:
1. Bấm giữ 20s → gọi `otaStart()`
2. Kết nối WiFi đã lưu
3. `ArduinoOTA.begin()`
4. LED xanh dương nháy liên tục
5. Loop `ArduinoOTA.handle()` trong tối đa 5 phút
6. Nếu nhận firmware → LED xanh lá 5 lần → restart
7. Nếu timeout → restart bình thường

**Test**:
```
1. Giữ nút 20s → vào OTA
2. Từ PlatformIO/Arduino: Upload OTA → firmware cập nhật thành công
3. Kiểm tra: Serial in version mới
```

### 5.2 Watchdog Timer

```cpp
#include "esp_task_wdt.h"

void setup() {
    esp_task_wdt_config_t wdt_config = {
        .timeout_ms = WDT_TIMEOUT_S * 1000,
        .idle_core_mask = (1 << 0),
        .trigger_panic = true
    };
    esp_task_wdt_init(&wdt_config);
    esp_task_wdt_add(NULL);

    // Reset WDT tại mỗi bước quan trọng:
    esp_task_wdt_reset();
}
```

### 5.3 Power Optimization

| Hành động | Mục đích |
|-----------|----------|
| Tắt SW_LED1/SW_LED2 trước sleep | Ngắt nguồn LED (~1mA/LED) |
| Kéo LED data pin LOW trước sleep | Tránh dòng rò qua data line |
| `WiFi.disconnect(true); WiFi.mode(WIFI_OFF)` trước sleep | Tắt radio |
| Giảm thời gian WiFi connect (dùng static IP nếu có thể) | Giảm wake time |
| Đo dòng thực tế sau mỗi phase | Phát hiện sớm |

---

### 5.4 Milestone 5 — Checkpoint

- [ ] OTA update thành công qua WiFi (bấm 20s trigger)
- [ ] OTA timeout 5 phút → restart bình thường
- [ ] WDT reset khi firmware treo > 15 giây
- [ ] Dòng deep sleep thực tế ≤ 50µA
- [ ] Thời gian wake+connect+check ≤ 5 giây

---

## Phase 6: System Test — Kiểm thử hệ thống

> **Mục tiêu**: Test toàn bộ hệ thống với 2 thiết bị thật.

### 6.1 Test với 2 thiết bị

| # | Test case | Kỳ vọng | Kết quả |
|---|-----------|---------|---------|
| 1 | A bấm nút → B nhận | B: LED nháy + motor rung trong ≤ 5 phút | ☐ |
| 2 | B bấm nút → A nhận | A: LED nháy + motor rung trong ≤ 5 phút | ☐ |
| 3 | A bấm → B ngủ đêm | B nhận khi wake (≤ 30 phút) | ☐ |
| 4 | Cả 2 bấm gần nhau | Mỗi thiết bị đều nhận tín hiệu | ☐ |
| 5 | A bấm liên tục 5 lần | Chỉ gửi 1 lần, 4 lần sau LED vàng | ☐ |
| 6 | A bấm khi mất WiFi | LED đỏ 3 lần, pending signal lưu RTC | ☐ |
| 7 | WiFi khôi phục → A wake | Pending signal gửi lại thành công | ☐ |
| 8 | Pending signal > 24h | Signal bị xóa, không gửi | ☐ |
| 9 | Pin A = 3.2V | A nhận: LED đỏ, không motor | ☐ |
| 10 | Pin A = 2.9V | A deep sleep vĩnh viễn | ☐ |
| 11 | Factory reset A | A vào AP Mode, config cũ bị xóa | ☐ |
| 12 | OTA update A | Firmware A cập nhật, version mới | ☐ |
| 13 | Captive Portal A | Chọn WiFi + màu LED → hoạt động đúng | ☐ |
| 14 | AP Mode timeout 5p | Thiết bị vào deep sleep tự động | ☐ |

### 6.2 Stress Test (≥ 50 lần gửi/nhận)

```
Lặp 50 lần:
  A bấm → đợi 2 phút → B bấm → đợi 2 phút
  Ghi nhận: thành công/thất bại, thời gian nhận
```

**Target**: ≥ 95% success rate (≥ 48/50)

### 6.3 Battery Life Test

```
1. Sạc đầy 2 thiết bị (4.2V)
2. Bật cả 2, cấu hình WiFi
3. Mỗi ngày bấm ~15 lần mỗi thiết bị
4. Ghi nhận VBAT mỗi ngày (qua MQTT bat field)
5. Đo đến khi thiết bị tự tắt (≤ 3.0V)
```

**Target**: ≥ 5 ngày

### 6.4 Checklist trước khi đóng vỏ

- [ ] Tất cả 14 test case ở 6.1 PASS
- [ ] Stress test ≥ 95% success
- [ ] Thời lượng pin ≥ 5 ngày
- [ ] Dòng deep sleep ≤ 50µA
- [ ] Thời gian wake+connect+check ≤ 5 giây
- [ ] Factory reset hoạt động
- [ ] OTA update thành công ít nhất 1 lần
- [ ] Captive Portal hoạt động trên iOS + Android
- [ ] LED pattern đúng cho mọi trạng thái
- [ ] Hàn dây pin/motor/công tắc chắc chắn
- [ ] Vệ sinh PCB sạch flux

---

## Phụ lục A: Bảng LED Pattern

| Pattern | LED1 | LED2 | Màu | Duration | Interval |
|---------|------|------|-----|----------|----------|
| Nhận tín hiệu | ON | ON | User RGB | 3000ms | 300ms blink |
| Đã gửi | ON | ON | Xanh lá (0,255,0) | 300ms | — |
| Anti-spam | ON | ON | Vàng (255,200,0) | 300ms | — |
| WiFi/MQTT fail | ON | ON | Đỏ (255,0,0) | 200ms | 200ms × 2 |
| Gửi fail (mất mạng) | ON | ON | Đỏ (255,0,0) | 200ms | 200ms × 3 |
| Pin yếu | ON | ON | Đỏ (255,0,0) | 3000ms | 500ms blink |
| Factory reset (giữ) | ON | ON | Trắng (255,255,255) | — | 100ms blink |
| Factory reset (xong) | ON | ON | Trắng (255,255,255) | 500ms | 500ms × 3 |
| OTA (chuẩn bị) | ON | ON | Xanh dương (0,0,255) | — | 500ms blink |
| OTA (đang chờ) | ON | ON | Xanh dương (0,0,255) | — | ON liên tục |
| OTA (thành công) | ON | ON | Xanh lá (0,255,0) | 300ms | 300ms × 5 |
| AP Mode | ON | ON | Trắng (255,255,255) | — | 1000ms blink |

---

## Phụ lục B: RTC Memory Layout

```cpp
// ====== RTC Memory (persist qua deep sleep, mất khi power off) ======
RTC_DATA_ATTR uint32_t rtcBootCount       = 0;     // Tổng số lần wake
RTC_DATA_ATTR uint32_t rtcLastSendEpoch   = 0;     // Epoch lần gửi cuối (anti-spam)
RTC_DATA_ATTR bool     rtcPendingSignal   = false;  // Có tín hiệu chờ gửi
RTC_DATA_ATTR uint32_t rtcPendingEpoch    = 0;     // Epoch tạo pending signal
RTC_DATA_ATTR uint32_t rtcLastNtpSync     = 0;     // Epoch lần NTP sync cuối
RTC_DATA_ATTR uint32_t rtcCurrentEpoch    = 0;     // Epoch hiện tại (ước tính)
RTC_DATA_ATTR uint32_t rtcSleepDuration   = 0;     // Thời gian sleep (giây)

// Tổng: ~28 bytes RTC memory (ESP32-C3 có 8KB RTC SRAM)
```

---

## Phụ lục C: Thư viện Dependencies

| Thư viện | Version | Nguồn | Mục đích |
|----------|---------|-------|----------|
| `Adafruit NeoPixel` | ≥ 1.12 | PlatformIO lib | SK6812 LED control |
| `PubSubClient` | ≥ 2.8 | PlatformIO lib | MQTT client |
| `WiFi.h` | built-in | ESP32 Core | WiFi STA/AP |
| `WiFiClientSecure.h` | built-in | ESP32 Core | TLS cho MQTT |
| `WebServer.h` | built-in | ESP32 Core | HTTP server (Portal) |
| `DNSServer.h` | built-in | ESP32 Core | DNS redirect (Portal) |
| `Preferences.h` | built-in | ESP32 Core | Flash storage |
| `esp_sntp.h` | built-in | ESP-IDF | NTP sync |
| `ArduinoOTA.h` | built-in | Arduino Core | OTA update |
| `esp_task_wdt.h` | built-in | ESP-IDF | Watchdog timer |

---

## Ước tính thời gian

| Phase | Nội dung | Thời gian ước tính |
|-------|----------|-------------------|
| **Phase 0** | Setup môi trường | 0.5 ngày |
| **Phase 1** | Foundation (LED, motor, button, ADC, sleep) | 2-3 ngày |
| **Phase 2** | Connectivity (WiFi, MQTT, NTP) | 2-3 ngày |
| **Phase 3** | Captive Portal + Config | 2-3 ngày |
| **Phase 4** | Integration (state machine) | 3-4 ngày |
| **Phase 5** | Polish (OTA, WDT, power opt) | 1-2 ngày |
| **Phase 6** | System Test | 2-3 ngày (+5 ngày pin test) |
| **Tổng** | | **~13-19 ngày** |

---

> **Ghi chú cuối**: Plan này dựa trên spec v1.1 đã chốt. Nếu có thay đổi spec, cập nhật plan tương ứng. Mỗi milestone phải PASS hết trước khi sang phase tiếp theo.
