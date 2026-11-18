# Phụ lục hướng dẫn gán nhãn v1.1 — Lê Trần Phi Hùng

## 1. Khi nào giữ box máy?

**CAL-01 · Câu hỏi 7.** Trước khi giữ một box, hãy tìm vật trong ảnh, xác nhận lớp rồi kiểm tra cả bốn cạnh. Áp dụng cùng cách duyệt cho mọi mức confidence. Trong lần chạy r03, YOLOE đề xuất một `three_wheeler` ở mép trái ảnh với điểm 0,28 dù tại đó không có xe ba bánh tương ứng. [Ảnh img_011](../../data/images/img_011.jpg)

Câu 1–6 còn mở. Ba ca trong vòng này chưa đủ để chốt cách phân loại xe điện, tách xe đỗ dày, xử lý hàng nhô ra, ước lượng phần người bị che, vẽ kính xe hoặc xử lý box chồng trong ảnh đêm.

## 2. Hai ca áp dụng quy tắc hiện có

| Ca | Dấu hiệu quan sát | Cách xử lý |
|---|---|---|
| CAL-02 | SAM tô cả áo và chân người lái vào thân tuk-tuk. | Theo mục 4.2.4, lần theo đường tiếp xúc giữa người và xe. Loại phần người ở phía trước khỏi mask xe. [Mask img_013](../masks/tuktuk_img013.jpg) |
| CAL-03 | Phép chấm bẫy ghi nhận box lỏng ở góc trên bên phải của [ảnh img_013](../../data/images/img_013.jpg). | Theo mục 4.1, so từng cạnh box với biên phần vật nhìn thấy. Thu cạnh còn ôm nền thừa. [Kết quả chấm bẫy](../qc/review_stats.md) |

## 3. Lịch sử thay đổi

v1.1 bổ sung cách quyết định giữ box theo CAL-01 và ghi hai ví dụ áp dụng quy tắc biên ở CAL-02 và CAL-03.
