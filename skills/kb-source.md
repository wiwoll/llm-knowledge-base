---
name: kb-source
description: Ingest a medical source by DOI, PubMed ID, or PMC ID with full bibliographic metadata. Auto-resolves identifiers via the PubMed MCP server when available, falls back to web fetch and CrossRef otherwise. Populates the medical frontmatter (study_type, evidence_level, journal, authors, etc.). Usage: /kb-source <DOI|PMID|PMCID|URL>
trigger: /kb-source
allowed-tools: Read, Write, Bash, WebFetch, mcp__claude_ai_PubMed__search_articles, mcp__claude_ai_PubMed__get_article_metadata, mcp__claude_ai_PubMed__lookup_article_by_citation, mcp__claude_ai_PubMed__convert_article_ids, mcp__claude_ai_PubMed__get_full_text_article
---

# KB Source

Ingest a medical source with rich bibliographic metadata. Use this skill in preference to `/kb-ingest` when the source is a DOI, PubMed ID, or a URL pointing to a PubMed/PMC/journal article. For non-medical web articles, plain notes, or images, use `/kb-ingest`.

## Steps

### 1. Read Config & Schema

Run in parallel:
```bash
cat ~/.claude/kb-config.json
cat ~/.claude/skills/kb-source/SCHEMA.md 2>/dev/null || cat ~/.claude/skills/SCHEMA.md 2>/dev/null
```

Extract `kb_path`, `default_output_lang`, `supported_langs` as before. The SCHEMA defines the medical frontmatter fields you must populate.

### 2. Parse the Identifier

Detect the type of the argument:

| Pattern | Type |
|---|---|
| Matches `^10\.\d+/.+` | DOI |
| Matches `^\d{6,9}$` | PMID |
| Matches `^PMC\d+$` (case-insensitive) | PMCID |
| Starts with `https://pubmed.ncbi.nlm.nih.gov/<digits>` | PMID extracted from URL |
| Starts with `https://www.ncbi.nlm.nih.gov/pmc/articles/PMC<digits>` | PMCID extracted from URL |
| Starts with `https://doi.org/10....` | DOI extracted from URL |
| Other URL | Pass through to `/kb-ingest` web flow as fallback |
| Anything else | Print error: `Unrecognized source identifier. Use DOI, PMID, PMCID, or URL.` and stop |

Set `ID_TYPE` and `ID_VALUE`.

### 3. Resolve to Canonical IDs

If `mcp__claude_ai_PubMed__convert_article_ids` is available in the toolset:

Call it with `ID_VALUE` to get all available identifiers (DOI, PMID, PMCID). Set:
- `DOI` ← resolved DOI (or null)
- `PMID` ← resolved PMID (or null)
- `PMCID` ← resolved PMCID (or null)

If the MCP tool is **not available**, do a best-effort fallback:
- DOI → use `WebFetch` on `https://api.crossref.org/works/{DOI}` to retrieve metadata as JSON
- PMID → use `WebFetch` on `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&id={PMID}&retmode=json`

### 4. Fetch Article Metadata

If `mcp__claude_ai_PubMed__get_article_metadata` is available:

