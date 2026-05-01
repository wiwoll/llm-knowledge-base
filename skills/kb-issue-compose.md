---
name: kb-issue-compose
description: Assemble a full magazine issue from drafted articles into a single composed document with TOC, page-budget arithmetic (divisible by 4), section ordering, ad slot placeholders, cross-references resolved (page numbers for CME → MC, editorial → CMEs, Kurzfachinformation refs). Validates with /kb-style-lint before composing. Usage: /kb-issue-compose <magazine-slug> <issue-id> [--render markdown|indesign-xml|pdf-mockup]
trigger: /kb-issue-compose
allowed-tools: Read, Write, Edit, Bash, Grep
---

# KB Issue Compose

Assemble drafted articles into a complete issue. Produces a single composed Markdown document, computes final page numbers, resolves all cross-references, and (optionally) emits a layout-ready format for InDesign or a PDF mockup.

## Steps

### 1. Read Spec & Plan

```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
cat {KB_PATH}/wiki/issues/{ISSUE_ID}/issue-plan.md
```

Internalize §2 (Page architecture) and §15 (Page-budget arithmetic).

### 2. Inventory the Issue Directory

```bash
find {KB_PATH}/wiki/issues/{ISSUE_ID} -name "*.md" -type f | sort
```

Expected layout:
```
issues/{ISSUE_ID}/
├── issue-plan.md
├── editorial.md
├── cme/
│   ├── {topic-1}.md
│   ├── {topic-2}.md   (optional)
│   └── fragen.md
├── medizin/
│   ├── study-{...}.md
│   ├── congress-{...}.md
│   └── review-{...}.md
├── news/
│   ├── {wissenschaft-slug-1}.md
│   ├── ...
│   └── markt-medizin-{...}.md
├── sponsored/
│   └── {sponsor}-{product}-{...}.md
└── praxismanagement/
    └── {topic-slug}.md
```

For any required file missing per the issue plan, log a gap. Compose proceeds with placeholders for missing sections.

### 3. Pre-Lint

Invoke `/kb-style-lint {ISSUE_ID}`. If there are **error**-severity issues, refuse compose:

```
Pre-compose lint failed: {N} errors across {N} files.
Run /kb-style-lint {ISSUE_ID} for details. Fix errors, then re-run /kb-issue-compose.
```

Warnings do not block.

### 4. Section Ordering

Per spec §2:

```
1. Cover (U1) — auto-generated placeholder unless cover_mode == "sponsored-takeover"
2. U2 ad — placeholder
3. EDITORIAL (printed p.1, PDF p.3)
4. INHALTSVERZEICHNIS (printed p.2, PDF p.4)
5. NEWS (Wissenschaft) — front block
6. CME-FORTBILDUNG
   - CME article 1
   - CME article 2 (if present)
   - Anleitung zur Online-Fortbildung page
   - Fortbildungsfragen page
7. MEDIZIN
   - Study summaries
   - Congress reports
   - External reviews
8. SONDERREPORT / PUBLIREPORTAGE (interspersed at appropriate spots, not strictly grouped)
9. MARKT & MEDIZIN (back of book, before Praxismanagement)
10. PRAXISMANAGEMENT
11. WEITERE RUBRIKEN
    - Auf einen Blick (if present)
    - Board (Editorial Board listing)
    - Impressum
    - Das Letzte (back-of-book teaser; from 2024+)
12. U3 ad — placeholder
13. U4 (back cover) ad — placeholder
```

Adjust per the issue plan's section list.

### 5. Page-Number Assignment

Walk through the section order, summing each article's `target_pages`. For each article, assign:
- `assigned_start_page`: PDF page number where it starts
- `assigned_end_page`: PDF page number where it ends

The TOC uses **printed page numbers** (PDF page − 2 typically, since the cover (U1) is PDF p.1 and U2 is PDF p.2; printed p.1 = PDF p.3 onwards). Track both.

After summing all articles:
- Compute total pages
- Determine ad allocation needed to round up to nearest multiple of 4
- Place mid-book ads at conventional positions (per spec: facing high-value editorial like CME, MS-DMT articles, etc.)

If total exceeds the planned page count, list overflow articles and prompt:
```
Page overflow: planned {N}, actual {M}. Overflow articles:
  - {file} ({extra} pages over budget)
Resolve by trimming or moving to next issue, then re-run.
```

### 6. Cross-Reference Resolution

For every article, find and replace placeholder cross-references:

| Placeholder | Resolution |
|---|---|
| `> Fortbildungsfragen auf Seite {placeholder TBD by /kb-issue-compose}` | actual MC page number |
| `Kurzfachinformation auf Seite {N}` | resolved to actual page where Kurzfachinformation prints |
| Editorial header bar dotted leaders (`{Topic} ............ Seite {N}`) | resolved CME page numbers |
| TOC entries | full page list with comma-separated WEITERE-RUBRIKEN line |
| Cover teaser block: 3 columns CME/Medizin/Praxismanagement | populated from picked highlights (use the issue plan's "marketing summary" or top 6–8 articles) |

### 7. Strip Editor-Only Annotations

For the print render specifically, strip:
- HTML comments `<!-- ... -->`
- The MC-question `> *Korrekt:* X — explanation` lines (the answer key — print version is reader-only; medizinonline.com has the keys)
- Any line tagged `<!-- editor-only -->` or `<!-- not-for-print -->`

For the editor render (`--render markdown` default), keep these.

### 8. Compose the Output

Write to `wiki/issues/{ISSUE_ID}/composed.md` (the master composed doc):

```markdown
---
type: composed-issue
magazine: {MAGAZINE_SLUG}
issue_id: {ISSUE_ID}
total_pages: {N}
divisible_by_four: true
generated_at: {now}
ad_allocation:
  U2: 1
  mid_book: {N}
  U3: 1
  U4: 1
section_pages:
  editorial: {N}
  toc: {N}
  news_front: {N}
  cme: {N}
  medizin: {N}
  sponsored: {N}
  markt_medizin: {N}
  praxismanagement: {N}
  weitere_rubriken: {N}
  ads_total: {N}
articles:
  - file: editorial.md
    section: editorial
    pages: [3, 3]
  - file: cme/{topic-1}.md
    section: cme
    pages: [6, 11]
  - …
lang: {DEFAULT_LANG}
editorial_status: draft
created_at: {now}
updated_at: {now}
tags: [composed, {issue_id}]
---

# {Magazine Title} {ISSUE_ID}

> Composed issue. {N} pages. Divisible by 4 ✓.

## Cover (U1)
[Placeholder: cover photo + 3-column teaser block. See issue-plan §Cover.]

## U2 (Inside-front-cover ad)
[Placeholder: ad slot — sponsor TBD.]

## Editorial (printed Seite 1)
{include verbatim from editorial.md, with cross-refs resolved}

## Inhaltsverzeichnis (printed Seite 2)
{auto-generated TOC per spec §4}

## News
{include each news/{wissenschaft-slug}.md}

## CME-Fortbildung (Seiten {start}–{end})
{include cme/{topic-1}.md}
{include cme/{topic-2}.md if present}
{include Anleitung zur Online-Fortbildung block}
{include cme/fragen.md, with answer keys stripped}

## Medizin
{include each medizin/{...}.md in plan order}

## Sonderreport / Publireportage
{include sponsored/{...}.md at planned positions}

## Markt & Medizin
{include news/markt-medizin-{...}.md}

## Praxismanagement
{include praxismanagement/{...}.md}

## Weitere Rubriken
{auto-generate Board, Impressum; include Auf einen Blick / Das Letzte if present}

## U3 (Inside-back-cover)
[Placeholder: typically congress or society announcement.]

## U4 (Back cover)
[Placeholder: pharma full-page ad.]
```

### 9. Optional Render Targets

#### `--render markdown` (default)
The composed.md file IS the render. Editor reviews in Obsidian or any Markdown viewer.

#### `--render indesign-xml`
Emit a structured XML at `wiki/issues/{ISSUE_ID}/composed.indesign.xml` mapping articles, sections, page numbers, and frontmatter to InDesign-compatible tags. (Format spec to be developed with the layout team — for now, produce a structured placeholder.)

#### `--render pdf-mockup`
Use `pandoc` (if available) to produce a styled PDF mockup with an InFo-NP-approximation stylesheet. This is for editor preview, not production print:
```bash
pandoc {KB_PATH}/wiki/issues/{ISSUE_ID}/composed.md -o {KB_PATH}/outputs/{ISSUE_ID}-mockup.pdf --pdf-engine=xelatex --css=...
```
If pandoc/xelatex is not installed, skip silently and inform the user.

### 10. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: compose issue {ISSUE_ID} ({N} pages)"
```

### 11. Print Summary

```
Issue composed: {ISSUE_ID}
  Datei: wiki/issues/{ISSUE_ID}/composed.md
  Total pages: {N} (divisible by 4 ✓)
  Section breakdown:
    Editorial: 1
    TOC: 1
    News: {N}
    CME: {N}
    Medizin: {N}
    Sonderreport / Publireportage: {N}
    Markt & Medizin: {N}
    Praxismanagement: {N}
    Weitere Rubriken: {N}
    Ads (U2 + mid + U3 + U4): {N}

  Cross-refs resolved:
    CME → Fortbildungsfragen page: ✓ (Seite {N})
    Editorial → CME page refs: ✓
    Kurzfachinformation refs: ✓
    TOC: ✓ (entries: {N})

Status: composed (editorial_status: draft).

Nächste Schritte:
  1. Layout-Team: composed.md übergeben (oder composed.indesign.xml).
  2. Korrekturabzug an Redaktion + Verkauf + Chefredaktion.
  3. Nach Freigabe: /kb-review {ISSUE_ID}/composed --status published --reviewer @chefredaktion
  4. Druckfreigabe.
```

## Notes

- Compose is **not idempotent** in the rendering: a re-run regenerates all cross-refs, page numbers, and the composed file. But the source articles (CME, Medizin, etc.) are **never modified** — their text is included verbatim minus stripped editor-only annotations.
- If layout iterates and changes page allocations, re-run compose to update cross-refs. The source articles can stay locked.
- The composed file is the contract for layout handoff. After layout returns the InDesign file, the editor's job is to do the final visual proofread; the composed.md is the textual gold standard.
- For sponsored content with multi-round correction loops, do NOT re-compose between every sponsor round — only compose after all sponsor sign-offs are folded back in.
- The Editorial header bar (CME page numbers) is the most error-prone cross-ref. Compose verifies these match the actual CME pages and refuses if mismatched.
