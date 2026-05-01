# Reverse-Engineering Report: CME Articles — InFo Neurologie & Psychiatrie

**Generated:** 2026-05-01 by parallel agent analysis (Agent 2 of 4).
**Scope:** CME articles — format, learning structure, MC questions, accreditation, authorship.

## Sample base
TOC scanned for all 18 issues. Deep reads on six articles spanning the time series:
- NP1-23: Palliative Care (Camartin, p.6) + Parkinson L-Dopa (Reichmann, p.10)
- NP3-23: Histaminintoleranz + Raucherentwöhnung
- NP3-24: Kardiopsychologie (Schmutz/Hess, Bern)
- NP4-24: ALS (p.6) + Burnout-Prävention (Burggraf, Praxismanagement)
- NP6-24: Epilepsie nach Schlaganfall (Galovic, Zürich + Winter, Mainz)
- NP3-25: ALS-Symposium 2024 (Schliebe)
- NP4-25: "Krebs & Psyche" (Schwarz-Herion, Ettlingen) — special "all-CME" issue
- NP1-26: Neuroenhancement (Blanke, Zürich)

## 1. CME identification markers

Visually distinctive elements are **completely consistent** issue-to-issue:
- A small page-header band reads **"CME-FORTBILDUNG"** on every page of a CME article (e.g. NP1-23 p.6 ff., NP1-26 p.8 ff.).
- The TOC editorial page (typically p.3 or p.5) carries a fixed block: **"Die Fortbildungsthemen in dieser Ausgabe:"** (singular: "Das Fortbildungsthema..." when only one CME, e.g. NP5-25, NP1-26), followed by topic + page number, ending with **"CME-Fortbildungsfragen ... Seite XX"**.
- A small box repeats: **"Credits auf medizinonline.com — Einloggen, Fragen beantworten und direkt CME-Zertifkat downloaden."** (the typo "Zertifkat" without "i" is reproduced literally throughout — a **stylistic fingerprint**).
- The article's first page shows: author photo/name box top-right, vertical sidebar callout **"> Fortbildungsfragen auf Seite XX"**, headline + subhead, a keyword line in italics (e.g. "Palliative Care | SENS-Modell | pulmonale Metastasen"; "Nootropika | Stimulanzien | Kognitive Leistungssteigerung").
- The MC-question pages have a yellow-to-grey decorative band labelled **"> Fortbildungsfragen"** with the topics again printed below.
- An "Anleitung zur Online-Fortbildung" page precedes or follows the questions ("Aktivieren Sie auf medizinonline.com Ihren kostenlosen Account...").

**Accreditation body**: Not named explicitly (no "SIWF", "FMH", "EACCME", or "BÄK" appears anywhere). Credits are distributed exclusively via the publisher's own platform **medizinonline.com** (formerly medizinonline.ch through 2023). Number of CME points is not printed in the magazine — it is shown only in the user's online cockpit after passing. The portal gives a **CME-Zertifikat (PDF)** which the physician then submits to their own society. The article in NP1-23 explicitly mentions: *"Dank des Sponsorings der Firma Bial ist die Teilnahme an dieser Fortbildung für Sie kostenlos. Die Fortbildung ist von Fachleuten erstellt und unter unbedingter Wahrung unserer redaktionellen Freiheit erstellt."* — confirming CME articles are **industry-sponsored but editorially independent**, with the sponsoring firm named in a footer.

## 2. Canonical CME article structure

Recurring skeleton (12-page CME = 6–7 magazine pages of body + 1 page disclaimer + 1 page MC questions):

1. **Headline pattern**: short topic noun (e.g. "Palliative Care", "Adulte ADHS", "Neuroenhancement") + creative subhead ("Eine zunehmende Herausforderung für die Ärzteschaft", "Kann man Intelligenz schlucken? Relevante Substanzklassen mal für Gesunde").
2. **Author block** top-right: title (Dr. med. / Prof. Dr. med. / Dr. phil. / M.Sc.), name, position, institution, postal address, email.
3. **Keyword/tag line** in italics under headline (3–4 keywords pipe-separated).
4. **Lead paragraph** opens with the black-square ▪ glyph and a clinical/societal hook.
5. **No formal "Lernziele:" header** — learning objectives are not printed. They are implicit in the MC questions.
6. **Body sections** with bold subheads. Recurring sequence (varies by topic):
   - Definition / Epidemiologie / Prävalenz
   - Pathophysiologie or Pathogenese
   - Klinik / Diagnostik / Differenzialdiagnosen (often a **Fallbeispiel** — case vignette — for clinical articles)
   - Therapie (drug classes individually; tables comparing options)
   - Praktisches Vorgehen / Anwendung in der Praxis
