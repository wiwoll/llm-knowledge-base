---
name: kb-import
description: Import an existing Obsidian vault into the knowledge base. Inspects each note and routes it intelligently — structured concept articles go to wiki/concepts/ (always editorial_status=draft, must be reviewed via /kb-review before relying on the content), raw research notes go to raw/notes/ for later compilation. Usage: /kb-import <vault-path>
trigger: /kb-import
allowed-tools: Read, Write, Edit, Bash
---

# KB Import

Import an existing Obsidian vault into the knowledge base. Each note is inspected and routed based on its content — the LLM decides whether it belongs in the compiled wiki or the raw staging area.

## Steps

### 1. Read Config

```bash
cat ~/.claude/kb-config.json
```

Extract `kb_path`. Expand `~` to the actual home directory path.
Set this as `KB_PATH` for all subsequent steps.

Also extract `default_output_lang` (default `"de"`) and `supported_langs` (default `["de","en"]`).
Set these as `DEFAULT_LANG` and `SUPPORTED_LANGS`.

**Language policy for import:** detect each note's language and store as `lang` in the frontmatter. Notes routed to `wiki/concepts/` keep their original language and `lang` value (the team can manually retranslate later via `/kb-merge` if they want consistency with `DEFAULT_LANG`). Notes routed to `raw/notes/` also store the detected `lang` so `/kb-compile` can use it.

### 2. Validate Source Vault

The argument after `/kb-import` is the source vault path. Expand `~` if present.

```bash
ls {VAULT_PATH}
```

If the directory does not exist, print:
```
Error: vault path not found: {VAULT_PATH}
```
And stop.

If `.kb/manifest.json` exists inside the vault, print:
```
This looks like a KB vault, not a plain Obsidian vault.
Use /kb-merge-vault {VAULT_PATH} instead.
```
And stop.

### 3. Scan Vault Files

Find all `.md` files in the vault, excluding hidden directories:

```bash
find {VAULT_PATH} -name "*.md" -not -path "*/.obsidian/*" -not -path "*/.trash/*" | sort
```

If no files are found, print `No markdown files found in vault.` and stop.

Set `TOTAL` = count of files found.
Print: `Found {TOTAL} notes to import. Inspecting...`

### 4. Read Manifest

```bash
cat {KB_PATH}/.kb/manifest.json
```

Keep in memory — you will update it as notes are routed to `raw/`.

### 5. Read Existing Index

```bash
cat {KB_PATH}/wiki/index.md
```

Keep in memory — you will append entries for notes routed to `wiki/concepts/`.

### 6. Classify and Route Each Note

For each `.md` file found in Step 3, read it and classify it:

**Classification criteria:**

| Route | Signals |
|---|---|
| `wiki/concepts/` | Has a clear concept definition or explanation; written as a reference article; structured with headings; could stand alone as documentation |
| `raw/notes/` | Personal shorthand or fleeting thoughts; incomplete sentences; meeting notes or journal entries; cites a specific external source (paper, article, book, URL) |

When in doubt between the two, route to `raw/notes/` — it will be compiled later.

---

#### If routed to `wiki/concepts/`:

1. Generate `SLUG` from the filename or first `#` heading: lowercase, replace spaces with `-`, strip special characters.
2. Check if `{KB_PATH}/wiki/concepts/{SLUG}.md` already exists:
   - If yes: skip this file, log as `skipped (concept already exists): {SLUG}`
3. Write to `{KB_PATH}/wiki/concepts/{SLUG}.md`:
   - Preserve all existing content exactly
   - Detect the note's language as `NOTE_LANG` (one of `SUPPORTED_LANGS`, fall back to `DEFAULT_LANG`)
   - If YAML frontmatter is missing, prepend:
     ```yaml
     ---
     lang: {NOTE_LANG}
     type: concept
     editorial_status: draft
     tags: [{infer 2-4 relevant tags from content}]
     imported_from: {original filename}
     created_at: {current UTC ISO 8601}
     updated_at: {current UTC ISO 8601}
     ---
     ```
   - If frontmatter exists, ensure `lang`, `type: concept`, `editorial_status: draft`, `imported_from`, `created_at`, `updated_at` are present (add missing ones, preserve any existing values for `lang`, `created_at`). **Always set `editorial_status: draft`** for imported articles regardless of any prior status — the new vault must verify content independently.
4. Append to `wiki/index.md` under `## Konzepte / Concepts` (or `## Concepts` for older index format), only if not already present:
   ```
   - [[concepts/{SLUG}]] — {one-line description in {DEFAULT_LANG} inferred from content; if NOTE_LANG ≠ DEFAULT_LANG, append "({NOTE_LANG})"}
   ```

---

#### If routed to `raw/notes/`:

1. Generate `SLUG` from the filename: lowercase, replace spaces with `-`, strip special characters.
2. If `{KB_PATH}/raw/notes/{SLUG}.md` already exists, append `-imported` to the slug.
3. Write to `{KB_PATH}/raw/notes/{SLUG}.md`:
   - Detect language as `NOTE_LANG` (one of `SUPPORTED_LANGS`, fall back to `DEFAULT_LANG`)
   - Prepend YAML frontmatter:
     ```yaml
     ---
     source: imported from {original file path}
     ingested_at: {current UTC ISO timestamp}
     type: note
     lang: {NOTE_LANG}
     status: uncompiled
     imported_from: {original filename}
     ---
     ```
   - Append original file content below frontmatter
4. Register in manifest:
   ```json
   "raw/notes/{SLUG}.md": {
     "status": "uncompiled",
     "ingested_at": "{current UTC ISO timestamp}",
     "source": "imported from {original file path}",
     "type": "note",
     "lang": "{NOTE_LANG}"
   }
   ```

---

### 7. Write Updated Files

1. Write the updated `wiki/index.md` back to disk
2. Write the updated `.kb/manifest.json` back to disk

### 8. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: import {N} notes from {VAULT_PATH}"
```

### 9. Print Summary

```
Import complete from: {VAULT_PATH}
─────────────────────────────────
  → wiki/concepts/   {N} notes (ready to browse in Obsidian)
  → raw/notes/       {N} notes (ready to compile)
     Skipped:        {N} (already existed)

Run /kb-compile to process the raw notes into the wiki.
```

### 10. Prompt for Compile

Ask: `Run /kb-compile now to process the imported raw notes? [y/n]`

If yes, invoke `/kb-compile`.
If no, stop.
