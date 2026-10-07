# Question Bank

Every question has an ID (referenced from `CURRICULUM.md` and `PROGRESS.md`), the week where it's taught, and a **Demonstration**: the experiment or number that earns a score of 3.

**Scoring** (see `DESIGN.md §6.2`): 0 = can't answer · 1 = with notes · 2 = cold, including follow-up "why?" · 3 = demonstrated with my own experiment or number.

Rules for `/quiz`: Claude asks the question, then at least one follow-up "why?" or "what would change if…?" before scoring. A 3 requires pointing at the artifact (file, plot, report, or logged number).

---

## Systems (SYS) — Weeks 1, 2, 4

| ID | Wk | Question | Demonstration (for a 3) |
|---|---|---|---|
| SYS-01 | 1 | What's the difference between stack and heap allocation, and when does each happen in C++? | Show object addresses from both; show a dangling pointer caught by ASan |
| SYS-02 | 1 | What is RAII, and how do `unique_ptr`/`shared_ptr` express ownership? | Thread pool code where every resource is released by a destructor |
| SYS-03 | 1 | What does move semantics buy you, and why does `noexcept` on a move constructor matter for `std::vector`? | Logging-class experiment output with and without `noexcept` |
| SYS-04 | 1 | Why must a condition-variable wait be in a loop with a predicate? | Thread pool test that would fail with an `if` instead of `while` |
| SYS-05 | 1 | What happens between compiling and linking? What causes "undefined reference"? | The deliberately broken CMake build and its fix |
| SYS-06 | 2 | What's the difference between a process and a thread? | Context-switch measurement: process vs thread |
| SYS-07 | 2 | How do virtual memory, page tables, and the TLB turn a virtual address into a physical one? | `/proc/<pid>/maps` walkthrough or copy-on-write RSS experiment |
| SYS-08 | 2 | Why does a context switch cost time — direct and indirect costs? | Measured µs per switch |
| SYS-09 | 2 | What happens, step by step, during a system call? | Annotated strace trace |
| SYS-10 | 2 | What do `fork` and `exec` do, and how does copy-on-write make `fork` cheap? | Fork-a-1GB-process experiment |
| SYS-11 | 2 | What system calls does a Python HTTP server make when handling one request? | Annotated `strace -f` output in week notes |
| SYS-12 | 4 | What is the USE method, and how would you apply it to a slow server? | M1 report's bottleneck section |
| SYS-13 | 4 | How do you detect lock contention? | Thread pool tasks/sec vs workers + perf/flame graph evidence |
| SYS-14 | 4 | How do you read a flame graph? What do width and height mean? | Flame graph of your RPC server with annotated hot path |
| SYS-15 | 4 | What is Little's Law, and how does it relate throughput, latency, and concurrency? | Check it against your RPC server's measured numbers |

## Networking (NET) — Week 3

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| NET-01 | 3 | What does a TCP connection setup and teardown cost, and why does connection pooling help? | Pooled vs new-connection latency measurement |
| NET-02 | 3 | What's the difference between TCP flow control and congestion control? | `tc netem` loss/latency experiment |
| NET-03 | 3 | What does `epoll` actually do, and how is it different from one thread per connection? | epoll server vs thread-pool server comparison |
| NET-04 | 3 | What is backpressure, and where does TCP provide it for free? | Slow-reader experiment: watch the sender block |
| NET-05 | 3 | What does HTTP/2 multiplexing change vs HTTP/1.1, and why does gRPC use it? | Short written comparison + gRPC port from W5 |
| NET-06 | 3 | Why do we report p99 rather than mean latency? | Latency distribution plot from your load generator |

