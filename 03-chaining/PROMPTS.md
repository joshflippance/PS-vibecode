# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Completion and Success Celebration View Chain

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:

Add a completion screen (/complete): A terminal celebration view that appears after a participant confirms an action. Match the clean, low-density layout, generous whitespace, and hairline borders of the main action-first dashboard (src/routes/index.tsx). It should display a success confirmation message, a summary of actions completed during the session, and a primary CTA button to restart the test or view the final metric evaluation against the kill switch.

Add a summary audit modal or drawer (/history): Triggered from the completion screen, this view lists a chronological log of all metrics acted on during the session. Match the high-contrast metadata tags, tight typography (Inter Tight for UI text, IBM Plex Mono for numeric labels), and flat card surface tokens defined in src/styles.css.

Navigation: write the router logic in TanStack Router so that confirming a final action transitions the user smoothly to the completion screen (/complete), with a secondary link to toggle the audit history view.

Build these in order so the completion screen (/complete) serves as the anchor for the audit history log.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the completion and history flow (/complete and /history):

- Use skeleton screens for the history log loading state.
- If no data is present, show the empty state: "To prevent unnecessary data spreadsheets."
- On fetch failure, trigger the error state: "Failed to load session results. Reset session to try again.

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
The completion and history screens need a professional Vercel-style polish.

Start by listing the 3 biggest gaps in typography and spacing compared to the main action-first dashboard (src/routes/index.tsx), focusing on type scale, hairline border weights, and card padding.

Once you've identified those, resize the headers and update the primary action button (--cta surface tokens) to match.

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- _____
- _____

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

_____
