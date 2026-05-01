---
name: kb-translate
description: Create or refresh a translation of a concept article in another supported language. Maintains bidirectional linkage via translation_of and translations frontmatter fields. Always preserves the canonical article and produces a draft translation that requires /kb-review before use. Usage: /kb-translate <article-slug> <target-lang>
trigger: /kb-translate
allowed-tools: Read, Write, Edit, Bash
---

# KB Translate

Produce a translation of an existing concept article into another supported language. Translations are **derived artefacts** — the canonical-language version remains the source of truth, and translations are kept in sync via the `translations` map on the canonical and `translation_of` pointer on the derivative.

## Steps

### 1. Read Config

```bash
cat ~/.claude/kb-config.json
```

Extract `kb_path`, `default_output_lang`, `supported_langs`. Set `KB_PATH`, `DEFAULT_LANG`, `SUPPORTED_LANGS`.

### 2. Parse Arguments

Two positional arguments:
- `<article-slug>` — the source article slug under `wiki/concepts/`
- `<target-lang>` — ISO 639-1, must be in `SUPPORTED_LANGS` and **different from** the source's `lang`

If either is missing, print:
```
Usage: /kb-translate <article-slug> <target-lang>
Example: /kb-translate glp-1-agonisten en
```
And stop.

If `<target-lang>` is not in `SUPPORTED_LANGS`, print error and list supported languages.

### 3. Resolve Canonical

Read `{KB_PATH}/wiki/concepts/{slug}.md`. If it has `translation_of: {canonical-slug}`, the user is asking to translate a translation — redirect to the canonical:

```
{slug} is a translation of {canonical-slug}. Translating from canonical instead.
```

Set `CANONICAL_SLUG` to either the resolved canonical or the originally provided slug.

Read the canonical. Note its `lang` (`SOURCE_LANG`).

If `SOURCE_LANG == TARGET_LANG`, print: `Source and target are the same language ({lang}). Nothing to do.` and stop.

### 4. Determine Target Slug

Conventions for translation slug:
- For a German canonical (`glp-1-agonisten`) translating to English → `glp-1-agonists`
- Use the most natural slug in the target language; do not just append `-en` or similar
- If the natural target slug already exists as a separate concept article, append `-{target-lang}` to disambiguate

Check whether `{KB_PATH}/wiki/concepts/{TARGET_SLUG}.md` already exists.

| Existing? | Action |
|---|---|
| No | Create new translation file |
| Yes, with `translation_of: {CANONICAL_SLUG}` | Refresh existing translation (the canonical has changed since the last translation) |
| Yes, but **not** linked to the canonical | Print conflict: `{TARGET_SLUG} already exists as an independent concept. Use /kb-merge to combine, or pass an explicit different slug.` and stop. |

### 5. Translate

Translate the canonical article body into `TARGET_LANG`:
- Preserve all `[[wiki-link]]` slugs verbatim — do not translate slugs
- Translate the article title
- Translate the body, headings, and any inline prose
- Preserve `## Sources` and `## Connected Concepts` as section headings translated into `TARGET_LANG` (e.g., `## Sources` → `## Quellen` for German)
- Leave the slug targets in those sections untouched
- Match the canonical's tone, depth, and length — do not summarize or expand
- For medical terminology, prefer the standard term in the target language. If a German source uses "Aufmerksamkeitsdefizit-/Hyperaktivitätsstörung", translate to "attention-deficit/hyperactivity disorder" (not "ADHD" unless the canonical also abbreviates).

### 6. Write Translation

Set `CREATED_AT` to current UTC ISO 8601.

Write to `{KB_PATH}/wiki/concepts/{TARGET_SLUG}.md`:

```markdown
---
lang: {TARGET_LANG}
type: concept
editorial_status: draft
tags: [{same tags as canonical}]
translation_of: {CANONICAL_SLUG}
translated_at: {CREATED_AT}
canonical_revision: {commit short hash of the canonical at translation time, from `git log -1 --format=%h -- wiki/concepts/{CANONICAL_SLUG}.md`}
created_at: {CREATED_AT}
updated_at: {CREATED_AT}
---

{Translated title as `# Heading`}

{Translated body}

## {Localized "Connected Concepts"}
{Same wiki-links as canonical}

## {Localized "Sources"}
{Same wiki-links as canonical}
```

### 7. Update Canonical

Read the canonical article. If it does not have a `translations:` map, add one. Add the entry:

```yaml
translations:
  {TARGET_LANG}: {TARGET_SLUG}
```

If a `translations` entry for `{TARGET_LANG}` already exists pointing elsewhere, overwrite it (with a warning) — but only if the existing slug really is stale.

Update the canonical's `updated_at` (the canonical itself wasn't edited, but its translations map was — note this carefully; some teams may prefer to leave `updated_at` untouched here. Default behavior: update it).

### 8. Update Index

In `{KB_PATH}/wiki/index.md`:

Append under `## Konzepte / Concepts`:
```
- [[concepts/{TARGET_SLUG}]] — {translated one-line description} `[draft] [{TARGET_LANG}]` (Übersetzung von [[concepts/{CANONICAL_SLUG}]])
```

If the canonical's index entry does not yet show its language, optionally augment it with `[{SOURCE_LANG}]`. Be idempotent — don't duplicate the badge if it already exists.

### 9. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: translate concepts/{CANONICAL_SLUG} → concepts/{TARGET_SLUG} ({TARGET_LANG})"
```

### 10. Print Confirmation

```
Translated: concepts/{CANONICAL_SLUG} ({SOURCE_LANG}) → concepts/{TARGET_SLUG} ({TARGET_LANG})
Status: draft (run /kb-review concepts/{TARGET_SLUG} after a human review)
```

## Notes

- Translations carry `editorial_status: draft` regardless of the canonical's status. A translation must be re-reviewed even if the canonical was already `published`.
- When the canonical changes, this skill **does not auto-detect staleness**. `/kb-lint` flags translations whose `canonical_revision` no longer matches the canonical's latest commit hash for the file — re-run `/kb-translate` on those.
- This skill works only on `wiki/concepts/`. To translate a Q&A answer or report, run `/kb-ask` directly in the target language instead.
