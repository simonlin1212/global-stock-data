<p align="center"><b>简体中文</b> | <a href="README.md">English</a></p>

<h1 align="center">global-stock-data</h1>

<p align="center">
  <b>给 AI 编程助手用的美股数据工具包</b><br>
  期权希腊字母 · 0DTE 流 · VIX 期限结构 · SEC 申报与内部人交易 · 做空数据 · 基本面 · 全市场筛选 · 全球利率汇率 · 台股 · 能源 · 美国宏观
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License: Apache 2.0"></a>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white" alt="Python 3.9+"></a>
  <a href="https://github.com/simonlin1212/global-stock-data/stargazers"><img src="https://img.shields.io/github/stars/simonlin1212/global-stock-data?style=social" alt="GitHub stars"></a>
  <a href="#架构"><img src="https://img.shields.io/badge/层数-17-2ea44f.svg" alt="层数"></a>
  <a href="#端点"><img src="https://img.shields.io/badge/端点-50%2B-2ea44f.svg" alt="端点"></a>
  <a href="#数据源"><img src="https://img.shields.io/badge/数据源-18-2ea44f.svg" alt="数据源"></a>
  <a href="#数据源"><img src="https://img.shields.io/badge/鉴权-零-success.svg" alt="零鉴权"></a>
</p>

<p align="center">
  <a href="#架构">架构</a> ·
  <a href="#合规分级">合规分级</a> ·
  <a href="#快速开始">快速开始</a> ·
  <a href="#端点">端点</a> ·
  <a href="#数据源">数据源</a> ·
  <a href="#faq">FAQ</a>
</p>

> **给 AI 编程助手用的美股全栈数据工具包** — 17 层架构 · 50+ 个端点 · 18 个数据源 · 全部零鉴权 · 仅依赖 `requests`。
>
> **官方源优先（V2.0）：** CBOE 官方期权链（完整希腊字母 + **0DTE** 异动 flow）、FINRA 全市场每日空头成交量、SEC EDGAR 申报事件流 + 免费全市场 screener、美债收益率曲线 / CFTC COT / 财报日历。**每个数据源标注了合规级别与条款原文**——因为"官方"不等于"可自由使用"。
>
> **V2.1 新增：** 全球利率汇率层——BIS 38 家央行政策利率、欧洲央行汇率 / 利率 / 欧元区收益率曲线、纽约联储 SOFR / EFFR、美联储 H.10 汇率 / H.6 M2、美国财政部国债拍卖与联邦债务，全部官方 S 级。Yahoo K 线修复全历史日线与时区问题，周 / 月 / 季线改由日线合并，并可直接取日本 / 台湾 / 韩国 / 印度 / 欧洲等市场。
>
> **V2.2 新增：** CBOE 波动率指数（VIX 1990 年起）与 VIX 期限结构、FINRA 做空余额（每月两期，含回补天数）、SEC Form 4 内部人逐笔买卖与 13F 机构持仓明细、台湾证交所 / 柜买中心官方开放接口（台股最新行情、本益比、月营收，可商用）、美国能源信息署 EIA 原油 / 天然气每周库存与现货价格（现货价为 B 级，见合规分级）。
>
> **V2.3 新增：** 美国宏观层——美国劳工统计局 BLS 免注册的公共接口：CPI / 核心 CPI、非农、失业率、平均时薪、PPI、JOLTS 职位空缺、进口价格、雇佣成本指数、劳动生产率等 23 个常用指标一次取齐，本地算环比 / 同比，附官方发布日程和每个指标的修订规则；另有劳工部每周初请 / 续请失业金（1967 年起）。全部 S 级，须注明来源与取数日期。

一个自包含的 Skill 文件，把分散在多个数据源里的美股/港股原始数据，整合成 AI 编程助手直接能用的工具集。你不用再背东财 secid 前缀、Yahoo crumb 鉴权流程、SEC EDGAR 的 CIK 映射——全部封装好了。

> *本工具只分发**代码，不分发市场数据**。数据由使用者自行按各源条款获取。商用请只依赖 S 级源。*

