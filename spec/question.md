# ❓ Câu hỏi cần xác nhận để hoàn thành Spec — Smart Couple Coin

> **Ngày tạo**: 2026-06-12
> **Mục đích**: Liệt kê các câu hỏi chưa được trả lời trong spec hiện tại. Cần chốt để hoàn thiện spec và bắt đầu code firmware.
> **Cách trả lời**: Ghi câu trả lời ngay sau mũi tên `=>` của mỗi câu hỏi.

---

## A. Phần cứng — Cần chốt trước khi hàn

### A1. Giá trị linh kiện

1. **R6 (pull-up nút bấm)**: Chốt giá trị nào? Hardware.md ghi chú "nên đổi sang 47k hoặc 10k" nhưng BOM vẫn là 100kΩ. Giá trị nhỏ hơn (10k) cho tín hiệu ổn định hơn khi bấm, nhưng tiêu thụ thêm ~0.33mA khi nhấn nút.
   => Chốt 47k
2. **SK6812MINI-012 là RGB (3 kênh) hay RGBW (4 kênh)?** Ảnh hưởng trực tiếp đến NeoPixel protocol trong firmware. Nếu RGBW → dùng `NEO_GRBW`, nếu RGB → dùng `NEO_GRB`.
   =>RGB
3. **R16 kích thước 0805 có chủ ý không?** R15 cùng chức năng (gate resistor) là 0603. Có phải do layout bắt buộc hay nhầm?
   => là 0603
4. **Số lượng R 100kΩ**: BOM ghi 9 con nhưng đếm từ schematic chỉ thấy 7 con (R6, R10, R14, R18, R19, R20, R21). 2 con còn lại là R nào?
   => Bỏ qua

### A2. PCB & Layout

5. **Test pad bổ sung**: Hardware.md gợi ý thêm TP cho GND, 3V3, EN/RST, GPIO9/BOOT. PCB hiện tại đã có những TP nào? Có cần rev PCB để thêm?
   => Đã có đủ testpoint
6. **R11 (0Ω jumper, DNP)**: Chức năng cụ thể là gì? Khi nào cần hàn R11? Tài liệu ghi "mặc định không hàn" nhưng không giải thích use case.
   => Không cần hàn, 2 pad dùng để hàn switch ngoài
7. **CHG_LED có nhìn thấy được qua vỏ không?** LED báo sạc nằm ở mặt top PCB gần khu vực nguồn — cần đảm bảo vỏ 3D có lỗ hoặc vùng trong suốt tại vị trí này.
   => Nhìn được

ngoài---

## B. Firmware — Cần chốt trước khi code

### B1. Timing & Thông số

8. **Giá trị X cho anti-spam** (FR-01): Khoảng bao nhiêu phút giữa 2 lần gửi tín hiệu? Spec ghi "X phút, mặc định đề xuất 1 phút". Chốt 1 phút?
   =>Nhận tín hiệu mỗi 5p(từ 7h đến 23),30p (từ 23h đến 7h) hoặc nhận khi d
9. **Thời gian đợi MQTT message khi polling**: Spec ghi ~500ms. Đây có phải giá trị chốt? Nếu WiFi chậm, 500ms có đủ không? Có nên tăng lên 1-2 giây?
   =>
10. **Watchdog timer timeout**: Process.md ghi 15 giây. Chốt giá trị này?
    => Chốt

### B2. Hành vi edge case

11. **Khi đang ở AP Mode (Captive Portal), có timeout không?** Nếu người dùng bật thiết bị nhưng không cấu hình WiFi, thiết bị có tự tắt/sleep sau X phút để tiết kiệm pin?
    => Tắt sau 5p
12. **Khi cắm sạc USB, thiết bị có tự bật không?** Hay vẫn phải gạt công tắc? Hiện tại spec nói "sử dụng khi sạc = Có" nhưng chưa rõ hành vi khi cắm USB mà công tắc đang OFF.p
    =>Vẫn cần gạt công tắc
13. **OTA update trigger**: Thiết bị kiểm tra OTA update khi nào? Mỗi lần wake? Mỗi 24h? Hay chỉ khi người dùng chủ động trigger (ví dụ bấm giữ nút 5 giây)?
    =>Chủ động trigger(giữ 20s)
14. **Khi gửi thất bại và lưu vào RTC memory** (FR-03, mục 5.3.7): Lưu tối đa bao nhiêu tín hiệu pending? Nếu mất mạng nhiều ngày, RTC có overflow không?
    =>Chỉ luư trong ngày
