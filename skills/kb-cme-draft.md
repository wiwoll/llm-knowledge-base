---
name: kb-cme-draft
description: Draft a CME-Fortbildung article for a magazine following the canonical structure (Definition → Pathophysiologie → Klinik/Diagnostik → Therapie → Praxis), with mandatory Take-Home-Messages box, sponsor disclosure, and Vancouver references. Always editorial_status=draft. Usage: /kb-cme-draft <magazine-slug> <issue-id> <topic-slug> [--author "Prof. Dr. ..."] [--sponsor "Bial"] [--pages 6] [--lang de]
trigger: /kb-cme-draft
allowed-tools: Read, Write, Edit, Bash, mcp__claude_ai_PubMed__search_articles, mcp__claude_ai_PubMed__get_article_metadata, mcp__claude_ai_PubMed__get_full_text_article, WebFetch
---

# KB CME Draft

Produce a CME article that fits the magazine's house style. The output is a draft for the Chefredaktion to review and the named author to validate.

## Steps

### 1. Read Spec & Plan

```bash
cat ~/.claude/skills/SCHEMA.md
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
cat ~/.claude/kb-config.json
cat {KB_PATH}/wiki/issues/{ISSUE_ID}/issue-plan.md
```

Locate the topic in the plan. Note: locked author, locked sponsor, locked pages, locked editorial voice. If the topic is not in the plan, refuse:
```
Error: Topic {topic-slug} not found in issue plan {ISSUE_ID}. Run /kb-issue-plan first or pass --force-without-plan.
```

### 2. Parse Arguments

`<magazine-slug> <issue-id> <topic-slug>` required. Optional flags override plan values.

### 3. Gather Source Material

Search the wiki for existing concept articles + sources matching the topic:
```bash
python3 {KB_PATH}/kb_search.py "{topic search query}" --top 15
```

Read the top-matching wiki articles. If insufficient: invoke PubMed MCP (when available) for 5–8 recent high-evidence sources (filter: meta-analysis, RCT, guideline, last 5 years). Pass each through `/kb-source` ingestion logic — but for time-critical drafting, just capture the metadata in memory; full ingestion can follow later via `/kb-source` invocations.

Also fetch the most recent **AWMF S3-Leitlinie / EAN guideline / NICE guideline** for the topic (use WebFetch on `https://www.awmf.org/leitlinien/aktuelle-leitlinien` or guideline-specific URL). German-language guidelines preferred for CH/DE audience.

### 4. Construct Article

Follow the magazine spec's CME structure verbatim (per spec §7.2). Build sections in order:

