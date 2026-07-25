<!-- Template: copy to docs/migration/component-map.md in Phase 0 (once per repo). Seed from the design-system library's Angular/React pairs and the exemplar pages' imports. Append rows at every Phase 2 flag resolution and Phase 5 handoff. Commit with migration PRs. -->

# Angular → React Component Map

Single source of truth for component mappings across all page migrations.

- Angular library package: `<package>`
- React library package: `<package>`
- Seeded at commit: `<sha>` · Last updated: <date>

Status values: `mapped` (library pair, no decision needed) · `needs-human` (open flag) · `decided` (human decision recorded — link it).

| Angular component | React equivalent | Type (library / custom / third-party) | Status | Decision note / adaptation |
|---|---|---|---|---|
| | | | | |

## Usage rules

1. Before porting any component, look it up here. A `needs-human` row blocks the port of that component until decided.
2. Library pairs with API differences (props, variants, events) get the adaptation noted in the last column — the next page reuses it.
3. Decisions reference the page status doc's decision log where they were made (e.g. `decided — see pages/settings.md 2026-07-30`).
