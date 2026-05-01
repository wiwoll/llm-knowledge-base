---
name: kb-ingest
description: Ingest a URL, PDF, image path, or plain text note into your personal knowledge base raw/ directory. Registers the content in the manifest for later compilation. For medical sources identified by DOI or PubMed ID, prefer /kb-source instead — it auto-resolves bibliographic metadata. Usage: /kb-ingest <url|path|note text>
trigger: /kb-ingest
allowed-tools: Read, Write, Edit, Bash, WebFetch
---

# KB Ingest

Stage content into the knowledge base `raw/` directory. Does not compile or modify the wiki.

## Steps

### 1. Read Config

Run:
```bash
cat ~/.claude/kb-config.json
```

Extract `kb_path`. Expand `~` to the actual home directory path (run `echo ~` if needed).
Set this as `KB_PATH` for all subsequent steps.

Also extract `default_output_lang` (default `"de"` if missing) and `supported_langs` (default `["de","en"]` if missing).
Set these as `DEFAULT_LANG` and `SUPPORTED_LANGS`.

### 2. Read Manifest

Run:
```bash
cat {KB_PATH}/.kb/manifest.json
```

Keep the parsed JSON in memory — you will update and write it back in Step 5.

### 3. Detect Input Type

The argument passed after `/kb-ingest` is the source. Classify it:

| Condition | Type |
|---|---|
| Starts with `http://` or `https://` | `web` |
| Ends with `.pdf` (case-insensitive) | `pdf` |
| Ends with `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.svg` (case-insensitive) | `image` |
| Anything else | `note` |

**Medical-source delegation:** If the source matches any of these patterns, suggest delegating to `/kb-source` (which auto-resolves bibliographic metadata via PubMed):

- DOI: matches `^10\.\d+/.+`
- PMID: matches `^\d{6,9}$`
- URL containing `pubmed.ncbi.nlm.nih.gov`, `ncbi.nlm.nih.gov/pmc`, or `doi.org/10.`

In those cases, print:
```
This looks like a medical source identifier. /kb-source <id> will auto-fetch DOI/PMID/journal/study-type metadata.
Continue with /kb-ingest anyway? [y/n]
```
If the user says no, stop and let them re-invoke `/kb-source`. Otherwise proceed with the basic web flow below.

### 4. Ingest by Type

Record the current UTC time in ISO 8601 format (e.g. `2026-04-05T10:00:00Z`) as `INGESTED_AT`. Set `CREATED_AT = INGESTED_AT` and `UPDATED_AT = INGESTED_AT` for the schema's core timestamp fields.

**Language detection:** After fetching/reading the content (sub-steps below), detect the source language. Set `LANG` to the ISO 639-1 code that matches one of `SUPPORTED_LANGS` (e.g. `de`, `en`). If detection is ambiguous or the language is not in `SUPPORTED_LANGS`, fall back to `DEFAULT_LANG`. For plain notes without enough text to detect, use `DEFAULT_LANG`.

---

#### Web

1. Fetch the URL content using the WebFetch tool. Request markdown format.
2. Extract the main article content — remove navigation bars, sidebars, footers, cookie banners, and ads. Keep headings, paragraphs, code blocks, and image references.
3. Generate a `slug` from the URL:
   - Take the URL path, strip leading/trailing slashes
   - Replace `/`, `.`, `?`, `=`, `&`, `#` with `-`
   - Lowercase, max 60 characters
   - Example: `https://arxiv.org/abs/1706.03762` → `abs-1706-03762`
4. Write to `{KB_PATH}/raw/web/{slug}.md` with this exact format:

```
---
source: {full URL}
ingested_at: {INGESTED_AT}
created_at: {CREATED_AT}
updated_at: {UPDATED_AT}
type: web
lang: {LANG}
status: uncompiled
tags: [{3–6 lowercase tags inferred from content}]
---

{cleaned markdown content}
```

Set `RAW_KEY` = `raw/web/{slug}.md`

---

#### PDF

1. Read the PDF file using the Read tool (pass the full file path).
2. Extract all text content, preserving section headings and paragraph breaks.
3. Generate a `slug` from the filename: strip `.pdf`, lowercase, replace spaces with `-`.
4. Write extracted text to `{KB_PATH}/raw/pdfs/{slug}.md` with this exact format:

```
---
source: {original file path}
ingested_at: {INGESTED_AT}
created_at: {CREATED_AT}
updated_at: {UPDATED_AT}
type: pdf
lang: {LANG}
status: uncompiled
tags: [{3–6 lowercase tags inferred from content}]
---

{extracted text}
```

5. Copy the original PDF file:
```bash
cp "{original path}" "{KB_PATH}/raw/pdfs/{slug}.pdf"
```

Set `RAW_KEY` = `raw/pdfs/{slug}.md`

---

#### Image

1. Read the image file using the Read tool (Claude will display it visually).
2. Write a detailed description of the image: what it shows, any text visible, diagrams, charts, or figures explained in words.
3. Generate a `slug` from the filename: strip extension, lowercase, replace spaces with `-`.
4. Get the original file extension (e.g. `png`).
5. Write description to `{KB_PATH}/raw/images/{slug}.md` with this exact format:

```
---
source: {original file path}
ingested_at: {INGESTED_AT}
created_at: {CREATED_AT}
updated_at: {UPDATED_AT}
type: image
lang: {LANG}
status: uncompiled
image_file: {slug}.{ext}
tags: [{3–6 lowercase tags inferred from image content}]
---

{detailed description in {LANG}}
```

The image description should be written in `LANG` if detectable from text within the image; otherwise in `DEFAULT_LANG`.

6. Copy the image:
```bash
cp "{original path}" "{KB_PATH}/raw/images/{slug}.{ext}"
```

Set `RAW_KEY` = `raw/images/{slug}.md`

---

#### Note

1. The argument is the note content directly (not a file path).
   - Exception: if the argument is a path to an existing file, read that file's contents instead.
2. Generate a `slug` from the first 6 words of the content: lowercase, join with `-`.
   - Example: "attention mechanism in transformer models" → `attention-mechanism-in-transformer-models`
3. Write to `{KB_PATH}/raw/notes/{slug}.md` with this exact format:

```
---
source: manual
ingested_at: {INGESTED_AT}
created_at: {CREATED_AT}
updated_at: {UPDATED_AT}
type: note
lang: {LANG}
status: uncompiled
tags: [{3–6 lowercase tags inferred from content; if too short to tag, use ["note"]}]
---

{content}
```

Set `RAW_KEY` = `raw/notes/{slug}.md`

---

### 5. Update Manifest

Add an entry to the manifest JSON (or update if the key already exists):

```json
"{RAW_KEY}": {
  "status": "uncompiled",
  "ingested_at": "{INGESTED_AT}",
  "source": "{source url or path}",
  "type": "{web|pdf|image|note}",
  "lang": "{LANG}"
}
```

Write the full updated JSON back to `{KB_PATH}/.kb/manifest.json`.

### 6. Confirm

Print a single confirmation line:
```
Ingested: {RAW_KEY}
```
