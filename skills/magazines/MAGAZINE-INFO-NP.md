# MAGAZINE-INFO-NP — Production Specification

Canonical production specification for **InFo Neurologie & Psychiatrie**, reverse-engineered from issues NP1-23 through NP1-26 (18 issues, 2023–early 2026). This document is the ground-truth reference for all magazine-production skills (`/kb-issue-plan`, `/kb-cme-draft`, `/kb-mc-questions`, `/kb-news-item`, `/kb-medizin-article`, `/kb-sonderreport`, `/kb-editorial`, `/kb-issue-compose`).

> **Source of truth:** the 4 reverse-engineering reports under `docs/reverse-engineering/info-np/`. When this spec and a report disagree, the report is more recent — flag the discrepancy in `/kb-style-lint` rather than guessing.

---

## 1. Identity

| Field | Value |
|---|---|
| Title | InFo Neurologie & Psychiatrie |
| Publisher | Prime Public Media AG, Neugasse 10, 8005 Zürich |
| ISSN | 1661-2671 |
| Frequency | 6 issues / year (NP1 through NP6) |
| Volume | year − 2002 (e.g., 2026 = Vol. 24) |
| Print run | ~6 000 copies |
| Issue identifier | `NP{N}-{YY}` (e.g., `NP3-26` = issue 3 of 2026) |
| Specialties | Neurology + Psychiatry (single dual-specialty title) |
| Audience country tilt | Switzerland primary; DACH secondary |
| Editorial language | German (Swiss orthography). Rare French CME pieces for francophone CH (e.g., NP6-24 p.21). |
| CME accreditation pipeline | medizinonline.com (Prime Public Media's own platform). The accrediting Fachgesellschaft is not printed on paper. |

---

## 2. Page architecture (anatomy of an issue)

Standard 44-page issue, divisible by 4 (prints observed: 40, 44, 48, 52, 56). Modal length: 44.

| # | Section (DE) | Position | Typical extent | Articles | Fixed? |
|---|---|---|---|---|---|
| 1 | Cover (Umschlag U1) | p.1 | 1 p | — | ✓ |
| 2 | Inside-front-cover ad (U2) | p.2 | 1 p | 1 ad | ✓ |
| 3 | EDITORIAL | p.3 (= printed p.1) | 1 p | 1 | ✓ |
| 4 | INHALTSVERZEICHNIS | p.4 (= p.2) | 1 p | — | ✓ |
| 5 | News (Wissenschaft) | early + scattered | 1–2 p | 4–8 short items | ✓ |
| 6 | CME-FORTBILDUNG | early-front, ~pp.6–17 | ~12 p | 1–2 articles + Online-CME instructions + Fragen page | ✓ |
| 7 | MEDIZIN | middle, ~pp.18–36 | 16–24 p | 8–14 articles | ✓ |
| 8 | SONDERREPORT / PUBLIREPORTAGE / MARKT & MEDIZIN | interspersed | 1–4 p per item | 0–3 per issue | rotating |
| 9 | PRAXISMANAGEMENT | back, pp.37–40 | 2–4 p | 1–3 articles | ✓ |
| 10 | Board / Auf einen Blick / Impressum | back | 1 p each | — | mostly ✓ |
| 11 | Das Letzte | back (intro NP5-24) | ½–1 p | 0–1 | new from 2024 |
| 12 | U3 / U4 ads | last 1–3 p | 2–3 p | ads | ✓ |

**Cover anomaly NP1-26:** page 1 is a full-page Kesimpta ad, the actual teaser cover frame moves to p.3. Treat this as a configurable `cover_mode: "teaser"` (default) vs. `"sponsored-takeover"` flag in the issue plan.

**CME volume reduction from NP5-25:** issues now ship with 1 CME article instead of 2. Skill defaults: 1 CME, but accept `--cme-count 2` flag.

---

## 3. Cover design pattern

Default ("teaser" mode):
- Top thin band: `Ausgabe {N} · {Monat YYYY} · www.medizinonline.com`
- Single full-bleed photo (mostly iStock)
- Logo wordmark "InFo Neurologie & Psychiatrie"
- 3-column teaser block titled by section: **CME-FORTBILDUNG** | **Medizin** | **Praxismanagement**
- 6–8 teasers per cover; each = topic-kicker (bold) + 1–2-line headline
- Image credit on TOC page: `Titelbild: <credit>, iStock`

URL change: medizinonline.ch → medizinonline.com from NP3-23 onward. Use `.com`.

---

## 4. TOC pattern

- Title: `INHALTSVERZEICHNIS`
- Section heads in CAPS, fixed order: `EDITORIAL` → `CME-FORTBILDUNG` → `(SONDERREPORT/PUBLIREPORTAGE)` → `MEDIZIN` → `PRAXISMANAGEMENT` → `WEITERE RUBRIKEN`
- Each entry: page number (left) + kicker line + headline (1–2 lines) + author/affiliation in italics where applicable
- `WEITERE RUBRIKEN` block at bottom: comma-separated page lists (e.g. `News  4, 5, 30, 33, 38`)
- Footer line: `Titelbild: <credit>, iStock`

---

## 5. Editorial pattern

| Aspect | Rule |
|---|---|
| Length | exactly 1 page (~350–500 words) |
| Header bar | `Die Fortbildungsthemen in dieser Ausgabe:` + dotted page-references to CME articles + line `Credits auf medizinonline.com — Einloggen, Fragen beantworten und direkt CME-Zertifkat downloaden.` (note: the typo `Zertifkat` is a stylistic fingerprint — preserve it). When only 1 CME, use singular `Das Fortbildungsthema in dieser Ausgabe:`. |
| Kicker | Small, e.g. "Subdurales Hämatom" |
| Headline | Bold, 1 line, often punning or culturally referenced (Burggraf-era) or imperative (Blanke-era) |
| Body opening | Black square ▪ glyph followed by a vignette (cultural / historical / news anecdote) |
| Body | Essayistic; 2–4 paragraphs; bridges from vignette to thematic relevance for the issue |
| Sign-off | One of three **voice profiles** (see below) — name + role on two lines, no title prefix below the line, no portrait, no email (until 2025 — Blanke-era starts including email) |
| Closer phrase | Burggraf: `Wir wünschen viel Spass bei der Lektüre!` / `Viel Freude bei der Lektüre.` Schliebe + Blanke: `Ich wünsche Ihnen eine erkenntnisreiche Lektüre der vorliegenden Ausgabe der InFo Neurologie & Psychiatrie.` |

### Voice profiles (configurable per issue)

| Profile | Active period | Tone | Headline pattern | Closer |
|---|---|---|---|---|
| `burggraf` | NP1-23 → NP1-25 | Warm / colloquial / metaphor-driven (e.g., "Tau-frisch im Alter!?", "Ein Indianerherz kennt keinen Schmerz – oder doch?") | Word-play, cultural reference, often question | "Viel Spass / viel Freude" |
| `schliebe` | NP2-25 → NP4-25 | Sober / academic / evidence-anchored ("Damit gilt als plausibel, dass …") | Descriptive | "Ich wünsche Ihnen eine erkenntnisreiche Lektüre …" |
| `blanke` | NP5-25 → NP1-26 | Wry / colloquial / occasionally satirical (e.g., "biokular schauen", "Lernen wir schielen") | Imperative / programmatic ("Zeit zum Schielen: Mensch vs. Algorithmus") | "Ich wünsche Ihnen eine erkenntnisreiche Lektüre …" |

---

## 6. Article-type taxonomy & frontmatter

Every article carries the schema base (`lang`, `tags`, `created_at`, `updated_at`, `editorial_status`) plus type-specific fields.

| Type | `magazine_section` | Length | Byline | Photo | Refs |
|---|---|---|---|---|---|
| `editorial` | EDITORIAL | 1 p | yes (Redaktion) | no | no |
| `cme` | CME-FORTBILDUNG | 4–8 p (typ. 6) | yes (external Prof./Dr. or in-house) | sometimes | Vancouver `[n]` |
| `cme-mc` | CME-FORTBILDUNG | 1 p | none | no | no |
| `medizin-study` | MEDIZIN | 1–2 p | staff initials `(lb)`/`(ub)` | iStock | Vancouver |
| `medizin-congress` | MEDIZIN | 1–4 p | staff initials | iStock | numbered + KONGRESS banner |
| `medizin-review` | MEDIZIN | 2–4 p | external author | author | Vancouver |
| `news-wissenschaft` | NEWS | ¼–½ p | staff initials | small icon | `Quelle:` line |
| `news-markt-medizin` | NEWS | ¼–½ p | tag `MARKT & MEDIZIN <Company>` | small icon / product | `Quelle: <press release>` |
| `sonderreport` | SONDERREPORT | 1–4 p | sponsor + writer | KOL photo | yes + Kurzfachinformation |
| `publireportage` | PUBLIREPORTAGE | 1–2 p | sponsor copy | product image | sometimes + Kurzfachinformation |
| `praxismanagement` | PRAXISMANAGEMENT | 1–3 p | staff or external | stock | numbered |
| `auf-einen-blick` | WEITERE RUBRIKEN | 1 p (infographic) | — | yes | — |
| `das-letzte` | WEITERE RUBRIKEN | ½–1 p | — | — | — |
| `kongress-anzeige` | ad | 1 p | — | logo | — |
| `pharma-anzeige` | ad | 1 p | pharma | product | references in fine print + Kurzfachinformation |
| `freianzeige` | ad | ¼ p | charity / society | — | donation account info |

---

## 7. CME-Fortbildung — full specification

### 7.1 Frontmatter
```yaml
type: cme
magazine: info-np
magazine_section: cme-fortbildung
lang: de  # or fr (rare; e.g., NP6-24)
editorial_status: draft
sponsor: <pharma-company-name>  # required; CMEs are pharma-funded
keywords: [keyword1, keyword2, keyword3, keyword4]  # 3–4, pipe-separated when rendered
target_pages: 6  # 4–8 typical
mc_question_count: 6  # 6–8
authors:
  - name: Prof. Dr. med. Heinz Reichmann
    affiliation: Direktor, Klinik für Neurologie, Universitätsklinikum Dresden
    address: Fetscherstraße 74, D-01307 Dresden
    email: name@example.com
    photo: present|absent
references: []  # Vancouver-style with DOIs
```

### 7.2 Mandatory body skeleton (in order)

1. **Headline** — short topic noun (e.g., "Palliative Care", "Neuroenhancement")
2. **Subheadline** — creative, descriptive ("Eine zunehmende Herausforderung für die Ärzteschaft", "Kann man Intelligenz schlucken?")
3. **Keyword line** — italics, pipe-separated 3–4 keywords
4. **Author block** — top-right: title + name + position + institution + postal address + email
5. **Lead paragraph** — opens with ▪ glyph, hooks with clinical/societal context
6. **Body sections** with bold subheads (recurring sequence — adapt to topic):
   - Definition / Epidemiologie / Prävalenz
   - Pathophysiologie / Pathogenese
   - Klinik / Diagnostik / Differenzialdiagnosen (with **Fallbeispiel** vignette for clinical topics)
   - Therapie (drug classes individually; comparison tables)
   - Praktisches Vorgehen / Anwendung in der Praxis
7. **Tables / figures** — `Tab. 1`, `Abb. 1`, `Übersicht 1`. Caption format: bold descriptive title, then explanatory sentence, then `modifiziert nach [n]` source. Image credit microtext: `iiievgeniy, istock`.
8. **TAKE-HOME-MESSAGES** — small-caps header, yellow box, 3–5 em-dash bullets `―` (NEVER `•`). **Mandatory.**
9. **Fazit / Konklusion** — 1 paragraph closing
10. **Sponsor disclosure** — verbatim formula (German):
    ```
    Dank des Sponsorings der Firma <X> ist die Teilnahme an dieser Fortbildung für Sie kostenlos.
    Die Fortbildung ist von Fachleuten erstellt und unter unbedingter Wahrung unserer redaktionellen
    Freiheit erstellt. Zu keinem Zeitpunkt hatte und hat die Firma Einfluss auf die Inhalte der
    Fortbildung. Alle Texte unterliegen lediglich der wissenschaftlichen Hoheit der Autoren und
    der Redaktion von PPM MEDIC.
    ```
    French parallel form available (NP6-24 p.21 template).
11. **Literatur** — Vancouver-style numbered. Format examples:
    ```
    1. Stocchi F, Antonini A, Barone P, et al.: Park Relat Disord 2014; 20: 204–211.
    2. Repantis D, Bovy L, Ohla K, et al.: Cognitive enhancement effects of stimulants … Psychopharmacology 2021; 238(2): 441–451. doi:10.1007/s00213-020-05722-6
    ```
    DOIs when open-access; **PMIDs are not used in print** (we still store them in our wiki for traceability).
12. **Vertical sidebar callout** on first page: `> Fortbildungsfragen auf Seite <n>`

### 7.3 What is NOT printed

- No `Lernziele:` header on paper (objectives are implicit in MC questions; we still draft them internally for the MC author and store in frontmatter as `learning_objectives:`).
- No author COI / `Interessenkonflikt:` section in print (we still capture in frontmatter as `coi:` for compliance archive).
- No CME-points number, no accrediting body name on paper.
- No answer key (resolved only via medizinonline.com).
- No `level of evidence (1A/1B)` callouts — Swiss-DE convention does not adopt the formal grading boxes.
- No URLs in body except for guidelines / registries (AWMF, NICE, BAG).

### 7.4 Tone

- Impersonal didactic: "Bei älteren Patient:innen >70 Jahre wird in der Regel …"
- "Sie"-form rarely; mostly third-person.
- Sentence length 18–25 words, German-academic.
- AWMF S3-Leitlinien, EAN/EFNS, NICE cited as evidence anchors.
- Hedging mandatory: "möglicherweise", "die Daten deuten darauf hin", "scheinen". Never: "bahnbrechend", "Game-Changer", "revolutionär" — except inside quotation marks attributing to a named expert.

### 7.5 Author profile expectations

- ~70% university-hospital senior physicians (Oberarzt, Leitender Arzt, Direktor, Professor)
- ~20% in-house Redaktion (when external recruitment falls short — Burggraf, Schliebe, Blanke)
- ~10% specialty consultants / private institutions
- 55% CH, 35% DE (DE authors marked with `(D)` postal address), rare AT/EU
- Title prefix in author block: `Dr. med.` (clinician), `Prof. Dr. med.` (academic), `PD Dr. med.` (Habilitierte), `Dr. phil.` (psychologist), append `M.Sc.`/`MBA`/`MHBA`/`Dr. oec.` where applicable.

---

## 8. CME-Fortbildungsfragen (MC quiz page)

### 8.1 Position
At the end of the CME section, after both articles. One page (NP3-25 p.16, NP1-23 p.17, NP1-26 p.18).

### 8.2 Required header
- Yellow-to-grey decorative band labelled `> Fortbildungsfragen`
- Topic line below band, repeating the article title(s)
- Required-articles footer: `Zur Beantwortung der Fragen sind folgende Artikel erforderlich: <article 1 title>, <article 2 title>`

### 8.3 Per-question structure
- **Stem** ending with mode marker in italics:
  - `(gesuchte Antwort ankreuzen)` — single best answer
  - `(alle gesuchten Antworten ankreuzen)` — multi-select
- **Options**: 4 (A/B/C/D) typical; 3 (A/B/C) acceptable; one correct in single-best, ≥1 correct in multi-select
- **Bloom level**: predominantly recall (1) and comprehension (2). Vignette-based application (level 3) is the trend from NP1-26 onwards.
- **Distractors**: real medical concepts within the topic; never gibberish.

### 8.4 Question budget per issue
- 1 CME article: 6–8 questions
- 2 CME articles: 11–14 questions in a combined set (e.g., NP3-25 ALS 8 + Autismus 3 = 11)

### 8.5 Mode mix
- ~60% single-best, ~40% multi-select (observed mean across issues).

### 8.6 Vignette example (NP1-26 Q5)

> **Ein 22-jähriger Student nimmt seit einigen Monaten Amphetamine (Adderall) zur Prüfungsvorbereitung. Welches Risikoprofil ist bei dieser Substanzklasse besonders relevant?** *(gesuchte Antwort ankreuzen)*
> A) Selektive Beeinträchtigung des Langzeitgedächtnisses bei guter Verträglichkeit.
> B) Niedriges Abhängigkeitspotenzial bei moderatem kardiovaskulärem Risiko.
> C) Hauptsächlich gastrointestinale Nebenwirkungen ohne relevantes Suchtpotenzial.
> D) Hohes Abhängigkeitspotenzial, kardiovaskuläre Risiken und Gefahr psychiatrischer Komplikationen wie Paranoia.

