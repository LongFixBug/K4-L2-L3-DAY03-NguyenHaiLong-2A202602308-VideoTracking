# Mini annotation guideline — Ngày 3 (tracking)

> Bản tổng hợp bổ sung sau lần chạy notebook và kiểm tra ảnh, ngày 15/09/2026.
> Nội dung dựa trên `GUIDE.md`, `CVAT_TASK_SPEC.md` và evidence trong báo cáo;
> không phải nhật ký được ghi trong lúc gán nhãn. Các ca bên dưới mô tả dữ liệu
> hiện có và cách kiểm tra, không ghi nhận thao tác sửa ID/bbox hoặc kiểm chéo chưa diễn ra.

Nhóm / tên: **Nguyễn Hải Long — 2A202602308**
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

Bổ sung: xe thật đang đỗ vẫn thuộc lớp `vehicle`. Không gán quầy hàng, biển
quảng cáo hoặc cấu trúc bên đường chỉ vì model dự đoán là xe. Các ID của model
và annotation là hai hệ đánh số riêng; cùng số không có nghĩa là cùng vật thể.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ cùng ID khi vẫn nhận ra cùng xe; với đoạn mất dấu ngắn, áp dụng ngưỡng lab dưới 2 giây (25 frame ở 12.5 fps). Bật Occluded khi phù hợp | Che khuất không tự tạo ra vật thể mới |
| Xe bị che lâu hơn ngưỡng trên | Theo GUIDE: quá 2 giây thì mở track mới. Ca đúng ranh giới 25 frame cần thống nhất với Lab Coach vì GUIDE chưa nêu rõ dấu bằng | Tránh nối identity khi evidence liên tục không đủ |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới**; kết thúc track cũ bằng Outside | Không kéo track qua thời gian xe ngoài ảnh |
| Hai xe cắt nhau / chồng lên nhau | Theo dõi từng xe ở trước, trong và sau chồng lấp; dùng hướng đi, vị trí và đặc điểm nhìn thấy để đối chiếu; không đổi ID theo thứ tự trái/phải hoặc theo ID model | Thứ tự vị trí có thể đổi trong khi identity vẫn giữ nguyên |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu từ frame đầu tiên xác định được là xe bốn bánh. Chưa có ngưỡng pixel riêng được ghi nhận; dùng tiêu chí nhận diện của lab, xem frame lân cận để xác minh và không lấy confidence model làm ngưỡng annotation |
| Xe đang đỗ, không di chuyển | Vẫn gán và duy trì track khi xe còn nhìn thấy trong ảnh. Cảnh báo bbox đứng im cần kiểm ảnh, không tự xóa track |
| Keyframe đặt dày ở đâu | Đặt dày quanh lúc xe vào/ra ảnh, rẽ/phanh, bị che hoặc bbox đổi hình dạng nhanh; kiểm frame giữa hai keyframe để phát hiện nội suy trôi. Chưa có khoảng cách keyframe cố định được ghi nhận |

Khi xe rời khung hình, đặt Outside theo frame thực tế; không để bbox nội suy
treo sau khi xe đã đi. Nếu phải chỉnh nhãn, sửa trong công cụ gán nhãn rồi export
MOT lại theo quy trình lab, không chỉnh tọa độ/ID trực tiếp trong MOT để tăng điểm.

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Các ca dưới đây đã được kiểm tra trong ảnh đầu ra notebook. `B#` là annotation,
`T#` là ReID; không dùng sự khác số ID đơn thuần để kết luận ID switch.

### Ca 1
- Clip / frame / ID: `clip_01`, frame **107**, model **T7**.
- Tình huống: bbox model nằm trên khu vực quầy/biển bên đường, không có bbox annotation tương ứng.
- Quyết định theo phạm vi lớp: khu vực này không được thêm vào nhãn chỉ vì model gọi là xe; chưa thực hiện chỉnh annotation.
- Lý do: ảnh cho thấy cấu trúc bên đường; định nghĩa lớp chỉ lấy xe bốn bánh thật. Đây là ví dụ model có bbox nhưng chưa phải người gán bỏ sót.
- Evidence: [frame 107](reports/evidence/frame_107.png).

### Ca 2
- Clip / frame / ID: `clip_01`, frame **107–109**, annotation **B7** (xe tải mép phải).
- Tình huống: xe mới vào ảnh, chỉ thấy một phần. Frame 107 không có bbox ReID chồng lấp B7; frame 108, T29 chỉ đạt IoU 0,4973; frame 109 đạt 0,5321.
- Quyết định trong dữ liệu hiện tại: annotation có B7; kiểm frame đầu theo ảnh và luật cắt bbox tại biên, không xóa nhãn vì model chưa khớp.
- Lý do: xe tải nhìn thấy ở rìa phải của frame 107. Ngưỡng IoU 0,5 là ngưỡng đánh giá, không phải ngưỡng để quyết định xe có tồn tại.
- Evidence: [frame 107](reports/evidence/frame_107.png); đối chiếu `outputs/model_reid_clip_01.txt` với annotation.

### Ca 3
- Clip / frame / ID: `clip_01`, frame **104–113**, annotation **B6**, model **T24 → T31**.
- Tình huống: xe B6 bị xe buýt B4 che một phần. ReID khớp T24 ở frame 104 (IoU 0,5220); log CLEAR ghi đổi sang T31 tại frame 113 (IoU 0,5661).
- Quyết định trong dữ liệu hiện tại: annotation giữ B6. Khi review cần theo dõi liên tục xe qua đoạn che và khoanh phần nhìn thấy; không đổi annotation sang ID mới chỉ để giống tracker.
- Lý do: đổi ID của model là finding cần kiểm, không tự chứng minh annotation sai. Đoạn ảnh cho thấy tình huống chồng lấp cần áp dụng luật ID và bbox đồng thời.
- Evidence: [frame 107](reports/evidence/frame_107.png), [frame 113](reports/evidence/frame_113.png), `outputs/eval_reid_vs_gold.json`.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Những điểm được làm rõ sau phân tích notebook:

- Xe đứng yên vẫn được gán; quầy/biển không được gán. Cảnh báo B1 đứng im ở frame 1–15 cần kiểm ảnh, không coi là lỗi Outside mặc định.
- Với xe ở mép ảnh, kiểm frame bắt đầu và cắt bbox tại biên; không dùng khả năng phát hiện của model làm tiêu chuẩn chọn frame.
- Với chồng lấp, giữ identity theo chuỗi quan sát và bbox phần nhìn thấy; xem frame trước/trong/sau thay vì chỉ xem một ảnh.
- Phân biệt đổi ID thực sự với cách đánh số khác nhau giữa model và annotation. ReID có IDF1 cao hơn ByteTrack trong lần chạy này nhưng vẫn có 2 ID switch.
- Ghi lại frame/ID và lý do mỗi lần sửa trong công cụ gán nhãn, export lại, chạy validator rồi đánh giá lại. Không sửa snapshot pre-gold đã khóa.

**Trạng thái evidence:** annotation, snapshot và gold hiện tại trùng hash nên
đều cho điểm 1,0000; chưa có chênh lệch file chứng minh rework. Chưa có
`reports/review_partner.md` hoặc thông tin người kiểm chéo, vì vậy các mục trên
không được ghi là kết luận đã thống nhất với peer reviewer.
