# Thông tin học viên

- Họ và tên: Lê Trần Phi Hùng
- MSHV: 2A202603353
- Lớp: L2B
- Tài khoản GitHub: hung-1503 (chủ repo origin)
- Repo bài nộp (URL): https://github.com/hung-1503/K5-L23-Day08-LeTranPhiHung-2A202603353-AI-Assisted-Labeling
- Commit chốt bộ nhãn và kết quả QC vòng hai: `74e697857fed88b1023d06cdf881cfdf9461ed10`.
- Máy: Windows, các lần chạy r01–r06 dùng CPU theo work/runs.csv
- CVAT (Docker): v2.81.0, theo image của container `cvat_server` trên máy.

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 621/727 (85,4%), 22/727 (3,0%), 78/727 (10,7%). Added 40/683 (5,9%).
- So với gold, precision của máy đạt 39/74 (52,7%), recall đạt 39/46 (84,8%).
- Bắt bẫy: 10/12 (83,3%)
- Gán tay mất 30 phút cho 4 ảnh (7,50 phút/ảnh). Duyệt có pre-label mất 80 phút cho 24 ảnh (3,33 phút/ảnh), thấp hơn 55,6% theo thời gian trung bình mỗi ảnh. [Nhật ký thời gian](submission/time_log.csv)
- Vòng đầu ghi nhận 7/233 lỗi theo đối tượng và 7/7 ảnh có lỗi. Sau khi sửa, vòng hai ghi nhận 4/230 (1,7%) theo đối tượng và 3/7 (42,9%) theo ảnh. [Báo cáo QC](submission/qc/qc_report_auto.md)
- **Rework** là gợi ý của script cho vòng hai vì 3/7 ảnh còn lỗi, vượt ngưỡng Release 2%. Chờ học viên chốt quyết định.
