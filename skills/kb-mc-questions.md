---
name: kb-mc-questions
description: Generate the Multiple-Choice question page (Fortbildungsfragen) for one or more CME articles in an issue. Produces 6–8 questions per article (or 11–14 combined for two-CME issues), with appropriate single-best vs. multi-select mix, 4 options per question, plausible distractors, and Bloom-level distribution following the magazine spec. Usage: /kb-mc-questions <magazine-slug> <issue-id> <topic-slug>[,<topic-slug-2>]
trigger: /kb-mc-questions
allowed-tools: Read, Write, Edit, Bash
---

# KB MC Questions

Generate the CME quiz page for an issue's CME article(s). Output goes to `wiki/issues/{ISSUE_ID}/cme/fragen.md` and is consumed by `/kb-issue-compose` for the print Fragen page and by `medizinonline.com` upload.

## Steps

### 1. Read Spec & CME Drafts

```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
```

For each topic-slug passed (1 or 2), read the CME draft:
```bash
cat {KB_PATH}/wiki/issues/{ISSUE_ID}/cme/{TOPIC_SLUG}.md
```

If the draft is missing, refuse:
```
Error: CME draft for {TOPIC_SLUG} not found. Run /kb-cme-draft first.
```

### 2. Plan Question Budget

Per spec §8.4:
- 1 CME article → 6–8 questions
- 2 CME articles → 11–14 questions combined (e.g., 8 + 3 = 11)

Mode mix per spec §8.5:
- Aim for ~60% single-best, ~40% multi-select
- Round to integer counts; ensure at least 1 of each mode for variety

### 3. Question Design Rules (per spec §8.3)

For each question:
- **Stem** ends with mode marker in italics:
  - `(gesuchte Antwort ankreuzen)` for single-best (exactly 1 correct)
  - `(alle gesuchten Antworten ankreuzen)` for multi-select (≥1 correct, may be all 4)
- **Options**: 4 (A/B/C/D). 3 options (A/B/C) acceptable for short factoid questions but rare.
- **Bloom level distribution**:
  - Default: ~70% recall (level 1) + ~30% comprehension (level 2)
  - For NP1-26-style modern issues: include ≥1 vignette-based application question (level 3) per article — short clinical case in stem, then ask for the most appropriate next step / risk profile / diagnosis.
- **Distractors must be plausible** medical concepts within the article's topic — not obviously wrong, not gibberish.
- **Coverage**: questions tightly follow article subheadings. Use the article's `learning_objectives` from frontmatter as the question scaffold.

### 4. Generate Questions

For each article, draft the planned number of questions following the article's body order:
- 1 question on Definition / Epidemiologie
- 1–2 questions on Pathophysiologie / Mechanismus
- 1–2 questions on Klinik / Diagnostik
- 2–3 questions on Therapie (the heart of clinical CME)
- 0–1 question on Praxis (if vignette-friendly)

Ensure:
- Correct answer can be **directly traced to a sentence in the article body**
- Distractors reference real concepts from the same topic (other drugs, alternative diagnoses, common confusions)
- The question stem does **not** include the answer hint
- For vignette questions, the case is realistic, anonymized, age-and-sex-specified, and references a typical Swiss-DACH clinical scenario

### 5. Build the Fragen Page

Set `FRAGEN_FILE = wiki/issues/{ISSUE_ID}/cme/fragen.md`.

```markdown
---
type: cme-mc
magazine: {MAGAZINE_SLUG}
magazine_section: cme-fortbildung
issue_id: {ISSUE_ID}
covers_articles: [{slug-1}, {slug-2}]
lang: {LANG (matches article lang; if mixed, use DEFAULT_LANG)}
editorial_status: draft
question_count: {N}
mode_distribution:
  single_best: {N}
  multi_select: {N}
bloom_distribution:
  recall: {N}
  comprehension: {N}
  application: {N}
created_at: {now}
updated_at: {now}
tags: [cme-mc, {magazine_slug}, {issue_id}]
---

# > Fortbildungsfragen

> {Article 1 title}
> {Article 2 title (if present)}

**Zur Beantwortung der Fragen sind folgende Artikel erforderlich:** {article 1 title}{; article 2 title if present}.

---

## Frage 1
**{Question stem ending with body context.} *(gesuchte Antwort ankreuzen)***

A) {Option A}
B) {Option B}
C) {Option C}
D) {Option D}

> *Korrekt:* D — {one-sentence explanation citing the article: "Siehe {Article 1 title}, Abschnitt Therapie."}

## Frage 2
**{Vignette-style stem, e.g., "Eine 67-jährige Patientin mit … Welche Therapie ist am ehesten indiziert?"} *(gesuchte Antwort ankreuzen)***

A) {…}
B) {…}
C) {…}
D) {…}

> *Korrekt:* B — …

…

## Frage {N}
**{stem} *(alle gesuchten Antworten ankreuzen)***

A) {…}
B) {…}
C) {…}
D) {…}

> *Korrekt:* A, C — …

---

## Anleitung zur Online-Fortbildung

Aktivieren Sie auf **medizinonline.com** Ihren kostenlosen Account. Wählen Sie die Ausgabe **{ISSUE_ID}** der Zeitschrift InFo Neurologie & Psychiatrie aus, beantworten Sie die Fragen und laden Sie Ihr CME-Zertifkat (PDF) direkt herunter.

```

(Note: the typo `Zertifkat` — missing `i` — is a stylistic fingerprint preserved verbatim per the magazine spec.)

The `> *Korrekt:* X — explanation` annotation is **internal only** — it is included in the markdown source for editor review and for upload to medizinonline.com, but stripped by `/kb-issue-compose` before the print render. Mark this clearly in the file:

```markdown
<!-- Korrekturschlüssel: nicht für den Druck. Wird beim /kb-issue-compose entfernt. -->
```

### 6. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: mc questions {ISSUE_ID} ({N} questions for {topic list})"
```

### 7. Print Summary

```
MC-Fragen erstellt: {FRAGEN_FILE}
  Fragen total: {N}
  Single-best: {N}    Multi-select: {N}
  Bloom: Recall {N}, Comprehension {N}, Application {N}
  Sprache: {LANG}
  Status: draft

Korrekturschlüssel ist im Markdown enthalten (HTML-Kommentar) und wird bei /kb-issue-compose vor dem Druck entfernt.
Hochzuladen auf medizinonline.com.
```

## Notes

- The `> *Korrekt:* X` lines stay in the markdown for editor review and accreditation-platform upload but are stripped before print.
- For two-CME issues, the questions for each article are interleaved by article in the file but counted in the same combined Fragen-set (per spec §8.4).
- If `learning_objectives` are missing in the CME draft frontmatter, infer them from the article body — but flag in the output file: `inferred_objectives: true`.
- Re-running this skill **regenerates** the fragen file but never silently overwrites a fragen file with `editorial_status` ≥ `fact-checked`. In that case, refuse and require `/kb-review` to demote first.
