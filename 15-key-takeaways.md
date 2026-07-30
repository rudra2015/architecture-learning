# Slide 15 — Key Takeaways & Close

**Slide type:** Summary / the ask
**Time:** 2 min
**The one thing:** The ask is small and specific. One endpoint, one pattern, a measured result.

## How to explain it

Resist summarising. The seven principles are on screen and they read faster than you speak. Your job in these two minutes is the close, not the recap.

1. **Do not read the seven.** Say: *"These are on the slide and in the deck. I'll pull out two."*

2. **Pull out two — one technical, one cultural.**
   - Technical: *"'If two calls have no data dependency, running them in sequence is a defect.' I'd like us to treat it that way in design review — not as an optimisation opportunity, as a defect."*
   - Cultural: *"'Without a baseline and tracing, optimisation is opinion.' Most performance arguments in this organisation are currently opinion, including some of mine. Measurement is what makes them decidable."*

   Two beats a summary of seven. Pick the two that fit the room — swap in the caching or resilience principle if that's where the energy was during Q&A.

3. **Deliver the ask, and be precise about it.** *"I'm not asking for a platform programme. I'm asking for one aggregation endpoint, instrumented properly, with parallel execution applied and the result measured. That's roughly a sprint. If it delivers what slide 14 suggests, we have a real number to plan the rest against. If it doesn't, we've spent a sprint and learned something specific about why."*

   Say the last sentence. A visible willingness to be wrong is what makes the ask easy to approve.

4. **Say why sequencing beats scope.** *"A measured 70% on one endpoint buys more support than a platform proposal — because it's evidence rather than argument, and because it's cheap enough to say yes to."*

5. **Close on the line, then stop.** *"Latency is a design outcome, not an infrastructure problem."* Then stop talking. Don't add a thank-you paragraph or restate the agenda. Let the silence open Q&A.

## Worked example

If you need one more concrete beat before the ask, name the candidate endpoint explicitly rather than speaking abstractly:

*"Concretely: I'd pick the account overview endpoint. It's the highest-traffic aggregation we have, it's visibly sequential, it has no cross-call dependencies I'm aware of, and it's owned by one team — so we don't need three teams to coordinate a sprint."*

Naming a specific endpoint converts the ask from a proposal into a decision. If you know the endpoint, name it. If you don't, say what you'd use to choose: highest traffic, clearly sequential, single-team ownership.

## Key points

- **Don't read the seven.** Pull out two.
- The ask is **one endpoint, one pattern, one sprint.**
- **Name the endpoint** if you can.
- State the **falsification condition** — what happens if it doesn't work.
- Sequencing beats scope: **evidence over argument.**
- Close on the line and **stop talking.**

## Likely pushback

| Question | Answer |
|---|---|
| "Just one endpoint? Why not the top five?" | "Because one endpoint proves the pattern and the measurement approach at the same time, and it's small enough that nobody has to defend the decision. If it works, five is an easy follow-up. If it doesn't, five would have been five times the waste." |
| "Who does the work?" | Have an answer ready before you present. Naming a team and a sprint turns this from a proposal into a decision made in the room. If you don't have one, say what you'd need. |
| "What if the result is inconclusive?" | "Then the measurement wasn't set up properly, which is itself worth knowing before we invest more. Inconclusive is a failure of instrumentation, not of the pattern — and I'd rather find that out on one endpoint." |
| "Can you take this to the architecture forum?" | Yes, always. Ask for a date in the room. |

## Do not

- Recap the whole deck.
- End with "any questions?" — end with the line, then wait. Silence pulls questions out better than an invitation does.
- Add scope in the closing minute because the room seemed receptive. The small ask is the strategy.
- Leave without a named next step: an owner, an endpoint, or a date.

## After the session

Three things to do the same day:

- **Send the deck with the illustrative-numbers caveat in the email body**, not just on the slide. Slides get forwarded without context.
- **Write down every objection you got.** They're the agenda for the follow-up, and the ones you couldn't answer are the real gaps in the proposal.
- **Book the follow-up before the momentum decays.** A week is fine; a month means starting over.
