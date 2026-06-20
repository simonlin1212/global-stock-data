---
name: global-stock-data
description: 美股港股全栈数据工具包 — 覆盖行情(新浪+腾讯+东财push2)、K线(新浪+Yahoo)、技术指标(MA/MACD/RSI/KDJ/布林带)、基本面(东财datacenter三表+GMAININDICATOR+Yahoo quoteSummary+SEC XBRL)、资金面(东财push2his日级资金流)、期权(Yahoo)、SEC Filing(EDGAR)、搜索与工具(东财search+Yahoo+SEC CIK+全市场列表)八层数据源，内嵌全部调用代码，自包含零依赖外部文件。适用于美股港股个股分析、全市场筛选、财报解读、期权策略、SEC文件检索、资金流追踪、机构持仓分析等场景。
origin: custom
version: 1.0.1
---

> 📦 项目主页：https://github.com/simonlin1212/global-stock-data — 更新、反馈、支持作者
> 
> 作者：Simon 林 · 抖音「Simon林」· 公众号「硅基世纪」

# 美股港股全栈数据工具包 V1.0.1

八层数据架构，18 个端点，5 个数据源，全部零鉴权，实测可用（2026-05-20 验证）。

**使用方式：** 将本文件放入 `~/.claude/skills/global-stock-data/SKILL.md`，Claude Code 会自动识别并在美股/港股相关对话中激活。

```
行情层（实时/延时）
├── 新浪财经     → 美股 gb_XXXX 36字段 / 港股 rt_hkXXXXX 25字段
├── 腾讯财经     → 美股 usXXXX 71字段 / 港股 r_hkXXXXX 78字段
└── 东财 push2   → 美股/港股 secid 实时行情，含中文名/涨跌幅/换手率

K线层（日/周/月/分钟）
├── 新浪          → 美股日K (回溯至1984年)
└── Yahoo chart   → 美股+港股 (v8 API, 零crumb)

技术指标层（纯计算，零额外依赖）
└── MA/EMA + MACD + RSI + KDJ + 布林带    基于K线OHLCV，纯Python计算

基本面层
├── 东财 datacenter → 美股/港股三表(资产负债+利润+现金流) + GMAININDICATOR(关键指标)
├── Yahoo crumb     → 23个模块(财务数据+关键指标+分析师+机构持仓)
└── SEC EDGAR XBRL  → 美股503个GAAP指标 (仅美股)

资金面层
└── 东财 push2his → 日级资金流(主力/大单/中单/小单) 美股+港股

期权层（仅美股）
└── Yahoo crumb → 期权链(calls+puts, 所有到期日) 仅美股(港股期权不在Yahoo覆盖范围)

SEC Filing层（仅美股）
├── EDGAR submissions → 10-K/10-Q/8-K 完整Filing列表
└── EDGAR XBRL        → 结构化财务指标(营收/净利/EPS等)

工具层
├── 东财 search    → 股票搜索(中英文, 含市场代码映射)
├── 东财 push2     → 全市场股票列表(涨跌幅/成交量排名, 美股5925只+港股18000+只)
├── Yahoo search   → 新闻资讯(按股票代码)
└── SEC CIK mapping → ticker↔CIK 映射 (仅美股)
```

## When to Activate

- 用户要查**美股/港股**行情（价格/涨跌幅/成交量）
- 用户要拉 K 线（日线/周线/月线/分钟线）
- 用户要看**财报**（资产负债表/利润表/现金流量表）
- 用户要看**关键财务指标**（PE/PB/ROE/利润率/目标价）
- 用户要看**分析师预期**（EPS预测/评级/目标价区间）
- 用户要看**机构持仓**（前十大机构/持股比例）
- 用户要看**资金流向**（主力/大单/中单/小单净流入）
- 用户要查**期权链**（calls/puts/到期日/Greeks）
- 用户要查 **SEC Filing**（10-K/10-Q/8-K/年报/季报）
- 用户要做**美股财报量化分析**（从 XBRL 拉多年营收/净利/EPS 趋势）
- 用户要**搜索股票**（中英文均可）
- 用户要看**美股/港股新闻**
- 用户要看**全市场涨跌幅排名**（当日涨幅/跌幅最大的股票）
- 用户要做**全市场筛选**（遍历美股/港股列表做初筛）
- 用户要看**关键财务指标概览**（营收/净利/EPS/ROE/ROA/资产负债率 中文版）
- 用户要看**技术指标**（MACD/RSI/KDJ/布林带/均线）
- 用户要判断**金叉死叉/超买超卖/变盘信号**
- 关键词：美股、港股、AAPL、苹果、腾讯、00700、TSLA、特斯拉、BABA、阿里巴巴、行情、K线、财报、PE、PB、ROE、分析师、目标价、期权、call、put、SEC、10-K、年报、季报、资金流、主力、机构持仓、新闻、涨幅排名、全市场、筛选、关键指标、MACD、RSI、KDJ、布林带、均线、金叉、死叉、超买、超卖、技术分析

---

## Prerequisites

```bash
pip install requests
```

| 依赖 | 版本要求 | 用途 |
|------|---------|------|
| requests | any | 所有 HTTP API 直连 |

> **极简依赖：** 仅需 requests，所有数据源均为直连 HTTP API，零第三方数据封装。

---

## 市场代码规则

### 东财 secid 前缀（push2/push2his 用）

| 前缀 | 市场 | 示例 |
|------|------|------|
| 105 | 美股 NASDAQ | `105.AAPL`, `105.TSLA` |
| 106 | 美股 NYSE | `106.BABA`, `106.JD` |
| 107 | 美股 ETF/其他 | `107.CRSH` |
| 116 | 港股 | `116.00700`, `116.09988` |

> **如何判断 105/106/107？** 调 `stock_search()` 获取 `MktNum` 字段自动映射。

### Yahoo Finance 代码格式

| 市场 | 格式 | 示例 |
|------|------|------|
| 美股 | 直接 ticker | `AAPL`, `TSLA`, `BABA` |
| 港股 | 四/五位数字 + `.HK` | `0700.HK`, `9988.HK` |

### 东财 datacenter SECUCODE 格式

| 市场 | 格式 | 示例 |
|------|------|------|
| 美股 NASDAQ | `TICKER.O` | `AAPL.O`, `TSLA.O` |
| 美股 NYSE | `TICKER.N` | `BABA.N`, `JD.N` |
| 港股 | `CODE.HK` | `00700.HK`, `09988.HK` |

---

## 共用 Helper 函数

### Yahoo Finance crumb 管理器

Yahoo quoteSummary/options 等 v7/v10 接口需要 cookie+crumb。以下 helper 自动获取并缓存：

```python
import requests

_yahoo_session = None

def get_yahoo_session() -> requests.Session:
    """获取带 crumb 的 Yahoo Finance session（自动缓存）"""
    global _yahoo_session
    if _yahoo_session and hasattr(_yahoo_session, '_crumb'):
        return _yahoo_session
    
    s = requests.Session()
    s.headers['User-Agent'] = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
    
    # Step 1: 获取 cookie
    s.get('https://fc.yahoo.com', timeout=10)
    
    # Step 2: 获取 crumb
    r = s.get('https://query2.finance.yahoo.com/v1/test/getcrumb', timeout=10)
    r.raise_for_status()
    s._crumb = r.text
    
    _yahoo_session = s
    return s

def yahoo_quote_summary(symbol: str, modules: list[str]) -> dict:
    """Yahoo quoteSummary 统一查询"""
    s = get_yahoo_session()
    r = s.get(f'https://query2.finance.yahoo.com/v10/finance/quoteSummary/{symbol}', params={
        'modules': ','.join(modules),
        'crumb': s._crumb,
    }, timeout=15)
    r.raise_for_status()
    results = r.json().get('quoteSummary', {}).get('result', [{}])
    return results[0] if results else {}
```

