# global-stock-data

美股港股全栈数据工具包 — 8 层架构 · 18 个端点 · 5 个数据源 · 全部零鉴权 · 仅依赖 requests

一个自包含的 Skill 文件，把分散在 5 个数据源里的美股/港股原始数据整合成 AI 编程助手直接能用的工具集。你不用再背东财 secid 前缀、Yahoo crumb 鉴权流程、SEC EDGAR 的 CIK 映射——全部封装好了。

> **V1.0.1 修复（2026-06-20 · PR #1）：** 5 个函数（个股行情/Yahoo K线/日级资金流/搜索/全市场列表）漏传 `params=params`，请求实际是裸 URL，**此前始终返回空数据** → 一次性补齐；并修了 `market_stock_list` 在东财 `diff` 返回 dict 时 `AttributeError` 的连带 bug（归一化 dict→list）。强烈建议升级。致谢 @APTX4869-maker。

> 兼容 [Claude Code](https://github.com/anthropics/claude-code) · [Codex](https://github.com/openai/codex) · [OpenClaw](https://github.com/anthropics/openclaw)
>
> Skill 文件本质是结构化 Markdown + 内嵌 Python，任何支持上下文注入的 AI 编程助手都能用。

---

## 架构

```
美股港股全栈数据 · 八层架构 · V1.0.1
│
├── 行情层      新浪(gb_/rt_hk) + 腾讯(us/r_hk) + 东财push2        实时报价 25-78 字段
├── K线层      新浪(回溯至1984) + Yahoo chart                       日/周/月/分钟 K线
├── 技术指标    MA/EMA + MACD + RSI + KDJ + 布林带                  纯Python计算，零额外依赖
├── 基本面      东财datacenter三表+GMAININDICATOR + Yahoo + SEC XBRL 财报+关键指标+估值+机构持仓
├── 资金面      东财push2his                                        日级主力/大单/中单/小单资金流
├── 期权层      Yahoo crumb                                         期权链 calls+puts (仅美股)
├── SEC Filing  EDGAR submissions + XBRL                            10-K/10-Q/8-K + 503个GAAP指标 (仅美股)
└── 工具层      东财search+push2列表 + Yahoo search + SEC CIK映射   搜索+全市场列表+新闻+ticker↔CIK
```

---

## 快速开始

**3 步，2 分钟。**

```bash
# 1. 创建 skill 目录
mkdir -p ~/.claude/skills/global-stock-data

# 2. 把 SKILL.md 放进去
curl -o ~/.claude/skills/global-stock-data/SKILL.md \
  https://raw.githubusercontent.com/simonlin1212/global-stock-data/main/SKILL.md

# 3. 安装依赖
pip install requests
```

启动 Claude Code，说一句「帮我看看 AAPL 的财报」，自动激活。

> **Codex / OpenClaw 用户：** 把 SKILL.md 的内容贴入你的系统 prompt 或项目上下文文件即可，内嵌的 Python 代码可直接执行。

---

## 18 个端点能力清单

### 行情层（实时/延时）

| 端点 | 数据 |
|------|------|
| 新浪财经 | 美股 36 字段（含中文名/EPS/PE）/ 港股 25 字段 |
| 腾讯财经 | 美股 71 字段 / 港股 78 字段（含 PE/PB/市值/换手率） |
| 东财 push2 | 美股/港股 secid 统一查询，含中文名/涨跌幅/换手率 |

### K线层（日/周/月/分钟）

| 端点 | 数据 |
|------|------|
| 新浪 | 美股日K线，回溯至 1984 年 |
| Yahoo chart | 美股 + 港股，v8 API 零 crumb，支持日/周/月/分钟 |

### 技术指标层（纯计算）

| 端点 | 数据 |
|------|------|
| 技术指标计算 | MA/EMA + MACD(DIF/DEA/柱状图) + RSI(6/12/24) + KDJ(K/D/J) + 布林带(上/中/下轨)，基于K线纯Python计算 |

### 基本面

| 端点 | 数据 |
|------|------|
| 东财 datacenter 三表 | 美股/港股三表（资产负债 + 利润 + 现金流），中文科目名 |
| 东财 GMAININDICATOR | 关键财务指标概览（美股49字段/港股75字段：ROE/ROA/EPS/毛利率/资产负债率） |
| Yahoo quoteSummary | 23 个模块（财务数据 + 关键指标 + 分析师 + 机构持仓） |
| SEC EDGAR XBRL | 美股 503 个 GAAP 指标（仅美股） |

### 资金面

| 端点 | 数据 |
|------|------|
| 东财 push2his | 日级主力/大单/中单/小单净流入，美股 + 港股 |

### 期权层（仅美股）

| 端点 | 数据 |
|------|------|
| Yahoo options | 期权链 calls + puts，所有到期日，含 Greeks |

### SEC Filing 层（仅美股）

| 端点 | 数据 |
|------|------|
| EDGAR submissions | 10-K/10-Q/8-K 完整 Filing 列表 |
| EDGAR XBRL | 结构化财务指标（营收/净利/EPS 等） |

### 工具层

| 端点 | 数据 |
|------|------|
| 东财 search | 股票搜索（中英文，含市场代码映射） |
| 东财 push2 列表 | 全市场股票列表（涨跌幅/成交量排名，美股5925+/港股18000+） |
| Yahoo search | 新闻资讯（按股票代码） |
| SEC CIK mapping | ticker ↔ CIK 映射（仅美股） |

### 鉴权要求

全部 5 个数据源**完全免费无 Key**。Yahoo crumb 由代码自动获取，SEC EDGAR 仅需标准 User-Agent。

---

## 使用示例

跟你的 AI 助手说这些话就能激活：

| 场景 | 说什么 |
|------|--------|
| 美股行情 | 「AAPL 现在什么价，PE 多少」 |
| 港股行情 | 「腾讯 00700 今天行情怎么样」 |
| K线分析 | 「拉一下 TSLA 最近半年日K线」 |
| 财报解读 | 「看看苹果最新一季的利润表」 |
| 估值分析 | 「BABA 的 PE/PB/ROE 和分析师目标价」 |
| 机构持仓 | 「哪些机构持有 NVDA，持股比例多少」 |
| 资金流向 | 「AAPL 最近资金是流入还是流出」 |
| 期权策略 | 「TSLA 下个月到期的期权链，看看 call 和 put」 |
| SEC Filing | 「苹果最近的 10-K 年报什么时候发的」 |
| 量化分析 | 「从 SEC XBRL 拉 MSFT 近 5 年营收和净利趋势」 |
| 搜索股票 | 「搜一下阿里巴巴的股票代码」 |
| 新闻 | 「NVDA 最近有什么新闻」 |
| 涨幅排名 | 「今天美股涨幅最大的 20 只股票」 |
| 全市场筛选 | 「遍历港股全市场，找出换手率最高的」 |
| 关键指标(中文) | 「看看苹果最近几季的 ROE、EPS 和资产负债率」 |
| 技术分析 | 「AAPL 的 MACD 和 RSI 怎么样，有没有金叉」 |
| 批量对比 | 「帮我对比 AAPL MSFT GOOGL 三家的估值」 |

---

## V1.0 亮点

| 特性 | 说明 |
|------|------|
| **全部零鉴权** | 5 个数据源全部免费无 Key，Yahoo crumb 自动管理 |
| **极简依赖** | 仅需 `requests`，零第三方数据封装 |
| **美股 + 港股双覆盖** | 行情/K线/财报/资金流均支持双市场 |
| **技术指标内置** | MA/EMA/MACD/RSI/KDJ/布林带，纯 Python 计算，拉完 K 线直接算，零额外依赖 |
| **全市场列表** | 东财 push2 一键获取美股 5925+/港股 18000+ 只股票，按涨跌幅/成交量排序 |
| **关键指标中英双版** | 东财 GMAININDICATOR（中文 49/75 字段）+ Yahoo quoteSummary（英文全品类） |
| **SEC 深度集成** | EDGAR Filing 列表 + XBRL 503 个 GAAP 指标，量化分析利器 |
| **期权链** | Yahoo 期权数据，含所有到期日和 Greeks |
| **智能代码映射** | 东财 secid 前缀自动判断（105/106/107/116），Yahoo `.HK` 后缀自动处理 |

---

## 数据源优先级

| 场景 | 第一优先 | 备选 | 说明 |
|------|---------|------|------|
| 美股行情 | 新浪 `gb_XXXX` | 腾讯 / 东财 push2 | 新浪有中文名+EPS+PE |
| 港股行情 | 腾讯 `r_hkXXXXX` | 新浪 / 东财 push2 | 腾讯字段最全(78个) |
| 美股K线 | 新浪 | Yahoo chart | 新浪回溯至1984年；Yahoo支持多周期 |
| 港股K线 | Yahoo chart | — | 新浪港股K线已失效 |
| 财报三表(中文) | 东财 datacenter | — | 中文科目名，按行展开 |
| 财报三表(结构化) | Yahoo quoteSummary | — | 英文，完整报表结构 |
| 关键指标(中文) | 东财 GMAININDICATOR | — | ROE/ROA/EPS/毛利率/资产负债率 |
| 关键指标(英文) | Yahoo quoteSummary | — | PE/PB/EV/利润率/目标价 |
| 分析师预期 | Yahoo quoteSummary | — | EPS预测+评级+升降级 |
| 机构持仓 | Yahoo quoteSummary | — | 前10大机构+内部人 |
| 资金流 | 东财 push2his | — | 日级主力/大单/中单/小单 |
| 期权链 | Yahoo options | — | 仅美股 |
| SEC Filing | EDGAR | — | 官方数据，仅美股 |
| 搜索 | 东财 search | Yahoo search | 东财有 secid 映射 |
| 新闻 | Yahoo search | — | 唯一稳定的新闻源 |
| 全市场列表 | 东财 push2 clist | — | 涨跌幅/成交量排名 |

---

## 数据源汇总

| 数据源 | 协议 | 鉴权 | 覆盖 |
|--------|------|------|------|
| 东财 push2 | HTTPS | 零 | 美股+港股 实时行情+全市场列表 |
| 东财 push2his | HTTPS | 零 | 美股+港股 资金流 |
| 东财 datacenter | HTTPS | 零 | 美股+港股 财报三表+GMAININDICATOR关键指标 |
| 东财 search API | HTTPS | 零 | 全球股票搜索+secid映射 |
| Yahoo Finance | HTTPS | cookie+crumb(自动) | 美股+港股 全品类 |
| 新浪财经 | HTTP | 零 | 美股+港股 行情、美股K线 |
| 腾讯财经 | HTTPS | 零 | 美股+港股 行情 |
| SEC EDGAR | HTTPS | 零(需UA) | 美股 Filing+XBRL |

> **架构原则：** 全部直连 HTTP API，零第三方数据封装依赖。Yahoo crumb 由 helper 自动管理，SEC EDGAR 仅需标准 User-Agent。

---

## FAQ

**Q: 和 a-stock-data 有什么关系？**
姊妹项目。a-stock-data 覆盖 A 股（沪深北），global-stock-data 覆盖美股和港股。两个 Skill 可以同时安装，互不冲突。

**Q: Yahoo Finance 需要 API Key 吗？**
不需要。代码自动获取 cookie + crumb，透明处理。如果 crumb 过期会自动刷新。

**Q: SEC EDGAR 有访问限制吗？**
有。SEC 要求请求携带 User-Agent 并限制每秒 10 次。代码已内置合规 UA，正常使用不会触发限流。

**Q: 港股期权数据有吗？**
没有。港股期权不在 Yahoo Finance 覆盖范围，需要港交所专有接口（付费）。当前期权层仅支持美股。

**Q: 在国内服务器跑，Yahoo/SEC 能访问吗？**
Yahoo Finance 和 SEC EDGAR 都是境外服务，国内直连可能不稳定。建议走代理，或优先使用东财/新浪/腾讯数据源。

**Q: 不用 Claude Code，能用吗？**
能。SKILL.md 本质是 Markdown + 内嵌 Python 代码。Codex、OpenClaw 或任何 AI 编程助手都能读取。你也可以直接把 Python 代码段复制出来在自己的脚本里跑。

---

## 更新日志

见 [CHANGELOG.md](./CHANGELOG.md)。

---

## Donate

如果这个工具帮到了你的投研工作流，欢迎请作者喝杯咖啡 ☕

<p align="center">
  <img src="./assets/wechat-sponsor.jpg" width="240" alt="微信赞赏码">
</p>
<p align="center">
  <a href="https://ifdian.net/a/simonlin">爱发电</a> ·
  <a href="https://buymeacoffee.com/simonlin1212">Buy Me a Coffee</a>
</p>

> 想要什么数据端点？欢迎开 [Issue](https://github.com/simonlin1212/global-stock-data/issues) 提需求，赞助者的 Issue 优先处理。

---

## Disclaimer

本项目仅提供数据获取工具，不构成任何投资建议。股市有风险，投资需谨慎。

---

## License

[Apache License 2.0](./LICENSE) — 自由使用，注明出处即可。

**作者：** Simon 林 · 抖音「Simon林」 · 公众号「硅基世纪」

---

<details>
<summary><b>🇬🇧 English</b></summary>

# global-stock-data

Full-stack data toolkit for US & HK stock markets — 8-layer architecture · 18 endpoints · 5 data sources · zero API keys · only depends on requests

A self-contained Skill file that consolidates raw US/HK stock data from 5 sources into a ready-to-use toolkit for AI coding assistants. No need to memorize Eastmoney secid prefixes, Yahoo crumb authentication flows, or SEC EDGAR CIK mappings — it's all handled.

> **V1.0.1 Fix (2026-06-20 · PR #1):** 5 functions (quote / Yahoo kline / daily fund flow / search / market list) were missing `params=params`, so requests hit a bare URL and **always returned empty data** → fixed all 5; also fixed a follow-on `AttributeError` in `market_stock_list` when Eastmoney's `diff` comes back as a dict (normalized dict→list). Upgrade strongly recommended. Thanks to @APTX4869-maker.

> Compatible with [Claude Code](https://github.com/anthropics/claude-code) · [Codex](https://github.com/openai/codex) · [OpenClaw](https://github.com/anthropics/openclaw)
>
> The Skill file is structured Markdown + embedded Python. Any AI coding assistant with context injection can use it.

---

## Architecture

```
US & HK Stock Full-Stack Data · 8-Layer Architecture · V1.0.1
│
├── Market Data       Sina(gb_/rt_hk) + Tencent(us/r_hk) + Eastmoney push2     Real-time quotes 25-78 fields
├── K-line            Sina(back to 1984) + Yahoo chart                          Daily/Weekly/Monthly/Minute
├── Technical Ind.    MA/EMA + MACD + RSI + KDJ + Bollinger Bands              Pure Python, zero extra deps
├── Fundamentals      Eastmoney datacenter+GMAININDICATOR + Yahoo + SEC XBRL   Statements+Key Metrics+Valuation+Holdings
├── Fund Flow         Eastmoney push2his                                        Daily main/large/medium/small order flow
├── Options           Yahoo crumb                                               Options chain calls+puts (US only)
├── SEC Filing        EDGAR submissions + XBRL                                  10-K/10-Q/8-K + 503 GAAP metrics (US only)
└── Tools             Eastmoney search+push2 list + Yahoo search + SEC CIK      Search+Market List+News+ticker↔CIK
```

---

## Quick Start

**3 steps, 2 minutes.**

```bash
# 1. Create skill directory
mkdir -p ~/.claude/skills/global-stock-data

# 2. Download SKILL.md
curl -o ~/.claude/skills/global-stock-data/SKILL.md \
  https://raw.githubusercontent.com/simonlin1212/global-stock-data/main/SKILL.md

# 3. Install dependencies
pip install requests
```

Launch Claude Code and say "Check AAPL's financials" — the skill activates automatically.

> **Codex / OpenClaw users:** Paste the contents of SKILL.md into your system prompt or project context file. The embedded Python code is ready to execute.

---

## 18 Endpoints

### Market Data (real-time / delayed)

| Endpoint | Data |
|----------|------|
| Sina Finance | US stocks 36 fields (incl. Chinese name/EPS/PE) / HK stocks 25 fields |
| Tencent Finance | US stocks 71 fields / HK stocks 78 fields (incl. PE/PB/Market Cap/Turnover) |
| Eastmoney push2 | US/HK real-time quotes via secid, incl. Chinese name/change%/turnover |

### K-line (Daily/Weekly/Monthly/Minute)

| Endpoint | Data |
|----------|------|
| Sina | US daily K-line, back to 1984 |
| Yahoo chart | US + HK, v8 API, zero crumb needed, daily/weekly/monthly/minute |

### Technical Indicators (Pure Calculation)

| Endpoint | Data |
|----------|------|
| Technical Indicators | MA/EMA + MACD(DIF/DEA/Histogram) + RSI(6/12/24) + KDJ(K/D/J) + Bollinger Bands, pure Python on K-line data |

### Fundamentals

| Endpoint | Data |
|----------|------|
| Eastmoney datacenter | US/HK three statements (Balance Sheet + Income + Cash Flow), Chinese labels |
| Eastmoney GMAININDICATOR | Key financial indicators overview (US 49 fields / HK 75 fields: ROE/ROA/EPS/margins) |
| Yahoo quoteSummary | 23 modules (Financials + Key Stats + Analysts + Institutional Holdings) |
| SEC EDGAR XBRL | 503 GAAP metrics (US only) |

### Fund Flow

| Endpoint | Data |
|----------|------|
| Eastmoney push2his | Daily main/large/medium/small order net inflow, US + HK |

### Options (US only)

| Endpoint | Data |
|----------|------|
| Yahoo options | Options chain calls + puts, all expiration dates, with Greeks |

### SEC Filing (US only)

| Endpoint | Data |
|----------|------|
| EDGAR submissions | 10-K/10-Q/8-K full filing list |
| EDGAR XBRL | Structured financial metrics (Revenue/Net Income/EPS etc.) |

### Tools

| Endpoint | Data |
|----------|------|
| Eastmoney search | Stock search (Chinese + English, with market code mapping) |
| Eastmoney push2 list | Full market stock list (sort by change%/volume, US 5925+ / HK 18000+) |
| Yahoo search | News by stock ticker |
| SEC CIK mapping | ticker ↔ CIK mapping (US only) |

### Authentication

All 5 data sources are **completely free, no API key needed**. Yahoo crumb is auto-managed. SEC EDGAR only requires a standard User-Agent.

---

## Usage Examples

Just tell your AI assistant:

| Scenario | Prompt |
|----------|--------|
| US Stock Quote | "What's AAPL's price and PE ratio" |
| HK Stock Quote | "How's Tencent 00700 doing today" |
| K-line Analysis | "Pull TSLA's daily K-line for the past 6 months" |
| Financial Statements | "Show Apple's latest quarterly income statement" |
| Valuation | "BABA's PE/PB/ROE and analyst target price" |
| Institutional Holdings | "Which institutions hold NVDA and their percentages" |
| Fund Flow | "Is money flowing into or out of AAPL recently" |
| Options | "TSLA options chain expiring next month, calls and puts" |
| SEC Filing | "When was Apple's latest 10-K annual report filed" |
| Quantitative Analysis | "Pull MSFT's 5-year revenue and net income trend from SEC XBRL" |
| Stock Search | "Search for Alibaba's stock ticker" |
| News | "What's the latest news on NVDA" |
| Top Gainers | "Top 20 US stocks by gain today" |
| Market Screening | "Scan all HK stocks for highest turnover" |
| Key Indicators (CN) | "Show Apple's ROE, EPS and debt ratio for recent quarters" |
| Technical Analysis | "What's AAPL's MACD and RSI, any golden cross?" |
| Batch Compare | "Compare valuations of AAPL MSFT GOOGL" |

---

## Data Source Priority

| Scenario | Primary | Fallback | Notes |
|----------|---------|----------|-------|
| US Quotes | Sina `gb_XXXX` | Tencent / Eastmoney push2 | Sina has Chinese name+EPS+PE |
| HK Quotes | Tencent `r_hkXXXXX` | Sina / Eastmoney push2 | Tencent has most fields (78) |
| US K-line | Sina | Yahoo chart | Sina goes back to 1984; Yahoo supports multi-period |
| HK K-line | Yahoo chart | — | Sina HK K-line is down |
| Statements (CN) | Eastmoney datacenter | — | Chinese labels, row-expanded |
| Statements (structured) | Yahoo quoteSummary | — | English, full report structure |
| Key Indicators (CN) | Eastmoney GMAININDICATOR | — | ROE/ROA/EPS/margins/debt ratio |
| Key Stats (EN) | Yahoo quoteSummary | — | PE/PB/EV/Margins/Target Price |
| Analyst Estimates | Yahoo quoteSummary | — | EPS forecast + ratings |
| Institutional Holdings | Yahoo quoteSummary | — | Top 10 institutions + insiders |
| Fund Flow | Eastmoney push2his | — | Daily main/large/medium/small |
| Options | Yahoo options | — | US only |
| SEC Filing | EDGAR | — | Official data, US only |
| Search | Eastmoney search | Yahoo search | Eastmoney has secid mapping |
| News | Yahoo search | — | Only stable news source |
| Market List | Eastmoney push2 clist | — | Sort by change%/volume |

---

## Data Sources

| Source | Protocol | Auth | Coverage |
|--------|----------|------|----------|
| Eastmoney push2 | HTTPS | None | US+HK Real-time Quotes + Market List |
| Eastmoney push2his | HTTPS | None | US+HK Fund Flow |
| Eastmoney datacenter | HTTPS | None | US+HK Financial Statements + GMAININDICATOR |
| Eastmoney search API | HTTPS | None | Global Stock Search + secid mapping |
| Yahoo Finance | HTTPS | cookie+crumb (auto) | US+HK All Categories |
| Sina Finance | HTTP | None | US+HK Quotes, US K-line |
| Tencent Finance | HTTPS | None | US+HK Quotes |
| SEC EDGAR | HTTPS | None (UA required) | US Filings + XBRL |

> **Architecture:** All sources use direct HTTP API calls. Zero third-party data wrapper dependencies. Yahoo crumb managed automatically. SEC EDGAR requires standard User-Agent only.

---

## Disclaimer

This project provides data access tools only and does not constitute investment advice. Investing involves risk.

---

## License

[Apache License 2.0](./LICENSE)

**Author:** Simon Lin · TikTok [@simonlin121212](https://www.tiktok.com/@simonlin121212) · Douyin "Simon林" · WeChat Official Account "硅基世纪"

</details>
