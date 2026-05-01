---
name: kb-ask
description: Ask a question against the wiki. Reads the index to navigate relevant articles, synthesizes a grounded answer with citations (including DOI/PMID for medical sources), and saves the answer to outputs/. Prefers peer-reviewed and high-evidence sources, flags claims that rest only on draft/unreviewed wiki articles. Usage: /kb-ask <your question>
trigger: /kb-ask
allowed-tools: Read, Write, Edit, Bash
---

# KB Ask

Answer a question using the knowledge base wiki. Uses `wiki/index.md` as the navigation layer — reads the index first, then pulls only the relevant articles. Does not load the full wiki into context.

## Steps

### 1. Read Config

Run:
```bash
cat ~/.claude/kb-config.json
```

Extract `kb_path`. Expand `~` to the actual home directory path.
Set this as `KB_PATH` for all subsequent steps.

Also extract `default_output_lang` (default `"de"`) and `supported_langs` (default `["de","en"]`).
Set these as `DEFAULT_LANG` and `SUPPORTED_LANGS`.

**Detect the question's language** (`QUESTION_LANG`). If it matches one of `SUPPORTED_LANGS`, use it. If unsupported, fall back to `DEFAULT_LANG`. The answer must be written in `QUESTION_LANG` regardless of the languages of the source articles consulted.

### 2. Read the Index

Run:
```bash
cat {KB_PATH}/wiki/index.md
```

This is your navigation map. It contains one-line summaries of every concept, source, and output in the wiki.

### 3. Identify Relevant Articles

Based on the question, scan the index and select the 3–5 most relevant articles. Selection priority:

- **Concept articles** (`concepts/`) — prefer when the question asks about a topic, mechanism, or idea. Within concepts, prefer those marked `[fact-checked]` or `[published]` over `[draft]` when both exist.
- **Source articles** (`sources/`) — prefer when the question asks about a specific paper, author, dataset, or work. For medical questions, prefer sources with higher `evidence_level` (1a/1b/2a > others) and `peer_reviewed: true`.
- **Output articles** (`outputs/`) — check these first if the question may already have been answered in a prior Q&A session.

### 4. Read Relevant Articles

For each selected article, read it:
```bash
cat {KB_PATH}/wiki/{article-path}.md
```

If reading an article reveals additional relevant concepts or sources (via its `## Sources` backlinks or body text), read those too. Read at most 8 articles total to stay within context.

### 5. Synthesize Answer

Write a clear, grounded answer to the question **in `QUESTION_LANG`**:
- Cite specific wiki articles inline using `[[wiki-link]]` format (e.g. `[[concepts/attention-mechanism]]`). The wiki-link slug stays as-is regardless of answer language.
- For medical claims, cite the underlying source by DOI or PMID inline as well, so the editorial team can verify externally without opening the source article: `[[concepts/glp-1-agonisten]] (Wilding et al., NEJM 2021, doi:10.1056/nejmoa2032183)`.
- If a consulted source is in a different language than `QUESTION_LANG`, translate any quoted material into `QUESTION_LANG` and note the original language briefly (e.g. "Laut Vaswani et al. (en): ...").
- If the wiki does not contain enough information to answer fully, say so explicitly — state what is known and what is missing. Do not fabricate.
- **Confidence calibration:** explicitly mark the strength of each major claim using one of three tags at the end of the relevant sentence:
  - `(belegt)` / `(established)` — supported by ≥2 fact-checked or published wiki articles, or by a single high-evidence source (`evidence_level` 1a/1b)
  - `(vorläufig)` / `(provisional)` — supported by draft articles or single observational/case-level sources
  - `(unklar)` / `(uncertain)` — wiki contains contradictory or insufficient evidence; explain the gap
- Match length to complexity: 1 paragraph for simple questions, structured sections with headings for complex ones.

### 6. Generate Output Slug and Path

Generate a `slug` from the question:
- Take the first 6–8 significant words (skip stop words like "what", "is", "the", "how", "does")
- Lowercase, join with `-`
- Example: "what is the attention mechanism?" → `attention-mechanism`

Set `OUTPUT_DATE` to today's date in `YYYY-MM-DD` format.
Set `OUTPUT_FILE` = `outputs/{OUTPUT_DATE}-{slug}.md`

### 7. Write Output File

Write to `{KB_PATH}/{OUTPUT_FILE}`:

```markdown
---
type: qa-answer
question: {exact question asked}
answered_at: {current UTC ISO timestamp}
created_at: {current UTC ISO timestamp}
updated_at: {current UTC ISO timestamp}
lang: {QUESTION_LANG}
tags: [{3–6 tags inferred from the question and answer topic}]
sources_consulted: [{comma-separated list of article paths read, e.g. "concepts/attention.md", "sources/vaswani-2017.md"}]
confidence_summary: {one of "established", "mixed", "provisional", "uncertain" — best descriptor of the answer overall}
---

# {Question}

{Synthesized answer in {QUESTION_LANG} with [[wiki-link]] citations inline and confidence tags as described above}

## Sources Consulted
{Bulleted list of all articles read, formatted as [[wiki-links]]. For medical sources, include DOI/PMID after the link: "- [[sources/wilding-2021]] — doi:10.1056/nejmoa2032183, evidence level 1b"}
```

### 8. Update Index

Append to the `## Antworten & Berichte / Outputs` section of `{KB_PATH}/wiki/index.md` (or the matching `## Outputs` heading if older index format):

```
- [[{OUTPUT_FILE without .md extension}]] — {one-line summary in {DEFAULT_LANG}: the question asked and the core answer in one sentence; mark with "(en)" or similar if QUESTION_LANG differs from DEFAULT_LANG}
```

Write the updated index back to disk.

### 9. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: answer '{question truncated to 50 chars}'"
```

### 10. Print Confirmation

```
Answer saved to: {OUTPUT_FILE}
```