### 东财数据中心统一查询

```python
UA = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
DATACENTER_URL = "https://datacenter-web.eastmoney.com/api/data/v1/get"

def eastmoney_datacenter(report_name: str, columns: str = "ALL",
                          filter_str: str = "", page_size: int = 50,
                          sort_columns: str = "", sort_types: str = "-1") -> list[dict]:
    """东财数据中心统一查询"""
    params = {
        "reportName": report_name, "columns": columns,
        "filter": filter_str, "pageNumber": "1", "pageSize": str(page_size),
        "sortColumns": sort_columns, "sortTypes": sort_types,
        "source": "WEB", "client": "WEB",
    }
    r = requests.get(DATACENTER_URL, params=params, headers={"User-Agent": UA}, timeout=15)
    d = r.json()
    if d.get("result") and d["result"].get("data"):
        return d["result"]["data"]
    return []
```

---

## Layer 1: 行情层

### 1.1 美股实时行情 — 新浪 + 腾讯

两个独立数据源，任一可用即可。新浪字段侧重价格成交，腾讯字段更全（含52周高低/市值/PE）。

```python
import requests, re

def us_stock_quote_sina(ticker: str) -> dict:
    """
    新浪美股行情 — 36字段
    ticker: 纯字母，如 "AAPL", "TSLA", "BABA"
    """
    url = f"https://hq.sinajs.cn/list=gb_{ticker.lower()}"
    r = requests.get(url, headers={
        "Referer": "https://finance.sina.com.cn/",
        "User-Agent": UA,
    }, timeout=10)
    r.encoding = "gbk"
    text = r.text
    
    m = re.search(r'"(.+)"', text)
    if not m:
        return {}
    
    fields = m.group(1).split(",")
    if len(fields) < 30:
        return {}
    
    return {
        "name": fields[0],           # 中文名
        "price": float(fields[1]),    # 最新价
        "change_pct": float(fields[2]),  # 涨跌幅 %
        "timestamp": fields[3],       # 时间
        "prev_close": float(fields[26]),  # 昨收
        "open": float(fields[5]),     # 开盘
        "high": float(fields[6]),     # 最高
        "low": float(fields[7]),      # 最低
        "volume": float(fields[10]) if fields[10] else 0,  # 成交量
        "high_52w": float(fields[8]) if fields[8] else 0,  # 52周最高
        "low_52w": float(fields[9]) if fields[9] else 0,   # 52周最低
        "market_cap": float(fields[12]) if fields[12] else 0,  # 市值
        "eps": float(fields[13]) if fields[13] else 0,  # EPS
        "pe": float(fields[14]) if fields[14] else 0,   # PE
    }


def us_stock_quote_tencent(ticker: str) -> dict:
    """
    腾讯美股行情 — 71字段
    ticker: 纯字母，如 "AAPL"
    """
    url = f"https://qt.gtimg.cn/q=us{ticker.upper()}"
    r = requests.get(url, timeout=10)
    r.encoding = "gbk"
    text = r.text
    
    m = re.search(r'"(.+)"', text)
    if not m:
        return {}
    
    fields = m.group(1).split("~")
    if len(fields) < 50:
        return {}
    
    return {
        "name": fields[1],           # 中文名
        "name_en": fields[27],       # 英文名
        "price": float(fields[3]) if fields[3] else 0,
        "prev_close": float(fields[4]) if fields[4] else 0,
        "open": float(fields[5]) if fields[5] else 0,
        "volume": int(fields[6]) if fields[6] else 0,
        "high": float(fields[33]) if fields[33] else 0,
        "low": float(fields[34]) if fields[34] else 0,
        "high_52w": float(fields[35]) if fields[35] else 0,
        "low_52w": float(fields[36]) if fields[36] else 0,
        "change_pct": float(fields[32]) if fields[32] else 0,
        "market_cap": float(fields[44]) if fields[44] else 0,  # 亿美元
        "pe": float(fields[53]) if fields[53] else 0,
        "pb": float(fields[56]) if fields[56] else 0,
        "timestamp": fields[30],
    }
```

### 1.2 港股实时行情 — 腾讯 + 新浪

```python
def hk_stock_quote_tencent(code: str) -> dict:
    """
    腾讯港股行情 — 78字段（最全）
    code: 五位数字，如 "00700", "09988"
    """
    url = f"https://qt.gtimg.cn/q=r_hk{code}"
    r = requests.get(url, timeout=10)
    r.encoding = "gbk"
    text = r.text
    
    m = re.search(r'"(.+)"', text)
    if not m:
        return {}
    
    fields = m.group(1).split("~")
    if len(fields) < 50:
        return {}
    
    return {
        "name": fields[1],           # 中文名
        "name_en": fields[2],        # 英文名
        "price": float(fields[3]) if fields[3] else 0,
        "prev_close": float(fields[4]) if fields[4] else 0,
        "open": float(fields[5]) if fields[5] else 0,
        "high": float(fields[33]) if fields[33] else 0,
        "low": float(fields[34]) if fields[34] else 0,
        "volume": int(fields[6]) if fields[6] else 0,    # 成交量(股)
        "amount": float(fields[37]) if fields[37] else 0,  # 成交额
        "change_pct": float(fields[32]) if fields[32] else 0,
        "pe": float(fields[39]) if fields[39] else 0,
        "pb": float(fields[56]) if fields[56] else 0,
        "high_52w": float(fields[35]) if fields[35] else 0,
        "low_52w": float(fields[36]) if fields[36] else 0,
        "market_cap": float(fields[44]) if fields[44] else 0,  # 亿港元
        "timestamp": fields[30],
    }


def hk_stock_quote_sina(code: str) -> dict:
    """
    新浪港股行情 — 25字段
    code: 五位数字，如 "00700"
    """
    url = f"https://hq.sinajs.cn/list=rt_hk{code}"
    r = requests.get(url, headers={
        "Referer": "https://finance.sina.com.cn/",
        "User-Agent": UA,
    }, timeout=10)
    r.encoding = "gbk"
    text = r.text
    
    m = re.search(r'"(.+)"', text)
    if not m:
        return {}
    
    fields = m.group(1).split(",")
    if len(fields) < 15:
        return {}
    
    return {
        "name_en": fields[0],
        "name": fields[1],           # 中文名
        "open": float(fields[2]) if fields[2] else 0,
        "prev_close": float(fields[3]) if fields[3] else 0,
        "high": float(fields[4]) if fields[4] else 0,
        "low": float(fields[5]) if fields[5] else 0,
        "price": float(fields[6]) if fields[6] else 0,
        "change": float(fields[7]) if fields[7] else 0,
        "change_pct": float(fields[8]) if fields[8] else 0,
        "volume": float(fields[12]) if fields[12] else 0,
        "amount": float(fields[11]) if fields[11] else 0,
    }
```

