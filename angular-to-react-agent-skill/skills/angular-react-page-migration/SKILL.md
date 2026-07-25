---
name: angular-react-page-migration
description: Ports Angular pages to React with strict 1:1 behavioral parity in a hybrid Angular/React monorepo, preserving the existing interop routing mechanism and validating parity with Playwright test agents. Use when migrating, porting, or converting an Angular page, view, or route to React. Use when resuming a partially migrated page. Use when verifying parity between an Angular page and its React port.
---

# Angular → React Page Migration

## Overview

This skill migrates one Angular page at a time to React in a hybrid monorepo where both frameworks run side by side. The discipline is fixed: **recon before porting, baseline before change, evidence over eyeballing**. Every migration captures the Angular page's behavior as a Playwright test suite first, ports the page 1:1 using the conventions already proven by previously migrated pages, then proves parity by running the *same unmodified suite* against the React version.

The skill maintains three working artifacts inside the app repo — a recon doc, a component map, and a per-page migration status doc — so work is resumable across sessions and human-reviewable in PRs.

## When to Use

Use this skill when:

- Migrating, porting, or converting an Angular page/view/route to React
- Resuming a partially migrated page (a status doc exists for it)
- Re-running or repairing parity validation for an already-ported page

Do NOT use this skill for:

- Building new React features that have no Angular counterpart
- Redesigning UX or visuals ("while we're here" improvements are out of scope)
- Changing the routing architecture or the interop mechanism itself
- Migrating shared infrastructure (build config, CI, shared services) — those are separate, human-planned efforts

## Ground Rules

These are non-negotiable and apply to every phase:

1. **Strict 1:1 parity.** Same UX, same behavior, same edge cases, same copy. If the React version is "better" than the Angular version, that is a bug — parity first, improvements as a separate ticket afterward.
2. **The interop routing mechanism is used as-is.** Angular owns routing and communicates with React via the established window-event mechanism (`<<FILL:interop-mechanism-name>>`, proven on 5+ migrated pages). This skill wires new pages into it. It never modifies, refactors, or "improves" it. The diff under `<<FILL:interop-file-paths>>` must be empty at handoff.
3. **Migrated pages are the convention source of truth.** When this skill's guidance and an existing migrated page disagree, the migrated page wins. Note the discrepancy in the status doc so the skill can be corrected.
4. **The team's `angular-to-react` standards file outranks everything.** When the team provides a coding-standards / migration-equivalents file, its mappings override both this skill and exemplar inference. Precedence: team standards file > exemplar migrated pages > this skill's catalogs.
5. **Custom components require a recorded human decision.** Any Angular component that is not from the shared design-system library triggers a human-judgment checkpoint (see below). Never guess.
6. **No phase starts until the previous phase's gate has evidence** written into the page's status doc. Confidence is not evidence; command output, file paths, and decision notes are.

## Working Artifacts

All three live in the app repo and are committed with the migration PR so reviewers see the decision trail, not just code. Colocate under the repo's existing migration docs location if one exists; otherwise use `docs/migration/`.

| Artifact | Path in app repo | Template | Created | Updated |
|---|---|---|---|---|
| Recon doc | `docs/migration/recon.md` | `references/recon.md` (embedded) | Phase 0, once per repo | On staleness (see Phase 0) |
| Component map | `docs/migration/component-map.md` | `templates/component-map.md` | Phase 0, once per repo | Every flag resolution (Phase 2) and handoff (Phase 5) |
| Page status doc | `docs/migration/pages/<page-slug>.md` | `templates/migration-status.md` | Phase 1 start, per page | Before leaving every phase |

**Resume rule:** on any invocation for a page, first check for its status doc. If it exists, read it and resume at the first unchecked gate. Never restart a phase whose gate is already checked with evidence.

## The Migration Workflow

