# GitHub Fund NAV Plugin

OpenAI portable finance skill plugin for querying **official** NAVs of Chinese public funds through the GitHub Actions workflow in `bvans/fund-nav-public`, plus Eastmoney-authoritative A-share index quotes/drawdown context and model-based QDII-LOF fair-NAV/premium-discount analysis.

## Package

- `plugin.json` — portable Agent Plugins manifest
- `skills/github-fund-nav-query/SKILL.md` — latest official NAV workflow
- `skills/github-fund-nav-history-query/SKILL.md` — historical NAV and transaction-return workflow
- `skills/github-index-market-query/SKILL.md` — A-share index latest quote/source-time/drawdown workflow
- `skills/github-index-market-query/references/index-market-contract.md` — index quote/history source contract
- `skills/qdii-lof-premium-estimator/SKILL.md` — QDII-LOF fair NAV and premium/discount estimation
- `skills/qdii-lof-premium-estimator/references/palmmicro-model.md` — PalmMicro source-code model notes
- `skills/github-fund-nav-query/references/repository-contract.md` — NAV repository paths and schemas

The official-NAV skills expect the GitHub app/connector to be connected with read/write access to `bvans/fund-nav-public`.

The QDII-LOF estimator keeps official NAV, model-estimated Fair NAV, exchange price, and premium/discount separate. For SH501312 it can use PalmMicro's live proxy-holdings model as one input, but it does not treat PalmMicro's estimate as an official IOPV or assume that the displayed holdings date is a true full-portfolio disclosure date.

It never treats intraday estimated NAV (`gsz`) as official NAV.

The index-market skill treats Eastmoney `push2` as the authoritative latest-quote source. Tencent `newfqkline` is allowed only as an explicitly labeled historical fallback for recent-high/drawdown calculations when Eastmoney `push2his` is unavailable from GitHub-hosted runners.