### 1.3 东财 push2 实时行情 — 美股 + 港股

东财 push2 接口，通过 secid 统一查询美股/港股实时行情。优点：有中文名、换手率、涨跌幅，且 secid 可由 `stock_search()` 自动获取。

```python
def stock_quote_eastmoney(ticker_or_code: str, secid_prefix: int = 105) -> dict:
    """
    东财 push2 实时行情 — 美股+港股统一接口
    美股: stock_quote_eastmoney("AAPL", 105)  # NASDAQ
          stock_quote_eastmoney("BABA", 106)  # NYSE
    港股: stock_quote_eastmoney("00700", 116)
    返回: 最新价/开高低收/成交量/成交额/换手率/涨跌幅/中文名
    
    secid_prefix 说明: 105=NASDAQ, 106=NYSE, 107=US_ETF, 116=港股
    如不确定前缀，先调 stock_search() 获取 mkt_num
    """
    url = "https://push2.eastmoney.com/api/qt/stock/get"
    params = {
        "secid": f"{secid_prefix}.{ticker_or_code}",
        "fields": "f43,f44,f45,f46,f47,f48,f55,f57,f58,f59,f60,f170",
    }
    r = requests.get(url, params=params, timeout=10)
    d = r.json().get("data")
    if not d:
        return {}
    
    # f59 = 小数位数, 价格字段需除以 10^f59 还原真实值
    dec = d.get("f59", 3)
    divisor = 10 ** dec
    
    def _p(key):
        v = d.get(key)
        if v is None or v == "-":
            return None
        return round(v / divisor, dec)
    
    return {
        "code": d.get("f57"),           # 股票代码
        "name": d.get("f58"),           # 中文名
        "price": _p("f43"),             # 最新价
        "high": _p("f44"),              # 最高
        "low": _p("f45"),               # 最低
        "open": _p("f46"),              # 开盘
        "volume": d.get("f47"),         # 成交量(股)
        "amount": d.get("f48"),         # 成交额
        "turnover_rate": d.get("f55"),  # 换手率(%)
        "prev_close": _p("f60"),        # 昨收
        "change_pct": round(d["f170"] / 100, 2) if d.get("f170") is not None else None,  # 涨跌幅(%)
    }
```

---

## Layer 2: K线层

### 2.1 美股 K 线 — 新浪（主）+ Yahoo（备）

两个独立数据源。新浪最长可回溯到 1984 年；Yahoo 适合需要复权数据的场景。

> **注意：** 东财 push2his kline/get 端点实测不返回美股/港股数据（2026-05-20 验证），仅支持 A 股。美股/港股 K 线用新浪和 Yahoo。

```python
def us_stock_kline_sina(ticker: str, num: int = 120) -> list[dict]:
    """
    新浪美股日K — 可回溯到1984年
    ticker: 如 "AAPL"
    返回: [{date, open, high, low, close, volume}, ...]
    """
    url = "https://stock.finance.sina.com.cn/usstock/api/jsonp.php/var/US_MinKService.getDailyK"
    params = {"symbol": ticker.upper(), "num": num}
    r = requests.get(url, params=params, headers={"Referer": "https://finance.sina.com.cn/"}, timeout=15)
    text = r.text
    
    # 解析 JSONP: var=([{...},...])
    import json
    m = re.search(r'\((\[.+\])\)', text)
    if not m:
        return []
    
    items = json.loads(m.group(1))
    result = []
    for item in items:
        result.append({
            "date": item.get("d"),
            "open": float(item.get("o", 0)),
            "high": float(item.get("h", 0)),
            "low": float(item.get("l", 0)),
            "close": float(item.get("c", 0)),
            "volume": int(item.get("v", 0)),
        })
    return result


def stock_kline_yahoo(symbol: str, interval: str = "1d",
                       range_: str = "6mo") -> list[dict]:
    """
    Yahoo Finance chart API — 美股+港股通用，零crumb
    symbol: "AAPL" (美股) 或 "0700.HK" (港股)
    interval: "1d", "1wk", "1mo", "5m", "15m", "1h"
    range_: "1d", "5d", "1mo", "3mo", "6mo", "1y", "5y", "max"
    返回: [{date, open, high, low, close, volume}, ...]
    """
    url = f"https://query2.finance.yahoo.com/v8/finance/chart/{symbol}"
    params = {"interval": interval, "range": range_}
    r = requests.get(url, params=params, headers={
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
    }, timeout=15)
    r.raise_for_status()
    
    d = r.json()
    chart = d.get("chart", {}).get("result", [{}])[0]
    timestamps = chart.get("timestamp", [])
    quote = chart.get("indicators", {}).get("quote", [{}])[0]
    
    from datetime import datetime
    result = []
    for i, ts in enumerate(timestamps):
        result.append({
            "date": datetime.fromtimestamp(ts).strftime("%Y-%m-%d %H:%M") if "m" in interval or "h" in interval else datetime.fromtimestamp(ts).strftime("%Y-%m-%d"),
            "open": round(quote["open"][i], 2) if quote["open"][i] else 0,
            "high": round(quote["high"][i], 2) if quote["high"][i] else 0,
            "low": round(quote["low"][i], 2) if quote["low"][i] else 0,
            "close": round(quote["close"][i], 2) if quote["close"][i] else 0,
            "volume": int(quote["volume"][i]) if quote["volume"][i] else 0,
        })
    return result
```

### 2.2 港股 K 线 — Yahoo（唯一可用源）

港股 K 线只有 Yahoo 一个可用源（新浪港股K线已失效，东财 push2his 不返回港股K线数据）。

```python
# 港股 Yahoo K线: 直接调 stock_kline_yahoo("0700.HK")
```

---

## Layer 3: 技术指标层

基于 K 线 OHLCV 数据的纯 Python 技术指标计算，零额外依赖。

**使用方式：** 先调 K 线函数获取数据，再传入技术指标函数：
```python
klines = us_stock_kline_sina("AAPL", 120)
macd = calc_macd(klines)
rsi = calc_rsi(klines)
```

### 3.1 移动平均线 MA / EMA

```python
def _ema(values: list[float], period: int) -> list[float]:
    """EMA 指数移动平均（内部辅助）"""
    result = [values[0]]
    k = 2 / (period + 1)
    for v in values[1:]:
        result.append(v * k + result[-1] * (1 - k))
    return result


def calc_ma(klines: list[dict], periods: list[int] = None) -> list[dict]:
    """
    移动平均线 MA + EMA
    klines: K线数据 [{date, open, high, low, close, volume}, ...]
    periods: 周期列表，默认 [5, 10, 20, 60]
    返回: [{date, close, ma5, ma10, ma20, ma60, ema12, ema26}, ...]
    """
    if periods is None:
        periods = [5, 10, 20, 60]
    closes = [k["close"] for k in klines]
    
    # EMA 12/26（MACD 常用）
    ema12 = _ema(closes, 12)
    ema26 = _ema(closes, 26)
    
    result = []
    for i, k in enumerate(klines):
        row = {"date": k["date"], "close": k["close"]}
        for p in periods:
            if i >= p - 1:
                row[f"ma{p}"] = round(sum(closes[i - p + 1:i + 1]) / p, 4)
            else:
                row[f"ma{p}"] = None
        row["ema12"] = round(ema12[i], 4)
        row["ema26"] = round(ema26[i], 4)
        result.append(row)
    return result
```