Call it with `PMID` (preferred) or DOI. Extract:
- Title
- Authors (full list; you'll truncate to 6 + "et al." for storage)
- Journal full name + NLM abbreviation
- Publication date
- Abstract
- Mesh terms (if present) → these become the basis for `tags`
- Publication types (e.g., `Randomized Controlled Trial`, `Meta-Analysis`) → map to `study_type`

If the MCP tool is not available, parse the metadata you fetched in Step 3.

### 5. Fetch Full Text (if available)

If `mcp__claude_ai_PubMed__get_full_text_article` is available **and** `PMCID` is set:

Call it to retrieve the full article text. Otherwise, work with the abstract only.

### 6. Classify Study Type and Evidence Level

Map the article's `Publication Types` and methodology to the SCHEMA's `study_type` enum:

| PubMed publication type | study_type |
|---|---|
| Randomized Controlled Trial | `rct` |
| Meta-Analysis | `meta-analysis` |
| Systematic Review | `systematic-review` |
| Observational Study + abstract mentions "cohort" | `cohort` |
| Observational Study + abstract mentions "case-control" | `case-control` |
| Cross-Sectional Studies | `cross-sectional` |
| Case Reports | `case-report` |
| Practice Guideline / Guideline | `guideline` |
| Editorial / Comment | `editorial` |
| Review (non-systematic) | `narrative-review` |
| Preprint | `preprint` |
| (none of the above) | `other` |

Assign `evidence_level` per Oxford CEBM (2011):

| study_type | typical evidence_level |
|---|---|
| `meta-analysis`, `systematic-review` (of RCTs) | `1a` |
| `rct` (single, low risk of bias) | `1b` |
| `cohort` (prospective) | `2b` |
| `cohort` (retrospective), `case-control` | `3b` |
| `cross-sectional` | `4` |
| `case-series`, `case-report` | `4` |
| `editorial`, `narrative-review` | `5` |
| `guideline` | (depends — annotate as `5` unless underlying RCT body cited) |

If unsure, use the higher (numerically larger) level — be conservative.

### 7. Detect Source Language

Set `LANG` from the article's language metadata if PubMed exposes it (`LA` field). Otherwise detect from title + abstract. Must be one of `SUPPORTED_LANGS`; fall back to `DEFAULT_LANG`.

### 8. Generate Slug

Slug priority (use first that succeeds):
1. First-author last name + year + first content word: `vaswani-2017-attention`
2. PMID: `pmid-33301246`
3. DOI tail: lowercased, slashes replaced with `-`

Set `SOURCE_SLUG`.

### 9. Write Raw File

Set `INGESTED_AT` to current UTC ISO 8601.
Set `CREATED_AT` to same.

Write to `{KB_PATH}/raw/web/{SOURCE_SLUG}.md`:

```markdown
---
source: {canonical URL — prefer https://pubmed.ncbi.nlm.nih.gov/{PMID}/ or https://doi.org/{DOI}}
ingested_at: {INGESTED_AT}
created_at: {CREATED_AT}
updated_at: {INGESTED_AT}
type: web
status: uncompiled
lang: {LANG}
tags: [{up to 8 lowercase MeSH-aligned tags}]
doi: {DOI or omit}
pmid: {PMID or omit}
pmcid: {PMCID or omit}
journal: {full journal name or omit}
journal_abbrev: {NLM abbreviation or omit}
publication_date: {YYYY-MM-DD or YYYY-MM or YYYY}
authors: [{up to 6 names; if more, last entry is "et al."}]
study_type: {value from Step 6}
evidence_level: {value from Step 6}
peer_reviewed: {true unless preprint/editorial}
---

# {Article title — preserve original language}

## Abstract

{Verbatim abstract from PubMed/CrossRef}

## Full Text

{If full text was fetched in Step 5: paste here. Otherwise: write "Full text not retrieved — only the abstract is available." Do NOT fabricate or paraphrase the full text.}

## Bibliographic Citation

{Full author list (no et al.)}. {Title}. *{Journal full name}*. {YYYY};{volume}({issue}):{pages}. doi:{DOI}. PMID: {PMID}.
```

### 10. Update Manifest

Add to `{KB_PATH}/.kb/manifest.json`:

```json
"raw/web/{SOURCE_SLUG}.md": {
  "status": "uncompiled",
  "ingested_at": "{INGESTED_AT}",
  "source": "{canonical URL}",
  "type": "web",
  "lang": "{LANG}",
  "doi": "{DOI or null}",
  "pmid": "{PMID or null}",
  "study_type": "{study_type}",
  "evidence_level": "{evidence_level}"
}
```

### 11. Confirm

```
Ingested medical source: {SOURCE_SLUG}
  Title: {title, truncated to 80 chars}
  Journal: {journal_abbrev} ({publication_date})
  Type: {study_type} (Evidence level {evidence_level})
  IDs: PMID:{PMID} DOI:{DOI}
Run /kb-compile to integrate into the wiki.
```

## Notes

- This skill **never** writes to `wiki/`. Compilation is the responsibility of `/kb-compile`, which preserves all medical frontmatter when generating source summaries and concept articles.
- If multiple identifiers are passed (e.g., `/kb-source 10.1056/nejmoa2034577 33301246`), use the first one and warn that subsequent identifiers were ignored.
- For preprints (bioRxiv, medRxiv), the same flow applies but `peer_reviewed: false` and `evidence_level: 5`.
- For non-English sources: title and abstract stay in original language; the eventual concept article in `wiki/concepts/` will be translated to `default_output_lang` by `/kb-compile`.
