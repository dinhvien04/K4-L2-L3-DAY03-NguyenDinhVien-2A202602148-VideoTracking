# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nhóm Học viên (Day 3 - Video Tracking)`
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

Bổ sung của nhóm (nếu có): Xe đang đỗ tĩnh ở lề đường vẫn tính là `vehicle` và bắt buộc phải gán xuyên suốt toàn bộ thời gian nó xuất hiện trong video.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Đây là cùng một thực thể xe, việc giữ nguyên ID giúp chỉ số `AssA` và `IDF1` không bị phạt do tách track. |
| Xe bị che lâu hơn ngưỡng trên | Mở track ID mới | Khi bị che quá lâu (> 2s), khó xác định chắc chắn xe hiện ra có phải xe cũ hay không, tránh nhầm lẫn gán sai ID. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Quy ước chung của bài toán MOT: một khi xe đã hoàn toàn rời khỏi góc nhìn camera thì quãng đời (trajectory) của track đó kết thúc. |
| Hai xe cắt nhau / chồng lên nhau | Duy trì ID độc lập cho từng xe; không đổi ID | Mỗi xe là một thực thể riêng biệt; gán trọn vẹn từng xe trước để đảm bảo không bị ID Switch khi hai xe đan chéo. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh (x=0, y=0 hoặc x=width, y=height), tuyệt đối không phỏng đoán phần nằm ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm sát phần **nhìn thấy được** (visible part), không vẽ bao trùm phần bị che khuất |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: nhìn thấy $\ge 15\%$ thân xe hoặc thấy rõ 1 cụm bánh/đèn |
| Xe đang đỗ, không di chuyển | Gán 1 track duy nhất từ frame đầu đến frame cuối clip; kiểm tra không để bbox bị xê dịch |
| Keyframe đặt dày ở đâu | Đặt dày (cách 3-5 frame) ở khúc xe rẽ, đổi hướng, thay đổi tốc độ hoặc bị che khuất; đặt thưa (15-25 frame) khi xe đi thẳng đều |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / frame 1 - 190 / `Track 3`
- Tình huống: Chiếc xe màu tối đỗ ở lề đường, không hề di chuyển trong suốt 190 frame của video.
- Quyết định: Bắt buộc gán đầy đủ từ frame 1 đến frame 190.
- Lý do: Schema yêu cầu gán mọi xe 4 bánh nhìn thấy được. Xe đỗ vẫn là vật thể thật; bỏ sót sẽ bị phạt 190 False Negative (FN), kéo MOTA tụt sâu.

### Ca 2
- Clip / frame / ID: `clip_01` / frame 100 - 101 / `Track 6`
- Tình huống: Chiếc xe bắt đầu tiến vào từ mép phải trên của khung hình. Ở frame 100 chỉ mới nhú một góc cản trước rất mờ.
- Quyết định: Không gán ở frame 100 (xoá box nháp ID -1), bắt đầu track từ frame 101 khi đã nhận diện rõ hình dạng đầu xe.
- Lý do: Tránh gán quá sớm khi chưa xác định được rõ vật thể, khớp đúng với quy ước xuất hiện của Ground Truth.

### Ca 3
- Clip / frame / ID: `clip_01` / frame 110 - 125 / `Track 4` và `Track 5`
- Tình huống: Hai xe chạy song song cùng chiều và gần như che khuất lẫn nhau ở góc nhìn camera.
- Quyết định: Vẽ bbox của Track 5 ôm sát phần thân xe nhìn thấy được (không bao gồm phần bị Track 4 che). Giữ nguyên ID 4 và ID 5 riêng rẽ.
- Lý do: Tránh lỗi bbox quá rộng làm giảm MOTP, đồng thời ngăn ngừa triệt để lỗi hoán đổi ID (ID switch).

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Bổ sung luật xe đỗ tĩnh**: Làm rõ trong phần 1 và phần 3 rằng mọi xe đỗ trong vùng nhìn thấy đều phải được theo dõi liên tục, không chỉ theo dõi xe có vận tốc chuyển động.
- **Quy định rõ ngưỡng bắt đầu track ở rìa ảnh**: Cụ thể hoá ngưỡng $\ge 15\%$ thân xe nhìn thấy để tránh việc hai người trong nhóm bắt đầu track lệch nhau 3-5 frame ở mép khung hình.
