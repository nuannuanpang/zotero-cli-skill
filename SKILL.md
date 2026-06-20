---
name: zotero-cli
description: Use when the user asks to interact with the local Zotero library from the command line, including searching items, semantic search, exporting citations or BibTeX, managing tags and collections, reading notes or annotations, or adding new references. Use when the Zotero MCP server is unavailable or unreliable.
---

# zotero-cli

Wrap the local `zotero-cli` tool to search, export, tag, and read notes. Use `--json` for JSON.

## When to Use

- User asks about Zotero, citations, references, bibliography, or BibTeX.
- User wants to search their own Zotero library.
- User asks to add a tag, read notes, or export.
- The Zotero MCP server is failing.

Do not use for web search.

## Prerequisites

- `zotero-cli` is installed and available on your PATH (e.g., `C:\Python314\Scripts` is in your `PATH`).
- Read-only commands use `--backend auto` (SQLite).
- Write commands require `--backend api` and Zotero running with Local API enabled.

## Global Options

Always include `--json` unless the user asks for plain text. Place it right after `zotero-cli`.

```bash
zotero-cli --json item find "Noise Protocol"
```

Use `--backend api` for any command that modifies the library.

```bash
zotero-cli --backend api --json item tag KEY --add survey
```

## Quick Reference

| Task | Command |
|------|---------|
| Search items | `item find "QUERY" --json` |
| Semantic search | `item semantic-search "QUERY" --top-k 10 --json` |
| Find similar items | `item similar KEY --top-k 10 --json` |
| Build semantic index | `item build-index --json` |
| Get item metadata | `item get KEY --json` |
| In-text citation | `item citation KEY --style ieee --json` |
| Bibliography entry | `item bibliography KEY --style ieee --json` |
| Single-item BibTeX | `item export --format bibtex KEY --json` |
| Batch BibTeX to file | `export bib --items KEY1,KEY2 --format bibtex --output refs.bib --json` |
| List collections | `collection list --json` |
| List collection items | `collection items KEY --json` |
| List items by tag | `tag items "TAG NAME" --json` |
| Read item notes | `item notes KEY --json` |
| Read item annotations | `item annotations KEY --json` |
| Add tag | `--backend api item tag KEY --add TAG --json` |
| Add note | `--backend api note add KEY --text "..." --json` |
| Import DOI | `--backend api import doi DOI --collection KEY --json` |

## Common Errors

| Symptom | Fix |
|---------|-----|
| `Local API is not enabled` | Enable Local API in Zotero settings or run `app enable-local-api --launch`. |
| `Item not found` | Verify the key with `item find "title" --json`. |
| `Backend sqlite cannot write` | Add `--backend api` and ensure Zotero is running. |
| Empty JSON array on search | Broaden the query or check `collection list` for the right key. |
| `Semantic search index not found` | Run `item build-index --json` first. |
| Citation returns only `[1]` | Use `item bibliography` for the full reference. |

## Example

```bash
# Search
zotero-cli --json item find "Noise Protocol Framework"

# Semantic search (requires index)
zotero-cli --json item semantic-search "post-quantum key exchange for satellites" --top-k 10

# Find items similar to a known item
zotero-cli --json item similar KEY --top-k 10

# Build or rebuild the semantic search index
zotero-cli --json item build-index

# Export BibTeX for the first result (replace KEY)
zotero-cli --json item export --format bibtex KEY

# Or batch-export several items to a file
zotero-cli --json export bib --items KEY1,KEY2 --format bibtex --output refs.bib

# Add a tag
zotero-cli --backend api --json item tag KEY --add protocol-survey
```

## Red Flags — STOP

- About to run a write command without `--backend api`.
- About to use plain text because "the user didn't ask for JSON" — the skill says always use `--json`.
- About to skip `--json` because a Quick Reference example looks like it doesn't need it — it does.
- Parsing human-readable output without `--json`.
- Guessing subcommand names instead of using the Quick Reference table.
- Using the Zotero MCP server while it is failing.
