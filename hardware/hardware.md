# Sơ đồ Phần Cứng & Phân Bổ Chân I/O - Smart Couple Coin

---

## I. Bảng Phân Bổ I/O (ESP32-C3 SuperMini)


| GPIO ESP32-C3 | Tên tín hiệu | Chiều        | Chức năng phần cứng            | Mức hoạt động / Ghi chú                     |
| --------------:| --------------| --------------| --------------------------------| ---------------------------------------------|
| GPIO0         | GPIO_BT      | Input        | Nút bấm chính                  | Nhấn = LOW, thả = HIGH; có pull-up lên 3V3  |
| GPIO1         | SW_LED1      | Output       | Bật/tắt nguồn LED RGB 1        | HIGH = bật LED1, LOW = tắt LED1             |
| GPIO2         | NC           | -            | Không sử dụng                  | Nên để trống vì có thể liên quan boot/strap |
| GPIO3         | ADC_BAT      | Analog Input | Đo điện áp pin qua cầu chia áp | ADC_BAT = VBAT / 2                          |
| GPIO4         | LED1         | Output       | Data điều khiển SK6812 LED 1   | Xuất tín hiệu LED addressable               |
| GPIO5         | NC           | -            | Không sử dụng                  | Có thể dự phòng                             |
| GPIO6         | LED2         | Output       | Data điều khiển SK6812 LED 2   | Xuất tín hiệu LED addressable               |
| GPIO7         | MOTOR        | Output       | Điều khiển MOSFET motor rung   | HIGH = motor rung, LOW = motor tắt          |
| GPIO8         | NC           | -            | Không sử dụng                  | Nên để trống vì có thể liên quan boot/strap |
| GPIO9         | BOOT / NC    | Input / TP   | Chân BOOT dự phòng             | Nên kéo ra test pad BOOT, không nối tải     |
| GPIO10        | SW_LED2      | Output       | Bật/tắt nguồn LED RGB 2        | HIGH = bật LED2, LOW = tắt LED2             |
| GPIO20        | RXD0         | UART Input   | UART RX / nạp firmware / debug | Nên kéo ra test pad                         |
| GPIO21        | TXD0         | UART Output  | UART TX / nạp firmware / debug | Nên kéo ra test pad                         |
| 3V3           | 3V3          | Power        | Nguồn 3.3V từ LDO AP2112K      | Cấp cho ESP32-C3 và mạch logic              |
| GND           | GND          | Power        | Mass hệ thống                  | Nối chung toàn mạch                         |



## II. Bảng Chức Năng Các Khối Mạch