7. **Tables and figures**: every CME has 1–3 tables and 1–2 figures, captioned "Tab. 1", "Abb. 1", with credit "modifiziert nach [n]" pointing at a literature reference.
8. **Take-Home-Messages** as a yellow boxed list with em-dash bullets (3–5 bullets). This box is **mandatory**.
9. **"Fazit"** or **"Konklusion"** or **"Zusammenfassung"** paragraph.
10. **Disclosures**: the sponsor disclaimer ("Dank des Sponsorings der Firma X...") appears on the final body page. Author conflicts of interest are **NOT printed in the magazine** — there is no "Interessenkonflikte"/"COI" section.
11. **References**: numbered Vancouver-style with **square brackets [1]** in body text. Bibliography uses concise format: *"Author, et al.: Journal Year; Vol(issue): pages. doi:..."* (e.g. "Stocchi F, Antonini A, Barone P, et al.: Park Relat Disord 2014; 20: 204–211."). Some shorter pieces print "Literaturliste bei Verlag" instead of full list. URLs to AWMF/NICE/Bundesamt receive "(letzter Zugriff: TT.MM.JJJJ)".
12. **MC-question block**: always at the **end of the CME section, after both articles** when an issue carries two CME items (sharing one combined question set). Located on a single page (e.g. NP1-23 p.17, NP3-25 p.16, NP1-26 p.18).
13. **Resolution/answer location**: Not printed in the magazine. Answers are revealed only inside the **medizinonline.com** quiz engine after submission. No follow-up issue resolution.

## 3. MC question patterns

- **Number per article**: Confirms 6–8 per individual CME article. When two CMEs run in one issue, the question block holds 11–14 combined (e.g. NP3-25 has 8 ALS + 3 Autism = 11 questions; NP3-23 has 3 + 4 = 7 questions across two CMEs).
- **Stem style**: predominantly **knowledge recall** ("Welche Aussage stimmt nicht?", "Welche Reaktionen können auftreten?", "Welches Parkinson-Syndrom hat die beste Prognose?"). Recent issues (NP1-26) introduce **mini-vignettes** ("Ein 22-jähriger Student nimmt seit einigen Monaten Amphetamine..."), suggesting a slow shift toward applied reasoning.
- **Answer-option count**: usually **4 options (A–D)** in 2023–2024; NP3-25 sometimes only **3 options (A–C)**; questions with multiple correct answers also use 4.
- **Single-best vs. multi-select**: Mixed within the same issue. The mode is signalled in italics under the stem: **"(gesuchte Antwort ankreuzen)"** = single best; **"(alle gesuchten Antworten ankreuzen)"** = multi-select. Roughly 60/40 single/multi.
- **Bloom level**: predominantly **recall (level 1) and comprehension (level 2)**. Application-level vignettes appear in NP1-26. Almost no analysis/evaluation items.
- **Distractor quality**: Generally plausible — distractors are real medical concepts within the topic (other Parkinson syndromes, alternative diagnostic tools, plausible drug classes). Occasionally one obviously wrong option appears ("D Schlaffe Lähmungen der Muskulatur" among Wirkfluktuationen).
- **Coverage**: questions tightly follow article subheadings; rarely test material outside the printed article — required-articles are listed at the bottom of the question page ("Zur Beantwortung der Fragen sind folgende Artikel erforderlich: ...").

**Verbatim examples**:

> *NP1-23 p.17, Q4*: **"Welches Parkinson-Syndrom hat die beste Prognose? (gesuchte Antwort ankreuzen)** A) Der Rigor-Akinese-Typ B) Der mit einer Mutation des GBA-Gens assoziierte Typ C) Der Tremor-dominante Typ D) Der marantische Typ"

> *NP3-23 p.19, Q5*: **"Wie hoch ist die Aufhörquote von nicht-assistierten Aufhörversuchen und von Aufhörversuchen mit dem Goldstandard, also einer Kombination von Verhaltenstherapie und medikamentöser Therapie? (alle gesuchten Antworten ankreuzen)** A) Spontane Erfolgsquote 15–20% B) Spontane Erfolgsquote 3–7% C) Erfolgsquote Goldstandard 15–20% D) Erfolgsquote Goldstandard 30–35%"