### 8.7 Online-CME instructions page
Adjacent page carrying the standard instruction block ("Aktivieren Sie auf medizinonline.com Ihren kostenlosen Account …").

---

## 9. News — Wissenschaft

### 9.1 Structure
- ~250–450 words per item, 2–3 items per page
- Body opens with ▪ glyph, optionally followed by author initials in parens: `■(ub)`, `■(lb)`
- Headline: two-deck (kicker + main, 4–9 words main)
- Lead-Box (Standfirst): 3–5 sentences in distinct frame above body summarising the finding

### 9.2 Source attribution (mandatory)
At end of item:
```
Quelle: «<press release title>», DD.MM.YYYY, <Institution> (<country code if foreign>)
```
Examples:
- `Quelle: «Neue Hoffnung auf Therapie bei seltener Stoffwechselerkrankung», 20.01.2023, Universität Leipzig (D)`

### 9.3 Tone rules
- Impersonal, neutral-clinical; no "wir" or "Sie"
- Hedging present ("möglicherweise", "deuten darauf hin", "scheinen")
- INN drug names only; brand-first signals sponsored content
- Press releases never copied verbatim — always rewritten in magazine voice
- Konjunktiv I/II for indirect speech: "Die Wissenschaftler vermuten, dass …"

### 9.4 Source preference
Process docs: `idw-online.de` (university press releases) is the canonical source for Wissenschafts-News.

