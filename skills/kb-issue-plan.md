---
name: kb-issue-plan
description: Generate or refine the Themenplan (issue plan) for a specific magazine issue (e.g., NP3-26). Surveys the wiki for sticky reader topics, current pharma sponsor pipeline, and upcoming congresses; produces a structured issue plan with section-level page allocation, topic list, candidate authors, and sponsor slots. Usage: /kb-issue-plan <magazine-slug> <issue-id> [--pages 44] [--editorial-voice burggraf|schliebe|blanke]
trigger: /kb-issue-plan
allowed-tools: Read, Write, Edit, Bash, WebFetch
---

# KB Issue Plan

Produce the Themenplan (issue plan) for a magazine issue, following the canonical magazine specification.

## Steps

### 1. Read Config + Magazine Spec + Mediadaten

Run in parallel:
```bash
cat ~/.claude/kb-config.json
cat ~/.claude/skills/SCHEMA.md
cat ~/.claude/skills/magazines/MAGAZINES.md
cat ~/.claude/skills/magazines/MEDIADATEN-2026.md
```

Determine `MAGAZINE_SLUG` from the first positional argument (or `default_magazine` from config). Then read the magazine-specific spec:
```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
```

Extract from the spec:
- Page architecture (default page count, section list, fixed vs. rotating sections)
- Article-type taxonomy
- Recurring sticky topics (spec §17)
- Pharma sponsor pool (spec §16)
- **Issue-planning methodology (spec §21)** — this is the master algorithm
- Resolved decisions (spec §19)

Extract from Mediadaten 2026:
- The row for the requested `<issue-id>` (e.g., NP3-26): Inserateschluss, Erscheinungsdatum, Kongresse, **Themenschwerpunkte** (the broad indication areas committed to advertisers)
- Auflage, Med. Herausgeber, Zielgruppe
- Standard rate-card numbers in case sponsor-slot suggestions need pricing context

If the issue-id is not in the Mediadaten table, refuse:
```
Error: <issue-id> not in Mediadaten 2026 publication plan. Available IDs: NP1-26 ... NP6-26 + SPECIAL Demenz / Depression & Angststörungen / Multiple Sklerose & ECTRIMS.
```

### 2. Parse Arguments

- `<magazine-slug>`: e.g., `info-np`. Required.
- `<issue-id>`: e.g., `NP3-26`. Required. Must match the magazine's issue-id pattern.
- `--pages <N>`: total pages (default from spec; must be divisible by 4)
- `--editorial-voice <profile>`: voice profile from spec (default: latest profile)
- `--cme-count <N>`: number of CME articles (default from spec)

### 3. Survey Wiki for Topic Signals

Read the wiki index and recent reflect reports:
```bash
cat {KB_PATH}/wiki/index.md
ls -t {KB_PATH}/outputs/*-kb-reflect-report.md 2>/dev/null | head -3 | xargs cat
```

Identify:
- Concept articles created or substantially updated in the last 60 days (proxy for hot topics)
- Sticky topics from the magazine spec that have new sources in the last 90 days
- Sources tagged with congresses upcoming or in the last 60 days

### 4. Survey Pharma Pipeline (optional, if WebFetch available)

