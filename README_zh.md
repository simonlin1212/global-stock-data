<p align="center"><b>简体中文</b> | <a href="README.md">English</a></p>

<h1 align="center">global-stock-data</h1>

<p align="center">
  <b>给 AI 编程助手用的美股数据工具包</b><br>
  期权希腊字母 · 0DTE 流 · SEC 申报 · 做空数据 · 基本面 · 全市场筛选
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License: Apache 2.0"></a>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white" alt="Python 3.9+"></a>
  <a href="https://github.com/simonlin1212/global-stock-data/stargazers"><img src="https://img.shields.io/github/stars/simonlin1212/global-stock-data?style=social" alt="GitHub stars"></a>
  <a href="#架构"><img src="https://img.shields.io/badge/层数-13-2ea44f.svg" alt="层数"></a>
  <a href="#端点"><img src="https://img.shields.io/badge/端点-30%2B-2ea44f.svg" alt="端点"></a>
  <a href="#数据源"><img src="https://img.shields.io/badge/数据源-11-2ea44f.svg" alt="数据源"></a>
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

> **给 AI 编程助手用的美股全栈数据工具包** — 13 层架构 · 30+ 个端点 · 11 个数据源 · 全部零鉴权 · 仅依赖 `requests`。
>
> **官方源优先（V2.0）：** CBOE 官方期权链（完整希腊字母 + **0DTE** 异动 flow）、FINRA 全市场每日空头成交量、SEC EDGAR 申报事件流 + 免费全市场 screener、美债收益率曲线 / CFTC COT / 财报日历。**每个数据源标注了合规级别与条款原文**——因为"官方"不等于"可自由使用"。

一个自包含的 Skill 文件，把分散在多个数据源里的美股/港股原始数据，整合成 AI 编程助手直接能用的工具集。你不用再背东财 secid 前缀、Yahoo crumb 鉴权流程、SEC EDGAR 的 CIK 映射——全部封装好了。

> *本工具只分发**代码，不分发市场数据**。数据由使用者自行按各源条款获取。商用请只依赖 S 级源。*

