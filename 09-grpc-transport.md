# Slide 09 — gRPC and the Transport Boundary

**Slide type:** Pattern / conditional recommendation
**Time:** 4 min
**The one thing:** This is an optimisation with a real operational cost. Recommend it conditionally, or the room will correctly read it as technology advocacy.

## How to explain it

Of all the patterns, this is the one most likely to be heard as "the architect wants a new toy." Structure the explanation to pre-empt that.

1. **Establish the boundary first, before any gRPC advocacy.** "REST stays on the public side. Every external contract, every browser, every partner. Nothing changes for anyone outside our perimeter. What we're discussing is the hop from the aggregator to our own services."

2. **Explain where the time actually goes, and be precise.** "The win is three things, and they're not equal:
   - *Payload size* — 18 KB of JSON becomes about 4 KB of Protobuf. That's transfer time and bandwidth.
   - *Serialisation* — roughly 8 ms to 1 ms. On one call that's noise. On a fan-out of four, per request, at our volume, it isn't.
   - *HTTP/2 multiplexing* — one connection carries all four calls. No connection pool contention, no head-of-line blocking at the connection level."

   Then: "Notice all three matter more the chattier the path is. That's the selection criterion."

3. **Mention the free extras briefly.** "Deadlines propagate through the call graph automatically, and clients are generated from the IDL. The generated clients are underrated — schema drift becomes a build failure instead of a production incident."

4. **Now give the counterpoint proper airtime.** Deliberately spend as long here as on the benefits:
   *"gRPC costs us something. Debugging is harder — you can't curl it, you can't read it in a proxy log without tooling. Load balancers need HTTP/2 support and L7 awareness, because L4 balancing on a multiplexed connection pins all traffic to one backend. Every consumer needs the toolchain. And there's a learning curve across teams that doesn't show up in any latency chart."*

5. **Land the conditional recommendation.** "So: gRPC where the traffic justifies it. High-volume internal fan-out — yes. A service called forty times a day — no, the 7 ms saving isn't worth the operational surface. This is not a mandate."

## Worked example

Make the volume argument concrete, because it's what turns this from preference into arithmetic:

*"At 500 requests per second, our dashboard makes 2,000 internal calls per second. At 7 ms of serialisation saved per call, that's 14 seconds of CPU per second across the fleet — a meaningful fraction of a machine. On a service called forty times a day, the same 7 ms saves nothing anyone can measure. Same technology, opposite decision."*

For the load-balancer trap, if the room is operationally minded:

*"The classic gRPC production surprise: you deploy behind an L4 load balancer, everything works in test, and in production 90% of traffic lands on one pod. HTTP/2 multiplexes over one long-lived connection, so L4 balancing balances connections — of which there's one. You need L7-aware balancing or client-side load balancing. It's fixable, but nobody expects it the first time."*

## Key points

- **Nothing changes externally.** Establish this before anything else.
- Three separate mechanisms: **payload, serialisation, multiplexing** — and they all scale with chattiness.
- **Generated clients turn schema drift into build failures.** Underrated benefit.
- The costs — **debuggability, L7 load balancing, toolchain, learning curve** — get equal airtime.
- **Conditional, per-path recommendation. Not a mandate.**

## Likely pushback

| Question | Answer |
|---|---|
| "This is a big migration for milliseconds." | "Agreed, if applied everywhere. Which is why I'm proposing it on the highest-volume internal paths only — probably two or three hops, not the whole estate. If we can't identify a path where the volume justifies it, we shouldn't do it at all." |
| "Our observability tooling doesn't parse gRPC." | "That's a genuine blocker and it should gate the decision. If Dynatrace can't trace it, we lose more in incident response than we gain in latency. Worth confirming before we commit — I'd treat it as a prerequisite, not a follow-up." |
| "What about the versioning story?" | "Protobuf's compatibility rules are stricter than JSON's, which is mostly good — add fields freely, never reuse tag numbers, never change types. Stricter than what teams are used to with JSON, so it needs a written convention and review discipline." |
| "Could we get most of this by just shrinking the JSON?" | "Partly, and it's worth doing regardless — trimming unused fields is free. You'd get some of the payload win, none of the serialisation win, and none of the multiplexing win. It's a reasonable first step that tells us whether payload size is actually the constraint." |

## Do not

- Present gRPC as strictly better. It's better on a specific axis at a specific volume.
- Rush the counterpoint box. Giving it equal time is what makes the recommendation credible.
- Let anyone leave thinking external APIs are changing.

## Transition

"Those are the performance patterns. The rest is about what happens when they fail."
