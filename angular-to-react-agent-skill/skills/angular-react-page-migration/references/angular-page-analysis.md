# Phase 2 — Angular Page Analysis

Systematic inventory of everything the Angular page does. Output goes into the page's status doc (`docs/migration/pages/<page-slug>.md`) using the table formats below. The port (Phase 3) implements *only* what this inventory lists — anything discovered later means this phase was incomplete.

## Procedure

Work from the route inward:

1. **Locate the entry.** Find the page's route definition in the Angular host (route path, module/component, lazy-loading, route params, query params). Note any guards and resolvers on the route.
2. **Walk the template tree.** Starting from the routed component's template, recursively list every component used. Classify each one (rules below).
3. **List services.** Every injected service on the page's components: what it does, whether it is page-local or shared, and every HTTP call it makes (method, endpoint, request/response shape, error handling, interceptor behavior that matters — auth headers, retries).
4. **Forms.** Identify each form: template-driven or reactive, fields, validators (sync and async), error display behavior, submit flow. Note what the React exemplars use for equivalent forms.
5. **Pipes and directives.** Every pipe (built-in and custom) and structural/attribute directive with behavioral meaning (not pure styling).
6. **Interop events.** Every interop routing event this page consumes or emits (cross-reference `references/interop-routing.md`): event name, payload, when fired, what reacts.
7. **Styles.** Where the page's styles live, any global styles it depends on, responsive breakpoints, animations/transitions.
8. **Observed behavior pass.** Open the page on DEV1 and exercise it against the Phase 1 spec (`specs/<page>.md`): loading states, empty states, error states, permissions/roles, edge inputs. Anything the code walk missed goes into the inventory now.

## Component classification rules

| Class | Rule | Action |
|---|---|---|
| `library` | Imported from the design-system Angular package (`<<FILL:angular-library-package>>`) | Look up React equivalent in the component map; if missing there, map it and add the row |
| `custom` | Defined in the app repo, not part of the design-system library | **Flag — human-judgment checkpoint.** Never port on a guess |
| `third-party` | From any other npm package | **Flag** — human decides: React equivalent package, library replacement, or rebuild |

## Flagging rules

Create a flag whenever:

- A component classifies as `custom` or `third-party` with no `decided` row in the component map
- A behavior cannot be fully derived from the Angular source (implicit interceptor magic, timing-dependent logic, code you cannot trace)
- The Angular page's observed behavior on DEV1 differs from what the code appears to do
- Two exemplar pages solve the same problem differently and it is unclear which pattern applies

Each flag is a row in the status doc's Flags table and is resolved only by a recorded human decision (format in SKILL.md, Human-Judgment Checkpoints). Batch flags and present them together — one interruption with five flags beats five interruptions.

## Inventory table formats (copy into the status doc)

```markdown
### Route & entry
| Field | Value |
|---|---|
| Angular route path | |
| Entry component | |
| Route/query params | |
| Guards / resolvers | |
| Lazy loaded | |

### Components
| Component | Class (library/custom/third-party) | React equivalent | Map status | Flag? |
|---|---|---|---|---|

### Services & API calls
| Service | Scope (page/shared) | HTTP calls (method, endpoint) | Notes (auth, errors, retries) |
|---|---|---|---|

### Forms
| Form | Type (template/reactive) | Fields & validators | Submit flow | React exemplar pattern |
|---|---|---|---|---|

### Pipes & directives
| Name | Kind | Behavior | React equivalent |
|---|---|---|---|

### Interop events
| Event | Direction (consume/emit) | Payload | When |
|---|---|---|---|

### Styles
| Aspect | Detail |
|---|---|
| Style location(s) | |
| Global dependencies | |
| Breakpoints / animations | |

### Flags
| # | Item | Question for human | Decision (link to decision log) |
|---|---|---|---|
```

## Phase 2 gate (restated)

- [ ] All seven inventory tables complete in the status doc
- [ ] Every component has a class and a map status
- [ ] Observed-behavior pass done against DEV1 and spec updated if it revealed gaps
- [ ] Every flag has a recorded human decision in the decision log
