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

- 1) The Power of Intentional Sequencing: Breaking a complex build into distinct phases (structure $\rightarrow$ logic/error-handling $\rightarrow$ visual polish) prevents the AI from getting overwhelmed, resulting in much cleaner, more maintainable code than asking for everything at once
- 2) Context Preservation Across Steps: You learn how early constraints and design tokens (like your Linear/Vercel aesthetic, font pairings, and semantic CSS classes) cascade and must be explicitly reinforced so the AI doesn't drift into generic defaults.
- 3) Edge-Case Resilience: Enforcing explicit rules for loading skeletons, empty states, and error triggers forces you to think about software states that are commonly glossed over in rapid prototyping.
- 4) Component-Driven Thinking: Separating structural layout from micro-refinements (like typography scales and button states) teaches you how to systematically audit UI gaps the way a senior frontend engineer would.

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

1. What Broke: "The Monolithic Prompt Trap" (Structural Collapse)
What broke: When you ask an AI to build a brand new feature (like a new screen + state management + styling) all in one massive prompt, it usually fails. It jumbles the component tree, invents random CSS styles instead of using your design tokens, or forgets to hook up the router.

The Fix (Step 1): Strict sequencing. Forcing the AI to build Screen A first as an "anchor," then Screen B, and finally writing the navigation logic explicitly prevents the components from being built out of order or disconnected.

2. What Broke: "The Happy-Path Illusion" (Unhandled States)
What broke: Developers and AI models naturally code for the "happy path" (when data loads instantly and perfectly). In production or user testing, this leads to jarring layout shifts, infinite blank screens when an API/fetch fails, or confusing blank spaces when there's no data.

The Fix (Step 2): Explicit logic constraints. Forcing the AI to handle loading skeletons, a specific empty-state phrase ("To prevent unnecessary data spreadsheets"), and a clear error message ensures the app behaves robustly under real-world conditions.

3. What Broke: "Design Drift" (Inconsistent UI Polish)
What broke: As new screens are added, AI models tend to revert to generic, default styles (e.g., standard blue buttons, harsh borders, or default system fonts), breaking the cohesive Linear/Vercel aesthetic established in your main dashboard.

The Fix (Step 3): A structured visual audit. By making the AI explicitly list the 3 biggest typography and spacing gaps before changing anything, and tying adjustments strictly to existing design tokens (like --cta), it preserves the look and feel of the rest of the application without messing up the underlying React logic.
