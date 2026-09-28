# FULLSTACK.md · Lab 1 Sections

Project: **The Dashboard Nobody Reads** (hypothesis prototype)
Stack: TanStack Start (React 19) front end and server functions · Lovable Cloud database with row-level security · email/password auth
Updated: 28 Sep 2026 (v2: adds request timeouts, inline Retry states, sign-up double-submit fix)

---

## 1. Data schema

| Entity | Key fields | Notes |
|---|---|---|
| **insights** | `id` (text PK, e.g. `charts`), `value`, `label`, `read`, `action`, `action_detail`, `evidence text[]`, `sort_order`, `created_at` | The 4 baseline metrics, each with one recommended action. Seeded. Public read-only. The app falls back to a bundled copy if the database is unreachable. |
| **quotes** | `id` (uuid PK), `quote`, `role`, `sort_order`, `created_at` | The 4 verbatim user quotes. Seeded. Public read-only. Same fallback as insights. |
| **test_sessions** | `id` (uuid PK), `session_token` (uuid, unique), `started_at`, `ended_at`, `elapsed_seconds`, `interacted`, `bounced`, `export_count`, `created_at` | One row per anonymous participant visit. The summary is re-synced every 5 s and on each event. Not linked to user accounts. |
| **session_events** | `id` (uuid PK), `session_id` → test_sessions (cascade), `insight_id` (nullable), `kind` enum `clicked \| acted \| export`, `label`, `at_seconds`, `occurred_at` | Chronological kill-switch log. Written immediately on each click, confirm, or export. |
| **invites** | `id` (uuid PK), `user_id` (owner), `sender_email`, `recipient_email`, `status` enum `pending \| accepted \| declined \| expired`, `sent_at`, `updated_at` (trigger), `is_demo` | Owner-scoped. `updated_at` is stamped by a `BEFORE UPDATE` trigger. Indexed on `(user_id, sent_at desc)` for pagination. The 50 demo rows have no owner, so no signed-in user can see them. |
| **auth.users** (managed) | `id`, `email`, `encrypted_password`, `email_confirmed_at` | Managed by the auth service. No custom profile table because no profile data is stored. |

**Enums:** `session_event_kind` (clicked, acted, export) · `invite_status` (pending, accepted, declined, expired)

**Functions and triggers:**
- `touch_updated_at()`: trigger on `invites` that sets `updated_at = now()`.
- `invite_stats()`: security-definer aggregate. Legacy and unused: execute is revoked from public roles, and it's kept only for server use.

**Relationships:** `session_events.session_id → test_sessions.id` (ON DELETE CASCADE). `invites.user_id` holds the auth user id. It has no foreign key to the managed auth schema, by design.

**Mocked vs real:**

| Real (persisted) | Mocked / demo |
|---|---|
| Sessions, events, invites, auth accounts | Insight numbers and evidence (demonstration copy, provenance unchecked) |
| | The 12 old-dashboard widgets (hard-coded in code) |
| | `.xlsx` export (only increments a counter) |
| | "Send invite" (records a row; no email is sent) |

---

## 2. Access rules

**Who can see and do what**

| Resource | Signed-out visitor | Signed-in user | App server (service role) |
|---|---|---|---|
| insights, quotes | Read | Read | Full |
| test_sessions | Insert. Read and update only the row matching the `x-session-token` header | Same | Full |
| session_events | Insert and read only for their own session (token match) | Same | Full |
| invites | None | Read, create, update and delete **only where `user_id = auth.uid()`** | Full |
| invite_stats() | No execute | No execute | Execute |

**Auth boundaries**

1. **Browser → database:** the publishable key only. Row-level security is enabled on every public table, and every table has explicit grants.
2. **Browser → server functions:** a bearer token is attached automatically. Invite functions (`getMyInviteStats`, `listMyInvites`, `createInvite`, `updateInviteStatus`) require a valid session and run *as the user*, so RLS applies. The server also filters on `user_id = context.userId`.
3. **Ownership comes from the session, never from client input.** `user_id` and `sender_email` are taken from the verified token's claims.
4. **Participant session writes** (`startSession`, `recordEvent`, `saveSessionSummary`) use the service-role client and are **not ownership-checked**. See Known gaps.
5. **Password reset:** the recovery link opens `/reset-password`, which works only with a valid recovery session. After the update the user is signed out and must sign in again.
6. **Leaked-password protection:** passwords found in known breaches are rejected on sign-up and on password change.
7. **Email confirmation is required** before first sign-in (auto-confirm off). Anonymous sign-ups are disabled.

**Known gaps (open)**
- Session writes bypass RLS: anyone who knows a `sessionId` can append events or overwrite its summary. Fix: issue `session_token` to the browser and send it as `x-session-token` from a publishable-key client.
- Researchers have no role model yet (no `user_roles` table), so there's no cross-session results view.

