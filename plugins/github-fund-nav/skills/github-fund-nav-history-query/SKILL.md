---
name: github-fund-nav-history-query
description: Query official historical unit NAVs for Chinese public funds from the user's bvans/fund-nav-public GitHub history archive, and use them to reconstruct transaction shares or calculate investment returns. Use when the user asks for a fund NAV on a past date, NAV over a date range, buy-date NAV, transaction-based profit/loss, or return calculations. Never use intraday estimated NAV (gsz).
---

Use the connected GitHub app and repository `bvans/fund-nav-public`.

This skill is for **historical official NAV and transaction-return calculations**. For a simple latest-NAV lookup, use `github-fund-nav-query` instead.

Read `references/repository-contract.md` before the first GitHub write in a conversation.

## Historical data contract

Full-market history is stored by actual NAV date:

```text
data/all/history/YYYY/YYYY-MM-DD.csv.gz
```

Columns:
- `code`
- `name`
- `type`
- `nav_date`
- `unit_nav`
- `accum_nav`
- `source`

Historical backfills use the same files and merge by fund code. The source may be `eastmoney:lsjz-backfill`.

## Query rules

1. Preserve fund codes as 6-digit strings.
2. For an exact historical date, read that date's history file and select the requested code.
3. If the date is not a NAV date (weekend/holiday) or the fund has no record on that date:
   - for transaction reconstruction, use the actual NAV date implied by the fund transaction/confirmation rules only when the transaction time and rule are known;
   - otherwise report that no official NAV exists for the requested date and show the nearest prior/next available NAV only as context, not as an invented exact-date value.
4. For a range query, read the relevant date-partitioned files and return official observations ordered by `nav_date`.
5. For dates earlier than the archive's coverage, use the repository's **Backfill fund NAV history** workflow/data path. Do not substitute estimated NAV.
6. Never use `gsz` or intraday estimates as official NAV.

## Transaction profit/loss

When the user provides transaction records:

1. Exclude transfers when the user requests cash-flow-only performance.
2. Distinguish buys, redemptions, dividends, refunds/cancellations, and pending transactions.
3. Prefer confirmed shares/confirmed NAV from the transaction record when available.
4. If confirmed shares are absent, reconstruct shares from the applicable official NAV only after considering:
   - transaction timestamp and fund cutoff;
   - weekends/holidays;
   - QDII delayed confirmation;
   - front-end subscription fees when applicable.
5. Do not assume `amount / same-calendar-day NAV` when the transaction's confirmation date may differ.
6. Latest valuation must use the latest **official** unit NAV and retain its actual `nav_date`.
7. Report separately:
   - gross cash buys;
   - redemption proceeds;
   - cash dividends;
   - estimated/confirmed remaining shares;
   - latest market value;
   - realized P/L when cost basis can be determined;
   - unrealized P/L;
   - total P/L;
   - return rate.
8. Label reconstructed values as estimates whenever fees, confirmation shares, or exact confirmation dates are unavailable.

## Backfill workflow

Repository workflow: **Backfill fund NAV history**.

Inputs:
- `start`: YYYY-MM-DD
- `end`: YYYY-MM-DD
- `codes`: optional comma/space-separated fund codes; blank means `funds.json`.

The backfill implementation splits broad ranges into small windows to avoid Eastmoney `lsjz` silently truncating older observations. After a backfill, verify that the earliest and latest expected NAV dates exist and inspect coverage before claiming completion.

## Output

For historical lookup:

| 代码 | 基金名称 | 净值日期 | 正式单位净值 | 来源 |
|---|---|---|---:|---|

For transaction-return analysis, provide an aggregate summary first, then per-fund results. Explicitly state any estimation assumptions that materially affect P/L.
