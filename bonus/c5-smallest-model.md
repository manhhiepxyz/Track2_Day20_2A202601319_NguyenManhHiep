# C5. Challenge "Model nhỏ nhất vẫn hữu ích"

**Thiết lập:**
Em đã kiểm tra ngưỡng chịu đựng của mô hình ở 2 mức độ lượng tử hóa (Quantization) được yêu cầu tải về trong bài lab: `UD-Q4_K_XL` (4-bit) và `UD-Q2_K_XL` (2-bit). Việc đánh giá không chỉ dựa trên tốc độ đọc/sinh token mà dựa trên khả năng hữu ích của mô hình trong thực tế.

**Số liệu:**
*   **UD-Q4_K_XL (2.97 GB):** Trả lời đúng các câu hỏi suy luận cơ bản, cấu trúc câu mạch lạc và chính xác với tốc độ cao (khoảng 60 tok/s).
*   **UD-Q2_K_XL (2.24 GB):** Sinh ra kết quả lỗi, lặp từ, hoặc sinh ra ngôn ngữ không tự nhiên (hallucination). Khi hỏi các phép toán cơ bản (ví dụ "What is 1+1?"), mô hình lặp lại câu hỏi hoặc sinh ra thông tin rác. Hơn nữa tốc độ cực chậm trên Vulkan (7 tok/s) do bị giới hạn tính toán (Compute-limited).

**Nhận xét (Điều rút ra):**
Mức quantization 4-bit (UD-Q4_K_XL) là "model nhỏ nhất vẫn còn hữu ích" (the smallest useful model) để chạy serving trong điều kiện bộ nhớ hạn chế (RAM/VRAM dưới 4GB). Việc cố gắng ép xuống 2-bit (UD-Q2_K_XL) tuy tiết kiệm được vỏn vẹn ~0.7 GB VRAM nhưng lại phá hủy hoàn toàn chất lượng đầu ra, khiến mô hình trở nên vô dụng cho bất kỳ bài toán thực tế nào (sinh rác, lặp từ). Do đó, sự đánh đổi bộ nhớ lấy chất lượng ở mức 2-bit là hoàn toàn không xứng đáng để triển khai trong production.
