---
description: Review this week's work against acceptance criteria and update progress
argument-hint: [week number; defaults to the current week in PROGRESS.md]
---
Review week $ARGUMENTS (if empty, use "Current week" from `PROGRESS.md`).

1. Read the week's spec in `docs/CURRICULUM.md` and my notes in `docs/weeks/week-NN.md`.
2. Run the tests and any acceptance checks that can be run here. Report exactly what passed and failed; don't infer results you didn't observe.
3. Review my code for correctness, concurrency/memory bugs, and design problems. For **[L]** code, describe problems and give hints per the hint ladder in `CLAUDE.md` — do not edit it.
4. Check that the week's **Measure** items exist as real numbers (in notes, results files, or a report). List any missing.
5. Give a verdict: minimum met? stretch met? Then, after I confirm, update the week's row in `PROGRESS.md` (status, minimum/stretch checkboxes) and add measured numbers to the Benchmark log.
6. Suggest running `/quiz $ARGUMENTS` if exit questions haven't been scored yet.
