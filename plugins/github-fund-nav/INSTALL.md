# Install GitHub Fund NAV Plugin

Plugin source repository:

`https://github.com/bvans/skills`

Plugin folder:

`plugins/github-fund-nav`

Marketplace:

`.agents/plugins/marketplace.json`

## ChatGPT desktop / local marketplace

If your ChatGPT desktop build exposes local plugin marketplaces, add the repository marketplace:

```bash
codex plugin marketplace add bvans/skills --ref main
```

Restart the ChatGPT desktop app, open **Plugins**, select **bvans Personal Plugins**, then install **基金净值查询**.

The GitHub plugin/app must remain connected because the skill reads and writes `bvans/fund-nav-public`.

## Business / Enterprise / Edu workspace import

Workspace admins can import the marketplace directly from GitHub:

- Repository: `https://github.com/bvans/skills`
- Path: leave blank (the manifest is at `.agents/plugins/marketplace.json`)
- Branch: `main`

After import, enable/install **基金净值查询** and make sure the GitHub app is available to the user.

## Current Plus-account limitation

OpenAI's current Skills help page lists direct skill creation/upload for eligible Business, Enterprise, Healthcare, and Edu users. If a Plus account does not show the Skills/custom marketplace entry, the package is ready but cannot be account-installed from that surface yet.

## Test prompts

- `查询 000218 最新正式净值`
- `刷新 110022 的最新净值`
- `查询 000218、162412、005051，告诉我各自净值日期`
