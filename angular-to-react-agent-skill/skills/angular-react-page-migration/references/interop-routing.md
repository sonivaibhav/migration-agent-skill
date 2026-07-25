# Interop Routing — Discovery & Wiring (Phases 3–4)

Angular owns routing. React pages are reached and leave via an established window-communication event mechanism (referred to here as `<<FILL:interop-mechanism-name>>`), proven on 5+ migrated pages.

**Prohibition (restated from Ground Rules): this mechanism is consumed, never changed.** At handoff, `git diff --stat -- <<FILL:interop-file-paths>>` must show zero changes. If a page seems to need a mechanism change, that is a human-judgment checkpoint, not a code change.

## Finding the contract (resolved in Phase 0)

Fill these during calibration by reading the mechanism's source and one exemplar page:

- Mechanism entry points: `<<FILL:interop-file-paths>>`
- Event names / message types: `<<FILL:interop-event-names>>` (grep anchors: the event-name constants, `window.addEventListener` / `dispatchEvent` / `postMessage` call sites)
- How a React page/route is **registered** so the Angular host knows it exists: `<<FILL:interop-registration-pattern>>`
- Payload shape for navigation events (route, params, query): `<<FILL:interop-payload-shape>>`

The authoritative reference is always how the exemplar pages do it — read the wiring of at least one exemplar before wiring a new page.

## Wiring checklist for a new page

1. **Register the route/page** exactly the way the most recent exemplar does (same file, same pattern, alphabetical/positional placement matching convention).
2. **Inbound navigation**: the page mounts when Angular routes to it. Verify route params and query params arrive through the interop payload and are read the way exemplars read them — not via a React router API.
3. **Outbound navigation**: any link/action that leaves the page emits the interop navigation event with the correct payload. No direct `window.location` mutation, no React-router navigation for Angular-owned destinations, unless an exemplar does exactly that.
4. **In-page URL state** (tabs, filters reflected in query params): copy the exemplar approach; confirm the Angular host preserves/reflects it.
5. **Bidirectional check**: manually drive Angular → React page → Angular and back. Browser back/forward must behave the same as on the Angular version of the page (verify against DEV1 behavior).

## Troubleshooting

| Symptom | Likely cause | Check |
|---|---|---|
| Page never mounts on navigation | Registration missing or name mismatch | Diff your registration against the exemplar's, character by character |
| Navigation event not received | Listener attached after the event fired (mount timing) | Where do exemplars attach listeners? Match their timing exactly |
| Params undefined on mount | Reading params from the wrong source (router API vs interop payload) | Trace an exemplar's param flow |
| Back button breaks / double entries | Page emits navigation instead of letting Angular own history, or vice versa | Compare outbound wiring with the exemplar; verify against DEV1 |
| Works locally, breaks on DEV3 | Environment base-path or registration differences | Compare local vs DEV3 config for the exemplar pages |

If a symptom survives exemplar comparison, raise it as a human-judgment checkpoint rather than patching the mechanism.

## Gate contribution

- [ ] Registration identical in pattern to the latest exemplar
- [ ] Inbound + outbound + back/forward verified manually on the local server (Phase 3) and on DEV3 (Phase 4)
- [ ] Zero diff under `<<FILL:interop-file-paths>>`
