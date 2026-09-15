# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Vũ Tiến Thăng`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): chỉ gán đối tượng nhìn thấy trong video; không dùng dự đoán của detector/model để thay thế việc xác định xe.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (xấp xỉ 2 giây ở 12.5 fps) | Một xe vẫn là cùng identity nếu còn liên kết được bằng vị trí, hướng chuyển động, hình dạng và bối cảnh trước/sau vùng che. |
| Xe bị che lâu hơn ngưỡng trên | tạo track mới; không tự nối ID cũ nếu không có bằng chứng chắc chắn | Tránh nối nhầm hai xe giống nhau sau một khoảng mất dấu dài. Nếu không chắc, ưu tiên tách track và ghi lại frame cần review. |
| Xe rời khung hình rồi quay lại | tạo **track mới** sau khi xe ra khỏi khung; không reuse ID cũ | Entry/exit là hai lần xuất hiện độc lập trong annotation MOT, trừ khi quy định của task nói rõ khác. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo quỹ đạo trước khi chồng; sau khi tách, đối chiếu hướng đi và đặc điểm xe, không đổi ID chỉ vì occlusion | Identity quan trọng hơn việc bbox tạm thời gần nhau; bbox chỉ ôm phần nhìn thấy của từng xe. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định chắc chắn là xe bốn bánh và có thể đặt bbox vào phần nhìn thấy; không đặt ngưỡng pixel cứng vì chưa có calibration riêng |
| Xe đang đỗ, không di chuyển | vẫn giữ cùng ID và bbox ở mọi frame còn nhìn thấy; không xóa vì không có chuyển động |
| Keyframe đặt dày ở đâu | đặt dày hơn ở lúc entry/exit, trước-sau occlusion/crossing, khi xe chạm rìa ảnh và nơi hình dạng thay đổi nhanh; vùng chuyển động đều có thể đặt thưa hơn. Phải xem lại frame giữa để tránh interpolation drift. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 54 / ID 4`
- Tình huống: bbox của ID 4 có IoU chỉ 0.517 so với gold, thuộc nhóm hình học sát ngưỡng.
- Quyết định: giữ ID 4, chỉ chỉnh bbox theo phần xe nhìn thấy sau khi xem lại ảnh; không đổi identity chỉ vì IoU thấp.
- Lý do: diagnostics xác định đây là loose box, không phải ID switch; identity vẫn liên tục.

### Ca 2
- Clip / frame / ID: `clip_01 / frames 63–78 / ID 5`
- Tình huống: diagnostics báo đoạn bbox xuất hiện trước thời điểm track tham chiếu 5 xuất hiện, tức một đoạn ghost/endpoint.
- Quyết định: kiểm tra frame bắt đầu thực sự của xe; không kéo track ngược về trước frame đầu tiên xác định được xe.
- Lý do: entry phải theo bằng chứng hình ảnh, không theo giả định xe đã tồn tại ngoài vùng quan sát.

### Ca 3
- Clip / frame / ID: `clip_01 / frames 102–106 / ID 6–7`
- Tình huống: các bbox của ID 6 và ID 7 có IoU thấp trong vùng xe gần nhau; diagnostics ghi nhận frame 102/103 của ID 6 và frame 106 của ID 7 là loose boxes.
- Quyết định: giữ identity theo quỹ đạo trước/sau, đặt bbox riêng theo phần nhìn thấy, không gán chồng hai xe vào một ID.
- Lý do: crossing/overlap dễ gây đổi ID; cần ưu tiên continuity và kiểm tra frame giữa hai keyframe.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Ghi riêng frame bắt đầu và kết thúc của từng track; không kéo bbox vào đoạn trước khi xe xuất hiện hoặc giữ bbox sau khi xe đã rời khung. Các đoạn cần đặc biệt kiểm tra gồm 63–78, 85–100, 133–135, 149–151 và 169–171.
- Khi bbox có IoU thấp nhưng identity vẫn liên tục, sửa hình học theo ảnh và giữ ID; chỉ đổi ID khi có bằng chứng về hai xe khác nhau hoặc một lần entry mới. Mọi thay đổi phải ghi theo frame–ID và lưu snapshot pre-gold trước rework.
