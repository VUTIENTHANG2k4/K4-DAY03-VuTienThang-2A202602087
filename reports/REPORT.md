# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Vũ Tiến Thăng`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | Chưa có log thời gian |
| Thời gian gán `clip_01` | Chưa có log thời gian |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | Chưa xác định từ file MOT; export không lưu keyframe |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che và xuất hiện lại: giữ cùng ID khi còn đủ bằng chứng liên tục, không tạo ID mới chỉ vì bbox ngắn bị mất.
2. Xe cắt nhau/chồng lên nhau: ưu tiên identity và chuyển động trước đó, đồng thời đặt bbox theo phần nhìn thấy.
3. Xe ở gần rìa khung hoặc rất nhỏ: bắt đầu/kết thúc track ở frame đầu/cuối còn xác định được là xe bốn bánh, không đoán phần ngoài ảnh.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Chưa có biên bản tua riêng; file nhãn cuối có 8 ID, từ 1 đến 8, trải trên 190 frame.
- Lượt 2: Chưa có biên bản frame đầu/cuối. Diagnostics sau đánh giá cho thấy nhãn có các bbox dư so với gold ở các đoạn đầu/cuối của một số track.
- Lượt 3: Chưa có biên bản frame giữa; các vùng cần xem lại theo diagnostics gồm frame 54, 84, 96, 102, 103 và 106 do IoU thấp (0.517–0.581).

Kiểm chéo với: chưa thực hiện hoặc chưa nộp biên bản. File `reports/review_partner.md` chưa tồn tại.
Số lỗi bạn tìm được trong bản của bạn ấy: chưa có dữ liệu. Số lỗi bạn ấy tìm được trong bản của bạn: chưa có dữ liệu.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Chưa có dữ liệu kiểm chéo để kết luận ca bất đồng. Guideline còn thiếu ngưỡng bắt đầu/kết thúc khi xe rất nhỏ hoặc mờ và cách xử lý cụ thể khi xe rời khung rồi quay lại; hai luật này nên được ghi bằng frame ví dụ.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | Chưa có: manifest không tồn tại |
| Thời điểm khóa | Chưa xác định: evidence pre-gold không tồn tại |
| Số row / frame / track trước khi mở reference | Chưa xác định; file pre-gold không tồn tại |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có |
| Nhãn hiện tại (không có pre-gold) | 0.7615 | 0.7457 | 0.7799 | 0.8457 | 0.9506 | 0.8970 | 0.8276 | 54 | 5 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Chưa có evidence rework | Chưa có | Chưa có | Không thể xác định thay đổi vì pre-gold và review chưa được nộp |
| Chênh lệch bbox so với gold | 54, 55, 84, 96, 102, 103, 106 | 4, 5, 6, 7 | Cần mở lại ảnh để xác nhận; diagnostics chỉ cho biết IoU thấp, không đủ căn cứ ghi là đã sửa |
| Ghost/endpoint | 63–78, 85–100, 133–135, 149–151, 169–171 | 4, 5, 8 | Chưa có evidence trước/sau để xác nhận rework |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / 2, 5, 7 |
| device | CUDA `0`; `persist=true`; 190 frame |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.7615 | 0.7457 | 0.7799 | 0.8457 | 0.9506 | 0.8970 | 0.8276 | 54 | 5 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.6991 | 0.6375 | 0.7714 | 0.8487 | 0.8635 | 0.7251 | 0.8299 | 93 | 77 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

IDF1 của nhãn tay (0.9506) cao hơn MOTA (0.8970). Với model ByteTrack, IDF1 (0.8746) cũng cao hơn MOTA (0.7487). IDF1 đo chất lượng ghép identity của các detection đã match, còn MOTA cộng dồn FP, FN và IDSW trên toàn bộ frame. Vì vậy một kết quả có association tốt nhưng còn nhiều detection bỏ sót/dư vẫn có thể có IDF1 cao và MOTA thấp; MOTA không chỉ phạt lỗi ID mà còn bị chi phối mạnh bởi FP/FN.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với gold, ReID tăng HOTA 0.7085→0.7635, DetA 0.6487→0.7110, AssA 0.7761→0.8204 và IDF1 0.8746→0.9001; IDSW vẫn là 2. Một ví dụ không cải thiện hoàn toàn là frame 87, track gold 5: ReID đổi từ track 17 sang 18. ByteTrack có hai switch ở frame 59 (gold 4) và 94 (gold 5), còn ReID chuyển các điểm khó sang frame 87 (gold 5) và 113 (gold 6). Kết quả cho thấy treatment tốt hơn về association tổng thể nhưng không loại bỏ switch. Đây không phải causal effect cô lập của ReID vì hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

ReID giảm FN 54→26 và tăng DetA 0.6487→0.7110, nhưng FP tăng 88→91. Vì FN giảm nhiều hơn FP tăng nên MOTA tăng 0.7487→0.7923. Phần còn lại là cả detector/coverage và association: FP/FN, DetA phản ánh detection và độ phủ; IDSW, AssA phản ánh association. Với ReID, AssA tốt hơn nhưng vẫn còn ghost track và fragmentation, nên không thể quy toàn bộ lỗi còn lại cho detector.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 87, identity gold 5: nhãn gold duy trì cùng identity, trong khi ReID có IDSW từ track 17 sang 18. Vì vậy ở điểm này annotation nhất quán hơn về continuity; đây là lỗi association của ReID, dù ID model không trùng số ID annotation.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Không có frame nào đủ bằng chứng để kết luận ReID đúng hơn và buộc phải sửa annotation. Phép so sánh ReID với nhãn tay vẫn cho MOTA 0.7251, thấp hơn cổng 0.75, FN 77 và một IDSW; do đó model chỉ là tín hiệu để review, không phải ground truth thay thế. Cần ảnh/video tại frame cụ thể trước khi sửa nhãn.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

1. Bắt buộc lưu thời gian bắt đầu/kết thúc cho từng clip và lưu keyframe hoặc log CVAT, vì file MOT không khôi phục được hai thông tin này.
2. Khi phát hiện lỗi bằng evaluation, ghi closure theo đúng frame–ID và lưu snapshot pre-gold trước mọi rework.
3. Bổ sung luật bằng ví dụ cho xe rất nhỏ/mờ, entry/exit ở rìa ảnh, xe rời khung quay lại và crossing; sau đó chạy ba lượt self-QC có checklist cố định.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt` (chưa có file nhãn, chỉ có thư mục rỗng)
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` (chưa có)
- [ ] `GUIDELINE_MINI.md` đã điền (vẫn còn placeholder)
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` (chưa có)
- [x] `reports/REPORT.md` (file này)