## Distributed systems (DS) — Weeks 5–8

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| DS-01 | 5 | Why do distributed systems need timeouts, and why is choosing them hard? | Fault-injection results with and without timeouts |
| DS-02 | 5 | Why isn't retrying always safe? How do idempotency keys fix it? | Counter ends at exactly 1,000 under 20% drops |
| DS-03 | 5 | What is partial failure, and why can't a client tell "slow" from "dead"? | Fault-injection proxy demo |
| DS-04 | 5 | What do serialization formats trade off, and what is schema evolution? | JSON vs protobuf size comparison |
| DS-05 | 6 | What is a quorum, and why does a majority guarantee overlap? | Written argument + replication lab (stretch) |
| DS-06 | 6 | How does Raft elect a leader, and what prevents two leaders in one term? | Week notes walkthrough or replication lab |
| DS-07 | 6 | When is a Raft log entry committed? | Week notes walkthrough |
| DS-08 | 7 | What's the difference between backpressure, admission control, and rate limiting? | `sched` library under 2× overload |
| DS-09 | 7 | Why does latency explode as utilization approaches 100%? | M/M/1 simulation plot |
| DS-10 | 7 | What does work stealing solve? When does SJF beat FIFO, and what does it cost? | Policy comparison plot |
| DS-11 | 8 | How do heartbeats detect failure, and what causes false positives? | Detection time vs heartbeat interval measurement |
| DS-12 | 8 | Why is your task queue at-least-once rather than exactly-once? | Duplicate-execution rate under worker kills |

## Python runtime (PY) — Week 6

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| PY-01 | 6 | What happens in the event loop when a coroutine hits `await` on a socket? | Your toy event loop |
| PY-02 | 6 | What is the GIL, and when do threads still help in Python? | Threads vs processes vs asyncio timings |

## GPU (GPU) — Weeks 9–10

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| GPU-01 | 9 | What is a warp, and why does divergence within a warp cost time? | Divergent vs non-divergent kernel timing |
| GPU-02 | 9 | Describe the GPU memory hierarchy with rough sizes and bandwidths for your GPU. | Table in week notes from spec + measurement |
| GPU-03 | 9 | Why is memory bandwidth so often the limit? | Measured achieved bandwidth vs peak |
| GPU-04 | 9 | Is a given operation compute-bound or memory-bound? How do you tell? | Your roofline plot |
| GPU-05 | 9 | What is occupancy, and why isn't maximum occupancy always best? | Nsight Compute output for one kernel |
| GPU-06 | 10 | What is memory coalescing, and why does it matter? | Coalesced vs strided access timing |
| GPU-07 | 10 | What is a CUDA stream, and how do you overlap copy and compute? | Nsight Systems trace showing overlap |
| GPU-08 | 10 | How does shared-memory tiling speed up matmul? | Naive vs tiled matmul GFLOP/s |
| GPU-09 | 10 | How do you write a fast reduction? | Naive vs warp-shuffle reduction timing |

## PyTorch (PT) — Weeks 11–12

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| PT-01 | 11 | What happens when you call `.cuda()` on a tensor? | Trace + profiler output |
| PT-02 | 11 | What are strides, views, and contiguous tensors? When does `reshape` copy? | Stride / `data_ptr` experiment |
| PT-03 | 11 | How does the dispatcher route `torch.add` to a CUDA kernel? | Your operator trace with file/line links |
| PT-04 | 11 | When does PyTorch synchronize CPU and GPU? | `.item()`-in-a-loop timing; sync debug mode output |
| PT-05 | 11 | What does the CUDA caching allocator do? Why do `memory_allocated` and `memory_reserved` differ? | Memory snapshot experiment |
| PT-06 | 12 | What is kernel launch overhead, and when does it dominate? | Profiler trace with GPU idle gaps |

## Kernels (KRN) — Weeks 12, 23

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| KRN-01 | 12 | What is Triton's programming model (programs, blocks, pointers, masks)? | Your Triton RMSNorm/softmax |
| KRN-02 | 12 | Why does a fused kernel beat a sequence of PyTorch ops? | Bytes-moved calculation + benchmark |
| KRN-03 | 12 | Why does Triton need masking, and what goes wrong without it? | Test on non-power-of-two shapes |
| KRN-04 | 23 | How does FlashAttention avoid materializing the attention matrix (tiling + online softmax)? | Written derivation + your paged decode kernel |
| KRN-05 | 23 | Why does your Triton kernel outperform (or not) the PyTorch implementation? | Kernel benchmark vs roofline |
| KRN-06 | 23 | What does autotuning search over, and why do the best configs differ by shape? | Autotune results table |

