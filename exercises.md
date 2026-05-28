# Ngày 1 — Bài Tập & Phản Ánh
## Nền Tảng LLM API | Phiếu Thực Hành

**Thời lượng:** 1:30 giờ  
**Cấu trúc:** Lập trình cốt lõi (60 phút) → Bài tập mở rộng (30 phút)

---

## Phần 1 — Lập Trình Cốt Lõi (0:00–1:00)

Chạy các ví dụ trong Google Colab tại: https://colab.research.google.com/drive/172zCiXpLr1FEXMRCAbmZoqTrKiSkUERm?usp=sharing

Triển khai tất cả TODO trong `template.py`. Chạy `pytest tests/` để kiểm tra tiến độ.

**Điểm kiểm tra:** Sau khi hoàn thành 4 nhiệm vụ, chạy:
```bash
python template.py
```
Bạn sẽ thấy output so sánh phản hồi của GPT-4o và GPT-4o-mini.

---

## Phần 2 — Bài Tập Mở Rộng (1:00–1:30)

### Bài tập 2.1 — Độ Nhạy Của Temperature
Gọi `call_openai` với các giá trị temperature 0.0, 0.5, 1.0 và 1.5 sử dụng prompt **"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> Khi temperature tăng từ 0.0 → 1.5, phản hồi đi từ **xác định và lặp lại** (gần như giống hệt nhau qua nhiều lần gọi, chọn các sự thật phổ biến như "Việt Nam có 54 dân tộc") sang **đa dạng và sáng tạo hơn** (chọn các góc độ ít gặp, ngôn từ giàu hình ảnh). Ở mức 1.5, phản hồi bắt đầu xuất hiện cấu trúc câu kém tự nhiên, lỗi chính tả tiếng Việt, và đôi khi sai sự thật (hallucination). Quy luật: **temperature cao = sáng tạo + rủi ro sai cao**, temperature thấp = ổn định + nhàm chán.

**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> Tôi sẽ đặt **temperature = 0.2 – 0.3**. Chatbot CSKH cần câu trả lời **nhất quán** (cùng câu hỏi → cùng đáp án để khách không bối rối), **chính xác** (giảm hallucination về chính sách, giá, quy trình), và **dễ kiểm thử/QA** (đầu ra ít ngẫu nhiên thì dễ viết test, dễ tái tạo bug). Đặt 0.0 hoàn toàn cũng được, nhưng giữ một chút ngẫu nhiên (0.2-0.3) giúp câu chữ tự nhiên hơn, tránh cảm giác máy móc cứng nhắc.

---

### Bài tập 2.2 — Đánh Đổi Chi Phí
Xem xét kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người thực hiện 3 lần gọi API, mỗi lần trung bình ~350 token.

**Ước tính xem GPT-4o đắt hơn GPT-4o-mini bao nhiêu lần cho workload này:**
> **Workload/ngày:** 10,000 users × 3 calls × 350 tokens = **10,500,000 tokens/ngày**. Giả định chia đều 50/50 input/output → 5.25M input + 5.25M output.
>
> | Model | Input cost | Output cost | **Tổng/ngày** | **Tổng/tháng (30 ngày)** |
> |---|---|---|---|---|
> | GPT-4o ($5 / $20 per 1M) | 5.25M × $5 = $26.25 | 5.25M × $20 = $105.00 | **$131.25** | **~$3,937.50** |
> | GPT-4o-mini ($0.15 / $0.60 per 1M) | 5.25M × $0.15 = $0.79 | 5.25M × $0.60 = $3.15 | **$3.94** | **~$118.13** |
>
> **GPT-4o đắt hơn GPT-4o-mini ~33.3 lần.** (Tỉ lệ này không đổi dù chia input/output thế nào, vì cả hai cặp giá đều có tỉ lệ 5/0.15 = 20/0.6 = 33.3.)

**Mô tả một trường hợp mà chi phí cao hơn của GPT-4o là xứng đáng, và một trường hợp GPT-4o-mini là lựa chọn tốt hơn:**
> **GPT-4o xứng đáng** — Phân tích hợp đồng pháp lý / tài liệu y tế tiếng Việt phức tạp: cần suy luận đa bước, hiểu ngữ cảnh dài, độ chính xác cao vì sai một câu có thể dẫn đến hậu quả pháp lý/sức khỏe. Khối lượng thấp (vài chục request/ngày), nên chênh lệch $0.004 vs $0.0001/call là không đáng kể so với chi phí thuê chuyên gia review.
>
> **GPT-4o-mini tốt hơn** — Phân loại sentiment bình luận sản phẩm sàn TMĐT (tích cực/tiêu cực/trung tính), hoặc tag intent cho chatbot CSKH. Đây là tác vụ đơn giản, lặp đi lặp lại với hàng triệu request/ngày — mini đủ chính xác (>95%), tiết kiệm 33x chi phí giúp dự án khả thi về mặt kinh doanh.

---

### Bài tập 2.3 — Trải Nghiệm Người Dùng với Streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì non-streaming lại phù hợp hơn?** (1 đoạn văn)
> **Streaming** quan trọng nhất trong các tình huống **người dùng phải chờ và đọc trực tiếp đầu ra**: chatbot hội thoại, trợ lý viết code (như Cursor/Copilot Chat), tóm tắt văn bản dài, hay sinh nội dung dài quá vài giây. Lý do là streaming **giảm độ trễ cảm nhận (perceived latency)** — người dùng thấy chữ chạy ngay sau ~200ms thay vì nhìn loading spinner 10-30 giây, tạo cảm giác hệ thống "sống" và phản hồi nhanh, đồng thời cho phép họ ngắt sớm nếu thấy hướng trả lời sai. Ngược lại, **non-streaming phù hợp hơn** khi: (1) cần **toàn bộ output trước khi xử lý** — ví dụ trả về JSON có cấu trúc, function calling, hoặc cần parse/validate trước khi hiển thị; (2) **tác vụ ngắn** (<1s) — streaming không tạo ra khác biệt UX rõ rệt; (3) **phải kiểm duyệt nội dung** trước khi cho user xem (moderation, lọc PII); và (4) **batch processing** chạy nền — không có người dùng nào đang chờ, nên chỉ tốn thêm phức tạp code mà không có lợi ích.


## Danh Sách Kiểm Tra Nộp Bài
- [x] Tất cả tests pass: `pytest tests/ -v`
- [x] `call_openai` đã triển khai và kiểm thử
- [x] `call_openai_mini` đã triển khai và kiểm thử
- [x] `compare_models` đã triển khai và kiểm thử
- [x] `streaming_chatbot` đã triển khai và kiểm thử
- [x] `retry_with_backoff` đã triển khai và kiểm thử
- [x] `batch_compare` đã triển khai và kiểm thử
- [x] `format_comparison_table` đã triển khai và kiểm thử
- [x] `exercises.md` đã điền đầy đủ
- [x] Sao chép bài làm vào folder `solution` và đặt tên theo quy định 