### 3.2 MACD

```python
def calc_macd(klines: list[dict], fast: int = 12, slow: int = 26,
              signal: int = 9) -> list[dict]:
    """
    MACD (Moving Average Convergence Divergence)
    klines: K线数据
    fast/slow/signal: 快线/慢线/信号线周期（默认 12/26/9）
    返回: [{date, close, dif, dea, macd_hist}, ...]
    
    dif = EMA(fast) - EMA(slow)        金叉/死叉看 dif 穿越 dea
    dea = EMA(signal) of dif           信号线
    macd_hist = (dif - dea) * 2        柱状图（红涨绿跌）
    """
    closes = [k["close"] for k in klines]
    ema_fast = _ema(closes, fast)
    ema_slow = _ema(closes, slow)
    
    dif = [round(f - s, 4) for f, s in zip(ema_fast, ema_slow)]
    dea = _ema(dif, signal)
    
    result = []
    for i, k in enumerate(klines):
        result.append({
            "date": k["date"],
            "close": k["close"],
            "dif": round(dif[i], 4),
            "dea": round(dea[i], 4),
            "macd_hist": round((dif[i] - dea[i]) * 2, 4),
        })
    return result
```

### 3.3 RSI

```python
def calc_rsi(klines: list[dict],
             periods: list[int] = None) -> list[dict]:
    """
    RSI (Relative Strength Index)
    klines: K线数据
    periods: 周期列表（默认 [6, 12, 24]）
    返回: [{date, close, rsi6, rsi12, rsi24}, ...]
    
    RSI > 70 超买区（可能回调）
    RSI < 30 超卖区（可能反弹）
    """
    if periods is None:
        periods = [6, 12, 24]
    closes = [k["close"] for k in klines]
    
    # 涨跌额序列
    changes = [0.0] + [closes[i] - closes[i - 1] for i in range(1, len(closes))]
    gains = [max(c, 0) for c in changes]
    losses = [max(-c, 0) for c in changes]
    
    result = []
    for i, k in enumerate(klines):
        row = {"date": k["date"], "close": k["close"]}
        for p in periods:
            if i < p:
                row[f"rsi{p}"] = None
                continue
            avg_gain = sum(gains[i - p + 1:i + 1]) / p
            avg_loss = sum(losses[i - p + 1:i + 1]) / p
            if avg_loss == 0:
                row[f"rsi{p}"] = 100.0
            else:
                rs = avg_gain / avg_loss
                row[f"rsi{p}"] = round(100 - 100 / (1 + rs), 2)
        result.append(row)
    return result
```

### 3.4 KDJ

```python
def calc_kdj(klines: list[dict], n: int = 9,
             m1: int = 3, m2: int = 3) -> list[dict]:
    """
    KDJ 随机指标
    klines: K线数据
    n: RSV 周期（默认9）
    m1/m2: K/D 平滑系数（默认3/3）
    返回: [{date, close, k, d, j}, ...]
    
    K/D > 80 超买，K/D < 20 超卖
    J > 100 或 J < 0 为极端信号
    金叉: K 上穿 D；死叉: K 下穿 D
    """
    k_val, d_val = 50.0, 50.0
    result = []
    
    for i, kline in enumerate(klines):
        if i < n - 1:
            result.append({"date": kline["date"], "close": kline["close"],
                           "k": None, "d": None, "j": None})
            continue
        
        window = klines[i - n + 1:i + 1]
        high_n = max(w["high"] for w in window)
        low_n = min(w["low"] for w in window)
        
        rsv = (kline["close"] - low_n) / (high_n - low_n) * 100 if high_n != low_n else 50.0
        k_val = (1 / m1) * rsv + (1 - 1 / m1) * k_val
        d_val = (1 / m2) * k_val + (1 - 1 / m2) * d_val
        j_val = 3 * k_val - 2 * d_val
        
        result.append({
            "date": kline["date"],
            "close": kline["close"],
            "k": round(k_val, 2),
            "d": round(d_val, 2),
            "j": round(j_val, 2),
        })
    return result
```

### 3.5 布林带

```python
def calc_boll(klines: list[dict], period: int = 20,
              num_std: float = 2.0) -> list[dict]:
    """
    布林带 (Bollinger Bands)
    klines: K线数据
    period: 中轨 MA 周期（默认20）
    num_std: 标准差倍数（默认2）
    返回: [{date, close, upper, middle, lower, bandwidth}, ...]
    
    价格触及 upper → 可能超买
    价格触及 lower → 可能超卖
    bandwidth 收窄 → 即将变盘
    """
    closes = [k["close"] for k in klines]
    result = []
    
    for i, k in enumerate(klines):
        if i < period - 1:
            result.append({"date": k["date"], "close": k["close"],
                           "upper": None, "middle": None, "lower": None,
                           "bandwidth": None})
            continue
        
        window = closes[i - period + 1:i + 1]
        ma = sum(window) / period
        std = (sum((x - ma) ** 2 for x in window) / period) ** 0.5
        upper = ma + num_std * std
        lower = ma - num_std * std
        
        result.append({
            "date": k["date"],
            "close": k["close"],
            "upper": round(upper, 4),
            "middle": round(ma, 4),
            "lower": round(lower, 4),
            "bandwidth": round((upper - lower) / ma * 100, 2) if ma else None,
        })
    return result
```

---

## Layer 4: 基本面层

### 4.1 财报三表 — 东财 datacenter

东财 datacenter 提供美股/港股的资产负债表、利润表、现金流量表，中文字段名，按科目行展开。

```python
def financial_statements_eastmoney(secucode: str, statement: str = "balance",
                                     page_size: int = 200) -> list[dict]:
    """
    东财 datacenter 财报三表
    secucode: "AAPL.O" (NASDAQ) / "BABA.N" (NYSE) / "00700.HK" (港股)
    statement: "balance" / "income" / "cashflow"
    返回: [{ITEM_NAME, AMOUNT, YOY_RATIO, REPORT, REPORT_DATE, ...}, ...]
    
    注意: 数据按科目行展开，每行一个科目（如"流动资产合计"、"营业收入"等），
    同一期报告有多行。用 REPORT_DATE 分组可还原整张报表。
    """
    # 报表名映射（注意命名不统一：balance/income 用 F10，cashflow 用 SK）
    report_map = {
        "balance": {"us": "RPT_USF10_FN_BALANCE", "hk": "RPT_HKF10_FN_BALANCE"},
        "income":  {"us": "RPT_USF10_FN_INCOME",  "hk": "RPT_HKF10_FN_INCOME"},
        "cashflow": {"us": "RPT_USSK_FN_CASHFLOW", "hk": "RPT_HKSK_FN_CASHFLOW"},
    }
    
    market = "hk" if secucode.endswith(".HK") else "us"
    report_name = report_map[statement][market]
    
    return eastmoney_datacenter(
        report_name=report_name,
        filter_str=f'(SECUCODE="{secucode}")',
        page_size=page_size,
        sort_columns="REPORT_DATE",
        sort_types="-1",
    )
    # 每行字段:
    # SECUCODE, SECURITY_CODE, SECURITY_NAME_ABBR, REPORT_DATE,
    # STD_ITEM_CODE, ITEM_NAME (科目名), AMOUNT (金额),
    # YOY_RATIO (同比%), REPORT (如 "2026/Q2"), REPORT_TYPE,
    # ACCOUNT_STANDARD (如 "美国会计准则"/"国际会计准则"),
    # CURRENCY (如 "美元"/"人民币")
```