| STT | Khối mạch                    | Linh kiện / Net chính                                                  | Chức năng                                                                 | Nguyên lý hoạt động                                                                                | Ghi chú                                                |
| ----:| ------------------------------| ------------------------------------------------------------------------| ---------------------------------------------------------------------------| ----------------------------------------------------------------------------------------------------| --------------------------------------------------------|
| 1   | **USB Type-C Input**         | J1 USB-C, R1/R3 5.1k, `V_CHARGE`, GND                                  | Nhận nguồn 5V từ cổng USB-C để sạc pin                                    | Hai điện trở 5.1k kéo CC1/CC2 xuống GND để báo thiết bị là sink, nguồn USB cấp 5V vào `V_CHARGE`   | Không dùng data USB, chỉ dùng sạc                      |
| 2   | **Bảo vệ USB**               | D3 SMF5.0A                                                             | Bảo vệ chống xung ESD/quá áp trên đường USB 5V                            | TVS kẹp xung áp từ `V_CHARGE` xuống GND                                                            | Cathode/vạch diode nối `V_CHARGE`, anode nối GND       |
| 3   | **Sạc pin Li-Po**            | U1 LTC4054, R5 6.8k, C1/C4 10uF, `VBAT`                                | Sạc pin Li-Ion/Li-Po 1 cell 4.2V                                          | Nguồn 5V từ USB vào VCC của LTC4054, IC sạc pin qua chân BAT ra `VBAT`                             | R5 = 6.8k cho dòng sạc khoảng 140–150mA                |
| 4   | **LED báo sạc**              | CHG_LED, R12 2.2k, chân `CHRG`                                         | Báo trạng thái đang sạc                                                   | Khi LTC4054 kéo chân `CHRG` xuống thấp, dòng chạy từ `V_CHARGE` qua R12 và LED xuống IC → LED sáng | Nên đặt tên net là `CHG_LED` để tránh nhầm với LED RGB |
| 5   | **Pin / nguồn VBAT**         | BAT+, BAT-, `VBAT`, GND                                                | Cấp nguồn chính cho toàn bộ mạch                                          | Pin Li-Po cấp điện áp khoảng 3.0–4.2V vào net `VBAT`                                               | Nên dùng pin có mạch bảo vệ hoặc thêm mạch bảo vệ pin  |
| 6   | **Đo điện áp pin**           | R2 1M, R4 1M, C5 100nF, `ADC_BAT`                                      | Đưa điện áp pin về mức phù hợp cho ADC ESP32-C3                           | Cầu chia áp 1M/1M tạo `ADC_BAT = VBAT / 2`                                                         | Pin đầy 4.2V → ADC khoảng 2.1V                         |
| 7   | **LDO 3V3**                  | U2 AP2112K, C2 10uF, C3 10uF, `3V3`                                    | Tạo nguồn 3.3V cấp cho ESP32-C3 và logic                                  | AP2112K hạ `VBAT` xuống `3V3` ổn định                                                              | C2 đặt gần VIN, C3 đặt gần VOUT                        |
| 8   | **Công tắc nguồn ngoài**     | SW pad, R21 100k, R11 0R/DNP, `LDO_en`                                 | Cho phép bật/tắt LDO bằng công tắc ngoài                                  | Công tắc nối `VBAT` sang `LDO_en` để bật LDO; R21 kéo EN xuống GND khi công tắc hở                 | R11 là jumper tùy chọn, mặc định không hàn             |
| 9   | **MCU trung tâm**            | U3 ESP32-C3 SuperMini, `3V3`, GND                                      | Điều khiển toàn bộ thiết bị, kết nối Wi-Fi/BLE, xử lý nút bấm, LED, motor | ESP32-C3 đọc nút, đo pin, gửi/nhận tín hiệu qua mạng, điều khiển LED và motor                      | Chân 5V của module nên để hở nếu đã cấp nguồn qua 3V3  |
| 10  | **Test UART / Debug**        | RXD0, TXD0, TP                                                         | Dùng để nạp/debug firmware qua UART                                       | TX/RX của ESP32-C3 được kéo ra test pad                                                            | Nên thêm thêm TP: GND, 3V3, EN/RST, GPIO9/BOOT         |
| 11  | **Công tắc nguồn LED RGB 1** | Q3 AO3401A, Q5 AO3400, R15/R17/R19, `SW_LED1`, `V_LED1`                | Bật/tắt nguồn riêng cho LED RGB 1 để tiết kiệm pin                        | GPIO `SW_LED1` bật NMOS Q5, Q5 kéo gate PMOS Q3 xuống GND, PMOS cấp `VBAT` cho `V_LED1`            | HIGH tại `SW_LED1` → LED1 có nguồn                     |
| 12  | **Công tắc nguồn LED RGB 2** | Q2 AO3401A, Q4 AO3400, R16/R13/R18, `SW_LED2`, `V_LED2`                | Bật/tắt nguồn riêng cho LED RGB 2                                         | Tương tự LED1, GPIO `SW_LED2` điều khiển cặp NMOS + PMOS để cấp nguồn LED2                         | HIGH tại `SW_LED2` → LED2 có nguồn                     |
| 13  | **LED RGB SK6812 số 1**      | D1 SK6812MINI, R7 330R, R20 100k, C10 10uF, `LED1`                     | Hiển thị hiệu ứng sáng/nháy cho thiết bị                                  | ESP32-C3 gửi data qua `LED1`; nguồn LED được cấp qua `V_LED1`                                      | Khi tắt nguồn LED, firmware nên kéo chân data LOW      |
| 14  | **LED RGB SK6812 số 2**      | D2 SK6812MINI, R8 330R, R14 100k, C11 10uF, `LED2`                     | LED RGB thứ hai để tăng hiệu ứng ánh sáng                                 | ESP32-C3 gửi data qua `LED2`; nguồn LED được cấp qua `V_LED2`                                      | Có thể dùng để tạo hiệu ứng đối xứng/trang trí         |
| 15  | **Motor rung**               | MOTOR+, MOTOR-, Q1 AO3400, R9 100R, R10 100k, C9 100nF, C12 47uF, SS14 | Tạo phản hồi rung khi nhận tín hiệu từ thiết bị kia                       | ESP32-C3 kéo `MOTOR` lên HIGH → Q1 dẫn → dòng chạy qua motor → motor rung                          | SS14 dập xung ngược, C9 giảm nhiễu, C12 chống sụt áp   |
| 16  | **Nút bấm chính**            | Button+, Button-, R6 100k, C8 100nF, `GPIO_BT`                         | Người dùng nhấn để gửi tín hiệu sang móc khóa còn lại                     | Bình thường `GPIO_BT` được kéo lên 3V3; khi nhấn nút, chân này bị kéo xuống GND                    | Nhấn = LOW, thả = HIGH; nên đổi R6 thành 47k hoặc 10k  |
| 17  | **Tụ ổn định nguồn MCU**     | C7 100nF, C6 47uF                                                      | Lọc nguồn 3V3 cho ESP32-C3                                                | C7 lọc nhiễu cao tần, C6 cấp dòng tức thời khi Wi-Fi hoạt động                                     | Đặt gần chân 3V3/GND của module                        |
| 18  | **Tụ ổn định nguồn motor**   | C12 47uF, C9 100nF                                                     | Giảm sụt áp và nhiễu khi motor bật/tắt                                    | C12 giữ nguồn VBAT ổn định, C9 lọc nhiễu phát ra từ motor                                          | C12 nên đặt gần motor driver                           |
