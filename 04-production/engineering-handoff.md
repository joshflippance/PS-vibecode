# Engineering Handoff — The Dashboard Nobody Reads

## 60-second read

This is a clickable prototype testing one hypothesis: if every metric ships with a single recommended action and no filters or drill-downs, users will act on the dashboard instead of bouncing or exporting to a spreadsheet. The app is a TanStack Start (React 19) SPA with no backend — all data is hard-coded, all session state is in-memory React context, and the "API" calls are a simulated 600ms delay. There are six screens: the action-first landing view, an action detail/confirm view, a completion screen, an audit history drawer, a twelve-widget "old dashboard" contrast view, and an evidence page. The kill switch is the whole point: each metric card tracks Not clicked / Clicked / Acted status, and any insight left unclicked after 15 seconds is flagged for removal. Nothing is persisted, no real data is fetched, no real .xlsx is generated. Treat this as a throwaway test rig, not production code.

## Architecture in plain language

### Frontend

Single TanStack Start app. File-based routes in `src/routes/` are thin entry points — each imports one screen component from `src/features/hypothesis-test/`. Routes:

| Route | Screen component |
|---|---|
| `/` | `action-first/ActionFirstLandingView` |
| `/action/$id` | `action-detail/ActionDetailView` (loader resolves insight, throws `notFound`) |
| `/complete` | `completion/CompletionScreen` |
| `/history` | `history/HistoryDrawer` (slide-over drawer) |
| `/old-dashboard` | `comparison/OldDashboardComparisonView` |
| `/evidence` | `evidence/EvidenceView` |

`src/routes/__root.tsx` wraps everything in a `SessionProvider`, renders the top header + nav and the sticky `ResultsBar` (kill-switch instrumentation), and holds the `<Outlet />`.

### Backend / data

There is no backend. Two static data modules hold all content:

- `src/features/hypothesis-test/data/insights.ts` — four metrics (`INSIGHTS`), each with `id`, `value`, `label`, `read`, `action`, `actionDetail`, `evidence[]`; plus `QUOTES` (four user-voice quotes) and `getInsight(id)`.
- `src/features/hypothesis-test/data/old-dashboard.ts` — `OLD_DASHBOARD_WIDGETS` (12 widgets derived from label/bar-pattern arrays) and `OLD_DASHBOARD_FILTERS`.

Session state lives in `src/features/hypothesis-test/session/SessionProvider.tsx` — a React context holding elapsed time, interacted flag, bounced flag, export count, per-metric status map, and a chronological `LogEntry[]`. `useSessionResults.ts` wraps the log behind a 600ms loading phase and resolves to `loading | ready | empty | error` for the completion/history screens.

### Key flows

1. **Act on an insight** — Landing view → click action pill (`markClicked`) → detail view → "Confirm and apply" (`markActed`) → back to landing. When the last unacted insight is confirmed, the button reads "Confirm and finish" and navigates to `/complete`.
2. **Bounce detection** — A 1s interval in `SessionProvider` ticks elapsed time; if no interaction occurs within 15s, `bounced` flips true and idle cards show "No click = no value. Remove this insight."
3. **Kill-switch evaluation** — `CompletionScreen` renders a Keep/Cut verdict per metric based on whether `statuses[id] === "acted"`. `ResultsBar` surfaces a live summary (session time, bounce state, actions clicked, exports, metrics to cut).
4. **Audit history** — `/history` opens as a right-side drawer over `/complete`, listing every clicked/acted/export event with a timestamp and kind tag. Backdrop click returns to `/complete`.
5. **Reset** — `reset()` on `CompletionScreen` clears all session state and returns to `/`.

## What's solid

- **Hypothesis-first design.** The whole app is built around one testable claim, and the kill switch is observable on every screen. The logic that decides Keep/Cut is genuinely wired to real interaction state.
- **Clean separation.** Data (`data/`), state (`session/`), shared UI (`components/`), and screens (`action-first/`, `action-detail/`, etc.) are grouped by feature. Routes are thin wrappers.
- **State handling.** `SessionProvider` is a focused, well-typed context — elapsed timer, interaction flag, status map, chronological log, reset. No prop drilling, no global store bloat.
- **Loading/empty/error states.** `useSessionResults` explicitly models all three states with mandated copy (`"To prevent unnecessary data spreadsheets."`, `"Failed to load session results. Reset session to try again."`). Both completion and history screens honor it.
- **Route hygiene.** Every route has unique `head()` metadata with `og:type` / `twitter:card`. The `action.$id` loader validates the id and throws `notFound`.

## What's duct tape

