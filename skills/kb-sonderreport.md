---
name: kb-sonderreport
description: Draft a Sonderreport, Publireportage, or Markt & Medizin sponsored content piece with mandatory disclosures, Kurzfachinformation reference, brand-name compliance (with ®/™), and editorial-firewall language. Refuses to advance off-label use. Always editorial_status=draft. Usage: /kb-sonderreport <magazine-slug> <issue-id> <slug> --kind sonderreport|publireportage|markt-medizin --sponsor "Biogen Switzerland AG" --product "Skyclarys|Omaveloxolon"
trigger: /kb-sonderreport
allowed-tools: Read, Write, Edit, Bash, WebFetch
---

# KB Sonderreport

Draft sponsored editorial content. This skill produces a draft that the sponsor's medical-affairs team will then review and amend through several correction loops (per the publisher's process docs: "Hoher Abstimmungsaufwand mit Kunden (mehrere Korrekturschleifen)").

## Steps

### 1. Read Spec

```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
```

Internalize §12 (Sonderreport / Publireportage) verbatim. The disclosure templates are legal/compliance language and **must be reproduced verbatim**.

### 2. Parse Arguments

Required:
- `<magazine-slug> <issue-id> <slug>` (slug like `biogen-skyclarys-langzeit`)
- `--kind sonderreport|publireportage|markt-medizin`
- `--sponsor "<Legal Name + Swiss address>"` — full legal name
- `--product "<brand>|<INN>"` (the | separator is intentional — brand for headline use, INN for body apposition)

Optional:
- `--kol "Title Name|Affiliation"` for KOL Q&A format
- `--medical-writing-cro "<CRO Name>"` if external medical writers were involved
- `--medical-writers "Name1, Name2"` (comma-separated)
- `--reprint-source "BrainMag 2025; 6: 8–9"` if it's a reprint (Sonderreport-Reprint chain)
- `--approval-code "Biogen-264206_04.2025"` (mandatory — provided by sponsor)
- `--kurzfachinformation-page "same"|"<page number ref>"` — where the Kurzfachinformation is printed
- `--target-pages 2` (default: kind-specific — sonderreport 2–4, publireportage 1–2, markt-medizin ½)
- `--language de|fr` (default `de`)

### 3. Off-Label Check

If the requested topic involves off-label use, **refuse**:
```
Refused: sponsored content cannot advance off-label use of {product}. Spec §12.6 forbids this.
Alternative: editorial discussion of off-label may proceed via /kb-medizin-article (review type, not product-tied).
```

### 4. Compose

#### Sonderreport (multi-page, often KOL or congress reprint)

```markdown
{TOP STRIP}: SONDERREPORT

# {Headline — brand-first allowed, e.g., "Skyclarys™ — Langzeitdaten der MOXIe-Extension"}

## {Subhead — descriptive}

{Lead paragraph — 2–4 sentences setting up the context. Use the brand name with ®/™.}

## Hintergrund / Klinischer Kontext
{1–2 paragraphs. Disease, current standard of care, unmet need.}

## Studiendaten / Evidenz
{Trial data with effect sizes, CIs, p-values. Cite trial name (e.g., MOXIe), reference numerically [1].}

## Diskussion / Einordnung
{Implications. Always within label.}

## Take-Home Box (optional, often present)
**Wussten Sie schon?**
- {Fact 1}
- {Fact 2}
- {Fact 3}

## Literatur
1. {primary trial reference}
2. {label / Swissmedic info}
3. {KOL publication if cited}

---

{KOL DISCLAIMER if KOL Q&A:}
> Die hier getätigten Aussagen spiegeln die professionelle Meinung von {KOL Name} wider.

---

{FUNDING DISCLOSURE — verbatim from spec §12.3, choose one based on context:}
> Dieser Beitrag wurde finanziert von {SPONSOR Legal Name}.

{IF MEDICAL-WRITING CRO USED:}
> Berichterstattung: {Medical writers}, {CRO Name}, im Auftrag von {Sponsor}.
> Die Medical-Writing-Unterstützung wurde durch {medical_writers} ({medical_writing_cro}) im Auftrag der {sponsor} erbracht.

{IF REPRINT:}
> © {Original Publisher}, {year}. Abdruck mit freundlicher Genehmigung des Verlags.
> Erstpublikation erschienen in {reprint_source}.

---

{FOOTER BLOCK — bottom of page in tiny type:}
{Sponsor Legal Name}
{Swiss address}
{Approval/Clearance code: e.g., Biogen-264206_04.2025}

{KURZFACHINFORMATION:}
{If kurzfachinformation_page == "same":}
**Kurzfachinformation {Brand}®**
Z: {Zusammensetzung — verbatim from official Fachinformation}
I: {Indikation}
D: {Dosierung}
KI: {Kontraindikationen}
VM: {Vorsichtsmassnahmen}
IA: {Interaktionen}
SS/St: {Schwangerschaft/Stillzeit}
UAW: {Unerwünschte Arzneimittelwirkungen}
P: {Packungen}
Abgabekat.: {A | B | C | D | E}
Stand der Information: {MM/YYYY}

{ELSE:}
Kurzfachinformation auf Seite {N}.
```

