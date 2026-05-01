---
name: kb-style-lint
description: Lint a magazine article (or all articles in an issue) against the magazine's house style — Swiss number formatting, drug-name discipline, hedging language, voice match, sponsored-content disclosures, schema completeness. Reports violations; does not auto-fix. Usage: /kb-style-lint <path-or-issue-id> [--magazine info-np] [--voice burggraf|schliebe|blanke]
trigger: /kb-style-lint
allowed-tools: Read, Bash, Grep
---

# KB Style Lint

Run a style audit against the magazine spec. Reports issues by severity (error / warning / info). Does **not** modify files.

## Steps

### 1. Read Spec

```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
```

### 2. Resolve Target

- If argument is a path to a single `.md` file → lint that file
- If argument is an issue-id (e.g., `NP3-26`) → lint every file under `wiki/issues/{ISSUE_ID}/**/*.md` recursively
- If argument is `wiki/issues/{ISSUE_ID}/cme/{slug}.md` etc. → lint that file

### 3. Per-File Linting

For each target file, parse YAML frontmatter and body, then run the relevant checks based on `type`.

### 4. Universal Checks (any article type)

| Check | Severity | Detail |
|---|---|---|
| `lang` set, in `supported_langs` | error | per SCHEMA.md |
| `editorial_status` set | error | concept articles must have it |
| Schwa Swiss orthography | error | no `ß` characters anywhere in body |
| Decimal separator | error | numbers in body use comma `,` for decimals (e.g., `1,5 mg`), not period |
| Thousands separator | warning | uses thin space ` ` not period or comma |
| Anführungszeichen | warning | uses Guillemets «...» not `"..."` (especially around quoted speech) |
| Hyped vocabulary | warning | `bahnbrechend`, `revolutionär`, `Game-Changer`, `Meilenstein` flagged unless inside «...» quotes |
| Banned headline punctuation | warning | trailing `!` (except in editorial flourish) |

### 5. Type-Specific Checks

#### `cme` articles
| Check | Severity | Detail |
|---|---|---|
| `TAKE-HOME-MESSAGES` box present | error | per spec §7.2 — mandatory |
| Take-Home bullets use `―` not `•` | warning | em-dash convention |
| Sponsor disclosure block present | error | verbatim formula match required |
| `Fazit` or `Konklusion` paragraph present | error | mandatory close |
| `keywords` line italic 3–4 keywords | warning | structural |
| Vertical sidebar `> Fortbildungsfragen auf Seite` placeholder present | error | required for cross-link to MC page |
| Author block has title + name + position + institution + address + email | error | full block per spec |
| References use Vancouver `[n]` brackets in body | warning | not `(Author 2024)` |
| References list at end | error | unless body has `Literaturliste bei Verlag` |
| No PMID printed in references | warning | spec §14.5 — store in frontmatter, not in print |
| Brand name only as parenthetical orientation | warning | brand-first wording → must be sponsored, not CME |
| ▪ glyph at start of lead | warning | lead-glyph convention |

#### `cme-mc` (MC questions page)
| Check | Severity | Detail |
|---|---|---|
| 6–8 questions per article (or 11–14 combined) | error | per spec §8.4 |
| Each Q ends with `(gesuchte Antwort ankreuzen)` or `(alle gesuchten Antworten ankreuzen)` | error | mode marker required |
| 4 options A–D (or 3 if explicitly justified) | error | structural |
| `> *Korrekt:* X — explanation` annotation present | error | for editor / medizinonline.com upload |
| `Aktivieren Sie auf medizinonline.com …` instruction block | error | mandatory |
| `Zertifkat` typo preserved | warning | stylistic fingerprint |

#### `news-wissenschaft`
| Check | Severity | Detail |
|---|---|---|
| Body opens with `■` glyph + optional `(initials)` | error | spec §9 |
| `Quelle: «...», DD.MM.YYYY, Institution` line at end | error | mandatory provenance |
| 250–450 words | warning | length budget |
| INN drug names only, no brand-first | error | spec §10 firewall |
| Konjunktiv I/II for indirect speech | info | suggest if direct quotes from researchers are paraphrased |
| Two-deck headline (kicker + main) | warning | structural |

#### `news-markt-medizin`
| Check | Severity | Detail |
|---|---|---|
| Tag line `MARKT & MEDIZIN  {Company}` at top | error | mandatory |
| Brand name allowed prominently | info | (this is what differentiates from editorial) |
| `Quelle: <release title>, DD.MM.YYYY` at end | error | mandatory |
| 250–450 words | warning | |
| No Kurzfachinformation in this format | warning | (Kurzfachinformation belongs to Sonderreport / pharma-anzeige) |

