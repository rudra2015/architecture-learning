# Slide 11 — Circuit Breaker

**Slide type:** Pattern deep-dive / configuration
**Time:** 3 min
**The one thing:** A breaker without a designed fallback just produces errors faster. The fallback is the design work; the breaker is plumbing.

## How to explain it

Everyone knows the three states. Don't teach them — spend the time on the two things that are actually done badly: slow-call detection and fallback design.

1. **Walk the states in 30 seconds.** "Closed, traffic flows. Failure rate crosses the threshold, it opens and rejects immediately. After a wait, half-open — a few probe calls. Succeed, close. Fail, back to open." That's enough. Point at the diagram and move.

2. **Land the core value proposition in one line.** *"A breaker converts a slow, resource-consuming failure into a fast, cheap one."* Then unpack it: "Without a breaker, a dead dependency costs us a thread and a timeout — say two seconds — for every single request. With one, it costs microseconds. That's the difference between a degraded feature and an exhausted thread pool."

3. **Highlight slow-call threshold specifically.** This is the most-missed configuration item: *"Look at the third row — slow calls, 2 seconds at 60%. Most breakers only count errors. But a dependency returning 200 OK after eight seconds is doing more damage than one returning 500 instantly, because it holds our resources the whole time. Slow-call detection is what catches the failure mode that actually hurts. If you configure one thing beyond the defaults, configure this."*

4. **Make the fallback the centre of the slide.** Point at the example box and be blunt: *"Here's the part that gets skipped. A breaker with no fallback converts a timeout into a 503 faster. That is not resilience, that's a faster error. The design work is deciding, per dependency, what a degraded-but-useful response looks like."* Then give the taxonomy: cached last-known-good, a reduced version of the feature, a static default, or honest absence. "Silent empty state is the one to avoid — the user can't distinguish 'no data' from 'broken'."

5. **Say the config is per-dependency.** "These numbers are a starting point, not a standard. A breaker inherited from another service's traffic profile will trip wrongly — either too early on a low-volume endpoint or too late on a high-volume one. Minimum-calls exists precisely to stop a 3-call sample from tripping anything."

## Worked example

The Recommendation Service case on the slide is deliberately chosen — walk it as a decision, not a description:

*"Recommendations degrade under load and they're not business-critical. So: breaker opens after 10 failures in 20 calls, and the dashboard falls back to a curated static list. Page latency stays at 220 ms. The user never sees an error. They see slightly less personalised content, and almost none of them notice. That's what a good fallback looks like — the degradation is invisible."*

Then contrast, so the room sees the decision has two possible answers:

*"Now take the balance service. There is no acceptable fallback. A stale balance is worse than no balance, and a made-up balance is a regulatory problem. So the fallback is an explicit error state in that widget — 'balance unavailable, try again' — while the rest of the page renders normally. Same pattern, opposite answer, because the harm profile is different."*

That contrast is the most valuable thirty seconds on this slide.

## Key points

- Converts a **slow, expensive failure into a fast, cheap one**.
- **Slow-call detection** is the most-missed configuration and catches the worst failure mode.
- **Fallback is the design work.** No fallback = faster errors.
- Fallback taxonomy: **cached, reduced, static default, honest absence.** Never a silent empty state.
- Config is **per-dependency**. Minimum-calls prevents small-sample tripping.

## Likely pushback

| Question | Answer |
|---|---|
| "How do we test this? Failures are rare in lower environments." | "Deliberately — fault injection in the test suite, and ideally a game day where we open a breaker in production and watch. An untested fallback is a fallback that doesn't work; you find out during the incident." |
| "What about the thundering herd when it half-opens?" | "That's what limiting half-open probes to five does — a controlled sample, not the full traffic. Add jitter to the open duration across instances so twenty pods don't all probe at the same instant." |
| "Who decides the fallback behaviour?" | "Product, with engineering framing the options. Engineering can implement any of the four, but only product can say whether a stale recommendation is acceptable. If nobody will decide, the default should be honest absence, not silent empty." |
| "Does this hide problems from us?" | "It hides them from users, which is the point — but only if the breaker opening pages us. Breaker state transitions must be alerts, not just metrics. A breaker that opens silently for three weeks is a feature that's been quietly broken for three weeks." |

## Do not

- Teach the three states. This audience knows them.
- Present the config numbers as a standard.
- Let the fallback point become a footnote. It's the reason the slide exists.

## Transition

"That's the service-to-service layer. Let me go back out to the edge for a moment, because there's a common conflation worth clearing up."
