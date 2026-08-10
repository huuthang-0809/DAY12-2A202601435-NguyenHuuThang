# Phiếu Phản Ánh — K3 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng trả lời mẫu bên dưới bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Hữu Thắng  Mã học viên: 2A202601435

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ khi deploy lên Railway, nếu quên khai báo `AGENT_API_KEY` thì tiến trình dừng ngay lúc khởi động và log chỉ rõ biến nào còn thiếu. Nhờ vậy bản cấu hình sai không vượt qua health check để nhận request thật. Nếu code dùng mặc định `"changeme"`, service vẫn có vẻ hoạt động nhưng mọi người đều có thể đoán được khóa này và gọi API trái phép; lỗi chỉ lộ ra sau khi production đã bị truy cập.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log thu được có dạng: `{"event":"ask_completed","level":"info","timestamp":"2026-08-10T05:40:12+00:00","user_id":"sv-test","tokens_in":3,"tokens_out":37,"cost_usd":0.00002265}`
> Từ dòng JSON này có thể lọc và đếm số sự kiện `ask_completed` theo `user_id` để theo dõi mức sử dụng của từng người. Tôi cũng có thể cộng `cost_usd`, `tokens_in` và `tokens_out` trên hệ thống thu thập log để làm cảnh báo chi phí. Chuỗi `print("đã trả lời xong")` không có các trường có cấu trúc nên không làm đáng tin cậy được hai việc đó.

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
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi build lại từ đúng Dockerfile one-stage ban đầu và đo bằng `docker images`. Phần chênh lệch chủ yếu đến từ image nền `python:3.11` đầy đủ, các công cụ và dữ liệu chỉ cần khi cài package, cùng cache/trung gian của quá trình cài đặt. Bản multi-stage dùng `python:3.11-slim`; stage runtime chỉ nhận các package đã cài và source cần chạy, không mang toàn bộ môi trường builder sang production.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, các layer lấy base image, đặt `WORKDIR`, copy `requirements.txt` và chạy `pip install` vẫn được dùng từ cache. Layer `COPY app ./app` và các layer đứng sau nó phải tạo lại. Nếu đặt `COPY . .` trước `RUN pip install`, chỉ một thay đổi trong source cũng làm layer copy đổi, kéo theo việc cài lại toàn bộ dependency dù `requirements.txt` không đổi; build sẽ chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Giả sử endpoint Python có lỗi cho phép thực thi lệnh, kẻ tấn công có thể chạy lệnh bên trong container với đúng quyền của tiến trình ứng dụng. Nếu tiến trình là root, họ có quyền root trong container, khi kết hợp với một lỗi thoát container, mount nhạy cảm hoặc quyền Docker cấp quá rộng, họ có thể sửa file hay chiếm quyền cao trên host. `USER appuser` cắt chuỗi ngay sau bước khai thác ứng dụng: lệnh của kẻ tấn công chỉ chạy bằng UID 10001 ít quyền, nên phạm vi gây hại và khả năng tận dụng bước tiếp theo bị giảm đáng kể.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 request trong hai giây liên tiếp: gửi 10 request ở cuối một phút, ví dụ `12:00:59.x`, rồi gửi thêm 10 request ngay sau khi bộ đếm reset ở `12:01:00.x`. Fixed window coi đó là hai phút khác nhau dù hai nhóm chỉ cách nhau khoảng một giây. Sliding window nhìn lại đúng 60 giây nên nhóm thứ hai vẫn thấy 10 request trước đó và bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tốc độ request trong cửa sổ 60 giây, còn cost guard giới hạn tổng tiền đã dùng trong cả tháng. Một người gửi chậm, chẳng hạn mỗi phút một request nhưng request nào cũng tốn nhiều token, vẫn qua rate limit nhưng cuối cùng bị cost guard chặn khi vượt ngân sách. Ngược lại, một người chưa tốn đáng kể ngân sách tháng nhưng gửi 11 request nhỏ liên tiếp sẽ bị rate limit chặn ở request thứ 11 trong khi cost guard vẫn còn đủ tiền để cho qua.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu `/health` cũng kiểm tra Redis thì khi Redis mất kết nối, cả ba container đều trả health check lỗi. Orchestrator đánh dấu cả ba process là không khỏe và lần lượt restart chúng. Các process mới vẫn không kết nối được Redis nên tiếp tục fail health check và rơi vào vòng lặp restart, làm mất cả endpoint không phụ thuộc Redis và khiến sự cố Redis lan thành sự cố toàn service. Với thiết kế hiện tại, `/ready` trả 503 để ngừng nhận traffic nhưng `/health` vẫn trả 200, nên container được giữ nguyên và có thể sẵn sàng lại ngay khi Redis phục hồi.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis dùng chung, dù request được phân phối qua ba replica, tôi vẫn thấy `history_length` tăng nhất quán theo lịch sử chung của cùng `X-User-Id` (mỗi lượt hỏi hoàn tất lưu thêm một message user và một message assistant). Nếu lưu bằng dict Python, mỗi replica có một bản riêng: request rơi vào replica khác sẽ thấy lịch sử bằng 0 hoặc một con số thấp hơn, nên kết quả có thể nhảy như `0, 0, 2, 0, 2, 4` thay vì tăng đều. Khi container restart, phần lịch sử trong dict của container đó còn mất hoàn toàn.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi thật tôi gặp trên Railway là `Invalid value for '--port': '$PORT' is not a valid integer`, sau đó health check `/health` hết thời gian và deploy thất bại. Tôi mở deploy log và thấy Uvicorn đang nhận nguyên chuỗi `$PORT`, chứng tỏ start command được chạy theo dạng không thực hiện shell expansion. Tôi sửa `railway.toml` thành `sh -c 'uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8000}'`, deploy lại, rồi log báo Uvicorn chạy tại `0.0.0.0:8080` và health check thành công.
