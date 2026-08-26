# NEPSE daily floorsheet and indices

Every trading day's complete floorsheet and index snapshot from the Nepal Stock
Exchange, committed automatically after market close.

<!-- BEGIN GENERATED -->

**Latest trading day:** `2026-08-26`  
**NEPSE Index:** 2,558.35 ▼ -1.38%  
**Floorsheet rows that day:** 61,442  
**Trading days in this repo:** 1 (from `2026-08-26`)  
**Last updated:** 2026-08-26T10:25:48Z (2026-08-26 16:10:48 NPT)

<!-- END GENERATED -->

## Layout

| Path | What it holds |
| --- | --- |
| `floorsheet/<YYYY>/<YYYY-MM-DD>.csv.gz` | Every trade of that session, one row per contract |
| `indices/<YYYY>/<YYYY-MM-DD>.json` | Raw API response for the main and sub indices |
| `indices/daily.csv` | Append-only long series, one row per index per day |
| `latest/floorsheet.csv.gz`, `latest/indices.json` | Most recent session, at a stable path |
| `manifest.json` | Machine-readable summary of what is in here |

## Floorsheet columns

`contract_id`, `business_date`, `trade_time`, `stock_symbol`, `security_name`,
`stock_id`, `buyer_member_id`, `buyer_broker_name`, `seller_member_id`,
`seller_broker_name`, `contract_quantity`, `contract_rate`, `contract_amount`,
`trade_book_id`

## Reading it

```python
import pandas as pd

fs = pd.read_csv("floorsheet/2026/2026-08-26.csv.gz")          # gzip is inferred
idx = pd.read_csv("indices/daily.csv", parse_dates=["business_date"])

# Turnover by symbol for the session
fs.groupby("stock_symbol")["contract_amount"].sum().sort_values(ascending=False).head(10)

# NEPSE Index series
idx[idx.index_name == "NEPSE Index"].set_index("business_date")["current_value"].plot()
```

## A note on index values

Use **`current_value`** as the day's closing level. NEPSE's API reports `close`
and `previous_close` as *the previous session's* close on both fields, so they
are preserved for completeness but are not the daily close. Sub-indices are
served with a reduced field set (`current_value`, `change`, `per_change` only),
so their `high`/`low`/52-week columns are empty.

## Source

Pulled from the official NEPSE API at <https://www.nepalstock.com.np>. This repo
is generated data only; it is not affiliated with or endorsed by NEPSE.
