# A-share index market data contract

## Repository

Repository: `bvans/fund-nav-public`

Primary latest file:

`data/indices/latest.json`

Daily/intraday snapshots:

`data/indices/snapshots/YYYY-MM-DD.jsonl`

Per-index history cache:

`data/indices/history/{code}.json`

Collector:

`fetch_index_quotes.py`

Configuration:

`indices.json`

Workflow:

`.github/workflows/fetch-index-quotes.yml`

## Scheduled collection

The workflow is scheduled on China-market weekdays around:

- 11:30 Asia/Shanghai
- 14:45 Asia/Shanghai
- 15:10 Asia/Shanghai

GitHub Actions schedules can be delayed. Always judge quote freshness from the record's `source_time`, not only from workflow/collection time.

## Top-level latest schema

Representative structure:

```json
{
  "source": "eastmoney:push2",
  "source_host": "push2delay.eastmoney.com",
  "checked_at": "2026-10-27T20:03:13+08:00",
  "timezone": "Asia/Shanghai",
  "market_phase": "closed",
  "indices": [],
  "history_hosts": {},
  "history_errors": {}
}
```

`checked_at` is collector time. It is not the market quote timestamp.

## Index record schema

Representative fields:

```json
{
  "code": "000852",
  "name": "中证1000",
  "secid": "1.000852",
  "history_secid": "2.000852",
  "source_name": "中证1000",
  "price": 6427.17,
  "pct_change": -3.07,
  "change": -203.61,
  "open": 6664.61,
  "high": 6717.99,
  "low": 6427.17,
  "prev_close": 6630.78,
  "volume": 195783204,
  "amount": 293430070933.5,
  "source_epoch": 1793088720,
  "source_time": "2026-10-27T16:12:00+08:00",
  "source_date": "2026-10-27",
  "history_status": "fresh",
  "history_source": "tencent:newfqkline",
  "history_host": "proxy.finance.qq.com",
  "history_note": "Eastmoney history unavailable; Tencent fallback used",
  "high_120d": 8991.85,
  "high_120d_date": "2026-05-14",
  "drawdown_from_120d_high_pct": -28.522
}
```

## Source semantics

### Eastmoney realtime

`source = eastmoney:push2`

This is the authoritative source for:

- latest index point
- daily percentage/point change
- open/high/low
- previous close
- volume/turnover amount
- source timestamp

Realtime Eastmoney secids include:

- 中证A500: `1.000510`
- 中证1000: `1.000852`
- 沪深300: `1.000300`
- 中证500: `1.000905`
- 上证指数: `1.000001`
- 深证成指: `0.399001`
- 创业板指: `0.399006`
- 科创50: `1.000688`

### Historical context

Preferred: `eastmoney:push2his`

Fallback: `tencent:newfqkline`

Historical data is only for recent-high/drawdown context. It must never replace Eastmoney as the latest quote source.

## Important invariants

1. `source_time` is the market-source timestamp; `checked_at` is collector time.
2. Never report a stale `source_date` as today's quote.
3. Never turn null/missing history into a fabricated drawdown.
4. Never treat today's high as a 120-day high unless the actual 120-day history proves it.
5. When Tencent history fallback is used, state that only historical context came from Tencent.
