---
name: india-stock-analysis-interactive
description: Interactive version of india-stock-analysis. Gathers data, presents web sources for user approval before deep analysis. Used by the /stock-report command via Source Scout and Stock Analyst agents.
---

# India Stock Analysis — Interactive (Source-Vetted)

Analyze Indian stocks listed on NSE and BSE with an interactive source approval step. All analysis is denominated in INR and follows Indian fiscal year conventions (April–March). No API keys required.

This skill is split into three phases. Phases 1–2 are run by the **Source Scout** agent. Phase 3 is run by the **Stock Analyst** agent.

---

## Data Sources

Use whichever broker MCP is connected. Both provide equivalent data for stock analysis.

### Option A: Groww MCP (if connected)
- `fetch_stocks_fundamental_data` — Financials, ratios, shareholding, mutual fund holdings
- `fetch_historical_candle_data` — OHLCV price history
- `get_historical_technical_indicators` — RSI, MACD, Bollinger, SMA, EMA, SuperTrend, VWAP, ADX, and more
- `get_ltp` — Live/last traded price and open interest
- `get_quotes_and_depth` — Real-time bid/ask and market depth
- `curate_symbols` — Resolve stock symbols and exchange
- `fetch_market_movers_and_trending_stocks_funds` — Market movers, gainers, losers
- `fetch_fundamentals_screener` — Screen stocks by fundamental criteria
- `fetch_technical_screener` — Screen stocks by technical signals
- `search_stock_and_others_symbol` — Search for stocks, indices, and companies
- `resolve_market_time_and_calendar` — Current market time, trading days, holidays

### Option B: Zerodha Kite MCP (if connected)
- `get_ltp` — Last traded price for instruments
- `get_quotes` — Real-time market quotes with depth
- `get_ohlc` — OHLC data for instruments
- `get_historical_data` — Historical OHLCV candle data
- `search_instruments` — Search and resolve trading instruments
- `get_holdings` — User's portfolio holdings
- `get_positions` — Current trading positions
- `get_margins` — Account margin details
- `get_profile` — User profile information

### Supplementary
- Web search for news, analyst reports, sector developments, and regulatory updates
- yfinance (free, no API key) as fallback for historical data

---

## Phase 1: Discovery (Automated)

**Run by: Source Scout agent**

This phase gathers all raw data and finds web sources. No user interaction needed.

### Steps:

1. **Resolve the symbol.** Call `curate_symbols` or `search_stock_and_others_symbol` with the company name to obtain the correct trading symbol and exchange (NSE/BSE). If no MCP is available, use yfinance with the `.NS` suffix.

2. **Fetch market data.** Gather the following in parallel where possible:

   a. **Current price and key stats:** Call `get_ltp` and `fetch_stocks_fundamental_data` (or yfinance `Ticker.info`) for: `marketCap`, `peRatio`, `pbRatio`, `roe`, `epsTtm`, `dividendYieldInPercent`, `industryPe`, `bookValue`, `debtToEquity`, `faceValue`, `returnOnAssets`, `returnOnEquity`, `operatingProfitMargin`, `netProfitMargin`, `currentRatio`, `beta`.

   b. **Financial statements:** Fetch 3–5 years of income statement, balance sheet, and cash flow data via `fetch_stocks_fundamental_data` with `view='all'` or yfinance `.financials`, `.balance_sheet`, `.cashflow`.

   c. **Price history:** Fetch daily candles for the last 1 year and weekly candles for the last 2 years. Calculate: 52-week high/low, YTD performance, SMAs (20, 50, 200), RSI (14), MACD (12, 26, 9), Bollinger Bands (20, 2), ATR (14).

   d. **Shareholding pattern:** Call `fetch_stocks_fundamental_data` with `view='shareholders_and_mutual_funds'` or fetch from Trendlyne/BSE.

3. **Run MoneyControl searches (mandatory).** These MUST be run as separate, dedicated searches — do not combine with other queries:

   a. `[Company Name] moneycontrol latest news [current year]` — recent news articles, corporate updates, and market commentary on MoneyControl
   b. `[Company Name] moneycontrol quarterly results analysis [current year]` — earnings analysis, financial performance coverage

   MoneyControl is a primary trusted source for Indian stock coverage. If either search returns MoneyControl URLs, include them all in the source list. If a search returns zero MoneyControl results, note this when presenting sources ("MoneyControl: no results found for [query]").

