# Câu hỏi cần trả lời để thiết kế vỏ 3D - Smart Couple Coin

> Mục tiêu: thu thập đủ thông tin cơ khí để vẽ/chỉnh sửa vỏ 3D cho PCB trong `hardware/image/`.
> Kích thước sản phẩm mục tiêu: 50 x 35 x 18 mm.
> Kích thước PCB theo spec: 44 x 28 mm.
Quy định chiều pcb: Pcb đặt nằm ngang, chiều nhìn là nhìn ở mặt top
> Cách trả lời: ghi câu trả lời ngay sau dấu `=>` của từng câu.

---

## 1. Thông tin PCB cần đo chính xác

1. Kích thước PCB thực tế sau khi sản xuất có đúng 44 x 28 mm không? Nếu khác, ghi kích thước đúng theo mm.
   => Đúng 44x28

2. Độ dày PCB là bao nhiêu mm? Mặc định thường là 1.6 mm, nhưng cần chốt để tính chân đỡ và khe giữ PCB.
   => 1.6mm

3. PCB có góc bo tròn không? Nếu có, bán kính bo góc PCB là bao nhiêu mm?
   => Bo nhẹ bán kính 1.5

4. PCB có lỗ bắt vít sẵn không? Nếu có, ghi số lượng lỗ, đường kính lỗ và tọa độ tâm lỗ theo góc mốc PCB.
   => Có 2 lỗ bắt vít size 1.8mm nằm ở góc trên bên trái và góc dưới bên phải.

5. Nếu PCB không có lỗ bắt vít, bạn muốn giữ PCB bằng cách nào: chân kẹp/cạnh kẹp trong vỏ, keo 2 mặt, hay vít chỉ bắt vào vỏ và PCB nằm trong khay?
   =>

6. Chọn góc mốc tọa độ cho toàn bộ đo đạc PCB: nhìn từ mặt top, lấy góc nào làm (0,0)? Đề xuất: góc dưới trái, cạnh USB-C nằm phía dưới.
   => Lấy tâm pcb làm gốc

---

## 2. Hướng đặt PCB trong vỏ

7. Khi người dùng nhìn mặt trước sản phẩm, mặt top PCB có quay lên phía người dùng không?
   => mặt top quay lên phía trên

8. USB-C nằm ở cạnh nào của vỏ: dưới, trái, phải, hay trên?
   => dưới trái

9. Lỗ xỏ dây móc khóa nằm ở cạnh nào? Spec đang đề xuất cạnh trên.
   => cạnh ngắn đối diện usb c

10. Bạn muốn hình dạng vỏ tổng thể là gì: chữ nhật bo góc, viên thuốc/con nhộng, trái tim, đồng xu oval, hay hình khác?
    => chữ nhật bo góc

11. Bán kính bo góc ngoài của vỏ mong muốn là bao nhiêu mm? Nếu không biết, đề xuất 6-8 mm cho cảm giác mềm.
    => 6mm

12. Có cần mặt trước/mặt sau cong nhẹ hay giữ phẳng để in 3D dễ hơn?
    => phẳng

---

## 3. Kích thước và khoảng hở

13. Kích thước 50 x 35 x 18 mm là kích thước ngoài tối đa bắt buộc hay có thể vượt nhẹ? Nếu được vượt, giới hạn tối đa là bao nhiêu?
    => 50 x 35 x 18 mm là kích thước tối đa

14. Chiều cao 18 mm đã bao gồm nút bấm, công tắc, nắp USB và diffuser nhô lên chưa?
    => đã bao gồm

15. Bạn muốn độ dày thành vỏ bao nhiêu mm? Đề xuất: FDM 1.2-1.6 mm, SLA 1.0-1.2 mm.
    => 1.5mm

16. Khoảng hở quanh PCB mong muốn là bao nhiêu mm mỗi bên? Đề xuất tối thiểu 0.5-0.8 mm.
    => 0.5mm

17. Khoảng hở trên linh kiện cao nhất tới nắp vỏ là bao nhiêu mm? Đề xuất tối thiểu 0.8-1.0 mm.
    => 0.5mm

18. Linh kiện cao nhất trên PCB là linh kiện nào và cao bao nhiêu mm tính từ mặt PCB? Cần đo thực tế module ESP32-C3, USB-C, tụ 47uF, nút bấm.
    =>esp32

---

## 4. Pin và motor

19. Kích thước pin Li-Po thực tế là bao nhiêu mm (dài x rộng x dày)? Spec ghi 220 mAh nhưng cần kích thước cell thực.
    => pin: 30x20x5mm; motor: đường tròn 10mmx2.7mm

20. Pin đặt trên mặt nào so với PCB: trên PCB, dưới PCB, bên cạnh PCB, hay nằm riêng trong khoang pin?
    =>Pin và motor dưới pcb. pin nằm góc trên trái còn motor nằm góc dưới phải

