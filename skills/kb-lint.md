---
name: kb-lint
description: Run health checks on the wiki. Finds thin articles, missing concepts, broken wikilinks, duplicate concepts, schema violations (missing lang/editorial_status/medical metadata for medical sources), stale translations (canonical revision changed), and suggests new article candidates. Prints a terminal summary and saves a full report to outputs/.
trigger: /kb-lint
allowed-tools: Read, Write, Edit, Bash, WebSearch
---

# KB Lint

Run 5 sequential health checks on the wiki using index-first navigation. Prints a terminal summary and saves a full report to `outputs/`.

## Steps

### 1. Read Config

```bash
cat ~/.claude/kb-config.json
```

Extract `kb_path`. Expand `~` to the actual home directory path.
Set this as `KB_PATH` for all subsequent steps.

Also extract `default_output_lang` (default `"de"`) and `supported_langs` (default `["de","en"]`).
Set these as `DEFAULT_LANG` and `SUPPORTED_LANGS`. The lint report and all rationales/suggestions are written in `DEFAULT_LANG`.

**Additional language-aware check:** also collect `lang_inconsistencies` — concept articles whose `lang` frontmatter does not match `DEFAULT_LANG`, and source articles missing a `lang` field. These are reported but only flagged as warnings (not errors) since older content may predate the language policy.

### 2. Read the Index

```bash
cat {KB_PATH}/wiki/index.md
```

This is your navigation map for all checks. Parse the entries under `## Concepts` and `## Sources` to get the full list of known articles.

Initialize a results object to collect findings from each check:
```
thin_articles: []
missing_concepts: []
broken_links: []
duplicate_concepts: []
new_suggestions: []
lang_inconsistencies: []
schema_violations: []        # missing required fields per SCHEMA.md
medical_metadata_gaps: []    # medical sources missing doi/pmid/study_type/evidence_level
editorial_status_gaps: []    # concept articles missing editorial_status
stale_translations: []       # translation_of references with mismatched canonical_revision
unverified_publications: []  # editorial_status=published but missing reviewed_by/reviewed_at
```

---

### 3. Check A — Thin Articles

For each entry under `## Concepts` in the index:

1. Read the article: `{KB_PATH}/wiki/concepts/{slug}.md`
2. Strip the YAML frontmatter and `## Sources` section
3. Count substantive sentences in the remaining body (ignore blank lines and single-word headings)
4. If fewer than 3 substantive sentences: add to `thin_articles`:
   ```
   { "file": "concepts/{slug}.md", "sentence_count": N }
   ```

---

### 4. Check B — Missing Concepts

For each entry under `## Sources` in the index:

1. Read the article: `{KB_PATH}/wiki/sources/{slug}.md`
2. Extract all `[[concepts/X]]` wikilinks from the `## Key Concepts` section
3. For each concept slug `X`, check if `{KB_PATH}/wiki/concepts/{X}.md` exists:
   ```bash
   ls {KB_PATH}/wiki/concepts/{X}.md 2>/dev/null
   ```
4. If the file does not exist, add to `missing_concepts`:
   ```
   { "concept": "X", "referenced_in": "sources/{slug}.md" }
   ```

Deduplicate: if the same missing concept is referenced in multiple sources, list it once with all referencing sources.

---

### 5. Check C — Broken Wikilinks

Scan all `.md` files in `{KB_PATH}/wiki/`:

```bash
grep -r '\[\[' {KB_PATH}/wiki/ --include="*.md" -h
```

For each `[[target]]` found:
1. Strip the `[[` and `]]`
2. If the target contains a `/`, it's an explicit path — check if `{KB_PATH}/wiki/{target}.md` exists
3. If no `/`, search for `{KB_PATH}/wiki/**/{target}.md`
4. If the file does not exist, add to `broken_links`:
   ```
   { "link": "[[target]]", "found_in": "{file path}" }
   ```

---

### 6. Check D — Duplicate Concepts

From the list of concept slugs parsed in Step 2, identify near-duplicate pairs:

- **Substring match:** one slug is a substring of another (e.g. `attention` and `attention-mechanism`)
- **Word overlap:** two slugs share 2+ words when split by `-`

For each candidate pair, add to `duplicate_concepts`:
```
{ "slug_a": "attention", "slug_b": "attention-mechanism", "reason": "substring match" }
```

---

### 6.5. Check D2 — Schema Compliance

For each `.md` file in `wiki/concepts/`, `wiki/sources/`, `outputs/`:

1. Read the YAML frontmatter
2. Check the schema rules from SCHEMA.md:
   - **All artefacts**: must have `lang`, `tags`, `created_at`, `updated_at`
   - **Concept articles**: must additionally have `type` ∈ {`concept`, `synthesis`} and `editorial_status` ∈ {`draft`, `in-review`, `fact-checked`, `published`}
   - **`fact-checked` or `published` concepts**: must have `reviewed_by` and `reviewed_at`
   - **Source articles**: must have `source`, `type`, `ingested_at`. If the source URL contains `pubmed`, `doi.org`, `nejm`, `thelancet`, `bmj`, `jamanetwork`, or other recognizably medical domains, flag missing `doi`/`pmid`/`study_type`/`evidence_level` as `medical_metadata_gaps`.
