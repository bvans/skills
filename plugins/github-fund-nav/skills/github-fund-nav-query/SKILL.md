---
name: github-fund-nav-query
description: Query the latest official unit NAV and NAV date for one or more 6-digit Chinese public fund codes through the user's bvans/fund-nav-public GitHub Actions workflow. Use when the user asks for latest fund NAVs, whether a NAV has been published, or asks to refresh/query specified fund codes. Do not use intraday estimated NAV (gsz) as official NAV.
---

Use the connected GitHub app and the repository `bvans/fund-nav-public`.

The workflow exists to provide **official unit NAV**, not intraday estimated NAV. Always preserve and report the actual `nav_date`. A QDII fund whose latest official NAV is T-1/T-2 is valid; never relabel that NAV as today's value.

Read `references/repository-contract.md` before the first GitHub write in a conversation.

## Inputs

Accept one or more 6-digit fund codes. Normalize separators such as commas, spaces, Chinese commas, and newlines. Reject or ask for correction only when a supplied fund code is not exactly 6 digits.

## Query workflow

1. For a normal lookup, first fetch `data/latest_nav.json` from `bvans/fund-nav-public`.
2. If every requested code is present and the user did not explicitly request refresh/re-query/current verification, return the matching records.
3. Trigger an on-demand refresh when any of these is true:
   - the user asks for "最新", "刷新", "重新查", "现在", "刚公布", or equivalent current verification;
   - any requested code is absent from `data/latest_nav.json`;
   - the cached record has an error;
   - the user explicitly asks to run the GitHub Actions query.
4. To refresh, fetch `query.json` first to obtain its current blob SHA. Do not modify `funds.json`.
5. Replace `query.json` with:
   ```json
   {
     "request_id": "<unique request id>",
     "codes": ["000218", "162412"]
   }
   ```
   Use a unique request id such as `chatgpt-YYYYMMDD-HHMMSS`. Preserve codes as strings so leading zeros are never lost.
6. Record the commit SHA returned by the GitHub file update.
7. Locate the GitHub Actions run named **Query fund NAV on demand** associated with that commit SHA and wait for it to complete. The workflow is triggered by the `query.json` push.
8. If the run succeeds, fetch `data/manual_latest_nav.json`.
9. Verify all of the following before using the result:
   - `query_mode == "manual"`;
   - `request_id` equals the request id written in step 5;
   - every requested code is present;
   - each used record has a non-null `unit_nav` and `nav_date`.
10. If the workflow fails or the result cannot be verified, do not invent a NAV. For codes already present in `data/latest_nav.json`, you may show the cached official NAV with a clear "cached" note. For other codes, report that the GitHub query failed.

## Result interpretation

For each fund, report:
- fund code;
- fund name;
- official unit NAV;
- NAV date;
- source when useful.

If `source_conflict=true`, explicitly warn that same-day sources disagree and include `same_day_source_values`. Do not silently choose one as unquestionably correct.

Never report `gsz` or other estimated values as official NAV.

## Output

For multiple funds, prefer a compact table:

| 代码 | 基金名称 | 净值日期 | 正式单位净值 | 状态 |
|---|---|---|---:|---|

Use "最新正式净值" to describe the value. If `nav_date` is earlier than the current date, say that the latest official disclosure is still dated that day rather than calling it stale or wrong.

If the user supplies a holding amount and asks to estimate shares, calculate `amount / unit_nav` only after explaining that the result is an estimate unless the holding amount is confirmed to use the same valuation date.


## Historical NAV queries

Use the repository history when the user asks for a NAV on a past date, purchase-date NAV, historical return, or transaction-based profit calculation.

1. Historical full-market files are stored at `data/all/history/YYYY/YYYY-MM-DD.csv.gz`.
2. The date in the filename is the **formal NAV date**, not the collection date.
3. For a requested fund/date, use the row whose `code` matches the 6-digit fund code.
4. Never substitute today's/latest NAV for a missing historical NAV.
5. If the exact date is not a NAV date (weekend/holiday) or the fund has delayed disclosure, distinguish:
   - transaction/application date;
   - confirmed NAV date;
   - formal `nav_date`.
   Do not assume they are identical.
6. For transaction profit calculations, prefer the platform's confirmed shares or confirmed NAV when available. Only reconstruct shares as `net subscription amount / confirmed unit NAV` when confirmed shares are unavailable, and state any fee assumption.
7. When history is missing, use the repository's **Backfill fund NAV history** workflow rather than inventing/interpolating a value. Backfill only the required codes/date range when practical.
8. After a backfill, verify that the expected historical files and requested fund rows actually exist before declaring success.
9. For QDII funds, preserve the disclosed NAV date even when it is T-1/T-2 relative to the transaction or query date.

### Historical return

For a simple NAV return between two formal NAV observations:

`return = end_unit_nav / start_unit_nav - 1`

This is **not** a total-return calculation when the fund distributed cash dividends during the interval. If dividends matter, use transaction/dividend cash flows or an appropriate adjusted/accumulated NAV series and label the methodology.

### Transaction-based P/L

For user cash-flow analysis, keep these concepts separate:
- gross subscriptions;
- redemption proceeds;
- cash dividends;
- remaining reconstructed/confirmed shares;
- latest market value;
- realized P/L;
- unrealized P/L;
- total P/L.

Do not call `subscriptions - redemptions` profit or principal without accounting for remaining shares and distributions.
