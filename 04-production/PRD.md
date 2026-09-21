# Product Requirements Document — The Dashboard Nobody Reads

## Problem

Users can see what happened in the current analytics dashboard but cannot quickly determine what to do next. The default experience presents 12 charts, places the most-requested metric six clicks and three filters away, and leaves users to interpret the data without guidance. The stated baseline is that 60% of sessions end within 15 seconds without interaction and 74% of users export data to a spreadsheet to perform the actual analysis.

The validated problem is that information density without decision guidance creates avoidable effort and abandonment. User feedback consistently describes the same failure: too many charts, excessive navigation, executive reporting needs that are not met, and no recommended next step. The prototype therefore tests the solution hypothesis that showing one recommended action per metric, immediately and without filters or extra navigation, will help action-oriented users engage with the product instead of bouncing or exporting raw data.

The hypothesis is considered supported when a participant does not bounce and clicks a recommended action. At the individual-insight level, an action that receives no click provides no demonstrated value and is a candidate for removal. The current prototype makes this criterion observable, but it does not constitute production validation: it stores no cross-session results, has no real analytics pipeline, and contains supplied demonstration metrics and quotes rather than live customer data.

## Users & jobs (primary user, job to be done)

**Primary user:** A marketing manager, growth analyst, product marketing manager, or product lead who opens an analytics product because they need to make or recommend a decision—not merely inspect charts.

**Job to be done:** When I review performance data, help me understand the important signal and the single next action immediately, so I can make progress without navigating through filters, interpreting a dense chart grid, or exporting raw data to a spreadsheet.

Supporting needs:

- Find the most decision-relevant metrics on the first screen.
- Understand why a recommended action is justified.
- Confirm an action and see that it was recorded.
- Review which insights earned engagement and which failed the kill-switch test.
- Compare the action-first experience with the existing high-density dashboard pattern.

## Scope (in / out)

### In

- An action-first landing view with four supplied baseline metrics.
- One plain-language interpretation and one recommended action per metric.
- Per-metric states: **Not clicked**, **Clicked**, and **Acted**.
- A focused action detail view with supporting evidence and confirmation.
- A 15-second no-interaction bounce rule.
- A visible kill-switch summary showing elapsed time, engagement state, action clicks, spreadsheet exports, and insights to cut.
- A completion screen summarising acted-on insights and the final keep/cut evaluation.
- A history drawer showing a chronological session event log.
- Loading skeleton, empty, and error states for completion/history results.
- An old-dashboard comparison view with 12 chart widgets, filters, and a tracked export action.
- An evidence view containing the four supplied quotes and baseline metrics.
- A reset control for running another prototype session.

### Out

- Real customer, product, or analytics data integrations.
- Persistent storage, participant identity, authentication, or cross-session reporting.
- A real `.xlsx` file download or spreadsheet export pipeline.
- Execution of the recommended business actions in another system.
- Real filtering, date selection, chart exploration, or metric drill-down on the old-dashboard view.
- Experiment assignment, control-group management, statistical significance calculations, or automated hypothesis validation.
- Administration tools for editing metrics, evidence, recommendations, or kill-switch thresholds.
- Production monitoring, privacy controls, retention policies, or data governance.

## Requirements (table with requirement, priority, acceptance criteria)

| Requirement | Priority | Acceptance criteria |
| --- | --- | --- |
| Present four decision metrics on the landing view | Must | The first screen shows 12 charts, 6 clicks, 74% exports, and 60% bounce as four distinct metric cards. |
| Provide one action per metric | Must | Every metric card shows one and only one primary recommended-action control without requiring filters, tabs, or drill-down to discover it. |
| Explain each metric in plain language | Must | Every card includes a concise interpretation of what the number means for the user. |
| Record an action click | Must | Selecting a recommended action changes that metric from **Not clicked** to **Clicked** and adds a timestamped click event to the current in-memory session. |
| Show action evidence and confirmation | Must | The action detail view displays the metric, proposed action, supporting evidence, and controls to confirm or defer. |
| Record a confirmed action | Must | Confirming changes the metric to **Acted** and adds a timestamped acted event. If other metrics remain, the participant returns to the action view. |
| Complete the session after the final confirmation | Must | Confirming the last outstanding action navigates to `/complete`, where the success message, session totals, acted actions, and keep/cut evaluation are visible. |
| Apply the bounce rule | Must | A session with no interaction at 15 seconds is displayed as bounced. Any interaction before that point changes the session to engaged. |
| Make the insight kill switch observable | Must | Each metric displays its current state. After 15 seconds, an unclicked metric displays “No click = no value. Remove this insight.” The summary identifies unclicked metrics as candidates to cut. |
| Track the export escape hatch | Must | Selecting “Download raw data (.xlsx)” increments the session export count and adds an export event. No actual file needs to be generated in this prototype. |
| Display live session results | Must | A persistent summary shows elapsed time, bounce/engagement state, number of action metrics clicked out of four, export count, and the current cut list. |
| Show chronological audit history | Must | `/history` lists all click, acted, and export events in session order with event type and seconds elapsed. It can be opened from the completion screen and closed back to it. |
| Handle session-results loading | Must | Completion and history views show skeleton rows for the explicit loading period. |
| Handle no session data | Must | When the event log is empty, completion/history displays exactly: “To prevent unnecessary data spreadsheets.” |
| Handle session-results failure | Must | If session results cannot be read, the view displays exactly: “Failed to load session results. Reset session to try again.” and provides a reset control. |
| Support repeat testing | Must | Reset clears elapsed time, engagement, exports, metric statuses, and the event log so a fresh session can begin. |
| Preserve supplied research context | Should | The four supplied user quotes and roles appear on screen, and the evidence view pairs them with the four baseline metrics. |
| Provide a comparison experience | Should | `/old-dashboard` visibly contains 12 dense widgets, apparent filter controls, no recommended actions, and the tracked export escape hatch. |
| Maintain the established visual language | Should | Screens use the shared neutral canvas, flat white surfaces, hairline borders, compact UI typography, mono numeric labels, and the semantic CTA surface token. |