21. Pin có cần có khoảng để băng keo 2 mặt/mút dán không? Nếu có, thêm độ dày bao nhiêu mm?
    =>1mm

22. Dây pin đi từ vị trí nào trên PCB đến pin? Cần khe/rãnh đi dây không?
    =>hàn đây pin vào pad trên pcb

23. Kích thước motor rung coin thực tế là bao nhiêu mm? Ghi đường kính x dày.
    =>đường tròn 10mmx2.7mm

24. Motor đặt ở vị trí nào trong vỏ? Có muốn motor chạm/sát vỏ để rung mạnh hơn không?
    =>sát với mặt dưới pcb

25. Motor được cố định bằng keo, bằng kẹp giữ, hay nằm trong hộc tròn riêng?
    =>hộc tròn

---

## 5. USB-C và nắp đậy

26. Tọa độ tâm cổng USB-C trên PCB là bao nhiêu theo góc mốc đã chọn?
    => -18.9mm x -6.99mm; rotate 270

27. Kích thước miệng cắt trên vỏ cho USB-C mong muốn là bao nhiêu mm (rộng x cao)? Cần đo theo đầu cáp USB-C thực tế.
    =>Rộng × cao: 10.5 × 4.5 mm

28. USB-C cần lùi/sát so với mặt ngoài vỏ như thế nào? Ví dụ: miếng kim loại USB-C ngang với mặt vỏ, lùi vào 0.5 mm, hay nhô ra.
    =>usbc được gắn trên pcb, cạnh ngắn trái pcb nằm sát vỏ

29. Có bắt buộc có nắp đậy silicon/nhựa mềm cho USB-C ở prototype này không, hay chỉ để lộ cổng USB-C trước?
    =>Chỉ lộ cổng usbc

30. Nếu có nắp đậy, nắp gắn bằng bản lề mềm liền thân vỏ, nắp rời, hay mua sẵn?
    =>nắp rời

31. Cần khoảng cắm đầu cáp USB-C dày/to không? Nếu có, ghi kích thước đầu cáp lớn nhất đã test.
    =>khoảng 4.5 mm

---

## 6. Nút bấm chính

32. Nút bấm nằm trên PCB hay là nút rời nối dây ra vỏ?
    =>nút rời

33. Tọa độ tâm nút bấm trên PCB là bao nhiêu theo góc mốc đã chọn?
    =>Nút bấm nằm trên cạnh vỏ, ở cạnh dài phía dưới



35. Bạn muốn nút bấm trên vỏ kiểu nào: nút nhựa rời, nút in liền thân đàn hồi, nút silicone membrane, hay để lộ tact switch?
    =>nút nhựa rời
nút nhựa rời
36. Nút bấm có cần nhô ra khỏi mặt vỏ không? Nếu có, nhô bao nhiêu mm?
    =>tối thiểu 2mm để dễ bấm

37. Đường kính/kích thước vùng bấm mong muốn là bao nhiêu mm?
    => dùng nút bấm 6x6x5mm

38. Cần lực bấm/cảm giác bấm ưu tiên: mềm, chắc tay, hay khó bấm để tránh bấm nhầm trong túi?
    =>

---

## 7. Công tắc nguồn

39. Công tắc nguồn là loại nào: slide switch, rocker switch, tact switch giữ trạng thái, hay công tắc rời nối dây?
    =>slide switch  8.6x3.6mm

40. Kích thước công tắc nguồn thực tế là bao nhiêu mm (thân công tắc và phần gạt nhô ra)?
    =>  8.6x3.6mm

41. Công tắc nguồn đặt ở cạnh nào của vỏ?
    =>Nằm ở cạnh ngắn, bên cạnh cổng usb c

42. Công tắc cần lỗ cắt kích thước bao nhiêu mm?
    =>8.6x3.6mm

43. Bạn muốn cần gạt công tắc nhô ra ngoài hay nằm âm trong khe để tránh gạt nhầm?
    =>

44. Dây công tắc nguồn đi từ pad SW trên PCB đến công tắc có cần rãnh/không gian riêng không?
    =>Không

---

## 8. LED, diffuser và LED báo sạc

45. Tọa độ tâm LED RGB D1 và D2 trên PCB là bao nhiêu theo góc mốc đã chọn?
    =>Nằm trên pcb
Không cần diffuser, chỉ làm 1 nắp vỏ mỏng phía trên


---

## 9. Kết cấu lắp ráp

53. Vỏ chốt dùng vít M2. Bạn muốn dùng mấy con vít: 2 vít hay 4 vít?
    =>2 vít ở 2 góc dưới của vỏ

54. Vị trí vít mong muốn ở đâu? Ghi tọa độ nếu PCB đã có lỗ, hoặc mô tả vị trí trên vỏ nếu vít chỉ bắt vào boss của vỏ.
    =>