> *NP1-26 p.20, Q5 (vignette style)*: **"Ein 22-jähriger Student nimmt seit einigen Monaten Amphetamine (Adderall) zur Prüfungsvorbereitung. Welches Risikoprofil ist bei dieser Substanzklasse besonders relevant? (gesuchte Antwort ankreuzen)** A) Selektive Beeinträchtigung des Langzeitgedächtnisses bei guter Verträglichkeit. B) Niedriges Abhängigkeitspotenzial bei moderatem kardiovaskulärem Risiko. C) Hauptsächlich gastrointestinale Nebenwirkungen ohne relevantes Suchtpotenzial. D) Hohes Abhängigkeitspotenzial, kardiovaskuläre Risiken und Gefahr psychiatrischer Komplikationen wie Paranoia."

## 4. Author profile

- **Specialty**: Almost always neurology or psychiatry; occasional adjacencies (palliative care physician, clinical psychologist, sociology/CSR consultant in NP4-25, ENT/internal medicine for cross-cutting topics like Histaminintoleranz, Raucherentwöhnung).
- **Affiliation**: ~70% **university hospitals** (Inselspital Bern, Universitätsspital Zürich, Klinik für Neurologie Dresden, Universitätsmedizin Mainz, UK Marburg, Kantonsspital Graubünden); ~20% **Prime Public Media in-house staff** (see below); ~10% specialty consultants/private institutions.
- **Country distribution**: **CH dominant (~55%)**, **DE strong (~35%)**, plus rare AT/EU contributions. The CH/DE split is conscious — bilingual relevance for Swiss readers.
- **Position**: senior physicians (Oberarzt, Leitender Arzt, Direktor, Professor) far more than residents.
- **Recurring "house authors"**:
  - **Leoni Burggraf** (Redaktion, Zürich): NP4-24 Burnout-Prävention; NP1-24 Adulte ADHS; NP5-23 Schizophrenie. Address: Neugasse 10, 8005 Zürich (Prime Public Media).
  - **Tanja Schliebe** (Chefredaktion): NP3-25 ALS, NP4-25 cover editorial.
  - **Dr. Ulf Blanke** (Redaktion, Zürich): NP5-25 Psychedelika; NP6-25 Adhärenz + KI in Neurologie (BOTH CMEs in NP6-25); NP1-26 Neuroenhancement. Notably, Blanke has authored **every CME** since late 2025 — possibly a temporary author shortage being papered over by editorial.
- **Credentials**: "Dr. med." standard for clinicians; "Prof. Dr. med." for academics; "PD Dr. med." for Habilitierte; "Dr. phil." for psychologists; "M.Sc." appended where applicable; "MHBA"/"MBA" rare; "Dr. oec." for the NP4-25 sociology author. Always written as title block beside a circular photo.

## 5. Topic distribution across 18 issues

| Issue | CME 1 | CME 2 |
|---|---|---|
| NP1-23 | Palliative Care | Parkinson (L-Dopa) |
| NP2-23 | Migräne | Arzneimittelinteraktionen |
| NP3-23 | Histaminintoleranz | Raucherentwöhnung |
| NP4-23 | Schizophrenie | Post-COVID |
| NP5-23 | Schizophrenie | Reizdarmsyndrom (Psyche) |
| NP6-23 | KI in der Medizin | Gerontopsychiatrie |
| NP1-24 | Epilepsie (Status epilepticus) | Adulte ADHS |
| NP2-24 | Migräne (Therapie) | Schlafstörungen im Alter |
| NP3-24 | Kardiopsychologie | T-Helfer-1-Zellen |
| NP4-24 | ALS | Burnout-Prävention (Praxismanagement) |
| NP5-24 | Alzheimer | Adipositas |
| NP6-24 | Epilepsie (Post-Stroke) | Maladie d'Alzheimer (FR) |
| NP1-25 | Künstliche Intelligenz | ADHS |
| NP2-25 | Parkinsonkrankheit | Chemsex |
| NP3-25 | ALS-Therapie | Schmerz und Autismus |
| NP4-25 | Krebs & Psyche (single CME, but issue is themed) | — |
| NP5-25 | Psychedelika | — (single-CME issue) |
| NP6-25 | Adhärenz in der Psychiatrie | KI in der Neurologie |
| NP1-26 | Neuroenhancement | — (single CME) |

