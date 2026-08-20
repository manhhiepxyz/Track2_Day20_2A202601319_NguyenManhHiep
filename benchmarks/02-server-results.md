# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=6` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 99 | 1.69 | 3600 | 13000 | 13000 | 7.8 | 0.0% |
| 50 | 94 | 1.61 | 25000 | 33000 | 35000 | 37.8 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.95x** (19% of linear) |
| P95 latency | **2.54x** |
| Effective concurrency at 50 users | 37.8 vs `--parallel 4` slots (occupancy/slot ratio 9.46) |

**Saturated.** Throughput delivered only 0.95x for 5x the offered load, and effective concurrency (37.8) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.95x while P95 moved 2.54x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

**Nhận xét:**
* **Điểm bão hòa:** Server đã bị bão hòa ngay từ mức ở hoặc dưới 10 users.
* **Bằng chứng:** Khi tải (offered load) tăng lên gấp 5 lần (từ 10 lên 50 users), thông lượng (throughput) thực tế hoàn toàn không tăng mà còn nhích nhẹ xuống (0.95x), trong khi độ trễ P95 (P95 latency) lại tăng vọt lên gấp 2.54 lần. Điều này có nghĩa là mọi request gửi thêm sau mức 10 users đều bị tắc nghẽn ở hàng đợi (queue) thay vì được xử lý song song, dẫn tới latency tăng mạnh mà throughput vẫn dậm chân tại chỗ. Effective concurrency nhảy vọt lên 37.8 vượt xa 4 slot của `--parallel 4`.
* **Giải pháp:** Để tăng goodput tại một mức SLO nhất định, knob (tham số) đầu tiên cần thay đổi là tăng số lượng slot xử lý đồng thời (`--parallel`). Bằng cách tăng số batch size, ta có thể tận dụng lợi thế của continuous batching, giúp throughput của hệ thống tăng lên, từ đó goodput cũng sẽ tăng.
