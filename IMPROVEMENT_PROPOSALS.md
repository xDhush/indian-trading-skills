# Improvement Proposals

Ideas for future enhancements to the stock analysis pipeline. These are not actively being worked on.

---

## 1. Dedicated MoneyControl MCP Server

**Problem:** MoneyControl has rich, structured financial data (P&L, balance sheet, cash flow, peer comparison, mutual fund holdings, analyst estimates) but no official API. Currently we access it via WebFetch on search results, which is fragile and unstructured.

**Proposal:** Build a lightweight MCP server that wraps MoneyControl scraping to provide structured tools like:
- `get_financials(symbol)` — 5-year P&L, balance sheet, cash flow
- `get_ratios(symbol)` — Key financial ratios with sector comparison
- `get_peer_comparison(symbol)` — Peer table with PE, PB, ROE, market cap
- `get_shareholding(symbol)` — Quarterly shareholding pattern
- `get_analyst_estimates(symbol)` — Consensus targets and recommendations

**Existing scrapers to evaluate:**
- [Web-Scrapers/Money-Control](https://github.com/Web-Scrapers/Money-Control) (39 stars) — Scrapes financials by sector/industry
- [Mfinance](https://github.com/shashank-556/Mfinance) — Python module for MoneyControl financial data
- [Financials-analysis-for-Financial-modelling](https://github.com/bsrikrishna/Financials-analysis-for-Financial-modelling) (31 stars) — 15 years of P&L and balance sheet export

**Effort:** Medium. Main risk is scraper fragility if MoneyControl changes their HTML structure.

---

## 2. Indian Stock Market MCP (yfinance-based)

**Problem:** The current pipeline uses yfinance as a fallback via Python scripts or inline code. An MCP would give all agents structured tool access to market data without needing code execution.

**Proposal:** Adopt or fork an existing Indian market MCP:
- [kai-stock-market-mcp](https://github.com/neerajadhav/kai-stock-market-mcp) (4 stars) — 9 tools: live quotes, historical data, fundamentals, financials (4yr), stock comparison, dividends, shareholders, indices, chart generation
- [nse-bse-mcp](https://github.com/vanshikaaa01/nse-bse-mcp) (2 stars) — Similar 9-tool coverage

Both are yfinance-based (no API key required) and would slot into the existing Groww/Zerodha MCP pattern in SKILL.md.

**Effort:** Low. Install and configure, then add as "Option C" in SKILL.md data sources.

---

## 3. IndiaQuant MCP for Advanced Analysis

**Problem:** The Stock Analyst agent currently computes technicals from raw price data. A specialized MCP could provide pre-computed signals, options analysis, and sentiment.

**Candidate:** [IndiaQuant-MCP-Server](https://github.com/deepak-05dktopG/IndiaQuant-MCP-Server) (1 star) — 10 tools including:
- Technical signals (RSI, MACD, Bollinger, etc.)
- Options chain analysis with Greeks
- News sentiment scoring
- Unusual activity detection
- Sector-level market scans

**Dependencies:** Requires NewsAPI and Alpha Vantage API keys (free tiers available).

**Effort:** Low-Medium. Install, configure API keys, evaluate data quality vs current approach.

---

## 4. MoneyControl News Integration

**Problem:** The Source Scout's web searches may miss MoneyControl's news and editorial content, which often has early coverage of quarterly results and management commentary.

**Proposal:** Use the `moneycontrol-api` PyPI package as a supplementary news source in the Source Scout phase. It provides `get_latest_news` and `get_business_news` endpoints.

**Package:** [moneycontrol-api](https://pypi.org/project/moneycontrol-api/) (v1.1.4)

**Effort:** Low. `pip install moneycontrol-api`, add a small script to fetch and format news URLs for the Source Scout.
