# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn quan sát được khi chạy code.

> Họ và tên: **Nguyễn Bá Chính**  
> Mã học viên: **2A202602654**

---

### Câu 1 — Fail fast (CP1)

Một tình huống cụ thể là khi deploy service lên cloud nhưng tôi quên cấu hình biến môi trường `AGENT_API_KEY`. Nếu chương trình có giá trị mặc định như `"changeme"` thì service vẫn khởi động bình thường và tôi có thể tưởng rằng bản deploy đã đúng. Trong khi đó, người khác có thể đoán hoặc biết khóa mặc định rồi gọi API.

Khi `agent_api_key` không có giá trị mặc định, chương trình báo lỗi ngay lúc khởi động. Nhờ đó tôi biết cấu hình production đang thiếu secret trước khi service nhận request thật. Tôi hiểu đây là ý nghĩa của **fail fast**: lỗi cấu hình nên xuất hiện càng sớm càng tốt thay vì để hệ thống chạy trong trạng thái không an toàn.

---

### Câu 2 — Log cho máy đọc (CP1)

Dòng log JSON tôi thu được:

`{"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T09:09:11.348883+00:00", "user_id": "sv01", "tokens_in": 1, "tokens_out": 35, "cost_usd": 2.115e-05}`

Từ dòng structured log này, tôi có thể lọc các request theo những trường cụ thể như `user_id`, `event` hoặc `level`. Ví dụ tôi có thể tìm toàn bộ request của user `sv01` hoặc đếm số lần sự kiện `ask_completed` xảy ra.

Ngoài ra, tôi có thể dùng các trường `tokens_in`, `tokens_out` và `cost_usd` để thống kê lượng token đã dùng và tổng chi phí. Nếu chỉ dùng `print("đã trả lời xong")` thì log không có cấu trúc và gần như không có đủ dữ liệu để lọc, tổng hợp hoặc giám sát tự động.

---

### Câu 3 — Kích thước image (CP2)

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | **[ĐO TRÊN MÁY CÓ DOCKER] MB** |
| Multi-stage | **[ĐO TRÊN MÁY CÓ DOCKER] MB** |

Phần dung lượng chênh lệch chủ yếu đến từ việc multi-stage build chỉ mang những thành phần cần thiết sang stage runtime. Stage builder có thể chứa các công cụ cài đặt hoặc build dependency, nhưng những thành phần đó không cần tồn tại trong image cuối.

Ngoài ra tôi sử dụng `python:3.11-slim` thay vì image Python đầy đủ, nên runtime image chứa ít package hệ điều hành hơn. Kết quả là image nhỏ hơn, tải và deploy nhanh hơn, đồng thời giảm bề mặt tấn công.

**Sau khi sang máy Docker tôi sẽ điền số MB thực tế từ `docker images`.**

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Dockerfile của tôi copy `requirements.txt` và cài dependency trước khi copy source code:

```dockerfile
COPY requirements.txt .
RUN pip install ...
COPY app ./app
COPY utils ./utils
```

Với cách sắp xếp này, khi chỉ sửa một ký tự trong `app/main.py`, các layer trước đó như base image, `COPY requirements.txt` và `pip install` không thay đổi nên Docker có thể lấy chúng từ cache. Chỉ các layer bắt đầu từ bước copy source code và các layer sau nó phải được tạo lại.

Nếu đặt `COPY . .` trước `RUN pip install`, chỉ một thay đổi nhỏ trong source code cũng làm layer `COPY` thay đổi. Docker sẽ mất cache cho toàn bộ các layer phía sau, bao gồm `pip install`, nên phải cài lại toàn bộ dependency dù `requirements.txt` không đổi.

**Tôi sẽ xác nhận lại bằng output build thực tế trên máy có Docker và sửa câu này nếu quan sát khác với dự kiến.**

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Giả sử code Python của service có một lỗ hổng cho phép attacker thực thi lệnh. Khi container chạy bằng root, đoạn code mà attacker điều khiển cũng chạy với quyền root bên trong container. Nếu hệ thống đồng thời có một lỗ hổng container escape, mount nhạy cảm hoặc cấu hình Docker không an toàn, quyền cao đó có thể làm hậu quả lan sang host.

Trong Dockerfile tôi tạo một user thường và dùng:

```dockerfile
USER appuser
```

Nhờ đó process FastAPI không chạy bằng root nữa. Nếu ứng dụng bị khai thác, attacker trước tiên chỉ có quyền của `appuser`, nên khả năng thay đổi các tài nguyên đặc quyền bị hạn chế. `USER` không tự loại bỏ mọi lỗ hổng, nhưng nó giảm quyền của process và giảm mức độ thiệt hại nếu ứng dụng bị compromise.

---

### Câu 6 — Cửa sổ trượt (CP3)

Nếu giới hạn là 10 request/phút nhưng sử dụng cách đếm theo phút đồng hồ, user có thể gửi **20 request trong khoảng 2 giây**.

