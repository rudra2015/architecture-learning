# Slide 01 — Title

**Slide type:** Framing
**Time:** 1 min
**The one thing:** This session is about design decisions, not about buying more servers.

## How to explain it

Three beats, and keep it short. The temptation on a title slide is to warm up. Don't — the 74% panel is doing the work.

1. **Name the scope.** "This is about latency on aggregation endpoints — the screens that pull from many services to render one view. Dashboards, account overviews, home pages."
2. **Point at the number.** "The claim I want to defend over the next 45 minutes is that we can take a page like that from 850 milliseconds to around 220, and that none of the changes required are infrastructure changes."
3. **Set the honesty marker immediately.** "Those figures are illustrative of a workload shaped like ours. I'll tell you where the real numbers need to come from at the end."

That third beat matters. Say it now, unprompted, and it stops being an attack surface for the rest of the session.

## Worked example

Don't use one yet. The banking dashboard example is introduced properly on slide 3 — spending it here weakens the reveal.

## Key points

- The subject is **structural latency**, not tuning.
- The savings come from **design choices already available to us**.
- The numbers are **illustrative** — flagged up front, not extracted from you later.

## Likely pushback

| Question | Answer |
|---|---|
| "Is 74% realistic?" | "On a genuinely sequential aggregation endpoint, yes — most of it comes from one change. On an endpoint that's already parallel, no. It depends entirely on the starting shape, which is why slide 3 is about the starting shape." |
| "We've heard this before." | Acknowledge it directly and move on: "Fair. What's different is that I'm proposing we prove it on one endpoint before proposing anything else." |

## Do not

- Read the chip row aloud. It's a table of contents; let them read it.
- Open with an apology, a joke about the meeting time, or a slide-count preview.
- Defend the 74% here. You defend it on slide 4, where the mechanism is visible.

## Transition

"So let me start with where the time actually goes."
