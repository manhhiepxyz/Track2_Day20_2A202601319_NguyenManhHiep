# C8. Semantic Cache (Bộ đệm ngữ nghĩa)

**Thiết lập:** 
Em đã chạy `python bonus/serving-regimes/semantic-cache-demo.py --offline --sweep` để mô phỏng một semantic cache offline (sử dụng thuật toán Bag-of-Words cục bộ làm bộ tính similarity thay vì một Embedder xịn).

**Số liệu thu được (ngữ cảnh Bag-of-Words giả lập):**
- **False Hit:** Các prompt có chứa cụm từ khoá đặc trưng như "goodput@SLO" bị đánh đồng điểm similarity = 1.0 với nhau dù bản chất câu hỏi có thể khác nhau (ví dụ: "Tell me what goodput@SLO is." so với "Can you define goodput@SLO?"). Nếu hai câu này có ý nghĩa khác nhau hoàn toàn trong một ngữ cảnh khác (vd: "Why does goodput@SLO fail?"), nó vẫn sẽ bị False Hit (trả về kết quả cũ sai lệch).
- **False Miss:** Câu 2 "Explain TTFT and TPOT." và Câu 4 "What does time to first token mean?". Rõ ràng câu 4 là diễn giải (paraphrase) của câu 2 (vì TTFT chính là Time To First Token). Thế nhưng thuật toán similarity thô sơ lại chấm điểm 0.0 (miss hoàn toàn), bắt LLM phải tính toán lại từ đầu gây lãng phí (False Miss).
- Dễ thấy, không có bất kỳ ngưỡng Threshold nào giải quyết được đồng thời cả 2 lỗi này, vì phân phối điểm số của một Embedder dở tệ luôn cực đoan (chỉ có 0 và 1).

**Phân tích sự khác biệt giữa Decoder và Embedder chuyên dụng:**
Decoder được huấn luyện để sinh ra từ (token) tiếp theo, nên State của nó (Mean-pooled decoder state) chỉ tập trung vào ngữ cảnh một chiều cục bộ ở cuối câu. Trong khi đó, một Embedding model (như BGE-M3 hay Qwen3-Embedding) được huấn luyện bằng Contrastive Learning trên toàn bộ câu (bi-directional) để kéo các câu cùng nghĩa lại gần nhau và đẩy các câu khác nghĩa ra xa trong không gian vector. Do đó, việc tận dụng lại mô hình Chat để làm Embedding sẽ tạo ra một sentence encoder rất yếu, bắt buộc phải dùng mô hình Embedder chuyên dụng cho Semantic Cache.
