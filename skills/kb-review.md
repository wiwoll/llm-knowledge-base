---
name: kb-review
description: Move a concept article through the editorial fact-check workflow (draft → in-review → fact-checked → published) and attribute the reviewer. Records reviewer, timestamp, and notes in the article frontmatter. Usage: /kb-review <article-slug> [--status fact-checked|in-review|published] [--reviewer @username] [--notes "..."]
trigger: /kb-review
allowed-tools: Read, Write, Edit, Bash
---

# KB Review

Promote (or demote) a concept article through the editorial workflow defined in `SCHEMA.md`. The LLM does not silently fact-check on its own — this skill records a human reviewer's decision.

## Steps

### 1. Read Config

```bash
cat ~/.claude/kb-config.json
```

Extract `kb_path`. Set `KB_PATH`.

### 2. Parse Arguments

Required positional argument: `<article-slug>` — the slug of a concept article (without the `concepts/` prefix and without `.md`). If the user passed a full path (`concepts/foo` or `concepts/foo.md`), normalize to bare slug.

Optional flags:
- `--status <value>`: target status from the editorial enum: `draft`, `in-review`, `fact-checked`, `published`. **Default: `fact-checked`.**
- `--reviewer <name>`: reviewer attribution (free-form, GitHub username preferred). If omitted, prompt the user interactively or default to the GitHub user from `git config user.name`.
- `--notes <text>`: optional free-text review notes.

### 3. Validate Article

```bash
ls {KB_PATH}/wiki/concepts/{slug}.md
```

If not found, print:
```
Error: wiki/concepts/{slug}.md not found.
```
And stop.

### 4. Read Current Frontmatter

Read the article. Parse the YAML frontmatter. Note the current `editorial_status` (default `draft` if missing).

**Status-transition rules:**

| From → To | Allowed? | Action |
|---|---|---|
| `draft` → `in-review` | ✓ | Mark as ready for review |
| `draft` → `fact-checked` | ✓ | Direct fact-check (skip in-review) |
| `in-review` → `fact-checked` | ✓ | Standard promotion |
| `in-review` → `draft` | ✓ | Send back for rewrite (records `review_notes`) |
| `fact-checked` → `published` | ✓ | Final clearance |
| `fact-checked` → `draft` | ⚠ | Allowed but requires `--notes` explaining why |
| `published` → anything | ✗ | Refuse — published articles are locked. Print: `Article is published and locked. Create a new draft revision instead.` |
| any forward jump skipping in-review | ✓ | Allowed (e.g., draft → published if reviewer is confident) |

### 5. Update Frontmatter

Set:
- `editorial_status`: target status
- `reviewed_by`: reviewer (only for `fact-checked` and `published`)
- `reviewed_at`: current UTC ISO 8601 (only for `fact-checked` and `published`)
- `review_notes`: notes (if provided)
- `updated_at`: current UTC ISO 8601 (always)

For `draft` and `in-review` statuses, **clear** any prior `reviewed_by` and `reviewed_at` (a new review must be performed).

### 6. Write Updated Article

Edit only the frontmatter. Body content is untouched.

### 7. Update Index

Read `{KB_PATH}/wiki/index.md`. Find the line containing `[[concepts/{slug}]]`. Append a status badge at the end of that line:

```
- [[concepts/{slug}]] — {existing description} `[{editorial_status}]`
```

If the line already has a status badge, replace it.

For `published` articles also:
- Move the index entry to a `## Veröffentlicht / Published` section at the top of the index, creating that section if it does not yet exist.

For `draft` and `in-review`, leave under `## Konzepte / Concepts` with the badge.

Write the updated index back.

### 8. Commit

```bash
cd {KB_PATH} && git add -A && git commit -m "kb: review {slug} → {status} (reviewer: {reviewer})"
```

### 9. Print Confirmation

```
Reviewed: concepts/{slug}
  Status: {previous_status} → {new_status}
  Reviewer: {reviewer}
  Notes: {notes or "—"}
```

## Notes

- This skill is the **only** way to move an article past `draft`. No other skill (`/kb-compile`, `/kb-merge`, `/kb-reflect`) ever sets `editorial_status` to anything other than `draft`.
- When a `published` article needs revision: do not invoke this skill. Instead, create a new draft revision in `wiki/concepts/{slug}-rev{N}.md` with `editorial_status: draft` and a `revision_of: {slug}` field. (A future `/kb-revise` skill will automate this.)
- For multi-reviewer workflows: the `--reviewer` field accepts a comma-separated list (e.g., `--reviewer "alice,bob"`) and stores it verbatim. Audit history beyond the latest review is in the git log.
