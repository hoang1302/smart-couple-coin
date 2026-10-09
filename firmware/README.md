# Firmware — Smart Couple Coin

**Trạng thái: chưa triển khai.** Thư mục này chưa có mã nguồn, cấu hình build hoặc firmware để nạp.

Phần cứng mục tiêu là ESP32-C3 SuperMini. Hướng phát triển trong tài liệu là C/C++ với Arduino framework; công cụ build và phiên bản thư viện sẽ được chốt khi tạo project firmware.

## Tài liệu tham khảo

- [Đặc tả sản phẩm và firmware](../spec/spec.md)
- [Phân bổ GPIO và phần cứng](../hardware/hardware.md)
- [Lộ trình phát triển firmware](../docs/firmware-plan.md)
- [Quy trình kiểm tra phần cứng và firmware](../docs/assembly-and-testing.md)

Các ví dụ code trong tài liệu chưa được xác nhận chạy trên mạch đã hàn.

## Bước triển khai đầu tiên

1. Kiểm tra mạch sau hàn và xác nhận đường nạp/debug.
2. Tạo cấu hình build cho ESP32-C3 và nạp chương trình thử cơ bản.
3. Kiểm thử lần lượt nút bấm, LED, motor và ADC pin trước khi thêm kết nối mạng.
4. Kiểm thử deep sleep và cơ chế thức dậy trên mạch thực tế.
5. Sau khi các ngoại vi hoạt động ổn định, triển khai Wi-Fi, MQTT và thử với hai thiết bị.

## Cấu hình riêng

Khi triển khai kết nối mạng, giữ mật khẩu Wi-Fi và thông tin đăng nhập MQTT trong file cấu hình riêng được Git bỏ qua, chẳng hạn `secrets.h` hoặc `.env`. Chỉ commit file mẫu không chứa thông tin đăng nhập thật.

Hướng dẫn cài đặt, build và nạp sẽ được bổ sung khi có project firmware hoạt động.