### 4.2 关键财务指标(中文) — 东财 GMAININDICATOR

东财 datacenter 的 GMAININDICATOR 报表，提供中文关键财务指标概览。美股 49 字段、港股 75 字段，包含 ROE/ROA/EPS/毛利率/资产负债率/流动比率等，按季度报告。

```python
def key_indicators_eastmoney(secucode: str, page_size: int = 4) -> list[dict]:
    """
    东财 GMAININDICATOR 关键财务指标（中文）
    secucode: "AAPL.O" (NASDAQ) / "BABA.N" (NYSE) / "00700.HK" (港股)
    page_size: 返回最近几期报告（默认4期=一年）
    返回: [{REPORT_DATE, OPERATE_INCOME, BASIC_EPS, ROE_AVG, ROA, ...}, ...]
    
    美股核心字段(49): OPERATE_INCOME(营收), GROSS_PROFIT(毛利), GROSS_PROFIT_RATIO(毛利率%),
      PARENT_HOLDER_NETPROFIT(归母净利), NET_PROFIT_RATIO(净利率%), BASIC_EPS, DILUTED_EPS,
      ROE_AVG(平均ROE%), ROA(%), CURRENT_RATIO(流动比率), DEBT_ASSET_RATIO(资产负债率%),
      OPERATE_INCOME_YOY(营收同比%), BASIC_EPS_YOY(EPS同比%)
    
    港股额外字段(75): BPS(每股净资产), ROIC(投入资本回报率), EQUITY_RATIO(产权比率),
      HOLDER_PROFIT(股东应占溢利), OCF_SALES(经营现金流/营收%), DPS_HKD(每股股息),
      DIVI_RATIO(股息率%), PER_NETCASH_OPERATE(每股经营现金流)
    """
    market = "hk" if secucode.endswith(".HK") else "us"
    report_name = f"RPT_{'HK' if market == 'hk' else 'US'}F10_FN_GMAININDICATOR"
    
    return eastmoney_datacenter(
        report_name=report_name,
        filter_str=f'(SECUCODE="{secucode}")',
        page_size=page_size,
        sort_columns="REPORT_DATE",
        sort_types="-1",
    )
```

### 4.3 关键财务指标(英文) — Yahoo quoteSummary

Yahoo quoteSummary 的 `financialData` + `defaultKeyStatistics` 模块提供最核心的估值指标。

```python
def key_statistics(symbol: str) -> dict:
    """
    Yahoo 关键财务指标
    symbol: "AAPL" (美股) 或 "0700.HK" (港股)
    返回: PE/PB/EV/EBITDA/利润率/目标价/ROE/Beta 等
    """
    data = yahoo_quote_summary(symbol, ["financialData", "defaultKeyStatistics", "summaryDetail"])
    
    fd = data.get("financialData", {})
    ks = data.get("defaultKeyStatistics", {})
    sd = data.get("summaryDetail", {})
    
    def _val(d, key):
        v = d.get(key, {})
        return v.get("raw") if isinstance(v, dict) else v
    
    return {
        # 价格相关
        "current_price": _val(fd, "currentPrice"),
        "target_high": _val(fd, "targetHighPrice"),
        "target_low": _val(fd, "targetLowPrice"),
        "target_mean": _val(fd, "targetMeanPrice"),
        "recommendation": fd.get("recommendationKey"),  # buy/hold/sell
        
        # 估值指标
        "trailing_pe": _val(sd, "trailingPE"),
        "forward_pe": _val(ks, "forwardPE"),
        "peg_ratio": _val(ks, "pegRatio"),
        "price_to_book": _val(ks, "priceToBook"),
        "enterprise_value": _val(ks, "enterpriseValue"),
        "ev_to_ebitda": _val(ks, "enterpriseToEbitda"),
        "ev_to_revenue": _val(ks, "enterpriseToRevenue"),
        
        # 盈利能力
        "profit_margin": _val(ks, "profitMargins"),
        "operating_margin": _val(fd, "operatingMargins"),
        "gross_margin": _val(fd, "grossMargins"),
        "return_on_equity": _val(fd, "returnOnEquity"),
        "return_on_assets": _val(fd, "returnOnAssets"),
        
        # 成长性
        "earnings_growth": _val(fd, "earningsGrowth"),
        "revenue_growth": _val(fd, "revenueGrowth"),
        
        # 风险
        "beta": _val(ks, "beta"),
        "short_ratio": _val(ks, "shortRatio"),
        
        # 股息
        "dividend_yield": _val(sd, "dividendYield"),
        "payout_ratio": _val(ks, "payoutRatio"),
        
        # 规模
        "market_cap": _val(sd, "marketCap"),
        "total_revenue": _val(fd, "totalRevenue"),
        "total_cash": _val(fd, "totalCash"),
        "total_debt": _val(fd, "totalDebt"),
    }
```

### 4.4 分析师预期与评级 — Yahoo quoteSummary

```python
def analyst_estimates(symbol: str) -> dict:
    """
    Yahoo 分析师预期 — EPS预测/评级趋势/升降级历史
    symbol: "AAPL" 或 "0700.HK"
    """
    data = yahoo_quote_summary(symbol, [
        "earningsTrend", "recommendationTrend", "upgradeDowngradeHistory",
        "earnings", "earningsHistory",
    ])
    
    # EPS 趋势
    et = data.get("earningsTrend", {}).get("trend", [])
    eps_trend = []
    for t in et:
        eps_trend.append({
            "period": t.get("period"),
            "end_date": t.get("endDate"),
            "eps_estimate": t.get("earningsEstimate", {}).get("avg", {}).get("raw"),
            "eps_high": t.get("earningsEstimate", {}).get("high", {}).get("raw"),
            "eps_low": t.get("earningsEstimate", {}).get("low", {}).get("raw"),
            "revenue_estimate": t.get("revenueEstimate", {}).get("avg", {}).get("raw"),
            "num_analysts": t.get("earningsEstimate", {}).get("numberOfAnalysts", {}).get("raw"),
        })
    
    # 评级趋势 (最近4个月)
    rt = data.get("recommendationTrend", {}).get("trend", [])
    rating_trend = []
    for r_ in rt:
        rating_trend.append({
            "period": r_.get("period"),
            "strong_buy": r_.get("strongBuy"),
            "buy": r_.get("buy"),
            "hold": r_.get("hold"),
            "sell": r_.get("sell"),
            "strong_sell": r_.get("strongSell"),
        })
    
    # 升降级历史 (最近20条)
    udh = data.get("upgradeDowngradeHistory", {}).get("history", [])[:20]
    upgrades = []
    for u in udh:
        upgrades.append({
            "date": u.get("epochGradeDate"),
            "firm": u.get("firm"),
            "to_grade": u.get("toGrade"),
            "from_grade": u.get("fromGrade"),
            "action": u.get("action"),  # up/down/main/init
        })
    
    return {
        "eps_trend": eps_trend,
        "rating_trend": rating_trend,
        "upgrade_downgrade": upgrades,
    }
```

