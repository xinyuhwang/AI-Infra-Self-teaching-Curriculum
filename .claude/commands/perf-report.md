---
description: Draft a performance report from raw benchmark results
argument-hint: <project path, e.g. m1-systems/rpc or inferx>
---
Draft a performance report for `$ARGUMENTS` using `docs/templates/perf-report.md`.

- Use only numbers from raw results files in the project (e.g. `results/*.json`) or values I've logged. If something the template needs is missing, list it as a TODO; never estimate.
- Compute percentiles from raw per-request data.
- Fill in Setup (hardware, versions, commit) from the results metadata; ask me for anything missing.
- Leave the **Bottleneck analysis** and **What I'd try next** sections as prompts with the relevant evidence listed (profiles, plots) — I write the conclusions.
- Save to `reports/<name>.md` and show me the path.
