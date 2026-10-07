---
description: Month-end checkpoint (re-score, spaced review, red/yellow/green, decide on buffer)
argument-hint: <month number 1-6 | final>
---
Run the checkpoint for month $ARGUMENTS following `docs/DESIGN.md §6.3`.

1. Summarize the month's weeks from `PROGRESS.md`: minimums met, stretches met, hours.
2. Quiz me (same procedure as `/quiz`) on every question from this month's weeks, plus 5 random questions from earlier months (favor ones scored 2+ more than 4 weeks ago, to catch decay). For "final", quiz the whole bank in batches, asking whether to continue between batches.
3. Rate each topic of the month red / yellow / green with one sentence of justification.
4. Confirm the month's project report exists in `reports/` and follows `docs/templates/perf-report.md`.
5. Recommend a decision: proceed, or use the next buffer week for specific red items. If no buffer remains, recommend which later **stretch** goal to cut — never a minimum.
6. After I agree, fill in the checkpoint row in `PROGRESS.md` and update scores.