---

## 10. News — Markt & Medizin

### 10.1 Format
- Tag line at top: `MARKT & MEDIZIN   <Company Name>` (the company is the sponsor of the slot, also the source)
- ~250–450 words, 2–3 per page
- Brand name allowed prominently (this is the differentiator from editorial News)
- Source: `Quelle: <pharma press release title>, DD.MM.YYYY`
- No `Anzeige` tag, no Kurzfachinformation block (these would mark a full pharma ad)
- Tone: neutral-positive; product-positive but no comparative slamming

### 10.2 Source preference
Direct pharma press releases. Cross-checked with Swissmedic / EMA / FDA filings when regulatory news.

### 10.3 Content boundary
Editorial News may cover the same drug as a Markt & Medizin item in the same issue — this is allowed. The differentiator is the tag line + the use of brand vs. INN as the lead.

---

## 11. Medizin section

Three sub-types share the section header `MEDIZIN`:

### 11.1 Study summary (`medizin-study`)
- 1–2 pages
- Staff byline `(lb)`, `(ub)`, etc.
- Source: PubMed free full text or university press release
- Vancouver references at end
- Lead-Box of 3–5 sentences

### 11.2 Congress report (`medizin-congress`)
- 1–4 pages
- Banner `KONGRESS DGN 2023` (or EAN, AAN, ECTRIMS, AAIC, etc.) at bottom-right of pages
- Often features a short interview with a referent
- Reference style: numbered abstracts (e.g., `Abstract 40. 96. Kongress der Deutschen Gesellschaft für Neurologie. 8.–11. November 2023.`)

