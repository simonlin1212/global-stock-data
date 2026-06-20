# Changelog

## v1.0.1 — 2026-06-20

### 修复（PR #1 @APTX4869-maker + 连带 bug）
- **5 个函数漏传 `params` 导致始终拿不到数据（PR #1）**：`stock_quote_eastmoney` / `stock_kline_yahoo` / `fund_flow_daily` / `stock_search` / `market_stock_list` 都构造了 `params` dict，但 `requests.get(url, ...)` 时漏了 `params=params`，请求实际是裸 URL → 服务端返回空（东财 push2 返回 `rc:102`/`data:null`）。一次性补齐 5 处。**实测确认**：漏传时 `data=None`，补齐后 `market_stock_list` 返回 `total=5990`。致谢 @APTX4869-maker 的排查与验证。
- **连带 bug：`market_stock_list` 在 `diff` 为 dict 时崩溃（本次一并修）**：补齐 params 后该函数能拿到响应，但东财 push2 的 `diff` 字段**有时是 list、有时是按序号为键的 dict**（如 `{"0":{...},"1":{...}}`）。旧代码 `for item in diff` 遇 dict 会拿到字符串键 → `AttributeError: 'str' object has no attribute 'get'`。新增 `if isinstance(diff, dict): diff = list(diff.values())` 归一化。**实测确认**：当前返回的 `diff` 正是 dict 结构，归一化后正常遍历。

### 说明
- 八层架构 / 18 端点 / 5 数据源不变；纯 bug 修复补丁。这 5 个端点此前**始终返回空数据**，升级强烈建议。

## v1.0 — 2026-05-20

### 首次开源发布

- **八层数据架构**：行情 / K线 / 技术指标 / 基本面 / 资金面 / 期权 / SEC Filing / 工具
- **18 个端点**覆盖美股 + 港股全品类数据
- **5 个数据源**：东财（push2 + push2his + datacenter + search）、Yahoo Finance（crumb 自动管理）、新浪财经、腾讯财经、SEC EDGAR
- 全部零鉴权（Yahoo crumb 自动获取，SEC 仅需 User-Agent）
- 仅依赖 `requests`，零第三方数据封装
- 内嵌完整 Python 代码，AI 编程助手直接可用
- 2026-05-20 全部端点实测验证

### 端点清单

| 层 | 端点 | 数据源 |
|----|------|--------|
| 行情 | 美股/港股实时报价 × 3 | 新浪 + 腾讯 + 东财push2 |
| K线 | 日/周/月/分钟 × 2 | 新浪 + Yahoo chart |
| 技术指标 | MA/EMA + MACD + RSI + KDJ + 布林带 × 1 | 纯Python计算（基于K线OHLCV） |
| 基本面 | 财报三表 + GMAININDICATOR + Yahoo 23模块 + SEC XBRL | 东财datacenter + Yahoo + EDGAR |
| 资金面 | 日级资金流 × 1 | 东财push2his |
| 期权 | 期权链 × 1 | Yahoo |
| SEC Filing | Filing列表 + XBRL × 2 | EDGAR |
| 工具 | 搜索 + 全市场列表 + 新闻 + CIK映射 × 4 | 东财search + 东财push2 + Yahoo + SEC |

### 实测发现与修正

- **东财 push2his K线不覆盖美股/港股**：kline/get 端点对美股(105.AAPL)/港股(116.00700) secid 返回空数据，仅资金流(fflow/daykline/get)正常。K线层改为新浪(美股) + Yahoo chart(美股+港股)
- **Yahoo 新闻需要 cookie**：v1/finance/search 裸请求返回 400，需先访问 fc.yahoo.com 获取 cookie
- **东财 push2 实时行情已验证**：push2.eastmoney.com/api/qt/stock/get 对美股(105.AAPL)和港股(116.00700)均返回完整数据
- **东财 GMAININDICATOR 已验证**：美股(RPT_USF10)返回49字段，港股(RPT_HKF10)返回75字段
- **东财 push2 全市场列表已验证**：美股(m:105)返回5925只，港股(m:116)返回18000+只
