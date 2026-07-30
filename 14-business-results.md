# Slide 14 — Business Results

**Slide type:** Data / justification
**Time:** 3 min
**The one thing:** These are illustrative. Say it before anyone asks, and the credibility you keep is worth more than the numbers you'd have claimed.

## How to explain it

This is the slide most likely to be quoted back at you in six months. Handle the provenance of the numbers first and everything else gets easier.

1. **Flag the provenance immediately — first sentence, unprompted.** *"Before I walk these: these are illustrative figures for a workload shaped like ours, not measurements from our estate. I'll come to what we'd need to do to make them real."*

   This costs you nothing and buys you the room. The alternative is someone asking "where did 850 come from?" at minute two and you spending the rest of the slide on the back foot.

2. **Walk the chart, because it does the attribution work.** "The chart matters more than the four stat cards, because it shows which change bought what:
   - Baseline 850.
   - Parallelism alone takes it to 380. **That's more than half the total win from one change.**
   - Caching takes it to 260.
   - gRPC takes it to 220."

   Then draw the conclusion out loud: *"Look at the shape. The first change delivers most of the value. Each subsequent one delivers less for more effort. That's the argument for the sequencing on slide 2, and it's also the argument for stopping when the curve flattens — we don't have to do all of this."*

   Directors respond well to being told where to stop.

3. **Use the stat cards selectively.** Don't read four cards. Pick the two that matter to this audience:
   - **Availability 99.5 → 99.95%** — "that's 3.6 hours of downtime per month becoming 22 minutes. Framed as minutes, it's a different conversation than framed as nines."
   - **Cache hit 20 → 85%** — "which is really a capacity statement: four times fewer origin reads means the existing fleet absorbs considerably more traffic before we buy anything."

4. **Give the second-order effects proper weight.** These are often more persuasive to leadership than the latency number: deferred infrastructure spend, fewer thread-exhaustion incidents, shorter MTTR, degraded-mode instead of hard failure, and latency budgets becoming a design input rather than a post-incident finding.

   The last one is worth saying slowly: *"Right now, latency is something we discover after we ship. The end state is that it's a number in the design review."*

5. **Close on measurement discipline.** "Baseline in Dynatrace first, then attribute each delta to a specific change. Unattributed wins get argued away — someone will say traffic was lower that week, and without attribution they'll be right to."

## Worked example

Convert the availability figure, because nines are abstract and minutes aren't:

*"99.5% sounds close to 99.95%. It's 3.6 hours of downtime a month versus 22 minutes. That's the difference between a quarterly incident review with an executive summary and one that nobody schedules."*

And a caution on the cache figure, if the room is quantitatively minded:

*"One honest note on the 85% — that's an aggregate. Hit rates vary enormously by entity. Profile might hit 95%; balance with a 30-second TTL might hit 40%. The blended number is right and the per-entity numbers are what we'd actually tune against."*

## Key points

- **Illustrative. Stated first, unprompted.**
- The chart shows **attribution and diminishing returns** — that's its job, not decoration.
- **Parallelism alone is more than half the win.** The most actionable fact in the deck.
- Convert nines to **minutes**.
- Cache hit rate is a **capacity** argument as much as a latency one.
- **Measure before you optimise**, or the win gets argued away.

## Likely pushback

| Question | Answer |
|---|---|
| "Where did these numbers come from?" | Already answered in your opening sentence. Reinforce: "Typical figures for this workload shape. Our real baseline is a week of work to establish and it should be the first thing we do." |
| "What's the p99, not the p95?" | "Fair challenge and I don't have it. p99 improves less from parallelism because it's dominated by the worst branch, and improves more from caching and breakers. Worth adding to the baseline work — p99 is the number the complaints come from." |
| "Doesn't caching just hide a slow backend?" | "Yes, for 85% of requests. The other 15% still feel it, and if the backend degrades further, the hit rate is what stands between us and an outage. Caching buys time to fix the backend; it isn't the fix." |
| "What's the cost side of this?" | "Redis infrastructure, engineering time, and ongoing config ownership. I haven't costed it because the ask is scoped to one endpoint — roughly a sprint. If that proves out, a full costing is a much easier exercise with real numbers behind it." |
| "How long until we see this?" | "First endpoint, parallelism only: a sprint, and it's measurable immediately. The full picture is quarters, and I'd want us to re-decide after the first result rather than commit now." |

## Do not

- Let anyone believe these are measured. Say it first.
- Read all four stat cards.
- Promise a timeline for the full roadmap.
- Defend a specific number under challenge — concede the uncertainty and point at the measurement plan. Defending an illustrative number is how you lose the real argument.

## Transition

"So — seven things to take away, and one thing I'm actually asking for."
