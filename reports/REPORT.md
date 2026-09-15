# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Nguyễn Đình Viễn (MSSV: 2A202602148)
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (Track Mode, Bounding Box) |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 75 phút |
| Số track đã vẽ trong `clip_01` | 8 track (sau khi hoàn thiện bổ sung xe đỗ) |
| Số keyframe trung bình mỗi track | 5 keyframe / track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe đang đỗ tĩnh suốt toàn bộ video (Track 3)**:
   - *Tình huống*: Chiếc xe màu tối đỗ ở lề đường từ frame 1 đến frame 190. Ban đầu trong lượt gán đầu tiên rất dễ bỏ qua xe này do mắt người có xu hướng chỉ chú ý vào các vật thể chuyển động.
   - *Cách xử lý*: Tạo 1 track riêng biệt kéo dài xuyên suốt từ frame 1 đến 190. Đặt keyframe ở đầu và cuối và kiểm tra định kỳ để bbox không bị xê dịch khỏi thân xe.

2. **Xe ra / vào khung hình ở mép ảnh (Track 1, 2 rời khung; Track 6, 7 vào khung)**:
   - *Tình huống*: Khi xe bắt đầu tiến vào hoặc chuẩn bị rời khỏi góc nhìn camera, một phần xe bị cắt bởi rìa ảnh. Nếu đoán phần ngoài ảnh thì bbox sẽ sai lệch, còn nếu quên bấm kết thúc thì bbox sẽ trôi lơ lửng (ghost bbox).
   - *Cách xử lý*: Tuân thủ nghiêm luật bbox: chỉ khoanh phần nhìn thấy được chạm sát mép ảnh, tuyệt đối không vẽ vượt ra ngoài. Ngay tại frame xe hoàn toàn khuất khỏi màn hình, bấm phím tắt `O` (outside) trên CVAT để ngắt track đúng thời điểm.

3. **Hai xe chạy gần nhau và che khuất nhau một phần (Track 4 và Track 5 ở đoạn frame 100-120)**:
   - *Tình huống*: Hai xe di chuyển song song, có thời điểm bbox của xe sau chồng lấn lên xe trước, rất dễ gây nhầm lẫn hoán đổi ID (ID switch) hoặc gộp nhầm track.
   - *Cách xử lý*: Áp dụng quy tắc "gán trọn vẹn từng xe từ đầu đến cuối trước khi sang xe khác". Với xe bị che một phần, bbox chỉ ôm trọn phần thân xe còn nhìn thấy được; duy trì đúng ID của từng xe xuyên suốt quá trình giao nhau mà không tách ID.

---

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1 (Kiểm tra ID)**: Phát nhanh toàn bộ clip ở tốc độ cao và tập trung nhìn vào số ID hiển thị trên từng bbox. Kết quả: Tất cả 8 xe đều giữ nguyên ID ổn định từ lúc vào đến lúc ra, không có hiện tượng nhấp nháy đổi số hoặc nhảy ID (ID switch = 0).
- **Lượt 2 (Kiểm tra biên track — Frame đầu / Frame cuối)**: Soi kỹ frame bắt đầu và frame kết thúc của từng track. Phát hiện ở frame 100 có 1 box thừa với `track_id = -1` (vẽ sớm trước khi Track 6 thực sự xuất hiện ở frame 101); đã loại bỏ box này để file đạt chuẩn MOT 1.1 không có ID âm.
- **Lượt 3 (Kiểm tra độ khít giữa các keyframe)**: Nhảy vào các frame ở giữa hai keyframe cách xa nhau (đoạn interpolation tự động). Tinh chỉnh lại một số frame ở khúc xe đổi góc nhìn phối cảnh (perspective) để bbox không bị trôi hay lỏng (đảm bảo MOTP và LocA cao).

Kiểm chéo với: Bạn Nguyễn Văn A (nhóm đối tác). Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 3 lỗi (1 lỗi quên bấm outside để bbox treo 8 frame, 1 xe ở rìa bị gán trễ 5 frame, 1 lỗi gán nhầm xe máy ngoài schema).
Số lỗi bạn ấy tìm được trong bản của bạn: 1 lỗi (phát hiện box thừa ID -1 ở frame 100).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?
- *Ca bất đồng*: Thời điểm bắt đầu track của xe vừa tiến vào rìa ảnh (Track 6 ở frame 101). Một bạn bắt đầu khi xe mới lú đầu khoảng 10% diện tích, bạn còn lại chờ xe vào rõ 30% mới vẽ.
- *Luật bổ sung vào `GUIDELINE_MINI.md`*: Quy định rõ ngưỡng tối thiểu để bắt đầu track xe ở mép ảnh là $\ge 15\%$ diện tích thân xe nhìn thấy được (hoặc thấy rõ ít nhất 1 cụm đèn/bánh xe).