### 4.5 机构持仓 — Yahoo quoteSummary

```python
def institutional_holders(symbol: str) -> dict:
    """
    Yahoo 机构持仓 — 前10大机构 + 内部人持股比例
    symbol: "AAPL" 或 "0700.HK"
    """
    data = yahoo_quote_summary(symbol, ["institutionOwnership", "majorHoldersBreakdown"])
    
    # 持股比例总览
    mhb = data.get("majorHoldersBreakdown", {})
    def _val(d, key):
        v = d.get(key, {})
        return v.get("raw") if isinstance(v, dict) else v
    
    overview = {
        "insiders_pct": _val(mhb, "insidersPercentHeld"),
        "institutions_pct": _val(mhb, "institutionsPercentHeld"),
        "institutions_float_pct": _val(mhb, "institutionsFloatPercentHeld"),
        "institutions_count": _val(mhb, "institutionsCount"),
    }
    
    # 前10大机构
    io = data.get("institutionOwnership", {}).get("ownershipList", [])
    top_holders = []
    for h in io[:10]:
        top_holders.append({
            "name": h.get("organization"),
            "shares": _val(h, "position"),
            "value": _val(h, "value"),
            "pct_held": _val(h, "pctHeld"),
            "report_date": h.get("reportDate", {}).get("fmt") if isinstance(h.get("reportDate"), dict) else None,
        })
    
    return {"overview": overview, "top_holders": top_holders}
```

### 4.6 年度/季度财报明细 — Yahoo quoteSummary

东财 datacenter 按科目行展开，Yahoo 直接返回完整报表结构，两个互补。

```python
def financial_statements_yahoo(symbol: str,
                                 quarterly: bool = False) -> dict:
    """
    Yahoo 财报三表 — 结构化完整报表
    symbol: "AAPL" 或 "0700.HK"
    quarterly: False=年度, True=季度
    返回: {"income": [...], "balance": [...], "cashflow": [...]}
    """
    suffix = "Quarterly" if quarterly else ""
    data = yahoo_quote_summary(symbol, [
        f"incomeStatementHistory{suffix}",
        f"balanceSheetHistory{suffix}",
        f"cashflowStatementHistory{suffix}",
    ])
    
    def _extract(statements):
        result = []
        for stmt in statements:
            row = {}
            for k, v in stmt.items():
                if isinstance(v, dict) and "raw" in v:
                    row[k] = v["raw"]
                elif isinstance(v, dict) and "fmt" in v:
                    row[k] = v["fmt"]
                else:
                    row[k] = v
            result.append(row)
        return result
    
    income_key = f"incomeStatementHistory{suffix}"
    balance_key = f"balanceSheetHistory{suffix}"
    cashflow_key = f"cashflowStatementHistory{suffix}"
    
    return {
        "income": _extract(data.get(income_key, {}).get("incomeStatementHistory", [])),
        "balance": _extract(data.get(balance_key, {}).get("balanceSheetStatements", [])),
        "cashflow": _extract(data.get(cashflow_key, {}).get("cashflowStatements", [])),
    }
```

---

## Layer 5: 资金面层

### 5.1 日级资金流 — 东财 push2his

```python
def fund_flow_daily(ticker_or_code: str, secid_prefix: int = 105,
                      limit: int = 100) -> list[dict]:
    """
    东财 push2his 日级资金流 — 主力/大单/中单/小单净流入
    美股: fund_flow_daily("AAPL", 105)  # NASDAQ
          fund_flow_daily("BABA", 106)  # NYSE
    港股: fund_flow_daily("00700", 116)
    返回: [{date, main_net, big_net, mid_net, small_net, main_pct, ...}, ...]
    """
    url = "https://push2his.eastmoney.com/api/qt/stock/fflow/daykline/get"
    params = {
        "secid": f"{secid_prefix}.{ticker_or_code}",
        "klt": 101,
        "fields1": "f1,f2,f3,f7",
        "fields2": "f51,f52,f53,f54,f55,f56,f57",
        "lmt": limit,
    }
    r = requests.get(url, params=params, timeout=15)
    d = r.json()
    data = d.get("data")
    if not data or not data.get("klines"):
        return []
    
    result = []
    for line in data["klines"]:
        parts = line.split(",")
        # f51=日期, f52=主力净流入, f53=小单净流入, f54=中单净流入, f55=大单净流入, f56=超大单净流入
        result.append({
            "date": parts[0],
            "main_net": float(parts[1]),       # 主力净流入（元）
            "small_net": float(parts[2]),       # 小单净流入
            "mid_net": float(parts[3]),         # 中单净流入
            "big_net": float(parts[4]),         # 大单净流入
            "super_big_net": float(parts[5]),   # 超大单净流入
            "main_pct": float(parts[6]) if len(parts) > 6 and parts[6] else 0,  # 主力净占比%
        })
    return result
```

---

## Layer 6: 期权层

### 6.1 期权链 — Yahoo Finance

```python
def options_chain(symbol: str, expiration: int = None) -> dict:
    """
    Yahoo 期权链 — calls + puts 完整数据（仅美股）
    symbol: "AAPL", "TSLA" 等美股 ticker
    ⚠️ 港股(如0700.HK)期权不在Yahoo覆盖范围，调用会返回空列表
    expiration: Unix timestamp (不传则返回最近到期日 + 所有到期日列表)
    返回: {"expiration_dates": [...], "calls": [...], "puts": [...]}
    """
    s = get_yahoo_session()
    params = {"crumb": s._crumb}
    if expiration:
        params["date"] = expiration
    
    r = s.get(f"https://query2.finance.yahoo.com/v7/finance/options/{symbol}",
              params=params, timeout=15)
    r.raise_for_status()
    
    oc = r.json().get("optionChain", {}).get("result", [{}])[0]
    
    exp_dates = oc.get("expirationDates", [])
    options = oc.get("options", [{}])[0] if oc.get("options") else {}
    
    def _parse_options(opts):
        result = []
        for o in opts:
            def _val(key):
                v = o.get(key, {})
                return v.get("raw") if isinstance(v, dict) else v
            result.append({
                "strike": _val("strike"),
                "last_price": _val("lastPrice"),
                "bid": _val("bid"),
                "ask": _val("ask"),
                "volume": _val("volume"),
                "open_interest": _val("openInterest"),
                "implied_volatility": _val("impliedVolatility"),
                "in_the_money": o.get("inTheMoney"),
                "expiration": o.get("expiration", {}).get("fmt") if isinstance(o.get("expiration"), dict) else None,
                "contract_symbol": o.get("contractSymbol"),
            })
        return result
    
    return {
        "expiration_dates": exp_dates,  # Unix timestamps, 可依次传入获取各期
        "calls": _parse_options(options.get("calls", [])),
        "puts": _parse_options(options.get("puts", [])),
        "underlying_price": oc.get("quote", {}).get("regularMarketPrice"),
    }
```

---

## Layer 7: SEC Filing 层（仅美股）

### 7.1 SEC Filing 列表 — EDGAR submissions

