# Indian Trading Skills — Comprehensive Usage Guide

> **Disclaimer**: This guide is for educational purposes only. Trading in securities involves substantial risk of loss. Nothing here constitutes financial advice. Read SEBI's investor charter before trading.

---

## Table of Contents

1. [How to Load a Skill](#how-to-load-a-skill)
2. [Skill 1: Technical Analyst](#1-technical-analyst)
3. [Skill 2: NSE VCP Screener](#2-nse-vcp-screener)
4. [Skill 3: India Stock Analysis](#3-india-stock-analysis)
5. [Skill 4: Scenario Analyzer](#4-scenario-analyzer)
6. [Skill 5: FII/DII Flow Tracker](#5-fiidii-flow-tracker)
7. [Skill 6: Options Strategy Advisor](#6-options-strategy-advisor)
8. [Skill 7: Backtest Expert](#7-backtest-expert)
9. [Skill 8: India Market Breadth](#8-india-market-breadth)
10. [Skill 9: India News Tracker](#9-india-news-tracker)
11. [Skill 10: Weekly F&O Trade Planner](#10-weekly-fo-trade-planner)
12. [Using Skills Together: Power Workflows](#using-skills-together-power-workflows)
13. [Common Mistakes Across All Skills](#common-mistakes-across-all-skills)
14. [Setup Checklist](#setup-checklist)

---

## How to Load a Skill

Each skill lives in `skills/<name>/SKILL.md`. To use a skill, paste its contents into your Claude conversation or reference it:

```
# Option A — Direct paste
Copy the contents of skills/technical-analyst/SKILL.md and paste into Claude.

# Option B — Claude Code (this repo)
Open Claude Code in this repo directory, then say:
"Load the technical analyst skill and analyze [chart/stock/ticker]"

# Option C — Claude Projects
Upload SKILL.md as a project document in Claude.ai/projects for persistent access.
```

**MCP Integration (optional but powerful)**:
- Zerodha Kite MCP or Groww MCP unlocks live prices, OI data, and order placement
- Without MCP: skills fall back to yfinance + web search (still functional)
- With MCP: real-time data, live order placement, options chain data

---

## 1. Technical Analyst

**File**: `skills/technical-analyst/SKILL.md`
**Category**: Chart Analysis | **Input**: Price chart images
**Best for**: Swing trading setups, identifying entry/exit zones, sector rotation

### What It Does

Performs systematic weekly chart analysis. You feed it chart images; it returns a structured report covering trend direction, support/resistance levels, moving average relationships, volume patterns, and 2–4 probability-weighted scenarios with specific price targets and invalidation levels.

### How to Use It

```
1. Load the skill (paste SKILL.md into Claude or use via Claude Projects)
2. Upload 1–5 weekly chart images (from TradingView, Zerodha, etc.)
3. Ask: "Analyze this chart" or "What is the weekly outlook for [stock]?"
4. Claude reads the technical_analysis_framework.md reference automatically
5. Receive: Trend analysis → S/R levels → MA relationships → Volume → 2-4 scenarios
```

**Best chart format**: Weekly candlestick with 10/20/50 EMA and volume bars visible. Export at minimum 800×600 resolution from TradingView.

**Example prompts**:
- *"Analyze the weekly charts I've uploaded. Give me the highest-probability scenario and the invalidation level."*
- *"Compare these 3 charts — which one has the strongest technical setup for a breakout?"*
- *"I'm in HDFC Bank at ₹1,720. Based on the chart, where should my stop-loss be?"*

### Pros

- **Probabilistic, not predictive**: Outputs 2–4 scenarios with % probabilities rather than a single prediction — more honest and useful
- **Systematic**: Follows a fixed framework, eliminating emotional bias from chart reading
- **Multi-timeframe awareness**: Identifies confluence across weekly/daily levels
- **Handles Indian market nuances**: Recognizes NSE-specific patterns like post-budget gaps, election result candles, quarterly result spikes
- **Fast throughput**: Can analyze 5–10 charts in one session

### Cons and Limitations

- **Requires chart images** — cannot fetch or render charts itself without MCP; you must supply them
- **Weekly charts only** — the framework is calibrated for weekly timeframes; applying it to intraday charts will produce lower-quality output
- **Lagging by nature** — all technical analysis is backward-looking; patterns identified on weekly charts may take weeks to play out
- **False breakouts are common in Indian mid-caps** — circuit breakers, operator activity, and low float stocks create patterns that look valid but fail; the skill cannot distinguish manipulated moves
- **Volume data quality**: yfinance NSE volume data sometimes has errors for illiquid stocks; always cross-check manually

### How to Extract Maximum Benefit

1. **Batch analyze on weekends**: Load 10–20 weekly charts every Sunday. Ask Claude to rank them by setup quality. This creates a watchlist before Monday opens.
2. **Use scenarios, not just the primary call**: The "Bull" and "Bear" scenarios have explicit invalidation levels — use these to set your GTT stop-losses before entering.
3. **Stack with VCP Screener**: Use the screener to filter technically strong stocks, then feed their charts to this skill for detailed entry planning.
4. **Ask for confluence zones**: "Where do the support level and the 50-week EMA converge?" — these tend to be the highest-quality zones.
5. **Track your scenario accuracy**: Write down which scenario played out every week. After 20 charts, you'll know how well-calibrated the probability estimates are for your chosen stocks.

### What to Completely Avoid

- **Do not treat probability estimates as exact**: "65% chance of breakout" is a calibrated guess, not a statistical certainty. These are relative rankings, not independent frequencies.
- **Never use this skill for intraday trading decisions** — weekly analysis is irrelevant on 5-minute timeframes.
- **Don't analyze charts of stocks in F&O ban** — the restricted open interest changes price dynamics; standard TA breaks down.
- **Avoid low-quality chart images** — blurry, zoomed-out, or clipped charts will produce vague analysis. Always ensure the last 52 weeks are visible.
- **Don't confuse the model by providing fundamentally distorted price histories** — stocks that have undergone recent bonus issues, splits, or restructuring have chart distortions that TA misreads. Verify the chart is adjusted for corporate actions.

---

## 2. NSE VCP Screener

**File**: `skills/nse-vcp-screener/SKILL.md`
**Category**: Stock Screening | **Input**: Universe selection (`nifty50`/`nifty200`/`nifty500`)
**Best for**: Position trading, finding Minervini-style momentum breakout candidates

### What It Does

Implements Mark Minervini's Volatility Contraction Pattern (VCP) screening pipeline against NSE-listed stocks. It pulls 260 days of price/volume history via yfinance, applies a 7-point Stage 2 Trend Template filter, then scores each passing stock across 5 dimensions (Trend, Contraction Quality, Volume, Pivot Proximity, Relative Strength). Stocks scoring 80+ are immediate watchlist candidates.

### How to Use It

**Standalone CLI** (fastest):
```bash
# Screen Nifty 500 for VCP setups
python3 skills/nse-vcp-screener/scripts/screen_vcp.py --universe nifty500

# Require at least 3 contractions (tighter filter)
python3 skills/nse-vcp-screener/scripts/screen_vcp.py --universe nifty500 --min-contractions 3

# Save report to a directory
python3 skills/nse-vcp-screener/scripts/screen_vcp.py --universe nifty200 --output-dir reports/

# Run Friday after market close for weekend analysis
```

**Via Claude Code**:
```
"Run the VCP screener on Nifty 500 and show me all stocks scoring above 70"
"Which stocks in Nifty 200 are closest to their pivot breakout point?"
"Compare RELIANCE and INFY VCP scores"
```

**Score interpretation**:
| Score | Action |
|-------|--------|
| 80–100 | Deploy (watchlist, set price alerts) |
| 65–79 | Watchlist only (monitor for improvement) |
| 50–64 | Marginal (skip unless strong conviction) |
| < 50  | Skip |

### Pros

- **Objective, repeatable**: Eliminates chart bias — same stock, same day, same score every time
- **India-specific calibration**: Uses NSE tickers, Nifty 50 as the relative strength benchmark, INR-denominated thresholds
- **Fully free**: No API keys — yfinance + niftystocks; runs offline after initial data fetch
- **Modular architecture**: Each calculator (trend, VCP, volume, pivot, RS) is independent — easy to adjust weights if you want to customize
- **Fast**: Nifty 500 typically screened in under 5 minutes on a standard laptop
- **Pivot proximity score**: Highlights stocks that are literally days away from a potential breakout trigger

### Cons and Limitations

- **Survivorship bias in universe**: Nifty 500 only includes large and mid-cap survivors. Genuine emerging multi-baggers in small-caps are not screened.
- **Data latency**: yfinance data may lag by 1 day for NSE. Do not run intraday; run post-market close only.
- **VCP is a momentum continuation pattern** — it works in bull markets and underperforms significantly in bear/sideways markets. Check the market breadth skill first.
- **False positives in manipulation-prone stocks**: Operator-driven stocks in the mid-cap segment can show perfect VCP-looking patterns that reverse instantly after breakout.
- **No earnings filter by default**: A stock can score 90 on VCP but have an earnings announcement next week — IV crush or bad results kill the trade. You must check the earnings calendar separately.
- **Relative strength uses Nifty 50 as benchmark**: Sector-specific RS (e.g., comparing a PSU bank to the PSU bank index) is not computed.
- **VCP works best for 2–8 week holding periods** — it is not a buy-and-hold or intraday tool.

### How to Extract Maximum Benefit

1. **Run every Friday after market close**: Weekly screening aligns with the weekly chart rhythm. Weekend gives you time to review without market pressure.
2. **Combine with market regime check**: Only trade VCP breakouts when the India Market Breadth skill shows a Healthy (60+) or Strong (80+) health score. In breadth < 40, skip all new entries.
3. **Sort by pivot proximity, not composite score**: A stock at 95% composite score but 30% away from pivot is less actionable than one at 75% score but 2% from pivot.
4. **Set price alerts, don't chase**: VCP breakouts should be bought at the pivot — a very specific price — not chased after 5% moves. Use your broker's price alert system.
5. **Volume confirmation is mandatory**: The breakout candle must have 150%+ above average volume. The skill computes this; use it as a hard filter.
6. **Filter by sector**: If FII/DII flows show institutional selling in IT, even a 90-score IT stock is high risk. Stack with the flow tracker.

### What to Completely Avoid

- **Do not buy stocks scoring below 65 just because they look good visually** — trust the composite score, not gut feel.
- **Never enter a VCP breakout the day before results** — earnings risk can negate a perfect setup in minutes.
- **Don't run the screener on illiquid small-caps** — force-fitting Minervini's US-market methodology on stocks with 10,000 daily volume creates noise, not signals.
- **Avoid buying on a "potential pivot" before price actually breaks** — VCP is a breakout strategy, not an anticipation strategy. Wait for the actual candle.
- **Don't ignore the trend template filter** — if a stock fails Stage 2 criteria (price below 200 DMA, or in Stage 3/4), no VCP score matters; it's a losing trade statistically.

---

## 3. India Stock Analysis

**File**: `skills/india-stock-analysis/SKILL.md`
**Category**: Fundamental + Technical Research | **Input**: Stock ticker (NSE/BSE)
**Best for**: Investment thesis development, due diligence, buy/hold/sell decisions

### What It Does

Comprehensive single-stock research covering 4 modes: basic info, fundamental analysis, technical analysis, or a full investment report (default). The full report includes business quality scoring, 50+ financial ratios, peer comparison, shareholding analysis, valuation matrix, bull/bear case, sector outlook, catalysts, and risk matrix.

### How to Use It

```
# Full comprehensive report (default)
"Analyze TITAN for me" → Full investment report
"Give me a comprehensive analysis of ZOMATO"

# Specific modes
"Give me the fundamental analysis of HDFC BANK"
"What's the technical outlook for BAJAJ FINANCE?"
"Basic stock information for IRCTC"

# With MCP (best quality)
"Analyze RELIANCE using live data from Zerodha"  → fetches live quotes, OI, fundamentals
"Compare INFOSYS vs TCS across all key metrics"
```

**Data source priority** (automatic):
1. Groww MCP → live fundamentals, historical candles
2. Zerodha Kite MCP → live quotes, technicals
3. yfinance → historical price/volume fallback
4. Web search → news, analyst reports, sector context

### Pros

- **50+ financial ratios**: Covers everything from basic P/E to sector-specific metrics (NIM for banks, EBITDA/EV for industrials, ARPU for telecoms)
- **India-specific context built in**: Understands DII, FII, promoter pledging, NCLT, grey market premiums, SEBI regulations
- **Fundamental scorecard (1–10)**: Single aggregated score for quick comparison
- **Sector-appropriate analysis**: Banking analysis uses NIM, GNPA, PCR — not generic ratios. Auto analysis uses EBITDA margins, EV/EBITDA.
- **Bull/bear case format**: Forces explicit documentation of what needs to go right (bull) vs what could go wrong (bear)
- **Peer comparison table**: Puts the target stock in context vs 3–5 closest peers

### Cons and Limitations

- **Data quality depends on source**: Without MCP, yfinance data for Indian stocks can have gaps — especially for quarterly EPS, promoter shareholding changes, and recent corporate actions.
- **Analyst report access is limited**: The skill can web-search for analyst reports but cannot access Bloomberg, Refinitiv, or broker research PDFs.
- **Forward estimates are inferred, not sourced**: Revenue/EPS projections are based on historical trends and sector assumptions, not sell-side consensus — treat as directional only.
- **Small-cap and micro-cap data is sparse**: For stocks outside Nifty 500, yfinance data is unreliable; MCP is essentially required.
- **Not a replacement for reading the annual report**: The skill synthesizes data but cannot substitute for reading MD&A, notes to accounts, auditor qualifications, and related party transactions in the actual AR.
- **Valuation models are inherently assumption-dependent**: DCF outputs from this skill are [Inference] — small changes in WACC or growth rate produce very different fair values.

### How to Extract Maximum Benefit

1. **Always request the full report, then drill down**: Start with the comprehensive report to identify the 2–3 most important angles, then ask follow-up questions on those specifically.
2. **Use the fundamental scorecard for portfolio comparison**: Run 5–10 stocks through the skill, record only the scorecard and valuation metrics, then rank them. Invest in the top 3.
3. **Ask explicitly about red flags**: "What are the top 3 risks I might be underestimating for this stock?" The skill flags things like promoter pledging, auditor changes, contingent liabilities.
4. **Cross-check with shareholding trends**: "How has the FII vs DII vs promoter shareholding changed over the last 4 quarters?" — this is a leading indicator the skill can compute.
5. **Use the peer comparison for sector rotation decisions**: "Compare ICICI Bank, HDFC Bank, Kotak, and Axis Bank on key metrics." This makes relative value clear.
6. **Request a re-run after quarterly results**: Always re-analyze after earnings to see if the thesis is intact.

### What to Completely Avoid

- **Don't make buy/sell decisions based on the report alone** — it's a research tool, not a trading signal. Combine with technicals and market timing.
- **Never skip reading the risk section**: The bull case is emotionally appealing; the risks section is where value is. Read it first.
- **Don't use the DCF fair value as a price target** — DCFs are sensitivity analyses, not predictions. ±20% is within normal error.
- **Avoid over-relying on P/E for growth stocks**: For high-growth companies (Zomato, Nykaa, etc.), P/E is not meaningful; use EV/Revenue or EV/EBITDA.
- **Don't analyze stocks in SEBI investigation or NCLT proceedings without noting the additional risk** — the skill may not always flag ongoing regulatory actions unless explicitly asked.

---

## 4. Scenario Analyzer

**File**: `skills/scenario-analyzer/SKILL.md`
**Category**: Event-Driven Analysis | **Input**: A market-moving headline
**Best for**: Trading around known events (RBI policy, Union Budget, crude moves, elections), portfolio hedging decisions

### What It Does

Takes any market headline and builds three structured scenarios (Base/Bull/Bear) with 1st, 2nd, and 3rd order sector impacts projected over an 18-month horizon. Identifies specific stocks that benefit or get hurt. References a sector sensitivity matrix and historical event playbooks to ground the analysis.

### How to Use It

```
"Analyze the scenario: RBI cuts repo rate by 25 bps"
"Analyze: India imposes 15% import duty on Chinese electronics"
"Build scenarios for: Crude oil hits $100/barrel"
"What are the implications of a BJP-majority Union Budget with capital gains tax hike?"
```

**Output structure**:
- Event classification + historical analogues
- Base scenario (most likely): probability, sector impacts, 3–5 stock ideas
- Bull scenario: upside case, conditions required
- Bear scenario: downside case, conditions required
- 1st/2nd/3rd order impact cascade

### Pros

- **3-order cascade analysis**: Goes beyond the obvious "oil up = oil companies benefit" to identify 2nd order (logistics, inflation, RBI response) and 3rd order (auto sector credit demand, rural consumption) effects
- **Historical calibration**: References actual Indian market reactions to RBI policy changes, Budget outcomes, crude oil moves, elections — not generic frameworks
- **Forces explicit scenario thinking**: Prevents anchor bias by making you consider all three scenarios before forming a view
- **Saves 2–3 hours of research**: For a major event like Union Budget, assembling sector impacts manually would take hours; this skill does it in minutes
- **Probability-weighted**: Assigns probabilities to each scenario so you can size positions accordingly

### Cons and Limitations

- **Historical patterns are approximate guides, not guarantees**: Indian markets have reacted very differently to the same RBI action in different macro environments (e.g., rate cuts during growth phases vs. rate cuts during crises behave oppositely).
- **3rd-order impacts are highly speculative**: The further the causal chain, the less reliable the projection. Treat 3rd-order impacts as hypothesis-generation, not high-conviction calls.
- **Cannot account for simultaneous conflicting events**: Real-world market moves are multi-causal. If a rate cut happens simultaneously with FII outflows, the skill's scenario may not capture the net effect accurately.
- **Political events are the hardest to model**: Election outcomes, policy reversals, and geopolitical surprises have historically produced market reactions that deviate sharply from any model.
- **Scenario probabilities are not statistically derived**: The 60/25/15 Base/Bull/Bear split is an informed estimate, not a model output.

### How to Extract Maximum Benefit

1. **Use before known events, not after**: The skill is most valuable the day before RBI policy, Budget day, or a major global event — when you can position in advance. Using it after an event is academic.
2. **Focus on the bear scenario for hedging decisions**: If your portfolio is long equities, the bear scenario tells you which positions are most exposed and suggests hedges.
3. **Combine with the Options Strategy Advisor**: After getting the scenario analysis, use the options skill to price protective puts or define-risk spreads that profit from the bear scenario.
4. **Track the scenario outcomes**: Did the base case play out? Which sectors responded as predicted? Build your own calibration log to know how reliable the predictions are.
5. **Use 1st-order for trading, 2nd/3rd-order for investing**: Short-term traders should act on 1st-order impacts; long-term investors should position for 2nd/3rd-order structural shifts.

### What to Completely Avoid

- **Don't use it for post-hoc rationalization**: "The market went up after RBI cut — tell me why" is a different, lower-value use case. Use it prospectively.
- **Don't assume the "Base" scenario always plays out**: Markets regularly surprise. Assign your own position sizing to account for 30–40% probability that the non-base scenario occurs.
- **Never rely solely on this for individual stock buys** — it identifies sectors, not specific stock quality. Cross-reference with the Stock Analysis skill before acting.
- **Avoid using for ultra-short-term (< 1 week) event trades without the options advisor**: Binary event trades on stocks are high-risk; always define your risk with options rather than naked stock positions.

---

## 5. FII/DII Flow Tracker

**File**: `skills/fii-dii-flow-tracker/SKILL.md`
**Category**: Institutional Flow Analysis | **Input**: None required (fetches live via web)
**Best for**: Understanding the dominant market force, timing entries/exits, gauging market direction

### What It Does

Fetches daily FII (Foreign Institutional Investor) and DII (Domestic Institutional Investor) net buy/sell data from NSE/BSE sources. Computes MTD/YTD totals, FII:DII ratio, rolling correlation with Nifty, and identifies the current flow regime (6 regimes: FII Net Buyer, FII Net Seller, DII Absorption, Dual Buying, Dual Selling, Transition).

### How to Use It

```
"Give me today's FII/DII flows and market interpretation"
"What has been the FII/DII trend this month?"
"Are FIIs buying or selling in the current week?"
"Compare FII flows in equity vs derivatives — what's the smart money doing?"
```

**Flow significance thresholds**:
| Daily Net Flow | Significance |
|---------------|-------------|
| < ₹1,000 cr  | Minor |
| ₹1,000–2,000 cr | Moderate |
| ₹2,000–5,000 cr | Significant |
| > ₹5,000 cr | Major |

### Pros

- **Tells you who is driving the market**: FIIs dominate large-cap moves; DIIs (primarily mutual funds via SIPs) absorb selling and support mid-caps. Knowing the regime helps you understand the "why" behind index moves.
- **Absorption rate is a gem**: The % of FII selling offset by DII buying tells you whether the market has structural support or is in free fall.
- **FII derivative positioning as leading indicator**: When FIIs are net short in index futures while selling in cash, it signals a coordinated bear view — much more powerful than just looking at cash flows.
- **Identifies the "DII Absorption" regime**: This is a historically strong buy signal — FIIs selling but DIIs aggressively absorbing = market likely to hold and reverse.
- **Seasonal patterns are calibrated**: The skill knows that FIIs typically sell in Jan–Mar (US tax season, calendar year end rotation) and buy after Indian Q1 results.

### Cons and Limitations

- **FII flows are not always predictive**: FIIs have been net sellers during many of India's strongest bull runs (2020–2021 rally was mostly DII and retail-driven). The relationship is not 1:1.
- **Data has a 1-day lag minimum**: You're always trading on yesterday's flows. For fast-moving markets, this is stale.
- **Sector-level FII data is unreliable and delayed**: SEBI requires FII sector disclosures but these are monthly, not daily. The skill can only estimate sector allocation.
- **DIIs now often buy on weakness strategically** — their SIP-driven inflows are not discretionary. When mutual fund managers are uncertain, they can also sit on cash, making DII data less reliable as a sentiment indicator.
- **Cannot distinguish between FII sub-types**: A sovereign wealth fund buying 5-year infrastructure bonds reads the same as a hedge fund doing arbitrage. Both show as FII inflows.

### How to Extract Maximum Benefit

1. **Watch the 10-day rolling trend, not the daily number**: A single day's ₹3,000 cr FII sell is noise. 10 consecutive days of ₹2,000+ cr selling is a regime shift. Ask for the trend.
2. **The DII:FII ratio at extremes is most useful**: When DII absorption > 90% of FII selling for 5+ days, the market is showing remarkable resilience — a historically bullish signal.
3. **Stack with market breadth**: FII buying + healthy breadth (80+ score) = strong bull regime. FII buying + declining breadth = narrowing rally, be cautious.
4. **Use FII futures data for directional bias**: "What is the FII net position in Nifty index futures?" is the most forward-looking data point. Net short in futures while net buying in cash = hedged institutional player, not outright bullish.
5. **Monthly cumulative flows trump daily noise**: Ask for "MTD FII flows" or "YTD FII flows" — the monthly picture is what drives Nifty trend; daily is just volatility.

### What to Completely Avoid

- **Don't buy or sell purely based on daily FII numbers** — a ₹2,000 cr sell day does not mean the market will fall. Context (broader trend, global cues, valuations) matters.
- **Never use FII flows in isolation for mid-cap/small-cap timing** — FII flows directly impact Nifty 50 large-caps. For small/mid-caps, DII + retail sentiment is more relevant.
- **Don't confuse FII flows with FPI flows** — they are often used interchangeably but technically FPIs include sub-categories. In current SEBI framework, FPIs are the correct term; the skill's logic handles this.
- **Avoid anchoring to a specific "magic number"** — the ₹5,000 cr "Major" threshold was calibrated to recent market conditions. In low-volume periods, even ₹2,000 cr can be significant; in high-activity months, ₹5,000 cr is routine.

---

## 6. Options Strategy Advisor

**File**: `skills/options-strategy-advisor/SKILL.md`
**Category**: F&O Strategy | **Input**: Underlying, view, expiry, risk tolerance
**Best for**: Structuring defined-risk option trades, understanding Greeks, comparing strategies before entry

### What It Does

Full options strategy engine covering 17 strategies. Given your market view and risk parameters, it fetches live data (with MCP) or uses the Black-Scholes CLI, computes net Greeks, calculates SEBI-compliant margins, draws ASCII payoff diagrams, and generates a complete strategy card with entry price, max profit/loss, breakeven points, and risk management guidance.

### How to Use It

```
# Strategy selection by view
"I'm bullish on NIFTY for this week. IV is 15%. What's the best strategy?"
"HDFC Bank results are next week. I expect a big move but don't know direction. Options idea?"
"I want to sell premium on BANKNIFTY but limit my risk. What are my options?"

# Direct strategy request
"Set up an Iron Condor on Nifty for this expiry"
"Give me a Bull Call Spread on RELIANCE — I'm bullish but want defined risk"
"Price a Long Straddle on NIFTY around the RBI policy date"

# CLI for quick pricing
python3 skills/options-strategy-advisor/scripts/black_scholes.py price --spot 24000 --strike 24500 --expiry 7 --vol 0.15 --type call
python3 skills/options-strategy-advisor/scripts/black_scholes.py strategy --name iron_condor --spot 24000 --expiry 7 --vol 0.15
```

**17 Strategies supported**: Covered Call, Cash-Secured Put, Protective Put, Collar, Bull/Bear Call/Put Spreads, Long/Short Straddle/Strangle, Iron Condor, Iron Butterfly, Calendar Spread, Diagonal Spread, Ratio Spread.

### Pros

- **Greeks at position level**: Shows net Delta, Gamma, Theta, Vega for the entire position — critical for managing multi-leg strategies
- **SEBI margin compliance**: Uses SPAN + exposure margin framework; gives you the actual capital requirement before you enter
- **ASCII payoff diagram**: Visual profit/loss curve across a range of underlying prices; helps you understand your risk profile instantly
- **India VIX context**: Interprets current IV levels against India VIX percentile to guide strategy selection (high IV → sell premium; low IV → buy premium)
- **Transaction cost modeling**: Includes STT (0.0625% for options sell), exchange fees, brokerage — so the P/L is realistic, not theoretical
- **17 strategies**: Covers beginner to advanced, directional to volatility plays

### Cons and Limitations

- **Black-Scholes limitations for Indian weekly options**: BS assumes European exercise (correct for NSE) but also assumes log-normal distribution. Indian options, especially weekly Nifty options near expiry, exhibit extreme fat tails and vol skew that BS underprices. Deep OTM strikes will be mispriced.
- **IV surface is not fully modeled**: The skill uses a single IV input, not a full volatility surface. In reality, different strikes have different IVs (volatility smile/skew). This matters most for strategies that span multiple strikes.
- **Margin calculations are approximate**: SEBI's SPAN margin can change intraday based on market conditions; treat the output as a baseline, not guaranteed.
- **No real-time options chain without MCP**: Without Zerodha or Groww MCP, you must manually input spot price, IV, and strikes. The CLI still works but is not live.
- **Theta decay is non-linear**: The skill models theta correctly in theory, but the actual decay acceleration in the last 3 days of weekly expiry is faster than BS predicts.
- **Liquidity is not assessed**: The skill cannot tell you whether the strike you want to trade has sufficient OI and bid-ask spread. A theoretically perfect strategy on an illiquid strike is unexecutable.

### How to Extract Maximum Benefit

1. **Always check India VIX percentile before choosing a strategy**: VIX < 12 → buy premium strategies (straddles, strangles, debit spreads). VIX > 20 → sell premium strategies (iron condors, short straddles). The skill will flag this, but internalize it.
2. **Ask for a comparison of 2–3 strategies for your view**: "I'm neutral to mildly bullish on Nifty. Compare Bull Call Spread vs Iron Condor vs Short Strangle." This shows the risk/reward tradeoff explicitly.
3. **Use the Greeks to manage existing positions**: "I'm long 2 lots of 24200 CE. Delta is 0.6, Vega is high. How do I hedge my vega risk?" The skill can suggest delta-neutral adjustments.
4. **The payoff diagram is your most important output**: Print it (or screenshot it) before entering. When the market moves against you at 2 PM, referencing the diagram prevents panic decisions.
5. **Use the CLI for quick strike comparison**: Before placing an order, run the CLI to compare the premium/delta of ATM vs 1 OTM vs 2 OTM strikes. Often 1 OTM has 80% of ATM's delta at 60% of the cost.

### What to Completely Avoid

- **Never trade weekly options without understanding theta decay**: Buying ATM weekly options on Monday means you need the underlying to move by Thursday just to recover theta loss. Ask the skill to show you the theta impact.
- **Don't sell naked options (unhedged short calls/puts)** — the margin requirement is high and the risk is theoretically unlimited. Use spreads (defined risk) instead.
- **Never trade during earnings/RBI policy without adjusting your IV assumption** — IV typically spikes 2–3x before events and collapses after. Price your strategy on post-event IV, not pre-event IV.
- **Avoid ratio spreads and complex multi-leg strategies if you're a beginner** — the margin, adjustment, and risk management complexity is substantial.
- **Don't enter any strategy if the bid-ask spread is > 10% of the option price** — this makes entry and exit costly. Check OI and market depth before executing.
- **Never ignore the F&O ban list** — stocks in the F&O ban list have restricted OI; options pricing on these is abnormal. The skill may not always flag this automatically.

---

## 7. Backtest Expert

**File**: `skills/backtest-expert/SKILL.md`
**Category**: Strategy Validation | **Input**: Strategy rules + backtest results
**Best for**: Validating any trading strategy before real-money deployment, identifying overfitting

### What It Does

A 5-dimensional scoring system for evaluating backtested strategies. You provide your strategy rules and backtest metrics (win rate, avg win/loss, drawdown, trade count, years tested, parameters, market segment). The skill scores it across Sample Size, Expectancy, Risk, Robustness, and Execution Realism — including India-specific cost modeling for STT, stamp duty, GST, and slippage.

### How to Use It

```
# Evaluate a backtest
python3 skills/backtest-expert/scripts/evaluate_backtest.py \
  --trades 150 --win-rate 0.58 --avg-win 2.5 --avg-loss 1.2 \
  --max-drawdown 15 --years 3 --parameters 4 --segment delivery

# Via Claude Code
"Evaluate my strategy:
 - 200 trades over 5 years
 - Win rate: 62%, Avg win: 3.2R, Avg loss: 1R
 - Max drawdown: 18%, Sharpe: 1.4
 - 6 parameters, NSE F&O options buying
 Should I deploy this?"

"My backtest shows 150% CAGR on Nifty futures. Is this too good to be true?"
```

**Verdict zones**:
| Score | Verdict |
|-------|---------|
| 80–100 | Deploy with position sizing |
| 60–79 | Refine and retest |
| 40–59 | Refine with caution (likely overfitted) |
| 0–39  | Abandon |

### Pros

- **India cost model is precise**: Many backtests fail in live trading due to incorrect cost assumptions. This skill uses actual STT rates (0.1% delivery, 0.025% intraday, 0.0625% F&O), stamp duty, exchange charges, and realistic slippage ranges for NSE.
- **Identifies the 6 failure patterns**: Cost Killer, Parameter Peak, Regime Specialist, Survivorship Illusion, Circuit Limit Trap, Gap Risk Destroyer — these are real Indian market failure modes many backtests miss.
- **Forces hypothesis articulation**: You must state your edge in one sentence. Vague strategies ("I buy when RSI is oversold") fail this test and correctly score lower.
- **Walk-forward analysis guidance**: The skill recommends how to split your data for in-sample/out-of-sample testing — essential for avoiding overfitting.
- **Multi-regime testing**: Checks whether the strategy works across bull (2017, 2021, 2023), bear (2008, 2020), and sideways (2015–2017, 2022) market regimes.

### Cons and Limitations

- **The skill evaluates strategy quality but cannot run backtests itself** — you must supply results from Amibroker, TradingView Pine Script, Python backtesting (Backtrader, zipline, vectorbt), or manual testing.
- **Monte Carlo and parameter sensitivity analysis** requires manual setup — the skill guides you on what to do, but the computation is yours to run.
- **Historical data quality for NSE is inconsistent**: yfinance has gaps and errors in pre-2010 Indian stock data. Circuit breakers in 2008/2020 are often missing. This affects regime testing quality.
- **Cannot detect look-ahead bias directly**: If your strategy code accidentally uses future data (e.g., using close price to generate signals on the same candle), the skill cannot detect this unless you describe the code precisely.
- **F&O expiry slippage is hard to model**: The actual slippage on weekly expiry Thursday for Nifty options can be 3–5x the normal slippage model. Few backtests account for this.

### How to Extract Maximum Benefit

1. **Run the cost model first, before anything else**: Ask the skill to calculate the round-trip cost for your segment (delivery/intraday/F&O) and strategy frequency. Many "profitable" backtests turn negative after realistic costs.
2. **Test across all 4 regimes**: Bull (2021, 2023), Bear (2020 March, 2008), Sideways (2015–2016), High-volatility (2020, 2022). A strategy that only works in one regime is not deployable.
3. **The 6 failure patterns are a checklist**: Go through all 6 explicitly. "Does my strategy suffer from the Circuit Limit Trap?" — Indian stocks hit upper/lower circuits; orders cannot execute. Most backtesting software ignores this.
4. **Stress test parameters by ±20%**: If your strategy uses a 20-day EMA, test 16, 18, 22, 24-day EMA. If performance degrades sharply with minor changes, you're overfit.
5. **Require a minimum 100 trades** before taking any score seriously. With < 100 trades, the win rate has too much variance to be statistically meaningful.

### What to Completely Avoid

- **Never deploy a strategy that scores below 60** — even at 60, you're in "Refine" territory. The human bias toward deploying something you worked hard on is strong; fight it.
- **Don't optimize until a score of 80 appears on the same dataset** — this is the definition of overfitting. Optimize on in-sample, validate on out-of-sample only.
- **Never ignore the circuit limit failure pattern for small-caps** — if your strategy trades mid/small-caps on NSE, and your backtest software doesn't model circuit breakers, your results are fantasy.
- **Don't count partial fills as full fills** — F&O markets, especially mid-week expiry options, often partially fill at your limit price. Assume 10–20% of trades in thin instruments will be partial.
- **Avoid comparing your gross backtest return to Nifty returns** — always compare risk-adjusted returns (Sharpe, Calmar) after all costs.

---

## 8. India Market Breadth

**File**: `skills/india-market-breadth/SKILL.md`
**Category**: Market Health Analysis | **Input**: None (fetches data)
**Best for**: Equity allocation decisions, confirming/denying market rally quality, early warning of reversals

### What It Does

Measures the internal health of the Indian market beyond the Nifty index level. Computes 5 breadth indicators (Advance/Decline ratio, % stocks above 200 DMA, % stocks above 50 DMA, New 52-week highs vs lows, Sector participation), scores them into a composite Health Score (0–100), and maps it to an equity exposure recommendation (25–100%).

### How to Use It

```
"What is the current market breadth and health score?"
"Is this Nifty rally healthy or narrow?"
"Should I increase equity exposure given current breadth?"
"Has breadth been deteriorating while Nifty makes new highs?"  ← The most important question
```

**Health zones and allocation**:
| Score | Zone | Equity Exposure |
|-------|------|----------------|
| 80–100 | Strong Breadth | 90–100% |
| 60–79 | Healthy | 75–90% |
| 40–59 | Neutral | 60–75% |
| 20–39 | Weakening | 40–60% |
| 0–19  | Critical | 25–40% |

### Pros

- **The most important divergence signal in markets**: When Nifty makes new highs but breadth is flat/declining, it historically precedes major corrections (detected before Oct 2021 top, Jan 2018 top, Jan 2008 top). This is the skill's single greatest value.
- **Goes beyond Nifty**: The Nifty 50 index is cap-weighted and can be propped up by 5–10 large-cap stocks. Breadth exposes whether the rally is broad-based (sustainable) or narrow (fragile).
- **% above 200 DMA is the primary long-term health gauge**: > 70% = structural bull market. < 40% = structural bear/correction. This is more stable than daily A/D ratio.
- **Sector participation score prevents false signals**: If only IT is rallying but all other sectors are flat, the participation score is low, signaling a narrow/unreliable rally.
- **Actionable allocation output**: Translates breadth data directly into an equity exposure recommendation.

### Cons and Limitations

- **A/D data from NSE includes all listed stocks** — including SME stocks, T2T stocks, and very illiquid names. These can distort the ratio. Ideally you'd want Nifty 500 breadth, but free data sources provide only the overall market.
- **McClellan Oscillator requires cumulative data history** — the skill's calculation accuracy improves with longer historical context. First-run breadth analysis may be less reliable than analysis run weekly over months.
- **Breadth data from web sources may have a 1-day lag** — real-time breadth requires an MCP or direct BSE data feed.
- **Breadth can stay divergent for months before resolving** — the Oct 2021 divergence lasted 2–3 months before the correction. Premature de-risking based on breadth divergence has a cost.
- **Small-cap breadth diverges from large-cap breadth** — if you invest primarily in Nifty 50 stocks, small-cap breadth deterioration matters less. The skill doesn't separate these well.

### How to Extract Maximum Benefit

1. **Run it weekly, every Sunday**: Track the health score as a time series. A declining trend (80 → 70 → 60 → 50) is more alarming than any single week's reading.
2. **The divergence check is your primary use case**: Ask specifically: "Is Nifty making new highs while breadth indicators are flat or declining?" This is the actionable question.
3. **Use it as a gating condition for the VCP screener**: Only act on VCP breakout signals when breadth > 60. Below that, even good setups fail at higher rates.
4. **Stack with FII/DII flows**: Healthy breadth + FII buying = best environment. Declining breadth + FII selling = stay defensive.
5. **Use the sector participation metric for rotation**: "Which sectors are showing the most stocks above their 200 DMA?" → this identifies sectors to overweight in the next month.

### What to Completely Avoid

- **Don't use breadth as a timing tool for exact entry/exit** — it's a regime classifier, not a market timer. Breadth at 40 does not mean sell everything today; it means reduce exposure over the next few weeks.
- **Never ignore a breadth health score below 30** — this is the level where historical NSE data shows significantly elevated probability of 10%+ index corrections within 3 months. This is not a lagging indicator at extremes.
- **Don't compare Indian breadth to US breadth as a benchmark** — NSE has different sector composition, more SMEs, and different trading patterns. Comparison is misleading.
- **Avoid acting on a single day's sharp A/D number** — a day with 2:1 advance/decline ratio after a holiday/settlement day is noise. Look at rolling averages.

---

## 9. India News Tracker

**File**: `skills/india-news-tracker/SKILL.md`
**Category**: News Intelligence | **Input**: Stock/sector/mode selection
**Best for**: Daily market briefing, tracking corporate events, monitoring regulatory changes

### What It Does

Aggregates and scores Indian market news from 4 tiers of sources (SEBI/BSE/NSE → MoneyControl/ET/Mint → Screener/Trendlyne → Social). Classifies each news item across 13 categories (Earnings, M&A, Regulatory, Institutional, etc.), assigns an impact score (1–10), and classifies sentiment (Bullish/Bearish/Neutral). Available in 7 modes from daily briefing to bulk/block deal monitoring.

### How to Use It

```
# Daily briefing (run every morning before 9 AM)
"Give me today's Indian market news briefing"
"What happened overnight that will impact Indian markets today?"

# Stock-specific
"Get all recent news for RELIANCE"
"What's the latest news on ADANI GROUP?"

# Sector watch
"What's happening in the pharma sector this week?"
"Any regulatory changes affecting Indian banking stocks?"

# Corporate actions
"What dividend/bonus/split actions are announced this week?"

# CLI mode
python3 skills/india-news-tracker/scripts/news_fetcher.py --stock RELIANCE
python3 skills/india-news-tracker/scripts/news_fetcher.py --sector banking --days 7
```

**Impact score interpretation**:
| Score | Level | Action |
|-------|-------|--------|
| 9–10 | Critical | Immediate attention |
| 7–8 | High | Review before trading |
| 5–6 | Medium | Awareness |
| 3–4 | Low | Background knowledge |
| 1–2 | Noise | Skip |

### Pros

- **4-tier source hierarchy**: Prioritizes official sources (SEBI, NSE, BSE exchange filings) over secondary media — reduces misinformation risk
- **Impact scoring prevents information overload**: The 1–10 system filters the daily flood of market news to what actually matters (+1 for Nifty 50 stocks, +1 if unexpected, -1 if priced in)
- **13 event categories**: Makes it easy to filter for only what's relevant to your portfolio (e.g., "show me only Earnings and M&A events, score 7+")
- **Pre-market briefing mode**: Designed to be run before 9:15 AM, synthesizing overnight developments, global cues, and corporate actions for the trading day
- **RSS feed integration**: The CLI (`news_fetcher.py`) uses feedparser to pull live RSS from BSE corporate announcements — actual company filings in near-real-time

### Cons and Limitations

- **Cannot access paywalled content**: Mint premium articles, ET Prime, Bloomberg Quint subscribers-only content is inaccessible. High-quality analysis from these sources is missed.
- **Social sentiment (Tier 4) is unreliable for individual stocks**: X/Twitter "sentiment" for Indian stocks is heavily influenced by promoter-affiliated accounts, tips channels, and WhatsApp-forwarded content.
- **No real-time BSE/NSE exchange feed**: The RSS feeds used have 15-minute delays. For time-sensitive corporate announcements (quarterly results, merger news), official BSE/NSE websites should always be checked directly.
- **Sentiment classification is approximate**: "Bearish" vs "Ambiguous" distinction for complex regulatory changes or mixed quarterly results involves judgment calls that may not always align with market reaction.
- **Cannot filter by your specific portfolio**: Without knowing your holdings, the briefing covers market-wide news. You'll need to specify your stocks/sectors explicitly.

### How to Extract Maximum Benefit

1. **Build a morning routine**: Run the daily briefing mode every day at 8:45 AM. 15 minutes of synthesized news > 1 hour of random scrolling.
2. **Create a personal watchlist briefing**: "Get news for HDFC Bank, Bajaj Finance, Infosys, and Tata Motors from the last 3 days" — personalize it to your portfolio.
3. **Use score 7+ as your action filter**: Events scoring 7 or above require you to reassess any position in that stock before the day's trade.
4. **Monitor bulk/block deals for institutional signals**: Institutional block deals often precede major price moves. A large mutual fund selling a significant stake is a structural signal, not just a trade.
5. **Track regulatory news for sector-wide impacts**: A single SEBI circular or RBI guideline can affect 10–20 stocks. The regulatory monitor mode catches these before they're widely covered.

### What to Completely Avoid

- **Never trade on Tier 4 (social media) signals** — these are noise at best, manipulation at worst. Social sentiment from WhatsApp groups and anonymous Twitter accounts has no place in a systematic process.
- **Don't treat news sentiment as a trading signal in isolation** — "Bullish news" does not mean "buy." Markets often sell on good news (buy the rumor, sell the news). Context and technical levels matter.
- **Avoid acting on unverified news** — the skill tries to cross-check, but breaking news from unofficial sources should be verified against BSE/NSE exchange filings before any trade.
- **Don't use it as your only research for a position** — news tells you what happened; it doesn't tell you whether the market has priced it in. Always pair with price action and fundamentals.
- **Never rely on it for real-time results monitoring** — quarterly results hit BSE/NSE filings before any news aggregator. Go directly to BSE corporate filings for live results.

---

## 10. Weekly F&O Trade Planner

**File**: `skills/weekly-fno-trade-planner/SKILL.md`
**Category**: F&O Trading Workflow | **Input**: Current date, available capital
**Best for**: Retail F&O traders wanting a disciplined, structured approach to weekly option buying

### What It Does

End-to-end weekly F&O trading workflow across 6 phases: macro thesis development → instrument selection → technical confirmation → strategy/entry planning → **gap probability analysis (Phase 4.5, the highest-value phase)** → execution → daily position management. Produces a complete Trade Card with exact strikes, entry price, stop-loss, targets, and risk:reward.

### How to Use It

```
# Sunday evening — weekly plan
"Run the weekly F&O trade planner for next week (week of [date])"
"What's the highest-conviction F&O trade for this week?"

# Monday morning — gap probability check (CRITICAL)
"Run Phase 4.5 gap probability check. GIFT Nifty is at 24,350.
 US futures are up 0.8%. Crude is flat. What should I do?"

# Daily management
"Review my position: Long 24200 CE at ₹145, underlying now at 24,400.
 It's Wednesday 2 PM. Should I book partial profit or trail?"

# Complete workflow
"Run the complete weekly F&O planner.
 My capital is ₹2,00,000. This week's dominant theme is [your macro view]."
```

### Phase 4.5 — The Gap Probability Engine (Most Critical Feature)

This is the skill's highest-value output. Before Monday open (or any day before close), it:
1. Checks GIFT Nifty vs previous close
2. Analyzes US futures, crude, Asian markets, dollar index
3. Calculates a gap probability score (0–100)
4. If score ≥ 60 AND conviction ≥ 4: **immediately buy 2–4 strikes OTM before today's close**

**Why this matters**: An OTM option the evening before a gap up/down costs 40–70% less than buying it after the gap opens. This is the single biggest edge available to retail F&O option buyers.

### Pros

- **Enforces discipline**: The 6-phase workflow forces you to answer hard questions (What is my thesis? What is my conviction? What is my stop-loss?) before entering — preventing impulsive trades
- **Phase 4.5 is uniquely valuable**: No other skill or tool in this repo (or most trading tools) systematically flags pre-gap entry opportunities. For weekly options, buying before a gap is a structural edge.
- **Complete Trade Card format**: Everything in one place — no last-minute mental math under market pressure
- **Non-negotiable risk rules**: 10 hard rules (max 40% capital, GTT stop always set, no averaging down, weekend rule, max 2 positions). These prevent the most common retail F&O disasters.
- **Daily position management SOP**: Trailing stop-loss framework based on profit achieved (entry=-30%, +30% profit→BE, +50%→+20%, etc.) removes emotion from profit-taking
- **Weekend rule**: Force-close weekly options before Friday close — prevents the common mistake of holding options into the weekend and losing 30–50% to theta.

### Cons and Limitations

- **SEBI's 2024 F&O restrictions**: SEBI significantly increased margins, restricted weekly expiry contracts to indices only (no more weekly stock options), and reduced the number of weekly expiry days. The skill's references may need updating with the latest SEBI circular.
- **The skill cannot execute trades** — without Zerodha Kite MCP, it only plans; you must execute manually. Execution timing on phase 4.5 requires you to be at your terminal before 3:20 PM.
- **GIFT Nifty is an imperfect pre-open indicator**: GIFT Nifty correlation with NSE Nifty open is ~80% in normal conditions; it breaks down during extreme events (elections, global crashes). Don't treat it as a guaranteed predictor.
- **Conviction scoring is subjective**: The 1–5 scale is your own assessment of thesis quality. Overconfident traders will consistently rate their thesis higher than warranted.
- **Retail F&O statistics are sobering**: SEBI's 2023 study found 89% of individual equity F&O traders incur losses. This skill provides structure but cannot guarantee profitability — discipline in execution is everything.
- **Phase 4.5 requires action before 3:20 PM on weekdays**: If you're not at your terminal daily, the pre-gap entry advantage is not accessible.

### How to Extract Maximum Benefit

1. **Make Phase 4.5 your daily habit, not just Monday**: Run gap probability check every day after 3:00 PM. A pre-gap entry on any weekday can be a major profit multiplier.
2. **Start with index F&O only**: Nifty and BankNifty have the best liquidity, tightest spreads, and most predictable behavior. Avoid stock options until you're profitable on indices for 3 months.
3. **Use maximum 40% capital per trade — always**: The skill specifies this as non-negotiable for a reason. F&O is leveraged; position sizing is what separates survivors from blown accounts.
4. **Paper trade the full workflow for 4 weeks before real money**: Go through all 6 phases in simulation mode. Discover where your judgment is weak without financial consequence.
5. **Journal every trade in full**: The Trade Card is your journal. Write down which phase you skipped or shortcut. Almost every losing trade will trace back to a skipped phase.
6. **Exit at T1 (first target), not greed**: The skill's partial profit rules (40–50% at T1) are calibrated for weekly options that decay fast. Holding for T2 or T3 requires the underlying to keep moving your way — unreliable.
7. **The "no averaging down" rule is your capital protector**: Weekly options are time-decaying assets. An option bought at ₹100 that goes to ₹60 is not "cheap" — it's ₹60 because time and momentum have moved against you.

### What to Completely Avoid

- **Never hold weekly options over the weekend** — this is rule #5 for a reason. 2 calendar days = 30–50% theta decay for ATM/OTM options. There is no scenario where this makes sense for weekly positions.
- **Don't trade F&O without a GTT stop-loss set immediately after fill** — human willpower to "close at stop" fails when you're in a meeting or away from your screen. Always set the GTT.
- **Never open more than 2 F&O positions simultaneously** — this is the diversification trap in F&O. Correlation between indices and stocks in a single market is high; 3 F&O positions often act like 3x your intended exposure.
- **Don't trade when conviction is < 3/5** — the skill has a hard gate: conviction below 3, no trade. If you're uncertain, stay out. There will be another week.
- **Avoid deep OTM options (> 4 strikes away) for directional trades** — they are lottery tickets, not trading instruments. Save deep OTM purchases only for pre-gap entries identified in Phase 4.5.
- **Never revenge trade**: After a stop-loss, the instinct to "make it back" is a capital-destruction machine. The skill specifies no revenge trading — enforce it.

---

## Using Skills Together: Power Workflows

### Workflow 1: Weekly Stock Selection (Best for Swing Traders)

```
Sunday 6 PM:
1. India Market Breadth → Is health score > 60? (Gate: if < 60, skip new entries)
2. FII/DII Flow Tracker → What is the institutional regime?
3. NSE VCP Screener → Run Nifty 500 screen, get top-10 candidates
4. Technical Analyst → Upload charts for top-5 VCP stocks, get scenario analysis
5. India Stock Analysis → Full report on 2–3 highest-conviction picks
→ Output: 2 watchlist stocks with entry price, stop-loss, and target

Monday 9 AM:
6. India News Tracker → Morning briefing, check for any news on watchlist stocks
→ Gate: If impact score 7+ negative event on watchlist stock, skip it
```

### Workflow 2: F&O Weekly Planning (Best for Options Traders)

```
Sunday 7 PM:
1. India Market Breadth → Breadth regime (determines bias: bullish/bearish/neutral)
2. FII/DII Flow Tracker → FII positioning in index futures
3. Scenario Analyzer → Any events this week? Budget/RBI/results?
4. Weekly F&O Trade Planner (Phase 1–4) → Build trade plan

Monday 8:45 AM:
5. India News Tracker → Daily briefing
6. Weekly F&O Trade Planner (Phase 4.5) → Gap probability check
→ If gap probability > 60%: buy before market open
→ If gap probability < 60%: wait for Phase 5 (9:15 AM entry)
```

### Workflow 3: Pre-Earnings Research (Best for Event Traders)

```
5 days before results:
1. India Stock Analysis → Full fundamental report, note EPS estimates
2. India News Tracker → Sector news, analyst upgrades/downgrades
3. Scenario Analyzer → "Company X reports results next week — build scenarios"
4. Options Strategy Advisor → Price a Long Straddle or Iron Condor for the event
→ Output: Position to take before results, exit plan for each scenario
```

### Workflow 4: Macro Event Response (Best for Institutional-Style Thinking)

```
Day before RBI Policy / Union Budget:
1. Scenario Analyzer → 3 scenarios with sector impacts
2. FII/DII Flow Tracker → Current institutional positioning
3. Options Strategy Advisor → Structure hedges or directional plays
4. India Stock Analysis → Identify top 2–3 stocks per scenario

Day of event:
5. India News Tracker → Real-time news tracking
6. Technical Analyst → Post-event chart analysis for confirmation
```

---

## Common Mistakes Across All Skills

1. **Using skills in isolation**: Each skill is most powerful in combination. Using VCP Screener without checking market breadth results in buying breakouts in a bear market.

2. **Ignoring the conviction/score thresholds**: Every skill has explicit gates (VCP score < 65 = skip, F&O conviction < 3 = no trade, backtest score < 60 = don't deploy). These thresholds exist for statistical reasons. Override them at your peril.

3. **Treating Claude's analysis as a guaranteed prediction**: All outputs are [Inference] unless citing specific data. The market doesn't care about the analysis. These are probabilistic frameworks, not oracles.

4. **Using skills during market hours under time pressure**: These skills are designed for pre-market research and post-market review — not for making 2-minute decisions while watching a live order book. Prepare before the market opens.

5. **Skipping the risk section**: Every skill has explicit risk flags, failure modes, and "what could go wrong" analysis. The bull case is easy to see; the risks require deliberate attention.

6. **Not updating analysis after corporate events**: A fundamental analysis from 3 months ago is stale after quarterly results. Re-run the stock analysis skill after every earnings release.

7. **Conflating data quality with analysis quality**: These skills output quality proportional to their input data quality. Bad data from yfinance → bad output. Always sanity-check key numbers against primary sources (BSE website, NSE website).

---

## Setup Checklist

```bash
# 1. Install dependencies
pip install -e ".[dev]"
pip install feedparser

# 2. Test individual CLIs
python3 skills/nse-vcp-screener/scripts/screen_vcp.py --universe nifty50
python3 skills/options-strategy-advisor/scripts/black_scholes.py price --spot 24000 --strike 24500 --expiry 7 --vol 0.15 --type call
python3 skills/backtest-expert/scripts/evaluate_backtest.py --trades 100 --win-rate 0.55 --avg-win 2.0 --avg-loss 1.0 --max-drawdown 12 --years 3 --parameters 3 --segment delivery
python3 skills/india-news-tracker/scripts/news_fetcher.py --stock RELIANCE

# 3. Optional: Configure MCP for live data
# Zerodha Kite MCP → live quotes, OI, order placement
# Groww MCP → live fundamentals, candle data

# 4. Load skills into Claude (choose one method):
# A: Paste SKILL.md contents directly into conversation
# B: Claude Projects → Upload SKILL.md as project document
# C: Claude Code → Open this repo directory and reference skill
```

**Recommended starting order for new users**:
1. India Market Breadth (understand the macro environment first)
2. India Stock Analysis (learn individual stock research)
3. Technical Analyst (layer on chart reading)
4. NSE VCP Screener (automate stock finding)
5. FII/DII Flow Tracker (institutional context)
6. Scenario Analyzer (event-driven thinking)
7. India News Tracker (daily intelligence)
8. Options Strategy Advisor (when comfortable with F&O basics)
9. Backtest Expert (when you have a strategy to validate)
10. Weekly F&O Trade Planner (final integration of everything)

---

*Generated: March 2026 | Covers all 10 skills as of commit `5bb7658`*
*This is a living document — update after major skill revisions.*
