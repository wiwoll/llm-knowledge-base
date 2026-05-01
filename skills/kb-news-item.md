---
name: kb-news-item
description: Draft a single news item — either a Wissenschafts-News (from a university press release on idw-online.de or similar) or a Markt & Medizin item (from a pharma press release). Follows the magazine's news conventions: ▪ glyph opener, hedged tone, Swiss number formatting, mandatory Quelle line, INN-vs-brand discipline. Usage: /kb-news-item <magazine-slug> <issue-id> --kind wissenschaft|markt-medizin --source <URL|press-release-text>
trigger: /kb-news-item
allowed-tools: Read, Write, Edit, Bash, WebFetch
---

# KB News Item

Draft a single news bulletin for a magazine issue, following the canonical news structure.

## Steps

### 1. Read Spec

```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
```

Internalize §9 (News-Wissenschaft) and §10 (Markt & Medizin), §14 (tone), §14.6 (drug-name handling), §14.7 (numbers), §14.5 (citation).

### 2. Parse Arguments

Required:
- `<magazine-slug>` `<issue-id>`
- `--kind wissenschaft` or `--kind markt-medizin`
- `--source <URL>` (preferred) or `--source <press-release-text>` (when no URL)

Optional:
- `--company <name>` — required for `markt-medizin` (the Markt & Medizin tag company)
- `--byline lb|ub|tsk|...` — staff initials for Wissenschaft byline; defaults to current Chefredaktion's initials
- `--target-words 350` — typical 250–450

### 3. Acquire Source Material

If `--source` is a URL, fetch with WebFetch and ask for markdown format.
Extract:
- Press-release title (verbatim — needed for `Quelle:` line)
- Date of release (DD.MM.YYYY)
- Issuing institution (university name + country code if foreign, e.g., `Universität Leipzig (D)`)
- Main finding / claim
- Study design, sample size, primary endpoint
- Effect size, p-value, CIs (when reported)
- Researcher quotes (capture verbatim — if quoted material will be used, German Guillemets «...»)
- Funding statement / DOI of the underlying paper if mentioned

For pharma press releases (`markt-medizin`), additionally extract:
- Brand name + INN of the product
- Indication
- Trial name (e.g., MOXIe, STEP, ULTIMATE)
- Regulatory status (Swissmedic / EMA / FDA approval, label expansion, etc.)
- Reimbursement status (Spezialitätenliste, kassenzulässig)

### 4. Compose the Item

#### For Wissenschafts-News
- **Two-deck headline**:
  - Kicker: 1–4 words naming the disease/area (e.g., "Multiple Sklerose", "Schlaganfall")
  - Main headline: 4–9 words, descriptive ("Lokale Biomarker können schwere Verläufe vorhersagen") or question form
- **Lead-Box (Standfirst)**: 3–5 sentences in distinct frame above body summarising the finding
- **Body** opens with `■({byline})` (e.g., `■(ub)`) followed by the body prose
  - Tone: impersonal, neutral-clinical
  - Use Konjunktiv I/II for indirect speech: "Die Wissenschaftler vermuten, dass …"
  - Hedge findings: "deuten darauf hin", "möglicherweise", "scheinen"
  - INN drug names only if drugs mentioned
  - 250–450 words
- **End footer**:
  ```
  Quelle: «{press-release title verbatim}», {DD.MM.YYYY}, {Institution} (country code if foreign)
  ```

#### For Markt & Medizin
- **Tag line at top**: `MARKT & MEDIZIN   {Company Name}`
- **Two-deck headline**:
  - Kicker: indication or product class
  - Main: brand-name allowed prominently
- **Body**:
  - 250–450 words, neutral-positive
  - Brand name allowed prominently (this is what differentiates it from editorial News)
  - Do NOT compare unfavourably to competitors
  - Cite trial name + primary results
  - Mention regulatory + reimbursement status when relevant
  - No Konjunktiv-distance from the company's claims (it's their slot)
- **End footer**:
  ```
  Quelle: {press-release title}, {DD.MM.YYYY}
  ```
- **No** Kurzfachinformation in this format (that's for Sonderreport/Publireportage). No `Anzeige` tag.

### 5. Frontmatter

```yaml
---
type: news-{kind}
magazine: {MAGAZINE_SLUG}
magazine_section: news
issue_id: {ISSUE_ID}
lang: {LANG}
editorial_status: draft
{For wissenschaft:}
byline: {staff initials}
{For markt-medizin:}
company: {Company Name}
product:
  brand: {brand}
  inn: {inn}
  indication: {indication}
trial: {trial name}
regulatory_status: {Swissmedic | EMA | FDA notes}
press_release:
  title: {verbatim title}
  date: {YYYY-MM-DD}
  source_institution: {institution name}
  source_url: {URL}
target_words: {N}
created_at: {now}
updated_at: {now}
tags: [news, {kind}, {issue_id}, ...]
---
```

### 6. Write

Set `NEWS_FILE`:
- `wiki/issues/{ISSUE_ID}/news/{slug}.md` (auto-generate slug from kicker + main headline, max 50 chars)

### 7. Style Checks

Inline lint before saving:
- Decimal separator must be comma (regex: `\b\d+\.\d+\b` should NOT match in numerical contexts)
- Thousands separator: thin space ` ` not `.` or `,`
- No `ß` (must be `ss`)
- No `bahnbrechend` / `revolutionär` / `Game-Changer` outside «...» quotes
- For Wissenschaft: no brand-name in lead — INN only
- For Markt & Medizin: brand name must appear in lead — that's the format

If any lint fails, attempt a fix in the prose; if still failing after one auto-pass, surface the failure as a warning in the output rather than refusing to save (editor reviews before publication anyway).

### 8. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: news ({kind}) {ISSUE_ID}/{slug}"
```

### 9. Print Confirmation

```
News-Item erstellt: {NEWS_FILE}
  Typ: {kind}
  Wörter: ~{N}
  Quelle: {press-release title} ({date})
  Status: draft

Style-Lint: {pass | <warnings>}

Hinzugefügt zur Heftplanung. Pro Newsseite passen 2–3 Items.
```

## Notes

- News items never carry author COI — they paraphrase publicly released material.
- Press releases must be **rewritten in magazine voice**, never copy-pasted. The `Quelle:` line credits but does not absolve.
- For news from a German institution, mark country: `Universität Leipzig (D)`. For Swiss institutions, no country tag.
- Markt & Medizin items can cover the same drug as a competing editorial News item in the same issue — that's allowed; the labels differentiate.
- If the press release is in English (e.g., FDA, EMA), translate to German per spec; preserve trial names verbatim (English brand names, English trial acronyms).