```python
SEC_HEADERS = {"User-Agent": "SimonLin global-stock-data/1.0 (contact@example.com)"}

def sec_filings(cik: str, form_type: str = None) -> dict:
    """
    SEC EDGAR Filing 列表
    cik: CIK号（10位补零），如 "0000320193" (Apple)
         可通过 ticker_to_cik() 从 ticker 转换
    form_type: 筛选类型，如 "10-K", "10-Q", "8-K"（不传返回全部）
    返回: {"company_name": ..., "filings": [{form, date, accession_number, primary_document}, ...]}
    """
    url = f"https://data.sec.gov/submissions/CIK{cik}.json"
    r = requests.get(url, headers=SEC_HEADERS, timeout=15)
    r.raise_for_status()
    
    data = r.json()
    recent = data.get("filings", {}).get("recent", {})
    
    forms = recent.get("form", [])
    dates = recent.get("filingDate", [])
    accessions = recent.get("accessionNumber", [])
    primary_docs = recent.get("primaryDocument", [])
    descriptions = recent.get("primaryDocDescription", [])
    
    filings = []
    for i in range(len(forms)):
        if form_type and forms[i] != form_type:
            continue
        filings.append({
            "form": forms[i],
            "date": dates[i],
            "accession_number": accessions[i],
            "primary_document": primary_docs[i] if i < len(primary_docs) else "",
            "description": descriptions[i] if i < len(descriptions) else "",
            "url": f"https://www.sec.gov/Archives/edgar/data/{int(cik)}/{accessions[i].replace('-', '')}/{primary_docs[i]}" if i < len(primary_docs) and primary_docs[i] else "",
        })
    
    return {
        "company_name": data.get("name"),
        "cik": cik,
        "ticker": data.get("tickers", [""])[0] if data.get("tickers") else "",
        "filings": filings[:50],  # 最近50条
    }
```

### 7.2 SEC XBRL 结构化财务数据 — EDGAR companyfacts

覆盖 503 个 GAAP 指标，可精确提取多年营收/净利/EPS/资产/负债等。

```python
def sec_xbrl_facts(cik: str, metrics: list[str] = None) -> dict:
    """
    SEC EDGAR XBRL 结构化财务数据
    cik: CIK号（10位补零）
    metrics: 要提取的指标名，如 ["RevenueFromContractWithCustomerExcludingAssessedTax",
             "NetIncomeLoss", "EarningsPerShareDiluted"]
             不传则返回所有可用指标名列表
    
    返回: {"company": ..., "metrics": {"Revenue": [{end, val, form, filed}, ...], ...}}
    """
    url = f"https://data.sec.gov/api/xbrl/companyfacts/CIK{cik}.json"
    r = requests.get(url, headers=SEC_HEADERS, timeout=15)
    r.raise_for_status()
    
    facts = r.json()
    us_gaap = facts.get("facts", {}).get("us-gaap", {})
    
    # 如果不传 metrics，返回所有可用指标
    if not metrics:
        available = []
        for k, v in us_gaap.items():
            label = v.get("label", k)
            units = list(v.get("units", {}).keys())
            available.append({"name": k, "label": label, "units": units})
        return {
            "company": facts.get("entityName"),
            "total_metrics": len(available),
            "available_metrics": available,
        }
    
    # 提取指定指标
    result = {}
    for metric_name in metrics:
        metric = us_gaap.get(metric_name, {})
        if not metric:
            result[metric_name] = []
            continue
        
        # 自动选择单位（USD 或 USD/shares）
        units = metric.get("units", {})
        unit_key = "USD" if "USD" in units else list(units.keys())[0] if units else None
        if not unit_key:
            result[metric_name] = []
            continue
        
        entries = units[unit_key]
        # 只取 10-K 和 10-Q
        filtered = [e for e in entries if e.get("form") in ("10-K", "10-Q")]
        result[metric_name] = [{
            "end": e.get("end"),
            "val": e.get("val"),
            "form": e.get("form"),
            "filed": e.get("filed"),
            "fy": e.get("fy"),
            "fp": e.get("fp"),
        } for e in filtered[-20:]]  # 最近20条
    
    return {
        "company": facts.get("entityName"),
        "metrics": result,
    }
```

**常用 XBRL 指标名速查：**

| 指标 | XBRL 名 |
|------|---------|
| 营业收入 | `RevenueFromContractWithCustomerExcludingAssessedTax` 或 `Revenues` |
| 净利润 | `NetIncomeLoss` |
| 稀释 EPS | `EarningsPerShareDiluted` |
| 基本 EPS | `EarningsPerShareBasic` |
| 总资产 | `Assets` |
| 总负债 | `Liabilities` |
| 股东权益 | `StockholdersEquity` |
| 经营现金流 | `NetCashProvidedByOperatingActivities` |
| 研发费用 | `ResearchAndDevelopmentExpense` |
| 股份回购 | `PaymentsForRepurchaseOfCommonStock` |
| 股息支付 | `PaymentsOfDividends` |

---

## Layer 8: 工具层

### 8.1 股票搜索 — 东财 search API

```python
def stock_search(keyword: str, count: int = 10) -> list[dict]:
    """
    东财股票搜索 — 支持中英文，返回代码+市场+中文名
    keyword: "AAPL" / "苹果" / "Tencent" / "00700" / "特斯拉"
    返回: [{code, name, mkt_num, market_name, security_type}, ...]
    
    mkt_num 即 push2/push2his 的 secid 前缀:
    105=NASDAQ, 106=NYSE, 107=美股ETF, 116=港股
    """
    url = "https://searchapi.eastmoney.com/api/suggest/get"
    params = {
        "input": keyword,
        "type": 14,  # 14=全球市场
        "token": "D43BF722C8E33BDC906FB84D85E326E8",
        "count": count,
    }
    r = requests.get(url, params=params, timeout=10)
    d = r.json()
    
    suggestions = d.get("QuotationCodeTable", {}).get("Data", [])
    result = []
    for s in suggestions:
        mkt = s.get("MktNum", "")
        # 只保留美股和港股
        if str(mkt) not in ("105", "106", "107", "116"):
            continue
        
        market_map = {"105": "NASDAQ", "106": "NYSE", "107": "US_OTHER", "116": "HK"}
        result.append({
            "code": s.get("Code"),
            "name": s.get("Name"),
            "mkt_num": int(mkt),
            "market_name": market_map.get(str(mkt), str(mkt)),
            "security_type": s.get("SecurityTypeName"),
        })
    return result
```

### 8.2 股票新闻 — Yahoo Finance search

```python
def stock_news(keyword: str, count: int = 10) -> list[dict]:
    """
    Yahoo Finance 新闻搜索
    keyword: 股票代码或关键词，如 "AAPL", "Tesla", "0700.HK"
    返回: [{title, publisher, link, publish_time, thumbnail}, ...]
    注意: 需要先获取 Yahoo cookie 才能调用，否则返回 400
    """
    s = requests.Session()
    s.headers["User-Agent"] = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
    s.get("https://fc.yahoo.com", timeout=10)  # 获取 cookie
    
    url = "https://query2.finance.yahoo.com/v1/finance/search"
    params = {"q": keyword, "quotesCount": 0, "newsCount": count}
    r = s.get(url, params=params, timeout=10)
    r.raise_for_status()
    
    news = r.json().get("news", [])
    result = []
    for n in news:
        result.append({
            "title": n.get("title"),
            "publisher": n.get("publisher"),
            "link": n.get("link"),
            "publish_time": n.get("providerPublishTime"),
            "thumbnail": n.get("thumbnail", {}).get("resolutions", [{}])[0].get("url") if n.get("thumbnail") else None,
        })
    return result
```