### 11.3 External review (`medizin-review`)
- 2–4 pages
- External author (Prof./Dr. external)
- Author photo + affiliation
- Vancouver references

---

## 12. Sonderreport / Publireportage

### 12.1 Top-of-page label
- `PUBLIREPORTAGE` for advertorial-style sponsored content (lighter, often single-page, product-tied)
- `SONDERREPORT` for longer multi-page sponsored content (often a congress reprint or KOL Q&A)
- `MARKT & MEDIZIN <Company>` is a separate, lighter category — see §10
- All labels: capitalised, top of page, distinct coloured strip from the grey `MEDIZIN` / `NEWS` strip

### 12.2 Mandatory frontmatter
```yaml
type: sonderreport  # or publireportage
magazine: info-np
magazine_section: sonderreport
sponsor:
  legal_name: Biogen Switzerland AG
  address: Neuhofstrasse 30, CH-6340 Baar
  approval_code: Biogen-264206_04.2025
product:
  brand: Skyclarys™
  inn: Omaveloxolon
  indication: Friedreich-Ataxie
medical_writing_cro: Cactus Life Sciences Switzerland AG  # if used
medical_writers: [Dr. Lea Furer, Dr. Sonja Mariotti Nesurini]  # if applicable
kol:  # optional
  name: PD Dr. med. G. Schoretsanitis
  affiliation: PUK Zürich
references: []
kurzfachinformation_page: same  # or page number reference: "Seite 27"
```

