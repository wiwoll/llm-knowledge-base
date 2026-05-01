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
| Editorial language | German (Swiss orthography). **French CME content is permanently planned** for francophone CH; previously constrained by editor staffing. Sample: NP6-24 p.21 (Maladie d'Alzheimer). With LLM-assisted production, French becomes feasible per issue on demand. |
| CME accreditation pipeline | medizinonline.com (Prime Public Media's own platform). PPM may **self-accredit** in many cases (default mode); on customer request, PPM beantragt CME-Punkte at the relevant Fachgesellschaft (CHF 2'250.– Aufpreis per Mediadaten 2026). The accrediting body is **not printed on paper** — varies per CME slot, surfaced only in the user's medizinonline.com cockpit. |
| Med. Herausgeber | Prof. Dr. med. Barbara Tettenborn, St. Gallen (Neurologie); Prof. Dr. med. Erich Seifritz, Zürich (Psychiatrie) |
| Buchung / Disposition | werbung@primemedic.ch, Tel. 044 250 28 70 |

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

**Cover anomaly NP1-26 (Kesimpta full-page):** This is a paid **Sonderwerbeform "Titelseite"** per Mediadaten 2026 (Prod. ID 020, CHF 10'495.–). **Einzelfall** — only when a customer purchases this specific format. Default cover stays the teaser layout. Skill flag: `cover_mode: "teaser"` (default) vs. `"sponsored-takeover"` (only on confirmed booking).

**CME volume:** Default is **2 CME articles per issue** (the canonical InFo NP format throughout 2023–early 2025). The 1-CME issues NP5-25 and NP1-26 were a **temporary Engpass-Notlösung** caused by editor recruiting gaps. **With this LLM-driven production system, the 2-CME format is restored as the default.** Skill flag: `--cme-count 2` (default) vs. `--cme-count 1` (legacy / explicit single-topic special).

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
| `burggraf` | NP1-23 → NP1-25 (retired) | Warm / colloquial / metaphor-driven (e.g., "Tau-frisch im Alter!?", "Ein Indianerherz kennt keinen Schmerz – oder doch?") | Word-play, cultural reference, often question | "Viel Spass / viel Freude" |
| `schliebe` | **active, permanent staff** (since NP2-25) | Sober / academic / evidence-anchored ("Damit gilt als plausibel, dass …") | Descriptive | "Ich wünsche Ihnen eine erkenntnisreiche Lektüre …" |
| `blanke` | **active, permanent staff** (since NP5-25) | Wry / colloquial / occasionally satirical (e.g., "biokular schauen", "Lernen wir schielen") | Imperative / programmatic ("Zeit zum Schielen: Mensch vs. Algorithmus") | "Ich wünsche Ihnen eine erkenntnisreiche Lektüre …" |

Schliebe and Blanke are both permanent fixtures of the editorial team. They alternate (or share by issue) and both voices are available to `/kb-editorial`. Burggraf is retired; the profile remains documented for archival accuracy and may be reactivated if a future editor matches that voice.

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
- **House style (decision logged 2026-05-01)**: use **`Patientinnen und Patienten`** consistently. The `Patient*innen` and `Patient:innen` variants seen in past issues are **not** house style — those should be replaced when an article is touched.
- For `Personen mit <condition>` style (people-first language), allow when clinically meaningful and consistent with the source.

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

## 19. Resolution log (editor decisions on open issues)

Decisions made 2026-05-01 by Chefredaktion:

1. **Patient nomenclature** → `Patientinnen und Patienten` is house style. Other variants are not.
2. **CME accreditation** → PPM may self-accredit (default). On customer request, PPM beantragt CME-Punkte at the relevant Fachgesellschaft (CHF 2'250.– surcharge per Mediadaten 2026 §31). Not printed on paper.
3. **CME 2 → 1** → was Engpass-Notlösung. With LLM-driven production: **2 CMEs per issue is restored as default**.
4. **Cover takeover (NP1-26)** → Einzelfall, only on explicit customer purchase of Sonderwerbeform "Titelseite" (CHF 10'495.– per Mediadaten 2026 §21).
5. **Mediadaten 2026** → integrated. See [`MEDIADATEN-2026.md`](MEDIADATEN-2026.md).
6. **Editorial voice** → Schliebe and Blanke are permanent staff. Burggraf retired. Profile per issue is explicit in `/kb-issue-plan`.
7. **Bilingual French** → permanent target (was staffing-constrained). LLM-assisted production removes the constraint; French CME becomes available on customer / editorial request.
8. **NP3-26 → NP4-26 planning method** → documented in §21 below.
9. **Other specialty titles** → produce InFo Neurologie & Psychiatrie first; expand to other 14 titles after the production loop is proven on InFo NP.

Open items (newly identified or carried forward):

- **KOL COI in print** — currently absent. Industry trend toward disclosure could require this. Capture in frontmatter for archive; print-rendering rule TBD.
- **PMID in print references** — confirmed: stored in frontmatter, suppressed in print (Vancouver-with-DOI only).
- **KONGRESS banner mandatory** for all conference-derived content vs. only major — TBD.
- **`Auf einen Blick` rubric** — paused since NP1-25 p.33; revival TBD.
- **Reader-engagement signals from medizinonline.com** — could feed back into `/kb-issue-plan` topic priority. Pipeline TBD.

---

## 20. Schema version

- Spec version: **1.1** (2026-05-01)
- Source: 18 issues NP1-23 → NP1-26 + 1 process docx (Themenplan NP3-26) + Mediadaten 2026
- Reverse-engineering reports (audit trail): `docs/reverse-engineering/info-np/{structure,cme,pharma,voice}.md`
- Editor decisions: §19 (all 9 originally-open items resolved 2026-05-01)
- Mediadaten reference: [`MEDIADATEN-2026.md`](MEDIADATEN-2026.md)

## 21. Issue planning methodology (how Themenpläne come together)

Reverse-engineered from the NP3-26 Themenplan docx (16 specific topics) compared against the Mediadaten 2026 NP3-26 row (7 broad Themenschwerpunkte + Kongress AAN). The methodology forward-applies to NP4-26 and every subsequent issue.

### 21.1 Inputs

`/kb-issue-plan` starts with three input sources:

1. **Mediadaten Themenschwerpunkte** for the target issue. Pre-committed broad indication areas, sold to advertisers as the basis for booking. NP3-26: ALS, Hirntumore, Neuropädiatrie, Depression, ADHS, Parkinson, Multiple Sklerose. Locked input.
2. **Kongress anchor(s)** for the issue (Mediadaten "Themenschwerpunkte" column also lists congress acronyms): AAN for NP3-26; ESOC + EAN + AAIC for NP4-26; ECTRIMS for NP6-26 etc. Locked input.
3. **Signal scan over the last 60–90 days** before Inserateschluss:
   - Recent Swissmedic / EMA / FDA approvals in neurology + psychiatry
   - Landmark trial readouts (NEJM, Lancet, JAMA, Lancet Neurology, JAMA Psychiatry)
   - Guideline updates (AWMF S3, EAN, AAN, NICE, BAG)
   - New drug classes / launches (e.g., Cobenfy / Muskarinerge, BTK-Inhibitoren)
   - Hot pipeline news (positive Phase III readouts, REMS changes, label expansions)
   - Reader-engagement signals from medizinonline.com (when wired)

### 21.2 Mapping rules

For each broad Mediadaten Themenschwerpunkt, **pick the most current specific subtopic** that the signal scan surfaces. This is what made the NP3-26 list specific:

| Mediadaten broad area | NP3-26 specific topic | Driver |
|---|---|---|
| Multiple Sklerose | BTK-Inhibitoren in der MS — der erwartete Paradigmenwechsel | new drug class (Tolebrutinib readouts) |
| Parkinson | Parkinson — neue Wirkmechanismen jenseits Levodopa | pipeline (subcutaneous L-Dopa, glia targets) |
| ALS | ALS und SMA — Stand der Gen- und Antisense-Therapien 2026 | Tofersen long-term + Risdiplam |
| Depression | Therapieresistente Depression — Esketamin und psychedelische Therapien | Esketamin Spezialitätenliste, MDMA pipeline |
| ADHS | ADHS — neue Nicht-Stimulanzien | Viloxazin etc. |
| Hirntumore | (covered in Medizin section, not promoted to CME) | — |
| Neuropädiatrie | (covered in Medizin section, not promoted to CME) | — |

**Add cross-cutting / signal-driven topics** beyond the Mediadaten list (these are the editor's value-add — currency, breadth, Swiss relevance):

| NP3-26 added topic | Driver |
|---|---|
| Anti-Amyloid-Therapie der Alzheimer-Krankheit | post-Lecanemab/Donanemab field state |
| Schlaganfall — Tenecteplase als neuer Standard | post-SHINE-2 consensus, Schweiz-relevant |
| Migräne — CGRP-Langzeitdaten und Gepants | accumulating long-term data + Gepants |
| Narkolepsie und Hypersomnien — Orexin-Renaissance | Orexin-2 agonist pipeline |
| Aneurysmatische SAB — IV vs. SC Nimodipin | recent comparator data |
| Milsaperidone (Bysanti) — neues Atypikum | drug launch |
| Muskarinerge Antipsychotika — die neue Wirkstoffklasse | Cobenfy / Xanomelin-Trospium |
| Rezidivprophylaxe in der Schizophrenie | LAI long-term real-world data |
| GLP-1-Agonisten in der Psychiatrie | cross-cutting (Wegovy + psych comorbidities) |
| Suchtmedizin — Buprenorphin und Benzodiazepin-Tapering | Praxismanagement-relevant |
| Schizophrenie — Clozapin nach REMS-Aufhebung | FDA REMS removal news |

### 21.3 Distribution across article types

From the topic list, distribute to article types per the section budget (§2):

- **2 CME articles**: pick the deepest, clinically actionable topics with longest reader half-life and clear sponsor-alignment opportunities. NP3-26 example: **BTK-Inhibitoren in der MS** + **Anti-Amyloid-Therapie der Alzheimer-Krankheit**.
- **8–14 Medizin articles** (mix of `study`, `congress`, `review`): the next tier. Congress reports anchor to the Kongress(e). Hirntumore + Neuropädiatrie find homes here.
- **2–4 News-Wissenschaft items**: newest signals (last-month Swissmedic announcement, fresh Phase III readout).
- **2–4 Markt & Medizin items**: pharma press-release-derived; tied to issue's sponsors.
- **0–3 Sonderreport / Publireportage**: customer-driven; depends on confirmed bookings 6–8 weeks before Inserateschluss.
- **1–3 Praxismanagement**: e.g., Suchtmedizin (Buprenorphin/Benzo) is naturally Praxismanagement.

### 21.4 Forward-applying to NP4-26 (worked example)

Mediadaten 2026 NP4-26 row:
- Inserateschluss: 13.08.2026 | Erscheinung: 28.08.2026
- Kongresse: **ESOC, EAN, AAIC**
- Themenschwerpunkte: **Demenz, SMA, Schizophrenie, Sucht, Trauma, Migräne, Schlafstörungen**
- SPECIAL same window: **Demenz** (separate Sonderausgabe 18.09.2026)

Applying §21.2 (illustrative — `/kb-issue-plan info-np NP4-26` produces this automatically using the signal scan as of August 2026):

| Mediadaten broad area | Specific NP4-26 topic candidate |
|---|---|
| Demenz | Lecanemab / Donanemab — Schweizer Real-World-Daten 2026; Blut-Biomarker p-Tau217 in der Praxis |
| SMA | Risdiplam Langzeit- und Onasemnogen-Abeparvovec Real-World-Daten |
| Schizophrenie | Muskarinerge Antipsychotika — Cobenfy in der Praxis nach erstem Jahr |
| Sucht | GLP-1-Agonisten in der Suchtmedizin — neue Evidenz |
| Trauma | MDMA-assistierte Psychotherapie bei PTBS — der regulatorische Stand 2026 |
| Migräne | Gepants vs. CGRP-mAbs — Therapie-Sequenzierung |
| Schlafstörungen | Orexin-Antagonisten — Daridorexant Langzeit-Daten |

Cross-cutting NP4-26 (illustrative):
- AAIC 2026 Highlights (congress report)
- EAN 2026 Highlights (congress report)
- ESOC 2026 — neue Schlaganfall-Daten (congress report)
- Anti-Tau-Therapie nach den ersten Phase-III-Lesungen (pipeline)
- Schlafapnoe-Diagnostik in der Hausarztpraxis (Praxismanagement)

CME picks (2): **Demenz — Diagnostik mit Blut-Biomarkern und Therapieentscheidungen 2026** + **Cobenfy bei Schizophrenie — Mechanismus, Evidenz, Praxis**. Both align with strong sponsor pools.

### 21.5 Encoding in `/kb-issue-plan`

The skill loads `MEDIADATEN-2026.md` (the issue row), runs the signal scan via the wiki + WebFetch (Swissmedic, EMA, FDA, AWMF, recent journal RSS), and produces the topic list with each topic tagged:
- `mediadaten_anchor` (broad area or `null` for cross-cutting)
- `signal_driver` (what surfaced this topic — approval/trial/guideline/launch/REMS)
- `congress_anchor` (link to issue's Kongress(e))
- `proposed_section` (cme | medizin-study | medizin-congress | medizin-review | praxismanagement | news-wissenschaft | markt-medizin | sonderreport)
- `sponsor_candidates` (pharma companies whose products appear — drives sponsor-slot booking)

The Chefredaktion reviews and locks topics; downstream skills draft articles per topic.