> 兼容 [Claude Code](https://github.com/anthropics/claude-code) · [Codex](https://github.com/openai/codex) · [OpenClaw](https://github.com/openclaw/openclaw)。Skill 文件本质是结构化 Markdown + 内嵌 Python，任何支持上下文注入的 AI 编程助手都能用。

---


## 架构

```
美股全栈数据 · 17 层架构 · V2.3
│
├── 行情层      新浪 + 腾讯 + 东财push2        实时报价 25-78 字段
├── K线层      新浪(回溯至1984) + Yahoo         日/周/月/分钟；日/台/韩/印/欧等市场换后缀
├── 技术指标    MA/EMA + MACD + RSI + KDJ + 布林  纯Python，零额外依赖
├── 基本面      东财三表+GMAININDICATOR + Yahoo + SEC XBRL
├── 资金面      东财push2his                    日级主力/大单/中单/小单
├── 期权层      Yahoo crumb                     期权链（仅美股）
├── SEC Filing  EDGAR submissions + XBRL        10-K/10-Q/8-K + 503个GAAP指标
└── 工具层      东财search+列表 + Yahoo + SEC CIK

━━━ V2.0 新增（官方源优先）━━━
├── 期权·CBOE   cdn.cboe.com 官方延时   ⭐ 全链+IV+完整希腊字母+0DTE+异动flow
├── 做空层      FINRA Reg SHO           ⭐ 全市场每日空头成交量(实测12,112只)+时序
├── 申报事件流  EDGAR 每日索引+全文检索 ⭐ Form4内部人/8-K/13F当日 + 2001至今检索
├── 全市场横截面 EDGAR frames           ⭐ 任意XBRL标签一次拿全市场(1,842~5,309家)=免费screener
└── 宏观/日历   Treasury + CFTC + Nasdaq ⭐ 收益率曲线 + COT + 财报日历

━━━ V2.1 新增 ━━━
└── 全球利率汇率 BIS + 欧洲央行 + 纽约联储 + 美联储 + 财政部 ⭐ 38家央行政策利率 + 欧元/美元汇率 + SOFR/EFFR + 欧元区收益率曲线 + 国债拍卖

━━━ V2.2 新增 ━━━
├── 波动率      CBOE 指数文件             ⭐ VIX 1990起 + VIX9D/3M/6M/1Y 期限结构 + VVIX/SKEW/OVX/GVZ
├── 做空余额    FINRA 每月两期            ⭐ 全美未平仓空头(实测22,595只) + 回补天数 + 排行
├── 内部人/13F  SEC Form 4 + 13F          ⭐ 内部人逐笔买卖(价格/股数/申报是否勾10b5-1) + 机构季度持仓明细
├── 台股        证交所 + 柜买中心 OpenAPI  ⭐ 上市/上柜全市场最新行情 + 本益比 + 月营收 + 融资融券（可商用）
└── 能源        美国能源信息署 EIA         ⭐ 原油/天然气每周库存 + WTI(1986起)/布伦特/Henry Hub 现货日价(B级)

━━━ V2.3 新增 ━━━
└── 美国宏观    美国劳工部 BLS + ETA       ⭐ CPI/核心CPI/非农/失业率/时薪/PPI/JOLTS 等 23 个指标 + 官方发布日程 + 每周初请/续请失业金
```

> 层数按「原有八层 + V2.0 五层（CBOE 期权链单算一层）+ V2.1 一层 + V2.2 两层（台股、能源）+ V2.3 一层（美国宏观）」计；V2.2 的波动率指数、做空余额、Form 4 / 13F 解析是原有层里的新小节，不另计层。

---

## 合规分级

各源条款差异极大，**"官方"不等于"可自由使用"**。以下结论来自逐家实读条款原文（2026-07-24；2026-10-11 复核 Yahoo / Nasdaq 并新读 4 家官方源；2026-10-12 新读台湾证交所 / 柜买中心与 EIA；2026-11-02 新读美国劳工统计局 BLS 与劳工部），引号内为原文：

<table>
<thead>
<tr><th nowrap>级别</th><th nowrap>可商用</th><th nowrap>可再分发</th><th>源</th><th>依据（原文摘录）</th></tr>
</thead>
<tbody>
<tr><td nowrap><strong>S</strong></td><td nowrap>✅</td><td nowrap>✅</td><td>SEC EDGAR / Treasury / CFTC / 美联储 / 纽约联储 / BIS / 欧洲央行 / EIA（库存） / 台湾证交所 / 柜买中心 / 美国劳工部（BLS / ETA）</td><td>EDGAR 明示 <em>&quot;for free&quot;</em>、<em>&quot;allow scripted access&quot;</em>，<strong>硬上限 10 请求/秒</strong>，须声明 User-Agent；美联储 <em>&quot;public domain&quot;</em>；纽约联储允许 <em>&quot;personal or business purposes&quot;</em>、<em>&quot;through an automated process&quot;</em>；BIS <em>&quot;unrestricted&quot;</em>；ECB <em>&quot;reused free of charge&quot;</em>；EIA 自己的数据 <em>&quot;public domain&quot;</em>；台湾两家用到的 13 个数据集都登记在政府资料开放平台，适用《政府资料开放授权条款》<em>「不限目的、時間及地域、非專屬、不可撤回、免授權金」</em>，两家 OpenAPI 说明页写 <em>「歡迎各位介接使用」</em>；网站条款的知识产权一节写明授权给开放平台的资料 <em>「不在此限」</em>，转载时须保持完整、<em>「不得任意增刪」</em>（柜买中心的 OpenAPI 页没有链接授权条款，平台上登记的是网页下载地址，归 S 级是本工具的判断）；BLS <em>&quot;public domain&quot;</em>，API 条款要求注明取数日期并声明 <em>&quot;BLS.gov cannot vouch for the data or analyses derived from these data after the data have been retrieved from BLS.gov.&quot;</em>，免 key 版每天 25 次、试图绕开可能被封；劳工部 <em>&quot;public domain&quot;</em>，不得暗示与劳工部有关联。<strong>BIS / ECB / 美联储 / 台湾两家 / BLS 须注明来源（BLS 还要注明取数日期），ECB 不得改动数据，纽约联储参考利率对外展示须附其声明</strong></td></tr>
<tr><td nowrap><strong>B</strong></td><td nowrap>⚠️自行确认</td><td nowrap>❌</td><td>FINRA / EIA 现货价格</td><td>FINRA 数据文件（每日空头成交量、做空余额）主动发布；但条款禁止 <em>&quot;data mining, scraping or harvesting tools&quot;</em>，并声明 <em>&quot;non-commercial use&quot;</em>。EIA 的 WTI / 布伦特 / Henry Hub 现货价来源是 <em>&quot;Refinitiv, an LSEG business&quot;</em>，第三方材料超出合理使用须版权方书面许可（归 B 级是本工具的判断）</td></tr>
<tr><td nowrap><strong>C</strong></td><td nowrap>❌需授权</td><td nowrap>❌</td><td>CBOE / 东财 / 新浪 / 腾讯</td><td>Cboe（期权链与波动率指数）要求 <em>&quot;approval in advance&quot;</em> + <em>&quot;license agreement&quot;</em>；东财 / 新浪 / 腾讯为站点前端接口，仅个人研究</td></tr>
<tr><td nowrap><strong>D</strong></td><td nowrap>❌</td><td nowrap>❌</td><td>Yahoo / Nasdaq</td><td>条款禁止未经许可的自动抓取。Yahoo：<em>&quot;using any automated means… for any purpose without our express, prior permission&quot;</em>；Nasdaq：<em>&quot;any process, whether automated or manual, to capture data… for any reason&quot;</em>。<strong>仅供个人学习研究，风险自负，不得商用或再分发</strong></td></tr>
<tr><td nowrap><strong>⛔ 已排除</strong></td><td nowrap>—</td><td nowrap>—</td><td>HKEX (CCASS)</td><td>条款明文禁止 robot/bot/spider/scraper，且适用于&quot;不论是否营利&quot; → <strong>本工具不提供该抓取代码</strong></td></tr>
</tbody>
</table>

一个已跑通的 HKEX 席位持股层被删掉了——发布违反条款的抓取代码，会让"给数据源分级"这件事本身失去意义。**本工具只分发代码，不分发数据。** 商用请只依赖 S 级源。

**那为什么 D 级的 Yahoo / Nasdaq 还留着？** 坦白说，它们和 HKEX 的条款性质相同。区别只在于有没有替代：Yahoo 是港股 K 线和美港以外市场 K 线的唯一来源，Nasdaq 是财报日历唯一免 key 的来源，而 HKEX 席位持股是可有可无的增量。所以代码保留、条款原文照贴（完整原文见 SKILL.md「数据源合规分级」），用不用由你按条款自行判断。

---

## 快速开始

**3 步，2 分钟。**

```bash
# 1. 创建 skill 目录
mkdir -p ~/.claude/skills/global-stock-data

# 2. 下载 SKILL.md
curl -o ~/.claude/skills/global-stock-data/SKILL.md \
  https://raw.githubusercontent.com/simonlin1212/global-stock-data/main/SKILL.md

# 3. 安装依赖
pip install requests
# 仅 Windows：另装 tzdata，Yahoo K 线才能按交易所时区算日期
pip install tzdata
```

⚠️ **用 SEC 和美国宏观（BLS）相关层之前，先把 SKILL.md 里的 `SEC_CONTACT` 改成你自己的真实姓名和邮箱**，否则 SEC 会把请求当成未声明的自动化工具拒绝，BLS 网站直接返回 403。代码在你忘记时会明确报错。

启动 Claude Code，说一句「帮我看看 AAPL 的财报」自动激活。**Codex / OpenClaw 用户**把 SKILL.md 内容贴入系统 prompt 或项目上下文文件即可，内嵌 Python 可直接执行。

---

## 端点

**价格类** — 新浪/腾讯/东财行情（25-78 字段）· 新浪 & Yahoo K线（美股回溯至 1984，日→分钟；日本 / 台湾 / 韩国 / 印度 / 欧洲等换代码后缀）· MA/EMA/MACD/RSI/KDJ/布林带本地计算。

**价值类** — 东财三表 + GMAININDICATOR（美股 49/港股 75 字段）· Yahoo quoteSummary（23 模块：财务、分析师、机构持仓）· SEC EDGAR XBRL（503 个 GAAP 指标）。

**资金/情绪类 ⭐** — CBOE 期权含完整希腊字母 + IV + **0DTE** + 异动识别 · FINRA 全市场每日空头成交量（12,112 只）· 东财日级资金流 · **V2.2：** CBOE VIX 1990 年起历史 + 期限结构（判断 contango / 倒挂）+ VVIX / SKEW · FINRA 做空余额（每月两期，22,595 只，含回补天数与排行）。

**申报/事件类 ⭐** — EDGAR 每日索引（当日 **Form 4 内部人 / 8-K / 13F**，单日 547 / 370 / 261）· 2001 至今全文检索 · 10-K/10-Q/8-K 列表 · **V2.2：** Form 4 内部人逐笔买卖（价格、股数、交易后持股、所在申报是否勾选 10b5-1 计划）· 13F 机构季度持仓明细（按市值排序含占比）。

**筛选/宏观 ⭐** — EDGAR frames（任意 XBRL 标签一次拿全市场，免费——净利润 CY2025Q1 覆盖 5,309 家）· 美债收益率曲线（1M~30Y）· CFTC COT · Nasdaq 财报日历。

**全球利率汇率 ⭐（V2.1）** — BIS 38 家央行政策利率（一次取全）· 欧洲央行汇率 / 三大利率 / 欧元区收益率曲线 · 纽约联储 SOFR / EFFR · 美联储 H.10 美元对 23 种货币 / H.6 M1·M2 · 财政部国债拍卖结果与日程 / 联邦债务 / 国债平均利率。

**台股 ⭐（V2.2）** — 证交所 + 柜买中心官方开放接口：上市 / 上柜全市场最新一个交易日的行情、本益比 / 殖利率 / 股价净值比、月营收、公司基本资料、融资融券、除权除息、股利（只有最新一期；台股历史 K 线用 Yahoo `.TW` / `.TWO`）。

**能源 ⭐（V2.2）** — EIA 原油（商业 + 战略储备）、汽油、馏分油与天然气每周库存 · WTI（1986 起）/ 布伦特 / Henry Hub 现货日价（B 级：EIA 转载的第三方数据）。

**美国宏观 ⭐（V2.3）** — 美国劳工统计局 BLS：CPI / 核心 CPI（季调与未季调）、住房分项、CPI-W、链式 CPI、失业率、U-6、劳动参与率、非农、平均时薪、PPI 与核心 PPI、JOLTS 职位空缺与离职率、进口价格、雇佣成本指数、劳动生产率，或任意 BLS 序列号 · 最新一期的环比 / 同比与下次发布日期 · BLS 官方发布日程 · 劳工部每周初请 / 续请失业金（1967 年起）。免 key 版每天 25 次请求，同一请求 1 小时内复用。

**工具类** — 股票搜索（中英文）· 全市场列表（美股 5925+ / 港股 18000+）· 按代码查新闻 · ticker↔CIK。

<details>
<summary>展开完整端点表</summary>

<br>

### 期权层 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| **CBOE 官方** | 全链 + **IV + delta/gamma/vega/theta/rho**，含 **0DTE 筛选** 和 **异动识别**（vol/OI > 1 = 新建仓） |
| Yahoo（后备） | 期权链，所有到期日——**无希腊字母** |

### 波动率指数 ⭐（V2.2，仅美股）

| 端点 | 数据 |
|------|------|
| CBOE `vol_index_history()` | VIX（1990 起）/ VIX1D / VIX9D / VIX3M / VIX6M / VIX1Y / VVIX / SKEW / OVX / GVZ 等指数日线 |
| CBOE `vix_term_structure()` | 最新一日各期限 VIX，判断期限结构是 contango 还是倒挂，附长短端差 |

### 做空层 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| FINRA Reg SHO | 全市场**每日空头成交量**（实测 12,112 只）、个股时序、占比排行 |
| FINRA 做空余额 `short_interest()` / `short_interest_all()` / `short_interest_ranking()`（V2.2） | 每月两期的**未平仓空头股数**（实测 22,595 只）、回补天数、较上期变化，按回补天数 / 空头股数 / 变化排行 |

### 申报事件流 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| EDGAR 每日索引 | 当日 **Form 4 内部人 / 8-K / 13F / 144**——单日 547 / 370 / 261 / 118 |
| EDGAR 全文检索 | 检索所有申报正文，回溯至 2001 |
| EDGAR submissions / XBRL | Filing 列表 + 结构化财务（503 个 GAAP 指标） |
| Form 4 解析 `insider_trades()`（V2.2） | 内部人逐笔：交易日、交易类型、股数、价格、金额、交易后持股、所在申报是否勾选 10b5-1 计划；附公开市场买入 / 卖出汇总 |
| 13F 解析 `holdings_13f()`（V2.2） | 机构季度持仓明细，按市值排序含占比，可与封面申报总额对账；2023 年前申报的千美元单位自动换算 |

### 全市场横截面 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| EDGAR frames | 任意 XBRL 标签一次拿全市场 = 免费 screener |

### 宏观/日历 ⭐

| 端点 | 数据 |
|------|------|
| Treasury / CFTC / Nasdaq | 收益率曲线（1M~30Y）· COT 持仓 · 财报日历（含盘前盘后 + EPS 预期） |

### 全球利率 / 汇率 / 央行 ⭐（V2.1）

| 端点 | 数据 |
|------|------|
| BIS `policy_rates()` | 48 个经济体政策利率（38 个仍在更新），一次取全 |
| 欧洲央行 `ecb_fx()` / `ecb_key_rates()` / `ecb_yield_curve()` / `ecb_series()` | 欧元参考汇率 · 存款便利等三大利率 · 欧元区国债曲线（AAA 或全部国家）· 任意 ECB 序列 |
| 纽约联储 `nyfed_rates()` | SOFR / EFFR / OBFR / TGCR / BGCR，含分位数与成交量；SOFR 平均与指数 |
| 美联储 `fed_data()` | H.10 美元对 23 种货币 · H.6 M1/M2 及组成 |
| 财政部 `fiscal_data()` | 国债拍卖结果（中标利率、投标倍数）与未来日程 · 每日联邦债务 · 国债平均利率 · 记账汇率 |

### 台股 ⭐（V2.2）

| 端点 | 数据 |
|------|------|
| `tw_stock()` | 单只台股最新行情 + 本益比 / 股价净值比 / 殖利率，自动判断上市还是上柜 |
| `tw_market()` | 上市（证交所）或上柜（柜买中心）全市场或单只：行情 · 估值 · 月营收 · 基本资料 · 融资融券 · 除权除息预告 · 股利决议（股利仅上市） |

### 能源 ⭐（V2.2）

| 端点 | 数据 |
|------|------|
| EIA `energy_prices()` | WTI（1986 起）/ 布伦特（1987 起）原油现货，美元/桶；Henry Hub 天然气（1997 起）。B 级：价格来自 Refinitiv，商用或再分发前自行确认 |
| EIA `energy_inventories()` | 原油（商业 + 战略储备）、汽油、馏分油等每周库存；美国本土及分区天然气储量，含与五年均值对比 |

### 美国宏观 ⭐（V2.3）

| 端点 | 数据 |
|------|------|
| BLS `us_macro()` | 23 个常用指标一次请求取齐：CPI / 核心 CPI（季调与未季调）、住房分项、CPI-W、链式 CPI、失业率、U-6、劳动参与率、就业人口比、非农、平均时薪、平均周工时、PPI 与核心 PPI、JOLTS 职位空缺与离职率、进口价格、ECI 同比、劳动生产率 |
| BLS `us_macro_latest()` | 各指标最新一期、上一期、去年同期，本地算环比 / 同比，附下次发布日期 |
| BLS `bls_series()` | 任意 BLS 序列号；超过 25 个序列或 10 年自动拆成多次请求 |
| BLS `bls_release_calendar()` | 官方发布日程：CPI / 就业报告 / PPI / JOLTS 等每次发布的日期和美东时间 |
| 劳工部 `us_jobless_claims()` | 每周初请、续请失业金（季调 / 未季调 / 四周均值）、保险失业率，1967 年起 |

### 价格 / 价值 / 资金 / 工具

| 端点 | 数据 |
|------|------|
| 新浪 / 腾讯 / 东财 push2 | 美股/港股实时行情，25-78 字段 |
| 新浪 / Yahoo chart | K线，日→分钟，美股回溯至 1984；Yahoo 可取全历史日线（标普 500 自 1927 年）及美港以外市场 |
| 东财 datacenter / GMAININDICATOR | 三表 + 关键指标（中英双版） |
| Yahoo quoteSummary | 23 模块：财务 / 分析师 / 机构持仓 |
| 东财 push2his | 日级主力/大单/中单/小单资金流 |
| 东财 search / push2 列表 / Yahoo search / SEC CIK | 搜索 / 全市场列表 / 新闻 / ticker↔CIK |

全部数据源免费无 Key。Yahoo crumb 自动管理，SEC EDGAR 与 BLS 仅需声明 User-Agent（BLS 免 key 版每天 25 次请求）。

</details>

---

## 使用示例

跟你的 AI 助手说这些话就能激活：

| 场景 | 说什么 |
|------|--------|
| 0DTE 期权流 ⭐ | 「NVDA 的 0DTE 期权异动看一下」 |
| 空头成交 ⭐ | 「TSLA 本周空头成交占比趋势」 |
| 做空余额 ⭐ | 「GME 最近几期做空余额和回补天数」 |
| VIX 期限结构 ⭐ | 「现在 VIX 期限结构是不是倒挂」 |
| 内部人申报 ⭐ | 「今天有哪些 Form 4 内部人申报」 |
| 内部人买卖 ⭐ | 「NVDA 高管近半年卖了多少股」 |
| 13F 持仓 ⭐ | 「伯克希尔最新一季 13F 前十大持仓」 |
| 申报全文检索 ⭐ | 「哪些公司在 8-K 里首次提到 HBM4」 |
| 全市场筛选 ⭐ | 「按 CY2025Q1 研发费用给全市场排名」 |
| 宏观 ⭐ | 「现在 10Y-2Y 美债利差多少」 |
| 全球央行 ⭐ | 「全球主要央行现在的政策利率」·「最近一周 SOFR 和 EFFR」 |
| 汇率 / 美债 ⭐ | 「欧元兑日元这个月走势」·「下周美债拍卖日程」 |
| 台股 ⭐ | 「台积电最新收盘价和本益比」·「上柜股票这个月营收」 |
| 原油 / 天然气 ⭐ | 「这周 EIA 原油库存多少」·「WTI 近一年走势」 |
| 美国宏观 ⭐ | 「最新 CPI 和核心 CPI 同比多少，下次什么时候公布」·「最近几周初请失业金」 |
| 其他市场 | 「丰田 7203.T 近一年日 K」·「台积电 2330.TW 全部历史」 |
| 美股/港股行情 | 「AAPL 现在什么价，PE 多少」·「腾讯 00700 今天行情」 |
| K线 | 「拉 TSLA 最近半年日K线」 |
| 财报/估值 | 「苹果最新一季利润表」·「BABA 的 PE/PB/ROE 和目标价」 |
| 机构持仓 | 「哪些机构持有 NVDA，持股比例多少」 |
| 技术分析 | 「AAPL 的 MACD 和 RSI 有没有金叉」 |
| 批量对比 | 「对比 AAPL MSFT GOOGL 三家估值」 |

---

## 数据源

<table>
<thead>
<tr><th>数据源</th><th nowrap>级别</th><th nowrap>鉴权</th><th>覆盖</th></tr>
</thead>
<tbody>
<tr><td><strong>SEC EDGAR</strong></td><td nowrap><strong>S</strong></td><td nowrap>需真实UA</td><td>美股 Filing / XBRL / <strong>申报流</strong> / <strong>全文检索</strong> / <strong>全市场横截面</strong> / <strong>Form 4 内部人交易</strong> / <strong>13F 持仓</strong></td></tr>
<tr><td><strong>US Treasury</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td><strong>收益率曲线（1M~30Y）</strong> · FiscalData <strong>国债拍卖 / 联邦债务 / 平均利率</strong></td></tr>
<tr><td><strong>CFTC</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td><strong>COT 持仓报告</strong></td></tr>
<tr><td><strong>美联储理事会</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td><strong>H.10 汇率 / H.6 货币存量</strong>（注明来源）</td></tr>
<tr><td><strong>纽约联储</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td><strong>SOFR / EFFR / OBFR / TGCR / BGCR</strong>（对外展示须附声明）</td></tr>
<tr><td><strong>BIS</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td><strong>全球央行政策利率</strong>（注明来源）</td></tr>
<tr><td><strong>欧洲央行</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td><strong>欧元汇率 / ECB 利率 / 欧元区收益率曲线</strong>（注明来源、不得改动）</td></tr>
<tr><td><strong>美国能源信息署 EIA</strong></td><td nowrap><strong>S</strong> / B</td><td nowrap>无</td><td><strong>原油 / 天然气每周库存</strong>（S，建议注明来源）· WTI / 布伦特 / Henry Hub 现货价格（B：EIA 转载 Refinitiv 数据，商用需自行确认）</td></tr>
<tr><td><strong>美国劳工部</strong>（劳工统计局 BLS + 就业与培训局 ETA）</td><td nowrap><strong>S</strong></td><td nowrap>需真实UA</td><td><strong>CPI / 非农 / 失业率 / PPI / JOLTS / 进口价格 / ECI / 生产率</strong> · <strong>发布日程</strong> · <strong>每周初请 / 续请失业金</strong>（注明来源与取数日期；免 key 版每天 25 次）</td></tr>
<tr><td><strong>台湾证交所</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td>上市股票<strong>最新行情 / 本益比 / 月营收 / 融资融券 / 除权息 / 股利</strong>（注明来源）</td></tr>
<tr><td><strong>柜买中心</strong></td><td nowrap><strong>S</strong></td><td nowrap>无</td><td>上柜股票<strong>最新行情 / 本益比 / 月营收 / 融资融券 / 除权息</strong>（注明来源；S 级为本工具判断）</td></tr>
<tr><td><strong>FINRA</strong></td><td nowrap><strong>B</strong></td><td nowrap>无</td><td>美股 <strong>全市场每日空头成交量</strong> · <strong>做空余额</strong>（商用需自行确认）</td></tr>
<tr><td><strong>CBOE</strong></td><td nowrap><strong>C</strong></td><td nowrap>无</td><td>美股 <strong>期权 + 希腊字母 + IV + 0DTE</strong> · <strong>VIX 等波动率指数</strong>（使用需 Cboe 事先授权）</td></tr>
<tr><td>东财（push2 / push2his / datacenter / search）</td><td nowrap>C</td><td nowrap>无</td><td>美股+港股 行情 / 资金流 / 三表 / 搜索</td></tr>
<tr><td>新浪</td><td nowrap>C</td><td nowrap>无</td><td>美股+港股 行情、美股K线</td></tr>
<tr><td>腾讯</td><td nowrap>C</td><td nowrap>无</td><td>美股+港股 行情</td></tr>
<tr><td>Yahoo Finance</td><td nowrap>D</td><td nowrap>cookie+crumb（自动）</td><td>美股+港股+其他市场 全品类（条款禁止自动抓取）</td></tr>
<tr><td>Nasdaq</td><td nowrap>D</td><td nowrap>无</td><td>美股 <strong>财报日历</strong>（条款禁止自动抓取）</td></tr>
</tbody>
</table>

**级别含义**：**S** = 政府与央行数据（含登记为台湾政府开放资料的交易所数据），可商用可再分发（部分要求注明来源）· **B** = 主动公开的数据，商用需自行确认 · **C** = 需事先授权或条款未核实，仅个人研究 · **D** = 条款禁止未经许可的自动抓取，仅供个人学习研究、风险自负、不得商用或再分发。依据原文见 [合规分级](#合规分级)。所有请求走直连 HTTP，内置线程安全限流器（SEC 按官方 10 req/s 硬上限设为 8 req/s）。

---

## FAQ

**和 a-stock-data 什么关系？** 姊妹项目。[a-stock-data](https://github.com/simonlin1212/a-stock-data) 覆盖 A 股，global-stock-data 覆盖美股，两个 Skill 可同时装、互不冲突。

**Yahoo 要 API Key 吗？** 不要，代码自动管理 cookie + crumb，过期自动刷新。

**SEC EDGAR 有限制吗？** 有，须声明 User-Agent + 每秒 10 次上限，代码已内置节流；记得改 `SEC_CONTACT`。

**港股期权有吗？** 没有，期权层仅美股（港股期权需港交所付费专有接口）。

**能查美港以外的股票吗？** 能查 K 线和 Yahoo 的基本面：日本 `.T`、台湾 `.TW`、韩国 `.KS`、印度 `.NS`、伦敦 `.L`、德国 `.DE` 等，代码格式见 SKILL.md「Yahoo Finance 代码格式」。注意 Yahoo 是 D 级来源。台股另有证交所 / 柜买中心的官方接口（S 级，只有最新一个交易日）：`tw_stock("2330")`。A 股请用姊妹项目 a-stock-data。

**全球利率汇率数据多新？** 欧洲央行、纽约联储每个工作日更新；美联储 H.10 每周一发布上一周；BIS 各国有几天到两三周的滞后。所有返回值都带日期，需要当天准确值请查对应央行。

**做空余额多久更新一次？** FINRA 每月两期，结算日是每月 15 日和月末（遇假日提前），文件在结算日之后约 8 个工作日才发布。每天都有的是另一套数据——每日空头成交量（9.1），它统计的是当天的做空成交，不是未平仓的空头。

**美国宏观数据有次数限制吗？会修订吗？** BLS 免注册的接口每天 25 次请求、每次最多 25 个序列和 10 年；`us_macro()` 一次请求就能取齐 23 个指标，同一请求 1 小时内不重复占次数。非农、PPI、JOLTS 等发布后还会修订，接口给的是最新修订值，不是当时第一次公布的数；用来回放历史时按 `bls_release_calendar()` 的真实发布日对齐。各指标的修订规则见 SKILL.md 16.5 节。

**为什么没有 GDP？** GDP 由美国经济分析局 BEA 发布，它的接口要注册 key；按本项目规则，要注册账号或申请 key 的数据源不接。

**国内服务器能访问 Yahoo/SEC 吗？** 境外服务，直连可能不稳，建议走代理或优先用东财/新浪/腾讯。

**不用 Claude Code 能用吗？** 能。SKILL.md 是 Markdown + 内嵌 Python，任何 AI 编程助手都能读，也可以直接把代码复制出来跑。

---

## 更新日志

见 [CHANGELOG.md](./CHANGELOG.md)。

## 免责声明

本项目仅提供数据获取工具，不构成任何投资建议。股市有风险，投资需谨慎。

## License

[Apache License 2.0](./LICENSE) — 自由使用，注明出处即可。**Author:** Simon Lin · X [@linsizhen](https://x.com/linsizhen) · Email: [simonlin0423@gmail.com](mailto:simonlin0423@gmail.com)