### 12.3 Required disclosure phrases

Pick the appropriate one — exact wording matters:

| Format | Use when |
|---|---|
| `Mit freundlicher Unterstützung der Firma <X>.` | Lightweight, especially reprints |
| `Dieser Beitrag wurde finanziert von <X>.` | Standard Sonderreport |
| `Dieser Artikel entstand mit finanzieller Unterstützung von <X>.` | Publireportage |
| `Die Medical-Writing-Unterstützung wurde durch <Author1>, <Author2> (<CRO Name>) im Auftrag von <Sponsor> erbracht.` | When external medical-writing CRO involved |
| `Die hier getätigten Aussagen spiegeln die professionelle Meinung von <Name> wider.` | KOL interview format |

### 12.4 Mandatory footer block
```
<Sponsor legal name>
<Swiss address>
<Approval/clearance code in format: <CompanyTag>-<digits>_<MM.YYYY> | MAT-CH-<digits>_<MM/YYYY> | CH_CP-<digits>_v<x>.<y>>
```

### 12.5 Kurzfachinformation
Tiny-print fact sheet on same page or at referenced page. Required fields: `Z` (Zusammensetzung) / `I` (Indikation) / `D` (Dosierung) / `KI` (Kontraindikationen) / `VM` (Vorsichtsmassnahmen) / `IA` (Interaktionen) / `SS/St` (Schwangerschaft/Stillzeit) / `UAW` (Unerwünschte Arzneimittelwirkungen) / `P` (Packungen) / `Abgabekat.` / `Stand der Information`.

