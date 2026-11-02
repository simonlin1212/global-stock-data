<p align="center"><a href="README_zh.md">简体中文</a> | <b>English</b></p>

<h1 align="center">global-stock-data</h1>

<p align="center">
  <b>US Market Data for AI Coding Assistants</b><br>
  Options Greeks · 0DTE Flow · VIX Term Structure · SEC Filings & Insider Trades · Short Data · Fundamentals · Screener · Global Rates & FX · Taiwan · Energy · US Macro
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License: Apache 2.0"></a>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white" alt="Python 3.9+"></a>
  <a href="https://github.com/simonlin1212/global-stock-data/stargazers"><img src="https://img.shields.io/github/stars/simonlin1212/global-stock-data?style=social" alt="GitHub stars"></a>
  <a href="#architecture"><img src="https://img.shields.io/badge/layers-17-2ea44f.svg" alt="Layers"></a>
  <a href="#endpoints"><img src="https://img.shields.io/badge/endpoints-50%2B-2ea44f.svg" alt="Endpoints"></a>
  <a href="#data-sources"><img src="https://img.shields.io/badge/sources-18-2ea44f.svg" alt="Sources"></a>
  <a href="#data-sources"><img src="https://img.shields.io/badge/auth-zero-success.svg" alt="Zero-Auth"></a>
</p>

---

<p align="center">
  <a href="#architecture">Architecture</a> ·
  <a href="#compliance-tiers">Compliance</a> ·
  <a href="#quick-start">Quick Start</a> ·
  <a href="#endpoints">Endpoints</a> ·
  <a href="#data-sources">Data Sources</a> ·
  <a href="#faq">FAQ</a>
</p>

> **Full-stack market data for US equities** — 17 layers · 50+ endpoints · 18 sources · zero-auth · only `requests`.
>
> **Official-source-first (V2.0):** CBOE options with full Greeks + **0DTE** flow, FINRA market-wide short volume, SEC EDGAR filing stream + a free market-wide screener, Treasury / CFTC / earnings calendar. **Every source is labeled with its compliance tier, terms quoted verbatim** — "official" does not mean "free to use".
>
> **New in V2.1:** a global rates & FX layer — policy rates for 38 central banks (BIS), ECB FX / key rates / euro-area yield curve, NY Fed SOFR / EFFR, Fed H.10 FX / H.6 money stock, and US Treasury auctions + federal debt, all tier S. Yahoo K-line now returns true full-history daily bars in exchange-local dates, builds weekly / monthly / quarterly bars from daily data, and works for Japan / Taiwan / Korea / India / Europe and more.
>
> **New in V2.2:** CBOE volatility indices (VIX since 1990) and the VIX term structure, FINRA short interest (twice a month, with days to cover), SEC Form 4 insider trades line by line and 13F institutional holdings, the Taiwan Stock Exchange / TPEx official open APIs (latest Taiwan quotes, P/E, monthly revenue; commercial use OK), and US EIA weekly crude oil / natural gas inventories plus spot prices (spot prices are tier B, see Compliance tiers).
>
> **New in V2.3:** a US macro layer — the US Bureau of Labor Statistics' registration-free public API: 23 common indicators in one request, including CPI / core CPI, nonfarm payrolls, the unemployment rate, average hourly earnings, PPI, JOLTS job openings, import prices, the Employment Cost Index and labor productivity, with month-over-month and year-over-year changes computed locally, the official release schedule and each indicator's revision rules; plus the Department of Labor's weekly initial / continued jobless claims (since 1967). All tier S; cite the source and the retrieval date.

A single self-contained Skill file that turns raw US stock data — scattered across many sources — into a toolkit an AI coding assistant can call directly. You no longer memorize Eastmoney secid prefixes, Yahoo crumb auth flows, or SEC EDGAR CIK mappings. That's all handled.

> *This project distributes **code, not market data**. Data is fetched by the user under each source's own terms. For commercial use, rely only on tier-S sources.*

