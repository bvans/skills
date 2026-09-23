# Repository contract

## Data repository

Repository: `bvans/fund-nav-public`

### Configured-fund cache

Path: `data/latest_nav.json`

Important fields:

```json
{
  "generated_at": "2026-09-23T20:12:59+08:00",
  "query_mode": "configured",
  "fund_count": 44,
  "official_nav_found": 44,
  "source_conflicts": 0,
  "funds": [
    {
      "code": "000218",
      "name": "国泰黄金ETF联接A",
      "nav_date": "2026-09-22",
      "unit_nav": 3.3625,
      "source": "eastmoney:pingzhongdata",
      "source_conflict": false,
      "same_day_source_values": [3.3625],
      "error": null
    }
  ]
}
```

### On-demand request

Path: `query.json`

Schema:

```json
{
  "request_id": "chatgpt-20260923-202148",
  "codes": ["000218", "162412", "110022"]
}
```

Updating this file on `main` triggers `.github/workflows/query-nav.yml`.

### On-demand result

Path: `data/manual_latest_nav.json`

Expected fields are the same as the configured cache plus:

```json
{
  "query_mode": "manual",
  "request_id": "chatgpt-20260923-202148"
}
```

The skill must verify `request_id` before returning results so a previous query is never mistaken for the current request.

## Source semantics

- `eastmoney:pingzhongdata`: official historical unit NAV parsed from Eastmoney/Tiantian Fund data.
- `eastmoney:fundgz-dwjz`: official prior NAV fields `dwjz + jzrq`, used as a second source/fallback.
- The collector deliberately ignores intraday `gsz`.
- `source_conflict=true` means same-date official sources returned different values.

## GitHub workflow

Workflow display name: `Query fund NAV on demand`

Trigger: push to `query.json` on `main`.

The workflow reads the codes, runs `fetch_nav.py`, and commits:
- `data/manual_latest_nav.json`
- `data/manual_latest_nav.csv`

The normal scheduled workflow remains separate and updates:
- `data/latest_nav.json`
- `data/latest_nav.csv`
- `data/history/YYYY-MM-DD.json`

Never modify `funds.json` for an ad-hoc query.
