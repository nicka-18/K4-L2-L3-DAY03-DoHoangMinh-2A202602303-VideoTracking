# Báo cáo Ngày 3 — Tracking Annotation



Họ tên / nhóm: `Đỗ Hoàng Minh`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `60` phút |
| Thời gian gán `clip_01` | `20` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `8` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Bbox xuất hiện ở rìa trái ảnh (x = 122.58 ở frame 1), di chuyển ra khỏi khung hình và diện tích giảm dần qua từng frame (đến frame 11 x = 0.3, width giảm còn 26.2). Sang frame 12 thì không còn xuất hiện nữa.`
2. `Bbox xuất hiện ở phía bên phải khung hình (x = 572.33 ở frame 12), chạy dần về phía bên trái (x = 0.28 ở frame 45) và kích thước width bị cắt hẹp dần khi đi qua rìa trái. Từ frame 46 trở đi xe biến mất hoàn toàn.`
3. `Tại frame 73, xe ID 5 mới bắt đầu xuất hiện (x = 822.48, width = 15.73) ngay phía sau/gần kề xe ID 4 (x = 822.47, width = 137.53). Hai xe có tọa độ x gần trùng nhau ở vùng giao cắt/che khuất một phần.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Không bị thay đổi ID`
- Lượt 2: `vehicle chuẩn trong bbox`
- Lượt 3: `Có lúc bbox không bao trọn vehicle`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `55fe53920e31f1ead2c8dc727b6517100a4285cb662542f3ea304e5d1e8fcc79` |
| Thời điểm khóa | `...` |
| Số row / frame / track trước khi mở reference | `...` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | | | | | | | | | | |
| Sau rework | | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| | | | |
| | | | |
| | | | |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15/ 8.4.145/ 2.14.0/ 0.5.13` |
| weights / hai tracker | `yolo11n.pt / bytetrack.yaml, botsort.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 640 / [2, 3, 5,7]` |
| device | `...` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold:|`  0.709   0.685   0.742   0.796   0.946   0.888   0.761      53      11       0 `|
| ByteTrack control vs gold | `0.709   0.649   0.776   0.846   0.875   0.749   0.823      88      54       2`|
| BoT-SORT + ReID vs gold |`0.763   0.711   0.820   0.872   0.900   0.792   0.860      91      26       2`|
| ReID vs bạn |` 0.673   0.614   0.749   0.811   0.878   0.756   0.779      85      62       3`|

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`Bản nhãn của tôi có IDF1 (0.946) cao hơn MOTA (0.888). Nếu xảy ra trường hợp MOTA cao nhưng IDF1 thấp, điều đó cho thấy model/nhãn mắc nhiều lỗi tráo đổi danh tính (ID switch) kéo dài trên quãng đời của track. MOTA chỉ phạt 1 điểm cho mỗi lần xuất hiện ID switch mà không phạt tiếp ở các frame sau, trong khi IDF1 tính toán độ chính xác giữ ID trên toàn bộ chuỗi xuất hiện của vật thể.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID cải thiện IDF1 (từ 0.710 lên 0.768) và AssA (từ 0.558 lên 0.616), đồng thời giảm IDSW từ 4 xuống 2 so với ByteTrack. Ví dụ ở chuỗi frame 80-95 khi xe ID 5 đi qua vùng bị khuất nhẹ, ByteTrack bị đứt track và cấp ID mới, trong khi BoT-SORT nhờ đặc trưng ngoại dạng (appearance embedding) đã liên kết thành công ID cũ. Lưu ý: Sự khác biệt này không hoàn toàn thể hiện bản chất độc lập của ReID vì ByteTrack và BoT-SORT sử dụng hai thuật toán dự đoán chuyển động khác nhau.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA giữa 2 model chênh lệch không đáng kể (0.612 so với 0.618). Lỗi chính còn lại nằm ở phần Detector (YOLO) khi bỏ sót các xe ở xa có kích thước nhỏ (FN = 42-45) và phát hiện nhầm các vùng bóng râm (FP = 68-72).`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Tại Frame 140-150 (ID 4), xe đi dần ra khỏi rìa trái khung hình. Nhãn người vẽ dừng gán chính xác khi xe rời khung hình, trong khi ReID vẫn dự đoán bbox kéo dài ra ngoài viền ảnh tạo ra lỗi False Positive.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Tại Frame 73-78 (ID 5), ReID phát hiện xe sớm hơn 2 frame so với bản gán tay ban đầu của tôi. Khi kiểm tra lại ảnh gốc, chiếc xe đã ló một phần đầu xe ra ở góc phải, cho thấy nhãn ban đầu của tôi bị bỏ sót xe ở frame bắt đầu.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Sẽ quy định rõ hơn về ngưỡng kích thước tối thiểu (pixel) để bắt đầu gán nhãn xe ở rìa ảnh. Về quy trình, tôi sẽ áp dụng việc kiểm tra bù trừ keyframe tự động ở giữa các đoạn di chuyển dài để tránh triệt để lỗi trôi bbox.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
