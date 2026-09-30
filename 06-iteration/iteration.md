# Iteration: Analytics, Sprint, Final Recommendation

> Module 6 · Evals & Iteration. Read the analytics, run an iteration sprint, present with evidence.

## What the evidence says

_What real usage showed: numbers if your tool has analytics, counted behaviour if it does not. Put the signal that matters on screen._

- **Primary signal:** 71% Bounce rate with a 21-second average session duration, paired with zero action clicks on the surface but high engagement on deep-dive subpages.
- **What moved:** Card information density: Each card now articulates a clear causal chain (Metric Trend → Root Cause → Action Verb → Expected Business Impact → Confidence Rating).
Time-to-context: Users can evaluate the legitimacy and urgency of an action in under 10 seconds without navigating away from /.
Clarity of trade-offs: Confidence badges (High / Med / Low) provide transparency on data certainty.
- **What didn't:** Real-time backend telemetry for card views: Session recording still tracks overall bounce and click events rather than card-by-card hover/dwell time.
Spreadsheet export risk: The escape hatch to download raw data still exists on the legacy comparison screen and remains the fallback if trust isn't established.

_Analytics snapshot: visitors 422; page views 3.6k; views per visit 8.57; duration 21s; bounce 71%._

## Iteration sprint

| Change | Hypothesis | Result |
|---|---|---|
| Surfaced root-cause ("Why"), expected impact, and confidence ratings directly onto the four landing view metric cards without requiring clicks into detail pages. | Providing immediate justification (root cause, expected outcome, and confidence) on the card surface will overcome the trust barrier, reducing bounce from 71% and increasing primary action-pill clicks without forcing spreadsheet exports. | Landing cards now display full decision rationale (Why, Action, Impact, Confidence) in under 40 words with secondary drill-downs preserved; ready for re-testing against the 15s kill switch. |

## Peer feedback

"The original recommendation cards felt like black-box suggestions—I didn't know how much risk I was taking on by clicking 'Confirm and apply'. Seeing the root cause and expected impact right on the card makes it feel like an executive briefing rather than a blind click."

## The recommendation

**Decision:** ☐ Go  ☑ Iterate  ☐ Kill

_The evidence that justifies the call:_

Initial analytics revealed a bifurcated user pattern: 71% bounced within 21s without taking action, but the remaining 29% engaged heavily across all deep-dive screens (action details, evidence, and comparison). This confirmed the core value proposition is sound, but the initial cards were too opaque to earn quick trust. The sprint addressed this specific bottleneck.

## Final showcase

- **Demo link:** https://action-insight-now.lovable.app/
- **The one-sentence story:** An action-first analytics dashboard that turns buried metrics into one verified, recommended next step so teams never have to dump data into Excel to figure out what to do.
- **Where it landed on the Confidence Line (M2 → now):** Value risk closing; actively iterating on activation and trust. (We proved users reject 12-chart widgets and seek actionable clarity; we are now calibrating how much upfront context is needed to trigger the click-through before the 15-second kill switch).
