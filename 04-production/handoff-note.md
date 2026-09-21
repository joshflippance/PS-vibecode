# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

This is a clickable prototype testing one hypothesis: if every metric ships with a single recommended action and no filters or drill-downs, users will act on the dashboard instead of bouncing or exporting to a spreadsheet. The app is a TanStack Start (React 19) SPA with no backend — all data is hard-coded, all session state is in-memory React context, and the "API" calls are a simulated 600ms delay. There are six screens: the action-first landing view, an action detail/confirm view, a completion screen, an audit history drawer, a twelve-widget "old dashboard" contrast view, and an evidence page. The kill switch is the whole point: each metric card tracks Not clicked / Clicked / Acted status, and any insight left unclicked after 15 seconds is flagged for removal. Nothing is persisted, no real data is fetched, no real .xlsx is generated. Treat this as a throwaway test rig, not production code.

## Architecture (plain language)

- **Frontend:** Single TanStack Start app. File-based routes in src/routes/ are thin entry points — each imports one screen component from src/features/hypothesis-test/. Routes:  Route	Screen component /	action-first/ActionFirstLandingView /action/$id	action-detail/ActionDetailView (loader resolves insight, throws notFound) /complete	completion/CompletionScreen /history	history/HistoryDrawer (slide-over drawer) /old-dashboard	comparison/OldDashboardComparisonView /evidence	evidence/EvidenceView src/routes/__root.tsx wraps everything in a SessionProvider, renders the top header + nav and the sticky ResultsBar (kill-switch instrumentation), and holds the <Outlet />.
- **Backend / data:** There is no backend. Two static data modules hold all content:  src/features/hypothesis-test/data/insights.ts — four metrics (INSIGHTS), each with id, value, label, read, action, actionDetail, evidence[]; plus QUOTES (four user-voice quotes) and getInsight(id). src/features/hypothesis-test/data/old-dashboard.ts — OLD_DASHBOARD_WIDGETS (12 widgets derived from label/bar-pattern arrays) and OLD_DASHBOARD_FILTERS. Session state lives in src/features/hypothesis-test/session/SessionProvider.tsx — a React context holding elapsed time, interacted flag, bounced flag, export count, per-metric status map, and a chronological LogEntry[]. useSessionResults.ts wraps the log behind a 600ms loading phase and resolves to loading | ready | empty | error for the completion/history screens.
- **Key flows:** Act on an insight — Landing view → click action pill (markClicked) → detail view → "Confirm and apply" (markActed) → back to landing. When the last unacted insight is confirmed, the button reads "Confirm and finish" and navigates to /complete.
Bounce detection — A 1s interval in SessionProvider ticks elapsed time; if no interaction occurs within 15s, bounced flips true and idle cards show "No click = no value. Remove this insight."
Kill-switch evaluation — CompletionScreen renders a Keep/Cut verdict per metric based on whether statuses[id] === "acted". ResultsBar surfaces a live summary (session time, bounce state, actions clicked, exports, metrics to cut).
Audit history — /history opens as a right-side drawer over /complete, listing every clicked/acted/export event with a timestamp and kind tag. Backdrop click returns to /complete.
Reset — reset() on CompletionScreen clears all session state and returns to /.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Hypothesis-first design. | solid | The whole app is built around one testable claim, and the kill switch is observable on every screen. The logic that decides Keep/Cut is genuinely wired to real interaction state. |
| All data is hard-coded.v | rough | INSIGHTS, QUOTES, widget counts, and the 12/6/74%/60% baseline numbers are literals in TypeScript files. There is no data source, no provenance, no API. The metrics are illustrative, not measured. |

## Risks & assumptions for the team

Assumption: clicking an action = value. The kill switch treats a click as proof the insight earned its place. A user could click by accident or curiosity. There's no distinction between "read" and "acted."
Assumption: 15s = bounce. The threshold is a guess. Real dashboards have different dwell-time distributions. Validate before shipping.
Assumption: the four metrics are representative. They're chosen to dramatize the hypothesis, not sampled from production data.
Risk: in-memory state leaks across tabs. Each browser tab gets its own SessionProvider, so multi-tab testing isn't isolated by URL — it's isolated by React tree. Adequate for a prototype; not for a real experiment.
Risk: no telemetry. The app records nothing externally. To actually prove the hypothesis, you need server-side event capture (click, act, bounce, export, reset) with a session id and timestamp — none of which exists here.
Risk: a11y unverified. The gauge charts are decorative SVG with no text alternative beyond the visible value. The drawer lacks a focus-trap. Not production-ready for assistive tech.

## How to run it

```
# Install deps (bun is the package manager in this project)
bun install

# Dev server (already running on localhost:8080 in the Lovable preview)
bun run dev

# Typecheck
bun run typecheck

# Build
bun run build
```
