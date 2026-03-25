# Calibre MCP Server

Calibre MCP server for Claude Code, designed around family reading workflows.

## Background

- Existing third-party [THeK3nger/calibre-mcp](https://github.com/THeK3nger/calibre-mcp) depends on Calibre Content Server or `calibredb` CLI
- This project is built from scratch for tighter workflow integration

## Technical Decisions

- **Python + FastMCP** — MCP ecosystem standard
- **Local mode only** — reads directly from Calibre Library directory, no Content Server needed
- **`calibredb` CLI** — stable, full-featured, doesn't require Calibre GUI running
- **Kindle push via `calibre-send-email` or direct SMTP** — Calibre built-in support

## Workflow Integration

### 1. Reading Quiz Generation (`/reading-quiz`)
`search_books("title:pirates past noon")` → `get_epub_path(book_id)` → unzip EPUB → read chapters

### 2. Kindle Push
After completing a book quiz, push the next book to Kindle:
`send_to_kindle(book_id, device="child1")`

### 3. Reading Progress Tracking
Track per-child reading status in Calibre custom columns:
`set_custom_column(book_id, "reading_status", "child1:completed")`

## Project Plan

### Phase 1: Basics (DONE)

**Goal: Replace manual Glob + unzip workflow**

- [x] Project scaffold (pyproject.toml, uv, FastMCP)
- [x] `search_books(query)` — Calibre search syntax, returns book list
- [x] `get_book_info(book_id)` — metadata (title, author, series, formats, file paths)
- [x] `get_epub_path(book_id)` — returns EPUB absolute file path
- [x] `list_series(series_name)` — list all books in a series, sorted by series_index
- [x] Configure as Claude Code MCP server
- [x] End-to-end validation

### Phase 2: Kindle Push

**Goal: One-click push books to Kindle**

- [ ] Calibre email config (SMTP, Kindle email address)
- [ ] `send_to_kindle(book_id, device="child1")` — convert + send
- [ ] Device management (multiple Kindles)
- [ ] Auto format conversion (EPUB → Kindle-compatible)

### Phase 3: Reading Management

**Goal: Calibre as central reading tracker**

- [ ] Custom column: `#reading_status` (per-child reading status)
- [ ] Custom column: `#quiz_score` (quiz scores)
- [ ] `mark_as_read(book_id, child, score)` — mark read + score
- [ ] `get_next_book(child, series)` — recommend next book based on progress

### Phase 4: Library Management (Nice to have)

- [ ] `add_book(file_path)` — import new books
- [ ] `edit_metadata(book_id, fields)` — edit metadata
- [ ] `get_custom_columns()` — list custom columns
- [ ] Batch tag management

## Installation

### Prerequisites

- [Calibre](https://calibre-ebook.com/) installed with `calibredb` in PATH
- [uv](https://docs.astral.sh/uv/) or [uvx](https://docs.astral.sh/uv/guides/tools/)

### Claude Code

Add to `~/.claude.json` under `mcpServers`:

```json
{
  "calibre": {
    "command": "uvx",
    "args": ["--from", "git+https://github.com/xmkevinchen/calibre-mcp.git", "calibre-mcp"],
    "env": {
      "CALIBRE_LIBRARY_PATH": "/path/to/Calibre Library"
    }
  }
}
```

Or for local development:

```json
{
  "calibre": {
    "command": "uvx",
    "args": ["--from", "/path/to/calibre-mcp", "calibre-mcp"],
    "env": {
      "CALIBRE_LIBRARY_PATH": "/path/to/Calibre Library"
    }
  }
}
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CALIBRE_LIBRARY_PATH` | Path to Calibre library directory | `~/Calibre Library` |
| `CALIBREDB_PATH` | Path to `calibredb` binary (if not in PATH) | auto-detected |

## References

- `calibredb` docs: `calibredb --help`, subcommands `calibredb list --help` etc.
- [FastMCP](https://github.com/jlowin/fastmcp): `pip install mcp[cli]`, use `@mcp.tool()` decorator
