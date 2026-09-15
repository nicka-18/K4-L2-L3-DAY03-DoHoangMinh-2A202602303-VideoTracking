# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Đỗ Hoàng Minh`
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

Bổ sung của nhóm (nếu có): Không có

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | ` Tránh ID switch khi xe bị chướng ngại vật nhỏ che tạm thời ` |
| Xe bị che lâu hơn ngưỡng trên | `mở track mới (ID mới)` | `Tránh nhầm lẫn identity khi không đủ thông tin tracking liên tục` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Đã ra khỏi khung là kết thúc track cũ theo chuẩn MOT` |
| Hai xe cắt nhau / chồng lên nhau | `giữ nguyên ID riêng cho từng xe` | `Mỗi xe duy trì track_id độc lập, bbox ôm phần nhìn thấy` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `xe rõ hình dạng xe 4 bánh và nhìn thấy biển số/đèn xe` |
| Xe đang đỗ, không di chuyển | `vẫn gán vehicle và giữ nguyên 1 track_id duy nhất suốt thời gian xuất hiện` |
| Keyframe đặt dày ở đâu | `đặt dày ở chỗ xe rẽ, phanh, đổi hướng hoặc bị che khuất` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / Frame 1 đến Frame 11 / ID 2`
- Tình huống: `Bbox xuất hiện ở rìa trái ảnh (x = 122.58 ở frame 1), di chuyển ra khỏi khung hình và diện tích giảm dần qua từng frame (đến frame 11 x = 0.3, width giảm còn 26.2). Sang frame 12 thì không còn xuất hiện nữa.`
- Quyết định: `Vẽ bbox ôm sát phần nhìn thấy ở rìa ảnh. Đến frame 11 bấm `outside` (phím O) để kết thúc track tại frame 12.`
- Lý do: `Tuân thủ luật bbox cắt bởi rìa ảnh (không đoán phần ngoài) và phải bấm `outside` ngay khi xe rời hoàn toàn khỏi khung hình để tránh lỗi bbox treo.`

### Ca 2
- Clip / frame / ID: `clip_01 / Frame 12 đến Frame 45 / ID 3`
- Tình huống: `Bbox xuất hiện ở phía bên phải khung hình (x = 572.33 ở frame 12), chạy dần về phía bên trái (x = 0.28 ở frame 45) và kích thước width bị cắt hẹp dần khi đi qua rìa trái. Từ frame 46 trở đi xe biến mất hoàn toàn.`
- Quyết định: Duy trì liên tục `track_id = 3` từ frame 12 đến frame 45, bấm `outside` từ frame 46.
- Lý do: `Xe di chuyển liên tục qua khung hình cần giữ nguyên 1 ID duy nhất. Xe bị cắt ở rìa chỉ vẽ phần còn nhìn thấy được cho tới khi ra khỏi hẳn.`

### Ca 3
- Clip / frame / ID: `clip_01 / Frame 73 đến Frame 80 / ID 4 và ID 5`
- Tình huống: `Tại frame 73, xe ID 5 mới bắt đầu xuất hiện (x = 822.48, width = 15.73) ngay phía sau/gần kề xe ID 4 (x = 822.47, width = 137.53). Hai xe có tọa độ x gần trùng nhau ở vùng giao cắt/che khuất một phần.`
- Quyết định: `Gán hai `track_id` độc lập (ID 4 và ID 5). Với xe ID 5, chỉ vẽ bbox bắt đầu từ frame 73 khi xác định rõ là một xe riêng biệt và chỉ ôm phần nhìn thấy được.`
- Lý do: `Khi hai xe chồng lên nhau/cắt nhau, mỗi xe phải duy trì track_id riêng và bbox chỉ ôm phần vật thể nhìn thấy, không gộp chung ID.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Cần bấm phím `O` (`outside`) ngay lập tức tại frame xe bắt đầu ra khỏi rìa ảnh để tránh lỗi bbox treo.`
- `Đối với xe chuyển động đổi hướng, cần kiểm tra kỹ frame ở giữa (midpoint) của hai keyframe để phát hiện bbox bị trôi.`
