---
name: outcometick-data
description: Find, price and download historical data for Polymarket and Predict.fun crypto up/down prediction markets, and validate, price or follow outcometick backtests, with the outcometick MCP tools. Use when the user asks for prediction-market tick data, order books, settlement or Chainlink price data from outcometick, or wants to backtest a Polymarket or Predict.fun strategy.
---

# outcometick: data and backtests

The `outcometick` MCP server that comes with this plugin reads the outcometick archive (historical data for crypto
up/down prediction markets on Polymarket and Predict.fun) and the hosted backtester. It reads, prices and validates.
It cannot submit a backtest or buy anything, and you must not try to do either for the user by other means: both
spend the user's money or credits, so the user does them.

## Ground rules

- Facts that change (prices, the downloadable date window, assets, intervals, dataset names, settlement rules) come
  from the tools or the docs at the time you answer, never from memory and never from this file.
- Authoritative reference, written for agents: https://outcometick.com/llms-full.txt. Read it before explaining file layouts,
  columns or how markets settle. Human docs: https://outcometick.com/docs (data API, schemas, settlement) and https://outcometick.com/docs/sdk
  (strategy SDK).
- Report what the tools return. Do not describe the archive as complete; if a day or file is missing, say so.
- Never echo, log or write a key (`ck_…`, `bt_…`) into files or chat output.

## Access

The first tool call opens a browser sign-in (OAuth); if it does not, run `/mcp` and authenticate `outcometick`.
The consent page asks for the user's existing keys: a data subscription key, a smart-money key and a backtest key,
each in its own field (data and smart-money keys both start with `ck_`). All are optional: with none, only the
public tools work. Keys are bought by the user at https://outcometick.com; point them there, do not buy for them.

| Tool | Needs | What it does |
| --- | --- | --- |
| `coverage` | none | What the archive holds per venue: assets, datasets, day range |
| `plans` | none | Data subscription plans and their live prices |
| `meta` | data key | The date window and filter values the data key can reach |
| `list_files` | data key | Search archive files the data key can download |
| `download_file` | data key | A short-lived signed URL for one file, with its sha256 |
| `backtest_capacity` | none | Which days can be backtested now, and how busy the queue is |
| `backtest_quote` | none | Price a backtest scope in credits without submitting it |
| `check_strategy` | none | Validate a strategy's files with the queue's own validator, and price it |
| `backtest_runs` | backtest key | Recent runs under the backtest key, and the credit balance |
| `backtest_status` | backtest key | One run's status, credits and report summary |
| `smart_plans` | none | Smart-money plans (daily Polymarket wallet lists) and their live prices |
| `smart_coverage` | none | Which smart-money days are published |
| `smart_days` | smart-money key | The days the smart-money key can download, with each list's status |
| `smart_download` | smart-money key | A short-lived signed URL for one day's list, with its sha256 |

## Finding and downloading data

1. `coverage` shows what the archive holds per venue. `plans` shows subscription plans with live prices.
2. With a data key, call `meta` first: it returns the date window the key can reach and the real values for venue,
   asset, interval and dataset. Use those values exactly in `list_files`.
3. `list_files` searches files. `asset` is the base symbol (`BTC`, not `BTCUSD`). An interval filter such as
   `5m` leaves out the period-less settlement streams; pass `["5m", "none"]` to get both. If it returns nothing,
   compare your filters with `meta` before telling the user the data does not exist.
4. `download_file` takes `date` and `name` exactly as `list_files` reported them and returns a short-lived signed
   URL plus the file's sha256. Download the URL (for example `curl -L -o <name> "<url>"`), check the sha256 before
   using the file, and call `download_file` again if the URL has expired.
5. For pandas or polars, `format: "parquet"` lists the typed Parquet copy of each file instead of the gz original.

## Smart-money lists

- `smart_plans` and `smart_coverage` need no key: what is sold and which days are published.
- With a smart-money key, `smart_days` lists the days the key can download and each list's status; take `day`
  and `list` from it for `smart_download`, which returns a short-lived signed URL and the sha256 to check.
  The file is a zstd-compressed CSV. A data subscription key does not open these, and the reverse.

## Backtests

- `backtest_capacity` (per venue) says which days can be backtested now and how busy the queue is.
- `backtest_quote` prices a scope in credits without submitting anything.
- `check_strategy` runs the queue's own validator on a strategy (send `manifest.json` and the source files as
  contents) and prices it. Fix what it rejects before the user submits.
- Submitting is the user's step: `ot submit <dir>` from the `outcometick` npm package, or https://outcometick.com/backtest. Show
  them the quote and the exact command, and let them run it.
- With a backtest key, `backtest_runs` lists recent runs and the credit balance; `backtest_status` shows one run
  and its report summary when done.
- Writing a strategy: https://outcometick.com/docs/sdk. Working templates in Python and Node.js:
  https://github.com/outcometick/polymarket-strategy-templates. Locally, `npm i -g outcometick` gives
  `ot init`, `ot check` and `ot run`.