#### Publireportage (1–2 pages, advertorial style)

Similar structure but lighter:
- `{TOP STRIP}: PUBLIREPORTAGE`
- 1-page format more common; product-tied
- Disclosure: `Dieser Artikel entstand mit finanzieller Unterstützung von {SPONSOR}.`
- Kurzfachinformation always required; same page or referenced

#### Markt & Medizin (½ page, press-release-derivative)

This skill can route to `/kb-news-item --kind markt-medizin` instead — that skill handles the lighter format. Defer there:
```
For kind=markt-medizin, /kb-news-item is the canonical skill.
Routing to: /kb-news-item {magazine-slug} {issue-id} --kind markt-medizin --source <press-release> --company "{sponsor}"
```

### 5. Frontmatter

```yaml
---
type: {kind}
magazine: {MAGAZINE_SLUG}
magazine_section: {section per kind}
issue_id: {ISSUE_ID}
slug: {SLUG}
lang: {LANG}
editorial_status: draft
sponsor:
  legal_name: {full legal name}
  address: {Swiss address}
  approval_code: {code}
product:
  brand: {brand}
  inn: {INN}
  indication: {indication}
  spezialitaetenliste: {true|false|kassenzulaessig: YYYY-MM-DD}
{Optional:}
kol:
  title: {…}
  name: {…}
  affiliation: {…}
medical_writing_cro: {CRO name}
medical_writers: [Author1, Author2]
reprint:
  source: {BrainMag 2025; 6: 8–9}
  publisher: {medEdition Verlag GmbH, Hirzel}
  year: {YYYY}
references: [...]
copyright_holder: Prime Public Media AG  # always — per spec §12.8
kurzfachinformation:
  on_same_page: {true|false}
  reference_page: {N if elsewhere}
target_pages: {N}
created_at: {now}
updated_at: {now}
off_label_check_passed: true  # mandatory; otherwise refused above
tags: [sponsored, {kind}, {sponsor-tag}, {product-tag}]
---
```

### 6. Style Discipline

- Brand name with ® / ™ throughout (NOT INN-first)
- INN in apposition: `Skyclarys™ (Omaveloxolon)`
- No competitor disparagement
- All efficacy/safety claims need numbered citations resolved in Literatur
- KOL disclaimer when interview format
- No off-label content
- Tone is promotional but factual; no Konjunktiv-distance from sponsor's claims

### 7. Write

Set `FILE = wiki/issues/{ISSUE_ID}/sponsored/{SLUG}.md`.

### 8. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: {kind} {ISSUE_ID}/{SLUG} ({sponsor.legal_name})"
```

### 9. Print

```
Sponsored draft erstellt: {FILE}
  Typ: {kind}
  Sponsor: {legal_name} ({approval_code})
  Produkt: {brand} ({INN}) — {indication}
  Zielseiten: {N}
  KOL: {name | "—"}
  Reprint: {source | "—"}
  Status: draft

Off-label check: PASSED.
Disclosure-Sätze: verbatim aus magazine spec §12.3 eingebunden.
Kurzfachinformation: {on same page | "Seite {N}"}.

Nächste Schritte:
  1. Sponsor-Medical-Affairs reviewt Draft (mehrere Korrekturschleifen erwartet).
  2. Nach Freigabe: /kb-review {issue}/sponsored/{slug} --status fact-checked --reviewer @{sponsor-contact}
  3. Heftzusammenstellung: /kb-issue-compose {ISSUE_ID}
```

## Notes

- The sponsor goes through ≥1 correction loop on every Sonderreport/Publireportage. Expect multiple rounds before publication; each round = a new commit on this file.
- The copyright clause `© Prime Public Media AG` is mandatory in the rendered output even when the sponsor commissioned the piece (per spec §12.8 and §11 of the pharma agent report).
- This skill **must not** rewrite the verbatim disclosure paragraphs — they are legal/compliance language.
- For French-language Sonderreports (rare; only when the audience is francophone CH), use French parallel disclosures from spec §7 / §12.3.
- After draft, the editor runs `/kb-style-lint {file}` to confirm Swiss number formatting, brand-name discipline, off-label compliance.
