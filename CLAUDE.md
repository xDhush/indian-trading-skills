# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup

```bash
pip install -e ".[dev]"          # Install all dependencies including dev tools
pip install feedparser            # Required by india-news-tracker (not in pyproject.toml)
```

## Commands

```bash
# Run all tests
pytest

# Run tests for a specific skill
pytest skills/nse-vcp-screener/scripts/tests/
pytest skills/options-strategy-advisor/scripts/tests/

# Lint
ruff check .
ruff format .

# Standalone CLI scripts
python3 skills/nse-vcp-screener/scripts/screen_vcp.py --universe nifty500
python3 skills/options-strategy-advisor/scripts/black_scholes.py price --spot 24000 --strike 24500 --expiry 7 --vol 0.15 --type call
python3 skills/backtest-expert/scripts/evaluate_backtest.py --trades 150 --win-rate 0.58 --avg-win 2.5 --avg-loss 1.2 --max-drawdown 15 --years 3 --parameters 4 --segment delivery
python3 skills/india-news-tracker/scripts/news_fetcher.py --stock RELIANCE
```

## Architecture

### Skill Structure

Each skill is self-contained under `skills/<skill-name>/`:
- **`SKILL.md`** — The core skill prompt Claude reads; defines the full workflow, decision logic, data sources, and output format. This is what Claude follows when a skill is invoked.
- **`references/`** — Static methodology docs, scoring frameworks, historical pattern databases, and regulatory guides that `SKILL.md` references.
- **`assets/`** — Output templates for reports.
- **`scripts/`** — Standalone Python CLIs (where present). Claude can invoke these directly, parse their JSON/Markdown output, and present results to the user.

### Skills with Python Scripts

Four skills have executable Python modules:

- **`nse-vcp-screener/scripts/`** — Modular pipeline: `screen_vcp.py` (orchestrator) → 5 independent calculators in `calculators/` → `scorer.py` (composite) → `report_generator.py`. Each calculator is a pure `df → score` function.
- **`options-strategy-advisor/scripts/black_scholes.py`** — Full Black-Scholes engine: pricing, Greeks, IV solver, 17 multi-leg strategies, ASCII payoff diagrams. CLI modes: `price`, `greeks`, `iv`, `strategy`.
- **`backtest-expert/scripts/evaluate_backtest.py`** — 5-dimensional scoring (sample size, expectancy, risk, robustness, execution realism), India cost modeling by segment (STT, stamp duty, GST), Deploy/Refine/Abandon verdict.
- **`india-news-tracker/scripts/news_fetcher.py`** — RSS feed parser + sentiment detector. Modes: daily briefing, `--stock`, `--sector`, `--days`, `--min-impact`.

### Data Sources

Skills use free sources only — no paid API keys required:
- **yfinance** — Price/volume data for VCP screener, Black-Scholes spot prices, market breadth
- **niftystocks** — Nifty 50/200/500 constituent lists
- **feedparser + RSS** — Corporate announcements, news headlines (news tracker)
- **Web search** — News context, FII/DII flows, macro events (via Claude)
- **Groww MCP / Zerodha Kite MCP** — Optional; live prices, OI, fundamentals, order placement. Skills document tool equivalence tables so either MCP can be swapped in.

### India Market Constants

When touching financial calculations:
- Risk-free rate: **7%** (91-day T-bill)
- Trading days/year: **252**
- Options exercise: **European only** (NSE)
- Weekly expiry: **Thursday** (Nifty), **Wednesday** (Bank Nifty)
- Settlement: **T+1** equities

### Adding a New Skill

Follow the existing pattern: create `skills/<name>/SKILL.md` as the primary deliverable. Add `references/` docs for any methodology that would be too long to embed inline. Add Python scripts only if the skill involves computation that benefits from a standalone CLI.
