# Slide 02 — Executive Summary

**Slide type:** Framing / thesis statement
**Time:** 3 min
**The one thing:** Five recurring causes, one sequenced roadmap. The order of the roadmap is the argument.

## How to explain it

This slide carries the thesis. Everything after it is evidence. Spend the time.

1. **State the thesis in one sentence.** "Most of our latency is architectural, not infrastructural." Then define the difference: architectural means the shape of the call graph and where we put state; infrastructural means CPU, memory, instance count. "If we add servers to an architectural problem, we get a more expensive version of the same latency."

2. **Walk the left column, but don't read it.** Group the five into two families: *"Three of these are sequencing problems — sequential calls, thread blocking, duplicate calls. Two are payload problems — no cache tier, oversized JSON."* Grouping is what makes five bullets memorable.

3. **Let the right column do the work.** Point at it once: "These are the four things I'd want us to be held to." Don't narrate each row.

4. **Defend the roadmap order — this is the part worth the time.** The sequence isn't arbitrary:
   - *Parallel and caching come first* because they return the most latency for the least architectural risk, and they're reversible.
   - *Async comes third* because it changes consistency guarantees — a bigger commitment than the first two.
   - *gRPC comes fourth* because it adds toolchain and operational cost, so it should be paid for by demonstrated need.
   - *Resilience comes late* not because it's least important, but because the earlier changes alter the traffic patterns you'd be tuning breakers against. Tune them first and you tune them twice.

## Worked example

Use a duplicate-call story, because it's the one everybody recognises and nobody defends: *"On a page like ours, the customer profile typically gets fetched three times — once by the header for the name, once by the greeting widget, once by the entitlements check. Three network calls for one object. Nobody designed that; it accumulated."*

## Key points

- Architectural vs. infrastructural — define the distinction explicitly.
- Five causes reduce to **two families**: sequencing and payload.
- The roadmap is **sequenced by risk and reversibility**, not by impact alone.
- Resilience last is deliberate, and you should say why before someone assumes it's an oversight.

## Likely pushback

| Question | Answer |
|---|---|
| "Why isn't resilience first? Availability matters more than speed." | "Agreed on priority, disagree on sequence. Breaker thresholds are tuned against a traffic profile. The first four changes alter that profile substantially. Tune first and we do it twice — and the second time nobody trusts the first set of numbers." |
| "Can we do these in parallel across teams?" | "Caching and parallelism, yes — they're independent. gRPC and async need the resilience layer to exist first, or we're introducing new failure modes without the tooling to contain them." |
| "What's the actual cost of all this?" | Don't invent a number. "I haven't costed the full roadmap, deliberately. The ask at the end is scoped to one endpoint, which is a sprint of work. If that proves out, costing the rest becomes a much easier conversation." |

## Do not

- Read all five problems and all four outcomes. That's nine items narrated; the room will disengage by item four.
- Present the roadmap as fixed. Say "this is the sequence I'd defend" — inviting a challenge to the order is a good use of the room's expertise.
- Claim the outcome column as achieved. It's a target.

## Transition

"Let me show you what the current shape actually looks like, because the first three problems are all the same problem."