## Data & events

### Prototype data

The following content is hard-coded demonstration data in the application and is not fetched from a live analytics source:

- Four baseline metrics: 12 charts, 6 clicks, 74% exports, and 60% bounce.
- Four user quotes and attributed roles.
- Recommended actions, plain-language interpretations, action details, and supporting evidence.
- Twelve old-dashboard widget names, displayed values, percentage changes, and chart bars.

These inputs were supplied to ground the prototype. The application does not independently verify their accuracy or provenance.

### Session data

The prototype records the following only in browser memory for the current page session:

- Elapsed seconds.
- Whether any interaction occurred.
- Derived bounce state after 15 seconds without interaction.
- Export count.
- Per-metric status: `idle`, `clicked`, or `acted`.
- Chronological event log with event identifier, metric identifier, event kind, label, wall-clock timestamp, and elapsed seconds.

Refreshing the application clears this data. There is no database, durable event store, participant identifier, or aggregation across sessions.

### Events

| Event | Trigger | Recorded properties | Current implementation |
| --- | --- | --- | --- |
| `clicked` | Participant selects a recommended action | Metric ID, action label, timestamp, elapsed seconds | Real within the current in-memory prototype session; not persisted or sent to analytics. |
| `acted` | Participant confirms an action | Metric ID, action label, timestamp, elapsed seconds | Real within the current in-memory prototype session; does not execute the business action. |
| `export` | Participant selects the old dashboard’s `.xlsx` control | Export label, timestamp, elapsed seconds | Counter and event are real in memory; file generation and download are mocked. |
| `bounced` | No interaction occurs by 15 elapsed seconds | Derived from elapsed time and interaction state | Computed live in the browser; no event is persisted. |
| `reset` | Participant selects reset | Clears all current session state | Functional in memory; reset itself is not added to the audit log. |

The completion/history “fetch” is simulated with a 600 ms delay over the in-memory event log. Skeleton and empty states are functional. The error state exists, but under normal prototype state it is not expected because the session log is always an array; there is no network request or real remote failure path.

### Measurement needed for production validation

A production experiment should persist at least: anonymous participant/session ID, experiment variant, metric/action ID, impression time, action click, action confirmation, export request, session end, and bounce outcome. Analysis should compare action-first and control experiences on action-click rate, bounce rate, and export rate, with a pre-agreed sample size and decision threshold. None of this production measurement is implemented in the prototype.

## Open questions

1. What prior study or dataset establishes that the solution hypothesis—not only the underlying problem—is validated?
2. What minimum action-click uplift, bounce reduction, and export reduction will count as success?
3. What sample size and test duration are required before applying the kill switch to an insight?
4. Should a click alone earn an insight a **Keep** decision, or must the participant also confirm the action?
5. Does any interaction prevent a bounce, or should only a recommended-action click count?
6. Should the 15-second bounce threshold vary by user role, device, or dashboard complexity?
7. Are the four metrics and quotes representative of production users, and who owns verification of their source and recency?
8. How will recommended actions be generated, reviewed, and kept current in production?
9. What does “confirm and apply” mean operationally—record intent only, create a task, change the dashboard, or trigger another workflow?
10. Should exports be discouraged universally, or are some export workflows legitimate and necessary?
11. What participant/session identifiers may be collected, and what consent, retention, and privacy requirements apply?
12. Should history include only acted-on metrics, as the product request states, or all clicked, acted, and export events, as the current prototype does?
13. How should results survive refreshes and be aggregated across participants once the prototype moves beyond moderated testing?
14. What real failure conditions should trigger the session-results error state?
