---
name: qdii-lof-premium-estimator
description: Estimate fair NAV and premium/discount for Chinese QDII LOFs, especially SH501312 华宝海外科技股票(QDII-LOF)A, using the latest official NAV anchor, current proxy holdings, market prices, and FX. Use when the user asks whether a QDII LOF is at a discount/premium, what a reasonable intraday NAV is, whether PalmMicro's estimate is trustworthy, or whether a discount is large enough to study for redemption arbitrage. Never present an estimated NAV as official NAV.
---

This skill estimates **fair/intraday NAV**, not official NAV. For official NAV, use the sibling skill `github-fund-nav-query` and preserve its actual `nav_date`.

Read `references/palmmicro-model.md` before the first detailed 501312 calculation in a conversation.

## Core principles

Keep four quantities separate:

1. **Official NAV** — the fund company's formally disclosed unit NAV for a specific `nav_date`.
2. **Estimated/Fair NAV** — a model-based estimate after the latest official NAV anchor.
3. **Exchange price** — the current LOF secondary-market price.
4. **Premium/discount** — calculated against the chosen NAV basis.

Never label an estimate as official NAV. Always say which NAV basis is used.

## Preferred workflow for SH501312

1. Obtain the latest official NAV and its actual date using `github-fund-nav-query`.
2. Inspect PalmMicro's current SH501312 holdings/estimate page when available:
   `https://www.palmmicro.com/woody/res/holdingscn.php?symbol=SH501312`
3. Extract, when available:
   - current proxy holding symbols and weights;
   - position factor (仓位估算值);
   - displayed holdings/anchor date;
   - official EST / fair EST;
   - any displayed premium/discount.
4. Do **not** interpret PalmMicro's displayed “基金持仓更新于 YYYY-MM-DD” as proof that the fund disclosed its true complete holdings on that date. For SH501312, PalmMicro source code advances this holdings date to the latest NAV date without necessarily changing the stored proxy weights.
5. If the live PalmMicro proxy basket is available, use it rather than the old source-code comment basket.
6. Obtain current/anchor prices for the proxy assets and relevant FX rates from current public market sources.
7. Recalculate an independent fair NAV from the official NAV anchor.
8. Compare:
   - your independent fair NAV;
   - PalmMicro fair EST;
   - current exchange price.
9. Report a range/uncertainty when the proxy basket is stale, incomplete, or the fund may have materially rebalanced.

## Estimation method

For a proxy holding `i` with stored weight `w_i`:

`asset_return_i = current_value_i / anchor_value_i - 1`

Adjust returns for currency when necessary. A practical currency-aware approximation is:

`r_i_rmb = (1 + local_asset_return_i) * (1 + currency_return_vs_cny_i) - 1`

Then:

`proxy_return = Σ(w_i * r_i_rmb)`

Apply the PalmMicro-style effective position factor `p` when one is available:

`estimated_fund_return = p * proxy_return`

Then:

`fair_nav = official_nav_anchor * (1 + estimated_fund_return)`

For exact reproduction of PalmMicro's implementation details, follow `references/palmmicro-model.md`.

### Premium/discount

Use:

`premium_discount = exchange_price / fair_nav - 1`

Interpretation:
- positive: premium;
- negative: discount.

State the percentage with the basis, e.g. “相对独立估算 Fair NAV 折价约 1.2%”.

## SH501312 uncertainty rules

SH501312 is an actively managed QDII-LOF. Its true intraday full holdings are not public.

Therefore:
- do not treat a sub-1% model discount as a risk-free arbitrage;
- do not claim precision to two decimals in the premium/discount unless the model inputs justify it;
- explicitly flag potential model error from manager rebalancing, undisclosed holdings, FX valuation timing, cash, fees, and overseas-market timing;
- when the independently estimated NAV and PalmMicro Fair EST differ materially, show both and explain why.

As a practical classification for **model confidence**, not investment advice:
- |discount| < 1%: usually within plausible model noise for an active QDII-LOF;
- 1%–2%: worth validating with an independent calculation;
- >2%–3%: potentially meaningful, but still verify redemption status/fees and NAV timing before discussing arbitrage.

Do not turn these bands into guaranteed-profit thresholds.

## Redemption-arbitrage checks

When the user asks whether a discount can be captured by redemption, verify separately:

1. The product is a LOF and redemption is currently open.
2. The user's broker supports the required on-exchange redemption / transfer path.
3. Minimum redemption units.
4. Short-holding redemption fee schedule.
5. Expected settlement time.
6. QDII market and FX exposure between purchase and redemption NAV determination.

For 501312, never infer profitability from the displayed discount alone. Compare the estimated gross discount against redemption fee, commission, model uncertainty, and NAV movement risk.

## Output format

Prefer a compact table:

| 项目 | 数值 | 口径/日期 |
|---|---:|---|
| 最新正式NAV | ... | YYYY-MM-DD |
| 独立估算Fair NAV | ... | 当前估算 |
| PalmMicro Fair EST | ... | 如可用 |
| 场内价格 | ... | 当前 |
| 独立估算折溢价 | ... | 场内价/Fair NAV-1 |

Then give:
- the main conclusion;
- key uncertainty sources;
- whether the observed discount/premium is large enough to merit further redemption-arbitrage checks.

When exact live inputs are unavailable, say which inputs are missing and do not fabricate them.
