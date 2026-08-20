# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=6` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 10980 | 383 / 687 | 16.0 / 16.3 | 1394 / 1694 / 1694 | 62.5 |
| UD-Q2_K_XL | 2.24 | 6752 | 517 / 927 | 137.3 / 137.8 | 9161 / 9551 / 9551 | 7.3 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **8.56x SLOWER** than `UD-Q4_K_XL` here, despite being 0.73 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Your observation

**Nhận xét:**
* **Về dung lượng:** Bản 2-bit (2.24 GB) nhẹ hơn bản 4-bit (2.97 GB) khoảng 0.73 GB.
* **Về tốc độ (Decode):** Bản 4-bit có tốc độ sinh chữ rất nhanh (62.5 tok/s), trong khi bản 2-bit lại bị tụt thê thảm xuống chỉ còn 7.3 tok/s (chậm hơn 8.56 lần).
* **Kết luận:** Trên cấu hình máy hiện tại, thiết bị bị giới hạn về sức mạnh tính toán (compute-limited) đối với định dạng Q2_K. Quá trình giải nén (dequantization) của bản 2-bit quá phức tạp nên tốn nhiều thời gian hơn cả băng thông RAM tiết kiệm được. Do đó, **bản 4-bit (UD-Q4_K_XL)** hoàn toàn xứng đáng và là lựa chọn tốt nhất để sử dụng trên máy này.
