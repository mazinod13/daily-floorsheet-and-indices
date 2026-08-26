**NEPSE Daily Data**
Auto-updating archive of daily NEPSE floorsheet and index data, committed automatically via GitHub Actions.

**How it works**
A scheduled GitHub Action runs after market close on each trading day, fetches the latest floorsheet and index data, and commits it to this repo if anything changed. This gives a free, versioned history of daily snapshots — useful for backtesting, audits, and diffing.
