<!-- Template: copy to docs/migration/pages/<page-slug>.md at Phase 1 start. This doc is the page's state machine — update it BEFORE leaving each phase; any session resumes at the first unchecked gate. Commit it with the migration PR. -->

# Migration Status — <Page Name>

- Slug: `<page-slug>`
- Angular route: `<path>`
- Started: <date> · Owner: <who>
- Environments used: DEV1 `<url>` · react-local `<url>` · DEV3 `<url>`

## Phase gates

- [ ] **Phase 0 — Recon current** — evidence: recon doc at commit `<sha>`, staleness check date
- [ ] **Phase 1 — Baseline green vs DEV1** — evidence: see Playwright evidence below
- [ ] **Phase 2 — Inventory complete, flags decided** — evidence: tables below, decision log
- [ ] **Phase 3 — Port renders/compiles/lints, interop nav works** — evidence: commands + output summary
- [ ] **Phase 4 — Vitest green · Storybook builds · parity green (local + DEV3)** — evidence: below
- [ ] **Phase 5 — Handoff complete** — evidence: SKILL.md Verification checklist all checked

## Inventory (Phase 2)

<!-- Paste the filled table formats from references/angular-page-analysis.md: -->

### Route & entry

### Components

### Services & API calls

### Forms

### Pipes & directives

### Interop events

### Styles

### Flags
| # | Item | Question for human | Decision (link to decision log) |
|---|---|---|---|

## Decision log

<!-- One line per human decision: -->
<!-- YYYY-MM-DD | <who> | <what was decided> | <why> -->

## Playwright evidence

- Spec: `specs/<page>.md` (commit `<sha>`)
- Suite: `tests/<page>.spec.ts` (commit `<sha>`)
- Baseline run (angular-dev1):
- Parity run (react-local):
- Parity run (react-dev3):
- Post-heal diff: <selectors only / none>

## Verify evidence (Phase 4)

- Vitest: `<command>` → <result>
- Storybook: stories at `<paths>`, build → <result>
- Interop nav manual check: <in / out / back-forward results>

## Skill discrepancies noticed

<!-- Where an exemplar or this repo contradicted the skill's guidance — feeds skill corrections. -->

## Handoff summary (Phase 5)

<!-- PR-ready: what was ported, flags raised, decisions made, evidence links. -->
