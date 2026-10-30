# India Stock Screener

Two tools for screening NSE-listed stocks with price/volume technicals, built on free Yahoo Finance data:

1. **`india_stock_screener.py`** – a command-line momentum screener. Ranks a stock universe by 12-1 momentum, applies trend / RSI / volume filters, caps sector concentration, flags breakouts, and exports a formatted Excel watchlist.
2. **`stock_screener_app.py`** – a Streamlit app for pattern-based scans (Tight Consolidation, Long Base Breakout, Box Setup) over any NSE index, with interactive candlestick charts.

> **Disclaimer:** For educational and research purposes only. Not investment advice. These screens have **not been backtested** (see [Limitations](#limitations)).

---

## 1. Momentum watchlist engine (`india_stock_screener.py`)

### Pipeline

| Step | Filter | Default |
|------|--------|---------|
| 1 | Liquidity: 20-day avg daily turnover | ≥ ₹5 Cr |
| 1 | Minimum price | ≥ ₹50 |
| 2 | Rank by **12-1 momentum** (return from 12 months ago to 1 month ago), keep top N | Top 40 |
| 3 | Trend: close above 50-EMA **and** 200-EMA | on |
| 4 | RSI(14) in a strength-but-not-overbought band | 50–70 |
| 5 | Up-day / down-day volume ratio (accumulation proxy) | ≥ 1.0 |
| 6 | Sector cap | ≤ 4 per sector |
| 7 | Breakout alert: close above prior 20-day high on ≥ 1.5× average volume | flagged, not filtered |

Stocks that survive steps 1–6 are scored 0–4 (trend, RSI, volume, breakout) and the top 20 are exported.

Skipping the most recent month in the momentum calculation follows the standard academic 12-1 construction, which avoids the short-term reversal effect.

### Run

```bash
pip install -r requirements.txt
python india_stock_screener.py
```

Output: a ranked table in the console plus `india_watchlist.xlsx` (sheets: `Watchlist`, `All Screened`). Breakout rows are highlighted.

### Configure

Everything is controlled by the `CONFIG` dict at the top of the file (liquidity thresholds, momentum window, EMA periods, RSI band, sector cap, watchlist size). The universe is the `NIFTY500_TICKERS` dict (`"TICKER.NS": "Sector"`); extend it as needed.

---

## 2. Pattern scanner (`stock_screener_app.py`)

```bash
streamlit run stock_screener_app.py
```

Pick a setup and a universe (Nifty 50 / 100 / 200 / 500, midcap / smallcap indices, sector indices, or your own tickers), tune the parameters in the sidebar, and click **Run Screener**.

| Setup | What it looks for |
|-------|-------------------|
| **Tight Consolidation** | Price confined to a narrow range for the last N sessions, in an uptrend, with minimum volume and market cap |
| **Long Base Breakout** | Common filters plus repeated rejections from the same resistance zone (min. touches within a tolerance band) |
| **Box Setup** | Long-base conditions plus a support floor with repeated bounces, i.e. a defined box |

Common filters: minimum average volume, minimum market cap, and an uptrend test (within X% of 52-week high, or 1y < 6m < now price progression). Each match gets an expandable card with metrics and a Plotly candlestick chart (EMAs 20/50/100/200, volume, resistance/support zones annotated).

Index constituents are pulled from the NSE archives CSVs; prices and market caps come from `yfinance`. Results are cached (prices 30 min, constituents 1 h).

---

## Project layout

```
india-stock-screener/
├── india_stock_screener.py   # CLI momentum screener -> Excel
├── stock_screener_app.py     # Streamlit pattern scanner
├── requirements.txt
├── LICENSE
└── README.md
```

## Limitations

Worth knowing before relying on any output:

- **No backtest.** The filters are standard technical heuristics; nothing here shows they produce excess returns on Indian equities after costs.
- **Survivorship / selection bias.** The CLI universe is a hand-picked list of ~100 current large/mid caps, not point-in-time Nifty 500 membership. Some tickers may be stale after renames, demergers or delistings, so check the "Failed tickers" line in the output.
- **Data quality.** Yahoo Finance is free and unofficial: occasional gaps, adjustment errors, rate limiting, and symbol changes.
- **RSI** uses a simple rolling mean rather than Wilder's smoothing, so values differ slightly from most charting platforms.
- **Support/resistance "touches"** count days whose high/low falls inside a tolerance band, so consecutive days at one level can inflate the count.
- The Streamlit app fetches market cap per ticker via `yf.Ticker(...).info`, which is slow on large indices (Nifty 500 can take several minutes).

## Roadmap

- [ ] Walk-forward backtest of the momentum pipeline (with transaction costs, NIFTY 500 benchmark)
- [ ] Point-in-time index membership to remove survivorship bias
- [ ] Wilder-smoothed RSI and ATR-based consolidation measure
- [ ] Merge the two tools behind one shared indicators module
- [ ] Unit tests for indicator functions
