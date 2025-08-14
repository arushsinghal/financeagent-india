# IndiaStock-Agent (Python SDK)

> **India-first finance SDK** for live NSE snapshots, 5-year OHLCV history, corporate announcements, India-focused news aggregation, and a simple **Buy/Watch/Avoid** verdict helper.  
> Ships as a Python package; easy to wrap behind FastAPI for a public API.

[![PyPI](https://img.shields.io/pypi/v/indiastock-agent.svg)](https://pypi.org/project/indiastock-agent/)
[![Python](https://img.shields.io/pypi/pyversions/indiastock-agent.svg)](https://pypi.org/project/indiastock-agent/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](#license)
[![Tests](https://img.shields.io/github/actions/workflow/status/<your-gh-username>/india-stock-agent/ci.yml?branch=main)](https://github.com/<your-gh-username>/india-stock-agent/actions)

---

## ✨ What’s inside

- **Snapshot (live-ish)** for NSE tickers (price, % change, basic ratios)  
- **History (5y OHLCV)** via `yfinance` for `.NS` / `.BO`  
- **Announcements**: NSE/BSE latest headlines + links  
- **News (India)**: Moneycontrol / ETMarkets / LiveMint / Business Standard / Reuters India (RSS/NewsAPI)  
- **Verdict helper**: simple, explainable **Buy/Watch/Avoid**  
- **Resolver** (optional): name → NSE ticker  
- **Caching + fallback**: tolerate fetch hiccups and rate limits

> **Scope:** This build is intentionally **India-only** (NSE/BSE). Other markets are disabled by design.

---

## 🧱 Install

```bash
pip install -U indiastock-agent
# optional extras
pip install -U yfinance
# if you use NewsAPI (optional)
# pip install newsapi-python



📦 Public API

indiastock_agent.api(symbol_list: list[str], market: str = "NSE_INDIA") -> dict | list[dict]

indiastock_agent.india.resolve(name: str) -> str
indiastock_agent.india.yf_history.history(symbol: str, period="5y") -> pandas.DataFrame
indiastock_agent.india.news.latest(company_name: str, days=7, limit=12) -> list[dict]
indiastock_agent.india.announcements.latest(symbol: str, limit=10) -> list[dict]
indiastock_agent.india.verdict.verdict(snapshot: dict|list, hist_df, news_items: list) -> dict




🔒 Markets & Symbols

Supported markets: NSE_INDIA (aliases: IN, INDIA, NSE)

Disabled: USA / HK / LSE / China

Tickers: use .NS suffix for history (TCS.NS, RELIANCE.NS)

Fallback to yfinance last close when provider is blocked (marked stale=True)


Rate Limiting, Caching, and Fallbacks

Low request rate, realistic headers, short TTL cache

Snapshot: provider → fallback to yfinance last close

History: yfinance

News & announcements: cache for 10–15 minutes



⚙️ Configuration
Variable	Default	Meaning
ISA_TTL_SNAPSHOT_SEC	30	Cache TTL for snapshot
ISA_TTL_NEWS_SEC	900	Cache TTL for news
ISA_TTL_ANN_SEC	900	Cache TTL for announcements
ISA_HTTP_TIMEOUT_SEC	15	HTTP timeout
ISA_USER_AGENT	(random)	Custom UA
NEWSAPI_KEY	(unset)	Use NewsAPI
🧪 Testing locally
python -m venv .venv && source .venv/bin/activate
pip install -U pip pytest
pip install -e .

pytest -q                 # unit tests
LIVE_TESTS=1 pytest -q tests/test_live_india.py

🧰 FastAPI Demo
from fastapi import FastAPI
import indiastock_agent as isa
from indiastock_agent.india import resolver, yf_history, news, announcements, verdict

app = FastAPI()

@app.get("/resolve")
def resolve_name(q: str):
    return {"symbol": resolver.resolve(q)}

@app.get("/snapshot/{symbol}")
def snapshot(symbol: str):
    return isa.api(symbol_list=[symbol], market="NSE_INDIA")

@app.get("/history/{symbol}")
def history(symbol: str, period: str = "5y"):
    return yf_history.history(symbol, period=period).to_dict()

@app.get("/news/{company}")
def latest_news(company: str):
    return news.latest(company)

@app.get("/announcements/{symbol}")
def latest_ann(symbol: str):
    return announcements.latest(symbol)

@app.get("/verdict/{symbol}")
def get_verdict(symbol: str):
    snap = isa.api(symbol_list=[symbol], market="NSE_INDIA")
    df = yf_history.history(symbol, period="1y")
    items = news.latest(symbol, days=30, limit=10)
    return verdict.verdict(snapshot=snap, hist_df=df, news_items=items)

⚖️ License

MIT. Based on the MIT-licensed FinanceAgent project, extended for India-only workflows.

🔒 Ethics & Fair Use

Respect robots.txt and provider ToS

Use caching, low request rates

Not financial advice

🗺️ Roadmap

 Expand resolver coverage

 Consensus “Buy/Hold/Sell” aggregator

 Sentiment scoring (FinBERT)

 Provider plugins

 Vector-DB backed Q&A

❓ FAQ

Q: Why is snapshot empty sometimes?
A: Provider may block; fallback to yfinance with stale=True

Q: API keys?
A: No, unless using NewsAPI
