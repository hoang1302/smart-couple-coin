# 📝 Review — Các điểm đã bổ sung/tối ưu Spec v1.1

> **Ngày cập nhật**: 2026-06-12\
> **Người thực hiện**: hado (Antigravity Agent)\
> **Nguồn**: Câu trả lời từ `spec/question.md` (31 câu hỏi)\
> **File đã sửa**: `spec/spec.md` (v1.0 → v1.1)

---

## 1. Tổng quan thay đổi

| Hạng mục | Số lượng thay đổi |
|----------|-------------------|
| Thông số chốt cứng (từ "TBD" → giá trị cụ thể) | **12** |
| Section mới thêm | **2** (OTA Mode 5.3.9, edge cases) |
| Acceptance criteria viết lại | **1** (mục 12 hoàn toàn mới) |
| Link sửa lỗi | **2** (BOM + hardware.md) |
| Trạng thái spec | Draft → **Đã chốt Phase 1** |

---

## 2. Chi tiết từng thay đổi

### 2.1 Header & Metadata
| Trước | Sau | Lý do |
|-------|-----|-------|
| Phiên bản 1.0 | **Phiên bản 1.1** | Cập nhật version |
| Trạng thái: Draft → Chờ duyệt | **Draft → Đã chốt Phase 1** | Các câu hỏi quan trọng đã được trả lời |
| — | Thêm dòng **Cập nhật: 2026-06-12** | Ghi nhận ngày sửa |

### 2.2 FR-01: Gửi tín hiệu
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| Anti-spam: "X phút, mặc định đề xuất 1 phút" | **Chốt 1 phút** (cứng) | #8 |
| Chỉ có bấm ngắn, bấm giữ 2-3s | Thêm: **Bấm giữ 10s** = Factory Reset, **Bấm giữ 20s** = OTA Mode | #13 |

### 2.3 FR-02: Nhận tín hiệu
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| "2 LED RGB (SK6812MINI)" | **SK6812MINI-012, 3 kênh RGB, protocol `NEO_GRB`** | #2 |

### 2.4 FR-05: Quản lý nguồn
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| Chưa đề cập hành vi cắm USB khi OFF | **Thiết bị KHÔNG tự bật** — vẫn phải gạt công tắc ON | #12 |
| "LED báo sạc sáng khi sạc, tắt khi đầy" | Thêm: **Nhìn thấy được qua vỏ** | #7 |

### 2.5 FR-06: OTA Update ⭐ Thay đổi lớn
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| Không có trigger cụ thể | **Bấm giữ nút 20 giây** để vào OTA mode | #13 |
| Không có firmware versioning | **Semantic versioning** (1.0.0), hiển thị trên Captive Portal | #24 |
| — | Thêm section **5.3.9 OTA Update Mode** với quy trình chi tiết | #13 |

### 2.6 NFR-03: Độ tin cậy
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| "Firmware PHẢI có WDT" (không có timeout) | **WDT timeout 15 giây** (chốt) | #10 |

### 2.7 Sơ đồ khối & GPIO
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| "Nút bấm (R6+C8)" | **"Nút bấm (R6 47k+C8)"** | #1 |
| "pull-up 3V3" | **"pull-up 47kΩ lên 3V3"** | #1 |
| Link BOM: `Đường dẫn tuyệt đối trên máy phát triển` | **`../hardware/bom.md`** (relative path) | Lỗi phát hiện |
| Link hardware: `Đường dẫn tuyệt đối trên máy phát triển` | **`../hardware/hardware.md`** (relative path) | Lỗi phát hiện |

### 2.8 AP Mode (Captive Portal) ⭐ Thay đổi lớn
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| "Chọn màu LED (color picker)" | **Color picker lưu dạng RGB** (r, g, b), full color wheel | #18 |
| "(Tùy chọn) Nhập MQTT topic ID cặp đôi" | **Hiển thị Pair ID (chỉ đọc, cố định)** | #17 |
| Không có timeout | **Timeout 5 phút** → tự deep sleep | #11 |
| — | Hiển thị firmware version trên trang web | #24 |
| — | Hiển thị Device ID ("A" hoặc "B") | — |

### 2.9 Timezone & NTP
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| Không đề cập timezone | **Lấy tự động từ NTP**, mặc định UTC+7 nếu không xác định được | #25 |

