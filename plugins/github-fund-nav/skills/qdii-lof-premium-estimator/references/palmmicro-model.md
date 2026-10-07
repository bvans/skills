# PalmMicro QDII/LOF estimation model notes

This reference documents the relevant public source code in `palmmicro/web` for reproducing and interpreting SH501312 estimates.

Repository:
- https://github.com/palmmicro/web

## Relevant files

### 1. `woody/res/php/_qdiimix.php`

SH501312 is handled by the QDII-mix estimator through:

`new HoldingsReference('SH501312')`

An old source-code comment records an early proxy basket:

`ARKW*19.56;ARKK*19.66;ARKF*16.75;ARKG*11.86;ARKQ*5.37;QQQ*8.88;SOXX*7.44;XLK*5.2`

Treat this comment only as a historical fallback/example. The live server reads proxy holdings/weights from its database and they may differ.

Critical behavior for SH501312:

`_updateStockHoldings()` updates `HoldingsDateSql` to the latest NAV date for SH501312, but does not automatically download a new SH501312 holdings file or rewrite its proxy weights.

Therefore, PalmMicro's displayed “基金持仓更新于 <date>” for SH501312 is an **anchor/model date**, not proof of a true full-portfolio disclosure on that date.

### 2. `php/stock/holdingsref.php`

The core estimator is `HoldingsReference::_estNav()`.

Source comment:

`(x - x0) / x0 = sum{ r * (y - y0) / y0 }`

In simplified form:

`estimated_return = Σ(weight_i × holding_price_ratio_change_i)`

The code:
- loads stored holding ratios from `HoldingsSql`;
- compares each holding's current/target-date price with its price on the holdings anchor date;
- applies USD/CNY and HKD/CNY adjustments depending on listing currency;
- subtracts the original total weight to convert price ratios into returns;
- multiplies the total change by `RefGetPosition()`;
- applies the resulting return to the anchor NAV.

For RMB-denominated fund A shares, the final USD/CNY conversion is also applied so that foreign-asset movement is expressed in RMB NAV terms.

`GetOfficialNav()` estimates for the official valuation date.

`GetFairNav()` calls `_estNav(false,...)` when current FX/holding dates have moved beyond the official date, so it is a “current/fair” continuation estimate rather than an official NAV.

### 3. `php/stock.php`

`RefGetPosition($ref)` first reads `FundPositionSql`; if absent it falls back to the reference's default position.

This is the source of PalmMicro's displayed “仓位估算值使用 X”.

### 4. `php/stock/qdiiref.php`

`QdiiGetStockPosition()` estimates effective position from observed fund NAV change versus reference-asset-and-FX change.

The source defines:

`POSITION_EST_LEVEL = 4.0`

The function only attempts the position inference when the reference asset's absolute move exceeds this threshold, reducing noise from small moves.

Simplified idea:

`position ≈ actual_fund_NAV_return / FX-adjusted_reference_return`

### 5. `woody/res/php/_submitstockoptions.php`

The admin path `STOCK_OPTION_HOLDINGS` supports manual holdings editing.

For symbols not covered by the automatic SSE/SZSE PCF download switch, it calls:

`_updateStockOptionHoldings(...)`

That function:
- writes the holdings date;
- deletes the old stored holdings;
- parses admin input such as `SYMBOL*ratio;SYMBOL2*ratio`;
- inserts the new proxy holdings into `HoldingsSql`.

SH501312 is not in the automatic PCF-download switch, so its proxy holdings can be manually maintained by the site administrator.

## Practical interpretation

PalmMicro's SH501312 model is best understood as:

1. anchor to an official NAV;
2. apply price moves of a maintained proxy basket;
3. apply FX changes;
4. damp/scale by an effective position factor;
5. re-anchor when new official NAV becomes available.

This design limits long-term cumulative drift because official NAVs repeatedly reset the anchor, but it does **not** eliminate short-term model error after undisclosed manager rebalancing.

## Independent-estimate checklist

For a fresh SH501312 estimate:

1. Latest official NAV and `nav_date`.
2. Current PalmMicro proxy holdings/weights.
3. Current PalmMicro position factor.
4. Each proxy asset's anchor-date and current price.
5. USD/CNY and, if relevant, HKD/CNY at anchor/current valuation times.
6. Current SH501312 exchange price.
7. Redemption status/fee only when evaluating arbitrage.

If the current proxy holdings cannot be observed, do not silently use the old 2023 comment basket as though it were current. Label it clearly as a fallback historical proxy.
