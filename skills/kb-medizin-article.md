---
name: kb-medizin-article
description: Draft a Medizin-section article — study summary, congress report, or external review. Length 1–4 pages depending on type; staff initials byline for study/congress, external author for review. Always editorial_status=draft. Usage: /kb-medizin-article <magazine-slug> <issue-id> <topic-slug> --kind study|congress|review [--source <ref>] [--author "..."]
trigger: /kb-medizin-article
allowed-tools: Read, Write, Edit, Bash, mcp__claude_ai_PubMed__search_articles, mcp__claude_ai_PubMed__get_article_metadata, mcp__claude_ai_PubMed__get_full_text_article, WebFetch
---

# KB Medizin Article

Draft a Medizin-section article (study, congress, or review).

## Steps

### 1. Read Spec & Plan

```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
cat {KB_PATH}/wiki/issues/{ISSUE_ID}/issue-plan.md
```

Confirm the topic is in the plan's Medizin list. Note locked author / source / page allocation.

### 2. Parse Arguments

`<magazine-slug> <issue-id> <topic-slug>` required.
`--kind` required: `study` (1–2 p), `congress` (1–4 p), `review` (2–4 p).
`--source` references the primary source (DOI, PubMed URL, congress abstract URL, etc.).
`--author` for `review`; for `study`/`congress` defaults to staff byline initials.

### 3. Acquire Source

#### For `study`
PubMed MCP if available; otherwise WebFetch the source URL. Capture:
- Full bibliographic citation
- Abstract verbatim
- Sample size, primary endpoint, effect size, p, CI
- Limitations explicitly stated by authors
- Funding source

#### For `congress`
WebFetch the abstract / congress page. Capture:
- Abstract title + presenter + affiliation
- Congress name + edition + dates + city (e.g., "96. Kongress der Deutschen Gesellschaft für Neurologie. 8.–11. November 2023.")
- Session type (oral / poster / late-breaker)
- Key findings + Q&A from on-demand material if accessible

#### For `review`
External author provides the manuscript. The skill formats and structures it (and may suggest improvements via inline editor comments) but does not invent the body.

### 4. Compose

#### `study` skeleton
- **Headline (two-deck)**: Kicker (disease area) + main descriptive headline
- **Lead-Box** (Standfirst): 3–5 sentences summarising the finding
- **Body** opens with `■({staff_initials})`:
  1. Hintergrund / clinical context
  2. Methodik (study design, n, primary endpoint, key inclusion criteria)
  3. Ergebnisse (effect size, CI, p; secondary endpoints relevant)
  4. Diskussion (limitations, comparison to existing evidence, hedged interpretation)
  5. Praxisrelevanz (1 short paragraph — what does this mean for the Swiss neurologist/psychiatrist)
- **References**: Vancouver `[1]`, Literatur list at end
- **Word count**: ~600–1200 (= 1–2 pages)

#### `congress` skeleton
- **Banner** (bottom-right of page): `KONGRESS   {CONGRESS NAME} {YEAR}` (uppercase, two-space gap)
- **Headline (two-deck)**: Kicker (disease area) + main
- **Lead-Box**
- **Body** opens with `■({staff_initials})`:
  1. Hintergrund
  2. Vortrag / Präsentation (presenter, title, key data)
  3. (Optional) **Kurzinterview** with the referent — Q&A format, 2–4 questions
  4. Einordnung
- **References**: numbered abstract numbers (`Abstract 40. {Congress full name}. {Date range}.`)
- **Word count**: ~600–2400 (1–4 pages)

#### `review` skeleton
- **Headline (two-deck)** + author block (top-right with photo)
- **Lead-Box**
- **Body** structured as the external author wrote it (subhead the body if missing); add the magazine's **▪** lead glyph if absent
- Optional **Tab.** / **Abb.** / **Übersicht** boxes
- **Take-Home-Messages** box optional for review; standard if review is therapy-focused
- **References**: Vancouver
- **Author conflict-of-interest**: capture in frontmatter (`coi:`) but do not print (per spec §7.3)
- **Word count**: ~1200–2400 (2–4 pages)

### 5. Frontmatter

```yaml
---
type: medizin-{kind}
magazine: {MAGAZINE_SLUG}
magazine_section: medizin
issue_id: {ISSUE_ID}
topic_slug: {TOPIC_SLUG}
lang: {LANG}
editorial_status: draft
target_pages: {N}
{For study:}
byline: {staff_initials}
primary_source:
  title: {paper title}
  doi: {DOI}
  pmid: {PMID}
  journal: {journal}
  publication_date: {YYYY-MM-DD}
  authors: [...]
  study_type: {rct|cohort|meta-analysis|...}
  evidence_level: {1a|1b|2a|2b|3a|3b|4|5}
{For congress:}
byline: {staff_initials}
congress:
  name: {full name}
  short_name: {DGN | EAN | ECTRIMS | AAN | AAIC | APA | DGPPN | SGN | SGPP}
  edition: {N}
  dates: {YYYY-MM-DD to YYYY-MM-DD}
  city: {city}
  abstract_id: {ID}
  presenter: {name + affiliation}
{For review:}
authors:
  - title: {Prof. Dr. med.}
    name: {…}
    institution: {…}
    address: {…}
    email: {…}
    photo: present|absent
coi:
  declared: {true|false|unknown}
  details: {free text}
created_at: {now}
updated_at: {now}
tags: [medizin, {kind}, ...]
---
```

### 6. Style Discipline (per spec)

- INN drug names only (brand parenthetical at most for orientation in `review`)
- Hedge language: "deuten darauf hin", "scheinen", "möglicherweise"
- No "wir" / no "Sie" in body
- Swiss number formatting (comma decimal, thin-space thousands)
- Anführungszeichen «...»
- For congress: include the **▪** glyph + staff byline initials

### 7. Write

Set `ARTICLE_FILE = wiki/issues/{ISSUE_ID}/medizin/{kind}-{TOPIC_SLUG}.md`.

### 8. Commit + Index

Append to wiki index under `## Hefte / Issues`:
```
- [[issues/{ISSUE_ID}/medizin/{kind}-{TOPIC_SLUG}]] — Medizin {ISSUE_ID}: {Title} ({byline or author}) `[draft]`
```

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: medizin ({kind}) {ISSUE_ID}/{TOPIC_SLUG}"
```

### 9. Print

```
Medizin-Artikel ({kind}) erstellt: {ARTICLE_FILE}
  Wörter: ~{N}
  Zielseiten: {N}
  Byline: {byline | author}
  Status: draft

Bei Kongressbeitrag: Optional Kurzinterview-Anfrage an Referent*in (siehe spec §11.2 Prozess).
```

## Notes

- Study summaries are written from the published paper, **not** from the press release if the paper is open-access. Press releases inform News items, full papers inform Medizin-study items.
- For review articles, the skill **never** fabricates the author's content; it formats and structures what the author has supplied. The output article is marked `inferred_from_outline: true` if generated from an outline rather than a full draft.
- Praxisrelevanz paragraph is optional but recommended — it bridges the academic finding to Swiss clinical practice and is high-value for the readership.
