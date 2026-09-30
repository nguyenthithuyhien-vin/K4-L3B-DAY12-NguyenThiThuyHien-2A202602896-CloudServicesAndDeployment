# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Thị Thùy Hiền |
| Mã học viên | 2A202602896 |
| Repo | https://github.com/nguyenthithuyhien/K4-L3B-DAY12-NguyenThiThuyHien-2A202602896-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | http://localhost:8000 (LOCAL_FALLBACK — chạy docker compose tại máy) |
| Platform | Docker Compose (phương án dự phòng LOCAL_FALLBACK; chưa đăng ký Railway/Render trong buổi lab) |
| Ngày deploy | 2026-09-30 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán / compose set 8000 |
| `AGENT_API_KEY` | ✅ | đặt trong file .env cục bộ, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis service trong docker compose (`redis://redis:6379/0`) |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i http://localhost:8000/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i http://localhost:8000/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST http://localhost:8000/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
=== /health ===
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    57  100    57    0     0  14415      0 --:--:-- --:--:-- --:--:-- 19000
HTTP/1.1 200 OK
date: Wed, 30 Sep 2026 11:07:10 GMT
server: uvicorn
content-length: 57
content-type: application/json

{"status":"ok","service":"day12-agent","version":"1.0.0"}
=== /ready ===
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    31  100    31    0     0   6805      0 --:--:-- --:--:-- --:--:--  7750
HTTP/1.1 200 OK
date: Wed, 30 Sep 2026 11:07:10 GMT
server: uvicorn
content-length: 31
content-type: application/json

{"status":"ready","redis":true}
=== /ask no key ===
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    59  100    39  100    20  10664   5468 --:--:-- --:--:-- --:--:-- 19666
HTTP/1.1 401 Unauthorized
date: Wed, 30 Sep 2026 11:07:10 GMT
server: uvicorn
content-length: 39
content-type: application/json

{"detail":"invalid or missing API key"}
=== /ask with key ===
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   367  100   337  100    30  31991   2847 --:--:-- --:--:-- --:--:-- 36700
HTTP/1.1 200 OK
date: Wed, 30 Sep 2026 11:07:10 GMT
server: uvicorn
content-length: 337
content-type: application/json

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud. (Mình đang nhớ 2 lượt trao đổi trước đó.)","user_id":"sv-test","history_length":2,"cost_usd":3.315e-05,"tokens":{"in":41,"out":45}}
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
Chưa kịp đăng ký / cấu hình tài khoản Railway hoặc Render trong thời gian lab.
Đã chạy đầy đủ stack bằng docker compose (agent + redis) trên máy local,
bật LOCAL_FALLBACK=true, và nộp screenshot minh chứng /health + docker compose ps.
```
