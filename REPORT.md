# REPORT — Lab Day 9

| | |
|---|---|
| Họ và tên | Nguyễn Minh Hiển |
| MSSV (đúng bằng `LAB_STUDENT_ID` trong `.env`) | 2A202602759 |
| GitHub | nmhien165-ui |
| Trip dùng cho bài tập sập và chạy lại | trip-07900 (lấy phần số 202602759 theo quy tắc đã xác nhận, `202602759 % 26 = 21`) |

## Kết quả cuối

- `dataset_version` của bạn: `1d2cfc50ec97` (manifest hash: `8146717fcfb0c416eac307757f1d19cd6703559c82ac3e6256a0394b71d2b92b`)
- Số khung train / val / test: 691 / 155 / 173 (tổng 1.019; 18 / 4 / 4 trip)
- `report/verify.txt`: tạo sau khi chạy `scripts/check_submission.py --save-verify`; cần có 27/27 checks passed.
- Ảnh yêu cầu: `report/screenshots/airflow_graph.png`, `minio_lake.png`, `metabase_dashboard.png` và ảnh task lỗi/clear.

## Bước 1 · Bronze

### Câu 1
Nguồn A dùng `ts_utc` dạng ISO, `speed_mps`, `heading_deg`, `lat/lon`, `alt_m`, `gas` và `steer`; nguồn B dùng `timestamp_ms` dạng số, `spd_kmh`, `hdg`, `latitude/longitude`, `altitude`, `throttle_pct` và `steering`. A và B cũng khác đơn vị tốc độ và quy ước tên cột. Bronze giữ riêng từng nguồn và để giá trị dạng chữ để bảo toàn dữ liệu đến, tránh ép kiểu theo giả định của một nguồn làm mất dấu schema drift; chuẩn hoá có kiểm soát ở Silver để lỗi vẫn truy được về dòng gốc.

## Bước 2 · Silver telemetry

### Câu 2
Silver đổi tốc độ B từ km/h sang m/s, đổi timestamp sang UTC thống nhất và trừ độ lệch đồng hồ B 1.500 ms. Sau chuẩn hoá, mẫu đầu B ở `2025-10-08 14:30:00.009 UTC`, lệch dưới 0,5 giây so với video; tốc độ cực đại A/B là 71/100 km/h. Mỗi drive nhận 3.021/3.234 dòng; mỗi drive cách ly 20 duplicate, 30 thiếu speed và 3 speed ngoài miền. 2.968 dòng A và 3.181 dòng B còn sạch. Rejects giữ lại bằng chứng và lý do để kiểm tra nguồn, thay vì âm thầm xoá mất dữ liệu.

### Câu 3
`max_gap_ms` là `warn` vì khoảng trống tín hiệu cần được nhìn thấy nhưng không nhất thiết làm dữ liệu khác mất hợp lệ; `in_range` là `error` vì tốc độ ngoài miền có thể làm sai phân tích. Trong `break_gate.json`, nới ngưỡng lên 1.000 m/s làm test `in_range_silver_telemetry_speed_mps__70__0` bắt 6 dòng; kết quả có 1 ERROR. Khôi phục ngưỡng cho PASS=6, WARN=1, ERROR=0. Cổng `error` chặn dữ liệu sai trước khi downstream dùng nó.

## Bước 3 · Silver frames

### Câu 4
`idempotency.json` ghi 1.246 dòng và fingerprint `6c7480dfde7e` trước lẫn sau hai lần chạy lại: số dòng và fingerprint không đổi. Cách `append` sai tạo 120 dòng cho trip có 60 frame, tức nhân đôi 2 lần. Ghi đè có điều kiện trên đúng partition trip thay thế nguyên partition trong một thao tác Delta, tránh cửa sổ dữ liệu bị xoá nhưng chưa ghi lại và không đụng partition khác.

### Câu 5
Frame đen được nhận diện là gần như không có thông tin hình ảnh và bị loại; frame tối do thiếu sáng được gắn cờ riêng nhưng vẫn giữ nếu không vi phạm luật khác. Ở ngưỡng dHash 0/2/6, A có 61/149/363 frame bị coi là trùng, B có 8/53/244. Mình giữ ngưỡng 2 đang dùng: ngưỡng 0 bỏ sót nhiều frame gần trùng, còn 6 có nguy cơ gộp cả cảnh khác nhau. Mức hợp lý còn phụ thuộc tốc độ xe và chuyển động trong ảnh: xe chậm tạo nhiều frame tương tự hơn.

| dHash khác tối đa | Drive A | Drive B |
|---:|---:|---:|
| 0 | 61 | 8 |
| 2 | 149 | 53 |
| 6 | 363 | 244 |

