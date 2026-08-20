# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` · `--parallel 4` · 14 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.91 of 4 slots (98%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 10373 |

Highest sampled value was **3.91 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation

**Nhận xét:**
* Peak batch width đạt mức **3.91 trên 4 slot**.
* Con số này **không khớp** với effective concurrency (lên tới 37.8) trong file `02-server-results.md`.
* Em **tin vào con số Peak batch width (3.91) hơn** để đánh giá năng lực xử lý thực tế của server. Lý do là vì effective concurrency tính bằng định luật Little bao gồm cả những request đang phải xếp hàng chờ (`requests_deferred = 46`), tức là nó phản ánh "tổng lượng tải đang bị kẹt trong hệ thống" (occupancy). Trong khi đó, peak batch width (3.91) đo chính xác số lượng request đang thực sự được gộp vào xử lý cùng lúc bên trong GPU (utilisation), không bao giờ có thể vượt quá số lượng slot (`--parallel 4`) đã khai báo.
