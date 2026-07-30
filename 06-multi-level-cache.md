# Slide 06 — Multi-Level Cache

**Slide type:** Pattern / architecture walkthrough
**Time:** 4 min
**The one thing:** Two tiers because they solve different problems — L1 buys latency, L2 buys consistency and origin protection.

## How to explain it

The mistake here is presenting the tiers as "fast and faster." They have genuinely different properties, and the trade-off between them is the substance of the slide.

1. **Walk the tiers with their latency numbers.** "Sub-millisecond in-process. Two to five milliseconds over the network to Redis. 180 milliseconds to origin. Two orders of magnitude between the ends."

2. **Contrast the two tiers properly.** This is the beat that matters:
   - *L1 (Caffeine)* — fastest possible, zero network. But it's **per-instance**. Twenty pods means twenty copies, twenty invalidation problems, and a user hitting a different pod may see different data. Also lost on every deploy.
   - *L2 (Redis)* — one shared view, survives deploys, and one entry serves the whole fleet. Costs a network hop and introduces a new dependency that can itself fail.

   Then: *"L1 buys latency. L2 buys consistency and origin protection. That's why both, not one."*

3. **Explain the hit-rate arithmetic, because it's the real argument.** "60% at L1, 25% at L2, 15% to origin. Which means 85% of requests never reach a backend service. The latency win is nice. The load reduction is what changes our capacity conversation." Directors care more about the second one.

4. **Walk the four patterns with a sentence each.** They're mostly familiar; the one to dwell on is event-driven refresh.

   *"Cache-aside and lazy loading are the defaults — everybody has these. Warming is what stops the cold-start cliff after a deploy. Event-driven refresh is the interesting one: normally TTL is a compromise between freshness and load. If a domain event invalidates the key the moment the data changes, you can hold a 24-hour TTL on something that must be correct within seconds. That's not a tuning improvement, it's removing the trade-off."*

5. **State the discipline line and mean it.** "Cache the read, never the decision. Anything that authorises, prices, or moves money reads from source of truth. This isn't a performance rule, it's a correctness rule, and it's the one that keeps us out of a regulatory conversation."

## Worked example

Cold start after deploy — concrete, familiar, and it makes the warming case without argument:

*"Deploy at 9 a.m. Twenty pods come up with empty L1 caches. Every request is a miss. Origin load goes up 5× for about ninety seconds, right as morning traffic peaks. Warming the top few thousand keys at startup removes that entirely, and it's maybe forty lines of code."*

For event-driven refresh, use the profile case: *"A customer updates their address. Under a 24-hour TTL they might see the old one for the rest of the day. With an event on the profile topic, the key is invalidated in under a second — and we still get the 24-hour TTL's load reduction for the other 99.99% of the time."*

## Key points

- L1 and L2 exist for **different reasons**, not just different speeds.
- L1 is **per-instance** — say this explicitly, it's the property people forget.
- **85% of requests never reach origin.** That's the capacity argument.
- Event-driven refresh **removes** the freshness/load trade-off rather than balancing it.
- "Cache the read, never the decision" is a correctness rule.

## Likely pushback

| Question | Answer |
|---|---|
| "Isn't L1 dangerous if pods disagree?" | "Yes, and that's a per-entity decision, not a global one. Reference data and profile — fine. Anything a user might see change within one session — L2 only. The TTL table on the next slide is where we make that call explicitly." |
| "What happens when Redis is down?" | "We fail open to origin, not closed. The cache is an optimisation, so its failure mode must be degraded latency, not an outage. That needs a circuit breaker around the Redis client too — which is slide 11's pattern applied here." |
| "Why not just Redis? One tier is simpler." | "Defensible, and if I had to pick one it'd be Redis. L1 earns its complexity on the very hot keys — the ones read on every single request, where 3 ms × every request is real. If we start with Redis only and add Caffeine where profiling justifies it, I'd support that." |
| "How do we know the hit rate will be 85%?" | "We don't. That's a target based on typical read/write ratios for this kind of data. It's measurable from day one — hit rate should be on the dashboard before we tune anything." |

## Do not

- Describe the tiers as just "fast" and "faster."
- Skip the per-instance property of L1.
- Present hit rates as forecasts. They're targets.

## Transition

"Which raises the question the technology doesn't answer — what goes in, and for how long."