Ví dụ:

- gửi 10 request vào khoảng `10:00:59`;
- đồng hồ chuyển sang `10:01:00`, counter của phút mới được reset;
- ngay sau đó gửi tiếp 10 request.

Mỗi phút riêng biệt vẫn chỉ có 10 request nhưng thực tế server vừa nhận khoảng 20 request gần như liên tiếp.

Sliding window 60 giây tránh được vấn đề này vì tại mỗi thời điểm nó đếm số request trong **60 giây gần nhất**, chứ không phụ thuộc vào ranh giới của phút trên đồng hồ.

---

### Câu 7 — Rate limit và cost guard (CP3)

Rate limit và cost guard giải quyết hai vấn đề khác nhau.

**Rate limit** giới hạn tốc độ hoặc số lượng request trong một khoảng thời gian. Trong bài của tôi nó kiểm tra số request trong cửa sổ 60 giây và trả `429 Too Many Requests` khi vượt giới hạn.

**Cost guard** giới hạn tổng số tiền user được phép tiêu trong tháng. Khi vượt ngân sách nó trả `402 Payment Required`.

Trường hợp rate limit cho qua nhưng cost guard chặn: một user chỉ gọi 1 request mỗi phút nên không vượt rate limit, nhưng trước đó đã sử dụng gần hết ngân sách tháng. Request mới không quá nhanh nhưng vẫn phải bị cost guard chặn.

Trường hợp ngược lại: một user mới bắt đầu sử dụng nên gần như chưa tốn tiền, nhưng gửi rất nhiều request trong vài giây. Cost guard vẫn còn ngân sách nhưng rate limiter phải chặn vì tốc độ request quá cao.

---

### Câu 8 — `/health` khác `/ready` (CP4)

Nếu tôi gộp `/health` và `/ready` thành một endpoint rồi cho endpoint đó kiểm tra Redis, khi Redis mất kết nối khoảng 30 giây có thể xảy ra chuỗi sự kiện sau:

1. Redis mất kết nối.
2. Cả 3 container đều kiểm tra Redis và trả trạng thái unhealthy.
3. Orchestrator hiểu rằng cả 3 container có vấn đề về liveness.
4. Nó có thể restart cả 3 container.
5. Service mất các instance đang chạy mặc dù bản thân process FastAPI không hề chết.
6. Redis có thể vẫn chưa phục hồi, nên các container mới khởi động lại vẫn gặp cùng lỗi.

Một lỗi dependency tạm thời vì thế có thể biến thành sự cố của toàn service.

Thiết kế tách hai endpoint tránh vấn đề này. `/health` chỉ kiểm tra process có còn sống hay không nên Redis chết thì nó vẫn có thể trả 200. `/ready` kiểm tra Redis và trả 503, nhờ đó load balancer tạm ngừng gửi traffic vào instance nhưng orchestrator không cần restart process một cách không cần thiết.

---

### Câu 9 — Stateless (CP4)

**Kết quả thực tế cần bổ sung sau khi chạy:**

```text
history_length: [DÁN CÁC GIÁ TRỊ QUAN SÁT ĐƯỢC]
```

Với thiết kế hiện tại, lịch sử hội thoại được lưu trong Redis. Nhiều instance của `ConversationStore` cùng sử dụng Redis nên dù các request đi vào các container khác nhau, chúng vẫn đọc được cùng một lịch sử của `X-User-Id`.

Nếu thay Redis bằng một `dict` Python trong từng container thì mỗi container sẽ có bộ nhớ riêng. Ví dụ request đầu vào container A làm history của A tăng, request tiếp theo vào container B lại không nhìn thấy dữ liệu của A. Khi load balancer phân phối request qua nhiều instance, `history_length` có thể tăng không đều, quay lại giá trị nhỏ hoặc trông như agent bị mất trí nhớ.

Redis đưa state ra khỏi từng process nên các instance có thể scale ngang mà vẫn chia sẻ cùng dữ liệu.

**Tôi sẽ bổ sung chuỗi `history_length` thực tế sau khi chạy `docker compose up --scale agent=3`.**

---

### Câu 10 — Deploy thật (CP5)

**Chưa điền trước khi deploy để không bịa lỗi hoặc output.**

Sau khi deploy tôi sẽ ghi lại một lỗi thực tế theo ba phần:

- **Thông báo lỗi:** `[DÁN LỖI THỰC TẾ]`
- **Cách tìm nguyên nhân:** `[LOG / HEALTH CHECK / CONFIG MÀ TÔI ĐÃ KIỂM TRA]`
- **Cách sửa:** `[THAY ĐỔI THỰC TẾ TÔI ĐÃ LÀM]`

Ví dụ các loại lỗi tôi sẽ kiểm tra nếu gặp là build Docker thất bại, sai `REDIS_URL`, readiness trả 503, thiếu biến môi trường hoặc service không sử dụng `$PORT` do platform cấp.