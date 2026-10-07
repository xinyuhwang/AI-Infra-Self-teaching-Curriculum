---
description: Quiz me from the question bank and record scores
argument-hint: <week number | question ID | month N | "random N">
---
Quiz me using `docs/QUESTIONS.md`. Selection: $ARGUMENTS
- A week number → that week's questions.
- A question ID (e.g. LLM-05) → that question only.
- "month N" → all questions from that month's weeks.
- "random N" → N questions drawn from weeks already completed in `PROGRESS.md`, favoring low or stale scores.

For each question, one at a time:
1. Ask the question. Wait for my answer. Don't give hints unless I ask.
2. Ask at least one follow-up ("why?", "what would change if…?", "how would you measure that?").
3. Score 0–3 using the rubric in `docs/QUESTIONS.md`. A 3 requires me to point at a real artifact (file, plot, report, logged number); check it exists.
4. Explain briefly what was missing or wrong. Be strict — generous scores defeat the purpose.

At the end: show a table of scores, then update the "Question scores" section of `PROGRESS.md` (score, date, evidence) and the Summary counts.
