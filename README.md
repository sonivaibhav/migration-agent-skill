# migration-agent-skill

Agent skills for migrating Angular pages to React with strict 1:1 parity.

- **[`angular-to-react-agent-skill/`](angular-to-react-agent-skill/)** — the `angular-react-page-migration` skill: a gated, evidence-driven workflow (recon & calibration → Playwright baseline → page analysis → 1:1 port → parity verification → handoff) plus a GitHub Copilot persona. See its [README](angular-to-react-agent-skill/README.md) for install and usage.
- **[`agent-skill/`](agent-skill/)** — vendored snapshot of [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) (MIT), used as the conventions reference and structural linter for authoring the skill. Not modified here.