### 12.6 Off-label rule
**Never** advance off-label use in sponsored content. Editorial may discuss off-label only in clinical-context articles, never in product-tied pieces.

### 12.7 Reprint chain (for Sonderreport reprints from BrainMag / medEdition Verlag / ECTRIMS Special)
```
© medEdition Verlag GmbH, Hirzel <year>. Abdruck mit freundlicher Genehmigung des Verlags.
Erstpublikation erschienen in BrainMag <year>; <issue>: <pages>.
```

### 12.8 Copyright
Sonderreport copyright belongs to **Prime Public Media AG**, not the sponsor. The publisher owns the output even when the sponsor commissioned it.

---

## 13. Praxismanagement

- Topics: insurance/EFAS, cyber-security, suicide prevention, Long-Covid, public health, Suchtprävention, Organspende, Burnout
- Position: pp.37–40 (back of book)
- 1–3 pages
- Author: staff (initials) or external (named)
- Refs: numbered

---

## 14. Tone of voice — full rule set

### 14.1 Person & address
- "Sie" → only Editorial and CME framing sentences
- "Wir" inclusive → only Editorial (especially Blanke-era)
- Body of News / Medizin → impersonal third person, never "wir", never "ich"
- Author byline initials in parentheses signal news-summary authorship: `(ub)` Blanke, `(lb)` Burggraf, etc. Always preceded by ▪ glyph at body start.

### 14.2 Tense
- Present for explanatory passages
- Perfect / Präteritum for trial reports ("wurden eingeschlossen", "konnte gezeigt werden")
- Konjunktiv I/II for indirect speech reporting expert claims

### 14.3 Sentence length
- 18–25 words typical; long German-academic sentences with nested subordinate clauses normal
- Editorials slightly shorter, especially Blanke

### 14.4 Hedging discipline
**Mandatory** for findings: "möglicherweise", "die Daten deuten darauf hin", "es ist plausibel, dass", "scheinen", "könnte"

**Forbidden in editorial voice** (only in attributed quotes):
- "bahnbrechend"
- "Game-Changer"
- "revolutionär"
- "Meilenstein" (use sparingly, with hedge)

### 14.5 Citation style

**News:** no inline cites; end with `Quelle: «<title>», DD.MM.YYYY, <Institution>`.

**CME / Medizin:** numbered Vancouver brackets `[1]`, `[2,3]`, `[4,15]` inline.

**End-of-article reference list:** Vancouver/AMA hybrid:
```
1. Whitebrook J, et al.: A suicide bereavement model …, Front Public Health, 2025; 13: 1596961.
2. Repantis D, Bovy L, Ohla K, et al.: Cognitive enhancement … Psychopharmacology 2021; 238(2): 441–451. doi:10.1007/s00213-020-05722-6
```

DOIs when open-access; **never use PMIDs in print**. NCT trial registry numbers occasionally; AWMF/NICE URLs with `(letzter Zugriff: TT.MM.JJJJ)`.

### 14.6 Drug-name handling

| Context | First mention | Subsequent |
|---|---|---|
| Editorial / CME / News-Wissenschaft | INN (e.g., Ocrelizumab) | INN |
| News-Wissenschaft (clarification) | INN, optionally `(Ocrevus®)` once | INN |
| Markt & Medizin | brand prominent (LEQEMBI IQLIK™), INN in passing | brand |
| Sonderreport / Publireportage | brand with ®/™ (Skyclarys™), INN in apposition | brand |

Class in apposition mandatory: "Lixisenatid. Es gehört zur Substanzklasse der GLP-1-(Glucagon-like Peptid-1) Rezeptoragonisten (GLP-1-Analoga)".

### 14.7 Numbers, units (Swiss conventions)

| Element | Rule | Example |
|---|---|---|
| Decimal separator | comma | `1,5 mg` |
| Thousands separator | thin space | `100 000` (NOT `100.000` or `100,000`) |
| Percent | digit + `%` no space, OR `Prozent` spelled out | `46%`, `70 Prozent` |
| Confidence interval | `95%-KI` | `(95%-KI −0,62 bis −0,06)` |
| p-value | `p = 0,001` | (no `p<0.05`) |
| Effect size | `HR 1,29; p=0,04`, `Cohen's d = −0,06` | |
| Dosage | μ allowed | `20 μg`, `0,5 mg/kg`, `100 mg i.v.` |
| Sample size | `n=78` or `n = 1554` | |
| Range | en-dash `–` | `1905–1929`, `3–5 Monate` |
| Currency | Swiss Franken context, write `CHF` | |

