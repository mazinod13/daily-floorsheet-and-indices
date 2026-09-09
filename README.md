# NEPSE daily floorsheet and indices

Every trading day's complete floorsheet and index snapshot from the Nepal Stock
Exchange, committed automatically after market close.

<!-- BEGIN GENERATED -->

**Latest trading day:** `2026-09-09`  
**NEPSE Index:** 2,545.40 ▼ -0.60%  
**Trades that session:** 50,804  
**Indices recorded:** 17 (4 main + 13 sub)  
**Trading days archived:** 8 (from `2026-08-26`)  
**Last updated:** 2026-09-09T13:56:54Z (2026-09-09 19:41 NPT)

<!-- END GENERATED -->

## Layout

| Path                                 | What it holds                                            |
| ------------------------------------ | -------------------------------------------------------- |
| `data/floorsheet/<YYYY-MM-DD>.csv` | Every trade of that session, one row per contract        |
| `data/indices/<YYYY-MM-DD>.csv`    | Main and sub indices for that session, one row per index |

Files are named by the business date NEPSE assigns the session, not by the date
the job happened to run. A missing date means the market did not trade.

This repo only grows forward from its first run. For everything before that, see
[Historical data](#historical-data) below.

Sessions run Sunday–Thursday and close at 15:00 NPT. A full day is around 60,000
trades, roughly 12 MB of CSV.

## Historical data

Years of back history live in a companion dataset on Hugging Face, which is a
better home for bulk files than a git repo:

**[huggingface:urekmazino69](https://huggingface.co/datasets/urekmazino69/Nepse-floorsheet-and-indices)**

|                                    | Coverage                 | Files              |
| ---------------------------------- | ------------------------ | ------------------ |
| `floorsheet/<YYYY-MM-DD>.csv.gz` | 2024-01-02 → 2026-08-26 | 603 trading days   |
| `indices/<YYYY-MM-DD>.csv.gz`    | 1997-07-20 → 2026-08-26 | 6,429 trading days |

Gzipped CSV, one file per trading day, same date-based naming as this repo.

```python
import pandas as pd

BASE = "https://huggingface.co/datasets/urekmazino69/Nepse-floorsheet-and-indices/resolve/main"
fs = pd.read_csv(f"{BASE}/floorsheet/2024-01-02.csv.gz")   # gzip is inferred
```

> **The columns are not the same as this repo's.** The historical set was
> assembled from a different source, so it uses `snake_case` names and a
> different field set — `contract_no` not `contractId`, `calculation_date` not
> `businessDate`, `sector` in place of `indexName`. It also *adds*
> `turnover_values`, `turnover_volume` and `total_transaction`, and *lacks*
> security names, broker names and NEPSE's internal IDs. Rename before
> concatenating the two; do not assume `pd.concat` will line up.

One quirk to handle on the historical indices: `percentage_change` changes
format mid-series — values are strings like `-0.01%` up to 2026-01-20 and plain
numbers like `-1.3100` after. Normalise on read:

```python
idx["percentage_change"] = idx["percentage_change"].astype(str).str.rstrip("%").astype(float)
```

## Floorsheet columns

One row per trade, sorted by `contractId` ascending, so the file reads
chronologically.

| Column                                   | Notes                        |
| ---------------------------------------- | ---------------------------- |
| `contractId`                           | Unique per trade             |
| `businessDate`                         | Trading date                 |
| `tradeTime`                            | Timestamp of the print       |
| `stockSymbol`                          | e.g.`NABIL`                |
| `securityName`                         | Full company name            |
| `stockId`                              | NEPSE's internal security ID |
| `buyerMemberId`, `buyerBrokerName`   | Buying broker                |
| `sellerMemberId`, `sellerBrokerName` | Selling broker               |
| `contractQuantity`                     | Shares                       |
| `contractRate`                         | Price per share              |
| `contractAmount`                       | Quantity × rate             |
| `tradeBookId`                          | NEPSE's internal trade ID    |

Broker IDs and names are only populated after close — NEPSE masks them while the
market is open.

## Index columns

`businessDate`, `indexId`, `indexCode`, `indexName`, `isSubIndex`, `close`,
`previousClose`, `change`, `perChange`, `high`, `low`, `fiftyTwoWeekHigh`,
`fiftyTwoWeekLow`, `sector`, `baseYearMarketCapitalization`, `generatedTime`

Seventeen rows per day: four main indices (`isSubIndex` = 0) — NEPSE Index,
Sensitive, Float, Sensitive Float — and thirteen sector sub-indices
(`isSubIndex` = 1) such as Banking, HydroPower, Life Insurance.

Sub-indices are served with a reduced field set, so `high`, `low`,
`fiftyTwoWeekHigh` and `fiftyTwoWeekLow` are empty on those rows. `sector` is
`ALL` for the four main indices and names the sector for the rest.

## Reading it

```python
import pandas as pd

fs = pd.read_csv("data/floorsheet/2026-08-26.csv")
idx = pd.read_csv("data/indices/2026-08-26.csv", parse_dates=["businessDate"])

# Turnover by symbol for the session
fs.groupby("stockSymbol")["contractAmount"].sum().sort_values(ascending=False).head(10)

# Most active brokers by value traded
fs.groupby("buyerBrokerName")["contractAmount"].sum().sort_values(ascending=False).head(10)

# Main indices only
idx[idx.isSubIndex == 0][["indexName", "close", "perChange"]]
```

Or read a single day without cloning:

```python
BASE = "https://raw.githubusercontent.com/mazinod13/daily-floorsheet-and-indices/main"
fs = pd.read_csv(f"{BASE}/data/floorsheet/2026-08-26.csv")
```

To build a series across days, concatenate the index files:

```python
from pathlib import Path

idx = pd.concat(
    pd.read_csv(p, parse_dates=["businessDate"])
    for p in sorted(Path("data/indices").glob("*.csv"))
)
nepse = idx[idx.indexCode == "NEPSE"].set_index("businessDate")["close"]
```

## A note on index values

Use `close` as the session's closing level and `previousClose` as the prior
session's — `close` minus `previousClose` reconciles with `change`.

This only holds for files written after market close, which is when the job
runs. If you ever fetch intraday, NEPSE reports the *previous* session's close
on both fields until the session settles, and the live level lives on a separate
`currentValue` field that is not archived here.

## License

The market data is NEPSE's. This repository only archives it and redistributes
it as-is, with no warranty as to accuracy or completeness; verify against NEPSE
directly before relying on it. This repo is not affiliated with or endorsed by
NEPSE. Any code here is MIT — see [LICENSE](LICENSE).