- **All data is hard-coded.** `INSIGHTS`, `QUOTES`, widget counts, and the 12/6/74%/60% baseline numbers are literals in TypeScript files. There is no data source, no provenance, no API. The metrics are illustrative, not measured.
- **The "fetch" is fake.** `useSessionResults` reads the in-memory log behind a `setTimeout(600)`. The error state has no real failure path — it can never actually trigger in normal use. The skeleton and error states exist to demonstrate UX, not to handle real async.
- **No persistence.** A page refresh wipes the session. There is no database, no localStorage, no Supabase. If you need a real experiment, this is the first thing to replace.
- **No real export.** The "Export to .xlsx" button on the old-dashboard view increments a counter and logs an event; it does not produce a file.
- **The runtime error in the preview log is stale.** A previous build referenced `src/lib/session.tsx` (now deleted); the current build imports from `src/features/hypothesis-test/session/SessionProvider`. All six routes return 200. Clearing the preview cache will clear the error.
- **Bounce rule is simplistic.** 15 seconds, no interaction, hard-coded. No idle detection, no scroll depth, no partial-attention heuristics. Fine for a prototype, not for a real analytics product.

## Risks and assumptions

- **Assumption: clicking an action = value.** The kill switch treats a click as proof the insight earned its place. A user could click by accident or curiosity. There's no distinction between "read" and "acted."
- **Assumption: 15s = bounce.** The threshold is a guess. Real dashboards have different dwell-time distributions. Validate before shipping.
- **Assumption: the four metrics are representative.** They're chosen to dramatize the hypothesis, not sampled from production data.
- **Risk: in-memory state leaks across tabs.** Each browser tab gets its own `SessionProvider`, so multi-tab testing isn't isolated by URL — it's isolated by React tree. Adequate for a prototype; not for a real experiment.
- **Risk: no telemetry.** The app records nothing externally. To actually prove the hypothesis, you need server-side event capture (click, act, bounce, export, reset) with a session id and timestamp — none of which exists here.
- **Risk: a11y unverified.** The gauge charts are decorative SVG with no text alternative beyond the visible value. The drawer lacks a focus-trap. Not production-ready for assistive tech.

## How to run it

```bash
# Install deps (bun is the package manager in this project)
bun install

# Dev server (already running on localhost:8080 in the Lovable preview)
bun run dev

# Typecheck
bun run typecheck

# Build
bun run build
```

### Verifying the full flow by hand

1. Open `/` — four metric cards, each with a status pill ("Not clicked") and an amber action button.
2. Wait 15 seconds without clicking — cards flip to "No click = no value. Remove this insight." and `ResultsBar` shows "bounce recorded".
3. Click any action → detail view → "Confirm and apply" → returns to landing; card status becomes "Acted".
4. Repeat until the last unacted insight → button reads "Confirm and finish" → navigates to `/complete`.
5. On `/complete`: session summary cells, actions-completed list, per-metric Keep/Cut evaluation.
6. Click "View audit history" → `/history` drawer with timestamped log. Close returns to `/complete`.
7. Click "Restart the test" → session resets → back to `/`.
8. Visit `/old-dashboard` → 12-widget contrast view → click "Export to .xlsx" → `ResultsBar` export count increments.
9. Visit `/evidence` → four user-voice quotes and the baseline metrics (12 charts, 6 clicks, 74% export, 60% bounce).

### File map

```
src/
├── routes/                      # thin route entry points (one per screen)
│   ├── __root.tsx               # SessionProvider, header/nav, ResultsBar, <Outlet/>
│   ├── index.tsx                # → ActionFirstLandingView
│   ├── action.$id.tsx           # loader → ActionDetailView
│   ├── complete.tsx             # → CompletionScreen
│   ├── history.tsx              # → HistoryDrawer
│   ├── old-dashboard.tsx        # → OldDashboardComparisonView
│   └── evidence.tsx             # → EvidenceView
└── features/hypothesis-test/
    ├── data/
    │   ├── insights.ts           # INSIGHTS, QUOTES, getInsight()
    │   └── old-dashboard.ts      # OLD_DASHBOARD_WIDGETS, OLD_DASHBOARD_FILTERS
    ├── session/
    │   ├── SessionProvider.tsx   # context: elapsed, interacted, bounced, statuses, log, reset
    │   └── useSessionResults.ts  # loading|ready|empty|error over the log (600ms)
    ├── components/
    │   ├── ResultsBar.tsx        # sticky kill-switch summary bar
    │   └── ResultsStates.tsx      # skeleton / empty / error UI
    ├── action-first/ActionFirstLandingView.tsx
    ├── action-detail/ActionDetailView.tsx
    ├── completion/CompletionScreen.tsx
    ├── history/HistoryDrawer.tsx
    ├── comparison/OldDashboardComparisonView.tsx
    └── evidence/EvidenceView.tsx
```
