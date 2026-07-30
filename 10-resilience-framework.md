# Slide 10 — Enterprise Resilience Framework

**Slide type:** Platform / organisational argument
**Time:** 3 min
**The one thing:** The argument here is consistency and auditability, not cleverness. It's an organisational slide wearing an architecture diagram.

## How to explain it

The technical content is thin — everyone knows what a retry is. The argument is about where the code lives and who owns it. Pitch it accordingly, especially to directors.

1. **State the problem as an organisational one.** "Resilience implemented per team drifts. Thirty repos, thirty retry policies, twenty of them wrong in a different way. During an incident, nobody can answer 'what does this service do when its dependency times out?' without reading the code."

2. **Show the layering, briefly.** "Application code holds business logic. The SDK sits between it and the HTTP client. Application code doesn't know retries exist."

3. **Spend the time on ordering, because it's the part with real technical content.** Rate limiter → bulkhead → circuit breaker → retry → timeout → fallback. Then explain the one that matters:

   *"Retry goes inside the breaker, not outside it. If retries wrap the breaker, then one logical call becomes three physical calls against a dependency that's already struggling — we're amplifying load precisely when we should be reducing it. Worse, the breaker sees three failures instead of one and its error-rate maths is wrong. Wrapped correctly, the breaker sees one logical failure and trips on an accurate signal."*

   This is the beat that earns credibility with the principal engineers in the room. Don't rush it.

4. **Make the four SDK benefits organisational, not technical.**
   - Safe defaults, explicit overrides — "the default is correct, and deviating requires a review."
   - Config in one place — "we can change a timeout without a code release across thirty services."
   - Uniform metrics — "every call emits the same spans, so one dashboard covers the estate."
   - One upgrade path — "when the next CVE lands in the HTTP client, it's one bump, not thirty tickets."

   The last one usually lands hardest with leadership.

5. **Be honest about what this is.** "Resilience4j or equivalent does the work. The SDK is an opinionated wrapper plus a metrics contract. We're not building a resilience library — we're building a policy."

## Worked example

The retry-amplification incident is the most vivid thing you can offer:

*"A dependency degrades. Every caller retries three times. The dependency is now receiving 3× traffic while already failing. It doesn't recover — the retries are what's keeping it down. This is a documented pattern; it's called a retry storm, and the fix is exponential backoff with jitter plus a breaker outside the retry. But you only get that fix everywhere if it ships as a default, because no individual team will think about it while writing a feature."*

That last clause is the actual argument for the SDK.

## Key points

- The problem is **drift across teams**, not absence of resilience.
- **Retry inside the breaker.** Explain why — load amplification and a corrupted error signal.
- Four benefits, all **organisational**: safe defaults, central config, uniform telemetry, single upgrade path.
- The SDK is a **wrapper and a policy**, not a new library.
- Backoff with jitter as a default is something no individual team will remember under delivery pressure.

## Likely pushback

| Question | Answer |
|---|---|
| "Who builds and maintains this? We don't have a platform team." | Don't dodge this. "That's the real question. It's roughly a two-sprint build and then ongoing ownership. If we can't fund ownership, the honest alternative is a documented standard plus a shared config template — weaker, but better than nothing and it doesn't create an unowned artefact." |
| "This becomes a bottleneck. Teams will need changes we can't ship fast." | "Which is why overrides have to be a first-class feature, not an exception process. Defaults are opinionated, overriding is allowed and logged. If overriding requires a ticket to another team, teams will fork the SDK and we're back to drift." |
| "Service mesh does this. Why not Istio?" | "A mesh does timeouts, retries, and breakers at the network layer, and it's a legitimate alternative — arguably better, because it's language-agnostic and doesn't need a code change. What it can't do is business-aware fallback: the mesh can fail a call, but it can't decide to serve a cached recommendation list instead. In practice you want both — mesh for transport-level policy, SDK for fallback semantics." |
| "How do we migrate thirty services?" | "We don't, not as a programme. New services adopt it by default; existing ones adopt when they're next touched. Anything else becomes a migration project that competes with delivery and loses." |

## Do not

- Explain what a retry or a timeout is.
- Present this as a technical necessity. It's an organisational one, and framing it honestly is more persuasive.
- Skip the ownership question. A director-level audience will spot an unowned artefact immediately.

## Transition

"Let me take one of those seven and show it properly, because it's the one that's most often configured wrong."
