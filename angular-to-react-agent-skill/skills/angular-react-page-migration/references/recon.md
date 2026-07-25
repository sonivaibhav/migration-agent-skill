# Phase 0 — Recon & Calibration

Run once per repo. Re-run only when the staleness rule fires. Output: `docs/migration/recon.md` (template at the bottom) and a seeded `docs/migration/component-map.md`.

## Staleness rule

The recon doc records the commit SHA it was generated at. Refresh the recon doc when either:

1. One or more migrated pages have merged since that SHA (`git log <sha>..HEAD -- <react-app-path>` shows page-level changes), or
2. During any phase you find a contradiction between the recon doc and the actual repo. Stop, fix the recon doc, then continue.

A refresh re-runs the Recon procedure below; Calibration and Standards ingestion only need re-running if their inputs changed.

## Part 1 — Recon procedure

Answer every question by reading code, not by assumption. Record answers in the recon doc.

### Repo topology

- Where do the Angular host and the React app live in the monorepo? What are their root paths?
- What package manager and workspace tooling is used? What are the exact commands to install, run the Angular host locally, run the React dev server, run Vitest, run Storybook, and run Playwright?
- What are the deployed environment URLs? Expected: Angular on DEV1 (`<<FILL:dev1-url>>`), React on DEV3 (`<<FILL:dev3-url>>`), React local dev server (`<<FILL:react-local-url>>`). Verify against CI/deploy config or team docs.

### React app conventions

- Folder layout: where do pages/routes, components, hooks, utils, and tests live? What is the naming convention?
- How does a React page register itself so the Angular host can route to it? (Cross-check with `references/interop-routing.md`.)
- Data fetching: what do existing pages use (plain fetch, axios, a wrapper, React Query)? Where do API clients and error handling live?
- Styling: CSS modules, styled-components, design-system tokens? What do migrated pages use?
- Vitest: where do test files sit relative to source? What setup/utils/render helpers exist? Pick one representative test file as the pattern to copy.
- Storybook: which components/pages have stories, where do stories live, and what story format is used? Pick one representative story.

### Exemplar pages

Read **2–3 already-migrated pages end-to-end** (component tree, data fetching, interop wiring, tests, stories). Nominate them as exemplars in the recon doc with one line each on why. If a human has already declared a canonical exemplar, use that one first.

### Component map seeding

Create `docs/migration/component-map.md` from `templates/component-map.md`, seeded with:

1. Every design-system library component pair: enumerate the Angular flavor's exports (`<<FILL:angular-library-package>>`) and match to the React flavor's exports (`<<FILL:react-library-package>>`). Status `mapped`.
2. Every mapping already used by the exemplar pages (grep their imports), including any custom-component decisions visible in their history. Status `decided`, with a note pointing at the exemplar.

## Part 2 — Calibration (first run only)

This skill was written outside the app repo and marks its guesses with `<<FILL:...>>` placeholders. Calibrate it against reality:

1. **Enumerate assumptions.** Grep the skill directory for `<<FILL:` — every hit is an unverified value. Beyond placeholders, verify these baked-in assumptions: Playwright test agents are initialized with `--loop=vscode`; specs live in `specs/` and generated tests in `tests/`; the app-repo docs location `docs/migration/` is acceptable; Vitest and Storybook commands exist as recon found them.
2. **Verify each against the repo.** Fill placeholder values in the recon doc AND replace the `<<FILL:...>>` markers in this skill's files with the real values, so future runs need no lookup.
3. **Record corrections.** Any assumed *step* that doesn't match reality (e.g. tests live elsewhere, a different agents loop is used, docs belong in another folder) goes into the recon doc's **Assumption Corrections** table. The recon doc overrides the skill wherever they conflict.
4. **Edit structurally.** If a mismatch invalidates a whole procedure (not just a value), edit the affected reference file in the skill directly and note the edit in the corrections table.

## Part 3 — Standards ingestion

The team maintains an `angular-to-react` coding-standards / migration-equivalents file (authoritative pattern mappings).

- If available now: read it, then reconcile `references/porting-patterns.md` against it. Where the team file and the catalog disagree, rewrite the catalog entry to match the team file and tag it `(team standard)`. Record the file's path in the recon doc.
- If not yet provided: record `pending` in the recon doc and raise it with the human. Until then, exemplar pages remain the top precedence.

Precedence (also stated in SKILL.md Ground Rules): **team standards file > exemplar migrated pages > this skill's catalogs.**

## Recon doc template

Copy this into `docs/migration/recon.md` and fill it:

```markdown
# Migration Recon

Generated at commit: <sha>
Generated on: <date>

## Topology
- Angular host root: <path>
- React app root: <path>
- Commands: install `<cmd>` · angular serve `<cmd>` · react serve `<cmd>` · vitest `<cmd>` · storybook `<cmd>` · playwright `<cmd>`
- Environments: DEV1 (Angular) `<url>` · DEV3 (React) `<url>` · react local `<url>`

## React conventions
- Pages/routing layout: ...
- Page registration with interop: ...
- Data fetching: ...
- Styling: ...
- Vitest pattern file: <path>
- Storybook pattern file: <path>

## Exemplar pages
| Page | Path | Why exemplar |
|---|---|---|

## Team standards file
Path: <path or "pending">
Patterns it overrides: <list or "none yet">

## Assumption Corrections
| Skill assumption | Reality in this repo | Correction applied where |
|---|---|---|
```

## Phase 0 gate (restated)

- [ ] Recon doc exists, current per staleness rule, all sections filled
- [ ] 2–3 exemplar pages nominated
- [ ] No unresolved `<<FILL:...>>` markers remain in the skill
- [ ] Assumption Corrections table complete (or explicitly empty)
- [ ] Standards file ingested, or recorded as pending with the human notified
- [ ] Component map created and seeded
