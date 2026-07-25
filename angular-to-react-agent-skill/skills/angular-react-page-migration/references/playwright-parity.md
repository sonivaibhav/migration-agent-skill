# Playwright Parity — Setup, Baseline, and Verification (Phases 1 & 4)

The parity backbone: capture the Angular page's behavior as a Playwright suite (Phase 1), then run the *same unmodified suite* against the React port (Phase 4). Uses [Playwright Test Agents](https://playwright.dev/docs/test-agents): **planner** (explores the app → markdown test plan), **generator** (plan → executable tests, verified live), **healer** (runs tests, repairs failures).

## Part A — Setup / repair checklist

The repo's Playwright setup is known to be partially working. Before Phase 1 of the *first* page (and whenever a later page hits setup errors), walk this checklist; fix forward at the first failing step.

1. **Package present**: `@playwright/test` in the repo's dependencies; `npx playwright --version` runs. Fix: install per the repo's package manager; `npx playwright install` for browsers.
2. **Test-agent definitions present**: `npx playwright init-agents --loop=vscode` has been run and its agent definitions exist under `.github/` (the `vscode` loop matches the team's Copilot/VS Code environment; VS Code ≥ 1.105 required). Fix: run the init command. **Never overwrite or hand-edit the generated agent files** — regenerate them via the same command after Playwright upgrades.
3. **Config sanity**: `playwright.config.*` exists and defines the three targets (Part B). Fix: add the projects/env-var handling below.
4. **Seed test passes**: `tests/seed.spec.ts` exists and passes against at least one target. The seed test encodes environment context (base URL handling, any auth/login steps) that the planner and generator reuse. Fix: repair the seed test first — nothing else works until it is green.
5. **Both apps reachable**: DEV1 (`<<FILL:dev1-url>>`) serves the Angular page; the local React dev server starts; DEV3 (`<<FILL:dev3-url>>`) serves the React app. If DEV environments need VPN/auth, encode that in the seed test/config, not in individual tests.

Record setup fixes in the page status doc (they are evidence for the Phase 1 gate).

## Part B — Target parameterization

The same suite must run against three base URLs. Use Playwright projects (or an env var if the repo already does — recon/calibration decides):

```ts
// playwright.config.ts — shape, adjust to repo conventions
projects: [
  { name: 'angular-dev1',  use: { baseURL: process.env.DEV1_URL } },
  { name: 'react-local',   use: { baseURL: 'http://localhost:<<FILL:react-local-port>>' } },
  { name: 'react-dev3',    use: { baseURL: process.env.DEV3_URL } },
],
```

Run with `npx playwright test tests/<page>.spec.ts --project=angular-dev1` (etc.). Tests must reference routes relative to `baseURL` only — never hardcode a host.

## Part C — Baseline capture (Phase 1, against Angular on DEV1)

1. **Plan**: invoke the **planner** agent against the live Angular page on DEV1. Prompt it with the page's route and scope: user flows, form validation behavior, loading/empty/error states, navigation in and out. Output: `specs/<page>.md`.
2. **Human skim**: the spec is the page's behavioral contract for the whole migration — skim it for missing flows (permissions, edge inputs) before generating. Add missing scenarios to the spec now; adding them after the port weakens the parity argument.
3. **Generate**: invoke the **generator** agent to turn the spec into `tests/<page>.spec.ts`, validated live against DEV1.
4. **Selector mandate** (enforce during generation; re-check the produced file): selectors must be role/label/text/testid-based (`getByRole`, `getByLabel`, `getByText`) — **never** DOM-structure selectors (CSS descendant chains, nth-child, framework-generated classes). The DOM *will* differ between Angular and React; behavior must not. A structure-dependent selector makes the parity run meaningless. Rewrite any offender before accepting the baseline.
5. **Green**: run the suite with `--project=angular-dev1` until green. Flakes get fixed now (waits on visible state, not timeouts), not suppressed.
6. **Evidence**: link the spec path, test path, and passing run output in the status doc.

## Part D — Parity run (Phase 4, against React)

1. Run the **unchanged** Phase 1 suite: first `--project=react-local` during development, then `--project=react-dev3` for the gate evidence.
2. On failures, the **healer** agent may repair **selectors only** (e.g. a testid that moved). Its scope rule:
   - **Selector change** → allowed. Verify the new selector still targets the semantically same element.
   - **Assertion change** → **forbidden.** An assertion change means observed behavior diverged: the port has a bug. Fix the port. If the divergence looks like it might be an acceptable difference, that is a human-judgment checkpoint (SKILL.md) — the human decides, and the decision is logged before any test edit.
3. **Diff discipline**: after healing, `git diff specs/<page>.md tests/<page>.spec.ts` must show selector-level changes only. Paste that diff (or "no changes") into the status doc.
4. **Evidence**: link passing run output for `react-local` and `react-dev3` in the status doc.

## Evidence format (status doc)

```markdown
### Playwright evidence
- Spec: specs/<page>.md (commit <sha>)
- Suite: tests/<page>.spec.ts (commit <sha>)
- Baseline run (angular-dev1): PASS <n> tests — <log link or pasted summary>
- Parity run (react-local): PASS — ...
- Parity run (react-dev3): PASS — ...
- Post-heal diff: selectors only / none
```