### 2.10 Mất kết nối & Pending Signal
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| "Lưu tín hiệu vào RTC memory" (không giới hạn) | **Tối đa 1 tín hiệu**, hết hạn sau **24 giờ** | #14 |
| — | Thêm: "Không giảm tần suất polling khi mất mạng dài ngày" | #23 |

### 2.11 Factory Reset
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| "Xóa toàn bộ config WiFi, màu LED, pair ID" | **Pair ID KHÔNG bị xóa** (cố định trong firmware) | #17 |

### 2.12 MQTT Protocol ⭐ Thay đổi lớn
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| Không đề cập account model | **1 account chung** cho tất cả cặp thiết bị | #26 |
| "Username/Password (cấu hình sẵn)" | **Hardcode trong firmware** (không cho nhập) | #16 |
| — | Thêm: Pair ID **cố định**, nạp khi sản xuất | #17 |
| `from`: "A" hoặc "B" | Thêm: **cố định trong firmware** | #17 |
| `bat`: "tùy chọn" | Chỉ để **logging**, thiết bị nhận không xử lý | #29 |
| — | Thêm ghi chú: **Chấp nhận race condition** khi cả 2 bấm đồng thời | #15 |
| — | Thêm ghi chú: **Chấp nhận mất tín hiệu cũ** với retained message | #28 |

### 2.13 Cơ khí
| Trước | Sau | Câu hỏi |
|-------|-----|---------|
| "vít M2 hoặc snap-fit" | **Vít M2 (chốt)** | #19 |
| "diffuser nhựa trong mờ" | **Diffuser nhựa mờ (chốt)** | #20 |
| — | Thêm: CHG_LED phải nhìn thấy qua vỏ | #7 |

### 2.14 Acceptance Criteria ⭐ Viết lại hoàn toàn
| Trước | Sau | Lý do |
|-------|-----|-------|
| 13 tiêu chí, không phân nhóm | **28 tiêu chí**, chia 7 nhóm | Rõ ràng hơn |
| "< 1 giây" (sai với polling model) | **"< 3 giây" (gửi) + "≤ 5 phút / 30 phút" (nhận)** | Sửa mâu thuẫn |
| "X phút" anti-spam | **1 phút** (chốt) | #8 |
| Không có OTA, WDT, NTP test | Thêm: OTA trigger, WDT test, NTP test | #10, #13 |
| Không có pending signal test | Thêm: Pending signal + hết hạn 24h | #14 |
| Không có AP timeout test | Thêm: AP Mode tự tắt 5 phút | #11 |
| Không có deep sleep current test | Thêm: **Dòng deep sleep ≤ 50µA** | — |
| Không có cắm USB OFF test | Thêm: Cắm USB khi OFF → chỉ sạc | #12 |

---

## 3. Câu hỏi chưa trả lời

Các câu sau chưa có câu trả lời (để trống hoặc Phase 2):

| # | Câu hỏi | Ảnh hưởng |
|---|---------|-----------|
| 9 | Thời gian đợi MQTT polling (500ms?) | Giữ mặc định 500ms trong spec |
| 21 | Chiều cao vỏ | Chưa chốt, giữ 18mm |
| 22 | Nút bấm nhô hay qua membrane | Chưa chốt, cần khi thiết kế vỏ 3D |
| 30 | BLE app (iOS/Android/framework) | Phase 2 |
| 31 | Server MQTT riêng | Phase 2 |

> **Nhận xét**: Các câu chưa trả lời thuộc Phase 2 hoặc cơ khí — **không chặn** việc bắt đầu code firmware Phase 1.

---

## 4. Trạng thái sẵn sàng

| Hạng mục | Trước | Sau |
|----------|-------|-----|
| Spec đủ để code firmware | ⚠️ Còn nhiều TBD | ✅ **Đã chốt Phase 1** |
| Acceptance criteria | ⚠️ Sai latency | ✅ **Chính xác** |
| MQTT protocol | ⚠️ Thiếu details | ✅ **Đầy đủ** |
| Nút bấm behavior | ⚠️ Chỉ có bấm ngắn/giữ | ✅ **Đầy đủ 4 mức** (ngắn/3s/10s/20s) |
| OTA workflow | ❌ Không có | ✅ **Chi tiết** |

---

> **Kết luận**: Spec v1.1 đã đủ chi tiết để bắt đầu phát triển firmware. Các thông số quan trọng đã được chốt cứng. Tiếp theo nên bắt đầu code firmware theo lộ trình Phase 1 trong `assembly-and-testing.md`.
