---
name: zotero-cli
description: Use when the user asks to interact with the local Zotero library from the command line, including searching items, semantic search, full-text search, reading notes or annotations, AI analysis of papers, exporting citations or BibTeX, importing references by DOI/PMID/file, managing tags and collections, finding duplicates, syncing, or adding new references. Use when the Zotero MCP server is unavailable or unreliable.
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
| Full-text search in PDFs | `item search-fulltext "QUERY" --limit 10 --json` |
| Search annotations | `item search-annotations "QUERY" --limit 10 --json` |
| Read annotations/highlights | `item annotations KEY --json` |
| AI analyze paper | `--backend api item analyze KEY --question "..." --model MODEL --json` |
| Get item context | `item context KEY --include-notes --include-bibtex --json` |
| Get item metadata | `item get KEY --json` |
| In-text citation | `item citation KEY --style ieee --json` |
| Bibliography entry | `item bibliography KEY --style ieee --json` |
| Single-item BibTeX | `item export --format bibtex KEY --json` |
| Batch BibTeX to file | `export bib --items KEY1,KEY2 --format bibtex --output refs.bib --json` |
| List collections | `collection list --json` |
| List collection items | `collection items KEY --json` |
| Find PDFs for collection | `--backend api collection find-pdfs KEY --json` |
| List items by tag | `tag items "TAG NAME" --json` |
| Read item notes | `item notes KEY --json` |
| Add tag | `--backend api item tag KEY --add TAG --json` |
| Add note | `--backend api note add KEY --text "..." --json` |
| Import by DOI | `--backend api import doi DOI --collection KEY --json` |
| Import by PMID | `--backend api import pmid PMID --collection KEY --json` |
| Import local PDF/file | `--backend api import file PATH --collection KEY --json` |
| Find duplicate items | `--backend api item duplicates --limit 50 --json` |
| Sync Zotero library | `--backend api sync --json` |
| Check app status / find data dir | `app status --json` |

## Common Errors

| Symptom | Fix |
|---------|-----|
| `Local API is not enabled` | Enable Local API in Zotero settings or run `app enable-local-api --launch`. |
| `Item not found` | Verify the key with `item find "title or KEY" --json`. |
| `Backend sqlite cannot write` | Add `--backend api` and ensure Zotero is running. |
| Empty JSON array on search | Broaden the query or check `collection list` for the right key. |
| `Semantic search index not found` | Run `item build-index --json` first. |
| `OPENAI_API_KEY is not set` | `item analyze` requires an OpenAI API key; use `item context` for model-independent output. |
| `No PDF found` for full-text/annotations | Ensure the item has an attached PDF. |
| Citation returns only `[1]` | Use `item bibliography` for the full reference. |
| Chinese title/author/abstract shows `?` / U+FFFD on Windows | CLI Bridge encoding bug. Recover title from `citationKey`, verify authors from `storage/<attachmentKey>/filename.pdf`, or use the Zotero GUI. |

## Chinese (CJK) Metadata on Windows

On Windows, the CLI Bridge plugin can corrupt non-ASCII characters returned by `zotero-cli`. This affects **reading** Chinese/Japanese/Korean metadata, not only writing.

### Affected fields

- `title`
- `abstractNote`
- `creators` names
- `tags`
- PDF filenames / `attachmentPath`

### Root cause

The data in Zotero's SQLite database is valid UTF-8. The corruption happens during transport through the CLI Bridge plugin to the terminal, where CJK characters are replaced by `?` or U+FFFD.

### Workarounds

1. **Use `citationKey` to recover the title.** It is ASCII pinyin/romaji and is not corrupted.
   - Example: `TangMingShengSATDEFMianXiangWeiXingHuLianWangAnQuanDeWeiXieFenXiYuDongTaiYanHuaKuangJia2026` → *SAT-DEF：面向卫星互联网安全的威胁分析与动态演化框架*.
2. **Check the real PDF filename in Zotero storage.** The filesystem name retains correct CJK text.
   - Location: `<data_dir>/storage/<attachmentKey>/<filename>.pdf`.
   - Find `<data_dir>` with `zotero-cli --json app status`.
3. **Open the PDF via Zotero URI.**
   - `zotero://open-pdf/library/items/<parentItemKey>?page=<n>`
4. **For write operations** involving CJK text (tags, abstracts), prefer the Zotero GUI over `zotero-cli js` to avoid writing garbled text back into the library.

