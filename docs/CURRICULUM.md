# Curriculum — Week-by-Week Specs

Each week follows the same format so Claude Code can parse it with `/start-week N`.

- **Goal** — one sentence.
- **Read** (~3h) — scoped; details and links in [`READING.md`](READING.md).
- **Experiments** (~3h) — small, throwaway measurements.
- **Build** (~5–7h) — directory, spec, and ownership tags **[L] / [P] / [S]** (see [`DESIGN.md §4`](DESIGN.md#4-collaboration-model-with-claude-code)).
- **Minimum** / **Stretch** — the week is done when the minimum is met.
- **Acceptance** — concrete checks.
- **Measure** — numbers to log in `PROGRESS.md`.
- **Exit questions** — IDs from [`QUESTIONS.md`](QUESTIONS.md).

## Overview

```
Month 1  Systems foundations     W1 C++ · W2 Linux/OS · W3 Networking · W4 Performance + RPC server
Month 2  Distributed systems     W5 Fundamentals · W6 Replication + asyncio · W7 Queues/scheduling · W8 Task queue
         ── B1 buffer ──
Month 3  GPU + PyTorch           W9 GPU arch + roofline · W10 CUDA · W11 PyTorch internals · W12 Profiling + Triton
Month 4  Distributed training    W13 DDP · W14 Collectives · W15 FSDP/memory/pipeline · W16 Tensor parallel + scaling
         ── B2 buffer ──
Month 5  LLM inference           W17 Model from scratch · W18 KV cache · W19 Continuous batching · W20 Server + benchmarks
Month 6  Serving systems         W21 Read vLLM · W22 PagedAttention · W23 Triton attention
         ── B3 buffer ──
Capstone                         W24 SGLang + prefix caching · W25 InferX v1.0 · W26 Benchmark & attribution
```

---

# Month 1 — Systems Foundations

**Month goal:** be comfortable reasoning about what happens below Python: memory, processes, threads, syscalls, sockets, and how to measure them.

**Month deliverable:** `m1-systems/rpc/` + `reports/m1-rpc-server.md`.

## Week 1 — C++ for Systems

**Goal:** read and write the C++ that infrastructure code is made of: ownership, RAII, threads, synchronization.

**Read:** *A Tour of C++* — the chapters on classes, essential operations (copy/move), templates, resource management / smart pointers, and concurrency. Use *C++ Primer* as a lookup reference only.

**Experiments:**
- Print `sizeof` and member offsets of a few structs; observe padding and reorder fields to shrink them.
- Write a class that logs constructor/copy/move/destructor calls; watch what `std::vector::push_back` does with and without `noexcept` move.
- Build a two-file program with CMake; deliberately cause and read an "undefined reference" linker error.

**Build:** `m1-systems/threadpool/`
- [S] CMake project with GoogleTest (or Catch2), sanitizer build presets (ASan, TSan).
- [L] `ThreadPool(size_t n)`, `submit(F&&, Args&&...) -> std::future<R>`, task queue guarded by `std::mutex` + `std::condition_variable`, graceful shutdown (drain queued tasks, then join).
- [P] Exception propagation through futures.

**Minimum:** thread pool with futures and graceful shutdown; tests pass under TSan.
**Stretch:** bounded queue with blocking `submit` (your first backpressure); `shutdown_now()` that discards pending tasks.

**Acceptance:**
- 10,000 tasks from 8 submitting threads complete with correct results.
- Destructor with pending tasks neither deadlocks nor leaks (ASan clean).
- TSan reports no races.

**Measure:** tasks/sec for trivially small tasks with 1, 2, 4, 8 workers. Is throughput flat or does it fall? (You're measuring lock contention.)

**Exit questions:** SYS-01, SYS-02, SYS-03, SYS-04, SYS-05

---

## Week 2 — Linux: Processes, Memory, System Calls

**Goal:** be able to trace a Python program down to system calls and explain what the kernel does with them.

**Read:** *OSTEP* — Processes, Process API, Limited Direct Execution, Address Spaces, Address Translation, Paging (incl. TLBs), Concurrency intro, Thread API. *TLPI* as reference for `fork`/`exec`/pipes/signals/mmap.

**Experiments:** `m1-systems/syscall-labs/`
- `strace -f -tt` a minimal Python HTTP server handling one request. Annotate every syscall.
- `fork()` a process that touched 1 GB; measure RSS before and after writes in the child (copy-on-write).
- Measure context-switch cost: two processes ping-ponging a byte over a pipe; compare with two threads using a condition variable.
- Share data between processes with `mmap`/shared memory vs a pipe; compare throughput.

**Build:** `m1-systems/syscall-labs/`
- [L] A tiny shell in C or C++: `fork` + `exec` + `wait`, one pipe (`a | b`), `SIGINT` handling.
- [S] Scripts that run each experiment and print results.

**Minimum:** annotated strace of the HTTP request; mini shell with one pipe; context-switch measurement.
**Stretch:** `/proc/<pid>/maps` walkthrough of a running Python process; explain each region.

**Acceptance:** shell runs `ls -l | wc -l` correctly; strace annotation is in the week notes.

**Measure:** context switch (process vs thread) in µs; fork time for a 1 GB process before/after writes.

**Exit questions:** SYS-06, SYS-07, SYS-08, SYS-09, SYS-10, SYS-11

---

## Week 3 — Networking

**Goal:** understand sockets, TCP behavior, and I/O multiplexing well enough to build and reason about a server.

**Read:** *Computer Networking: A Top-Down Approach* — the transport-layer chapter (TCP reliability, flow control, congestion control) and HTTP sections. *Beej's Guide* end-to-end (it's short). `man 7 epoll`. Skim the HTTP/2 and gRPC overviews.

**Experiments:**
- Wireshark/`tcpdump` a TCP handshake and an HTTP request; label SYN/SYN-ACK/ACK and teardown.
- Use `tc netem` on loopback to add 50 ms latency and 1% loss; observe throughput of a bulk transfer.
- Compare new-connection-per-request vs a reused connection (connection pooling).

**Build:** `m1-systems/tcp-server/`
- [L] TCP server: acceptor thread → your W1 thread pool → handler. Length-prefixed message framing.
- [L] Second version using `epoll` (single event loop thread, non-blocking sockets).
- [S] Load generator: N concurrent clients, records per-request latency to a file.

**Minimum:** thread-pool server + load generator; latency percentiles at 1, 10, 100 clients.
**Stretch:** `epoll` version and a comparison with the thread-pool version.

**Acceptance:** correct framing under partial reads/writes (test with a client that sends one byte at a time).

**Measure:** req/s, p50/p95/p99 vs number of clients, both server versions if done.

**Exit questions:** NET-01, NET-02, NET-03, NET-04, NET-05, NET-06

---

## Week 4 — Linux Performance + Month 1 Project

**Goal:** profile instead of guess, and combine Month 1 into a measured RPC server.

**Read:** *Systems Performance* — Methodologies (USE method, workload characterization), CPUs, Memory, and Benchmarking chapters. Brendan Gregg's flame graph page.

**Experiments:**
- `perf stat` cache misses on row-major vs column-major array traversal.
- `perf record` + flame graph of your W3 server under load.
- Use `vmstat`, `pidstat`, `iostat`, `sar` during a load test; identify each column you rely on.

**Build:** `m1-systems/rpc/` — **Mini RPC Server**
```
Client ──TCP──► RPC Server ──► Queue ──► Thread Pool
                     └──► Metrics
```
- [P] Protocol: request ID, method name, payload, deadline. Use protobuf or a hand-written binary format.
- [L] Server-side timeouts (drop requests whose deadline has passed before running them), graceful shutdown (stop accepting, drain, exit).
- [L] Metrics: request count, in-flight, queue depth, latency histogram.
- [S] Load test driver and report generator.

**Minimum:** concurrent requests with IDs, timeouts, metrics, graceful shutdown; performance report.
**Stretch:** find and fix one bottleneck you identified with `perf`, and report before/after.

**Acceptance:** under 100 concurrent clients, requests past deadline are rejected with an explicit error; `SIGTERM` drains in-flight requests.

**Measure (report `reports/m1-rpc-server.md`):**
```
100 concurrent clients
Throughput: X req/s   p50: X ms   p95: X ms   p99: X ms
CPU: X%   Memory: X MB
Bottleneck identified: ...
```

**Exit questions:** SYS-12, SYS-13, SYS-14, SYS-15

**Checkpoint:** run `/checkpoint 1`.

---

# Month 2 — Distributed Systems

**Month goal:** move from "one machine doing many things" to "many machines cooperating despite failure."

**Month deliverable:** `m2-distributed/taskqueue/` + `reports/m2-taskqueue.md`.

## Week 5 — Distributed Systems Fundamentals

**Goal:** reason about partial failure, timeouts, retries, and idempotency.

**Read:** *DDIA* — "Reliable, Scalable, and Maintainable Applications", "Encoding and Evolution", "The Trouble with Distributed Systems". (Chapter numbers differ between editions; use the titles.)

**Experiments:**
- Port your RPC protocol to gRPC + protobuf in Python; compare message sizes vs JSON.
- Inject faults into the M1 RPC server (random delay, random drop) and observe client behavior with and without timeouts.

**Build:** `m2-distributed/rpc-client/` (small)
- [L] Client with deadlines, retries with exponential backoff + jitter, and idempotency keys.
- [L] Server-side dedup table so a retried non-idempotent request (e.g. "increment counter") executes once.
- [S] Fault-injection proxy that delays/drops/duplicates messages.

**Minimum:** retries + backoff + idempotency keys, demonstrated with the fault-injection proxy.
**Stretch:** retry budgets (cap retries as a fraction of traffic) and a demo of a retry storm without them.

**Acceptance:** with 20% message drops, a counter incremented 1,000 times by retrying clients ends at exactly 1,000.

**Measure:** success rate and p99 latency vs drop rate (0%, 5%, 20%) with and without retries.

**Exit questions:** DS-01, DS-02, DS-03, DS-04

---

## Week 6 — Replication, Consensus (conceptual) + asyncio Internals

**Goal:** understand replication and Raft conceptually; understand the asyncio event loop that later serving code depends on.

**Read:** *DDIA* — "Replication" and "Consistency and Consensus". The Raft paper (sections on leader election and log replication). Python docs on the asyncio event loop; skim `asyncio/base_events.py`.

**Experiments:**
- Use the Raft visualization (raft.github.io) to step through a leader failure and a split vote.
- Write a 30-line event loop with `selectors` and generators; compare its structure to `asyncio`.
- Measure: 10,000 concurrent sleeping coroutines vs 10,000 threads (memory, startup time).
- Show the GIL: CPU-bound work in threads vs processes vs `asyncio`.

**Build:**
- [L] Toy event loop (`m2-distributed/toy-loop/`): schedule callbacks, timers, and socket readiness.
- [P] (Stretch) `m2-distributed/kv-replication/`: 3-node leader/follower KV store with heartbeats, and simulated leader failure, worker failure, network timeout, and retry. Leader election may be simplified.

**Minimum:** toy event loop; written walkthrough of Raft election and log commit in week notes.
**Stretch:** the 3-node replication lab.

**Acceptance:** toy loop serves multiple concurrent echo clients on one thread.

**Measure:** coroutine vs thread memory per task; GIL experiment timings.

**Exit questions:** DS-05, DS-06, DS-07, PY-01, PY-02

---

## Week 7 — Queues, Scheduling, Backpressure

**Goal:** build the request-scheduling mental model that LLM serving reuses almost unchanged.

**Read:** Little's Law and M/M/1 intuition (any short source; see `READING.md`). Articles on backpressure, load shedding, and admission control.

**Experiments:**
- Simulate an M/M/1 queue in Python; plot mean and p99 latency vs utilization from 10% to 99%.
- Compare FIFO vs shortest-job-first on a mix of short and long tasks: mean vs tail latency.

**Build:** `m2-distributed/sched/` — a small asyncio library
```
Requests ──► Request Queue ──► Scheduler ──► Workers
```
- [L] Bounded queue with admission control (reject with 429 when full or when estimated wait exceeds the deadline).
- [L] Pluggable policies: FIFO, priority, shortest-job-first (with a cost estimate).
- [L] Token-bucket rate limiter.
- [S] Simulation harness + plots.

**Minimum:** bounded queue + admission control + two policies + latency-vs-load plot.
**Stretch:** work stealing between per-worker queues; fairness across tenants.

**Acceptance:** under 2× overload, accepted requests keep bounded p99 latency; excess is rejected rather than queued forever.

**Measure:** p50/p99 latency and rejection rate vs offered load, per policy.

**Exit questions:** DS-08, DS-09, DS-10

---

## Week 8 — Month 2 Project: Distributed Task Queue

**Goal:** build the coordinator/worker system that becomes "scheduler + GPU workers" later.

**Read:** the MapReduce paper (focus on master, worker failure handling, and stragglers).

**Build:** `m2-distributed/taskqueue/` — Python, FastAPI, asyncio, Redis or PostgreSQL, Docker Compose. **No Celery/Kafka for the core scheduler.**
```
API ──► Coordinator ──► Worker × N
```
- [S] Docker Compose setup, API skeleton, persistence layer.
- [L] Worker registration and heartbeats; failure detection by missed heartbeats.
- [L] Task assignment with leases (task ownership expires; another worker can take it).
- [L] Retries with max attempts; dead-letter state.
- [P] Result storage; metrics endpoint.

**Minimum:** all of the above with at-least-once semantics.
**Stretch:** speculative re-execution of stragglers (from MapReduce); coordinator restart recovery from persisted state.

**Acceptance:** `docker kill` a worker mid-task → the task is re-run elsewhere and completes; no task is lost; duplicated execution is detectable.

**Measure (report `reports/m2-taskqueue.md`):** tasks/s vs worker count; time-to-detect worker failure vs heartbeat interval; duplicate execution rate under failures.

**Exit questions:** DS-11, DS-12

**Checkpoint:** run `/checkpoint 2`.

---

## Buffer B1

Default use: Week 6 replication lab stretch, or deepen asyncio. See `DESIGN.md §6.4`.

---

# Month 3 — GPU + PyTorch Systems

**Month goal:** understand what the GPU is doing under PyTorch and be able to answer "where is my time actually going?"

**Month deliverable:** `m3-gpu/kernels/` + `reports/m3-kernels.md`.

## Week 9 — GPU Architecture + Roofline

**Goal:** a working model of SMs, warps, memory hierarchy, and the compute-vs-bandwidth tradeoff.

**Read:** *PMPP* — chapters on GPU compute architecture/scheduling and memory architecture/data locality. The architecture whitepaper for **your** GPU generation. The roofline model (Williams et al. or any good explainer).

**Experiments:**
- Find your GPU's peak FLOP/s (per dtype) and HBM bandwidth from the spec sheet; compute the ridge point (FLOP/byte).
- Measure achieved bandwidth with a large `tensor.copy_()` and achieved FLOP/s with a large matmul. What fraction of peak do you get?
- Compute the arithmetic intensity of: vector add, softmax, a 4096×4096 matmul, and a batch-1 matrix-vector product. Place each on your roofline.

**Build:** `m3-gpu/roofline/`
- [P] Script that measures bandwidth and FLOP/s for a set of PyTorch ops and plots them on your GPU's roofline.

**Minimum:** roofline plot with measured points and a written explanation.
**Stretch:** sweep matmul sizes and show the transition from memory-bound to compute-bound.

**Acceptance:** each plotted op is labeled compute-bound or memory-bound, with reasoning.

**Measure:** peak vs achieved bandwidth and FLOP/s.

**Exit questions:** GPU-01, GPU-02, GPU-03, GPU-04, GPU-05

---

## Week 10 — CUDA Programming

**Goal:** write, check, and benchmark basic CUDA kernels.

**Read:** *PMPP* — data-parallel computing, multidimensional grids, performance considerations (coalescing), reduction. CUDA C++ Programming Guide as reference (streams, events).

**Build:** `m3-gpu/cuda/`
- [S] Build system (CMake or `torch.utils.cpp_extension`), correctness-test harness against PyTorch, CUDA-event timing helper.
- [L] Vector add → naive matmul → tiled shared-memory matmul → reduction (naive → warp-shuffle) → row-wise softmax.
- [L] Overlap host-to-device copies with compute using two streams and pinned memory.

**Minimum:** all five kernels correct; tiled vs naive matmul comparison.
**Stretch:** online softmax (single pass); stream overlap shown in Nsight Systems.

**Acceptance:** all kernels match PyTorch within tolerance on random inputs, including non-multiple-of-block-size shapes.

**Measure:** GB/s for bandwidth-bound kernels, GFLOP/s for matmul, as % of the roofline from Week 9.

**Exit questions:** GPU-06, GPU-07, GPU-08, GPU-09

---

## Week 11 — PyTorch Internals

**Goal:** trace a PyTorch call from Python to a CUDA kernel.

**Read:** ezyang's "PyTorch internals" blog post; PyTorch docs on the dispatcher and CUDA semantics (streams, caching allocator). Navigate `c10/`, `aten/src/ATen/native/`, `torch/`.

**Experiments:**
- Pick two operators (e.g. `torch.add`, `torch.softmax`); find their entries in `native_functions.yaml`, their dispatch keys, and their CUDA kernels.
- Show views vs copies: `transpose`, `view`, `reshape`, `contiguous`; print `stride()` and `data_ptr()`.
- Find hidden synchronizations: `.item()`, `.cpu()`, printing a CUDA tensor, `nonzero()`. Use `torch.cuda.set_sync_debug_mode`.
- Watch the caching allocator: `memory_allocated` vs `memory_reserved`, and `torch.cuda.memory_snapshot`.

**Build:** `m3-gpu/pytorch-tracing/`
- [L] Written traces (markdown + file/line links) for two operators.
- [P] A custom op registered via `torch.library` that wraps your W10 softmax.

**Minimum:** two operator traces; sync-point and stride experiments.
**Stretch:** custom op with autograd support.

**Acceptance:** custom op passes `torch.library.opcheck` (or equivalent tests) and matches `torch.softmax`.

**Measure:** cost of an accidental sync inside a loop (with vs without `.item()`).

**Exit questions:** PT-01, PT-02, PT-03, PT-04, PT-05

---

## Week 12 — Profiling + Triton Basics + Month 3 Project

**Goal:** profile PyTorch workloads and implement one op three ways.

**Read:** PyTorch Profiler tutorial; Nsight Systems and Nsight Compute quick-starts; the official Triton tutorials (vector add, fused softmax, layer norm).

**Experiments:**
- Profile a small MLP training step with the PyTorch profiler; export a Chrome trace; find launch overhead and gaps where the GPU is idle.
- Nsight Compute one kernel; read achieved memory throughput and occupancy.

**Build:** `m3-gpu/kernels/` — **Operator optimization project.** Pick **RMSNorm** (it reappears in InferX) or **Softmax**.
- [S] Benchmark harness: shapes sweep, CUDA-event timing, correctness checks, plots.
- [L] Triton implementation (start from the tutorial pattern, write it yourself).
- [P] CUDA implementation (reuse/extend W10).
- Compare: PyTorch eager baseline, `torch.compile`, CUDA, Triton.

**Minimum:** Triton + PyTorch baseline, correct, benchmarked across shapes, profiler-backed explanation.
**Stretch:** CUDA version; Triton autotuning; fused RMSNorm + residual add.

**Acceptance:** max abs error within tolerance vs reference in fp32 and bf16.

**Measure (report `reports/m3-kernels.md`):** latency, achieved GB/s and % of peak bandwidth, memory, per implementation and shape.

**Exit questions:** PT-06, KRN-01, KRN-02, KRN-03

**Checkpoint:** run `/checkpoint 3`.

---

# Month 4 — Distributed Training

**Month goal:** combine distributed systems + GPU + PyTorch; understand communication and memory as the limits of scale. Focus on what transfers to inference: collectives and **tensor parallelism**.

**Month deliverable:** `m4-dist-training/` + `reports/m4-scaling.md`.

**Hardware note:** develop everything on CPU with the `gloo` backend (`torchrun --nproc_per_node=4` works on one machine), then run measurements on a rented multi-GPU node with NCCL. See `DESIGN.md §3`.

## Week 13 — Data Parallelism (DDP)

**Goal:** understand DDP from process groups up.

**Read:** PyTorch Distributed overview + DDP tutorial; the PyTorch DDP paper (VLDB 2020: bucketing, overlap of communication with backward).

**Build:** `m4-dist-training/`
- [S] Small GPT-style model + synthetic/text dataset + training loop; `torchrun` launch scripts.
- [L] **Manual data parallelism**: broadcast initial params, then `all_reduce` gradients after `backward()`, averaged by world size.
- [P] Switch to `DistributedDataParallel`; confirm identical loss curves vs manual version.

**Minimum:** manual DP and DDP both match single-process training (same effective batch, same seed) within tolerance.
**Stretch:** implement gradient bucketing with backward hooks in your manual version.

**Acceptance:** loss curves of 1-process and N-process runs overlap.

**Measure:** step time breakdown (forward / backward / all-reduce) for manual vs DDP.

**Exit questions:** DT-01, DT-02, DT-03

---

## Week 14 — NCCL and Collectives

**Goal:** know what each collective does, what it costs, and why communication becomes the bottleneck.

**Read:** NCCL docs (collective operations section); a ring all-reduce explainer; `nccl-tests` README.

**Experiments:**
- [L] Implement ring all-reduce yourself with point-to-point `send`/`recv` (reduce-scatter phase + all-gather phase). Verify against `dist.all_reduce`.
- Run `nccl-tests` (or a PyTorch microbenchmark) for all-reduce, all-gather, reduce-scatter, broadcast, all-to-all across message sizes.
- Set `NCCL_DEBUG=INFO`; identify the topology/transport NCCL picked.

**Build:** `m4-dist-training/collectives/`
- [L] Ring all-reduce.
- [S] Bandwidth-vs-message-size benchmark and plots (algorithm bandwidth and bus bandwidth).

**Minimum:** ring all-reduce correct; collective bandwidth curves.
**Stretch:** compute vs communication overlap measured in a profiler trace of DDP.

**Acceptance:** your ring all-reduce matches `dist.all_reduce` for odd sizes not divisible by world size.

**Measure:** bus bandwidth per collective; latency floor for small messages.

**Exit questions:** DT-04, DT-05, DT-06

---

## Week 15 — FSDP, Memory, Pipeline Parallelism

**Goal:** account for every byte of training memory and know the tools that shard it.

**Read:** ZeRO paper (stages 1–3); PyTorch FSDP paper (or FSDP2 docs); GPipe paper (pipeline bubbles). Skim ZeRO-Infinity's offload idea.

**Experiments:**
- [L] Predict memory for your model: parameters + gradients + optimizer states (Adam) + activations, in fp32 and mixed precision. Then measure with `torch.cuda.max_memory_allocated`. Explain the gap.
- Toggle activation checkpointing; measure memory saved and step time added.

**Build:** `m4-dist-training/`
- [P] FSDP version of the W13 training script with mixed precision.
- [L] A memory calculator script: given model config + parallelism + dtype → predicted memory per GPU.

**Minimum:** memory prediction vs measurement table; FSDP run.
**Stretch:** a toy 2-stage pipeline with micro-batches; measure the bubble fraction vs number of micro-batches.

**Acceptance:** memory prediction within ~15% of measured peak (or the gap is explained).

**Measure:** peak memory and step time for DDP vs FSDP, with/without checkpointing.

**Exit questions:** DT-07, DT-08, DT-09, DT-10

---

## Week 16 — Tensor Parallelism + Month 4 Scaling Project

**Goal:** implement Megatron-style tensor parallelism (needed for inference later) and write the scaling report.

**Read:** Megatron-LM paper (column-parallel and row-parallel linear layers; the MLP and attention splits).

**Build:** `m4-dist-training/`
- [L] `ColumnParallelLinear` and `RowParallelLinear` with the correct collectives; a tensor-parallel transformer MLP block and attention block.
- [S] Scaling harness: 1 / 2 / 4 GPUs for DDP, FSDP, TP.

**Minimum:** TP MLP block matches the single-GPU block numerically; scaling numbers for DDP and FSDP at 1/2/4 GPUs.
**Stretch:** full TP transformer layer; TP inference forward pass timing (preview of InferX TP stretch).

**Acceptance:** TP block output matches reference within tolerance for TP=2 and TP=4.

**Measure (report `reports/m4-scaling.md`):** throughput, GPU utilization, memory, communication time, scaling efficiency. The report must answer: **why doesn't 4 GPUs give 4× performance?**

**Exit questions:** DT-11, DT-12, DT-13

**Checkpoint:** run `/checkpoint 4`.

---

## Buffer B2

Default use: rented multi-GPU time and finishing the scaling report.

---

# Month 5 — LLM Inference (InferX is born)

**Month goal:** understand inference mathematically and operationally, and build InferX v0.1: a working server with KV cache and continuous batching.

**Month deliverable:** `inferx/` v0.1 + `reports/m5-inferx-v0.1.md`.

## Week 17 — Transformer Inference from Scratch

**Goal:** implement the reference model yourself and understand prefill vs decode.

**Read:** the Llama architecture (RMSNorm, RoPE, SwiGLU, GQA) via the HF `modeling_llama.py` source; gpt-fast's `model.py` as a readable reference; revisit the Week 9 roofline.

**Build:** `inferx/inferx/model/`
- [S] `inferx/` package skeleton, `pyproject.toml`, pinned model revision in `config.py`, weight loader from safetensors.
- [L] `llama.py`: the full model (embedding, RoPE, GQA attention, SwiGLU MLP, RMSNorm, LM head), **no KV cache yet**.
- [L] `sampling.py`: greedy, temperature, top-k, top-p, seeded.
- [P] `tests/test_correctness.py`: logits match HF `transformers` within tolerance; greedy generation matches token-for-token (see `DESIGN.md §7.8`).

**Minimum:** model matches HF; naive generation (recompute everything each token) works.
**Stretch:** place prefill (long prompt, batch 1) and decode (1 token, batch 1) on your roofline using measured times.

**Acceptance:** correctness test green.

**Measure:** tokens/s of naive generation vs sequence length (it should degrade — that's next week's motivation).

**Exit questions:** LLM-01, LLM-02, LLM-03, LLM-04

---

## Week 18 — KV Cache

**Goal:** add a KV cache and make its memory cost second nature.

**Read:** any clear KV-cache explainer; re-read GQA motivation.

**Experiments:**
- [L] By hand, from the model's `config.json`: KV bytes per token = `layers × 2 × kv_heads × head_dim × bytes_per_element`. Then per sequence at 2k/8k/32k context, and how many sequences fit in your free GPU memory. Verify empirically with `torch.cuda.memory_allocated()`.
- Do the same calculation for a 7–8B model and a 70B model (paper exercise).

**Build:** `inferx/inferx/cache/kv_cache.py`
- [L] Contiguous per-sequence KV cache (preallocate to max length); prefill writes the prompt's K/V; decode appends one position per layer.
- [P] Update `llama.py` to take positions and a cache; keep the correctness test green.

**Minimum:** cached generation matches uncached generation token-for-token; memory math verified.
**Stretch:** show the waste: allocate for max length, generate short outputs, report % of reserved KV memory actually used.

**Acceptance:** correctness test green with cache enabled.

**Measure:** tokens/s with vs without cache vs sequence length; predicted vs measured KV memory.

**Exit questions:** LLM-05, LLM-06, LLM-07

---

## Week 19 — Batching and Scheduling

**Goal:** go from static batching to continuous batching and feel why it matters.

**Read:** the Orca paper (iteration-level scheduling, selective batching).

**Build:** `inferx/inferx/engine/`
- [L] Static batching first: pad a batch of prompts, run to completion of the longest. Measure.
- [L] `request.py`: request lifecycle states (`DESIGN.md §7.2`).
- [L] `scheduler.py`: continuous batching with `max_num_seqs` and `max_num_batched_tokens`, FCFS, admission while budget allows (`DESIGN.md §7.4`).
- [P] `engine.py`: the step loop (`DESIGN.md §7.3`); mixed prefill + decode steps.
- [P] `tests/test_scheduler.py`: budget limits respected; no starvation in FCFS; finished requests free resources.

**Minimum:** continuous batching engine (offline, no HTTP yet) that produces the same outputs as single-request generation.
**Stretch:** preemption by recompute when KV memory runs out.

**Acceptance:** batched greedy outputs equal unbatched greedy outputs for every request.

**Measure:** throughput and mean latency, static vs continuous, on a workload with highly variable output lengths.

**Exit questions:** LLM-08, LLM-09, LLM-10

---

## Week 20 — Serving + Benchmark Harness (InferX v0.1)

**Goal:** put the engine behind an API and build the benchmark harness used for the rest of the curriculum.

**Read:** vLLM's serving metrics definitions (TTFT, TPOT, ITL); skim CUDA graphs, speculative decoding (Leviathan et al.), and quantization overviews — conceptual only this week.

**Build:**
- [P] `server/api.py`, `server/streaming.py`: OpenAI-compatible `/v1/completions` with SSE streaming; engine thread ↔ asyncio hand-off; client disconnect → ABORTED.
- [L] Metrics collection in the engine (`DESIGN.md §7.7`) and `/metrics`.
- [S] `benchmarks/harness.py`: ShareGPT-style dataset loader, closed-loop (fixed concurrency) and open-loop (Poisson rate) modes, raw per-request results to JSON, percentile tables. **Design it to also target any OpenAI-compatible server** so it can benchmark vLLM and SGLang later.

**Minimum:** concurrent streaming requests work; harness produces TTFT/TPOT/throughput/percentiles.
**Stretch:** CUDA graphs for the decode step at a few fixed batch sizes.

**Acceptance:** 32 concurrent streaming clients complete correctly; killing a client mid-stream frees its resources.

**Measure (report `reports/m5-inferx-v0.1.md`):** TTFT, TPOT, tokens/s, p50/p95/p99 vs concurrency; KV memory utilization.

**Exit questions:** LLM-11, LLM-12, LLM-13, LLM-14

**Checkpoint:** run `/checkpoint 5`.

---

# Month 6 — Serving Systems: vLLM, PagedAttention, Triton

**Month goal:** study production serving systems and retrofit their key ideas into InferX.

## Week 21 — Read vLLM

**Goal:** understand vLLM's architecture and which bottleneck each component solves.

**Read:** the vLLM / PagedAttention paper (full). vLLM source at a **pinned commit**: entrypoint/API server, engine core, scheduler, KV cache manager / block pool, worker and model runner.

**Build:** `docs/weeks/week-21.md` + `docs/papers/vllm.md`
- [L] Architecture map: for each component, its file(s), its responsibility, and **the bottleneck it solves**. Trace one request end-to-end with file/line references.
- [L] Compare with InferX v0.1: a table of "vLLM does X, InferX does Y, consequence Z".
- [S] Run vLLM on your GPU with your reference model; benchmark it with **your** harness.

**Minimum:** architecture map + request trace + first vLLM numbers from your harness.
**Stretch:** trace tensor-parallel execution (how workers are launched and coordinated).

**Acceptance:** you can explain the scheduler's decision for one step from the code.

**Measure:** vLLM vs InferX v0.1 baseline (same model, GPU, dataset).

**Exit questions:** VLLM-01, VLLM-02

---

## Week 22 — PagedAttention → InferX v0.2

**Goal:** understand why naive KV allocation fragments memory, then fix it in your own engine.

**Read:** re-read the PagedAttention memory-management sections; OSTEP paging chapter (for the analogy).

```
Virtual memory:  virtual page  → page table  → physical frame
KV cache:        logical block → block table → physical KV block in GPU memory
```

**Build:** `inferx/inferx/cache/block_manager.py`
- [L] Block manager (`DESIGN.md §7.5`): preallocated block pool, free list, per-sequence block tables, `can_allocate / allocate / append_slot / free`.
- [P] Slot mapping in the model runner; attention via gather-to-contiguous + SDPA (`DESIGN.md §7.6`, v0.2).
- [L] Scheduler uses block availability for admission and triggers preemption.
- [P] `tests/test_block_manager.py`: no leaks after mixed workloads; allocations never exceed pool; free→reuse works.

**Minimum:** paged KV cache with correctness test green.
**Stretch:** sweep block size (8/16/32) and measure internal fragmentation vs overhead.

**Acceptance:** correctness test green; block pool fully free after every benchmark run.

**Measure:** max concurrent sequences and throughput, v0.1 (contiguous) vs v0.2 (paged); KV utilization.

**Exit questions:** VLLM-03, VLLM-04, VLLM-05

---

## Week 23 — Triton Attention → InferX v0.3

**Goal:** write the kernels that make paged attention fast.

**Read:** FlashAttention and FlashAttention-2 papers (tiling + online softmax; work partitioning). Triton's fused-attention tutorial.

**Build:** `inferx/inferx/kernels/`
- [L] `rmsnorm.py`: port your Week 12 Triton RMSNorm into InferX.
- [L] `paged_attention.py`: Triton **decode** attention kernel that reads K/V directly from blocks via the block table (one query token per sequence, GQA-aware).
- [S] Kernel microbenchmarks and correctness tests vs the gather + SDPA path.

**Minimum:** Triton RMSNorm integrated; paged decode kernel correct and integrated.
**Stretch:** autotune block sizes; split-K over long contexts (flash-decoding style).

**Acceptance:** kernel output matches gather+SDPA within tolerance for ragged sequence lengths and partial last blocks; end-to-end correctness test green.

**Measure:** decode-step latency and end-to-end tokens/s, v0.2 vs v0.3; kernel achieved bandwidth vs roofline.

**Exit questions:** KRN-04, KRN-05, KRN-06

**Checkpoint:** run `/checkpoint 6`.

---

## Buffer B3

Default use: polish v0.3; write a public blog post explaining PagedAttention through your own implementation.

---

# Capstone

## Week 24 — SGLang + Prefix Caching → InferX v0.4

**Goal:** study SGLang's design philosophy and add prefix caching to InferX.

**Read:** the SGLang paper (RadixAttention, structured generation, frontend language). Skim SGLang's scheduler and radix cache source at a pinned commit. Sarathi-Serve (chunked prefill) and DistServe or Splitwise (prefill/decode disaggregation) for context.

**Build:**
- [L] `cache/prefix_cache.py`: block hashing by (parent hash, tokens), refcounts, LRU eviction of unreferenced blocks (`DESIGN.md §7.5`).
- [L] Comparison doc `docs/papers/vllm-vs-sglang.md`: scheduling, memory management, prefix caching, batching, and design philosophy.
- [S] Benchmark workload with shared prefixes (e.g. a long common system prompt + varied questions).

**Minimum:** prefix caching works and shows a TTFT improvement on the shared-prefix workload; comparison doc.
**Stretch:** chunked prefill; simple structured generation (regex- or JSON-constrained decoding via logit masking).

**Acceptance:** correctness test green with prefix caching on; cache hit rate reported.

**Measure:** TTFT and throughput with/without prefix caching; cache hit rate.

**Exit questions:** VLLM-06, VLLM-07, VLLM-08

---

## Week 25 — InferX v1.0: Refactor and Harden

**Goal:** turn the evolving codebase into the clean architecture in `DESIGN.md §7`. **Refactor, not rebuild.**

**Build:**
- [P] Restructure into the final package layout; clear interfaces between server / engine / scheduler / cache / runner.
- [P] Configuration (`max_num_seqs`, `max_num_batched_tokens`, block size, KV memory fraction, prefix caching on/off).
- [S] README with architecture diagram, quickstart, and design decisions; CI running unit + correctness tests (CPU-safe subset).

**Required feature set (v1.0):** OpenAI-compatible `/v1/completions`, streaming, continuous batching, **paged KV cache**, prefix caching, request scheduler with preemption, sampling, metrics, configurable limits, concurrency.

**Stretch:** tensor parallelism (reuse Week 16 layers); speculative decoding; CUDA graphs for decode.

**Acceptance:** all tests green; a fresh clone runs the quickstart.

**Measure:** re-run the full benchmark suite to confirm no regression vs v0.4.

**Exit questions:** re-score any LLM-* and VLLM-* questions below 3.

---

## Week 26 — Benchmark, Attribute, Report

**Goal:** no new features. Produce the final comparison and explain it.

**Build:** `inferx/benchmarks/compare.py` + `reports/final-inferx-vs-vllm-vs-sglang.md`
- [S] Orchestration to run InferX, vLLM, and SGLang with identical model, revision, dtype, GPU, `max_model_len`, dataset, output lengths, concurrency levels, and request rates (`DESIGN.md §8.3`).
- [L] **Attribution ladder:** vLLM/SGLang with features disabled → progressively enabled (no CUDA graphs → + CUDA graphs → + prefix caching → defaults). Record exact flags and versions.
- [L] The report: results tables and plots, then the analysis — where the gap comes from (kernels, launch overhead, scheduling, memory management), and what you would do next.

**Measure:** TTFT, TPOT, tokens/s, requests/s, p50/p95/p99, GPU utilization, GPU memory, KV cache utilization — for all three systems and every ladder rung.

**Exit questions:** LLM-15 — and the final goal statement in `DESIGN.md §1`.

**Final checkpoint:** run `/checkpoint final` — re-score the entire question bank.
