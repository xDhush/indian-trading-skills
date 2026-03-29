# /stock-report — Generate a Stock Investment Report

Generate a comprehensive, simplified investment report for an Indian stock using three agents in sequence.

**Stock:** $ARGUMENTS

---

## Setup

1. Derive the **stock slug** from the stock name: lowercase, hyphens for spaces (e.g., "HDFC Bank" → "hdfc-bank").
2. Get **today's date** in `YYYY-MM-DD` format.
3. Set the **output directory**: `stock-reports/<stock-slug>/<date>/`
4. Set the **file prefix**: `<stock-slug>-<date>`

## Directory Preparation

1. Check if `stock-reports/<stock-slug>/` exists. If not, create it.
2. Check if `stock-reports/<stock-slug>/<date>/` exists. If not, create it.
3. If the date folder already exists and has files (`.sources.md`, `.raw.md`, `.md`), ask: **"Found existing files for <stock-slug>/<date>. Overwrite?"**
   - Yes → proceed.
   - No → stop.

## Agent Pipeline

Run these agents **sequentially** — each depends on the previous agent's output.

### Agent 1: Source Scout

Launch **source-scout** (`.claude/agents/source-scout.md`).

**Parameters:**
- Stock to analyze: `$ARGUMENTS`
- Sources file path: `<output-dir>/<file-prefix>.sources.md`

Wait for completion before proceeding.

### Agent 2: Stock Analyst

Launch **stock-analyst** (`.claude/agents/stock-analyst.md`).

**Parameters:**
- Stock to analyze: `$ARGUMENTS`
- Sources file: `<output-dir>/<file-prefix>.sources.md`
- Raw analysis output path: `<output-dir>/<file-prefix>.raw.md`
- Market data from Source Scout: pass along data returned by Agent 1

Wait for completion before proceeding.

### Agent 3: Report Writer

Launch **report-writer** (`.claude/agents/report-writer.md`).

**Parameters:**
- Stock: `$ARGUMENTS`
- Raw analysis file: `<output-dir>/<file-prefix>.raw.md`
- Sources file: `<output-dir>/<file-prefix>.sources.md`
- Final report output path: `<output-dir>/<file-prefix>.md`

Wait for completion.

## Completion

```
Report complete for [Stock Name]:
  Report:  stock-reports/<stock-slug>/<date>/<file-prefix>.md
  Raw:     stock-reports/<stock-slug>/<date>/<file-prefix>.raw.md
  Sources: stock-reports/<stock-slug>/<date>/<file-prefix>.sources.md
```

## Error Handling

Each agent saves its output file before the next agent starts, so completed work is never lost.

- **Agent 1 fails** → stop. No files created.
- **Agent 2 fails** → stop. `.sources.md` is already saved.
- **Agent 3 fails** → stop. `.sources.md` and `.raw.md` are already saved.