#### 4a. Header block
- **Headline**: short topic noun (1–3 words)
- **Subheadline**: 1 line, descriptive or thoughtful (use the spec's voice profile to flavour)
- **Keyword line**: italic, 3–4 keywords pipe-separated
- **Author block** (top-right): title + name + position + institution + postal address + email + photo flag
- **Vertical sidebar callout**: `> Fortbildungsfragen auf Seite {placeholder TBD by /kb-issue-compose}`

#### 4b. Body
- **Lead paragraph**: opens with `▪` glyph, hooks with clinical/societal context, stems from a current data point or guideline change
- **Section 1 — Definition / Epidemiologie**: bold subhead. Prevalence figures with Swiss data when available (Spezialitätenliste, BAG); otherwise DACH or international. Hedge language.
- **Section 2 — Pathophysiologie / Pathogenese**: mechanism. Cite primary literature `[n]`.
- **Section 3 — Klinik / Diagnostik / Differenzialdiagnosen**: clinical features, diagnostic algorithm, DD table. Insert a **Fallbeispiel** vignette in `Übersicht 1` for clinical topics (one realistic anonymized case ~150 words).
- **Section 4 — Therapie**: drug classes individually; comparison `Tab. 1` with INNs (no brand names in body); guideline anchors (AWMF S3, EAN); evidence inline `[n]`.
- **Section 5 — Praktisches Vorgehen / Anwendung in der Praxis**: actionable for the Swiss neurologist/psychiatrist; reimbursement notes (Spezialitätenliste, kassenzulässig); Swiss limitatio if relevant.

Each section bold-headed; subsections allowed.

#### 4c. Tables / figures
At least 1 table + 1 figure. Captioning per spec §14.9:
- `Tab. 1: <bold descriptive title>. <Explanatory sentence.> Modifiziert nach [n].`
- `Abb. 1: <bold descriptive title>. <Explanatory sentence.>`
Image credit microtext at margin: `<author>, istock` (lowercase, italic).

#### 4d. Take-Home-Messages (mandatory)
```
TAKE-HOME-MESSAGES
― <statement 1: an actionable insight, 1 sentence>
― <statement 2: an evidence-anchored claim>
― <statement 3: a guideline-aligned recommendation>
― <statement 4: a caveat or open question>
― <statement 5 (optional): a Swiss-specific reimbursement / pathway note>
```
3–5 bullets. Em-dashes `―` not `•`.

#### 4e. Fazit
1 paragraph closing — synthesize the article into a clinically actionable bottom line. Hedged where evidence warrants.

#### 4f. Sponsor disclosure (verbatim, German)
```
Dank des Sponsorings der Firma {SPONSOR} ist die Teilnahme an dieser Fortbildung für Sie kostenlos.
Die Fortbildung ist von Fachleuten erstellt und unter unbedingter Wahrung unserer redaktionellen
Freiheit erstellt. Zu keinem Zeitpunkt hatte und hat die Firma Einfluss auf die Inhalte der
Fortbildung. Alle Texte unterliegen lediglich der wissenschaftlichen Hoheit der Autoren und der
Redaktion von PPM MEDIC.
```
French parallel (when `--lang fr`):
```
Grâce au sponsoring de la société {SPONSOR}, la participation à cette formation est gratuite pour vous. 
La formation est élaborée par des experts et rédigée dans le respect absolu de notre liberté éditoriale. 
À aucun moment la société n'a eu ou n'a d'influence sur le contenu de la formation. 
Tous les textes relèvent uniquement de la souveraineté scientifique des auteurs et de la rédaction de PPM MEDIC.
```

#### 4g. Literatur (Vancouver)
Numbered list, deduplicated. Format:
```
1. Author A, Author B, Author C, et al.: Journal-Abbrev Year; Vol(issue): pages. doi:10.xxxx/xxxxx
```
Include DOIs when open-access. **Do not include PMIDs in print** (store them in frontmatter instead). AWMF/NICE URLs: `(letzter Zugriff: TT.MM.JJJJ)`.
Cap at ~30–40 references. If shorter than 8, write `Literaturliste bei Verlag` instead.

### 5. Frontmatter

```yaml
---
type: cme
magazine: {MAGAZINE_SLUG}
magazine_section: cme-fortbildung
issue_id: {ISSUE_ID}
topic_slug: {TOPIC_SLUG}
lang: {LANG}
editorial_status: draft
sponsor: {SPONSOR}
sponsor_disclosure_lang: {LANG}
keywords: [keyword1, keyword2, keyword3, keyword4]
target_pages: {N}
mc_question_count: {6 to 8}
authors:
  - title: {Prof. Dr. med.}
    name: {full name}
    position: {Direktor / Oberarzt / Leitender Arzt / etc.}
    institution: {Klinik für ...}
    address: {street, postal code city, country}
    email: {email}
    photo: present  # or absent
learning_objectives:  # NOT printed; informs the MC questions
  - {objective 1}
  - {objective 2}
  - {objective 3}
  - {objective 4}
coi:  # NOT printed; archive for compliance
  declared: {true|false|unknown}
  details: {free text or omit}
references:
  - vancouver: "1. Author A, et al.: Journal Year; Vol(issue): pages."
    doi: "10.xxxx/xxxxx"
    pmid: "12345678"
    cited_at: [3, 7]  # body sections where cited
  # …
created_at: {now}
updated_at: {now}
tags: [cme, {magazine_slug}, {topic_slug}, {medical-area-tags}]
---
```

### 6. Write the Article

Set `ARTICLE_FILE = wiki/issues/{ISSUE_ID}/cme/{TOPIC_SLUG}.md`.

Compose the rendered Markdown body following 4a–4g, in `LANG`.

### 7. Trigger MC Question Generation

Invoke `/kb-mc-questions {magazine_slug} {issue_id} {topic_slug}` (does not auto-launch — print a recommendation):
```
CME draft saved. Generate MC questions:
  /kb-mc-questions {magazine_slug} {issue_id} {topic_slug}
```

### 8. Update Index

Append to `wiki/index.md` under `## Hefte / Issues`:
```
- [[issues/{ISSUE_ID}/cme/{TOPIC_SLUG}]] — CME {ISSUE_ID}: {Title} ({author}) `[draft]`
```

### 9. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: cme draft {ISSUE_ID}/{TOPIC_SLUG}"
```

### 10. Print Confirmation

```
CME-Entwurf gespeichert: {ARTICLE_FILE}
  Sprache: {LANG}
  Sponsor (für Disclosure): {SPONSOR}
  Autor*in: {author}
  Zielseiten: {N}
  Take-Home-Messages: {N} Bullets
  Literatur: {N} Quellen
  Status: draft

Nächste Schritte:
  /kb-mc-questions {MAGAZINE_SLUG} {ISSUE_ID} {TOPIC_SLUG}    # Quiz erstellen
  Editor-Review → /kb-review {…}                               # Promotion in workflow
  Author-Validierung (extern)                                  # vor Publikation Pflicht
```

## Notes

- The article is **always** `editorial_status: draft`. CME content cannot be auto-promoted past draft — the named external author must approve, and the Chefredaktion must run `/kb-review`.
- The standard sponsor-disclosure paragraph **must** appear verbatim. Skill never paraphrases this — it is legal/compliance language.
- Brand names appear in the body **only** as parenthetical orientation at first INN mention (e.g., "Methylphenidat (Handelsnamen u.a. Ritalin®, Concerta®)"). Brand-first wording is the marker of sponsored content and must not appear in CME body — that is what `/kb-sonderreport` is for.
- For francophone CH (rare), `--lang fr` produces the French parallel including the French sponsor disclosure formula. Otherwise default `de`.
- The author block is preserved exactly as commissioned. If `--author` is omitted, the skill falls back to in-house byline (current Chefredaktion or last active editor) and flags this in the article frontmatter as `authors[0].in_house: true`.