## Distributed training (DT) — Weeks 13–16

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| DT-01 | 13 | Why does DDP need all-reduce? | Manual DP matching single-process loss |
| DT-02 | 13 | What do gradient bucketing and comm/compute overlap buy? | Step-time breakdown, manual vs DDP |
| DT-03 | 13 | What are rank, world size, and a process group? | Launch scripts + manual DP code |
| DT-04 | 14 | How does ring all-reduce work, and what does it cost in bandwidth terms? | Your ring all-reduce + bandwidth plot |
| DT-05 | 14 | Why is all-reduce = reduce-scatter + all-gather, and why does that matter for FSDP? | Ring implementation phases |
| DT-06 | 14 | Why does communication become the bottleneck as you scale? | Compute vs comm time measurement |
| DT-07 | 15 | Model memory = params + grads + optimizer states + activations. Compute it for your model. | Prediction vs measurement table |
| DT-08 | 15 | What problem does FSDP/ZeRO solve, and what's the difference between its stages? | DDP vs FSDP memory comparison |
| DT-09 | 15 | What does activation checkpointing trade? | Memory saved vs time added |
| DT-10 | 15 | What is a pipeline bubble, and how do micro-batches shrink it? | Toy pipeline measurement (stretch) or calculation |
| DT-11 | 16 | How do column-parallel and row-parallel linear layers split an MLP? Which collective is needed where? | TP block matching reference |
| DT-12 | 16 | What's the difference between tensor and pipeline parallelism, and when do you use each? | Written comparison citing your measurements |
| DT-13 | 16 | Why doesn't distributed training scale linearly? | `reports/m4-scaling.md` |

## LLM inference (LLM) — Weeks 17–20, 26

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| LLM-01 | 17 | Why are prefill and decode different workloads? | Prefill vs decode on your roofline |
| LLM-02 | 17 | What are MQA and GQA, and what do they save? | KV-per-token calc for MHA vs GQA configs |
| LLM-03 | 17 | Why is batch-1 decode memory-bound? | Decode step time vs bytes of weights read |
| LLM-04 | 17 | How do temperature, top-k, and top-p change sampling? | `sampling.py` tests |
| LLM-05 | 18 | Derive KV cache size per token and per sequence for your model. | Predicted vs measured memory |
| LLM-06 | 18 | Why does the KV cache consume so much memory at scale? | 8B/70B paper calculation |
| LLM-07 | 18 | What does the KV cache trade, and what does naive allocation waste? | Utilization % of max-length allocation |
| LLM-08 | 19 | Why is continuous batching better than static batching? | Static vs continuous benchmark |
| LLM-09 | 19 | Why can a larger batch improve throughput but hurt latency? | Throughput/latency vs `max_num_seqs` sweep |
| LLM-10 | 19 | Preemption: recompute vs swap — what does each cost? | Preemption counts + latency impact |
| LLM-11 | 20 | Define TTFT and TPOT. What mainly drives each? | v0.1 report |
| LLM-12 | 20 | What problem do CUDA graphs solve in decode? | Decode step time with/without (stretch) or profiler gaps |
| LLM-13 | 20 | How does speculative decoding speed up generation without changing the output distribution? | Written explanation; implementation is a W25 stretch |
| LLM-14 | 20 | Why does weight quantization speed up decode more than prefill? | Roofline argument with your numbers |
| LLM-15 | 26 | Where does the performance gap between InferX and vLLM/SGLang come from? | Final report attribution ladder |

## Serving systems (VLLM) — Weeks 21–24

| ID | Wk | Question | Demonstration |
|---|---|---|---|
| VLLM-01 | 21 | How does vLLM's scheduler decide which requests enter the next step? | Code trace with file/line refs |
| VLLM-02 | 21 | Name vLLM's main components and the bottleneck each solves. | Architecture map |
| VLLM-03 | 22 | What problem does PagedAttention solve? | v0.1 vs v0.2 max concurrent sequences |
| VLLM-04 | 22 | How does a KV block manager work? | Your `block_manager.py` + tests |
| VLLM-05 | 22 | What does block size trade off? | Block-size sweep (stretch) or reasoning with numbers |
| VLLM-06 | 24 | How does prefix caching (RadixAttention / hashed blocks) work, and when does it help? | TTFT with/without on shared-prefix workload |
| VLLM-07 | 24 | How do vLLM and SGLang differ in design philosophy? | `vllm-vs-sglang.md` |
| VLLM-08 | 24 | What do chunked prefill and prefill/decode disaggregation solve? | Written comparison; chunked prefill if built |