| Phase | Purpose | Gate | Reference |
|---|---|---|---|
| 0. Recon & Calibration | Learn the repo; correct this skill's assumptions | Recon doc current, placeholders resolved | `references/recon.md` |
| 1. Playwright baseline | Capture Angular behavior as a green test suite | Baseline green vs Angular (DEV1) | `references/playwright-parity.md` |
| 2. Page analysis | Inventory everything the Angular page does | Inventory complete, flags decided | `references/angular-page-analysis.md` |
| 3. Port | Build the React page 1:1 | Renders, compiles, lints, interop nav works | `references/porting-patterns.md`, `references/interop-routing.md` |
| 4. Verify | Prove parity with evidence | Vitest + Storybook + unchanged parity suite green | `references/playwright-parity.md` |
| 5. Handoff | Finalize artifacts, PR-ready summary | All Verification checkboxes evidenced | (below) |

### Phase 0 — Recon & Calibration (once per repo, staleness-checked)

Runs once per repo, not per page. Check whether `docs/migration/recon.md` exists and is current (staleness rule in the reference). If current, skip to Phase 1.

Otherwise, read `references/recon.md` and:

1. **Recon**: analyze the React app's structure and tooling (folder layout, data fetching, Vitest patterns, Storybook usage), read 2–3 already-migrated pages end-to-end, and nominate them as exemplars. Produce/refresh the recon doc.
2. **Calibration** (first run only): walk every assumption this skill makes — folder layout, commands, tooling, env URLs, Playwright setup shape, placeholder values marked `<<FILL:...>>` — and verify each against the actual repo. Record every mismatch in the recon doc's **Assumption Corrections** section (the recon doc overrides this skill on conflict). Where a mismatch is structural, edit this skill's reference files directly so later runs start correct.
3. **Standards ingestion**: if the team's `angular-to-react` coding-standards/migration-equivalents file is available, reconcile `references/porting-patterns.md` against it (team file wins) and record its path in the recon doc. If not yet provided, note it as pending.
4. Seed `docs/migration/component-map.md` from the design-system library's Angular/React component pairs plus mappings already used by the exemplar pages.

**Gate:** recon doc exists and is current; exemplar pages named; all `<<FILL:...>>` placeholders resolved; assumption corrections recorded; standards file ingested or noted pending.

### Phase 1 — Playwright Baseline (Angular, before)

The Angular page has no tests. This phase creates them — the baseline suite *is* the Angular page's behavioral spec.

Read `references/playwright-parity.md` and:

1. Create the page's status doc from `templates/migration-status.md`.
2. Verify/repair the Playwright test-agents setup (checklist in the reference; the setup is known to be partially working).
3. Use the **planner** agent against the live Angular page (DEV1 environment) to produce `specs/<page>.md`, then the **generator** to produce `tests/<page>.spec.ts`. Enforce the selector mandate: role/label/text selectors only — the DOM will change between Angular and React; behavior must not.
4. Run the suite against DEV1 until green.

**Gate:** baseline suite green against the Angular page on DEV1; run output linked in the status doc.

### Phase 2 — Angular Page Analysis

Read `references/angular-page-analysis.md` and produce a complete inventory in the status doc: route/module entry, every component classified (design-system library / custom / third-party), injected services and their HTTP calls, forms, pipes, guards/resolvers, interop events consumed and emitted, and styles.

Every custom component and every behavior you cannot fully derive from the Angular source becomes a **flag**. Flags are resolved only by a recorded human decision (see Human-Judgment Checkpoints).

**Gate:** inventory tables complete in the status doc; every flag has a recorded decision.

### Phase 3 — Port

Read `references/porting-patterns.md` and `references/interop-routing.md`, then implement the React page:

1. Follow the recon doc's conventions and copy structure from the exemplar migrated pages.
2. Swap components via the component map; use the React flavor of the design-system library.
3. Apply the idiom catalog with its precedence rule (team standards file > exemplars > catalog).
4. Wire the page into the interop routing mechanism using the wiring checklist — registration, inbound navigation, outbound navigation.

**Gate:** page renders on the local React dev server; TypeScript compiles; lint passes; interop navigation into and out of the page works manually.

### Phase 4 — Verify

