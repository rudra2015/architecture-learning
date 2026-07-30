# Slide 12 — API Gateway & Edge Architecture

**Slide type:** Clarification / separation of concerns
**Time:** 3 min
**The one thing:** NGINX and the API Gateway do different jobs. Conflating them is why gateway configs become unmaintainable.

## How to explain it

The value of this slide is clearing up a conflation, not teaching what a gateway is. Frame it that way up front.

1. **State the split as a principle.** *"Two layers, two jobs. Edge concerns belong in something built for connections. API concerns belong in something built for policy."*
   - **NGINX** — TLS termination, L7 load balancing, static assets, connection handling, DDoS absorption. "Connection-level work. It doesn't know what an API is and shouldn't."
   - **API Gateway** — authentication, JWT validation, per-API rate limits, routing, versioning, transformation. "Policy work. It doesn't care about TLS ciphers."

2. **Explain what happens when you conflate them.** "Put API-level policy into NGINX and you get a config file with three thousand lines of Lua that one person understands and nobody wants to change. Put connection handling into the gateway and you've built a load balancer with worse performance characteristics than the one you already have."

3. **Walk the responsibilities grid quickly.** Eight tiles, don't read them. Group instead: "Four of these are security and access — auth, JWT, rate limiting, versioning. Four are traffic and operations — routing, logging, monitoring, transformation."

4. **Make the pattern-vs-product point, and make it the memorable one.** *"'API Gateway' is a pattern — a single policy enforcement point in front of many services. Zuul, Spring Cloud Gateway, Kong, APIM are implementations. Choose on operational fit, not on the label."*

   Then land the callback that ties the slide to the deck: *"And this matters concretely. Zuul 1's filter chain is blocking — one thread per request, held for the duration. That is precisely the thread-blocking problem from slide 3, just moved to the edge. Slow backend, exhausted gateway threads, and now everything behind it is unreachable, not just the slow service. The pattern was fine; the implementation reintroduced the problem we were trying to solve."*

5. **Give the selection criteria.** "So what should we actually choose on? Filter model — blocking or non-blocking. Whether it supports the auth mechanisms we already use. Config-as-code or a UI. Whether our observability tooling traces it. Not on which one has the best marketing."

## Worked example

The Zuul 1 → Zuul 2 / Spring Cloud Gateway migration is the strongest example because it's a real, well-documented industry lesson:

*"Teams adopted Zuul 1 for exactly the right reasons, then hit thread exhaustion at the edge under load — because the filter chain was synchronous. The industry moved to Zuul 2 and Spring Cloud Gateway, both non-blocking. Same pattern, different implementation, problem gone. The lesson isn't 'Zuul was bad' — it's that choosing the pattern doesn't finish the decision."*

For the conflation point, if the room needs it grounded:

*"You know you've conflated them when someone asks to add a per-customer rate limit and the answer is 'that means an NGINX config change and a restart.' Rate limiting is an API policy. It shouldn't require touching the thing that terminates TLS."*

## Key points

- **Edge = connections. Gateway = policy.** One sentence, states the whole slide.
- Conflation produces **unmaintainable config** in one direction, **worse load balancing** in the other.
- Eight responsibilities group into **security/access** and **traffic/operations**.
- **Pattern ≠ implementation.** Choose on operational fit.
- Zuul 1's blocking filter chain is **slide 3's problem at the edge** — the callback that makes this slide stick.

## Likely pushback

| Question | Answer |
|---|---|
| "Isn't two layers just extra latency?" | "Roughly a millisecond. Against the 850 we started with, it's noise — and it buys a separation that keeps both layers comprehensible. If we ever find that millisecond matters, we've solved a lot of other things first." |
| "Service mesh replaces the gateway, doesn't it?" | "Different axis. A mesh handles east-west, service to service. The gateway handles north-south, outside to inside. They coexist — the mesh doesn't do JWT validation for external callers, and the gateway doesn't do mTLS between pods." |
| "Could the BFF just do all of this?" | "It could, and then every BFF reimplements auth and rate limiting. That's the drift argument from slide 10 in a different costume. Cross-cutting concerns want one enforcement point." |
| "We already have Zuul 1 in production." | "Then this is a real item, not a hypothetical. It's worth knowing whether our edge thread pool has headroom under a slow-backend scenario. That's a load test, not a migration — let's find out before deciding anything." |

## Do not

- Explain what an API gateway is.
- Read the eight responsibility tiles.
- Turn the Zuul point into criticism of whoever chose it. It was the right call at the time; the lesson is about implementation properties, not judgement.

## Transition

"So that's every pattern individually. Here's what they look like composed."
