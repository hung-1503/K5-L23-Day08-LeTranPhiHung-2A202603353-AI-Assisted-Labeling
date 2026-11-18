# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 727; đối tượng sau duyệt: 683; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 621 | 621/727 = 85.4% | 727 box máy |
| edited (sửa) | 22 | 22/727 = 3.0% | 727 box máy |
| deleted (xoá) | 78 | 78/727 = 10.7% | 727 box máy |
| added (vẽ thêm) | 40 | 40/683 = 5.9% | 683 đối tượng sau duyệt |

Trong số deleted + added, **14 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 26.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 348 | 4 | 25 | 10 |
| bicycle | 3 | 1 | 2 | 6 |
| motorbike | 202 | 10 | 22 | 11 |
| car | 50 | 2 | 3 | 3 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 3 | 4 |
| three_wheeler | 14 | 5 | 22 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 46 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 35 | 7 | 52.7% (KTC 95%: 41–64%) | 84.8% (KTC 95%: 72–92%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - missing: 3/3
  - extra: 3/3
  - loose: 2/3
  - lọt: img_005.jpg (wrong_class), img_013.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
