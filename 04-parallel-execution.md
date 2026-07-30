# Slide 04 — Parallel API Execution

**Slide type:** Pattern / highest-value change
**Time:** 4 min
**The one thing:** Wall clock becomes the slowest call, not the sum. But the guardrails are where the work is.

## How to explain it

The pattern is obvious to this audience. Spend 30 seconds on the mechanism and three minutes on why it goes wrong — that's the part that earns credibility with principal engineers.

1. **State the mechanism once.** "Fan out, wait on all four, merge. `CompletableFuture.allOf` or `Mono.zip` depending on your stack. This is not a new idea."

2. **Point at the timeline.** "Wall clock is now 230 — the slowest call. Not 850. And notice we didn't make a single downstream service faster."

3. **Now spend most of the time on the three guardrails.** Frame them as the reason this fails in practice:

   - **Independence must be verified, not assumed.** "Somebody has to sit with the endpoint and confirm no call consumes another's output. In my experience one in four supposedly independent fan-outs has a hidden dependency — usually an ID that gets threaded through." If you have a real instance of this, use it; it's the most credible thing you can say on this slide.

   - **Dedicated executor, always.** This is the one that causes production incidents. "The default `CompletableFuture` executor is the common ForkJoinPool. It's sized to CPU count — typically 7 or 15 usable threads on our instances — and it's shared with every parallel stream in the JVM, including library code you didn't write. Put a blocking HTTP call on it and you will starve something unrelated. Give the fan-out its own bounded executor."

   - **Timeout per branch.** "Without this, we've made the page as slow as the slowest dependency on its worst day. With it, a slow branch degrades to a fallback and the other three still render. The timeout is what converts this from an optimisation into a resilience improvement."

4. **Close on the honest bound.** "60–80% is the range on genuinely sequential endpoints. On something already partially parallel, expect much less. The win is front-loaded — the first endpoint you fix is the biggest one."

## Worked example

The ForkJoinPool starvation story is the one to tell, because it's specific and everyone recognises the shape:

*"A team parallelises a fan-out with `CompletableFuture.supplyAsync` and no executor argument. Works perfectly in test. In production, under load, an unrelated batch job that uses parallel streams starts timing out — because both are on the common pool and the HTTP calls are holding threads that the batch job needs. The symptom appears in a service nobody changed."*

That story does more for the guardrail than any amount of explanation.

## Key points

- Wall clock = **slowest call**, not the sum.
- **No downstream service was made faster.** Say this explicitly — it pre-empts "shouldn't we fix the slow service instead?"
- Three guardrails: **independence, dedicated executor, per-branch timeout.**
- The ForkJoinPool default is a **production trap**, not a style preference.
- Expected range **60–80%**, front-loaded on the worst endpoint.

## Likely pushback

| Question | Answer |
|---|---|
| "Doesn't this quadruple load on downstream services?" | "It changes the arrival pattern, not the volume — same four calls, compressed in time. That does mean bursty load, which is exactly why per-dependency thread pools and rate limits matter. That's the next slide." |
| "Why not go fully reactive instead?" | "Reactive gets you the same latency win plus better thread economics, and costs a rewrite plus a learning curve across the team. `CompletableFuture` gets most of the latency win on the existing codebase this quarter. I'd take the incremental version first and treat reactive as a separate decision." |
| "How do we size the dedicated executor?" | "Threads ≈ target throughput × p99 latency, plus headroom. For a 200 req/s endpoint calling a 200 ms dependency, that's roughly 40 threads. There's a fuller version of the rule on the next slide." |
| "What does the page look like when one branch times out?" | "That's a product decision we need to make per widget, and we should make it before we ship this. Options are cached last-known-good, a skeleton, or hiding the widget. Silent empty state is the one to avoid." |

## Do not

- Spend time explaining what a `CompletableFuture` is.
- Present 60–80% as a guaranteed outcome.
- Skip the executor point to save time. It is the single most common way this pattern causes an incident.

## Transition

"Which brings us to the thread pools themselves — because compressing four calls into one burst makes pool design matter a lot more than it did."