**Recurring themes**: ALS (3×), Migräne (2×), Epilepsie (3×), Schizophrenie (2×), ADHS (2×), KI/Digital (4× — clearly trending up), Parkinson (3×). **Neurology articles (~55%)** slightly outnumber psychiatry (~35%); cross-cutting/Praxismanagement (~10%, e.g. Burnout, Histamin, Raucherentwöhnung).

**2025–2026 trend**: shift to **single-CME issues** (NP4-25 themed all-CME special, NP5-25 and NP1-26 single CME). Suggests recruitment fatigue / editorial filling gaps.

## 6. Tone & language

- **Formality**: high — "Sie"-form for the reader is rare; CME prose is **impersonal and didactic** ("Bei älteren Patienten >70 Jahre wird in der Regel...").
- **Patient nomenclature**: shift from generic "Patienten" (2023) → **"Patient:innen" / "Patient*innen"** (2024 onward) reflecting gendering policy.
- **Sentence length**: longer than the magazine's news section. Average 18–25 words; nested subordinate clauses common.
- **Evidence calibration**: **AWMF S3-Leitlinien** are cited (NP1-23, NP3-23 Raucherentwöhnung); EAN/EFNS guidelines for neurology; NICE for digital tools. Studies cited with sample sizes ("n=200 Betroffene mit MG-ADL-Score ≥3"). HR/p-values inline ("HR 1,29; p=0,04").
- **Style markers**: "Es konnte gezeigt werden, dass...", "Gemäss Leitlinien...", liberal use of em-dashes, parenthetical author citations.
- **Compared to news/magazine sections**: CME prose is more sober, less narrative.
- **Swiss German Vorsicht**: Helvetisms preserved ("ss" never "ß"; "Massnahme", "grosse"). German-affiliated authors keep "ß" in their addresses.

## 7. CME Do's and Don'ts

**Do's**
- Always include Take-Home-Messages box (3–5 em-dash bullets).
- Always include a Fazit/Konklusion paragraph closing the body.
- Cite **primary literature with DOI**; AWMF/EAN guidelines for therapy claims.
- Use INN drug names freely (Levodopa, Tofersen, Methylphenidat); brand names appear in parentheses ("Methylphenidat, besser bekannt unter Handelsnamen wie Ritalin oder Concerta") for orientation, not promotion.
- Include a **Fallbeispiel** when topic is clinical.
- Use German throughout; one French CME exists (NP6-24 Maladie d'Alzheimer) for francophone CH.
- Reference style: Vancouver, square brackets, full bibliography.
- One sponsor-disclosure paragraph at end of body.

**Don'ts**
- No printed learning objectives ("Lernziele:" header is absent).
- No author COI section (handled at sponsorship-policy level).
- No CME points / no accrediting body name printed.
- No answer key inside the magazine.
- No promotional brand-name pushes inside CME body.
- No "level of evidence" (1A/1B) call-outs — the German-Swiss style does not adopt the US/UK formal grading boxes.
- No URLs in body except for guidelines/registries.

## 8. Information needed to produce a CME from scratch

Minimal "recruiter handoff" pack:

1. **Topic + 3–4 keywords** (exact subhead is the writer's choice).
2. **Target word count**: ~3,500–5,000 words main body (= 6–8 magazine pages).
3. **Target audience emphasis**: CH primarily (mention Swiss prevalence data, Spezialitätenliste status, Bundesamt für Gesundheit refs); secondary DE.
4. **Author block**: title, name, function, full institutional postal address, email, photograph.
5. **Issue's other CME topic** (so combined MC question pool is balanced).
6. **6–8 MC questions** with single-best vs. multi-select indicated, 4 options each, one correct answer per single-best Q.
7. **3–5 Take-Home-Messages**.
8. **1–3 tables, 1–2 figures** with captions and source attributions.
9. **Vancouver references with DOIs**, max ~30–40.
10. **Sponsor identification** (which pharma firm funds the slot — for the standard disclaimer footer).
11. **Optional Fallbeispiel** vignette for clinical topics.
12. **No COI form needed** for in-magazine print.

**Ideal handoff format**: a single Word/Markdown file containing front-matter (topic, author, sponsor), body with H2/H3 headings matching the canonical structure, table/figure placeholders with captions, references list, then a separate MC-question block.