1. Write Vitest tests following the patterns the React app already uses (per recon doc).
2. Write Storybook stories following existing usage.
3. Run the **unchanged** Phase 1 suite against the React page — first on the local dev server, then on DEV3. The **healer** agent may fix *selectors only*. If the healer wants to change an *assertion*, stop: the port has a behavior bug — fix the port, not the test.

**Gate:** Vitest green; Storybook builds with the new stories; parity suite green against DEV3 with run output linked in the status doc.

### Phase 5 — Handoff

1. Complete the Verification checklist below inside the status doc, each item with evidence.
2. Append any new component mappings and decisions to the component map.
3. Confirm zero diff under the interop mechanism's paths (`git diff --stat` on `<<FILL:interop-file-paths>>`).
4. Write a short PR-ready summary: what was ported, flags raised, decisions made, evidence links.

**Gate:** every Verification checkbox checked with evidence.

## Human-Judgment Checkpoints

Stop and ask a human — do not proceed on a guess — in exactly these situations:

1. **Custom component with no library equivalent** (Phase 2): present the component, its usage, and candidate options (port it, replace with a library composite, rebuild). Wait for a decision.
2. **Behavior you cannot reproduce or fully understand from the Angular source** (Phases 2–3): describe what is unclear and what you observed on DEV1.
3. **Healer proposes an assertion change** (Phase 4): this means observed behavior diverged. Present the divergence; the human decides whether it is a port bug (default assumption) or an accepted difference.

Record every decision as one line in the status doc's decision log: `YYYY-MM-DD | <who> | <what was decided> | <why>`.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This page is simple, skip the baseline" | Simple pages hide interop and edge-case behavior. The baseline takes minutes; a parity bug found in production takes days. |
| "I'll improve the UX while I'm here" | Scope is 1:1 parity. Improvements are a separate ticket after parity is proven. |
| "The interop mechanism is clunky; a cleaner bridge would…" | It is proven across 5+ pages. Redesigning it is permanently out of scope for this skill. |
| "The recon doc is a few pages old, close enough" | Conventions drift with every merged page. Run the staleness check. |
| "The healer fixed the test, so parity holds" | The healer fixes *selectors*. If it changed an *assertion*, the port's behavior diverged. |
| "Angular has no tests, so parity can't be verified anyway" | That is why Phase 1 exists: the baseline suite *is* the Angular page's test suite. |
| "I'll add Vitest tests and stories after all pages are migrated" | Each page ships with its tests and stories or the Phase 4 gate fails. Batch-later never happens. |
| "This custom component obviously maps to library component X" | 'Obvious' mappings are still recorded human decisions. Flag it. |
| "The DOM is different in React so I rewrote the test" | Use role/text selectors from the start. A rewritten test no longer proves parity. |
| "I already know this codebase, skip recon" | The gate needs the *artifact*, not your confidence. Later sessions and other agents resume from the recon doc. |

## Red Flags

Stop and re-check the workflow if any of these appear:

- The diff touches files under the interop mechanism's paths
- The React page has UI elements, states, or copy absent from the Angular page (or vice versa)
- A parity spec or test assertion was edited during Phase 4
- A data-fetching or state pattern appears that exists in no migrated exemplar page
- Porting started with no status doc or no green baseline
- A custom component was ported with an empty decision log
- The component map was not updated at handoff
- A `<<FILL:...>>` placeholder is still unresolved after Phase 0

## Verification

Definition of done for one page migration — every box requires linked evidence in the status doc:

- [ ] Baseline suite green against the Angular page on DEV1 (run output linked)
- [ ] The *same, unmodified* suite green against the React page locally
- [ ] The *same, unmodified* suite green against the React page on DEV3
- [ ] Vitest tests for the new page pass
- [ ] Storybook stories exist and Storybook builds
- [ ] Interop navigation into and out of the page works
- [ ] Every human-judgment checkpoint has a recorded decision
- [ ] Status doc: all phase gates checked with evidence
- [ ] Component map updated with this page's mappings and decisions
- [ ] `git diff --stat` shows zero changes under the interop mechanism's paths