#### `medizin-study` / `medizin-congress` / `medizin-review`
| Check | Severity | Detail |
|---|---|---|
| Lead-Box (Standfirst) present | error | 3–5 sentence summary frame above body |
| Body opens with `■({initials})` for study/congress | error | structural |
| For congress: `KONGRESS  {NAME} {YEAR}` banner | warning | inconsistent practice; flag missing |
| Praxisrelevanz paragraph | info | recommend for study type |
| INN drug names | error | spec §14.6 |
| Vancouver references in body | warning | structural |

#### `sonderreport` / `publireportage`
| Check | Severity | Detail |
|---|---|---|
| Top-of-page label `SONDERREPORT` or `PUBLIREPORTAGE` (capitalised) | error | spec §12.1 |
| Funding-disclosure formula present, verbatim | error | spec §12.3 — exact wording |
| Sponsor footer block (legal name + address + approval code) | error | spec §12.4 |
| Approval code format match | warning | regex: `\w+-\d+_\d{2}\.\d{4}` or `MAT-CH-\d+_\d{2}/\d{4}` or `CH_CP-\d+_v\d+\.\d+` |
| Kurzfachinformation present or referenced | error | spec §12.5 |
| KOL disclaimer if interview format | error | spec §12.3 |
| Off-label discussion absent | error | spec §12.6 — refusal-grade if found |
| Reprint chain footer if reprint | error | spec §12.7 |
| Brand with ®/™ throughout | warning | format requirement |
| Copyright `© Prime Public Media AG` | error | spec §12.8 — even when sponsor commissioned |
| `editorial_status: draft` | error | sponsored content cannot self-promote |

#### `editorial`
| Check | Severity | Detail |
|---|---|---|
| Header bar `Die Fortbildungsthemen in dieser Ausgabe:` (or singular) | error | spec §5 — verbatim |
| Page references to CME articles | error | dotted leader to page numbers |
| `Credits auf medizinonline.com — …` line with `Zertifkat` typo | error | preserve typo |
| Body opens with ▪ glyph | error | structural |
| Voice-profile match (vs `--voice` flag or frontmatter `voice_profile`) | warning | hedge / metaphor / closer match |
| Closer phrase matches voice | warning | spec voice table |
| Two-line signature, no decoration | error | structural |

### 6. Cross-File Checks (when linting an issue)

| Check | Severity | Detail |
|---|---|---|
| Editorial references match CME articles in issue | error | header bar topic + page must align |
| MC question page covers exactly the issue's CME articles | error | `covers_articles` frontmatter must match files in `cme/` directory |
| Total page budget divisible by 4 | error | sum of `target_pages` across all files + ad allocations |
| Each Sonderreport's `kurzfachinformation_page` references a real page | error | structural |
| At least one News-Wissenschaft and one Markt & Medizin slot | warning | typical issue has both |

### 7. Schema Compliance

Run the same schema checks as `/kb-lint` Check D2 (per `~/.claude/skills/SCHEMA.md`):
- core fields present
- editorial-status workflow valid
- timestamps present and well-formed

### 8. Report

Print to stdout:

```
Style-Lint: {target}
─────────────────────────────────────────
{file 1}
  ✗ ERROR: {check} — {detail with line/match}
  ⚠ WARN:  {check} — {detail}
  ℹ INFO:  {check} — {detail}

{file 2}
  ✓ Clean.

…

─────────────────────────────────────────
Total: {N errors}, {N warnings}, {N infos} across {N files}.

Page budget (when issue-mode): {sum} pages — divisible by 4 ✓ / ✗ (delta {N}).
```

Also write a structured report to `outputs/{YYYY-MM-DD}-kb-style-lint-{target}.md`:

```markdown
---
type: style-lint-report
generated_at: {now}
target: {target}
magazine: {MAGAZINE_SLUG}
voice_profile: {profile if relevant}
lang: {DEFAULT_LANG}
---

# Style-Lint Report — {target}

## Summary
- Errors: {N}
- Warnings: {N}
- Infos: {N}

## Per-file findings
{...}

## Page budget
{...}
```

### 9. Commit (only the report — no source-file edits)

```bash
cd {KB_PATH} && git add outputs/ && git commit -m "kb: style-lint {target} ({N} errors)" 2>/dev/null || true
```

## Notes

- This skill **never** auto-fixes. It reports. Auto-fix is reserved for a future `/kb-style-fix` (post-MVP).
- Errors block publication. Warnings should be reviewed but do not block. Infos are improvement suggestions.
- Voice-profile matching is heuristic: it checks for presence/absence of metaphor density, hedge frequency, closer-phrase match. False positives expected; final judgement stays with the editor.
- Run as part of every issue's pre-print review: `/kb-style-lint NP3-26` before `/kb-issue-compose`.
