# migration-agent-skill

Authoring home for the **`angular-react-page-migration`** agent skill: a gated, evidence-driven workflow for porting Angular pages to React with strict 1:1 parity in a hybrid monorepo, validated end-to-end with [Playwright Test Agents](https://playwright.dev/docs/test-agents).

The sibling [`agent-skill/`](../agent-skill/) repo (vendored copy of addyosmani/agent-skills) is the conventions reference (skill anatomy, linter) only — nothing in it is modified here.

## Layout

```
skills/angular-react-page-migration/   # the skill — self-contained, copy this dir wholesale
  SKILL.md                             # workflow spine: ground rules, 6 phases, gates, DoD
  references/                          # loaded per phase (progressive disclosure)
    recon.md                           # Phase 0: recon, one-time calibration, standards ingestion
    angular-page-analysis.md           # Phase 2: inventory procedure + flagging rules
    porting-patterns.md                # Phase 3: Angular→React idiom catalog (team standards win)
    interop-routing.md                 # Phases 3–4: interop discovery + wiring checklist
    playwright-parity.md               # Phases 1 & 4: setup repair, baseline, parity, healer rules
  templates/                           # instantiated INTO the app repo, not read as guidance
    migration-status.md                # per-page state machine (docs/migration/pages/<slug>.md)
    component-map.md                   # app-wide mapping table (docs/migration/component-map.md)
agents/
  angular-react-migrator.agent.md      # thin Copilot persona; all substance stays in SKILL.md
```

## Install into the app repo (GitHub Copilot)

```bash
APP=~/path/to/app-repo
mkdir -p "$APP/.github/skills" "$APP/.github/agents"
cp -R skills/angular-react-page-migration "$APP/.github/skills/"
cp agents/angular-react-migrator.agent.md "$APP/.github/agents/angular-react-migrator.agent.md"
```

Notes:

- The `.agent.md` suffix is **mandatory** — Copilot silently ignores plain `.md` agent files.
- Playwright's `npx playwright init-agents --loop=vscode` also writes agent files into `.github/`. Never overwrite those; this skill *consumes* them.
- Optional discoverability line for `.github/copilot-instructions.md`:
  `Angular→React page migrations must follow the angular-react-page-migration skill.`
- Source of truth stays here; re-copy after skill edits (copy, not symlink).

Invoke in Copilot chat: `@angular-react-migrator migrate the <page> page`.

## Placeholders

Repo-specific values are marked `<<FILL:...>>` throughout the skill. The skill's Phase 0 (Recon & Calibration) resolves them all on first run in the app repo, records assumption corrections, and ingests the team's `angular-to-react` coding-standards file when provided. Find remaining markers with:

```bash
grep -rn '<<FILL:' skills/
```

## Validating the skill (structural)

Uses the sibling `agent-skill/` repo's linter without touching it (run from this directory):

```bash
node -e "console.log(JSON.stringify(require('$PWD/../agent-skill/scripts/lib/skill-lint.js').lintSkill('angular-react-page-migration','$PWD/skills', new Set(['angular-react-page-migration'])), null, 2))"
wc -l skills/angular-react-page-migration/SKILL.md   # must be < 500
```

## Dry-run observation checklist

Score the skill on its first real page (smallest pending page, on a branch, invoked with only `@angular-react-migrator migrate <page>`). Every miss becomes a skill edit — usually a new Common Rationalizations row or a sharpened gate.

- [ ] Ran/refreshed Phase 0 recon (and calibration on first run) before touching any code
- [ ] Created the page status doc before the baseline
- [ ] Got a green Playwright baseline against the Angular page (DEV1) before writing any React
- [ ] Used role/label/text selectors in generated tests (no DOM-structure selectors)
- [ ] Stopped and asked at the first custom component instead of guessing
- [ ] Left interop mechanism files untouched (`git diff --stat` on those paths)
- [ ] Ran the unmodified baseline suite against the React port (local, then DEV3)
- [ ] Healer changes were selector-only; any behavior divergence escalated to a human
- [ ] Updated the component map and completed the status doc at handoff
