# Historical NAV repository contract

Repository: `bvans/fund-nav-public`.

## Full-market history

Path pattern:

`data/all/history/YYYY/YYYY-MM-DD.csv.gz`

Rows are keyed by fund code and actual official NAV date. Backfilled observations are merged into existing daily files rather than replacing the entire file.

## Latest NAV

Use `data/all/latest_nav.json` for full-market latest official NAV when available. The configured-focus cache `data/latest_nav.json` contains only `funds.json` funds.

## Backfill

Implementation: `backfill_nav.py`
Workflow: `.github/workflows/backfill-nav.yml`
Display name: `Backfill fund NAV history`

The historical source is Eastmoney's formal `f10/lsjz` data. The collector must use bounded date windows because broad-range requests can silently return only a trailing subset.

Never treat an existing file as proof of complete coverage. Verify requested codes and dates.
