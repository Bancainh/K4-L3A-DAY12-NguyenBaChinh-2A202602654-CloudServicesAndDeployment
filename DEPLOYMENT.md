# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Bá Chính |
| Mã học viên | 2A202602654 |
| Repo | https://github.com/Bancainh/K4-L3A-DAY12-NguyenBaChinh-2A202602654-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| URL kiểm tra | http://localhost:8000 |
| Platform | Railway (platform dự kiến) - hiện sử dụng Local Docker Compose với LOCAL_FALLBACK |
| Ngày kiểm tra | 28/09/2026 |

## Biến Môi Trường

| Biến | Đã cấu hình | Ghi chú |
|------|-------------|---------|
| `PORT` | Dùng mặc định | Compose hiện không truyền biến này; service dùng cổng 8000 |
| `AGENT_API_KEY` | Có | Lưu trong `.env`, không commit |
| `REDIS_URL` | Có | Redis trong Docker Compose |
| `RATE_LIMIT_PER_MINUTE` | Có | Cấu hình qua biến môi trường |
| `MONTHLY_BUDGET_USD` | Có | Cấu hình qua biến môi trường |
| `LOG_LEVEL` | Có | Cấu hình log |
| `LOCAL_FALLBACK` | Có | true |

Không ghi giá trị thật của `AGENT_API_KEY` trong tài liệu.

## Kết Quả Kiểm Tra

Kết quả được ghi nhận từ lần thực hành trên máy có Docker trước đó, chưa
chạy lại trên máy hiện tại:

Docker Compose chạy thành công hai service:

- agent
- redis

Health endpoint:

```text
HTTP/1.1 200 OK
{"status":"ok","service":"day12-agent","version":"1.0.0"}
```

Ảnh đã lưu: [local-fallback.png](screenshots/local-fallback.png).

## Phần Chưa Kiểm Chứng

- Máy hiện tại không có Docker, chưa build/chạy lại image sau khi sửa
  healthcheck theo `PORT` và thêm `exec` vào lệnh khởi động.
- Chưa bổ sung kết quả thực tế của `/ready` và `/ask` không có API key.
- Chưa chạy 3 instance; Compose hiện map cố định `8000:8000`, cần điều chỉnh
  cách công bố cổng hoặc dùng reverse proxy trước khi scale.
- Chưa deploy Railway/Render, chưa có Public URL HTTPS hoặc ảnh dashboard cloud.
- Theo rubric, phương án `LOCAL_FALLBACK=true` có trần CP5 là 9/15 điểm.

## Kiểm Tra Trên Máy Không Có Docker

Chạy từ thư mục gốc repo sau khi kích hoạt môi trường ảo và cài dependency:

```powershell
python -m pytest tests/test_cp1.py tests/test_cp2.py tests/test_cp3.py tests/test_cp4.py -v -m "not docker"
```

Lệnh này kiểm tra logic Python và cấu trúc cấu hình Docker; không chứng minh
image build được, Redis thật hoạt động hoặc cloud deployment đã hoàn tất.

## Kiểm Tra Khi Có Máy Chạy Docker

Đặt `LOCAL_FALLBACK=true` trong `.env` cục bộ, giữ API key ngoài Git, rồi chạy:

```powershell
docker compose up -d --build
docker compose ps
curl.exe -i http://localhost:8000/health
curl.exe -i http://localhost:8000/ready
curl.exe -i -X POST http://localhost:8000/ask -H "Content-Type: application/json" --data-raw '{}'
python -m pytest tests/ -v
python grade.py --no-bonus
```

Kết quả mong đợi (chưa phải kết quả đã đo lại): `/health` và `/ready` trả 200;
`/ask` không có key trả 401. Chụp kết quả thực tế, che secret nếu có và bổ sung
vào tài liệu trước khi nộp.
