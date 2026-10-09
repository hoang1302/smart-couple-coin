| STT | Nhóm                  | Linh kiện                 | Giá trị / Mã linh kiện         |    SL | Ghi chú                                     |
| --: | --------------------- | ------------------------- | ------------------------------ | ----: | ------------------------------------------- |
|   1 | MCU                   | Module ESP32-C3 SuperMini | ESP32-C3 SuperMini             |     1 | Điều khiển chính, Wi-Fi/BLE                 |
|   2 | Sạc pin               | IC sạc Li-Po              | LTC4054ES5-4.2                 |     1 | Sạc pin 1 cell 4.2V                         |
|   3 | Nguồn                 | LDO 3.3V                  | AP2112K-3.3                    |     1 | Tạo nguồn 3V3 cho ESP32                     |
|   4 | USB                   | Cổng USB Type-C           | USB-C 16P / charging only      |     1 | Chỉ dùng sạc, không dùng data               |
|   5 | Pin                   | Pin Li-Po                 | 1 cell 3.7V, khoảng 150–220mAh |     1 | Nên dùng pin có mạch bảo vệ                 |
|   6 | Motor                 | Motor rung coin           | 3V coin vibration motor        |     1 | Motor rung phản hồi                         |
|   7 | LED RGB               | LED addressable           | SK6812MINI-012                 |     2 | LED hiệu ứng                                |
|   8 | LED báo sạc           | LED SMD                   | LED 0805                       |     1 | Báo đang sạc                                |
|   9 | MOSFET N              | NMOS                      | AO3400                         |     3 | 1 con motor, 2 con kéo gate PMOS            |
|  10 | MOSFET P              | PMOS                      | AO3401A                        |     2 | Công tắc nguồn cho 2 LED RGB                |
|  11 | Diode dập xung motor  | Schottky diode            | SS14                           |     1 | Dập xung ngược motor                        |
|  12 | TVS USB               | TVS diode                 | SMF5.0A                        |     1 | Bảo vệ đường VBUS USB                       |
|  13 | Nút bấm               | Tact switch / side button | Nút nhấn thường hở             |     1 | Nhấn = LOW                                  |
|  14 | Điện trở CC USB       | Resistor                  | 5.1k                           |     2 | R1, R3                                      |
|  15 | Điện trở set dòng sạc | Resistor                  | 6.8k                           |     1 | R5, dòng sạc khoảng 140–150mA               |
|  16 | Điện trở chia áp pin  | Resistor                  | 1M                             |     2 | R2, R4; ADC_BAT = VBAT/2                    |
|  17 | Điện trở LED sạc      | Resistor                  | 2.2k                           |     1 | R12                                         |
|  18 | Điện trở data LED     | Resistor                  | 330R                           |     2 | R7, R8                                      |
|  19 | Điện trở gate         | Resistor                  | 100R                           |     3 | R9, R15, R16                                |
|  20 | Điện trở kéo          | Resistor                  | 100k                           |     9 | Pull-up/pull-down cho nút, MOSFET, LED data |
|  21 | Jumper tùy chọn       | Resistor 0R / DNP         | 0R                             |     1 | R11, mặc định không hàn                     |
|  22 | Tụ gốm                | Capacitor                 | 100nF                          |     4 | C5, C7, C8, C9                              |
|  23 | Tụ gốm                | Capacitor                 | 10uF                           |     6 | C1, C2, C3, C4, C10, C11                    |
|  24 | Tụ nguồn              | Capacitor                 | 47uF                           |     2 | C6, C12                                     |
|  25 | Pad pin               | Solder pad / connector    | BAT+, BAT-                     | 1 cặp | Nối pin Li-Po                               |
|  26 | Pad motor             | Solder pad / connector    | MOTOR+, MOTOR-                 | 1 cặp | Nối motor rung                              |
|  27 | Pad công tắc nguồn    | Solder pad                | LDO_en / VBAT                  | 1 cặp | Gắn công tắc ngoài                          |
|  28 | Test pad              | Test pad                  | RXD0, TXD0                     |     2 | UART debug/nạp firmware                     |