---

## 3. Edge cases hardened (Before / After)

### 3a. Empty / first-run

| Case | Before | After |
|---|---|---|
| Signed-out visitor, invite section | Showed global demo totals to everyone | "See how your invites are doing" with a **Get Started** button that goes to `/auth` |
| Signed-in user with zero invites | Blank cards, or a "0" on every card | "No invites yet" with a **Get Started** button that opens the invite composer |
| History log with no events | Blank list | "To prevent unnecessary data spreadsheets." (exact copy) |
| Insights/quotes tables empty | Screen would render nothing | Falls back to the bundled copy |
| Page emptied after an edit elsewhere | Blank page N | Automatically steps back to the last real page |
| New account | No data visible | Clean empty state (demo rows are ownerless on purpose) |
| Sign-up / sign-in form, empty fields | Form allowed empty submission attempts or lacked clear initial state handling | Required field validation: `required` on email and password, `type=email`, `minLength=6` on the password, so the browser blocks submission until fields are valid |

### 3b. Bad / malicious input

| Case | Before | After |
|---|---|---|
| Invalid recipient email | Any string accepted | Zod check: trimmed, lowercased, valid email, max 255 characters, plus native `type=email` in the browser |
| Forged `user_id` / `sender_email` in a request | Client could set them | Ignored: both come from verified token claims |
| Editing someone else's invite by id | Possible under the earlier aggregate model | RLS plus `eq(user_id)`. Returns "Invite not found." with no data leak |
| Invalid status value | Free text | `z.enum(pending, accepted, declined, expired)` plus a database enum |
| Page number abuse (`page=-1`, huge) | Unbounded | Must be an integer from 1 to 10,000 |
| Unknown `/action/:id` | Crash risk | `notFound()`, with no event written |
| Account probing via "Forgot password" | Would reveal whether an email exists | Same message either way: "If an account exists…" |
| Weak or breached password | Allowed | Minimum 8 characters on reset, and breached passwords are rejected |
| Reading invite emails anonymously | Public aggregate function exposed | Execute revoked; no public access to the table |
| Rapid spam-clicking "Sign up" / "Sign in" | Multiple overlapping requests fired, ending in a "Failed to fetch" error | Synchronous in-flight guard (`useRef`) ignores every click after the first; the button is disabled with a spinner and "Creating account…" / "Signing in…", and re-enables only when the request resolves or fails |
| Event payload tampering | — | Zod-validated `kind`, `atSeconds ≥ 0`, uuid `sessionId` (**ownership still unchecked**, see gaps) |

### 3c. Failure / offline

| Case | Before | After |
|---|---|---|
| Invite stats fail to load | Silent blank | Inline error: "Couldn't load your invite numbers." with **Retry** |
| Invite list fails to load | Silent blank | Inline error: "Couldn't load your invites." with **Retry** |
| Status change fails to save | Value silently reverts | The row shows "Not saved." with **Retry** (replays the same value) |
| Send invite fails | No feedback | Inline message under the composer |
| Slow network | Layout jump | Skeleton cards and rows. The previous page stays visible, dimmed, while the next page loads |
| Two edits to the same invite at once | Undefined | **Last write wins**: no version check, the trigger stamps `updated_at`, and the list re-reads after every save so it shows what was actually stored |
| Database unreachable for content | Screen error | Bundled insights and quotes fallback |
| History load failure | Blank | "Failed to load session results. Reset session to try again." with **Reset session** (the load is simulated, so this has no real trigger yet) |
| Expired or reused reset link | Would silently fail | "Link expired" screen with a way back to sign in |
| Password-reset rate limit (429) | Raw error | "Too many attempts. Please wait a few minutes and try again." |
| Sign-out with requests in flight | 401 storm, stale cache | Cancel queries, clear the cache, sign out, then navigate with history replace |
| Stalled request (no response) | Skeletons pulsed indefinitely; UI could appear frozen | Every invite read and write is wrapped in a 10 s timeout (`withTimeout`) that rejects into the inline error state |
| Failed query retry storm | Default 3 retries with backoff (~7 s of skeletons) | One retry after 800 ms, then the inline error with **Retry**; mutations never auto-retry |
| Action page fails to load (`/action/:id`) | Route could hang or crash | Route error view: "Couldn't load this action. Check your connection and try again." with **Retry** (re-runs the loader) |
| Auth check fails while offline | Invite section could stay on skeletons forever | Network failure is treated as signed out; the section always leaves the loading state |
| Sign-up / sign-in request throws (network drop) | Raw "Failed to fetch" or unhandled rejection | Caught; shows "Couldn't reach the server. Check your connection and try again." and the button re-enables |
| Participant goes offline mid-session | Events lost (fire-and-forget) | **Not yet handled.** See backlog |

---

