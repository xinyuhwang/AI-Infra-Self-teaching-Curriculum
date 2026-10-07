---
description: Start a curriculum week (scaffold notes, set up [S] tasks, list my [L] tasks)
argument-hint: <week number, e.g. 3>
---
Start week $ARGUMENTS of the curriculum.

1. Read the "Week $ARGUMENTS" section of `docs/CURRICULUM.md`, plus `CLAUDE.md` ownership rules.
2. Check `PROGRESS.md`. If the previous week's minimum is not met (status is not ✓ or ★), tell me and ask whether to continue or finish it first. Do not proceed until I answer.
3. Create `docs/weeks/week-NN.md` (zero-padded) from `docs/templates/week-notes.md`, filled in with the week's goal, reading, experiments, minimum/stretch, acceptance criteria, measurements, and exit question IDs.
4. Create the project directory named in the Build section if it doesn't exist.
5. Implement only the **[S]** tasks. Run them to confirm they work.
6. For **[L]** tasks: create empty files or stubs with signatures and docstrings describing the contract, plus failing tests that encode the acceptance criteria. No implementation logic.
7. For **[P]** tasks: list them and remind me to write my design or first attempt in the week notes before you help.
8. Set this week's status to ▶ in `PROGRESS.md`.
9. Finish with a short plan for the week sized to ~12–15 hours, in the order I should work.
