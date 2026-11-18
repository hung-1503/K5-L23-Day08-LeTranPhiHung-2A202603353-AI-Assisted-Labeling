# Báo cáo Day 08 — 2A202603353 - Lê Trần Phi Hùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26-m | 0.15 | Đạt recall 93% (mẫu số 15 vật gold), vượt mục tiêu 90%; chấp nhận precision 14/32 = 44% để giữ recall (`r05`). |
| bicycle | YOLO26-m | mặc định 0.3 | Không có ngưỡng ≤0.5 đạt recall 90% (cao nhất 60%, mẫu số 5); giữ 0.3 của recipe thay vì ép chọn ngưỡng riêng (`r05`). |
| motorbike | YOLO26-m | 0.1 | Đạt recall 90% (mẫu số 10 vật gold), precision 9/18 = 50%; đây là ngưỡng cao nhất còn giữ mục tiêu recall (`r05`). |
| car | YOLO26-m | mặc định 0.3 | Không có ngưỡng ≤0.5 đạt recall 90% (cao nhất 88%, mẫu số 8); giữ 0.3 của recipe vì tăng ngưỡng sẽ bỏ sót xe (`r05`). |
| bus | YOLO26-m | mặc định 0.3 | Chỉ có 0 vật trong 4 ảnh gold, mẫu quá ít (<5); không chọn ngưỡng riêng, dùng 0.3 của recipe (`r05`). |
| truck | YOLO26-m | mặc định 0.3 | Chỉ có 1 vật trong 4 ảnh gold, mẫu quá ít (<5); không chọn ngưỡng riêng, dùng 0.3 của recipe (`r05`). |
| three_wheeler | YOLOE-s, prompt: `three_wheeler` | 0.15 | Đạt recall 100% và precision 7/9 = 78% (mẫu số 7 vật gold); tốt hơn dùng YOLO26 cho lớp này (`r06`). |

Nguồn số đo là [bảng quét ngưỡng trên 4 ảnh gold](qc/threshold_sweep.csv). Hai đầu vào là dự đoán YOLO26-m (`r05`) và YOLOE-s (`r06`), cùng chạy ở ngưỡng 0.05.

Lệnh sau tái tạo đúng tệp đã gộp (845 box, khớp SHA-256). Giá trị 0.3 được truyền qua `--conf` cho các lớp còn lại.

```bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --thr "person=0.15,motorbike=0.1,three_wheeler=0.15" --split batch,gold --out work/prelabel/merged_review/pre.coco.json
```

- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`), mask bám khá tốt người/xe rõ; vẫn cần review khi box đầu vào rộng hoặc chồng (`r04`).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Tuk-tuk bị YOLO26 gọi `car` là lỗi lớp, không chỉ là lỗi confidence; dùng YOLOE với prompt `three_wheeler` rồi review thủ công các box thấp điểm (`r03`).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): YOLOE có box `three_wheeler` giả ở mép trái `img_011` (0.28); các tuk-tuk rõ ở `img_002`/`img_032` được nhận đúng, nhưng một số box 0.27–0.40 cần kiểm tra (`r03`).
- Giới hạn: chỉ có 4 ảnh gold; hơn nữa ngưỡng được chọn và chấm trên cùng dữ liệu gold, nên precision/recall của máy trên gold là **lạc quan**. Các lớp ít vật chỉ cho ước lượng thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 621 | 85,4% | 727 box máy |
| edited | 22 | 3,0% | 727 box máy |
| deleted | 78 | 10,7% | 727 box máy |
| added | 40 | 5,9% | 683 đối tượng sau duyệt |

- Nguồn: [review_stats.md](qc/review_stats.md) và [review_stats.json](qc/review_stats.json), chạy trên 24 ảnh batch; accepted nghĩa là cùng lớp và IoU >= 0.9, không có nghĩa là đã xác minh đúng ngoài thực tế. Script loại tổng cộng 25 đối tượng pre/final quanh bẫy khỏi bảng. File [batch_reviewed.json](labels/batch_reviewed.json) có 700 đối tượng, gồm 6 `ignore_region`; sau khi bỏ ignore và vùng bẫy còn 683 đối tượng dùng trong bảng. Accepted + edited + deleted = 721, thấp hơn mẫu số 727 vì 6 box máy nằm trong ignore_region không bị tính là deleted.
- Số box bị xóa (78/727) gấp khoảng 3,5 lần số box được chỉnh (22/727). Người duyệt còn vẽ thêm 40/683 đối tượng. Trong 40 lượt thêm, 14 lượt là đổi lớp theo phép ghép của script, nên ước tính có 26/40 lượt thêm do máy bỏ sót.
- Lớp nào máy tệ nhất? `three_wheeler` đáng ưu tiên kiểm tra vì có 22 deleted, 5 edited, 4 added nhưng chỉ 14 accepted. Ví dụ ngoài vùng bẫy: `img_003.jpg` có 8 box máy lớp này, 4 bị xóa và 1 bị chỉnh; `img_014.jpg` có 3 box máy lớp này nhưng không có box cùng lớp trong nhãn cuối. Đây là dấu hiệu box thừa/nhầm lớp so với nhãn đã duyệt, không phải bằng chứng nhãn cuối hoàn toàn đúng. Các lớp ít mẫu như truck (2 accepted, 3 deleted, 4 added) và bicycle (3 accepted, 6 added) cũng yếu, nhưng chưa đủ dữ liệu để xếp hạng chắc chắn.
- So với gold: **máy** có TP=39, FP=35, FN=7; precision = 39/(39+35) = **52,7%** (mẫu số 74 dự đoán, KTC 95%: 41–64%); recall = 39/(39+7) = **84,8%** (mẫu số 46 vật gold, KTC 95%: 72–92%). File gold có 52 đối tượng nhưng 6 là ignore_region, nên recall không dùng mẫu số 52. Chỉ có 4 ảnh gold và ngưỡng được chọn/chấm trên cùng dữ liệu, vì vậy các số đo này **lạc quan**, chưa đại diện chất lượng trên ảnh mới. Gold không có trong nhãn batch sau duyệt nên không đo residual/new error.
- Kiểm tra nhãn bằng `validate_labels.py --all`: gold 4 ảnh/52 đối tượng và batch 24 ảnh/700 đối tượng đều OK về cấu trúc; còn 10 cảnh báo box cao dưới 20 px trên `img_026`, `img_027`, `img_033`, `img_034`. Cần đối chiếu guideline, không tự xóa hay chuyển thành ignore chỉ dựa vào cảnh báo; script không xác nhận đúng lớp/ôm sát vật.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Nguồn: mục 3 của [review_stats.md](qc/review_stats.md). Bắt đúng **10/12 = 83,3%** bẫy, đạt ngưỡng tối thiểu 80% (ít nhất 10/12), nhưng còn lọt **2/12 = 16,7%**. KTC 95% của tỷ lệ bắt bẫy là 55–95%; chỉ 12 bẫy nên chưa thể kết luận người duyệt ổn định trên lô khác.
- Theo loại: missing 3/3, extra 3/3, wrong_class 2/3, loose 2/3. Hai bẫy lọt là `img_005.jpg` (wrong_class) và `img_013.jpg` (loose).
- Nguyên nhân cần kiểm tra: bẫy sai lớp phù hợp với việc kiểm tra chưa kỹ bước 3 (đối chiếu lớp), bẫy box lỏng phù hợp với việc kiểm tra chưa kỹ bước 4 (biên box). Đây là suy luận từ loại lỗi, chưa có nhật ký để khẳng định đã bỏ bước nào. Cần mở lại hai ảnh để xác nhận và rà tương tự trên toàn batch.
- **Accepted cao không tự chứng minh chất lượng tốt:** 621/727 = 85,4% chỉ cho biết nhiều box gần như không đổi; vẫn có 2/12 bẫy chưa sửa đúng. Việc đạt 10/12 bẫy cũng không thay thế QC nhãn. Cần tiếp tục quét ảnh khi ẩn nhãn máy, kiểm tra lớp và biên box ngay cả khi confidence cao.

## 4. Máy có đáng dùng không?

- Gán tay mất 30 phút cho 4 ảnh gold, trung bình **7,50 phút/ảnh**. Duyệt có pre-label mất 80 phút cho 24 ảnh batch, trung bình **3,33 phút/ảnh**. Chênh lệch là **4 phút 10 giây mỗi ảnh**, tương ứng mức giảm **55,6%** so với gán tay, tính bằng `1 − (80/24)/(30/4)`. [Nhật ký thời gian](time_log.csv)
- Hai nhóm ảnh khác nhau về độ khó và mật độ vật. Người duyệt cũng đã quen quy tắc sau khi gán gold. Vì vậy, chênh lệch quan sát được có thể bao gồm cả ảnh hưởng của độ khó và kinh nghiệm.
- Duyệt có pre-label tốn ít thời gian hơn trên hai nhóm ảnh đã đo. Vẫn còn 2/12 bẫy lọt và 10 cảnh báo box nhỏ cần đối chiếu khi QC.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Trên mỗi ảnh, ẩn toàn bộ nhãn máy và quét ít nhất 15 giây, ghi số vật theo lớp trước khi hiện box; kiểm tra đã làm đủ 24/24 ảnh.
  2. Duyệt đủ bốn mục bỏ sót → thừa → sai lớp → box lỏng; kiểm tra lại `img_005.jpg` và `img_013.jpg`, rồi chạy lại thống kê sau khi export để xác nhận hai bẫy đã được sửa đúng, không chỉ nhìn tỷ lệ accepted.
  3. Bấm giờ riêng gold_manual và batch_review, ghi số ảnh/phút vào time_log, không tính thời gian nghỉ; báo cáo cùng tỷ lệ bắt bẫy và QC độc lập với accepted, không suy ra tiết kiệm khi chưa có số đo.

## 5. Mask SAM cho 3 vật (Part 5)

SAM dùng hai loại điểm gợi ý. Bấm vào phần xe muốn tô gọi là **click dương**. Bấm vào phần muốn bỏ khỏi vùng tô gọi là **click âm**. Cả hai đều được tính vào tổng số lần bấm.

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013, tuk-tuk nhìn ngang | SAM 2.1-b | 3 lần (2 vào xe, 1 vào áo người lái) | Lẫn một phần người lái. Không thấy hàng rời. | Mái và thân xe liền mạch, nhưng phần áo và chân sát buồng lái cũng bị tô. | [Ảnh mask](masks/tuktuk_img013.jpg) |
| img_002, tuk-tuk bị xe máy che đầu | SAM 2.1-b | 4 lần (2 vào tuk-tuk, 1 vào người lái, 1 vào người đi xe máy phía trước) | Phần áo xanh của người lái và người đi xe máy phía trước nằm ngoài mask. Không thấy hàng rời. | Mask giữ mái, thân và bánh sau. Qua khoảng hở ở khoang khách, một số chi tiết nền cũng bị tô. | [Ảnh mask](masks/tuktuk_img002.jpg) |
| img_032, xích lô ở tiền cảnh | SAM 2.1-b | 3 lần (2 vào xích lô, 1 vào người đứng cạnh) | Người đứng cạnh xe nằm ngoài mask. Không thấy hàng rời. | Mask bắt được ghế và hai bánh, nhưng tô kín cả khoảng hở giữa các nan bánh. | [Ảnh mask](masks/cyclo_img032.jpg) |

- Mục 4.2 yêu cầu bám phần nhìn thấy và giao vùng tiếp xúc cho vật ở phía trước. Theo đó, phần người lái bị tô ở img_013 cần tách khỏi mask xe. Với img_032, từng khe nan nhỏ hơn khoảng 5% diện tích xe có thể được lấp theo quy tắc lỗ nhỏ. Một ví dụ minh họa cách xét nhiều khe nhỏ liền kề sẽ giúp người duyệt áp dụng ngưỡng nhất quán.

## 6. Calibration (Part 6)

- **Số quyết định.** Có 3 ca. Quan trọng nhất là `CAL-01` vì áp dụng cho mọi box máy đề xuất. Ở `img_011`, YOLOE gán nhầm lớp `three_wheeler` cho một vùng ở mép trái ảnh. [Nhật ký calibration](qc/calibration_decisions.csv)
- **Quy tắc mới trong v1.1.** Với mọi mức confidence, trước khi giữ box, người duyệt phải tìm được vật trong ảnh, xác nhận đúng lớp và kiểm tra cả bốn cạnh. Hai ca còn lại áp dụng quy tắc sẵn có về tách người khỏi mask xe và vẽ box sát phần vật nhìn thấy. [Phụ lục v1.1](guidelines/guideline_v1.1_calibrated.md)
- **Nguyên nhân gốc.** Ở `CAL-01`, v1.0 còn để mở câu hỏi khi nào có thể giữ box dựa vào confidence. Ở `CAL-02`, SAM tô lẫn áo và chân người lái vào mask tuk-tuk. Ở `CAL-03`, box lỏng vẫn còn sau duyệt. Nhật ký chưa đủ để xác định vì sao người duyệt bỏ lọt lỗi này.

## 7. QC và quyết định (Part 6)

- **Mẫu và cách kiểm.** Cả ba vòng dùng 2 ảnh có cờ `can_review` và 5 ảnh chọn ngẫu nhiên (seed 42). Vòng đầu, phiếu ghi Lê Trần Phi Hùng là người duyệt, có Codex hỗ trợ. Vòng hai, đối chiếu tọa độ và lớp của nhãn đã sửa với 7 ảnh chụp CVAT đã cung cấp. Một số vật ở xa bị chữ nhãn che. Vòng ba đối chiếu bản export mới với nhãn và phiếu vòng hai. [Mẫu vòng ba](qc/qc_sample.csv)

| Vòng | Lỗi theo đối tượng | Ảnh có lỗi | Loại lỗi ghi nhận |
|---|---|---|---|
| 1, trước sửa | 7/233 (3,0%) | 7/7 (100%) | 4 nhãn thừa và 3 box sai kích thước. Phiếu chọn một lỗi đại diện ở img_034. |
| 2, sau sửa | 4/230 (1,7%) | 3/7 (42,9%) | 4 nhãn thừa, gồm một box ở img_003, một box ở img_024 và hai box cao dưới 20 px ở img_034. |
| 3, sau export mới | 1/226 (0,4%) | 1/7 (14,3%) | Còn box car trên vệt sáng ở img_024, ID #402 trong release vòng ba. |

- **Sửa sau vòng đầu.** Đã xử lý 7 mục của phiếu. Riêng ở img_014, box xe máy đen được khôi phục và thu sát yên xe; ở img_024, box xe trên vệt sáng được xóa. Bản sửa giữ cả box xe trắng mà học viên đã thêm ở ảnh này. Bản vòng hai dùng nhãn sửa cho 7/24 ảnh, tổng cộng 697 nhãn gồm 6 vùng ignore. [QC vòng đầu](qc/round1/qc_report_auto.md)
- **Giới hạn.** Cận trên Wilson 95% theo script là 75% ở vòng hai và 51% ở vòng ba. Bảy ảnh gồm hai ca khó được chọn chủ đích và được dùng lại qua các vòng. [QC vòng hai](qc/round2/qc_report_auto.md) · [QC vòng ba](qc/qc_report_auto.md)
- **Kết quả vòng ba.** Bản release có 693 nhãn trên 24 ảnh. Ba trong bốn box ghi lỗi ở vòng hai đã được xóa đúng. Ở img_024, box bị xóa nằm tại (496,48; 510,33), còn box cần kiểm nằm tại (694,83; 561,75). Cần kiểm lại cả hai vị trí trong CVAT. Script gợi ý **Rework** vì tỷ lệ 1/7 ảnh lỗi vượt ngưỡng Release 2%. [Phiếu vòng ba](qc/qc_review.csv) · [Manifest](release/release_manifest.json)

## 8. Ghi chú cá nhân

- Công cụ hỗ trợ là Codex, dùng để đối chiếu số liệu và soạn bản nháp báo cáo cùng phụ lục calibration. Các số liệu được kiểm tra lại với log và tệp do script sinh. Phần phân tích và các quy tắc trong bản nháp cần học viên tự rà và chốt.
- **Tính độc lập của gold.** Có 4 ảnh gán tay và đã ghi 30 phút cho bước này. Chưa có xác nhận về việc người gán đã xem pre-label hoặc hỏi AI trước khi hoàn tất gold hay chưa. Vì vậy, khi đọc precision và recall, cần tính đến khả năng nhãn gold chịu ảnh hưởng của máy.
- Môi trường chạy là Windows, Python 3.12. Nhật ký r01–r06 ghi thiết bị CPU. Container CVAT trên máy dùng image `cvat/server:v2.81.0`.
- **Nguồn dự đoán.** Nhật ký ghi nhận các lần chạy model trên CPU. Bước quét ngưỡng và gộp nhãn đọc kết quả từ `work/prelabel/y26_all/` và `work/prelabel/yoloe_all/`. Chưa có thông tin để xác nhận việc dùng `data/cache/` ở các bước khác.
- Bộ kiểm tra tiêu đề README ban đầu chỉ nhận MSHV toàn chữ số. Đã sửa để nhận cả mã chữ và số, đồng thời giữ bước đối chiếu với mã đã dùng tạo bẫy.
- Script QC trước đây tính tỷ lệ lỗi trên toàn mẫu dù phiếu còn trống, nên có thể gợi ý Release khi chưa kiểm ảnh nào. Đã sửa để báo tiến độ khi còn ảnh chưa kiểm và tính kết quả khi phiếu đủ.
- **Lần sau**, sau khi sửa trong CVAT, sẽ mở lại đúng frame và kiểm tra hai việc: box cần sửa vẫn còn trên đúng vật và box cần xóa đã biến mất. Bước này giúp phát hiện sớm trường hợp xóa nhầm cả box như ở img_014.