## 4. Verification log (28 Sep 2026)

| Check | Method | Result |
|---|---|---|
| Signed-out invite empty state and Get Started | Playwright | Pass. Goes to `/auth`, no console errors |
| Signed-in empty state | Playwright (real account) | Pass: "No invites yet" and Get Started |
| Create 11 invites | Playwright | Pass |
| Pagination | Playwright | Pass: "1–10 of 11", page 2 shows 1 row |
| Status edit persists and stats update | Playwright | Pass: accepted, "11 sent · 1 accepted" |
| Test data cleanup | SQL delete | Done |
| Inline error and Retry | — | **Not exercised** (no failure injected) |
| Simultaneous edits | — | **Not exercised** |
| Password reset email delivery | — | **Not exercised** |
| Action page offline, then Retry | Playwright (blocked network) | Pass: error view with Retry shown; Retry loaded the page once reconnected |
| Invite section timeouts / auth-check offline | — | **Not exercised** in a browser |
| Sign-up double-submit guard | Code review | Implemented; **not yet exercised** in a browser |
| Typecheck | `tsgo --noEmit` | Clean |

---

## 5. Stress test results

_What we threw at it, and what held or broke._

| # | Test performed | Result | Engineering status |
|---|---|---|---|
| 1 | Rapid-fire spam clicking of the "Sign up" button to simulate high-frequency input | **Broke (before fix):** the UI processed overlapping requests at once, producing unhandled state and a network failure ("Failed to fetch") | Originally documented as a known gap needing submission throttling / disable-on-click. **Now fixed** in code (in-flight guard, disabled button, loading state, friendly error). Re-run in a browser to confirm |
| 2 | Blocked network, then opened an action page | **Held:** inline error with Retry; recovered after reconnecting | Done |
| 3 | 11 invites created, paged and edited | **Held:** pagination and status edits correct | Done (test rows deleted) |

## 6. Stress tests to run next

1. **Ownership:** user A calls `updateInviteStatus` with user B's invite id. Expect "Invite not found."
2. **No token:** call the invite functions without a bearer token. Expect 401.
3. **Session forgery:** post `recordEvent` with another session's id. It currently **succeeds**, which proves the gap. It should fail once fixed.
4. **Last-write-wins race:** fire 20 concurrent status updates on one invite. The final row equals the last committed write, and `updated_at` equals the latest time.
5. **Pagination under churn:** delete rows while on the last page. The UI steps back without a blank page.
6. **Failure injection:** block the database host. Expect the Retry states to appear and recover once the host is unblocked.
7. **Load:** 200 concurrent participants with 10 events each. Expect p95 write under 300 ms and no lost events.
8. **Sign-up spam (re-test):** click Sign up 20 times in 1 s. Expect exactly one request and no "Failed to fetch".
9. **Auth abuse:** reset-password spam triggers the rate-limit message, and a breached password is rejected.

---

## 7. Backlog (next hardening)

- Issue `session_token` to the browser and move session writes to RLS (closes the forgery gap).
- Queue events offline and flush via `sendBeacon` when the tab closes.
- Make click/act events idempotent with a unique `(session_id, insight_id, kind)` constraint.
- Add a `user_roles` table and `has_role()`, plus a researcher-only results page (Keep/Cut per insight across sessions).
- Move the bounce threshold and keep-rate into an `experiment_config` table.
- Replace the simulated history load with a real query of `session_events`.
- Browser-verify the sign-up double-submit fix and invite-section timeout states.
- Optional: branded auth emails (needs a custom domain) and real invite email delivery.


---

## Appendix: Edge cases hardened (lab summary)

| Case | Before | After |
|---|---|---|
| Empty / first-run state | Form allowed empty submission attempts or lacked clear initial state handling. | Implemented required field validation or disabled submission until mandatory fields contain valid inputs |
| Bad / malicious input | Rapid spam-clicking the submit button triggered multiple duplicate requests or network errors (e.g., "Failed to fetch"). | Added submission debouncing, disabled the button immediately upon first click, or implemented loading state protection to prevent duplicate rapid-fire payloads. |
| Failure / offline | Network or server timeouts during repeated requests caused unhandled promise rejections or generic connection failures. | Added explicit error boundary handling and user-facing fallback messaging (such as clean error states instead of raw crashes). |

## Appendix: Stress test results (lab summary)

_What you threw at it, and what held / broke._

- **Test performed:** Rapid-fire spam testing by clicking the "Sign up" button multiple times in succession to simulate high-frequency user input.
- **Result:** The UI attempted to process overlapping requests simultaneously, leading to unhandled state payloads and a network failure state ("Failed to fetch").
- **Engineering status:** Documented as a known gap requiring submission throttling/button-disable states on click to prevent duplicate execution payloads. *Update: the fix has since been implemented (see section 3b); browser re-test pending.*