3. For each violation, add to the appropriate list (`schema_violations` for missing core fields, `editorial_status_gaps` for concept-specific issues, `unverified_publications` for fact-check issues, `medical_metadata_gaps` for medical metadata).

### 6.6. Check D3 — Translation Staleness

For each concept article with `translation_of: {canonical-slug}`:

1. Read the article's `canonical_revision` value
2. Get the canonical's current commit hash for its file:
   ```bash
   cd {KB_PATH} && git log -1 --format=%h -- wiki/concepts/{canonical-slug}.md
   ```
3. If the hashes differ, add to `stale_translations`:
   ```
   { "translation": "concepts/{slug}.md", "canonical": "concepts/{canonical-slug}.md",
     "recorded_revision": "{canonical_revision}", "current_revision": "{current_hash}" }
   ```

---

### 7. Check E — New Article Suggestions

**Step E1 — Wiki gap analysis:**

From the `missing_concepts` list (Check B), rank by frequency of cross-source references. The most-referenced missing concepts are the highest-priority candidates.

Also scan source summaries for concept terms that appear repeatedly in body text but are not indexed under `## Concepts` in `index.md`.

Compile up to 5 gap candidates with rationale (e.g. "referenced 3× across sources").

**Step E2 — Web search (optional enrichment):**

For each thin article found in Check A, perform a focused web search:
```
WebSearch: "{concept name} explained"
```

Summarize the top result in 1–2 sentences. This becomes the "web-imputed" note in the report.

For each gap candidate from E1 not already in the wiki, perform:
```
WebSearch: "{concept name} overview"
```

Summarize the top result in 1–2 sentences as supporting context for the suggestion.

Add all suggestions to `new_suggestions`:
```
{ "concept": "flash-attention", "reason": "web search: modern efficient attention variant relevant to existing content", "web_imputed": true }
```

---

### 8. Write Full Report

Set `REPORT_DATE` to today's date in `YYYY-MM-DD` format.
Set `REPORT_FILE` = `outputs/{REPORT_DATE}-kb-lint-report.md`

Write to `{KB_PATH}/{REPORT_FILE}` (in `DEFAULT_LANG`):

```markdown
---
lang: {DEFAULT_LANG}
type: lint-report
generated_at: {current UTC ISO timestamp}
---

# KB Lint Report — {REPORT_DATE}

## Thin Articles
{If none: localized "None found."}
{For each: "- [[{file without .md}]] — {N} sentences (needs expansion)"}

## Missing Concepts
{If none: localized "None found."}
{For each: "- `[[concepts/{concept}]]` referenced in {sources} but no article exists"}

## Broken Wikilinks
{If none: localized "None found."}
{For each: "- `[[{link}]]` in {found_in}"}

## Duplicate Concepts
{If none: localized "None found."}
{For each: "- `{slug_a}` and `{slug_b}` may overlap ({reason}) — consider merging"}

## Language Inconsistencies
{If none: localized "None found."}
{For each: "- {file} — lang: {actual} (expected {DEFAULT_LANG} for concept) / missing lang field"}

## Schema Violations
{If none: localized "None found."}
{For each violation: "- {file} — missing required field(s): {list}"}

## Editorial Status Gaps
{If none: localized "None found."}
{For each: "- {file} — {reason: missing editorial_status / fact-checked but missing reviewer / etc.}"}

## Medical Metadata Gaps
{If none: localized "None found."}
{For each: "- {file} (medical source) — missing: {list of missing medical fields}. Suggest: re-ingest via /kb-source <DOI|PMID>."}

## Stale Translations
{If none: localized "None found."}
{For each: "- {translation} is based on canonical revision {recorded} but canonical is now at {current}. Re-run /kb-translate {canonical} {lang}."}

## Unverified Publications
{If none: localized "None found."}
{For each: "- {file} marked as published but missing reviewed_by/reviewed_at. Run /kb-review {slug} --status published --reviewer @username."}

## New Article Suggestions
{If none: localized "No suggestions."}
{For each: "- **{concept}** — {reason in {DEFAULT_LANG}}{if web_imputed: ' *(web-imputed)*'}"}
```

All headings, rationales, and "None found"/"No suggestions" placeholders are localized into `DEFAULT_LANG` (e.g., "Keine gefunden." / "Keine Vorschläge." for German).

---

### 9. Update Index

Append to `## Antworten & Berichte / Outputs` (or `## Outputs` if older index format) in `{KB_PATH}/wiki/index.md`:

```
- [[{REPORT_FILE without .md extension}]] — lint report: {N} issues found, {M} suggestions
```

Write the updated index back to disk.

### 10. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: lint report {REPORT_DATE}"
```

### 11. Print Terminal Summary

```
KB Lint Report — {REPORT_DATE}
──────────────────────────────
{✗ if any, ✓ if none} Thin articles:       {N} found
{✗ if any, ✓ if none} Missing concepts:    {N} found
{✗ if any, ✓ if none} Broken wikilinks:    {N} found
{✗ if any, ✓ if none} Duplicate concepts:  {N} pairs found
  New suggestions:     {N} candidates
──────────────────────────────
Full report: {REPORT_FILE}
```
