---
name: kb-compile
description: Compile all uncompiled raw/ content into the wiki. Writes source summaries, creates/updates concept articles with Obsidian backlinks, and updates the index. Run after /kb-ingest to process new content.
trigger: /kb-compile
---

# KB Compile

Batch process all uncompiled raw files into the wiki. Incremental — only processes files with `status: uncompiled` in the manifest.

## Steps

### 1. Read Config

Run:
```bash
cat ~/.claude/kb-config.json
```

Extract `kb_path`. Expand `~` to the actual home directory path.
Set this as `KB_PATH` for all subsequent steps.

Also extract `default_output_lang` (default `"de"` if missing) and `supported_langs` (default `["de","en"]` if missing).
Set these as `DEFAULT_LANG` and `SUPPORTED_LANGS`.

**Language policy for compilation:**
- **Source summaries** (`wiki/sources/`): write in the source's own language (the `lang` value from the raw frontmatter). This preserves the voice of the source.
- **Concept articles** (`wiki/concepts/`): write in `DEFAULT_LANG` (the editorial output language). This keeps the conceptual layer consistent for the team.
- **Index entries** (`wiki/index.md`): write one-line descriptions in `DEFAULT_LANG`.

### 2. Read Manifest

Run:
```bash
cat {KB_PATH}/.kb/manifest.json
```

Collect all entries where `status` is `"uncompiled"`. If there are none, print `Nothing to compile.` and stop.

### 3. Read Existing Index

Run:
```bash
cat {KB_PATH}/wiki/index.md
```

Keep this in memory — you will append to it throughout this process.

### 4. Process Each Uncompiled File

For each file at `{RAW_KEY}` with `status: uncompiled`, do the following sub-steps in order.

---

#### 4a. Read the raw file

Read `{KB_PATH}/{RAW_KEY}` using the Read tool.

Parse the YAML frontmatter to get `source`, `ingested_at`, `type`, and `lang`.
If `lang` is missing in older raw files, fall back to `DEFAULT_LANG`.
Set `SOURCE_LANG` to the parsed `lang`. The content below the frontmatter block is the main body.

---

#### 4b. Write source summary

Derive `SOURCE_SLUG` from `{RAW_KEY}`: take the filename portion without extension.
Example: `raw/web/abs-1706-03762.md` → `SOURCE_SLUG` = `abs-1706-03762`

Write to `{KB_PATH}/wiki/sources/{SOURCE_SLUG}.md`:

```markdown
---
source: {value of `source` from raw frontmatter}
ingested_at: {value of `ingested_at` from raw frontmatter}
type: {value of `type` from raw frontmatter}
lang: {SOURCE_LANG}
tags: [{3–8 lowercase tags you assign based on content, comma-separated, e.g. ml, transformers, attention}]
---

# {Title: infer from content, URL, or filename — keep in source language}

## Summary
{2–4 sentence summary in {SOURCE_LANG} of the source's main contribution, argument, or subject matter}

## Key Concepts
{Bulleted list of 3–8 key concepts this source covers, each formatted as [[concepts/{concept-slug}]] — {brief description in {SOURCE_LANG}}. The concept slug itself MUST be the canonical slug used in DEFAULT_LANG (so all sources point to the same concept article regardless of their own language). Example: a German article about "Aufmerksamkeitsmechanismus" still links to [[concepts/aufmerksamkeitsmechanismus]] if DEFAULT_LANG is "de", or to [[concepts/attention-mechanism]] if DEFAULT_LANG is "en".}

## Notable Details
{Any specific facts, figures, quotes, findings, or techniques worth preserving verbatim — in their original language}

## Backlinks
- Source file: [[{RAW_KEY without .md extension}]]
```

---

#### 4c. Create or update concept articles

From the Key Concepts list you wrote in 4b, extract each concept slug (the part inside `[[concepts/{concept-slug}]]`).

For each concept slug:

**If `{KB_PATH}/wiki/concepts/{concept-slug}.md` does NOT exist:**

Create it:
```markdown
---
lang: {DEFAULT_LANG}
tags: [{relevant tags from the source}]
---

# {Concept Name in {DEFAULT_LANG} (title-case)}

{2–4 paragraph article in {DEFAULT_LANG} explaining this concept clearly. Write it as a standalone reference: define the concept, explain why it matters, describe how it works, and note any important variants or related ideas. Assume the reader knows the field but is encountering this concept for the first time. Translate / synthesize the source content into {DEFAULT_LANG}; do not copy verbatim from a foreign-language source.}

## Sources
- [[sources/{SOURCE_SLUG}]]
```

**If `{KB_PATH}/wiki/concepts/{concept-slug}.md` DOES exist:**

Read it. Then update it:
1. Add any new information from the current source not already covered in the article body. Write the new prose in the article's existing `lang` (which is `DEFAULT_LANG` for concepts). If the source is in a different language, translate the relevant insight into the article's language before integrating.
2. Append `- [[sources/{SOURCE_SLUG}]]` to the `## Sources` section if not already present

---

#### 4d. Update wiki/index.md

For each new concept article created in 4c (skip if the concept entry already exists in the index):

Append under `## Konzepte / Concepts` (use whichever heading exists in the index; if both, use the `## Konzepte / Concepts` combined heading):
```
- [[concepts/{concept-slug}]] — {one-line description in {DEFAULT_LANG}}
```

For the source summary (skip if already in index):

Append under `## Quellen / Sources`:
```
- [[sources/{SOURCE_SLUG}]] — {one-line description in {DEFAULT_LANG}: what this source is and its main contribution; include the source's own lang in parentheses if different from DEFAULT_LANG, e.g. "(en)"}
```

Only add entries not already present. Check by scanning existing index content.

---

#### 4e. Update manifest entry for this file

Update the entry for `{RAW_KEY}` in the in-memory manifest JSON:

```json
"{RAW_KEY}": {
  "status": "compiled",
  "ingested_at": "{original ingested_at}",
  "compiled_at": "{current UTC ISO timestamp}",
  "source": "{original source}",
  "type": "{original type}",
  "lang": "{SOURCE_LANG}",
  "wiki_articles": ["sources/{SOURCE_SLUG}.md", "concepts/{slug1}.md", "concepts/{slug2}.md"],
  "tags": ["{tags you assigned in 4b}"]
}
```

---

### 5. Write Updated Files

After processing all uncompiled files:

1. Write the full updated `wiki/index.md` back to disk (with all appended entries)
2. Write the full updated manifest back to `{KB_PATH}/.kb/manifest.json`

### 6. Rebuild Search Index

If `kb_search.py` exists in `{KB_PATH}`, rebuild the search index:

```bash
python3 {KB_PATH}/kb_search.py --rebuild
```

If the file doesn't exist (first run before search tool is installed), skip this step silently.

### 7. Commit Changes

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: compile {N} source(s) into wiki"
```

Where N is the count of files just compiled.

### 8. Run Reflect

Invoke the `/kb-reflect` skill to discover connections across newly compiled content.

### 9. Print Summary

```
Compiled {N} file(s):
  - {RAW_KEY_1} → wiki/sources/{slug1}.md, wiki/concepts/...
  - {RAW_KEY_2} → wiki/sources/{slug2}.md, wiki/concepts/...
```