> 兼容 [Claude Code](https://github.com/anthropics/claude-code) · [Codex](https://github.com/openai/codex) · [OpenClaw](https://github.com/anthropics/openclaw)。Skill 文件本质是结构化 Markdown + 内嵌 Python，任何支持上下文注入的 AI 编程助手都能用。

---

## 寻找深圳的 AI 工作机会

作者目前正在寻找深圳的 AI 相关工作机会，重点关注腾讯等大型科技企业及金融机构的 **AI 投研产品、FDE 与 AI 咨询 / 解决方案岗位**。

兼具金融机构从业经历与 AI 产品实战，持续构建金融市场数据工具和多智能体系统，开源项目累计获得 **17K+ GitHub Stars**。

联系：[simonlin0423@gmail.com](mailto:simonlin0423@gmail.com)

---

## 架构

```
美股全栈数据 · 13 层架构 · V2.0
│
├── 行情层      新浪 + 腾讯 + 东财push2        实时报价 25-78 字段
├── K线层      新浪(回溯至1984) + Yahoo         日/周/月/分钟
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
```

---

## 合规分级

各源条款差异极大，**"官方"不等于"可自由使用"**。以下结论来自 2026-07-24 逐家实读条款原文，引号内为原文：

| 级别 | 可商用 | 可再分发 | 源 | 依据（原文摘录） |
|---|---|---|---|---|
| **S** | ✅ | ✅ | SEC EDGAR / Treasury / CFTC | EDGAR 明示 *"for free"*、*"allow scripted access"*；**硬上限 10 请求/秒**，须声明 User-Agent |
| **B** | ⚠️自行确认 | ❌ | FINRA | 数据文件主动发布；但条款禁止 *"data mining, scraping or harvesting tools"*，并声明 *"non-commercial use"* |
| **C** | ❌需授权 | ❌ | CBOE / Nasdaq / Yahoo / 东财 / 新浪 / 腾讯 | Cboe 要求 *"approval in advance"* + *"license agreement"*；Yahoo 写明 personal use only |
| **⛔ 已排除** | — | — | HKEX (CCASS) | 条款明文禁止 robot/bot/spider/scraper，且适用于"不论是否营利" → **本工具不提供该抓取代码** |

一个已跑通的 HKEX 席位持股层被删掉了——发布违反条款的抓取代码，会让"给数据源分级"这件事本身失去意义。**本工具只分发代码，不分发数据。** 商用请只依赖 S 级源。

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
```

⚠️ **用 SEC 相关层之前，先把 SKILL.md 里的 `SEC_CONTACT` 改成你自己的真实姓名和邮箱**，否则 SEC 会把请求当成未声明的自动化工具拒绝。代码在你忘记时会明确报错。

启动 Claude Code，说一句「帮我看看 AAPL 的财报」自动激活。**Codex / OpenClaw 用户**把 SKILL.md 内容贴入系统 prompt 或项目上下文文件即可，内嵌 Python 可直接执行。

---

## 端点

**价格类** — 新浪/腾讯/东财行情（25-78 字段）· 新浪 & Yahoo K线（美股回溯至 1984，日→分钟）· MA/EMA/MACD/RSI/KDJ/布林带本地计算。

**价值类** — 东财三表 + GMAININDICATOR（美股 49/港股 75 字段）· Yahoo quoteSummary（23 模块：财务、分析师、机构持仓）· SEC EDGAR XBRL（503 个 GAAP 指标）。

**资金/情绪类 ⭐** — CBOE 期权含完整希腊字母 + IV + **0DTE** + 异动识别 · FINRA 全市场每日空头成交量（12,112 只）· 东财日级资金流。

**申报/事件类 ⭐** — EDGAR 每日索引（当日 **Form 4 内部人 / 8-K / 13F**，单日 547 / 370 / 261）· 2001 至今全文检索 · 10-K/10-Q/8-K 列表。

**筛选/宏观 ⭐** — EDGAR frames（任意 XBRL 标签一次拿全市场，免费——净利润 CY2025Q1 覆盖 5,309 家）· 美债收益率曲线（1M~30Y）· CFTC COT · Nasdaq 财报日历。

**工具类** — 股票搜索（中英文）· 全市场列表（美股 5925+ / 港股 18000+）· 按代码查新闻 · ticker↔CIK。

<details>
<summary>展开完整端点表</summary>

<br>

### 期权层 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| **CBOE 官方** | 全链 + **IV + delta/gamma/vega/theta/rho**，含 **0DTE 筛选** 和 **异动识别**（vol/OI > 1 = 新建仓） |
| Yahoo（后备） | 期权链，所有到期日——**无希腊字母** |

### 做空层 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| FINRA Reg SHO | 全市场**每日空头成交量**（实测 12,112 只）、个股时序、占比排行 |

### 申报事件流 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| EDGAR 每日索引 | 当日 **Form 4 内部人 / 8-K / 13F / 144**——单日 547 / 370 / 261 / 118 |
| EDGAR 全文检索 | 检索所有申报正文，回溯至 2001 |
| EDGAR submissions / XBRL | Filing 列表 + 结构化财务（503 个 GAAP 指标） |

### 全市场横截面 ⭐（仅美股）

| 端点 | 数据 |
|------|------|
| EDGAR frames | 任意 XBRL 标签一次拿全市场 = 免费 screener |

### 宏观/日历 ⭐

| 端点 | 数据 |
|------|------|
| Treasury / CFTC / Nasdaq | 收益率曲线（1M~30Y）· COT 持仓 · 财报日历（含盘前盘后 + EPS 预期） |

### 价格 / 价值 / 资金 / 工具

| 端点 | 数据 |
|------|------|
| 新浪 / 腾讯 / 东财 push2 | 美股/港股实时行情，25-78 字段 |
| 新浪 / Yahoo chart | K线，日→分钟，美股回溯至 1984 |
| 东财 datacenter / GMAININDICATOR | 三表 + 关键指标（中英双版） |
| Yahoo quoteSummary | 23 模块：财务 / 分析师 / 机构持仓 |
| 东财 push2his | 日级主力/大单/中单/小单资金流 |
| 东财 search / push2 列表 / Yahoo search / SEC CIK | 搜索 / 全市场列表 / 新闻 / ticker↔CIK |

全部数据源免费无 Key。Yahoo crumb 自动管理，SEC EDGAR 仅需声明 User-Agent。

</details>

---

## 使用示例

跟你的 AI 助手说这些话就能激活：

| 场景 | 说什么 |
|------|--------|
| 0DTE 期权流 ⭐ | 「NVDA 的 0DTE 期权异动看一下」 |
| 空头成交 ⭐ | 「TSLA 本周空头成交占比趋势」 |
| 内部人申报 ⭐ | 「今天有哪些 Form 4 内部人申报」 |
| 申报全文检索 ⭐ | 「哪些公司在 8-K 里首次提到 HBM4」 |
| 全市场筛选 ⭐ | 「按 CY2025Q1 研发费用给全市场排名」 |
| 宏观 ⭐ | 「现在 10Y-2Y 美债利差多少」 |
| 美股/港股行情 | 「AAPL 现在什么价，PE 多少」·「腾讯 00700 今天行情」 |
| K线 | 「拉 TSLA 最近半年日K线」 |
| 财报/估值 | 「苹果最新一季利润表」·「BABA 的 PE/PB/ROE 和目标价」 |
| 机构持仓 | 「哪些机构持有 NVDA，持股比例多少」 |
| 技术分析 | 「AAPL 的 MACD 和 RSI 有没有金叉」 |
| 批量对比 | 「对比 AAPL MSFT GOOGL 三家估值」 |

---

## 数据源

| 数据源 | 级别 | 鉴权 | 覆盖 |
|--------|------|------|------|
| **SEC EDGAR** | **S** | 需真实UA | 美股 Filing / XBRL / **申报流** / **全文检索** / **全市场横截面** |
| **US Treasury** | **S** | 无 | **收益率曲线（1M~30Y）** |
| **CFTC** | **S** | 无 | **COT 持仓报告** |
| **FINRA** | **B** | 无 | 美股 **全市场每日空头成交量**（商用需自行确认） |
| **CBOE** | **C** | 无 | 美股 **期权 + 希腊字母 + IV + 0DTE**（使用需 Cboe 事先授权） |
| **Nasdaq** | **C** | 无 | 美股 **财报日历**（条款未核实） |
| 东财（push2 / push2his / datacenter / search） | C | 无 | 美股+港股 行情 / 资金流 / 三表 / 搜索 |
| Yahoo Finance | C | cookie+crumb（自动） | 美股+港股 全品类（**personal use only**） |
| 新浪 | C | 无 | 美股+港股 行情、美股K线 |
| 腾讯 | C | 无 | 美股+港股 行情 |

**级别含义**：**S** = 政府数据，可商用可再分发 · **B** = 主动公开的数据文件，商用需自行确认 · **C** = 需事先授权或条款未核实，仅个人研究。依据原文见 [合规分级](#合规分级)。所有请求走直连 HTTP，内置线程安全限流器（SEC 按官方 10 req/s 硬上限设为 8 req/s）。

---

## FAQ

**和 a-stock-data 什么关系？** 姊妹项目。[a-stock-data](https://github.com/simonlin1212/a-stock-data) 覆盖 A 股，global-stock-data 覆盖美股，两个 Skill 可同时装、互不冲突。

**Yahoo 要 API Key 吗？** 不要，代码自动管理 cookie + crumb，过期自动刷新。

**SEC EDGAR 有限制吗？** 有，须声明 User-Agent + 每秒 10 次上限，代码已内置节流；记得改 `SEC_CONTACT`。

**港股期权有吗？** 没有，期权层仅美股（港股期权需港交所付费专有接口）。

**国内服务器能访问 Yahoo/SEC 吗？** 境外服务，直连可能不稳，建议走代理或优先用东财/新浪/腾讯。

**不用 Claude Code 能用吗？** 能。SKILL.md 是 Markdown + 内嵌 Python，任何 AI 编程助手都能读，也可以直接把代码复制出来跑。

---

## 更新日志

见 [CHANGELOG.md](./CHANGELOG.md)。

## 免责声明

本项目仅提供数据获取工具，不构成任何投资建议。股市有风险，投资需谨慎。

## 赞赏

如果这个工具帮到了你 ☕

<p align="center">
  <a href="https://buymeacoffee.com/simonlin1212"><img src="./assets/bmc-qr.png" width="180" alt="Buy Me a Coffee"></a>
</p>

想要什么数据端点？欢迎开 [Issue](https://github.com/simonlin1212/global-stock-data/issues)，赞助者的 Issue 优先处理。

## License

[Apache License 2.0](./LICENSE) — 自由使用，注明出处即可。**Author:** Simon Lin · X [@linsizhen](https://x.com/linsizhen) · Email: [simonlin0423@gmail.com](mailto:simonlin0423@gmail.com)
