# MATACS AI Proposal Triage Design

- **Name:** Muhammad Uzair Attari
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** Approximately 1.5 hours. The additional time went into refining the process-flow diagram and double-checking edge cases.

## How I Used AI

I used Claude to brainstorm the workflow, help draft the extraction prompt, and sanity-check the scoring calculations. I reviewed the suggestions, checked the logic against the assessment requirements, and made the final design decisions myself.

**Where I disagreed with the AI:**

I kept AI responsible for extracting proposal facts, not assigning scores or deciding rankings. The scoring and eligibility checks are handled by deterministic code using the admin-defined criteria and fixed formulas. I chose this separation because model-generated scores can vary between runs, while rule-based calculations are repeatable, testable, and easier for reviewers to verify.

## Files

- [WRITEUP.md](WRITEUPS.md) – Answers to the five design questions, including the reasoning behind the approach.
- [PROMPTS.md](PROMPTS.md) – The proposal fact-extraction prompt and its security instructions.
- [ranking.json](ranking.json) – Expected outcomes for all five sample proposals.
- [Process Flow Diagram](diagram/flow.pdf) – End-to-end workflow showing AI extraction, rule-based processing, and human checkpoints.