# AI Infrastructure Self-Teaching Curriculum → InferX

A ~7-month self-directed curriculum (26 content weeks + 3 buffer weeks, 12–15 h/week) that goes from systems foundations to a hand-built LLM inference engine, **InferX**, benchmarked against vLLM and SGLang.

> **Goal:** be able to read an AI infrastructure system, understand where its compute / memory / communication bottlenecks come from, reproduce a simplified version, benchmark it, and make a principled optimization.

Beating vLLM is not the goal. The final deliverable is a report that explains, with numbers I produced, *where the gap comes from*.

## Roadmap

```text
Month 1  Systems foundations      C++ · Linux · Networking · Performance        → RPC server
Month 2  Distributed systems      Failure · Replication · Scheduling            → Distributed task queue
         ── buffer ──
Month 3  GPU + PyTorch            GPU arch · CUDA · PyTorch internals · Triton  → RMSNorm kernel study
Month 4  Distributed training     DDP · NCCL · FSDP · Tensor parallelism        → Scaling report
         ── buffer ──
Month 5  LLM inference            Model · KV cache · Continuous batching        → InferX v0.1
Month 6  Serving systems          vLLM · PagedAttention · Triton attention      → InferX v0.2 – v0.3
         ── buffer ──
Capstone                          SGLang + prefix caching · v1.0 · Benchmarks   → InferX v1.0 + final report
```

Each project feeds the next: the thread pool becomes a TCP server, then an RPC server; the scheduling library and task queue shape the InferX scheduler; the Triton kernels and tensor-parallel layers end up inside InferX.

## InferX (capstone)

A lightweight LLM inference engine for one Llama-architecture model family:

- OpenAI-compatible `/v1/completions` with SSE streaming
- Continuous batching with a token-budget scheduler and preemption
- Paged KV cache with a block manager, plus hashed-block prefix caching
- Triton RMSNorm and a paged-attention decode kernel
- Prometheus metrics (TTFT, TPOT, p50/p95/p99, KV utilization)
- Correctness gate: greedy output matches Hugging Face `transformers` token-for-token

See [`docs/DESIGN.md §7`](docs/DESIGN.md) for the architecture.

## How progress is measured

- Every week has **Explain / Build / Measure** exit criteria and a minimum vs. stretch outcome.
- A [question bank](docs/QUESTIONS.md) of 92 questions, each scored 0–3. A **3** requires an experiment or number I produced, and the count of 3s is the main progress metric.
- Month-end checkpoints re-score questions with spaced review and decide whether to use a buffer week.
- Benchmarks follow fixed rules: warmup, median of 3 runs, percentiles from raw per-request data, environment recorded.

Current status lives in [`PROGRESS.md`](PROGRESS.md).

## Repository layout

```text
├── CLAUDE.md            # Rules for Claude Code (pair programmer / reviewer / examiner)
├── PROGRESS.md          # Tracker: weekly status, question scores, benchmark log
├── docs/
│   ├── DESIGN.md        # Principles, collaboration model, InferX architecture, benchmark rules
│   ├── CURRICULUM.md    # Week-by-week specs
│   ├── QUESTIONS.md     # Question bank
│   ├── READING.md       # Scoped books and papers
│   ├── templates/       # Week notes, paper notes, perf report
│   ├── weeks/           # My notes per week
│   └── papers/          # Paper notes
├── reports/             # One performance report per project
├── m1-systems/          # Month 1 projects
├── m2-distributed/      # Month 2 projects
├── m3-gpu/              # Month 3 projects
├── m4-dist-training/    # Month 4 projects
└── inferx/              # Capstone engine (self-contained, extractable)
```

Project directories are created as each week starts.

## Working with Claude Code

Claude Code is a pair programmer, reviewer, and examiner, not a ghostwriter. Every build task is tagged:

| Tag | Who writes it |
| --- | --- |
| **[L]** Learner-owned | Me. Claude reviews, writes tests, and gives hints, but writes no implementation unless I say "show me" (logged). |
| **[P]** Pair | I write a first attempt or design; Claude helps finish, refactor, debug. |
| **[S]** Scaffold | Claude writes it (build files, harnesses, plotting, CI). |

Commands: `/start-week N` · `/review` · `/quiz <week | ID | month N>` · `/checkpoint <N | final>` · `/perf-report <project>`. See [`CLAUDE.md`](CLAUDE.md).

## License

[Apache 2.0](LICENSE)