4. **Run general web searches.** Execute 4–6 additional targeted searches for editorial and opinion sources:
   - Latest quarterly results and financial performance
   - Analyst price targets and recommendations
   - Recent company news (last 30 days)
   - Shareholding pattern and institutional activity
   - Sector outlook and macro developments
   - Management commentary / concall highlights (if available)

5. **Compile all source URLs** found from the web searches (steps 3 and 4). Record each source with:
   - Title
   - URL
   - Which search query found it (one line)

6. **Compile data sources** used (yfinance, MCP tools, Trendlyne, etc.).

---

## Phase 2: Source Approval (Interactive)

**Run by: Source Scout agent**

Present all gathered sources to the user for approval.

### Presentation Format:

```
═══ DATA SOURCES (used automatically, for your reference) ═══
  • yfinance — [SYMBOL].NS (price, financials, balance sheet, technicals)
  • Trendlyne — shareholding pattern
  • [Groww MCP / Zerodha MCP — if connected, list tools used]

═══ WEB SOURCES (need your approval) ═══
  1. [Source Name] Title of article or page
     └─ Why: [which search query found this, one line]
     └─ URL: https://...
  2. [Source Name] Title of article or page
     └─ Why: [one line]
     └─ URL: https://...
  ...
  N. [Source Name] Title of article or page
     └─ Why: [one line]
     └─ URL: https://...

Which sources do you approve? (e.g., "1-8", "all", "all except 9,10")
```

### After User Approves:

Ask the user:

```
Search for more sources, or proceed with analysis?
```

**If "more" or user specifies a topic:**
- If the user mentioned a specific topic, search for that.
- Otherwise, auto-detect gaps in coverage. Examples of gaps:
  - No management commentary / concall transcript found → search for it
  - No peer comparison data found → search for peer analysis
  - No sector outlook found → search for sector report
  - No recent regulatory news → search for regulatory updates
- Present new sources in the same numbered format (continuing from previous numbering).
- Repeat the approval cycle.

**If "proceed":**
- Move to Phase 3.

### Save Sources File:

Before moving to Phase 3, save the sources file to the path provided by the slash command (e.g., `stock-reports/hdfc-bank/2026-03-29/hdfc-bank-2026-03-29.sources.md`).

**Sources file format:**

```markdown
# Sources — [Company Name] ([Date])

## Data Sources (used automatically)
- yfinance: [SYMBOL].NS — price, financials, technicals, balance sheet
- Trendlyne: shareholding pattern page
- [Other MCP tools if used]

## Approved Web Sources
1. [Source Name] Title
   - URL: https://...
   - Found via: [search query description]
2. ...

## Rejected Sources
N. [Source Name] Title
   - URL: https://...
   - Found via: [search query description]
   - Rejected by user
```

---

## Phase 3: Deep Analysis (Automated)

**Run by: Stock Analyst agent**

Using only the approved sources and the data gathered in Phase 1, perform a comprehensive analysis.

### Steps:

1. **Read the sources file** to know which URLs are approved.

2. **Fetch content from approved web sources.** Use `WebFetch` to extract relevant information from each approved URL.

3. **Perform fundamental analysis** using the framework in `references/fundamental-analysis.md`:

   a. **Business Quality Assessment**
   - What does the company do? Competitive moat?
   - Management quality and promoter track record
   - Market position and competitive advantages
   - Corporate governance indicators

   b. **Financial Health**
   - Revenue and profit trends (3–5 year view)
   - Margin analysis (operating, net, EBITDA)
   - Cash flow quality (OCF vs reported profit)
   - Balance sheet strength (debt levels, current ratio, interest coverage)

   c. **Shareholding Pattern (India-Specific)**
   - Promoter holding percentage and trend
   - Promoter pledge percentage
   - FII holding trend
   - DII holding trend
   - Quarter-over-quarter changes

   d. **Valuation**
   - PE vs Industry PE and Sector PE (premium/discount)
   - PB vs Sector PB
   - PEG ratio assessment
   - EV/EBITDA comparison
   - Earnings yield vs risk-free rate (7%)

   e. **Growth Assessment**
   - Revenue growth trajectory
   - EPS growth trend
   - Capex plans and return on invested capital

   f. **Risk Factors**
   - Company-specific risks
   - Sector/regulatory risks
   - Promoter-related risks
   - Macro risks

