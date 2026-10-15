<!-- Generated from the outcometick monorepo by scripts/publish-claude-plugin.mjs. Edits here are overwritten. -->
# outcometick for Claude Code

A Claude Code plugin for [outcometick](https://outcometick.com): historical data for crypto up/down prediction markets on
Polymarket and Predict.fun, and a hosted backtester.

It adds:

- the hosted outcometick MCP server (`https://outcometick.com/v1/mcp`, Streamable HTTP, OAuth), and
- the `outcometick-data` skill, which tells Claude how to use it: find and download files, price and validate backtests,
  follow runs.

The server only reads, prices and validates. It never submits a backtest or buys anything.

## Install

In Claude Code:

```
/plugin marketplace add outcometick/claude-plugin
/plugin install outcometick@outcometick
```

The first time a tool runs, Claude Code opens the outcometick sign-in page in your browser (or run `/mcp` and
choose `outcometick`). Paste your data key and/or backtest key there, or leave both empty to use the public tools
only. Your client gets a token, not your key, and the token stops working when the key is revoked or expires.

## Tools

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

## Without the plugin

```
claude mcp add --transport http outcometick https://outcometick.com/v1/mcp
```

or, with a key in a header instead of OAuth, `--header "Authorization: Bearer <key>"`. Docs: https://outcometick.com/docs.

## Community

- Discord (English and 中文): https://discord.gg/ZDMqkptwm5
- Telegram: https://t.me/+TcK_hJOzOz5mZTk1

License: MIT.
