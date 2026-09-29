# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

Họ và tên: Nguyễn Xuân Trường  Mã học viên: 2A202602761

---

### Câu 1 — Fail fast (CP1)

Giả sử tôi deploy lên Railway nhưng quên set biến `AGENT_API_KEY`. Nếu code
để mặc định `"changeme"`, app vẫn khởi động bình thường, `/health` xanh, tôi
tưởng mọi thứ ổn. Trong khi đó bất kỳ ai đoán được `"changeme"` đều gọi
`/ask` được và tiêu tiền LLM của tôi, đến khi nhìn hóa đơn mới biết. Còn khi
không có mặc định, `Settings()` ném `ValidationError` ngay lúc khởi động,
deploy fail và log báo rõ thiếu `agent_api_key`, nên tôi sửa được ngay khi còn
đang nhìn màn hình.

---

### Câu 2 — Log cho máy đọc (CP1)

Dòng log thu được:

```
<dán một dòng ask_completed thật từ docker compose logs agent>
```

Hai việc làm được với log JSON mà `print("đã trả lời xong")` không làm được:
1. Lọc và tổng hợp theo trường, ví dụ cộng `cost_usd` theo `user_id` để biết
   user nào tiêu nhiều nhất hôm nay.
2. Đặt cảnh báo/đếm tỷ lệ lỗi theo `level` hoặc `event` (ví dụ đếm số event
   lỗi trong 5 phút). Máy chỉ làm được vì mỗi dòng là một JSON parse được.

---

### Câu 3 — Kích thước image (CP2)

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 850 MB |
| Multi-stage | 271 MB |

Phần chênh lệch (~580 MB) chủ yếu đến từ base image và công cụ build. Bản đầu
dùng `python:3.11` đầy đủ (kèm trình biên dịch, header, nhiều gói hệ thống) và
`COPY . .` mang cả những thứ thừa. Bản multi-stage dùng `python:3.11-slim`, cài
dependency ở stage `builder`, rồi chỉ `COPY --from=builder /install` sang
stage runtime, nên không mang theo compiler hay cache của quá trình cài.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Khi sửa một ký tự trong `app/main.py` và build lại, các layer đứng trước
`COPY app ./app` (stage builder, `useradd`, `WORKDIR`, `pip install`, copy
`/install`) đều hiện `CACHED`. Layer `COPY app ./app` phải chạy lại vì nội
dung thay đổi, và layer `COPY utils ./utils` đứng sau nó cũng chạy lại vì
Docker huỷ cache từ layer đầu tiên thay đổi trở đi. Nếu đặt `COPY . .` lên
trước `RUN pip install`, mỗi lần sửa code sẽ huỷ cache ngay ở bước đó và
`pip install` chạy lại toàn bộ, làm build chậm hơn nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Chuỗi sự kiện: (1) code Python có lỗ hổng, ví dụ cho phép chạy lệnh tùy ý;
(2) kẻ tấn công thực thi lệnh trong container; (3) nếu container chạy root thì
lệnh đó chạy với quyền root trong container, có thể sửa mọi file, cài công cụ
và tận dụng lỗi cấu hình hoặc lỗ hổng runtime để thoát ra; (4) khi thoát ra
host, uid 0 trong container có thể trở thành quyền cao trên host. Lệnh `USER
appuser` (uid 10001) cắt đứt chuỗi ở bước 3: dù kẻ tấn công chạy được lệnh,
họ chỉ có quyền của một user thường, không sửa được file hệ thống, và nếu có
thoát ra cũng chỉ mang quyền thấp.

---

### Câu 6 — Cửa sổ trượt (CP3)

Với cách đếm theo phút đồng hồ và hạn mức 10/phút, người dùng có thể gửi tối
đa **20 request trong 2 giây**: 10 request lúc 10:00:59 (thuộc phút 10:00) và
10 request lúc 10:01:01 (thuộc phút 10:01, bộ đếm vừa reset). Cả hai đợt đều
"đúng luật" theo từng phút riêng. Sliding window không có kẽ hở này vì luôn
đếm 60 giây gần nhất tính từ thời điểm hiện tại, nên đợt thứ hai vẫn bị tính
cùng với đợt đầu.

---

### Câu 7 — Rate limit và cost guard (CP3)

Rate limit giới hạn **số lượng** request trong một khoảng thời gian; cost
guard giới hạn **số tiền** tiêu trong tháng. Rate limit cho qua nhưng cost
guard phải chặn: user gửi đúng 10 request/phút (không vượt hạn mức) nhưng mỗi
request tốn rất nhiều token, nên ngân sách tháng cạn trong vài phút, khi đó
trả 402. Tình huống ngược lại: user gửi 100 request cực rẻ trong vài giây, chi
phí còn xa ngân sách nhưng vượt 10 request/phút nên rate limit chặn với 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp làm một và cho kiểm tra Redis, khi Redis mất kết nối 30 giây thì:
1. Cả 3 container cùng gọi Redis trong health check và cùng thất bại.
2. Health check thất bại liên tiếp quá số lần cho phép, cả 3 bị đánh dấu
   unhealthy.
3. Orchestrator restart cả 3 container gần như cùng lúc.
4. Trong lúc đó không còn container nào phục vụ request, dịch vụ ngừng hẳn.
5. Khi Redis quay lại, các container vẫn đang khởi động lại nên dịch vụ tiếp
   tục gián đoạn thêm một lúc.
Một sự cố nhỏ của dependency biến thành sự cố toàn hệ thống. Tách ra thì
`/health` không đụng Redis nên không bị restart oan, còn `/ready` trả 503 chỉ
khiến load balancer tạm ngừng gửi request.

---

### Câu 9 — Stateless (CP4)

Tôi chạy `/ask` hai lượt với cùng `X-User-Id` và thấy `history_length` là 0
rồi 2 (lịch sử nằm ở Redis). Lưu ý: khi thử `--scale agent=3` tôi gặp lỗi trùng
cổng 8000 vì compose map cổng cố định, nên phần nhiều instance dưới đây là suy
luận chứ không phải quan sát trực tiếp. Nếu lịch sử nằm trong dict Python của
từng container, mỗi container có RAM riêng, nên với 3 instance sau load
balancer `history_length` sẽ nhảy lộn xộn (ví dụ 0, 0, 0, 2, 0...) tùy request
rơi vào container nào, vì container nhận request sau không thấy dữ liệu của
container trước. Agent sẽ "mất trí nhớ" ngẫu nhiên.

---

### Câu 10 — Deploy thật (CP5)

<Ghi MỘT lỗi thật bạn gặp khi deploy: thông báo lỗi là gì, bạn tìm nguyên
nhân bằng cách nào (ví dụ xem `railway logs`, kiểm tra Variables), và sửa
ra sao.>