---

## 3. Chấm với gold — trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Lần chấm đầu | 0.635 | 0.521 | 0.776 | 0.852 | 0.771 | 0.607 | 0.832 | 31 | 194 | 0 |
| Sau rework | 0.780 | 0.770 | 0.793 | 0.846 | 0.970 | 0.939 | 0.826 | 31 | 4 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **CÓ (ĐẠT XUẤT SẮC)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bỏ sót track (FN) | 1-190 | 3 | Bổ sung hoàn chỉnh Track 3 (xe đang đỗ ở lề đường, 190 frame). Giảm ngay 190 FN giúp MOTA nhảy vọt từ 0.607 lên 0.939 |
| ID âm / Box thừa (FP) | 100 | -1 | Xoá bỏ bounding box vẽ thử nghiệm chưa gán ID ở frame 100 (xe Track 6 trong gold bắt đầu từ frame 101) |
| Bbox trôi / Chưa khít | 51-54 | 4 | Tinh chỉnh lại keyframe của Track 4 ở thời điểm mới vào khung hình cho khớp sát mép thân xe |

---

## 4. Kết quả model và so sánh ba chiều

Cấu hình: model `yolo26n.pt`, tracker `bytetrack.yaml`, conf `0.25`, imgsz `960`

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.780 | 0.770 | 0.793 | 0.846 | 0.970 | 0.939 | 0.826 | 31 | 4 | 0 |
| model vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| model vs bạn | 0.645 | 0.591 | 0.708 | 0.811 | 0.850 | 0.707 | 0.778 | 90 | 83 | 3 |

*(Ghi chú: Kết quả so sánh model phản ánh đặc trưng zero-shot của YOLO+ByteTrack khi chưa fine-tune trên dữ liệu góc quay CCTV chuyên biệt).*

---

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- Trong bài của em, cả hai chỉ số đều rất cao và **IDF1 (0.970) cao hơn MOTA (0.939)**. Điều này cho thấy chất lượng liên kết track (association) đạt độ chính xác gần như hoàn hảo (`ID switch = 0`).
- Nếu gặp trường hợp **MOTA cao mà IDF1 thấp**, điều đó phản ánh nhãn phát hiện vật thể tốt (ít bỏ sót, ít box thừa) nhưng **bị nhảy ID hoặc bị tách track nghiêm trọng**.
- **Lý do MOTA không phạt nặng lỗi ID**: MOTA được định nghĩa là $1 - \frac{\text{FP} + \text{FN} + \text{IDSW}}{\text{GT}}$, nghĩa là mỗi lần đổi ID chỉ bị tính là **1 lỗi** (IDSW = 1). Nếu một chiếc xe chạy 100 frame nhưng bị đứt ID ở giữa thành 2 track (mỗi track 50 frame), MOTA chỉ bị trừ đúng 1 đơn vị lỗi (MOTA vẫn có thể $\ge 95\%$). Ngược lại, **IDF1** đo lường sự tương thích ID trên toàn bộ tuổi thọ của vật thể qua bài toán ghép cặp 1-1 (bipartite matching) — việc track bị cắt đôi sẽ khiến 50 frame còn lại bị tính toàn bộ thành IDFP và IDFN, làm IDF1 tụt dốc thảm hại.

**2. DetA và AssA của model lệch nhau bao nhiêu? Cái nào kéo HOTA xuống — model không tìm ra xe, hay tìm ra rồi nhưng đánh mất ID?**

- Với kết quả model vs gold, **DetA (0.649) thấp hơn đáng kể so với AssA (0.776)**, độ lệch là **0.127**.
- **DetA chính là yếu tố kéo HOTA xuống**. Model YOLO chạy zero-shot từ tập pre-trained COCO gặp khó khăn trong việc phát hiện các xe ở xa có kích thước pixel nhỏ hoặc các xe có góc chụp từ trên cao (CCTV view) khác với góc chụp ngang phổ biến của COCO (gây 54 FN và 88 FP).
- Ngược lại, AssA của ByteTrack đạt mức rất tốt (0.776, chỉ có 2 lần ID switch) vì thuật toán Kalman Filter kết hợp IoU 2 vòng (giữ lại cả các detection điểm thấp ở vòng 2) giúp duy trì ID bền bỉ một khi xe đã được phát hiện.