> Compatible with [Claude Code](https://github.com/anthropics/claude-code) · [Codex](https://github.com/openai/codex) · [OpenClaw](https://github.com/openclaw/openclaw). The Skill file is structured Markdown + embedded Python — any AI coding assistant with context injection can use it.

---


## Architecture

```
US Full-Stack Data · 17-layer architecture · V2.3
│
├── Market Data      Sina + Tencent + Eastmoney push2        Real-time quotes, 25-78 fields
├── K-line           Sina(back to 1984) + Yahoo chart        Daily / weekly / monthly / minute; JP/TW/KR/IN/EU via suffix
├── Technical Ind.   MA/EMA + MACD + RSI + KDJ + Bollinger   Pure Python, zero extra deps
├── Fundamentals     Eastmoney + Yahoo + SEC XBRL            Statements + key metrics + holdings
├── Fund Flow        Eastmoney push2his                      Daily main/large/medium/small flow
├── Options          Yahoo crumb                             Options chain (US only)
├── SEC Filing       EDGAR submissions + XBRL                10-K/10-Q/8-K + 503 GAAP metrics (US)
└── Tools            Eastmoney + Yahoo + SEC CIK             Search + market list + news + ticker↔CIK

━━━ New in V2.0 (official-source-first) ━━━
├── Options·CBOE     cdn.cboe.com official feed   ⭐ full chain + IV + delta/gamma/vega/theta/rho + 0DTE + flow
├── Short Volume     FINRA Reg SHO                ⭐ market-wide daily short volume (12,112 symbols) + series
├── Filing Stream    EDGAR daily index + FTS      ⭐ Form 4 / 8-K / 13F same-day + full-text since 2001
├── Market Screener  EDGAR frames                 ⭐ any XBRL tag across the whole market (1,842~5,309 co.) — free
└── Macro / Calendar Treasury + CFTC + Nasdaq     ⭐ yield curve (1M~30Y) + COT + earnings calendar

━━━ New in V2.1 ━━━
└── Global Rates/FX  BIS + ECB + NY Fed + Fed + Treasury ⭐ 38 central-bank policy rates + EUR/USD FX + SOFR/EFFR + euro curve + auctions

━━━ New in V2.2 ━━━
├── Volatility       CBOE index files             ⭐ VIX since 1990 + VIX9D/3M/6M/1Y term structure + VVIX/SKEW/OVX/GVZ
├── Short Interest   FINRA, twice a month         ⭐ all US open short positions (22,595 symbols) + days to cover + rankings
├── Insiders / 13F   SEC Form 4 + 13F             ⭐ insider trades (price/shares/10b5-1 box on the filing) + quarterly institutional holdings
├── Taiwan           TWSE + TPEx OpenAPI          ⭐ latest quotes + P/E + monthly revenue + margin for all listed/OTC stocks (commercial OK)
└── Energy           US EIA                       ⭐ weekly crude/gas inventories + WTI (since 1986)/Brent/Henry Hub daily spot prices (tier B)

━━━ New in V2.3 ━━━
└── US Macro         US Dept. of Labor BLS + ETA  ⭐ CPI/core CPI/payrolls/unemployment/earnings/PPI/JOLTS and more (23 indicators) + release schedule + weekly jobless claims
```

> Layer count: the original 8 layers + 5 in V2.0 (the CBOE options chain counts as its own layer) + 1 in V2.1 + 2 in V2.2 (Taiwan, energy) + 1 in V2.3 (US macro). The V2.2 volatility indices, short interest and Form 4 / 13F parsers are new sections inside existing layers and are not counted as layers.

---

## Compliance tiers

Source terms differ a lot. **"Official" does not mean "free to use".** These tiers come from reading each source's actual terms of service (2026-07-24; Yahoo / Nasdaq re-checked and 4 new official sources read on 2026-10-11; TWSE / TPEx and EIA read on 2026-10-12; the US Bureau of Labor Statistics and Department of Labor read on 2026-11-02). Quotes are verbatim.

<table>
<thead>
<tr><th nowrap>Tier</th><th nowrap>Commercial</th><th nowrap>Redistribute</th><th>Sources</th><th>Basis (quoted)</th></tr>
</thead>
<tbody>
<tr><td nowrap><strong>S</strong></td><td nowrap>✅</td><td nowrap>✅</td><td>SEC EDGAR / Treasury / CFTC / Federal Reserve Board / NY Fed / BIS / ECB / EIA (inventories) / TWSE / TPEx / US Dept. of Labor (BLS / ETA)</td><td>EDGAR: <em>&quot;Anyone can access and download this information <strong>for free</strong>&quot;</em>, <em>&quot;We <strong>allow scripted access</strong>&quot;</em> — hard limit <strong>10 req/s</strong>, User-Agent required. Fed Board: <em>&quot;public domain&quot;</em>. NY Fed: <em>&quot;personal or business purposes&quot;</em>, <em>&quot;through an automated process&quot;</em>. BIS: <em>&quot;unrestricted&quot;</em>. ECB: <em>&quot;reused free of charge&quot;</em>. EIA's own data: <em>&quot;public domain&quot;</em>. TWSE / TPEx: all 13 datasets we use are registered on Taiwan's government open data platform under the Open Government Data License, which allows use for any purpose, non-exclusive, irrevocable and royalty-free; both OpenAPI pages say they welcome integration, and the intellectual-property section of both sites' terms exempts data licensed to the open data platform, while asking that reproductions stay complete, with nothing arbitrarily added or removed (TPEx's OpenAPI page does not link the license and the platform lists web download URLs, so rating it S is this tool's judgment). BLS: <em>&quot;public domain&quot;</em>; its API terms ask users to cite the retrieval date and state <em>&quot;BLS.gov cannot vouch for the data or analyses derived from these data after the data have been retrieved from BLS.gov.&quot;</em> The key-free tier allows 25 requests a day, and trying to get around the limits can get you blocked. Department of Labor: <em>&quot;public domain&quot;</em>, without implying any affiliation with or endorsement by the DOL. <strong>BIS / ECB / Fed Board / TWSE / TPEx / BLS require attribution (BLS also the retrieval date), ECB data must not be modified, and NY Fed reference rates need its notice when shown to others</strong></td></tr>
<tr><td nowrap><strong>B</strong></td><td nowrap>⚠️ verify first</td><td nowrap>❌</td><td>FINRA / EIA spot prices</td><td>FINRA files (daily short volume, short interest) are published for download, but terms prohibit <em>&quot;data mining, scraping or harvesting tools (including robots)&quot;</em> and state <em>&quot;<strong>non-commercial use</strong>&quot;</em>. EIA's WTI / Brent / Henry Hub spot prices are sourced from <em>&quot;Refinitiv, an LSEG business&quot;</em>; reproducing third-party material beyond fair use needs the copyright owner's written permission (tier B is this tool's judgment)</td></tr>
<tr><td nowrap><strong>C</strong></td><td nowrap>❌ needs permission</td><td nowrap>❌</td><td>CBOE / Eastmoney / Sina / Tencent</td><td>CBOE (options chains and volatility indices) requires <em>&quot;<strong>approval in advance</strong>&quot;</em> + <em>&quot;<strong>execution of a license agreement</strong>&quot;</em>; Eastmoney / Sina / Tencent are website front-end endpoints, personal research only</td></tr>
<tr><td nowrap><strong>D</strong></td><td nowrap>❌</td><td nowrap>❌</td><td>Yahoo / Nasdaq</td><td>Terms prohibit automated access without permission. Yahoo: <em>&quot;using any automated means… for any purpose without our express, prior permission&quot;</em>. Nasdaq: <em>&quot;any process, whether automated or manual, to capture data… for any reason&quot;</em>. <strong>Personal study and research only, at your own risk; no commercial use or redistribution</strong></td></tr>
<tr><td nowrap><strong>⛔ Excluded</strong></td><td nowrap>—</td><td nowrap>—</td><td>HKEX (CCASS)</td><td>Terms prohibit <em>&quot;any '<strong>robot</strong>', '<strong>bot</strong>', '<strong>spider</strong>', '<strong>scraper</strong>'...&quot;</em> and apply <em>&quot;<strong>whether or not for gain</strong>&quot;</em> → <strong>this tool ships no scraper for it</strong></td></tr>
</tbody>
</table>

A working HKEX shareholding layer was built, then removed. Shipping code that violates a source's terms would defeat the point of grading sources in the first place. **This project distributes code, not data.** For commercial use, rely only on tier S.

**So why keep tier-D Yahoo and Nasdaq?** Honestly, their terms are the same kind as HKEX's. The difference is that they have no substitute: Yahoo is the only source for HK K-lines and for markets beyond the US and HK, and Nasdaq is the only key-free earnings calendar, while the HKEX shareholding layer was a nice-to-have. So the code stays and the terms are quoted in full (see "数据源合规分级" in SKILL.md). Whether to use them is your call under those terms.

---

## Quick start

**3 steps, 2 minutes.**

```bash
# 1. Create the skill directory
mkdir -p ~/.claude/skills/global-stock-data

# 2. Download SKILL.md
curl -o ~/.claude/skills/global-stock-data/SKILL.md \
  https://raw.githubusercontent.com/simonlin1212/global-stock-data/main/SKILL.md

# 3. Install the dependency
pip install requests
# Windows only: also install tzdata so Yahoo K-line dates use the exchange's time zone
pip install tzdata
```

⚠️ **Before using the SEC or US macro (BLS) layers, set `SEC_CONTACT` in SKILL.md to your real name and email.** SEC requires a declared User-Agent; requests without one are rejected as an *Undeclared Automated Tool*, and the BLS website returns 403. The code raises a clear error if you forget.

Launch Claude Code and say "check AAPL's financials" — the skill activates on its own. **Codex / OpenClaw users:** paste SKILL.md into your system prompt or project context file; the embedded Python runs as-is.

---

## Endpoints

**Price** — Sina / Tencent / Eastmoney quotes (25-78 fields) · Sina & Yahoo K-line (US back to 1984, daily→minute; Japan / Taiwan / Korea / India / Europe and more via ticker suffix) · MA/EMA/MACD/RSI/KDJ/Bollinger computed locally.

**Value** — Eastmoney three statements + GMAININDICATOR (US 49 / HK 75 fields) · Yahoo quoteSummary (23 modules: financials, analysts, institutional holdings) · SEC EDGAR XBRL (503 GAAP metrics).

**Flow & positioning ⭐** — CBOE options with full Greeks + IV + **0DTE** + unusual-flow detection · FINRA market-wide daily short volume (12,112 symbols) · Eastmoney daily fund flow · **V2.2:** CBOE VIX history since 1990 + term structure (contango or backwardation) + VVIX / SKEW · FINRA short interest (twice a month, 22,595 symbols, with days to cover and rankings).

**Filings & events ⭐** — EDGAR daily index (same-day **Form 4 insider / 8-K / 13F**, one day: 547 / 370 / 261) · full-text search since 2001 · 10-K/10-Q/8-K list · **V2.2:** Form 4 insider trades line by line (price, shares, holdings after, whether the filing ticks the 10b5-1 box) · 13F quarterly institutional holdings (sorted by value, with weights).

**Screener & macro ⭐** — EDGAR frames (any XBRL tag across the whole market, free — net income CY2025Q1 = 5,309 companies) · Treasury yield curve (1M~30Y) · CFTC COT · Nasdaq earnings calendar.

**Global rates & FX ⭐ (V2.1)** — policy rates for 38 central banks in one call (BIS) · ECB FX / three key rates / euro-area yield curve · NY Fed SOFR / EFFR · Fed H.10 USD vs 23 currencies / H.6 M1·M2 · Treasury auction results and schedule / federal debt / average interest rates.

**Taiwan ⭐ (V2.2)** — TWSE + TPEx official open APIs: the latest trading day's quotes, P/E / dividend yield / P/B, monthly revenue, company profiles, margin trading, ex-dividend notices and dividends for every listed and OTC stock (latest day only; for Taiwan K-line history use Yahoo `.TW` / `.TWO`).

**Energy ⭐ (V2.2)** — EIA weekly crude (commercial + SPR), gasoline, distillate and natural gas inventories · WTI (since 1986) / Brent / Henry Hub daily spot prices (tier B: third-party data republished by EIA).

**US macro ⭐ (V2.3)** — US Bureau of Labor Statistics: CPI / core CPI (seasonally adjusted and not), shelter, CPI-W, chained CPI, unemployment rate, U-6, participation rate, nonfarm payrolls, average hourly earnings, PPI and core PPI, JOLTS openings and quits rate, import prices, the Employment Cost Index, labor productivity, or any BLS series ID · the latest period's MoM / YoY change and next release date · the official BLS release schedule · Department of Labor weekly initial / continued jobless claims (since 1967). The key-free tier allows 25 requests a day; identical requests are reused for an hour.

**Tools** — stock search (CN+EN) · full-market list (US 5925+ / HK 18000+) · news by ticker · ticker↔CIK.

<details>
<summary>Full endpoint tables</summary>

<br>

### Options ⭐ (US only)

| Endpoint | Data |
|----------|------|
| **CBOE official** | Full chain + **IV + delta/gamma/vega/theta/rho**, plus **0DTE filtering** and **unusual-flow** detection (vol/OI > 1 = new positioning) |
| Yahoo (fallback) | Options chain, all expiries — **no Greeks** |

### Volatility indices ⭐ (V2.2, US only)

| Endpoint | Data |
|----------|------|
| CBOE `vol_index_history()` | Daily VIX (since 1990) / VIX1D / VIX9D / VIX3M / VIX6M / VIX1Y / VVIX / SKEW / OVX / GVZ and more |
| CBOE `vix_term_structure()` | Latest VIX at each tenor, whether the curve is in contango or backwardation, plus the long-short spread |

### Short volume ⭐ (US only)

| Endpoint | Data |
|----------|------|
| FINRA Reg SHO | Market-wide **daily short volume** (12,112 symbols), per-symbol series, ratio ranking |
| FINRA short interest `short_interest()` / `short_interest_all()` / `short_interest_ranking()` (V2.2) | **Open short positions** twice a month (22,595 symbols), days to cover, change vs prior period; rank by days to cover / shares short / change |

### Filing stream ⭐ (US only)

| Endpoint | Data |
|----------|------|
| EDGAR daily index | Same-day **Form 4 / 8-K / 13F / 144** — one day: 547 / 370 / 261 / 118 |
| EDGAR full-text | Search every filing body back to 2001 |
| EDGAR submissions / XBRL | Filing list + structured financials (503 GAAP metrics) |
| Form 4 parser `insider_trades()` (V2.2) | Insider trades line by line: trade date, transaction type, shares, price, value, holdings after, whether the filing ticks the 10b5-1 box; plus an open-market buy / sell summary |
| 13F parser `holdings_13f()` (V2.2) | Quarterly institutional holdings, sorted by value with weights, reconciled against the cover-page total; pre-2023 filings in thousands are converted to dollars |

### Market screener ⭐ (US only)

| Endpoint | Data |
|----------|------|
| EDGAR frames | One XBRL tag across the whole market in one call — a free screener |

### Macro / calendar ⭐

| Endpoint | Data |
|----------|------|
| Treasury / CFTC / Nasdaq | Yield curve (1M~30Y) · COT · earnings calendar (pre/post-market + EPS est.) |

### Global rates / FX / central banks ⭐ (V2.1)

| Endpoint | Data |
|----------|------|
| BIS `policy_rates()` | Policy rates for 48 economies (38 still updated), all in one call |
| ECB `ecb_fx()` / `ecb_key_rates()` / `ecb_yield_curve()` / `ecb_series()` | EUR reference rates · deposit facility and the other two key rates · euro-area government curve (AAA or all issuers) · any ECB series |
| NY Fed `nyfed_rates()` | SOFR / EFFR / OBFR / TGCR / BGCR with percentiles and volume; SOFR averages and index |
| Fed Board `fed_data()` | H.10 USD vs 23 currencies · H.6 M1/M2 and components |
| Treasury `fiscal_data()` | Auction results (high yield, bid-to-cover) and upcoming schedule · daily federal debt · average interest rates · Treasury reporting FX rates |

### Taiwan ⭐ (V2.2)

| Endpoint | Data |
|----------|------|
| `tw_stock()` | One Taiwan stock's latest quote + P/E / P/B / dividend yield; listed or OTC detected automatically |
| `tw_market()` | Listed (TWSE) or OTC (TPEx), whole market or one stock: quotes · valuation · monthly revenue · company profile · margin trading · ex-dividend notices · dividend resolutions (dividends: TWSE only) |

### Energy ⭐ (V2.2)

| Endpoint | Data |
|----------|------|
| EIA `energy_prices()` | WTI (since 1986) / Brent (since 1987) crude spot, USD per barrel; Henry Hub natural gas (since 1997). Tier B: prices come from Refinitiv, verify before commercial use or redistribution |
| EIA `energy_inventories()` | Weekly crude (commercial + SPR), gasoline, distillate and other stocks; Lower 48 and regional natural gas storage vs the 5-year average |

### US macro ⭐ (V2.3)

| Endpoint | Data |
|----------|------|
| BLS `us_macro()` | 23 common indicators in one request: CPI / core CPI (SA and NSA), shelter, CPI-W, chained CPI, unemployment rate, U-6, participation rate, employment-population ratio, nonfarm payrolls, average hourly earnings, average weekly hours, PPI and core PPI, JOLTS openings and quits rate, import prices, ECI 12-month change, labor productivity |
| BLS `us_macro_latest()` | Latest, previous and year-ago values per indicator, MoM / YoY computed locally, plus the next release date |
| BLS `bls_series()` | Any BLS series ID; requests over 25 series or 10 years are split automatically |
| BLS `bls_release_calendar()` | Official release schedule: date and Eastern time of every CPI / Employment Situation / PPI / JOLTS release and more |
| Dept. of Labor `us_jobless_claims()` | Weekly initial and continued claims (SA / NSA / 4-week average) and the insured unemployment rate, since 1967 |

### Price / value / flow / tools

| Endpoint | Data |
|----------|------|
| Sina / Tencent / Eastmoney push2 | US/HK real-time quotes, 25-78 fields |
| Sina / Yahoo chart | K-line, daily→minute, US back to 1984; Yahoo gives full daily history (S&P 500 since 1927) and markets beyond US/HK |
| Eastmoney datacenter / GMAININDICATOR | Statements + key metrics (bilingual) |
| Yahoo quoteSummary | 23 modules: financials / analysts / institutional holdings |
| Eastmoney push2his | Daily main/large/medium/small fund flow |
| Eastmoney search / push2 list / Yahoo search / SEC CIK | Search / full-market list / news / ticker↔CIK |

Every source is free, no API key. Yahoo crumb is auto-managed; SEC EDGAR and BLS only need a declared User-Agent (the key-free BLS tier allows 25 requests a day).

</details>

---

## Usage examples

Just tell your AI assistant:

| Scenario | Prompt |
|----------|--------|
| 0DTE options flow ⭐ | "Show NVDA's 0DTE options flow — unusual activity" |
| Short volume ⭐ | "TSLA's short volume ratio trend this week" |
| Short interest ⭐ | "GME's short interest and days to cover over the last few periods" |
| VIX term structure ⭐ | "Is the VIX term structure inverted right now?" |
| Insider filings ⭐ | "Any Form 4 insider filings today for my watchlist" |
| Insider trades ⭐ | "How many shares did NVDA insiders sell in the past six months?" |
| 13F holdings ⭐ | "Berkshire's top 10 holdings in its latest 13F" |
| Full-text filings ⭐ | "Which companies first mentioned 'HBM4' in an 8-K" |
| Market screener ⭐ | "Rank all filers by R&D expense for CY2025Q1" |
| Macro ⭐ | "What's the 10Y-2Y Treasury spread right now" |
| Central banks ⭐ | "Current policy rates of the major central banks" · "SOFR and EFFR over the past week" |
| FX / Treasuries ⭐ | "EUR/JPY this month" · "Next week's Treasury auction schedule" |
| Taiwan ⭐ | "TSMC's latest close and P/E" · "This month's revenue for OTC-listed stocks" |
| Oil / gas ⭐ | "This week's EIA crude inventories" · "WTI over the past year" |
| US macro ⭐ | "Latest CPI and core CPI year over year, and when's the next release?" · "Initial jobless claims over the last few weeks" |
| Other markets | "Toyota 7203.T daily K-line for the past year" · "TSMC 2330.TW full history" |
| US / HK quote | "AAPL's price and PE" · "How's Tencent 00700 today" |
| K-line | "Pull TSLA's daily K-line for the past 6 months" |
| Statements / valuation | "Apple's latest income statement" · "BABA's PE/PB/ROE + target" |
| Institutional holdings | "Which institutions hold NVDA, and what %" |
| Technical analysis | "AAPL's MACD and RSI — any golden cross?" |
| Batch compare | "Compare valuations of AAPL MSFT GOOGL" |

---

## Data sources

| Source | Tier | Auth | Coverage |
|--------|------|------|----------|
| **SEC EDGAR** | **S** | real UA | US filings / XBRL / **filing stream** / **full-text search** / **market screener** / **Form 4 insider trades** / **13F holdings** |
| **US Treasury** | **S** | none | **Yield curve (1M~30Y)** · FiscalData **auctions / federal debt / average rates** |
| **CFTC** | **S** | none | **COT reports** |
| **Federal Reserve Board** | **S** | none | **H.10 FX / H.6 money stock** (attribution) |
| **NY Fed** | **S** | none | **SOFR / EFFR / OBFR / TGCR / BGCR** (notice required when shown to others) |
| **BIS** | **S** | none | **Central-bank policy rates** (attribution) |
| **ECB** | **S** | none | **EUR FX / ECB key rates / euro-area yield curve** (attribution, no modification) |
| **US EIA** | **S** / B | none | **Weekly crude oil / natural gas inventories** (S, attribution recommended) · WTI / Brent / Henry Hub spot prices (B: Refinitiv data republished by EIA, verify before commercial use) |
| **US Department of Labor** (BLS + ETA) | **S** | real UA | **CPI / payrolls / unemployment / PPI / JOLTS / import prices / ECI / productivity** · **release schedule** · **weekly jobless claims** (cite source and retrieval date; key-free tier: 25 requests a day) |
| **Taiwan Stock Exchange** | **S** | none | Listed stocks' **latest quotes / P/E / monthly revenue / margin / ex-dividend / dividends** (attribution) |
| **TPEx** | **S** | none | OTC stocks' **latest quotes / P/E / monthly revenue / margin / ex-dividend** (attribution; S is this tool's judgment) |
| **FINRA** | **B** | none | US **market-wide daily short volume** · **short interest** (verify before commercial use) |
| **CBOE** | **C** | none | US **options + Greeks + IV + 0DTE** · **VIX and other volatility indices** (needs Cboe's prior approval) |
| Eastmoney (push2 / push2his / datacenter / search) | C | none | US+HK quotes / fund flow / statements / search |
| Sina | C | none | US+HK quotes, US K-line |
| Tencent | C | none | US+HK quotes |
| Yahoo Finance | D | cookie+crumb (auto) | US+HK+other markets, all categories (terms prohibit automated access) |
| Nasdaq | D | none | US **earnings calendar** (terms prohibit automated access) |

**Tiers:** **S** = government and central-bank data (including exchange data registered as Taiwan government open data), commercial + redistribution OK (some require attribution) · **B** = published data, verify before commercial use · **C** = needs prior approval or terms unverified, personal research only · **D** = terms prohibit automated access without permission; personal study and research only, at your own risk, no commercial use or redistribution. See [Compliance tiers](#compliance-tiers) for the quoted basis. All requests go through direct HTTP with a built-in thread-safe rate limiter (SEC capped at 8 req/s against the official 10).

---

## FAQ

**Q: How does this relate to a-stock-data?**
Sister project. [a-stock-data](https://github.com/simonlin1212/a-stock-data) covers China A-shares; global-stock-data covers US and HK. Install both — they don't conflict.

**Q: Does Yahoo Finance need an API key?**
No. The code fetches cookie + crumb automatically and refreshes on expiry.

**Q: Any limits on SEC EDGAR?**
Yes — a declared User-Agent and 10 req/s. Set `SEC_CONTACT` to your real email; the built-in throttle stays under the limit.

**Q: HK options data?**
No. HK options aren't in Yahoo's coverage and need HKEX's paid feed. The options layer is US only.

**Q: Can I look up stocks outside the US and HK?**
K-lines and Yahoo fundamentals, yes: Japan `.T`, Taiwan `.TW`, Korea `.KS`, India `.NS`, London `.L`, Germany `.DE` and more — see "Yahoo Finance 代码格式" in SKILL.md. Note that Yahoo is a tier-D source. Taiwan also has the official TWSE / TPEx APIs (tier S, latest trading day only): `tw_stock("2330")`. For China A-shares, use the sister project a-stock-data.

**Q: How fresh are the global rates and FX?**
ECB and NY Fed update every business day; Fed H.10 is published each Monday for the prior week; BIS lags by a few days to about three weeks depending on the country. Every row carries its date — check the central bank itself when you need today's exact value.

**Q: How often is short interest updated?**
FINRA publishes it twice a month, for settlement dates on the 15th and at month-end (moved earlier for holidays), about 8 business days after settlement. The daily dataset is a different one — daily short volume (9.1), which counts that day's short-sale trades, not open short positions.

**Q: Are there limits on the US macro data? Does it get revised?**
The registration-free BLS API allows 25 requests a day, up to 25 series and 10 years per request; `us_macro()` gets all 23 indicators in one request, and identical requests are reused for an hour. Payrolls, PPI, JOLTS and others are revised after release, and the API returns the latest revised values, not what was first published. When replaying history, line data up with the real release dates from `bls_release_calendar()`. Each indicator's revision rules are in section 16.5 of SKILL.md.

**Q: Why no GDP?**
GDP comes from the Bureau of Economic Analysis, whose API needs a registered key. This project does not use sources that require an account or a key.

**Q: Running from a server in mainland China — can it reach Yahoo / SEC?**
They're overseas and may be flaky on a direct connection. Use a proxy, or lean on the Eastmoney / Sina / Tencent sources.

**Q: Can I use it without Claude Code?**
Yes. SKILL.md is Markdown + embedded Python — Codex, OpenClaw, or any AI coding assistant can read it, and you can copy the Python into your own scripts.

---

## Changelog

See [CHANGELOG.md](./CHANGELOG.md).

## Disclaimer

This project provides data-access tools only. It is not investment advice. Investing involves risk.

## License

[Apache License 2.0](./LICENSE) — **Author:** Simon Lin · X [@linsizhen](https://x.com/linsizhen) · Email: [simonlin0423@gmail.com](mailto:simonlin0423@gmail.com)
