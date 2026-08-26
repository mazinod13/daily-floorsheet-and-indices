**NEPSE Daily Data**

Auto-updating archive of daily NEPSE floorsheet and index data.

**How it works**

Every trading day after market close, a scheduled job fetches the day's floorsheet and index values from NEPSE and commits them here. The result is a free, versioned history of daily snapshots — useful for backtesting, audits, and diffing.

This repository holds **data only**. The fetcher that writes to it lives in a separate private repository and pushes here via a deploy key. Nothing here is edited by hand.

**Structure**

```
└── data/
    ├── floorsheet/YYYY-MM-DD.csv   # one row per trade
    └── indices/YYYY-MM-DD.csv      # one row per index
```

Files are named by the business date NEPSE assigns the session, not by the date the job happened to run.

**Schedule**

Sunday–Thursday, the NEPSE trading week, shortly after the 15:00 NPT close. Holidays produce no commit — if a date is missing, the market did not trade that day.

There is no backfill: the archive begins with the first successful run and grows forward.

**Columns**

`data/floorsheet/YYYY-MM-DD.csv` — one row per trade, sorted by `contractId`:

| Column | Notes |
| --- | --- |
| `contractId` | unique per trade, ascending through the session |
| `businessDate` | trading date |
| `tradeTime` | timestamp of the print |
| `stockSymbol` | e.g. `NABIL` |
| `securityName` | full company name |
| `buyerMemberId` / `sellerMemberId` | broker IDs |
| `contractQuantity` | shares |
| `contractRate` | price per share |
| `contractAmount` | quantity × rate |
| `stockId` | NEPSE's internal security ID |
| `tradeBookId` | NEPSE's internal trade ID |

`data/indices/YYYY-MM-DD.csv` — one row per index, sorted by name:

`businessDate`, `index`, `close`, `high`, `low`, `previousClose`, `change`, `perChange`, `fiftyTwoWeekHigh`, `fiftyTwoWeekLow`, `currentValue`

Four indices are recorded: NEPSE Index, Sensitive Index, Float Index, Sensitive Float Index.

**Using it**

```python
import pandas as pd

trades = pd.read_csv("data/floorsheet/2026-08-26.csv")
turnover = trades.groupby("stockSymbol")["contractAmount"].sum().sort_values(ascending=False)
```

Or read a single day without cloning:

```python
BASE = "https://raw.githubusercontent.com/mazinod13/daily-floorsheet-and-indices/main"
trades = pd.read_csv(f"{BASE}/data/floorsheet/2026-08-26.csv")
```

A full session is roughly 40,000 trades (~5 MB of CSV).

**Source**

[nepalstock.com.np](https://www.nepalstock.com.np) — NEPSE's own site, read through the JSON API its front end uses. No third-party aggregator.

That API is undocumented and can change without notice, so treat this archive as best-effort. A missing date means either a market holiday or a failed run.

**License**

The market data is NEPSE's. This repository only archives it, and redistributes it as-is with no warranty as to accuracy or completeness — do not rely on it for trading decisions without verifying against NEPSE directly. Check NEPSE's terms before any commercial use.

Any code in this repository is MIT — see [LICENSE](LICENSE).
