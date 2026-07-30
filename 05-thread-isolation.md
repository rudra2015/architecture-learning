# Slide 05 — Thread Pool Architecture & Bulkheads

**Slide type:** Pattern / failure containment
**Time:** 4 min
**The one thing:** Pool size is a property of the dependency, not of our traffic. Isolation converts a platform outage into a feature degradation.

## How to explain it

The room knows what a thread pool is. What they often haven't internalised is that pool sizing is derived from the *dependency's* behaviour, and that sharing a pool is a coupling decision.

1. **Establish the two pools have different jobs.** "The request handler pool is sized for incoming concurrency. The client pools are sized for downstream behaviour. These are unrelated numbers, and configuring one from the other is where it goes wrong."

2. **Make the sizing rule concrete.** Little's Law, but don't call it that unless the room would enjoy it: *"Threads equals throughput times latency. 100 requests per second against a dependency with a 200 ms p99 needs 20 threads to keep up. If that dependency degrades to 800 ms, the same traffic now needs 80. You don't have 80. That's the incident."* This is the single most useful thing on the slide — it explains why pools fail without any code changes on our side.

3. **Walk the failure scenario on the diagram.** Point at the Loan pool: "Loan is degraded. Its 20 threads are consumed. New Loan requests queue briefly and then fail fast. Now look at the other three — 12 of 30, 9 of 30, 6 of 20. Completely unaffected."

4. **Land the counterfactual, which is the actual argument.** *"On a shared pool, that same Loan degradation consumes every available thread. Profile requests start timing out. Account requests start timing out. Endpoints that never touch Loan start returning 503s. Then the health check fails, the instance gets pulled, and the load redistributes onto instances that are about to do the same thing."* Describe the cascade — that's what makes the case, not the healthy-state diagram.

5. **Name the trade-off honestly.** "The cost is that we're now holding more threads in total than a shared pool would need, and we have four numbers to maintain instead of one. That's real. It buys us blast radius containment."

## Worked example

The cascading outage is the canonical story and most senior people have lived through a version of it:

*"The classic incident report reads: a downstream recommendation service slowed from 80 ms to 4 seconds. Within 90 seconds, the entire dashboard tier was unhealthy — including endpoints that don't call recommendations at all. Nothing was wrong with those endpoints. They just couldn't get a thread."*

The Netflix Hystrix origin story is the same shape if you want an external reference, but the anonymous incident-report version usually lands better because the room fills in their own memory of it.

## Key points

- Two pool types, **two unrelated sizing inputs**.
- **Threads ≈ throughput × p99 latency**, plus headroom. Re-derive when the dependency's SLA changes.
- Isolation converts **platform outage → feature degradation**.
- The cascade is the argument. The healthy-state diagram is just the setup.
- The cost — more total threads, more config — is real and should be stated.

## Likely pushback

| Question | Answer |
|---|---|
| "Isn't this what Hystrix did, and isn't Hystrix dead?" | "Yes and yes. Hystrix is in maintenance; Resilience4j is the current implementation and it does semaphore-based bulkheads as well as thread-pool ones. The pattern outlived the library." |
| "Semaphore bulkhead or thread-pool bulkhead?" | "Thread pool if the call is blocking — it gives you real isolation and timeout enforcement. Semaphore if you're already non-blocking, since it's much cheaper and you don't need the thread hand-off. Mixing them in one service is fine as long as it's documented." |
| "We'll end up with 200 threads doing nothing most of the time." | "Mostly idle threads are cheap — about 1 MB of stack each, and no CPU. The expensive thing is the shared pool that looks efficient right up until it takes the service down." |
| "Who maintains these numbers?" | Be honest: "This is the real cost of the pattern. It needs an owner per dependency and a review trigger when the dependency's SLA changes. If we can't commit to that, we should use semaphore bulkheads with generous limits instead — less precise, much less maintenance." |

## Do not

- Present the healthy-state diagram as the point. The point is the cascade you describe verbally.
- Give exact thread counts as recommendations. They're illustrative of the method, not prescriptions.
- Skip the maintenance cost. An audience of directors will spot an unfunded operational burden.

## Transition

"Isolation stops a bad dependency from spreading. The next pattern stops us from calling it at all."
