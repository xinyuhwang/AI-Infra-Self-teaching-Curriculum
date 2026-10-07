# <Project> — Performance Report

**Date:** · **Commit:** · **Week:**

## Question
What am I trying to learn from these measurements?

## Setup
| Item | Value |
|---|---|
| Hardware (CPU / GPU / memory) | |
| OS, driver, CUDA | |
| Library versions (PyTorch, Triton, ...) | |
| Model + revision + dtype (if applicable) | |
| Workload / dataset | |
| Load model (closed-loop concurrency / open-loop rate) | |

## Method
Warmup, number of runs (median of 3), how latency is measured, where raw data lives.

## Results
| Config | Throughput | p50 | p95 | p99 | CPU / GPU util | Memory |
|---|---|---|---|---|---|---|
| | | | | | | |

<plots>

## Bottleneck analysis
What limits performance, and the evidence (profile, flame graph, roofline position, counters).

## What I'd try next
Ranked by expected impact, with the reasoning.