15. **Khi cả 2 thiết bị bấm nút gần đồng thời**: Mỗi thiết bị vừa publish vừa check retained message. Có race condition không khi cả 2 cùng clear retained message?
    =>Khôgn cần

### B3. Captive Portal

16. **MQTT credentials**: Username/password HiveMQ được hardcode trong firmware hay cho người dùng nhập trên Captive Portal? Nếu hardcode → mỗi cặp thiết bị cần firmware riêng?
    =>Hardcode
17. **Pair ID**: Ai tạo pair ID? Firmware tự sinh (từ MAC address)? Hay người dùng tự nhập trên Captive Portal? Spec ghi "(Tùy chọn) Nhập MQTT topic ID cặp đôi" — tùy chọn nghĩa là nếu không nhập thì dùng giá trị nào? cố định
    =>cố định
18. **Color picker**: Lưu giá trị màu dạng gì? RGB (r, g, b)? Hex (#FF0000)? HSL? Có giới hạn preset màu hay full color wheel?
    =>RGB

---

## C. Cơ khí & Vỏ — Cần chốt trước khi thiết kế 3D

19. **Phương pháp lắp vỏ**: Vít M2 hay snap-fit? Snap-fit dễ mở nhưng có thể lỏng. Vít chắc hơn nhưng cần lỗ bắt vít trên PCB.
    =>Vít M2
20. **Vật liệu diffuser LED**: Nhựa trong mờ (frosted acrylic)? Resin in 3D trong suốt? Hay lớp film mờ dán lên?
    =>Nhựa
21. **Chiều cao vỏ**: Spec ghi 18mm "có thể điều chỉnh". Các linh kiện cao nhất trên PCB (module ESP32-C3, tụ 47µF, USB-C) cao khoảng bao nhiêu mm? Cần clearance bao nhiêu cho pin Li-Po?
    =>
22. **Vị trí nút bấm trên vỏ**: Nút nằm trên mặt top PCB (hình 3D) — vỏ cần lỗ/nắp cho nút. Có muốn nút nhô ra ngoài vỏ hay bấm qua lớp nhựa mềm (silicone membrane)?
    =>

---

## D. Vận hành & Bảo trì

23. **Khi cả 2 thiết bị mất mạng dài ngày** (ví dụ đi du lịch không WiFi): Thiết bị vẫn wake polling vô ích, tiêu pin. Có cần logic "giảm tần suất polling nếu liên tục không có tín hiệu" không?
    => Không cânf
24. **Firmware versioning**: Spec ghi "v1.0" nhưng chưa có convention cho version firmware. Dùng semantic versioning (1.0.0)? Firmware version có hiển thị trên Captive Portal không?
    =>cần
25. **Timezone**: Wake schedule dựa vào giờ 07:00-23:00 ban ngày. Múi giờ nào? UTC+7 (Việt Nam)? Hay lấy từ NTP và cho người dùng cấu hình timezone trên Captive Portal?
    =>ntp

---

## E. MQTT & Server

26. **HiveMQ Cloud account**: Dùng 1 account chung cho tất cả các cặp thiết bị, hay mỗi cặp 1 account riêng?
    =>Dùng chung
27. **MQTT QoS**: Spec ghi QoS 1 cho signal message. Có cần QoS 2 (exactly once) để tránh duplicate không? Hay QoS 1 + check `ts` là đủ?
    =>1
28. **Retained message race condition**: Nếu thiết bị A gửi tín hiệu, nhưng thiết bị B chưa wake để nhận, rồi A gửi tiếp lần 2 — B chỉ nhận được 1 retained message (mới nhất). Có chấp nhận mất tín hiệu cũ không?
    =>Chấp nhận
29. **Message `bat` field**: Gửi phần trăm pin trong message — thiết bị nhận có xử lý gì với thông tin pin của đối phương không? Hay chỉ để logging?
    =>logging

---

## F. Bảo mật & Tương lai (Phase 2, tham khảo)

30. **Khi chuyển sang Phase 2 — BLE qua app**: App trên iOS, Android, hay cả hai? Framework nào (Flutter, React Native, Native)?
    =>
31. **Server MQTT riêng Phase 2**: Self-host (VPS) hay managed service? Budget cho server hàng tháng?
    =>

---

> **Tổng cộng: 31 câu hỏi**
> **Ưu tiên cao** (cần chốt trước khi code firmware): Câu 2, 8, 11, 12, 13, 16, 17, 25
> **Ưu tiên trung bình** (cần chốt trước khi hàn/lắp ráp): Câu 1, 3, 4, 5, 6, 7
> **Ưu tiên thấp** (chốt sau cũng được): Các câu còn lại