## Example

```bash
# Search
zotero-cli --json item find "Noise Protocol Framework"

# Search by Zotero key (useful when titles contain CJK characters)
zotero-cli --json item find "7TL6ARFA"

# For CJK items, JSON metadata may be garbled on Windows.
# Recover the title from citationKey (ASCII pinyin/romaji)
# or verify authors/title from the real PDF filename:
#   <data_dir>/storage/<attachmentKey>/<filename>.pdf

# Semantic search (requires index)
zotero-cli --json item semantic-search "post-quantum key exchange for satellites" --top-k 10

# Find items similar to a known item
zotero-cli --json item similar KEY --top-k 10

# Build or rebuild the semantic search index
zotero-cli --json item build-index

# Full-text search inside attached PDFs
zotero-cli --json item search-fulltext "Diffie-Hellman" --limit 10

# Search annotations across the library
zotero-cli --json item search-annotations "key management" --limit 10

# Read highlights/annotations of a specific paper
zotero-cli --json item annotations KEY

# Get complete context for a paper (notes + BibTeX + metadata)
zotero-cli --json item context KEY --include-notes --include-bibtex

# AI analyze a paper (requires OPENAI_API_KEY)
zotero-cli --backend api --json item analyze KEY --question "What are the main contributions?" --model gpt-4o-mini

# Export BibTeX for the first result (replace KEY)
zotero-cli --json item export --format bibtex KEY

# Or batch-export several items to a file
zotero-cli --json export bib --items KEY1,KEY2 --format bibtex --output refs.bib

# Add a tag
zotero-cli --backend api --json item tag KEY --add protocol-survey

# Import a paper by DOI into a collection
zotero-cli --backend api --json import doi "10.1000/example" --collection COLLECTION_KEY --tag unread

# Import a paper by PMID
zotero-cli --backend api --json import pmid 12345678 --collection COLLECTION_KEY

# Import a local PDF
zotero-cli --backend api --json import file "/path/to/paper.pdf" --collection COLLECTION_KEY

# Find duplicate items
zotero-cli --backend api --json item duplicates --limit 50

# Trigger Zotero sync
zotero-cli --backend api --json sync

# Automatically find missing PDFs for a collection
zotero-cli --backend api --json collection find-pdfs COLLECTION_KEY

# Enrich an existing item from its PDF (English abstract/tags)
# Step 1: extract PDF to Markdown with marker_single
# Step 2: parse Abstract/Keywords from the Markdown
# Step 3: write back with zotero-cli js
zotero-cli --backend api --json js "return (async function() {
  var item = Zotero.Items.getByLibraryAndKey(1, 'KEY');
  item.setField('abstractNote', 'Extracted abstract text...');
  item.addTag('keyword-one');
  item.addTag('keyword-two');
  await item.saveTx();
  return { key: item.key, abstractNote: item.getField('abstractNote'), tags: item.getTags().map(t => t.tag) };
})();" --wait 15
```

## Enrich Workflow

To automatically populate `abstractNote` and tags from a PDF:

1. Locate the PDF path via `item context KEY`.
2. Convert PDF to Markdown with `marker_single` (local, free) or `firecrawl parse` (costs credits).
3. Extract the English abstract and keywords from the Markdown.
4. Write back using `zotero-cli --backend api js` with `item.setField('abstractNote', ...)` and `item.addTag(...)`.
5. Verify with `item get KEY`.

Known limitation on Windows: Chinese/Japanese/Korean characters are corrupted when passed through the CLI Bridge plugin in **both directions** (reading and writing). For CJK items, recover metadata from `citationKey` or the PDF filename, and perform CJK write operations (abstracts/tags) in the Zotero GUI rather than through `zotero-cli js`. Use English abstracts/tags when writing via the CLI.

## Red Flags — STOP

- About to run a write command without `--backend api`.
- About to use plain text because "the user didn't ask for JSON" — the skill says always use `--json`.
- About to skip `--json` because a Quick Reference example looks like it doesn't need it — it does.
- Parsing human-readable output without `--json`.
- Guessing subcommand names instead of using the Quick Reference table.
- Using the Zotero MCP server while it is failing.
- Trusting CJK metadata from `zotero-cli` output without verifying against `citationKey`, the PDF filename, or the Zotero GUI.
- Writing non-ASCII characters through `zotero-cli js` on Windows without warning the user about the CLI Bridge encoding bug.
