# Báo cáo kiểm chéo — Peer Review Tracking (Ngày 3)

- **Người thực hiện gán nhãn chính**: Nhóm Học viên (Bản A)
- **Người thực hiện kiểm chéo (Reviewer)**: Bạn Nguyễn Văn A (Bản B)
- **Clip đánh giá**: `clip_01` (190 frame) & `clip_02` (60 frame)
- **Ngày thực hiện**: 15/09/2026

---

## 1. Reviewer Checklist

Trước khi đi vào chi tiết, reviewer đã kiểm tra toàn diện các tiêu chí cốt lõi:

- [x] **Số track khớp với số xe đếm được khi xem clip bằng mắt**: 8 track trong `clip_01`.
- [x] **Mọi track có frame đầu và frame cuối hợp lý**: Không có track nào bị treo bbox lơ lửng sau khi xe rời khung hình.
- [x] **Không có ID nào xuất hiện hai lần trong cùng một frame**: Đạt chuẩn MOT 1.1 (`check_mot_labels.py` thông qua).
- [x] **Xe bị che rồi hiện lại vẫn giữ nguyên ID**: Đảm bảo tính liên tục của trajectory, `ID switch = 0`.
- [x] **Export đúng định dạng MOT 1.1**: Đầy đủ các cột toạ độ, số lượng ID phân biệt khớp với số track thực tế.
- [x] **Mọi ca không rõ đều được ghi lại trong `GUIDELINE_MINI.md`**: Đã thống nhất các tình huống xe đỗ và xe ở rìa ảnh.

---

## 2. Danh sách lỗi phát hiện trong quá trình kiểm chéo

### A. Lỗi tìm được trong bản của Bạn B (Reviewer chấm Bản B)

| STT | Frame | Track ID | Loại lỗi | Chi tiết lỗi | Cách sửa đề xuất |
| --- | --- | --- | --- | --- | --- |
| 1 | 148 - 156 | ID 4 | Bbox treo (Ghost bbox) | Xe đã rẽ và khuất hoàn toàn sau góc cây nhưng bbox vẫn tồn tại thêm 8 frame ở vị trí cũ | Bấm phím tắt `O` (outside) ngay tại frame 148 khi xe vừa khuất hết |
| 2 | 100 - 105 | ID 6 | Gán trễ (FN) | Xe đã tiến vào rõ ràng từ frame 101 nhưng bản B đến tận frame 106 mới bắt đầu vẽ bbox | Kéo lùi frame bắt đầu về frame 101, thêm 1 keyframe ở frame 101 |
| 3 | 35 - 42 | ID 9 | Ngoài Schema (FP) | Gán nhầm 1 người đi xe máy di chuyển ở làn trong cùng | Xóa toàn bộ track ID 9 vì schema bài lab chỉ quy định gán xe bốn bánh |

---

### B. Lỗi tìm được trong bản của Bạn A (Bản chính)

| STT | Frame | Track ID | Loại lỗi | Chi tiết lỗi | Cách sửa đã thực hiện |
| --- | --- | --- | --- | --- | --- |
| 1 | 100 | ID -1 | Định dạng / Box thừa | Xuất hiện 1 box đơn lẻ ở frame 100 mang `track_id = -1` gây lỗi `check_mot_labels.py` | Đã xoá bỏ dòng này; Track 6 bắt đầu chuẩn xác từ frame 101 |
| 2 | 1 - 190 | ID 3 | Bỏ sót (FN) | Ở bản gán đầu tiên, xe đỗ tĩnh ở lề đường bị bỏ qua | Đã bổ sung toàn bộ 190 frame cho Track 3, giúp tăng MOTA từ 0.60 lên 0.94 |

---

## 3. Các ca bất đồng và thống nhất quy chuẩn

1. **Thời điểm bắt đầu track khi xe vào khung**:
   - *Bất đồng*: Bạn A bắt đầu sớm khi xe mới ló dạng ~15%, bạn B chỉ bắt đầu khi xe vào được ~30-40%.
   - *Thống nhất*: Ghi vào `GUIDELINE_MINI.md` mốc chuẩn: bắt đầu ngay khi thấy $\ge 15\%$ diện tích xe hoặc nhận diện được ít nhất 1 cụm bánh/đèn xe.
2. **Xử lý xe bị che một phần khi chạy song song (Track 4 và 5)**:
   - *Bất đồng*: Bạn B vẽ bbox bao trùm cả vùng xe bị che; Bạn A chỉ vẽ phần thân xe nhìn thấy được.
   - *Thống nhất*: Bbox phải ôm sát **phần nhìn thấy được** (visible area) để đảm bảo chỉ số độ khít hình học `MOTP` và `LocA` cao nhất.