**3. Một chỗ bạn đúng và model sai (frame, ID, vì sao):**

- **Vị trí**: Frame 1 đến 60, đối với chiếc xe đỗ tĩnh ở lề đường (Track 3).
- **Phân tích**: Con người có tri thức ngữ cảnh (context) nên dễ dàng nhận biết chiếc xe đỗ dù xe không chuyển động và bị bóng râm che phủ một phần. Model YOLO thường bị giảm confidence ở xe tĩnh hoặc ByteTrack tự động xóa track sau khi detector miss liên tiếp một số frame (`max_time_lost`), dẫn đến việc model bỏ sót hoặc tách xe đỗ thành nhiều mẩu track vụn.

**4. Một chỗ model đúng và bạn sai (frame, ID, vì sao):**

- **Vị trí**: Frame 51 đến 53, đối với Track 4 khi vừa xuất hiện ở mép dưới bên phải.
- **Phân tích**: Bằng mắt thường, học viên bắt đầu vẽ bbox từ frame 51 khi phần đầu xe vừa chớm vào. Tuy nhiên, trong Gold track này chỉ bắt đầu từ frame 54 (khi xe đã lộ diện rõ hơn). Detector của model nhờ ngưỡng ngưỡng NMS và độ nhạy pixel đã kích hoạt đúng thời điểm tương đương Gold hơn, tránh được việc vẽ quá sớm ở rìa khi vật thể chưa đủ điều kiện nhận diện.

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

- Loại bất đồng nhiều nhất là **"Bạn có bbox, model không có" (Model FN - Bỏ sót)**.
- **Ý nghĩa về clip**:
  1. Clip giao thông CCTV có góc quay góc rộng (wide angle) từ trên cao, khiến các xe ở phía xa (như đoạn trên của giao lộ) có kích thước rất nhỏ và mờ.
  2. Môi trường ánh sáng đường phố có bóng đổ phức tạp và các vật thể tĩnh (xe đỗ). Detector tổng quát pre-train trên COCO khó đạt recall cao trên miền dữ liệu này nếu không được fine-tune chuyên biệt trên dataset UA-DETRAC.

---

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

1. **Bổ sung rõ ràng trong `GUIDELINE_MINI.md`**:
   - Quy định rõ ràng quy tắc cho xe đỗ tĩnh: bắt buộc phải gán đầy đủ từ đầu đến cuối clip nếu xe nằm trong khung hình, tránh bỏ sót như bài học ở lần chấm đầu.
   - Thống nhất ngưỡng kích thước/tỷ lệ hiển thị tối thiểu để bắt đầu track xe ở mép khung hình ($\ge 15\%$ diện tích xe hoặc thấy rõ 1 cụm bánh/đèn).
2. **Cải tiến quy trình làm việc**:
   - **Thứ tự ưu tiên gán**: Luôn rà soát và khoanh vùng các xe đỗ tĩnh trước tiên, sau đó mới gán từng xe chuyển động theo thứ tự xuất hiện.
   - **Tận dụng phím tắt CVAT**: Sử dụng phím `O` (outside) ngay khi xe khuất và `M` (merge) khi nối track thay vì vẽ lại để tránh hoàn toàn lỗi sinh ID âm (`-1`) hoặc tách track.
   - **Kiểm tra 3 lượt tự động**: Luôn chạy script `tools/check_mot_labels.py` ngay sau khi export để phát hiện sớm các lỗi định dạng trước khi bước vào khâu đánh giá chất lượng.

---

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt` (Đã chuẩn hoá, 0 lỗi định dạng, đạt cổng xuất sắc)
- [x] `annotations/clip_02/gt.txt` (Bài warm-up)
- [x] `GUIDELINE_MINI.md` đã điền đầy đủ
- [x] `outputs/eval_vs_gold.json` (Kết quả chấm vs gold của clip_01)
- [x] `outputs/model_clip_01.txt`
- [x] `outputs/eval_model_vs_gold.json`, `outputs/eval_model_vs_me.json`
- [x] `reports/review_partner.md` (Biên bản kiểm chéo)
- [x] `reports/REPORT.md` (Báo cáo tổng kết hoàn chỉnh)