55. Đầu vít nằm ở mặt nào: mặt sau/bottom để mặt trước đẹp hơn, hay mặt trước để dễ tháo?
    =>mặt trước

56. Lỗ vít dùng vít loại nào: M2 tự ren vào nhựa, M2 + insert đồng nung nóng, hay M2 + ê-cu?
    =>dùng M2 tự ren vào nhựa

57. Có cần tháo mở để thay pin không, hay vỏ sau khi lắp có thể ít mở lại?
    =>Ít mở

58. Bạn muốn top lắp vào bottom bằng rabbet/lip chống bụi không? Đề xuất có gờ chống bụi 0.6-1.0 mm.
    =>không

59. Có cần gioăng cao su/silicone để chống bụi tốt hơn không, hay tolerance khít là đủ cho prototype?
    =>không

---

## 10. Lỗ xỏ dây móc khóa

60. Dây xỏ/móc khóa dùng dây mềm, vòng kim loại split ring, hay dây đeo silicone?
    =>vòng kim loại

61. Đường kính lỗ tối thiểu là bao nhiêu mm? Spec đề xuất >= 3 mm.
      =>3.2mm

62. Phần quanh lỗ xỏ dây cần dày tối thiểu bao nhiêu mm để không gãy khi kéo? Đề xuất thành vật liệu quanh lỗ >= 2 mm.
    =>2mm

63. Lỗ xỏ dây nằm trong thân vỏ hay là tai/lug nhô ra ngoài?
    => nằm trong thân

64. Hướng xỏ dây: xỏ ngang qua chiều rộng vỏ hay xỏ từ mặt trước ra mặt sau?
    => xỏ ngang

---

## 11. Công nghệ in 3D và hoàn thiện

65. Bạn sẽ in bằng FDM hay SLA/resin?
    =>

66. Vật liệu vỏ mong muốn: PLA, PETG, ABS, resin, nylon, hay vật liệu khác?
    =>

67. Màu sắc vỏ mong muốn cho prototype?
    =>

68. Có cần bề mặt/texture nào: trơn mịn, nhám nhẹ, vân trái tim, logo/chữ khắc laser/khắc nổi?
    =>

69. Dung sai máy in thực tế là bao nhiêu? Nếu không biết, ghi model máy in hoặc để mặc định: FDM khe lắp 0.2-0.3 mm, SLA khe lắp 0.1-0.2 mm.
    =>

70. Mặt nào được ưu tiên đẹp nhất khi in? Mặt top, bottom, hay cạnh bên?
    =>

---

## 12. Thẩm mỹ và nhận diện sản phẩm

71. Sản phẩm là một cặp. Hai vỏ có cần khác màu/khác ký hiệu A-B không?
    =>

72. Có cần khắc logo/tên "Smart Couple Coin" trên vỏ không? Nếu có, đặt ở mặt nào?
    =>

73. Có cần ký hiệu ON/OFF gần công tắc không?
    =>

74. Có cần ký hiệu sạc/USB gần cổng USB-C không?
    =>

75. Phong cách mong muốn: tối giản, dễ thương, cao cấp, công nghệ, hay quà tặng tình yêu?
    =>

---

## 13. Ưu tiên khi có xung đột

76. Nếu không đủ không gian trong 50 x 35 x 18 mm, ưu tiên giữ yêu cầu nào trước: pin lớn, chống bụi, vỏ mỏng/dễ đẹp, dễ tháo lắp, hay độ bền?
    =>

77. Có chấp nhận tăng chiều cao từ 18 mm lên 19-20 mm nếu cần thêm clearance cho pin/module ESP32 không?
    =>

78. Có chấp nhận bỏ nắp USB-C ở prototype đầu tiên để đơn giản hóa vỏ không?
    =>

79. Có chấp nhận dùng keo để cố định pin/motor trong prototype, thay vì tạo kẹp giữ phức tạp không?
    =>

80. Yêu cầu nào là bắt buộc không được thay đổi?
    =>

---

## Thông tin tối đa có từ project hiện tại

- Sản phẩm: Smart Couple Coin, dạng móc khóa đôi.
- Kích thước vỏ mục tiêu: 50 x 35 x 18 mm.
- PCB trong spec: 44 x 28 mm, 2 lớp.
- Vỏ: 2 nửa top + bottom, in 3D, dùng vít M2.
- Cần có: USB-C lộ ra ngoài, công tắc nguồn ngoài, 1 nút bấm, 2 LED RGB với diffuser, CHG_LED nhìn thấy qua vỏ, pin Li-Po 220 mAh, motor rung coin, lỗ xỏ dây móc khóa.
- Ảnh PCB tham chiếu: `hardware/image/3d_top.png`, `hardware/image/3d_bottom.png`.
- File FreeCAD hiện có: `hardware/enclosure/CoinCase.FCStd` có `TopLid`, `BottomShell`, `PCB_Reference`, `Battery_Reference`, `Motor_Reference`, `ESP32_Reference`.