### 14.8 Orthography
- Swiss: `ss` instead of `ß` always (even in proper names not on the magazine's own pages — e.g., `Frauenklinikstrasse`)
- German-affiliated authors keep `ß` in their address only (`Reinhardtstraße 27 C, D-10117 Berlin`)
- Anführungszeichen: «...» (Guillemets); not `"..."` or `„..."` 

### 14.9 Visual conventions

| Element | Wording |
|---|---|
| Figure ref | `Abb. 1` (NOT `Fig.` or `Bild`) |
| Boxed list | `Übersicht 1` |
| Table | `Tab. 1` |
| Take-Home box header | `TAKE-HOME-MESSAGES` (small caps) |
| Take-Home bullets | em-dash `―` (NOT `•`) |
| Internal cross-ref | `(Abb. 1)`, `(Übersicht 1)`, `wie in Abbildung 5 gezeigt` |
| Image credit microtext | `iiievgeniy, istock` (lower-case, italic) |

### 14.10 Patient nomenclature
- 2023: `Patienten` (generic masculine)
- 2024+: gender-aware variants present, **inconsistent** within issues:
  - `Patient*innen`, `Patient:innen`, `Patientinnen und Patienten`
  - **Pending house-style decision** — until then, default to `Patientinnen und Patienten` (longest, most formal, present in latest issue NP6-25)
- For `Personen mit <condition>` style (people-first language), allow when the source uses it

### 14.11 Recurring rhetorical templates (verbatim)

| Slot | Template |
|---|---|
| Editorial closer (Burggraf) | `Wir wünschen viel Spass bei der Lektüre!` / `Viel Freude bei der Lektüre.` |
| Editorial closer (Schliebe / Blanke) | `Ich wünsche Ihnen eine erkenntnisreiche Lektüre der vorliegenden Ausgabe der InFo Neurologie & Psychiatrie.` |
| News opener archetype | `■In einer Studie wurden …` / `■Forschende des … haben …` / `■Eine [britische / amerikanische / interdisziplinäre] Forschergruppe hat nun …` |
| Trial result phrasing | `In der …-Studie konnte gezeigt werden, dass …` |
| Hedged finding | `Die Daten deuten darauf hin, dass …` |
| Editorial signature | `Leoni Burggraf, Redaktion` / `Tanja Schliebe, Chefredaktion` / `Dr. Ulf Blanke, Redaktion` (two-line, no decoration) |
| Source attribution | `Quelle: «<title>», DD.MM.YYYY, <Institution>` |
| CME teaching teaser link | `> Fortbildungsfragen auf Seite <n>` |
| KONGRESS banner | `KONGRESS   DGN 2023` (uppercase, two-space gap) |
| Take-home bullet | `― <statement>` |

---

## 15. Page-budget arithmetic

- Total pages must be divisible by 4 (print constraint)
- Modal: 44 (12 of 19 sampled issues)
- Distribution per 44-page issue (rough averages):
  - Editorial 1 + TOC 1 + CME 12 + Medizin ~18 + Sonderreport/Publireportage ~2 + Praxismanagement ~3 + News/Board/Auf-einen-Blick ~2 + Impressum ½ + Das Letzte ½ + ads (covers + interior) ~6 = 44
- Ad share: 14% pure ads + 11–16% sponsored editorial = ~25–30% commercial
- Pure unpaid editorial: ~70–75%
- Issues sometimes go 48 (NP1-26), 52 (NP3-23), 56 (NP5-25). Always +4 increments.

---

## 16. Pharma sponsor pool (recurring)

Tier-1 anchors (every / nearly every issue):
- **Roche** (Ocrevus / Ocrelizumab in MS — multi-page placements)
- **Biogen** (Tysabri / Natalizumab; Skyclarys / Omaveloxolon for FA; Vumerity)
- **Sandoz** (generics: Sertralin, Lacosamid, Zonisamid)
- **Schwabe Pharma** (phyto: Lavendelöl/Laitea/Silexan, Ginkgo/Tebokan, Dormiplant)

Tier-2: Novartis (Kesimpta), Janssen-Cilag (Spravato/Esketamin, Trevicta/Xeplion), Lundbeck (Vyepti), Eli Lilly (Emgality), Bial (Ongentys), BMS (Zeposia), Sanofi (Aubagio), Jazz (Wakix/Sunosi/Xywav), Merck KGaA (Mavenclad), Teva (Ajovy), UCB (Vimpat/Briviact).

Tier-3 occasional: Pfizer, AbbVie, Novo Nordisk (Wegovy — yes, in this neuro/psych journal), Idorsia (Quviviq), Eisai (LEQEMBI), Stoke Therapeutics, Recordati, Neuraxpharm.

4–8 distinct pharma sponsors per typical issue (display ads + sponsored content + Markt & Medizin items).

---

## 17. Doctor's choice — recurring sticky topics

Multi-year recurring topics (proxy for reader interest):
- **MS DMTs** — every issue (Ocrelizumab, Cladribin, Ofatumumab, Diroximelfumarat, Ozanimod, Teriflunomid, Natalizumab)
- **Migraine prophylaxis** — CGRP class (Eptinezumab, Erenumab, Fremanezumab, Galcanezumab; Gepants)
- **ALS gene therapy** — Tofersen, Riluzol
- **Alzheimer anti-amyloid** — Lecanemab, Aducanumab
- **Insomnia / orexin antagonists** — Daridorexant, Suvorexant
- **Narcolepsy** — Pitolisant, Solriamfetol
- **Schizophrenie LAI** — Paliperidonpalmitat
- **ADHS adult** — multiple substance classes
- **AI / digital** — trending up (4× CME from 2023→2026: KI in der Medizin, KI in der Neurologie, ParkAI etc.)

---

## 18. Quality watermarks ("feels like InFo NP")

Concrete tells that mark a piece as fitting the publication:

1. Black-square ▪ + parenthetical author initials at body start
2. Standfirst-box of 3–5 sentences immediately under kicker+headline
3. Swiss number formatting (comma decimal, thin-space thousands, no ß)
4. `Quelle: «...», DD.MM.YYYY, Institution (D)` with country tag for foreign sources
5. `TAKE-HOME-MESSAGES` with em-dash bullets at end of CME
6. INN-first drug naming with class in apposition
7. Editorial signature `[Name], Redaktion / Chefredaktion` two-line, no titles below
8. Hedged tone: `deuten darauf hin`, `könnten`, `scheinen`
9. French Guillemets «...» for quoted speech
10. `Wir wünschen [eine spannende / erkenntnisreiche] Lektüre` editorial close

---

## 19. Open issues (require human editor decision)

These cannot be resolved from the PDFs alone — they should be settled before full automation:

1. **Patient*innen vs. Patient:innen vs. Patientinnen und Patienten** — inconsistent across same year. House style needs decision.
2. **CME accreditation pipeline** — which Schweizer Fachgesellschaft awards the credits via medizinonline.com? (Not printed.) Needed for compliance metadata even if not on paper.
3. **CME 2 → 1 reduction** — is single-CME the new normal or a content-shortfall stopgap? Default skill behaviour: 1 CME, accept `--cme-count 2` flag for fall-back to old format.
4. **Cover mode NP1-26** — sponsored takeover (Kesimpta) a one-off rate-card change or a permanent option? Skill flag: `cover_mode: teaser|sponsored-takeover`.
5. **Sonderreport vs. Publireportage rate-card** — distinction in price/length not visible in PDFs. Mediadaten doc would resolve.
6. **Editorial voice profile per issue** — currently inferred from sign-off; should be explicit in `/kb-issue-plan` (`editorial_voice: burggraf|schliebe|blanke|<new>`).
7. **KOL COI** — currently not printed. Industry trend toward disclosure could mandate this in future. Capture in frontmatter (`kol.coi:`) for archive even if not rendered.
8. **PMID in references** — currently never printed. We store PMIDs in our wiki sources but suppress in print output. Confirm.
9. **KONGRESS banner mandatory** for all conference-derived content vs. only major (DGN/EAN/ECTRIMS)? Currently inconsistent.
10. **Reader-engagement instrumentation** — no Letters to the editor, no Top-Read. Should the digital edition (medizinonline.com) feed this back? If so, surface to `/kb-issue-plan` as topic-priority signal.
11. **Bilingual policy** — French CME (NP6-24 only). Permanent option for francophone CH?
12. **Auf einen Blick** rubric — last seen NP1-25 p.33. Retired or paused?
13. **Other 14 specialty titles** — production specs to be reverse-engineered analogously. Suggested order: highest-volume titles first (Onkologie? Kardiologie? Pädiatrie?).

---

## 20. Schema version

- Spec version: **1.0** (2026-05-01)
- Source: 18 issues NP1-23 → NP1-26 + 1 process docx (Themenplan NP3-26)
- Reverse-engineering reports (audit trail): `docs/reverse-engineering/info-np/{structure,cme,pharma,voice}.md`
