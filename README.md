# zotero-cli Claude Code Skill

A personal Claude Code skill that wraps the local [`zotero-cli`](https://github.com/jcierocki/zotero-cli) tool, replacing the unreliable Zotero MCP server with deterministic, JSON-driven commands for searching, exporting, tagging, and reading notes from your local Zotero library.

## What it does

- Search your Zotero library from the command line.
- Export citations, bibliographies, and BibTeX entries.
- Manage tags, collections, and notes.
- Read item notes and annotations.
- Enforce `--json` output and correct backend selection (`auto` for reads, `api` for writes).

## Requirements

- [zotero-cli](https://github.com/jcierocki/zotero-cli) installed at `C:\Python314\Scripts\zotero-cli`.
- Zotero running with the **Local API** enabled for any write operation (tag, note, import).
- Claude Code personal skills directory at `C:\Users\mxn28\.claude\skills\`.

## Installation

Clone this repo into your Claude Code personal skills directory:

```powershell
# PowerShell
New-Item -ItemType Directory -Force -Path "C:\Users\$env:USERNAME\.claude\skills\zotero-cli"
git clone https://github.com/nuannuanpang/zotero-cli-skill.git "C:\Users\$env:USERNAME\.claude\skills\zotero-cli"
```

Restart Claude Code so the skill is discovered.

## Quick examples

```bash
# Search
zotero-cli --json item find "Noise Protocol"

# Export BibTeX for one item
zotero-cli --json item export --format bibtex ITEM_KEY

# Batch export to a file
zotero-cli --json export bib --items KEY1,KEY2 --format bibtex --output refs.bib

# Add a tag (write operation: use --backend api)
zotero-cli --backend api --json item tag ITEM_KEY --add survey

# List items with a specific tag
zotero-cli --json tag items "Post-Quantum Cryptography"
```

See `SKILL.md` for the full command reference and `IMPLEMENTATION-PLAN.md` for the RED/GREEN/REFACTOR development log.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `Local API is not enabled` | Enable Local API in Zotero settings or run `app enable-local-api --launch`. |
| `Backend sqlite cannot write` | Add `--backend api` and ensure Zotero is running. |
| Empty JSON search result | Broaden the query or check `collection list` for the right key. |
| Citation returns only `[1]` | Use `item bibliography` for the full reference. |

## License

MIT License — see [LICENSE](./LICENSE).