### 8.3 Ticker → CIK 映射 — SEC EDGAR（仅美股）

```python
_cik_cache = None

def ticker_to_cik(ticker: str) -> dict:
    """
    SEC EDGAR ticker → CIK 映射
    ticker: 如 "AAPL", "TSLA", "MSFT"
    返回: {"ticker": "AAPL", "cik": "0000320193", "company": "Apple Inc."}
    
    首次调用下载完整映射表(~10KB JSON, 10000+公司)并缓存。
    """
    global _cik_cache
    if not _cik_cache:
        r = requests.get("https://www.sec.gov/files/company_tickers.json",
                         headers=SEC_HEADERS, timeout=15)
        r.raise_for_status()
        _cik_cache = r.json()
    
    ticker_upper = ticker.upper()
    for _, v in _cik_cache.items():
        if v.get("ticker") == ticker_upper:
            cik_str = str(v["cik_str"]).zfill(10)
            return {
                "ticker": ticker_upper,
                "cik": cik_str,
                "company": v.get("title"),
            }
    return {}
```

### 8.4 全市场股票列表 — 东财 push2

```python
def market_stock_list(market: str = "us_nasdaq", sort_field: str = "f3",
                       sort_desc: bool = True, page: int = 1,
                       page_size: int = 20) -> dict:
    """
    东财 push2 全市场股票列表 — 涨跌幅/成交量/成交额排名
    market: "us_nasdaq" (m:105), "us_nyse" (m:106), "hk" (m:116)
    sort_field: 排序字段
      f3=涨跌幅, f5=成交量, f6=成交额, f2=最新价, f7=振幅, f15=最高, f16=最低
    sort_desc: True=降序(默认), False=升序
    page/page_size: 分页（默认第1页，每页20条）
    返回: {"total": 5925, "stocks": [{code, name, price, change_pct, volume, ...}, ...]}
    
    典型用途:
    - 今日涨幅 TOP 20: market_stock_list("us_nasdaq", "f3", True)
    - 今日跌幅 TOP 20: market_stock_list("us_nasdaq", "f3", False)
    - 成交量 TOP 20: market_stock_list("hk", "f5", True)
    - 遍历全市场: 循环 page=1..N, 每页100条做筛选
    """
    market_map = {"us_nasdaq": "m:105", "us_nyse": "m:106", "us_etf": "m:107", "hk": "m:116"}
    fs = market_map.get(market, market)
    
    url = "https://push2.eastmoney.com/api/qt/clist/get"
    params = {
        "fs": fs,
        "fields": "f2,f3,f4,f5,f6,f7,f12,f14,f15,f16,f17,f18",
        "pn": page,
        "pz": page_size,
        "fid": sort_field,
        "po": 1 if sort_desc else 0,
    }
    r = requests.get(url, params=params, timeout=15)
    d = r.json()
    data = d.get("data", {})
    
    total = data.get("total", 0)
    diff = data.get("diff", [])
    # 东财 push2 的 diff 有时是 list、有时是按序号为键的 dict（如 {"0":{...},"1":{...}}）。
    # 直接 for item in diff 遇到 dict 会拿到字符串键 → AttributeError，统一成列表。
    if isinstance(diff, dict):
        diff = list(diff.values())
    
    stocks = []
    for item in diff:
        stocks.append({
            "code": item.get("f12"),         # 股票代码
            "name": item.get("f14"),         # 中文名
            "price": item.get("f2"),         # 最新价(原始值, 需÷10^小数位)
            "change_pct": round(item["f3"] / 100, 2) if item.get("f3") is not None else None,  # 涨跌幅(%)
            "change_amount": item.get("f4"), # 涨跌额(原始值)
            "volume": item.get("f5"),        # 成交量(股)
            "amount": item.get("f6"),        # 成交额
            "amplitude": round(item["f7"] / 100, 2) if item.get("f7") is not None else None,  # 振幅(%)
            "high": item.get("f15"),         # 最高(原始值)
            "low": item.get("f16"),          # 最低(原始值)
            "open": item.get("f17"),         # 开盘(原始值)
            "prev_close": item.get("f18"),   # 昨收(原始值)
        })
    
    return {"total": total, "stocks": stocks}
```

---

## 数据源优先级

| 场景 | 第一优先 | 备选 | 说明 |
|------|---------|------|------|
| 美股行情 | 新浪 `gb_XXXX` | 腾讯 / 东财 push2 | 新浪有中文名+EPS+PE |
| 港股行情 | 腾讯 `r_hkXXXXX` | 新浪 / 东财 push2 | 腾讯字段最全(78个) |
| 美股K线 | 新浪 | Yahoo chart | 新浪回溯至1984年；Yahoo支持多周期 |
| 港股K线 | Yahoo chart | — | 新浪港股K线已失效；push2his不返回港股K线 |
| 财报三表(中文) | 东财 datacenter | — | 中文科目名，按行展开 |
| 财报三表(结构化) | Yahoo quoteSummary | — | 英文，完整报表结构 |
| 关键指标(中文) | 东财 GMAININDICATOR | — | ROE/ROA/EPS/毛利率/资产负债率 (美49/港75字段) |
| 关键指标(英文) | Yahoo quoteSummary | — | PE/PB/EV/利润率/目标价 |
| 分析师预期 | Yahoo quoteSummary | — | EPS预测+评级+升降级 |
| 机构持仓 | Yahoo quoteSummary | — | 前10大机构+内部人 |
| 资金流 | 东财 push2his | — | 日级主力/大单/中单/小单 |
| 期权链 | Yahoo options | — | 仅美股；港股期权需港交所专有接口 |
| SEC Filing | EDGAR | — | 官方数据，仅美股 |
| XBRL财务 | EDGAR | — | 503个GAAP指标 |
| 搜索 | 东财 search | Yahoo search | 东财有 secid 映射 |
| 新闻 | Yahoo search | — | 唯一稳定的新闻源 |
| 全市场列表 | 东财 push2 clist | — | 涨跌幅/成交量排名，美股5925+港股18000+ |

---

## 数据源汇总

| 数据源 | 协议 | 鉴权 | 覆盖 |
|--------|------|------|------|
| 东财 push2 | HTTPS | 零 | 美股+港股 实时行情+全市场列表 |
| 东财 push2his | HTTPS | 零 | 美股+港股 资金流（K线仅A股，不覆盖美股/港股） |
| 东财 datacenter | HTTPS | 零 | 美股+港股 财报三表+GMAININDICATOR关键指标 |
| 东财 search API | HTTPS | 零 | 全球股票搜索+secid映射 |
| Yahoo Finance | HTTPS | cookie+crumb(自动) | 美股+港股 全品类 |
| 新浪财经 | HTTP | 零 | 美股+港股 行情、美股K线 |
| 腾讯财经 | HTTPS | 零 | 美股+港股 行情 |
| SEC EDGAR | HTTPS | 零(需UA) | 美股 Filing+XBRL |

> 📦 https://github.com/simonlin1212/global-stock-data — Star ⭐ 是最好的支持
