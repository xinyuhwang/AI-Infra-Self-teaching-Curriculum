# Reading List

Reading is capped at ~3 hours/week. Everything here is **scoped**: read the named chapters, not the whole book. Chapter numbers differ between editions, so chapters are named by title.

## Core library

| # | Book | Role | Read (by week) |
|---|---|---|---|
| 1 | *A Tour of C++* — Stroustrup | C++ foundation | W1: classes, essential operations, templates, resource management, concurrency |
| 2 | *C++ Primer* — Lippman et al. | Reference only | Look things up; never read linearly |
| 3 | *Operating Systems: Three Easy Pieces* — Arpaci-Dusseau (free: pages.cs.wisc.edu/~remzi/OSTEP/) | OS foundation | W2: Processes, Process API, Limited Direct Execution, Address Spaces, Address Translation, Paging, TLBs, Concurrency intro, Thread API · W1/W2 review: Condition Variables · W22: Paging (again, for the analogy) |
| 4 | *The Linux Programming Interface* — Kerrisk | Reference only | W2: fork/exec/wait, pipes, signals, mmap as needed |
| 5 | *Computer Networking: A Top-Down Approach* — Kurose & Ross | Networking | W3: transport layer (TCP reliability, flow control, congestion control), HTTP sections |
| 6 | *Beej's Guide to Network Programming* (free) | Practical sockets | W3: whole guide |
| 7 | *Systems Performance* (2nd ed.) — Gregg | Performance method | W4: Methodologies, CPUs, Memory, Benchmarking · later: Network as needed |
| 8 | *Designing Data-Intensive Applications* — Kleppmann | Distributed systems | W5: Reliable/Scalable/Maintainable, Encoding and Evolution, The Trouble with Distributed Systems · W6: Replication, Consistency and Consensus · Optional: Data Models, Storage and Retrieval |
| 9 | *Programming Massively Parallel Processors* — Hwu, Kirk, El Hajj | GPU programming (learning path) | W9: compute architecture & scheduling, memory architecture & data locality · W10: data-parallel computing, multidimensional grids, performance considerations, reduction |
| 10 | *CUDA C++ Programming Guide* — NVIDIA | Reference only | W10+: programming model, streams/events, memory spaces |

## Other technical sources

| Week | Source |
|---|---|
| W4 | Brendan Gregg's flame graph page and USE method page |
| W6 | Raft visualization at raft.github.io; Python asyncio docs and `asyncio/base_events.py` |
| W7 | Any short treatment of Little's Law and M/M/1 queues; articles on load shedding and backpressure |
| W9 | NVIDIA architecture whitepaper for **your** GPU; a roofline-model explainer |
| W11 | Edward Yang's "PyTorch internals" blog post; PyTorch docs: CUDA semantics, dispatcher, `torch.library` |
| W12 | PyTorch Profiler tutorial; Nsight Systems / Nsight Compute docs; Triton tutorials (vector add, fused softmax, layer norm) |
| W13–14 | PyTorch Distributed overview and DDP tutorial; NCCL docs; `NVIDIA/nccl-tests` |
| W17 | Hugging Face `modeling_llama.py`; gpt-fast's `model.py`; nanoGPT for a minimal training-side reference |
| W20 | vLLM docs on serving metrics and benchmarking |
| W21 | vLLM source at a pinned commit |
| W23 | Triton fused-attention tutorial |
| W24 | SGLang source at a pinned commit |

## Papers

Roughly one every 1–2 weeks. Notes go in `docs/papers/<name>.md` using [`templates/paper-notes.md`](templates/paper-notes.md).

| Week | Paper | Why it's here |
|---|---|---|
| W6 | *In Search of an Understandable Consensus Algorithm* (Raft) | Leader election, log replication |
| W8 | *MapReduce: Simplified Data Processing on Large Clusters* | Coordinator/worker, failure handling, stragglers |
| W13 | *PyTorch Distributed: Experiences on Accelerating Data Parallel Training* | DDP bucketing and overlap |
| W15 | *ZeRO: Memory Optimizations Toward Training Trillion Parameter Models* | Optimizer/gradient/parameter sharding |
| W15 | *PyTorch FSDP: Experiences on Scaling Fully Sharded Data Parallel* | ZeRO-3 in PyTorch |
| W15 | *GPipe* | Pipeline parallelism and bubbles |
| W15 (skim) | *ZeRO-Infinity* | Offloading to CPU/NVMe |
| W16 | *Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism* | Tensor parallelism |
| W19 | *Orca: A Distributed Serving System for Transformer-Based Generative Models* | Iteration-level (continuous) batching |
| W20 (skim) | *Fast Inference from Transformers via Speculative Decoding* (Leviathan et al.) | Speculative decoding |
| W21–22 | *Efficient Memory Management for Large Language Model Serving with PagedAttention* (vLLM) | Paged KV cache |
| W23 | *FlashAttention* | Tiling + online softmax, IO-awareness |
| W23 | *FlashAttention-2* | Better work partitioning |
| W24 | *SGLang: Efficient Execution of Structured Language Model Programs* | RadixAttention, structured generation |
| W24 | *Sarathi-Serve* (Taming Throughput-Latency Tradeoff in LLM Inference) | Chunked prefill |
| W24 | *DistServe* or *Splitwise* | Prefill/decode disaggregation |

## How to read a paper

Answer these four questions in the notes. Memorization is not the goal.

1. What bottleneck existed before this system?
2. What was the proposed solution?
3. What tradeoff did they make?
4. Which resource are they optimizing: compute, memory, communication, latency, or cost?

Add a fifth for anything you later implement: **what did I find that the paper didn't tell me?**
