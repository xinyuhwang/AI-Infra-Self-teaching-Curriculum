# AI Infrastructure Self-Learning Curriculum — Design Doc

**Status:** Living document · **Owner:** me · **Collaborator:** Claude Code
**Related docs:** [`CURRICULUM.md`](CURRICULUM.md) (week-by-week specs) · [`QUESTIONS.md`](QUESTIONS.md) (question bank) · [`READING.md`](READING.md) (books & papers) · [`../PROGRESS.md`](../PROGRESS.md) (tracker) · [`../CLAUDE.md`](../CLAUDE.md) (instructions for Claude Code)

---

## 1. Goal

> I can read an AI infrastructure system, understand where its compute / memory / communication bottlenecks come from, reproduce a simplified version, benchmark it, and make a principled optimization.

The concrete end state is **InferX**, a lightweight LLM inference engine with continuous batching, a paged KV cache, prefix caching, Triton kernels, and a benchmark report that explains *why* it is slower or faster than vLLM and SGLang on the same workload.

"I know vLLM" is not the goal. The goal is being able to *demonstrate*, with numbers I produced, why systems like vLLM are built the way they are.

### Non-goals

- Competitive-programming C++ or exhaustive language coverage.
- Production Raft, production Kubernetes, or cluster operations.
- Training large models. Training appears only as far as it explains communication, memory, and parallelism.
- Beating vLLM/SGLang on throughput. Explaining the gap is the deliverable.

---

## 2. Design principles

1. **Every week contains learning and implementation.** No week is reading-only.
2. **One compounding codebase.** Projects build on each other instead of being thrown away (§5). The scheduler ideas from Week 7 reappear in Week 19; the server from Week 20 becomes InferX.
3. **Measure before optimizing; measure after optimizing.** Every project produces numbers, recorded in a standard format (§8).
4. **Correctness before performance.** Every kernel and every engine change is checked against a reference implementation before it is benchmarked.
5. **Progress is defined by what I can demonstrate, not what I've read.** Topic coverage is not progress (§6).
6. **Minimum vs. stretch.** Each week has a minimum outcome that fits the time budget and an optional stretch outcome. Missing a stretch goal is normal; missing a minimum goal triggers a buffer week.
7. **I write the code that teaches the concept; Claude Code writes the scaffolding** (§4).

### The core loop

```
Learn → Implement → Measure → Find bottleneck → Read source/paper → Optimize → Measure again
```

This loop matters more than any individual book.

---

## 3. Constraints and assumptions

### Time budget

**12–15 hours/week**, split roughly as:

| Block | Hours | Activity |
|---|---|---|
| Reading / lectures | 3 | Scoped chapters and papers only (see `READING.md`) |
| Experiments | 3 | Small, throwaway measurements (strace, profilers, microbenchmarks) |
| Project | 5–7 | The week's build |
| Review | 1 | Notes, benchmark log, question-bank scoring |

26 content weeks + 3 buffer weeks ≈ **29 weeks (~7 months) ≈ 380 hours**. That budget is the reason reading assignments are scoped to specific chapters rather than whole books.

### Hardware

| Phase | Requirement | Fallback |
|---|---|---|
| Months 1–2 | Any Linux machine (native, VM, or WSL2; `perf` works best on native Linux) | Cloud VM |
| Months 3, 5, 6 | One NVIDIA GPU. Ampere or newer recommended (bf16, modern Triton features). ≥16 GB VRAM is comfortable for 0.5B–1B models. | Cloud GPU instance |
| Month 4 + TP stretch in Week 25 | 2–4 GPUs on one node with NCCL | Learn the APIs on CPU with the `gloo` backend; rent a multi-GPU node for a few *planned* days of measurement |

Schedule rental days deliberately: arrive with scripts written and tested on CPU/1 GPU so paid time is spent measuring, not debugging.

### Reference model for Months 5–6

Pick **one** small Llama-architecture model (RMSNorm, RoPE, SwiGLU, GQA), e.g. a ~0.5B–1B Qwen or Llama checkpoint, and pin the exact Hugging Face revision in `inferx/config.py`. Supporting one model family well is the point; a generic model loader is out of scope.

---

## 4. Collaboration model with Claude Code

Claude Code can write all of this. That would defeat the purpose. Each build task in `CURRICULUM.md` is tagged with an **ownership level**:

| Tag | Meaning | What Claude Code may do |
|---|---|---|
| **[L] Learner-owned** | The concept being learned *is* this code (e.g. the thread pool's queue + condition variable, the KV block manager, the scheduler policy, the Triton kernel). | Review, ask Socratic questions, write tests and benchmarks for it, point to docs. **No implementation code** unless I explicitly say "show me" — and then it gets logged in the week notes. |
| **[P] Pair** | I design and write a first attempt; Claude helps finish, refactor, or debug. | Write code *after* my attempt exists or after I've written the design in the week notes. |
| **[S] Scaffold** | Supporting infrastructure that isn't the lesson (CMake, Dockerfiles, load generators, plotting, CLI parsing, CI, report formatting). | Write it fully. |

**Hint ladder** for [L] tasks when I'm stuck — Claude escalates one step at a time and only when asked:

1. A question that points at the issue ("What happens to waiting workers when `stop_` becomes true?").
2. The relevant concept or doc section.
3. Pseudocode or a diagram.
4. Actual code — only on explicit request, and logged.

Claude Code also acts as **examiner**: `/quiz` draws from `QUESTIONS.md` and scores my answers against the rubric in §6. See [`../CLAUDE.md`](../CLAUDE.md) and `.claude/commands/`.

---

## 5. Repository structure and codebase evolution

```
ai-infra-curriculum/
├── CLAUDE.md                  # Instructions for Claude Code
├── README.md
├── PROGRESS.md                # The tracker (§6)
├── docs/
│   ├── DESIGN.md              # This file
│   ├── CURRICULUM.md          # Week-by-week specs
│   ├── QUESTIONS.md           # Question bank with IDs
│   ├── READING.md             # Books, chapters, papers
│   ├── templates/             # Week notes, paper notes, perf report
│   ├── weeks/week-NN.md       # My notes per week (created by /start-week)
│   └── papers/<paper>.md      # Paper notes
├── reports/                   # Performance reports (one per project)
│
├── m1-systems/
│   ├── threadpool/            # W1  C++ thread pool
│   ├── syscall-labs/          # W2  strace/perf experiments
│   ├── tcp-server/            # W3  thread-pool TCP server + epoll variant
│   └── rpc/                   # W4  Mini RPC server (M1 project)
├── m2-distributed/
│   ├── kv-replication/        # W6  3-node replicated service (stretch)
│   ├── sched/                 # W7  scheduling/backpressure library
│   └── taskqueue/             # W8  Distributed task queue (M2 project)
├── m3-gpu/
│   ├── cuda/                  # W10 CUDA kernels
│   ├── pytorch-tracing/       # W11 operator traces & notes
│   └── kernels/               # W12 RMSNorm/Softmax: PyTorch vs CUDA vs Triton
├── m4-dist-training/          # W13–16 DDP → FSDP → TP, scaling study
│
└── inferx/                    # W17 → W26: grows into the capstone
    ├── pyproject.toml
    ├── inferx/
    │   ├── config.py
    │   ├── server/   (api.py, streaming.py)
    │   ├── engine/   (engine.py, scheduler.py, request.py, batch.py)
    │   ├── cache/    (kv_cache.py, block_manager.py, prefix_cache.py)
    │   ├── model/    (llama.py, loader.py, runner.py, sampling.py)
    │   ├── kernels/  (rmsnorm.py, attention.py, paged_attention.py)
    │   └── distributed/ (tensor_parallel.py)       # stretch
    ├── benchmarks/   (harness.py, datasets.py, compare.py, results/)
    └── tests/        (test_correctness.py, test_block_manager.py, test_scheduler.py, ...)
```

`inferx/` is self-contained (own `pyproject.toml`, own tests) so it can later be extracted into its own repository with `git subtree split` or `git filter-repo` while keeping history.

### How the codebase compounds

```
W1 thread pool ──► W3 TCP server ──► W4 RPC server (request IDs, timeouts, metrics, p99)
                                            │  same shape: queue → scheduler → workers
W7 sched library ──► W8 task queue ─────────┤  (heartbeats, retries, failure detection)
                                            ▼
W12 Triton kernels ─────────────► W17 model ► W18 KV cache ► W19 continuous batching
W16 TP study ─────────────┐                                        │
                          │       W20 server + benchmark harness ◄─┘   = InferX v0.1
                          │                     │
                          │       W22 paged KV cache retrofit           = InferX v0.2
                          │       W23 Triton RMSNorm + paged attention  = InferX v0.3
                          │       W24 prefix caching                    = InferX v0.4
                          └─────► W25 refactor, hardening, (TP stretch) = InferX v1.0
                                  W26 benchmark & attribution report
```

Code isn't always imported across stages (the W7 scheduler is request-based; the W19 scheduler is token-budget-based), but the **design** and the **benchmark discipline** carry forward. Each stage's README names what it inherits.

---

## 6. Progress-tracking system

Three layers, from weekly to monthly.

### 6.1 Weekly exit criteria

Every week in `CURRICULUM.md` has three kinds of checks:

- **Explain** — exit questions (IDs into `QUESTIONS.md`) I can answer without notes.
- **Build** — an artifact with acceptance criteria (tests that pass, behaviors that work).
- **Measure** — at least one number I produced, logged in `PROGRESS.md`.

A week is **done** when its *minimum* outcome is met. Stretch outcomes are tracked separately.

### 6.2 Question bank scoring (0–3)

| Score | Meaning |
|---|---|
| 0 | Can't answer |
| 1 | Can answer with notes |
| 2 | Can explain cold, including follow-up "why?" questions |
| 3 | Demonstrated with an experiment or number I produced (the "Demonstration" column in `QUESTIONS.md`) |

Scores are recorded in `PROGRESS.md`. **The count of 3s is the primary progress metric.** Questions that decay from 2 → 1 at a checkpoint are scheduled for review.

### 6.3 Month-end checkpoints

At the end of each month (run `/checkpoint N`):

1. Re-score every question tagged with that month **and a random sample of 5 from earlier months** (spaced retrieval).
2. Rate each topic red / yellow / green.
3. Confirm the month's project report exists in `reports/`.
4. Red topics or unmet minimums → use the next buffer week. If no buffer remains, cut a stretch goal later, never a minimum.

### 6.4 Buffer weeks

| Buffer | Placement | Default use if nothing is red |
|---|---|---|
| B1 | After Week 8 | Raft lab stretch, or go deeper on asyncio |
| B2 | After Week 16 | Rent multi-GPU time; finish the scaling report |
| B3 | After Week 23 | Polish InferX v0.3; write a blog post on PagedAttention |

---

## 7. InferX architecture (capstone design)

This section is the target design for InferX v1.0. Earlier versions implement subsets (see §5).

### 7.1 Components

```
Client
  │  HTTP (OpenAI-compatible /v1/completions, SSE streaming)
  ▼
Server (asyncio, FastAPI)        server/api.py, server/streaming.py
  │  enqueue Request; await per-request token stream
  ▼
Engine (step loop, own thread)   engine/engine.py
  │
  ├── Scheduler                  engine/scheduler.py   — which sequences run this step
  ├── Request Manager            engine/request.py     — lifecycle, sampling params, outputs
  ├── KV Cache Manager           cache/block_manager.py, cache/prefix_cache.py
  └── Model Runner               model/runner.py       — builds batch tensors, runs forward, samples
          │
          ▼
      PyTorch model (model/llama.py) ── kernels/ (Triton RMSNorm, paged attention)
```

### 7.2 Request lifecycle

```
WAITING ──(admitted, blocks allocated)──► RUNNING ──(EOS / max_tokens / stop)──► FINISHED
   ▲                                         │
   └────────────(preempted: free blocks, ────┘          any state ──(client disconnect)──► ABORTED
                 recompute later)
```

A `Request` holds: id, prompt token ids, output token ids, sampling params, status, block table, timestamps (arrival, first scheduled, first token, each token, finish) for metrics.

### 7.3 Engine step loop

```python
while running:
    new = drain(incoming_queue)                     # from the server
    scheduler.add(new)
    plan = scheduler.schedule()                     # prefill list, decode list, preempted list
    if plan.empty(): wait_for_work(); continue
    batch = runner.prepare(plan, block_manager)     # token ids, positions, block tables, slot mapping
    logits = runner.forward(batch)
    next_tokens = sampler.sample(logits, plan.sampling_params)
    for req, tok in zip(plan.sequences, next_tokens):
        req.append(tok); emit(req.id, tok)          # thread-safe hand-off to the asyncio loop
        if req.finished(): block_manager.free(req); scheduler.finish(req)
```

The engine runs in a dedicated thread; tokens are handed to per-request `asyncio.Queue`s via `loop.call_soon_threadsafe`. (A separate engine process is a possible v1.x change; note the GIL implications in the W25 notes.)

### 7.4 Scheduler policy (v1)

- **Limits:** `max_num_seqs` (sequences per step), `max_num_batched_tokens` (token budget per step), and KV blocks available.
- **Order:** FCFS. Running decodes are scheduled first (each costs 1 token of budget); waiting prefills are admitted while budget and blocks allow.
- **Preemption:** when a running sequence can't get a new block, preempt the most recently admitted sequence by **recompute** (free its blocks, push it back to WAITING).
- **Stretch:** chunked prefill (split long prompts across steps; see Sarathi-Serve), priority classes.

### 7.5 KV cache manager

- GPU memory is preallocated as `num_blocks × block_size` slots per layer for K and V.
- Each sequence has a **block table** (logical block → physical block), like a page table.
- Interface: `can_allocate(req)`, `allocate(req)`, `can_append(req)`, `append_slot(req) -> slot`, `free(req)`, `utilization()`.
- **Prefix caching (v0.4):** full blocks are hashed by (parent hash, token ids); blocks are reference-counted; freed blocks with refcount 0 go to an LRU evictable pool instead of being discarded.

### 7.6 Model runner and attention strategy

The attention path evolves deliberately:

| Version | Attention implementation | Why |
|---|---|---|
| v0.1 (W18–20) | Contiguous per-sequence KV tensors; padded batch + `scaled_dot_product_attention` | Simple, correct, shows the fragmentation problem |
| v0.2 (W22) | Paged KV; gather blocks into contiguous buffers, then SDPA | Correct paged memory management, slow attention |
| v0.3 (W23) | Triton paged-attention **decode** kernel reading directly from blocks; prefill stays SDPA | Removes the gather; the main kernel lesson |

Sampling supports greedy, temperature, top-k, top-p, and a per-request seed.

### 7.7 Metrics

Exposed at `/metrics` (Prometheus text format) and written per benchmark run:
TTFT, TPOT, inter-token latency, end-to-end latency (p50/p95/p99), queue time, requests/s, output tokens/s, running/waiting sequence counts, KV block utilization, preemption count, GPU memory.

### 7.8 Correctness contract

`tests/test_correctness.py`: for a fixed set of prompts, greedy decoding through InferX must match the Hugging Face `transformers` reference **token-for-token** for N tokens (same dtype; use fp32 if bf16 nondeterminism causes divergence, and document it). Every performance change must keep this test green.

---

## 8. Benchmarking and reporting conventions

### 8.1 Rules

- **Record the environment** with every result: GPU, driver, CUDA, PyTorch, Triton, model + revision, dtype, git commit.
- **Warm up** before measuring; run each configuration **3×** and report the median run.
- **Percentiles come from raw per-request latencies**, never from averages of averages. Store raw data as JSON in `benchmarks/results/`.
- **Know which load model you're using.** Closed-loop (fixed concurrency) is fine for throughput curves; open-loop (Poisson arrivals at a fixed rate) is needed for honest tail latency, because closed-loop clients slow down when the server does (coordinated omission).
- **GPU timing uses CUDA events or a profiler**, never `time.time()` around asynchronous calls without synchronization.

### 8.2 Report template

Every project ends with a report in `reports/` using [`templates/perf-report.md`](templates/perf-report.md): question, setup, method, results table, bottleneck analysis, what I'd try next.

### 8.3 Final comparison (Week 26)

Same model, revision, dtype, GPU, `max_model_len`, prompt dataset (ShareGPT-style variable lengths), output lengths, and request rates for InferX, vLLM, and SGLang. Then an **attribution ladder**: run vLLM/SGLang with features progressively enabled (eager mode / no CUDA graphs → + CUDA graphs → + prefix caching → full defaults) to explain where the gap to InferX comes from. Exact flag names change between versions; check `--help` for the installed version and record the flags used.

---

## 9. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Falling behind (most likely) | Minimum/stretch split; buffer weeks; cut stretch, never minimums |
| Claude Code writes the learning for me | Ownership tags + hint ladder (§4); "show me" events logged |
| Reading expands to fill all time | Reading is scoped by chapter in `READING.md`; 3h cap |
| GPU cost (Month 4) | CPU `gloo` development first; planned rental days |
| Engine bugs hidden by fast numbers | Correctness test gates every benchmark |
| Library/flag drift (vLLM, SGLang, Triton move fast) | Pin versions per project; record them in every report |

---

## 10. Change log

| Date | Change |
|---|---|
| (start date) | v1 of the design |
