# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Nguyen Manh Hiep
**Cohort:** A20-K3
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime _(rubric 1, 2 — 10 điểm)_

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11
- **CPU:** 11th Gen Intel(R) Core(TM) i5-11400H @ 2.70GHz
- **Cores:** 6 physical / 12 logical
- **CPU extensions:** AVX2 / AVX-512
- **RAM:** 15.8 GB
- **Accelerator:** nvidia_cuda, vulkan (GeForce RTX 3050 Laptop GPU, 4096 MiB)
- **llama.cpp asset đã tải:** llama-b10488-bin-win-vulkan-x64.zip
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** gemma-4-E2B-it-UD-Q4_K_XL.gguf + gemma-4-E2B-it-UD-Q2_K_XL.gguf

**Chạy ở đâu:** laptop của tôi

**Setup story** (≤ 80 chữ):
File `lab.ps1` gốc bị lỗi encoding khi in ký tự "—" khiến PowerShell báo lỗi. Em đã phải sửa lại thành "-" và truyền thêm biến môi trường `$env:PYTHONIOENCODING="utf-8"` để lệnh chạy bình thường trên Windows.

---

## 2. Đo lường _(rubric 3, 4, 5 — 20 điểm)_

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
| ------------ | --------: | --------: | ----------------: | ----------------: | -------------------: | -------------: |
| UD-Q4_K_XL   |      2.97 |     10980 |         383 / 687 |       16.0 / 16.3 |   1394 / 1694 / 1694 |           62.5 |
| UD-Q2_K_XL   |      2.24 |      6752 |         517 / 927 |     137.3 / 137.8 |   9161 / 9551 / 9551 |            7.3 |

**Quan sát** (≤ 60 chữ):
Bản 4-bit nhanh hơn bản 2-bit tới 8.56 lần (62.5 so với 7.3 tok/s) dù nặng hơn 0.73 GB. Lý do là máy bị giới hạn tính toán (compute-limited) khi giải nén Q2 trên Vulkan GPU. Bản 4-bit hoàn toàn xứng đáng dùng.

---

## 3. Serving under load _(rubric 8, 9, 10 — 20 điểm)_

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users |  RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
| ----: | ---: | -------: | -------: | -------: | ---------------: | -------: |
|    10 | 1.69 |     3600 |    13000 |    13000 |              7.8 |     0.0% |
|    50 | 1.61 |    25000 |    33000 |    35000 |             37.8 |     0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 0.95×
- **P95 tăng:** 2.54×
- **Effective concurrency ở 50 users:** 37.8 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.91 / 4 slots

**Saturation reading** (≤ 80 chữ):
Server bão hòa ngay mức 10 users. Bằng chứng là RPS giảm (0.95x) nhưng P95 tăng vọt (2.54x) khi tải tăng 5x, tức là request bị tắc ở queue. Để tăng goodput@SLO, cần chỉnh knob `--parallel` lên cao hơn để xử lý đồng thời nhiều hơn (continuous batching).

---

## 4. Integration _(rubric 12, 13 — 15 điểm)_

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day                   | Piece          | Real hay stub? |
| --------------------- | -------------- | -------------- |
| N16 Cloud/IaC         | stub           | stub           |
| N17 Data pipeline     | stub           | stub           |
| N18 Lakehouse         | stub           | stub           |
| N19 Vector + features | stub           | stub           |
| N20 Serving           | `llama-server` | real           |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.0 ms
- llm: 3281.6 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ):
Nút thắt cổ chai nằm 100% ở LLM generation vì embed/retrieve đang được làm giả (stub). Điều này khớp với kỳ vọng. Nếu muốn giảm latency 2x, phải tối ưu thẳng vào giai đoạn LLM (dùng model nhỏ hơn hoặc tối ưu GPU offload).

---

## 5. The single change that mattered most _(rubric 11 — 10 điểm)_

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** Thay đổi Thread Sweep (`-t`) từ 1 đến 24

```
before:  65.8 tok/s (t=1)
after:   65.4 tok/s (t=24)
speedup: 1.00× (Không đổi)
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Đường cong tuning hoàn toàn phẳng lỳ, không có peak ở số lượng physical cores như lý thuyết thông thường của CPU.

Lý do của sự bất ngờ này là do em đang chạy backend Vulkan (dùng RTX 3050) với tham số `ngl=99` (đẩy 100% tính toán ma trận lên GPU). Vì GPU đang gánh toàn bộ quá trình tính toán inference, CPU gần như không có vai trò tính toán. Do đó, việc thay đổi số lượng CPU threads (`-t`) hoàn toàn không mang lại bất kỳ sự thay đổi nào về tốc độ. Đây là minh chứng rõ nhất cho sự khác biệt giữa Compute-bound trên CPU và GPU-offloaded inference.

---

## 6. Bonus _(optional — tối đa 20 điểm)_

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** Chạy Batch-size Sweep để đánh giá tốc độ xử lý chunked prefill (Tiêu chí B2, B3)

**Numbers:**

```
before:  509.1 tok/s (-b 128 -ub 128)
after:   557.0 tok/s (-b 256 -ub 256)
speedup: 1.09×
```

**Điều này nói lên gì mà deck chưa nói:**

Việc tăng kích thước chunked prefill (micro-batch size `-ub`) giúp tăng thông lượng xử lý prefill (từ 509 lên 557 tok/s) do tận dụng tốt hơn phần cứng tính toán song song của GPU. Tuy nhiên, việc đẩy kích thước `-ub` lên quá lớn (chẳng hạn 512) khiến hệ thống ngưng trệ hoàn toàn (0.0 tok/s), có thể do chạm mức giới hạn VRAM (4GB) của card đồ họa RTX 3050, dẫn đến tràn bộ nhớ. 

Hơn nữa, deck cũng chưa nhấn mạnh rằng việc chọn `-ub` lớn nhất có thể đo lường được (557 tok/s) không phải lúc nào cũng là cấu hình lý tưởng khi chạy production (serving). Dù nó giúp throughput cao nhất, nhưng mỗi micro-batch lớn sẽ giam giữ GPU trong một khoảng thời gian dài hơn. Việc ngâm GPU quá lâu sẽ gây ảnh hưởng tiêu cực tới Time To First Token (TTFT) của các luồng xử lý khác đang nằm chờ trong queue (hiệu ứng "chặn xe bus" - head-of-line blocking). Do đó, cấu hình `-b` và `-ub` lý tưởng nhất cần phải được chọn dựa trên sự thoả hiệp giữa giới hạn phần cứng (VRAM) và ngưỡng latency P95 mục tiêu (SLO) của máy chủ.

---

## 7. Điều làm bạn ngạc nhiên nhất _(optional)_

Ngạc nhiên nhất là bản nén 2-bit lại chạy chậm hơn bản 4-bit gấp 8 lần do máy bị compute-limited với Q2_K trên Vulkan backend. Điều này đi ngược lại lầm tưởng "model càng nhẹ chạy càng nhanh".

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
