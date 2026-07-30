# Slide 13 — End-to-End Optimised Architecture

**Slide type:** Synthesis / architecture walkthrough
**Time:** 4 min
**The one thing:** Nothing here is novel. The value is that the patterns compose, and that observability is designed in rather than bolted on.

## How to explain it

This is the busiest slide in the deck. The risk is narrating every box. Use a fixed route and stick to it — the legend tells them what the colours mean, so you don't have to.

1. **Set expectations before you start pointing.** *"Nothing on this diagram is new to anyone in this room. The claim isn't novelty — it's that these compose, and that they were designed together rather than accumulated."*

2. **Follow one request, left to right.** This is the discipline that keeps the slide legible:
   - "Client hits NGINX — TLS, load balancing."
   - "Gateway — auth, rate limit, route."
   - "Dashboard BFF — this is where the aggregation happens. Parallel fan-out, four calls at once."
   - "Before it calls anything, it checks L1 then L2. 85% of the time it stops here and returns in single-digit milliseconds."
   - "On a miss, it fans out over gRPC to the four domain services, which read from their own data stores."

3. **Then point at the resilience box, once.** "Every one of those outbound arrows goes through the SDK — timeout, retry, breaker, bulkhead, fallback, rate limit. That's not drawn per arrow because it would be unreadable, but it applies to all of them."

4. **Then the async lane.** "Anything that doesn't need to complete before the response goes to Kafka. Four consumer groups, independent offsets. The user's response has already been sent by the time these run."

5. **Finish on the observability rail, and make it a point rather than a list.** *"This isn't a sidebar. Every pattern on this diagram makes the system harder to reason about. Parallel execution means four things happen at once. Caching means a request might not touch the service you're debugging. Async means the work happens after the response. Every one of those trades comprehensibility for speed — and distributed tracing is what buys it back. If we ship the patterns without the rail, we've built a fast system nobody can debug."*

   That's the strongest sentence available on this slide. Land it deliberately.

## Worked example

Trace one request through both paths — cache hit and cache miss — because it makes the 85% concrete:

*"Cache hit path: client, NGINX, gateway, BFF, L1. Return. Total maybe 15 milliseconds, and no domain service was involved at all. That's 85% of requests. Cache miss: same route to the BFF, then four parallel gRPC calls, slowest is 190 ms, populate both cache tiers, return at about 220. Then the order event goes to Kafka and four consumers pick it up over the next second, long after the user has their page. Two very different paths, one design."*

## Key points

- **Nothing is novel.** Say it first; it disarms the "we know all this" reaction.
- Follow **one request**, left to right, then down. Don't inventory boxes.
- **85% never reach a domain service.** The most important number on the diagram.
- The resilience box applies to **every outbound arrow**.
- Observability is **compensation for the comprehensibility these patterns cost** — not decoration.

## Likely pushback

| Question | Answer |
|---|---|
| "This is a two-year programme." | "As a whole, yes — and that's not what I'm proposing. This is the direction, not the plan. The ask at the end is one endpoint and one pattern. This slide exists so we know what the endpoint is heading towards, not so we approve it." |
| "What's the migration order?" | "Same as the roadmap on slide 2, and it's incremental by endpoint rather than by pattern. One endpoint gets parallelism and caching end to end; then the next. Doing it pattern-by-pattern across all endpoints means nothing is finished for a year." |
| "Where does the BFF pattern's proliferation problem land? One BFF per channel?" | "Fair — and it's not on the diagram. Mobile and web have different aggregation needs, and the honest answer is either one BFF with channel-aware responses or two BFFs with duplicated logic. Both are defensible; it needs its own decision." |
| "What happens if Redis goes down in this picture?" | "Fail open to origin. Latency goes from 220 back towards 850, everything still works. That's why the breaker sits around the Redis client too — the cache failing must never be an outage." |

## Do not

- Narrate every box. Follow the route.
- Read the legend aloud.
- Present this as the plan. It's the target state; the plan is one endpoint.

## Transition

"So what's it worth, and how would we know?"
