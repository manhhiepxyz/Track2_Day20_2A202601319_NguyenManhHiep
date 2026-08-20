# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **6 physical · 12 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 65.8 | 100% |
| 3 | 65.7 | 100% |
| 6 | 65.8 | 100% |
| 12 | 65.6 | 100% |
| 24 | 65.4 | 99% |

**Best**: `-t 6` at 65.8 tok/s
**Slowest tested**: `-t 24` at 65.4 tok/s (1.01x spread)
**Against the physical-core default** (`-t 6`, 65.8 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=6 make bench
```

## Your explanation

**Giải thích:**
Đường cong (curve) ở đây hoàn toàn phẳng lỳ, dao động cực nhỏ quanh mốc ~65.8 tok/s ở mọi mức thread (từ 1 đến 24). Kết quả này trái ngược với quy luật thông thường (kỳ vọng peak ở số core vật lý là 6).
Nguyên nhân là do trên máy này, `llama.cpp` đang dùng backend **Vulkan** và tham số `ngl=99` đã đẩy toàn bộ các layer (fully offloaded) lên tính toán trên card đồ họa (GPU). Vì CPU gần như không phải xử lý việc tính toán ma trận nào cả, nên số lượng luồng (threads) cấu hình cho CPU (`-t`) không mang lại bất kỳ tác động nào đến tốc độ decode cuối cùng.
