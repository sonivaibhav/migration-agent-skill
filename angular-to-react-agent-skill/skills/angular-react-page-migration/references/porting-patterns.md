# Phase 3 — Porting Patterns (Angular → React)

Idiom catalog for the 1:1 port. **Precedence rule: the team's `angular-to-react` standards file (when provided) > exemplar migrated pages > this catalog.** Every pattern below ends the same way: confirm against an exemplar page — if the exemplar differs, the exemplar wins (and note the discrepancy in the status doc).

## Team standards file

<!-- Filled by Phase 0 standards ingestion. -->
- Path: `<<FILL:team-standards-file-path>>`
- Patterns overridden by it: _none recorded yet_

When the file is present, entries below that were rewritten to match it are tagged `(team standard)`.

## Structure

- **Start from an exemplar, not from scratch.** Copy the folder/file skeleton of the closest exemplar page (page component, child components, hooks, tests, stories) and rename. This inherits naming, layout, and wiring conventions for free.
- **One Angular component → one React component** by default. Do not merge or split components during a parity port; restructuring is post-parity work.

## Component API

| Angular | React |
|---|---|
| `@Input() foo` | prop `foo` |
| `@Input() foo` with setter logic | prop + `useEffect`/derived state, or compute-on-render if pure |
| `@Output() changed = new EventEmitter<T>()` | callback prop `onChanged: (value: T) => void` |
| `@ViewChild` DOM access | `useRef` |
| Content projection `<ng-content>` | `children` / named slots as props |

Confirm prop-naming style (e.g. `onX` callbacks) against an exemplar.

## Template constructs

| Angular | React |
|---|---|
| `*ngIf="cond"` / `*ngIf ... else` | `{cond && <X/>}` / ternary |
| `*ngFor="let item of items; trackBy"` | `items.map(item => <X key={...}/>)` — key mirrors the trackBy identity |
| `[class.active]="cond"` / `ngClass` | conditional className (use the app's classnames helper if exemplars do) |
| `[style.x]` / `ngStyle` | inline style object or styled API per exemplar |
| `ngSwitch` | ternary chain or a small lookup map |
| Pipes in templates (`| date`, `| currency`, custom) | plain functions called in JSX; reuse the app's formatting utils — check exemplars before writing a new formatter |

## Lifecycle

| Angular | React |
|---|---|
| `ngOnInit` (fetch/subscribe) | `useEffect(..., [])` or the app's data-fetching hook — whichever exemplars use |
| `ngOnChanges` | `useEffect` with the prop in deps, or derive during render if pure |
| `ngOnDestroy` (cleanup) | `useEffect` cleanup return |
| `ngAfterViewInit` | `useEffect` + `useRef` |

## Services, DI, and state

The app has no global state manager on either side — do not introduce one.

- **Page-local service state** → `useState`/`useReducer` in the page component, or a custom hook if the exemplars extract hooks.
- **Shared/injectable service** → check how exemplars consume the equivalent: usually a plain module function or an existing custom hook. Never re-create Angular DI with context providers unless an exemplar already does.
- **Service with HTTP calls** → the app's established data-fetching pattern (from the recon doc). Match loading/error handling exactly to the Angular behavior, not to what looks nicer.

## Observables (RxJS)

| Angular pattern | React port |
|---|---|
| `http.get(...).subscribe(...)` | one-shot fetch via the app's data-fetching pattern |
| `combineLatest` of inputs | derived values in render, or `useMemo` |
| Subject-based component communication | lift state up / callback props |
| Debounced `valueChanges` | debounced handler (use the app's existing debounce util) |
| Long-lived subscription to a shared stream | check exemplars; likely a hook wrapping subscribe/unsubscribe in `useEffect` |

Do not carry RxJS into the React page unless an exemplar already does for the same need.

## Forms

Port each form to whatever the exemplar pages use (recon doc records it — e.g. React Hook Form or controlled components). Parity requirements regardless of library:

- Same fields, same validators (including async), same validation *timing* (on blur vs on change vs on submit — verify on DEV1)
- Same error messages and same display behavior
- Same submit flow including disabled states and double-submit protection

## Design-system component swap

For each `library` component: take the React flavor from `<<FILL:react-library-package>>` per the component map row. Verify prop parity — same variant, size, disabled/loading states, event handlers. If the React flavor's API differs in a way the map row doesn't cover, update the map row with the adaptation note.

## Routing & interop

All navigation wiring follows `references/interop-routing.md`. Never use a React router API for navigation that crosses into Angular-owned routing.

## Phase 3 gate (restated)

- [ ] Page renders on the local React dev server
- [ ] TypeScript compiles; lint passes
- [ ] Every inventory row from Phase 2 is implemented (walk the tables)
- [ ] Interop navigation into and out of the page works manually
- [ ] No pattern used that contradicts the team standards file or exemplars without a noted reason
