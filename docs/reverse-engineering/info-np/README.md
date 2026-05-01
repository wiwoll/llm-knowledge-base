# Reverse-Engineering Audit Trail — InFo Neurologie & Psychiatrie

This directory contains the four parallel-agent analysis reports that produced the canonical magazine specification at [`skills/magazines/MAGAZINE-INFO-NP.md`](../../../skills/magazines/MAGAZINE-INFO-NP.md).

Methodology (2026-05-01):
- Source corpus: 18 issues NP1-23 → NP1-26, plus the publisher's process documentation (`Themenplan NP3-26 Neurologie Psychiatrie.docx`).
- Four general-purpose agents ran in parallel, each with a non-overlapping focus area, sampling strategically across all issues.
- Each agent produced a structured report that became one input to the consolidated spec.

Reports:
1. [`structure.md`](structure.md) — Magazine structure, editorial blueprint, evolution 2023→2026
2. [`cme.md`](cme.md) — CME articles deep-dive (format, MC questions, accreditation, authorship, topic distribution)
3. [`pharma.md`](pharma.md) — Pharma-sponsored content, ad ecology, doctor's-choice signals, compliance norms
4. [`voice.md`](voice.md) — Tone of voice, language register, do's and don'ts, recurring rhetorical templates

The spec is updated as new issues land or as the open-issues list (spec §19) is resolved by editorial decisions. Re-runs of the agent process should be logged here with a date suffix (e.g., `structure-2027-01.md`) so the evolution can be tracked.
