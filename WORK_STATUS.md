# Trạng thái công việc

## Mục tiêu hiện tại
- Bài Day 9 với MSSV `2A202602759` đã được đẩy lên repo GitHub Public; bàn giao đường dẫn nộp.

## Việc đã hoàn thành
- Docker Desktop, Airflow, MinIO, Postgres và Metabase chạy cục bộ. `.env` chứa `LAB_STUDENT_ID=2A202602759`; cờ `LAB_FAIL_ONCE` đã gỡ.
- Từ phần số `202602759`, phép chia dư 26 chọn map index 21, trip `trip-07900`.
- Bốn thí nghiệm idempotency, break-gate, leakage, threshold đã chạy lại; năm JSON bằng chứng đều ghi MSSV mới.
- DAG `day09_reset` thành công; run crash-once `manual__2026-10-12T16:17:39+00:00` fail đúng `silver_frames[21]`, sau đó task này thành công ở try 2. Bảy downstream bị `upstream_failed` được clear đúng trong cùng run; DAG kết thúc `success`.
- Đã chụp lại ảnh Graph lỗi/xanh, mapped task lỗi/phục hồi, log crash, dashboard và câu hỏi riêng Metabase. Ảnh MinIO bucket `lake/` vẫn chứng minh cấu trúc bronze/silver/gold.
- Lượt `step all --reset` cuối sau khi gỡ cờ fail-once đạt 27/27; `report/evidence/run_summary.json` ghi `dataset_version=1d2cfc50ec97`, 1.019 frame chia 691 train/155 val/173 test. `report/verify.txt` ghi verify thật 27/27.
- `REPORT.md` đã điền đủ 10 câu, MSSV đúng, trip crash đúng và số liệu Metabase mới. ZIP dữ liệu course không được đưa lên Git nhưng giữ bản cục bộ.

## Việc đang làm
- Không còn việc kỹ thuật đang làm. Người nộp cần gửi đường dẫn repo lên hệ thống LMS.

## File đã sửa/tạo
- `REPORT.md`, `WORK_STATUS.md`, `report/verify.txt`.
- `report/evidence/run_summary.json`, `idempotency.json`, `break_gate.json`, `leakage.json`, `threshold.json`, `crash_once.log`.
- Tám ảnh trong `report/screenshots/`: năm ảnh Airflow, hai ảnh Metabase, một ảnh MinIO.
- `.env` chỉ sửa cục bộ và được Git ignore; ZIP dữ liệu vẫn còn trên đĩa nhưng bị loại khỏi Git.

## Quyết định quan trọng
- Giữ nguyên mã, tests, Compose và tài liệu scaffold. Ba file được checker bảo vệ giữ đúng byte Git HEAD; `core.autocrlf=false` chỉ áp dụng ở repo cục bộ.
- Phần số của MSSV chữ-số dùng cho công thức trip theo quy tắc người dùng đã xác nhận.
- Ảnh Airflow/Metabase chụp từ service thật bằng Edge headless khi browser helper của Codex không khởi tạo được; không chỉnh sửa nội dung ảnh.
- Remote `main` được giữ nguyên trong lúc người dùng đính chính MSSV; chỉ bản đúng được đẩy lên.
- Người dùng đã xác nhận công khai họ tên và MSSV `2A202602759` trên repo Public. Các commit local chưa từng đẩy đã được dựng lại để không đưa mã nhập nhầm vào lịch sử remote.
- Checker nộp bài đạt 17/17; đã push `main` và đối chiếu SHA remote/local trùng nhau ở thời điểm kiểm tra. GitHub API xác nhận repo Public.

## Lỗi/vấn đề còn tồn tại
- Không có lỗi kỹ thuật còn tồn tại. Người nộp cần nộp link repo Public theo GUIDE.md.

## Bước tiếp theo cần làm
- Nộp `https://github.com/nmhien165-ui/K4-L3L4-Track4-DataPipeline4ComputerVision` trên LMS.
