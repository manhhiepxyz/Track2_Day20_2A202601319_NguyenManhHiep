# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 3558.8 | 3558.9 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 3171.1 | 3171.2 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 3115.0 | 3115.2 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **3281.6** · total **3281.8**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Your observation

**Nhận xét:**
* Giai đoạn **LLM** chiếm **100%** tổng thời gian xử lý của pipeline (khoảng ~3281 ms / câu hỏi).
* Các giai đoạn **embed** và **retrieve** chỉ mất 0.0 ms. Nguyên nhân không phải vì máy chạy siêu tốc, mà vì hai giai đoạn này đang được **làm giả (stub)** trong code của bài lab, chúng không chạy thật.
* Qua đó cho thấy, trong một ứng dụng RAG thực tế, việc sinh văn bản (LLM generation) gần như luôn luôn là nút thắt cổ chai (bottleneck) lớn nhất về mặt thời gian so với các công đoạn khác.