4. **Perform technical analysis:**

   a. **Trend Analysis**
   - Primary trend (weekly), secondary trend (daily)
   - Price position relative to key SMAs (20, 50, 200 DMA)
   - Golden cross / death cross status

   b. **Support and Resistance Levels**
   - From pivot points, swing lows/highs, round numbers

   c. **Momentum Indicators**
   - RSI, MACD, Stochastic, ADX

   d. **Volatility Assessment**
   - Bollinger Band width and position, ATR trend

   e. **Volume Analysis**
   - Volume trend, OBV, VWAP position

   f. **Technical Outlook**
   - Short-term (1–2 weeks) and medium-term (1–3 months)

5. **Form investment thesis:**
   - Bull case (3–5 reasons to buy)
   - Bear case (3–5 reasons for caution)
   - Base case scenario

6. **Assign a fundamental score** from 1–10 based on the scorecard in `references/fundamental-analysis.md`.

7. **Compile peer comparison** using data for 3–5 comparable companies.

8. **Identify catalysts** — near-term, medium-term, long-term.

9. **Save the raw analysis** to the path provided by the slash command (e.g., `stock-reports/hdfc-bank/2026-03-29/hdfc-bank-2026-03-29.raw.md`).

### Raw Analysis Output Format:

The raw analysis file should contain all findings organized under these sections:

```markdown
# Raw Analysis — [Company Name] ([Date])

## Company Info
[Symbol, exchange, sector, market cap category, business description]

## Current Price & Key Stats
[CMP, change, 52W high/low, market cap, key ratios]

## Financial Statements
[3-5 year income statement, balance sheet, cash flow highlights]

## Fundamental Analysis
[Business quality, financial health, shareholding, valuation, growth, risks — all subsections]

## Technical Analysis
[Trend, support/resistance, momentum, volatility, volume, outlook — all subsections]

## Investment Thesis
[Bull case, bear case, base case]

## Fundamental Score
[Score /10 with category breakdown]

## Peer Comparison
[Table of peers with key metrics]

## Catalysts
[Near-term, medium-term, long-term]

## Risk Assessment
[Risk matrix]

## Sources Used
[Reference to the .sources.md file]
```

---

## India-Specific Considerations

Throughout all phases, apply these India-specific guidelines:

- **Currency**: All prices and figures in INR (Rs.). Use Cr (Crore = 10 million) and L (Lakh = 100,000).
- **Fiscal Year**: April–March. Reference as FY24 (April 2023 – March 2024), FY25, etc.
- **Market Hours**: NSE/BSE trade 9:15 AM to 3:30 PM IST, Monday to Friday.
- **Promoter Holding**: >70% very high control; 50–70% strong; 30–50% moderate; <30% caution.
- **Promoter Pledge**: 0% ideal; 1–10% low risk; 10–20% moderate; >20% red flag; >50% serious concern.
- **FII/DII Holdings**: Indicate institutional confidence. Track trends.
- **Regulatory Context**: SEBI regulations, LODR compliance.
- **Index Membership**: Note Nifty 50, Bank Nifty, sectoral indices, etc.
- **Dual Listing**: Most companies on both NSE and BSE. Use NSE data by default.
- **T+1 Settlement**: Indian markets follow T+1 settlement.
- **Lot Size (F&O)**: Mention if the stock is in the F&O segment.

## Sector-Specific Metrics

For banking stocks, include: NIM, Gross NPA, Net NPA, CASA Ratio, CAR, Provision Coverage Ratio, Credit Cost, Cost-to-Income ratio.

For IT stocks, include: Revenue per employee, utilization rate, attrition, deal TCV, offshore-onsite mix.

For pharma stocks, include: R&D spend %, ANDA pipeline, US FDA observations, domestic vs export split.

For auto stocks, include: Monthly dispatches, realization per vehicle, capacity utilization, order book.

For FMCG stocks, include: Volume growth, rural vs urban split, distribution reach.

Refer to `references/fundamental-analysis.md` Section 5 for full sector-specific metric tables.

## Error Handling

- If a broker MCP tool call fails, note the missing data and proceed. Use yfinance or web search as fallback.
- If the stock symbol cannot be resolved, ask the user to clarify.
- If the company is not listed on Indian exchanges, inform the user.
- If historical data is limited (recent IPO), adjust timeframes and note the limitation.
