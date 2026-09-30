# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng placeholder bằng câu trả lời thật của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Thị Thùy Hiền  Mã học viên: 2A202602896

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu `agent_api_key` mặc định là `"changeme"` mà mình quên set secret trên Railway, app vẫn khởi động bình thường và `/ask` nhận mọi request dùng khóa đó. Bot quét Internet tìm được URL công khai trong vài giờ, gọi API miễn phí → hóa đơn LLM tăng mà mình chỉ phát hiện khi nhìn billing. Không có mặc định thì deploy fail ngay lúc `Settings()` validate, mình còn đang nhìn màn hình dashboard nên sửa được trước khi service nhận traffic.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ log: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-30T11:04:00+00:00", "user_id": "sv01", "tokens_in": 12, "tokens_out": 40, "cost_usd": 0.0001}`. (1) Lọc theo `user_id` để xem ai tiêu nhiều token nhất trong ngày. (2) Đếm tỷ lệ `ask_completed` theo phút và cảnh báo khi `cost_usd` cộng dồn vượt ngưỡng — `print` thuần text không query/aggregation được.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~1000+ MB (python:3.11 đầy đủ, ước lượng) |
| Multi-stage | 298 MB (đo thực tế: docker images agent) |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Chênh lệch chủ yếu là toolchain/compiler và header của base image đầy đủ (`build-essential`, thư viện hệ thống dùng lúc compile wheel) cộng phần không cần ở runtime. Multi-stage chỉ copy kết quả `pip install` sang stage `slim`, bỏ compiler và lớp builder → image nhỏ hơn nhiều, pull/deploy nhanh hơn.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Với Dockerfile hiện tại: layer `COPY requirements.txt` và `RUN pip install` vẫn lấy từ cache vì requirements không đổi; chỉ các layer sau (`COPY app`, `COPY utils`, …) phải chạy lại. Nếu `COPY . .` đứng trước `pip install`, mọi thay đổi code vô hiệu hóa cache từ layer đó trở đi → mỗi lần sửa một dấu phẩy cũng phải cài lại toàn bộ dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Lỗ hổng RCE trong app → attacker chạy lệnh trong container với UID 0 → nếu có lỗ hổng kernel/container escape hoặc volume mount nhạy cảm (ví dụ `/var/run/docker.sock`), họ thao tác host với quyền root. Lệnh `USER appuser` buộc process chạy UID thường nên dù RCE thành công, quyền bên trong container bị hạn chế và khó leo thang lên host hơn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request trong ~2 giây: gửi 10 request lúc 10:00:59 (còn trong phút cũ), rồi ngay lúc 10:01:01 gửi thêm 10 request (phút mới đã reset bộ đếm). Sliding window 60 giây không reset theo đồng hồ nên không có kẽ hở này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn **số lần** gọi trong cửa sổ thời gian; cost guard giới hạn **số tiền** tiêu trong tháng. Rate limit cho qua / cost guard chặn: user chỉ gọi 2 request/phút nhưng mỗi request 50k token → cháy ngân sách. Ngược lại: user spam 30 request ngắn/phút với prompt rẻ → vượt 10 req/phút (429) dù tổng chi phí tháng vẫn thấp.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối → cả 3 container báo unhealthy vì probe phụ thuộc Redis → orchestrator restart cả 3 cùng lúc → trong lúc restart không còn instance phục vụ → khi Redis sống lại cũng không còn container sẵn sàng → sự cố dependency nhỏ biến thành outage toàn cụm. Tách `/health` (không đụng Redis) và `/ready` (có kiểm tra Redis) tránh restart hàng loạt: LB chỉ ngừng đẩy traffic vào instance chưa ready.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis chung, `history_length` tăng dần đều dù request rơi vào container khác nhau. Nếu dùng dict trong RAM mỗi process, số đó sẽ “nhảy” tùy instance: hỏi vào A thì length tăng, lần sau vào B thì về 0 hoặc length của B — agent như mất trí nhớ ngẫu nhiên.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần chạy local bằng compose, agent start rồi tắt ngay với `ValidationError: agent_api_key Field required`. Nguyên nhân: quên truyền `AGENT_API_KEY` vào service trong `docker-compose.yml`. Sửa bằng cách thêm `AGENT_API_KEY: ${AGENT_API_KEY}` (đọc từ `.env`, không hardcode) rồi `docker compose up -d` lại — `/health` trả 200.
