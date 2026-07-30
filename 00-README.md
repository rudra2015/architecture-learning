# Speaker Notes — Performance & Latency Optimization

One file per slide. Each file follows the same structure so you can scan them in order or pull a single one before a section.

## Structure of each file

| Section | What it gives you |
|---|---|
| **Slide type** | What kind of explanation this slide needs — framing, walkthrough, data, or decision |
| **Time** | Target minutes. Total is ~48 min, leaving room for Q&A in a 60-min slot |
| **The one thing** | If they remember nothing else from this slide |
| **How to explain it** | The narrative beats, in order, with what to actually say |
| **Worked example** | The concrete case to reach for. Abstract patterns don't land; a specific broken dashboard does |
| **Key points** | The claims that must be made out loud, not just shown |
| **Likely pushback** | The questions a room of principal engineers will actually ask, with answers |
| **Do not** | Failure modes specific to this slide |
| **Transition** | The bridge sentence to the next slide |

## Session flow

The deck has four movements. Signposting them out loud helps the room track where you are.

1. **Frame the problem** — slides 1–3. Establish that latency is architectural, not an infrastructure shortfall.
2. **The patterns** — slides 4–11. Eight slides, six patterns. This is the body of the session.
3. **Compose them** — slides 12–13. Show the patterns working together, not as a menu.
4. **Justify and close** — slides 14–15. What it's worth, and what you're actually asking for.

## Before you present

Three things to settle first:

- **Replace the illustrative numbers with your own baseline.** 850 ms → 220 ms is plausible, not measured. If someone asks "is that our number?" and the answer is no, you lose the room for the rest of the session. Pull real p95 from Dynatrace for one endpoint and use that instead.
- **Decide what you are asking for.** The deck as written closes on "let us instrument one endpoint and prove the first pattern." If the ask is bigger — funding, headcount, a platform team — slide 15 needs rewriting.
- **Know which patterns you have already tried and failed at.** Someone in the room will remember. Naming it first ("we attempted this in 2024 and it stalled because X") is far stronger than being reminded.

## Adapting for a shorter slot

- **30 minutes:** slides 1, 2, 3, 4, 6, 10, 13, 14, 15. Cut thread pools, cache policy, async, gRPC, circuit breaker.
- **15 minutes:** slides 2, 3, 4, 13, 15. Problem, one pattern proven, target state, ask.