## Bước 4 · Gold

### Câu 6
Chia ngẫu nhiên theo frame làm cả 26/26 trip xuất hiện ở nhiều split và cả 26 có frame ở cả train lẫn test. Chia theo trip giữ 0 trip bị cắt. Các frame liền nhau gần như trùng cảnh; nếu cảnh đã xuất hiện ở train lại lọt sang test, mô hình có thể nhận ra cảnh thay vì khái quát hoá, làm điểm test cao giả tạo.

### Câu 7
`dataset_version` ổn định hơn ngày giờ hoặc số lượng vì được tính từ fingerprint SHA-256 rút gọn của các cặp `frame_id|split` đã sắp thứ tự; cùng mapping cho cùng version. Lần chạy lại với MSSV này giữ version `1d2cfc50ec97`, dù khởi tạo lại lake. MSSV tham gia phân công trip vào split, nên mapping và version thay đổi giữa người nộp; manifest còn lưu hash video nguồn để truy nguồn dữ liệu.

### Câu 8
Manifest ghi 1.019 frame, split 691/155/173, luật loại `is_black`, `is_blurry`, `is_near_dup`, `not sync_ok`, và chia theo trip, phân tầng theo drive, thứ tự MD5. Lineage ghi version Bronze telemetry A/B, trips và video (0/0/0/1), Silver frames/telemetry/frames_flagged/frame_telemetry (25/0/0/0), cùng SHA-256 riêng của video A và B. `dataset_version`, `manifest_sha256`, checksum đầu vào, lineage và quy tắc split giúp đối chiếu rồi tái tạo đúng dataset.

## Bước 5 · Điều phối và phục vụ

### Câu 9
Airflow tạo 26 mapped task `silver_frames`. Trip `trip-07900` (map index 21) fail đúng một lần; log ghi `simulated crash while extracting trip-07900`. Có 7 task downstream chuyển `upstream_failed`. Clear riêng mapped task đỏ (hộp thoại báo 1 task) làm trip thành công ở lần thử 2; trên Airflow này, bảy task downstream vẫn giữ `upstream_failed`, nên cần clear riêng đúng bảy task bị chặn trong cùng run để chúng tiếp tục. Cả DAG sau đó xanh. Partition retry giữ dữ liệu không nhân đôi; `dataset_version` vẫn `1d2cfc50ec97`, và verify vẫn đạt 27/27.

### Câu 10
Ở 500 xe/ngày, điểm nghẽn đầu tiên là trích xuất FFmpeg và mapped tasks tuần tự (mỗi DAG run giới hạn một task frame hoạt động): cần worker pool và xử lý song song có giới hạn tài nguyên. Tiếp theo là lake/object I/O và nhiều partition nhỏ; cần object storage bền, tối ưu partition/file size và throughput. Cuối cùng, DuckDB đơn máy cùng warehouse Postgres có giới hạn compute/concurrency; chuyển biến đổi lớn sang engine phân tán và warehouse mở rộng. Đây là các hướng scale compute, ELT và xử lý video cần đối chiếu với slide 12, 22, 23.

## Câu hỏi riêng trên Metabase

Đã thêm vào dashboard câu hỏi **Trips with the lowest usable frame share**:

```sql
select drive_id, trip_id, split, frames, frames_kept,
       round(100.0 * frames_kept / nullif(frames, 0), 1) as kept_pct
from marts.mart_trip_summary
where frames > 0
order by kept_pct asc, frames desc
limit 10
```

Câu hỏi xếp các trip có tỷ lệ frame dùng được thấp nhất. Trip đầu là `trip-04622` ở split train: 26/60 frame, 43,3%; kế tiếp là `trip-03904` ở train với 11/22, 50,0%. Đây là câu hỏi vận hành giúp ưu tiên xem lại cảnh có tỷ lệ loại frame cao.

## Nhìn lại

Điều dễ bỏ sót là frame tối không đồng nghĩa frame đen: drive B có 61 frame tối được giữ nhưng chỉ 4 frame đen bị loại. Khi ngưỡng dHash tăng từ 2 lên 6, số frame bị coi là trùng tăng từ 149 lên 363 ở A và 53 lên 244 ở B; cần kiểm chứng ngưỡng trên các tốc độ/điều kiện cảnh khác nhau. Một lỗi pipeline có thể chưa bắt là cảm biến có bias ổn định nhưng vẫn nằm trong miền 0–70 m/s: các test range và duplicate có thể vẫn qua dù giá trị có sai lệch hệ thống.
