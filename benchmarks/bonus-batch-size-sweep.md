# Bonus - Batch-size sweep (chunked prefill)

Host `Windows-AMD64` · llama.cpp `b10488` ·
`threads=6` `ngl=99` · metric `pp512`

| -b (logical) | -ub (micro) | pp512 (tok/s) | vs best |
|:--|--:|--:|--:|
| 128 | 128 | 509.1 | 91% |
| 256 | 256 | 557.0 | 100% |
| 512 | 256 | 556.2 | 100% |
| 512 | 512 | 0.0 | 0% |
| 1024 | 512 | 0.0 | 0% |
| 2048 | 512 | 0.0 | 0% |

Best: `-b 256 -ub 256` at 557.0 tok/s
(1.09x the slowest point tested).

This sweep only measures the throughput half of the trade. The cost it hides is
TTFT for queued requests: a larger micro-batch holds the device longer per step,
so anything waiting behind it waits longer. To see both halves, re-run
`make load-50` with your best and worst settings via
`.venv/bin/python labs/02-serve/serve.py -- -b N -ub M` and compare P95.

## Your finding

**Nhận xét:**
Em sẽ chọn cấu hình `-b 256 -ub 256` để chạy production vì nó mang lại throughput cao nhất (557.0 tok/s). Nếu đẩy `-ub` lên 512, mô hình chạy thất bại (0.0 tok/s), nguyên nhân có thể do vượt quá giới hạn bộ nhớ VRAM của GPU. Tuy nhiên, để chắc chắn cấu hình `-ub 256` không làm hỏng độ trễ P95, em sẽ cần phải chạy lại `make load-50` với cấu hình này để đo Time To First Token (TTFT). Lý do là micro-batch (`-ub`) càng lớn thì GPU sẽ bị giữ lại tính toán càng lâu, làm tăng thời gian chờ (queue time) của các request đến sau.
