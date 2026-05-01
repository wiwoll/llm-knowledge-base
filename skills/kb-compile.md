---
name: kb-compile
description: Compile all uncompiled raw/ content into the wiki. Writes source summaries, creates/updates concept articles with Obsidian backlinks, and updates the index. New concept articles are always written with editorial_status=draft (use /kb-review to promote). Run after /kb-ingest or /kb-source to process new content.
trigger: /kb-compile
allowed-tools: Read, Write, Edit, Bash
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

**Schema policy:** all new wiki artefacts conform to `~/.claude/skills/SCHEMA.md`. In particular:
- Every wiki artefact gets `lang`, `tags`, `created_at`, `updated_at`.
- Concept articles get `type: concept` and **always** `editorial_status: draft`. Never set any other editorial status — promotion is the job of `/kb-review`.
- Source summaries propagate medical metadata (`doi`, `pmid`, `pmcid`, `journal`, `journal_abbrev`, `publication_date`, `authors`, `study_type`, `evidence_level`, `peer_reviewed`) verbatim from the raw frontmatter when present. Omit fields that are absent — do not synthesize them.
- When updating an existing concept article, refresh `updated_at` but leave `editorial_status`, `reviewed_by`, `reviewed_at` untouched. If editorial_status was `published`, **do not modify the body**: append a comment to the manifest entry noting that compilation skipped a published article and continue. Surface this to the user at the end via the summary.

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

Parse the YAML frontmatter completely. Capture:
- `source`, `ingested_at`, `type`, `lang`, `tags`
- Medical metadata when present: `doi`, `pmid`, `pmcid`, `journal`, `journal_abbrev`, `publication_date`, `authors`, `study_type`, `evidence_level`, `peer_reviewed`, `funding_disclosed`, `coi_disclosed`, `population`, `sample_size`

If `lang` is missing in older raw files, fall back to `DEFAULT_LANG`.
Set `SOURCE_LANG` to the parsed `lang`. Set `MEDICAL_META` to the dict of medical fields that were present (omit absent ones). The content below the frontmatter block is the main body.

---

#### 4b. Write source summary

Derive `SOURCE_SLUG` from `{RAW_KEY}`: take the filename portion without extension.
Example: `raw/web/abs-1706-03762.md` → `SOURCE_SLUG` = `abs-1706-03762`

Write to `{KB_PATH}/wiki/sources/{SOURCE_SLUG}.md`:

```markdown
---
source: {value of `source` from raw frontmatter}
ingested_at: {value of `ingested_at` from raw frontmatter}
created_at: {value of `created_at` from raw frontmatter, or ingested_at if absent}
updated_at: {current UTC ISO 8601 timestamp}
type: {value of `type` from raw frontmatter}
lang: {SOURCE_LANG}
tags: [{merge tags from raw frontmatter with any additional 3–8 lowercase tags you derive from content; deduplicate}]
{For each field in MEDICAL_META, emit it on its own line preserving the value verbatim:
  doi: {DOI}
  pmid: {PMID}
  pmcid: {PMCID}
  journal: {journal}
  journal_abbrev: {journal_abbrev}
  publication_date: {publication_date}
  authors: [{authors}]
  study_type: {study_type}
  evidence_level: {evidence_level}
  peer_reviewed: {peer_reviewed}
  funding_disclosed: {funding_disclosed}
  coi_disclosed: {coi_disclosed}
  population: {population}
  sample_size: {sample_size}
}
---

# {Title: infer from content, URL, or filename — keep in source language}

## Summary
{2–4 sentence summary in {SOURCE_LANG} of the source's main contribution, argument, or subject matter}

## Key Concepts
{Bulleted list of 3–8 key concepts this source covers, each formatted as [[concepts/{concept-slug}]] — {brief description in {SOURCE_LANG}}. The concept slug itself MUST be the canonical slug used in DEFAULT_LANG (so all sources point to the same concept article regardless of their own language). Example: a German article about "Aufmerksamkeitsmechanismus" still links to [[concepts/aufmerksamkeitsmechanismus]] if DEFAULT_LANG is "de", or to [[concepts/attention-mechanism]] if DEFAULT_LANG is "en".}

## Notable Details
{Any specific facts, figures, quotes, findings, or techniques worth preserving verbatim — in their original language. For medical sources, prioritize: primary endpoint result with effect size + 95% CI + p-value, sample size, study population, key adverse events, limitations explicitly stated by authors.}

## Bibliographic Citation
{Only emit this section if the source has medical metadata. Format:}
{authors joined with ", "}. {title}. *{journal}*. {publication_date};{volume}({issue}):{pages}. {if doi}doi:{DOI}. {/if}{if pmid}PMID: {PMID}.{/if}

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
type: concept
editorial_status: draft
tags: [{relevant tags from the source}]
created_at: {current UTC ISO 8601}
updated_at: {current UTC ISO 8601}
---

# {Concept Name in {DEFAULT_LANG} (title-case)}

{2–4 paragraph article in {DEFAULT_LANG} explaining this concept clearly. Write it as a standalone reference: define the concept, explain why it matters, describe how it works, and note any important variants or related ideas. Assume the reader knows the field but is encountering this concept for the first time. Translate / synthesize the source content into {DEFAULT_LANG}; do not copy verbatim from a foreign-language source.

For medical concepts, the article structure should follow this template:
1. **Definition** — what is it, in plain language
2. **Mechanism / Pathophysiology** — how it works (drug mechanism, disease pathophysiology, etc.) when relevant
3. **Evidence base** — what the strongest available studies show, with effect sizes and confidence intervals when known. Cite source articles inline using [[sources/{slug}]].
4. **Clinical relevance / Practical implications** — what this means for editorial coverage / patient care.
5. **Open questions / Caveats** — explicitly mark what is uncertain, contested, or based on weak evidence.

For non-medical concepts, use a free 2–4 paragraph structure.}

## Sources
- [[sources/{SOURCE_SLUG}]]
```

**Citation discipline:** every factual claim about effect sizes, mechanisms, or recommendations must cite a `[[sources/{slug}]]` inline. If a claim cannot be sourced from the wiki content, do not include it.

**If `{KB_PATH}/wiki/concepts/{concept-slug}.md` DOES exist:**

Read it. Check `editorial_status`:
- If `published`: skip body modification entirely. Only append the new source to `## Sources` and update `updated_at`. Note in the manifest entry's compile log that this article was skipped due to its published status.
- If `fact-checked`: append source and update `updated_at` and **demote `editorial_status` back to `draft`** since the body changed. This forces re-review.
- If `in-review` or `draft`: proceed with full update as before.

For all non-published cases:
1. Add any new information from the current source not already covered in the article body. Write the new prose in the article's existing `lang` (which is `DEFAULT_LANG` for concepts). If the source is in a different language, translate the relevant insight into the article's language before integrating.
2. Append `- [[sources/{SOURCE_SLUG}]]` to the `## Sources` section if not already present
3. Set `updated_at` to current UTC ISO 8601

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
