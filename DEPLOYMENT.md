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
| `PORT` | Có | Service chạy port 8000 |
| `AGENT_API_KEY` | Có | Lưu trong `.env`, không commit |
| `REDIS_URL` | Có | Redis trong Docker Compose |
| `RATE_LIMIT_PER_MINUTE` | Có | Cấu hình qua biến môi trường |
| `MONTHLY_BUDGET_USD` | Có | Cấu hình qua biến môi trường |
| `LOG_LEVEL` | Có | Cấu hình log |
| `LOCAL_FALLBACK` | Có | true |

Không ghi giá trị thật của `AGENT_API_KEY` trong tài liệu.

## Kết Quả Kiểm Tra

Docker Compose chạy thành công hai service:

- agent
- redis

Health endpoint:

```text
HTTP/1.1 200 OK
{"status":"ok","service":"day12-agent","version":"1.0.0"}