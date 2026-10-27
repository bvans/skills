---
name: github-index-market-query
description: Query the latest A-share index quote, daily change, source timestamp, recent high and drawdown from the user's bvans/fund-nav-public GitHub index cache. Use for 中证1000、中证A500、沪深300、中证500、上证指数、深证成指、创业板指、科创50 current/latest point queries and drawdown checks. The latest quote must remain Eastmoney-authoritative; Tencent may only be used as labeled historical context.
---

Use the public data repository `bvans/fund-nav-public`.

This skill is for **A-share index market quotes**, not public-fund NAVs. For fund unit NAV, use `github-fund-nav-query`.

Read `references/index-market-contract.md` before interpreting the data contract.

## Primary source

Always start with:

`data/indices/latest.json`

Raw URL:

`https://raw.githubusercontent.com/bvans/fund-nav-public/main/data/indices/latest.json`

Select the requested index by `code`.

Tracked codes currently include:

- `000510` — 中证A500
- `000852` — 中证1000
- `000300` — 沪深300
- `000905` — 中证500
- `000001` — 上证指数
- `399001` — 深证成指
- `399006` — 创业板指
- `000688` — 科创50

## Quote authority rules

1. Treat the latest quote as authoritative only when the top-level `source` is `eastmoney:push2`.
2. Preserve and inspect:
   - `source_date`
   - `source_time`
   - `source_epoch`
3. Never call an older quote "today's" or "current" merely because `checked_at` is recent. `checked_at` is collection time; `source_time` is the market-source timestamp.
4. Never substitute Tencent history for the latest point. Tencent is allowed only for historical K-line context.
5. Never use a search-engine snippet, Investing.com, or an unrelated quote page as the first source when the repository cache contains a valid Eastmoney quote.

## Freshness handling

For "latest/current/现在/实时" requests:

1. Read `data/indices/latest.json`.
2. If the requested record's `source_date` is the expected latest China A-share trading date and `source_time` is sufficiently current for the question, use it directly.
3. During market hours, if the user explicitly needs a live/current point and `source_time` is materially older than the current market time, query Eastmoney `push2` directly using the configured `secid` and keep the same field semantics.
4. If direct Eastmoney verification fails, report the repository quote with its exact `source_time` and say it is the latest verified cached quote. Do not invent or silently replace it with another source.
5. On weekends/holidays, the prior trading day's `source_date` is valid; state the date explicitly.

## Fields

For the requested index, use:

- `price` — latest/closing index level from Eastmoney
- `pct_change` — daily percentage change
- `change` — point change
- `open`
- `high`
- `low`
- `prev_close`
- `volume`
- `amount`
- `source_time`
- `source_date`

For historical context:

- `history_status`
- `history_source`
- `history_host`
- `history_note`
- `high_120d`
- `high_120d_date`
- `drawdown_from_120d_high_pct`

## Historical/drawdown semantics

The collector prefers Eastmoney `push2his` for daily history. GitHub-hosted runners may be rejected by that endpoint, so the collector can fall back to Tencent `newfqkline`.

When `history_source == "tencent:newfqkline"`:

- the **latest quote is still Eastmoney**;
- Tencent is only the labeled historical series used for recent-high/drawdown calculations;
- do not describe the whole record as Tencent-sourced.

If `high_120d` or `drawdown_from_120d_high_pct` is null, say historical context was unavailable. Never use today's intraday high as a fake 120-day high.

## Cross-checks

For a quoted daily percentage change, optionally verify:

`(price / prev_close - 1) * 100`

Allow normal rounding differences.

If the source timestamp/date, price, and percentage change are internally inconsistent, stop and report the inconsistency rather than choosing a value silently.

## Output

For a simple latest-index request, prefer:

`中证1000：6427.17，-3.07%，东方财富源时间 2026-10-27 16:12（北京时间）。`

When drawdown is relevant, add the exact high/date and label the history source if it is not Eastmoney.

For monitoring/automation use, always make the data lineage explicit enough to distinguish:

- Eastmoney realtime quote;
- Tencent historical fallback;
- collection time vs market-source time.