For each Tier-1 / Tier-2 sponsor in the magazine spec, check for fresh:
- Swissmedic approvals (https://www.swissmedic.ch/swissmedic/de/home/ueber-uns/aktuell/medien.html)
- EMA / FDA recent approvals in the magazine's specialty area
- Upcoming or recent press releases

Skip this step if WebFetch is unavailable; flag in output that the pipeline survey was skipped.

### 5. Identify Upcoming Congresses

For the issue's release window (compute roughly: `NP3` ≈ June, `NP1` ≈ February, etc.), list relevant congresses:
- CH: SGN, SGPP, SSMG
- DE: DGN, DGPPN
- EU: EAN, ECTRIMS, ECNP
- US: AAN, AAIC, APA

Cross-reference against any congress information already captured in the wiki.

### 6. Allocate Sections

Using the magazine spec's page architecture, build a section budget for the target page count. For 44 pages of `info-np`:

```
Editorial         1
TOC               1
News (front)      1
CME-Fortbildung   ~12 (1 article × ~10p + Anleitung 1p + Fragen 1p, OR 2 × ~5p + Fragen 2p)
Medizin           ~18
Sonderreport/Publireportage  ~2
Praxismanagement  ~3
Weitere Rubriken (Board, AeB, Impressum, Das Letzte)  ~2
Anzeigen (U2, U3, U4, mid-book)  ~6
─────────────────
                  44 (must be divisible by 4)
```

Adjust per spec for the chosen page count.

### 7. Generate Topic List (per spec §21 methodology)

The spec §21 defines the master algorithm. Apply it:

**Step 7a — Mediadaten anchors**: take the broad Themenschwerpunkte for this issue from Mediadaten 2026. These are **locked** (committed to advertisers).

**Step 7b — Mediadaten anchors → specific subtopics**: for each broad area, run the signal scan from Step 4 + 5 + wiki signals from Step 3 to identify the **single most current specific subtopic**. Example for NP3-26 Multiple Sklerose anchor → "BTK-Inhibitoren in der MS — der erwartete Paradigmenwechsel" (driver: Tolebrutinib readouts).

**Step 7c — Cross-cutting / signal-driven topics** (the editor's value-add): add 5–10 topics outside the Mediadaten anchors based on:
- Recent Swissmedic / EMA / FDA approvals
- Landmark Phase III readouts (NEJM, Lancet, JAMA, Lancet Neurology, JAMA Psychiatry)
- Guideline updates (AWMF, EAN, AAN, NICE)
- New drug classes / launches
- Cross-specialty bridges (e.g., GLP-1 in Psychiatry)
- Regulatory changes affecting Swiss practice (REMS removals, label expansions)

**Step 7d — Each topic gets tagged**:
- `mediadaten_anchor` — the broad area, or `null` for cross-cutting
- `signal_driver` — specific reason this topic is hot now (approval / trial / guideline / launch / REMS / etc.)
- `congress_anchor` — the issue's Kongress(e) the topic ties to (or `null`)
- `proposed_section` — `cme | medizin-study | medizin-congress | medizin-review | praxismanagement | news-wissenschaft | markt-medizin | sonderreport`
- `sponsor_candidates` — pharma companies whose products appear in the topic (drives sponsor-slot booking; cross-reference with Tier-1/2/3 list in spec §16)

**Step 7e — Distribution to article types** (per spec §21.3):
- **2 CMEs**: pick the deepest, most clinically actionable, sponsor-alignable topics — typically one neurology + one psychiatry
- **8–14 Medizin articles**: rest of the locked topics, plus congress reports
- **2–4 News-Wissenschaft**: newest signals
- **2–4 Markt & Medizin**: tied to confirmed sponsors with current press releases
- **0–3 Sonderreport / Publireportage**: only confirmed bookings (status: `confirmed`); proposed slots use status: `proposed-pending-booking`
- **1–3 Praxismanagement**: practice-management-relevant topics (Burnout, Suchtprävention, EFAS, etc.)

**Status taxonomy per topic**:
- `locked`: committed (directly continuing a published wiki article, or already announced via prior issue's Das Letzte teaser, or anchored to a Mediadaten Themenschwerpunkt)
- `proposed`: candidate awaiting Chefredaktion approval
- `confirmed-booking`: sponsor slot with signed booking
- `proposed-pending-booking`: sponsor slot proposed to a candidate sponsor

### 8. Suggest Authors

For each proposed CME or external Medizin/Praxismanagement article:
- Search wiki sources for KOLs cited frequently in that topic (proxy for author candidacy)
- Search recent congress speaker lists in the topic area
- Note language preference (CH-DE primary, fall back to DE)
- Flag in-house byline as fall-back per spec §7.5 if no external candidate confirmed

### 9. Write the Issue Plan

Set `PLAN_FILE = wiki/issues/{ISSUE_ID}/issue-plan.md` (create directory if needed).

```markdown
---
type: issue-plan
magazine: {MAGAZINE_SLUG}
issue_id: {ISSUE_ID}
target_pages: {N}
editorial_voice: {voice}
lang: {DEFAULT_LANG}
editorial_status: draft
created_at: {now}
updated_at: {now}
tags: [issue-plan, {MAGAZINE_SLUG}, {ISSUE_ID}]
---

# Themenplan {ISSUE_ID}

> Auto-generated by /kb-issue-plan on {date}. Editor must review and lock topics before draft commissioning.

## Heftparameter
- Gesamtumfang: {N} Seiten (durch 4 teilbar ✓)
- Editorial-Stimme: {voice}
- CME-Artikel: {cme_count} ({page allocation})
- Sprache: Deutsch (Swiss)

## Seitenbudget
| Sektion | Seiten | Kommentar |
|---|---|---|
| Editorial | 1 | |
| Inhaltsverzeichnis | 1 | |
| News (Wissenschaft) | … | {count} Items |
| CME-Fortbildung | … | {topics} |
| Medizin | … | {topic count} Beiträge |
| Sonderreport / Publireportage | … | {sponsor candidates} |
| Praxismanagement | … | |
| Weitere Rubriken | … | |
| Anzeigen | … | U2, U3, U4 + Inserate |
| **Total** | **{N}** | |

## Themenliste

### CME-Fortbildung
{For each CME slot:}
- **{Topic}** — Status: {locked|proposed} | Autor*in-Vorschlag: {name+affiliation} | Sponsor: {company or TBD} | Begründung: {why now (wiki signal / congress / approval)}

### Medizin
{For each Medizin article:}
- **{Topic}** ({Type: study|congress|review}) — Status: {locked|proposed} | Autor*in: {staff/external} | Quelle: {primary source}

### Praxismanagement
{For each PM article:}
- **{Topic}** — Status: {locked|proposed} | Autor*in: {…}

### News (Wissenschaft)
{2–4 candidate items, each with topic + likely source}

### Markt & Medizin
{2–4 candidate items, each with sponsor + product + recent press release reference}

### Sonderreport / Publireportage
{For each sponsored slot:}
- **Sponsor**: {company} | **Produkt**: {brand (INN)} | **Format**: {sonderreport|publireportage} | **Status**: {confirmed|proposed} | **Hinweise**: {disclosure language, KOL availability, reprint chain}

## Kongressbezug
- Erscheinungsfenster: {month-year}
- Relevante Kongresse: {list with dates}

## Offene Punkte für Chefredaktion
1. {Per the magazine spec §19 open issues — list any that affect this plan}
2. {Decisions needed before draft commissioning}

## Buchungsversand-Text (Kunden / Agenturen)
{1–2 paragraphs in marketing tone summarising the top 3–5 themes for the issue, suitable for sending to Kunden/Agenturen as Buchungsgrundlage. Per the process docs (Mediadaten / Heftplan): "Konkretisierung der Themenliste auf Basis aktueller Literatur, bevorstehender Kongresse und neuer wissenschaftlicher Entwicklungen".}
```

### 10. Update Wiki Index

Add under `## Hefte / Issues` (create the section if missing):
```
- [[issues/{ISSUE_ID}/issue-plan]] — Themenplan {ISSUE_ID}: {N} Seiten, {short topic preview} `[draft]`
```

### 11. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: issue plan for {MAGAZINE_SLUG} {ISSUE_ID}"
```

### 12. Print Summary

```
Themenplan {ISSUE_ID} erstellt.
  Datei: {PLAN_FILE}
  Umfang: {N} Seiten
  Editorial-Stimme: {voice}
  CME-Themen: {N}
  Medizin-Beiträge: {N}
  Sonderreport-Slots: {N}

Nächste Schritte:
  1. Editor reviewt Themenplan, markiert „proposed" Themen als „locked".
  2. Pro CME: /kb-cme-draft {issue}/{topic-slug}
  3. Pro Medizin-Beitrag: /kb-medizin-article {issue}/{topic-slug}
  4. Sonderreports: /kb-sonderreport {issue}/{sponsor-slug}-{product-slug}
  5. Editorial: /kb-editorial {issue} --voice {voice}
  6. Heftzusammenstellung: /kb-issue-compose {issue}
```

## Notes

- A Themenplan is always `editorial_status: draft` until the Chefredaktion explicitly promotes it via `/kb-review`.
- Re-running `/kb-issue-plan` on an existing issue refreshes signals but **never overwrites** locked topics — locked entries are preserved verbatim.
- The plan is the contract for downstream skills: `/kb-cme-draft` reads the plan to know which topic + author to draft, etc.
- For the first issue in a new magazine spec, the plan also provides the seed for Mediadaten — the annual plan extending across all 6 issues of a year.
