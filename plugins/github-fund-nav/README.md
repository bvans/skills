# GitHub Fund NAV Plugin

OpenAI portable skill plugin for querying the latest **official** unit NAV of specified Chinese public funds through the GitHub Actions workflow in `bvans/fund-nav-public`.

## Package

- `plugin.json` — portable Agent Plugins manifest
- `skills/github-fund-nav-query/SKILL.md` — workflow instructions
- `skills/github-fund-nav-query/references/repository-contract.md` — repository paths and schemas

The skill expects the GitHub app/connector to be connected with read/write access to `bvans/fund-nav-public`.

It can query funds outside the configured holding list by updating `query.json`, waiting for the on-demand GitHub Action, and verifying `data/manual_latest_nav.json`.

It never treats intraday estimated NAV (`gsz`) as official NAV.
