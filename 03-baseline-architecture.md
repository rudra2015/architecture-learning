# Slide 03 — Baseline Architecture

**Slide type:** Diagnosis / architecture walkthrough
**Time:** 3 min
**The one thing:** There are two separate problems here — wall-clock latency and a blocked thread — and they need different fixes.

## How to explain it

This is the diagnostic slide. Get the separation of the two problems right and slides 4 and 5 explain themselves.

1. **Walk the call path quickly.** Left card, ten seconds: "Client, gateway, dashboard service. Standard." Don't linger — nobody in this room needs this explained.

2. **Move to the timeline and let it sit.** This is the slide's centre of gravity. "Profile finishes, then Account starts. Account finishes, then Loan starts. Nothing here is waiting on anything else's data — this is purely how the code was written."

3. **Do the arithmetic out loud.** "190 plus 210 plus 220 plus 230. The page costs the sum of every call it makes. Add a fifth widget next quarter and the page gets slower by the full cost of that widget — not by a bit, by all of it."

4. **Now separate the second problem explicitly.** Pause and mark the shift: *"There's a second thing happening on this chart that's easy to miss. For all 850 milliseconds, the request thread is doing nothing. It isn't computing. It's blocked on a socket."* Then land the consequence: "This is why you see thread pool exhaustion at 12% CPU. We're not out of compute — we're out of threads to hold open."

5. **Third card, one line each.** They're consequences, not new information.

## Worked example

The Netflix home page is the cleanest public analogue and the brief calls for it: *"A Netflix home row — continue watching, trending, because-you-watched — is exactly this shape. Independent reads, one screen. The difference is they fan out and we don't."*

If the room is BFSI-heavy, the banking version lands harder because it's ours: *"Our account overview is four domain reads with no dependency between them. Profile doesn't need the balance. Loan doesn't need the card."*

## Key points

- The four calls are **independent** — establish this explicitly, because slide 4's entire argument depends on it.
- Latency is **additive**, so every new feature has a compounding cost.
- The blocked thread is a **separate failure mode** from slow response, and it's the one that causes outages rather than complaints.
- Thread exhaustion at low CPU is the diagnostic signature. Name it — several people in the room will have seen it on a dashboard and not known what they were looking at.

## Likely pushback

| Question | Answer |
|---|---|
| "Are they genuinely independent? Loan pricing might need the profile." | "Good — and that's the check that has to happen per endpoint before this pattern applies. Where there's a real dependency, that branch stays sequential and we parallelise around it. I'd rather find one true dependency now than assume none exist." |
| "Isn't 190 ms for a profile fetch the actual problem?" | "It's *a* problem, and worth its own investigation. But fixing it gets us from 850 to maybe 700. Fixing the sequencing gets us to 230 without touching any downstream service. Different order of magnitude, and one of them doesn't require four other teams." |
| "We're on Netty/WebFlux, threads aren't blocked." | "Then the second problem is already solved for you and the first one isn't. The timeline still applies — reactive code written sequentially is still sequential." |

## Do not

- Skip the thread-blocking point to save time. It's the setup for slide 5, and slide 5 makes no sense without it.
- Show sympathy for the design. It accumulated; nobody chose it. Blame is a distraction.
- Get drawn into optimising the 190 ms. That's a different session.

## Transition

"So the first fix is the obvious one — and it's obvious enough that the interesting part is the caveats."
