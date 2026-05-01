---
name: kb-editorial
description: Draft the issue Editorial in the requested voice profile (burggraf | schliebe | blanke | custom). One page, ~350–500 words, opens with ▪ glyph + a vignette, bridges to issue themes, signs off with name + role. Always editorial_status=draft. Usage: /kb-editorial <magazine-slug> <issue-id> [--voice burggraf|schliebe|blanke] [--vignette <hint>]
trigger: /kb-editorial
allowed-tools: Read, Write, Edit, Bash
---

# KB Editorial

Generate the issue Editorial — a 1-page essayistic foreword in one of the magazine's defined voice profiles.

## Steps

### 1. Read Spec & Plan

```bash
cat ~/.claude/skills/magazines/MAGAZINE-{MAGAZINE_SLUG_UPPER}.md
cat {KB_PATH}/wiki/issues/{ISSUE_ID}/issue-plan.md
```

Internalize §5 (Editorial pattern) and the voice profile table.

### 2. Parse Arguments

- `<magazine-slug> <issue-id>` required
- `--voice` defaults from spec's latest active profile (currently `blanke` for `info-np`)
- `--vignette <hint>` optional — a topic, anecdote, or recent event the Editorial should hook off (e.g., "Geburtstag von Tau-Protein-Entdecker", "Recent EAN consensus paper", "Swissmedic decision on …")

### 3. Identify Issue's Anchor Themes

From the issue plan, extract the 2–3 strongest themes (locked CME topics + most prominent Medizin pieces). The Editorial should bridge from a vignette to one or more of these themes.

### 4. Compose

#### Voice profile rules (verbatim from spec §5)

| Profile | Tone signals |
|---|---|
| `burggraf` | Warm, colloquial, metaphor-driven. Headline often punning or culturally referenced ("Tau-frisch im Alter!?", "Ein Indianerherz kennt keinen Schmerz – oder doch?"). Closer: `Wir wünschen viel Spass bei der Lektüre!` or `Viel Freude bei der Lektüre.` |
| `schliebe` | Sober, academic, evidence-anchored ("Damit gilt als plausibel, dass …"). Headline descriptive, no wordplay. Closer: `Ich wünsche Ihnen eine erkenntnisreiche Lektüre der vorliegenden Ausgabe der InFo Neurologie & Psychiatrie.` |
| `blanke` | Wry, colloquial, occasionally satirical ("biokular schauen", "Lernen wir schielen"). Headline imperative or programmatic ("Zeit zum Schielen: Mensch vs. Algorithmus"). Closer: same as `schliebe`. |

#### Structure (universal)

1. **Header bar (verbatim formula)**:
   ```
   Die Fortbildungsthemen in dieser Ausgabe:
   {Topic 1} ............................. Seite {N}
   {Topic 2 (if 2 CMEs)} .................. Seite {N}
   CME-Fortbildungsfragen ................. Seite {N}

   Credits auf medizinonline.com — Einloggen, Fragen beantworten und direkt CME-Zertifkat downloaden.
   ```
   For single-CME issues, use `Das Fortbildungsthema in dieser Ausgabe:` (singular). Preserve typo `Zertifkat`.

2. **Kicker** (small, e.g. "Subdurales Hämatom", "Multiple Sklerose", "KI in der Medizin")

3. **Headline** in the chosen voice's pattern. ~3–10 words.

4. **Body** (~350–500 words):
   - Opens with ▪ glyph
   - First paragraph: a vignette (cultural, historical, news, personal anecdote, scientific curio)
   - Middle paragraphs: bridge from the vignette to the issue's themes; reference the locked CME / Medizin topics by name
   - Voice-appropriate tone throughout

5. **Closer**: voice-specific (see profile table above)

6. **Signature** (two lines, no decoration, no "Mit freundlichen Grüssen"):
   ```
   {Full name}
   {Redaktion | Chefredaktion}
   ```
   For `blanke` profile, optionally add email below the signature line.

### 5. Frontmatter

```yaml
---
type: editorial
magazine: {MAGAZINE_SLUG}
magazine_section: editorial
issue_id: {ISSUE_ID}
lang: {LANG}
editorial_status: draft
voice_profile: {burggraf|schliebe|blanke|custom}
signed_by:
  name: {full name}
  role: {Redaktion|Chefredaktion}
  email: {optional}
vignette_hook: {free-text description of the hook used}
themes_bridged: [theme-slug-1, theme-slug-2]
target_words: 400
created_at: {now}
updated_at: {now}
tags: [editorial, {issue_id}, voice-{profile}]
---
```

### 6. Style Discipline

- "Sie" and "wir" allowed (this is one of the few sections where direct address is permitted)
- Metaphors and humour permitted per voice profile
- Hedging still applies for any factual claims about evidence
- Swiss orthography
- Anführungszeichen «...»
- The editorial header bar with CME page references is mandatory and verbatim — even on issues with only 1 CME (use singular)

### 7. Write

Set `FILE = wiki/issues/{ISSUE_ID}/editorial.md`.

### 8. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: editorial {ISSUE_ID} (voice: {profile})"
```

### 9. Print

```
Editorial-Entwurf erstellt: {FILE}
  Voice: {profile}
  Wörter: ~{N}
  Vignette: {hook description}
  Themen-Bridge: {themes}
  Status: draft

Hinweis: Editorial wird zuletzt im Heftzyklus geschrieben — sobald CMEs und Medizin-Beiträge stehen,
kann die Themen-Bridge präziser auf finale Inhalte verweisen. Re-run after content lock empfohlen.
```

## Notes

- The Editorial header bar is one of the magazine's strongest fingerprints — preserve the line spacing, wording, dotted-leader page references, and the `Zertifkat` typo verbatim.
- Voice mismatch is a common error mode: a `blanke` editorial that reads too sober suggests the wit dial is set too low; a `schliebe` editorial that reads too colloquial suggests the dial is set too high. After draft, run `/kb-style-lint --voice {profile}` to flag mismatches.
- For a new editorial voice (e.g., a new Chefredaktion successor), pass `--voice custom` and the skill will use a generic structure; the spec should be updated with the new profile's rules after a few issues observed.
- The Editorial is **the editor's voice** — Claude produces a draft, but the named signer is expected to revise to taste. Never auto-promote past `draft`.
