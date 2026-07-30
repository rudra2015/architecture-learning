# Slide 08 — Asynchronous Processing

**Slide type:** Pattern / architectural trade-off
**Time:** 4 min
**The one thing:** This is the first pattern that changes our consistency guarantees. That makes it a bigger commitment than everything before it.

## How to explain it

Slides 4–7 were essentially free — faster with no semantic change. This one isn't, and being upfront about that is what makes the rest of the deck credible.

1. **Show the before/after strip and do the arithmetic.** "900 milliseconds becomes 120. But look at what actually changed — we didn't make notification faster. We stopped waiting for it."

2. **Give them the decision test immediately, before the architecture.** This is the most useful sentence on the slide: *"If a failure in this work should fail the user's request, it is not async work."* Then apply it: "Payment capture fails, the order fails — synchronous. Receipt email fails, the order stands — async. The test isn't 'is this slow', it's 'does the user's outcome depend on it'."

3. **Walk the event flow briefly.** Order publishes to `orders.v1`; three consumer groups read independently. "Each group commits its own offset, so a slow analytics consumer doesn't hold up notifications. That independence is the whole point of a log rather than a queue."

4. **Now spend real time on the cost.** Don't let this be a footnote:
   - **Idempotency is mandatory.** "Kafka gives at-least-once by default. Your notification consumer will process the same message twice — on rebalance, on redeploy, on a commit failure. Without a dedup key, the customer gets two emails. In a payments context, worse."
   - **Eventual consistency is user-visible.** "The user gets their 120 ms response before the audit record exists. If someone queries the audit API in that window, it isn't there. That's not a bug, it's the design — but it has to be a *decided* design, and the UI has to be honest about what's confirmed versus in flight."
   - **Dead-letter handling is not optional.** "A poison message with no DLQ blocks the partition. Everything behind it stops. Decide the DLQ and the replay procedure before the first message flows, not during the first incident."

5. **Name the debuggability cost.** "In the synchronous version, one stack trace tells you what happened. In this version, you need correlation IDs across four services and a tracing tool to answer 'did it happen?'. That's real operational overhead and it's why observability is on the target-state diagram, not bolted on later."

## Worked example

The double-notification story is the one to tell, because it's mundane and everyone believes it:

*"Consumer processes a message, sends the email, and the pod is killed before it commits the offset. On restart, the message is redelivered. Customer gets two emails. Annoying. Now make it a payment confirmation SMS during a market event, and it's a contact-centre spike and a complaint. The fix is small — a dedup key with a short-lived store — but it has to be designed in, not discovered."*

For the eventual-consistency point: *"User completes a transfer, sees success, immediately opens the statement page. The audit record is 200 ms behind. What does the statement show? If we haven't decided, the answer is whatever the code happens to do."*

## Key points

- The test: **does the user's outcome depend on it?** Not "is it slow."
- **Consumer groups commit independently** — that's what makes a log better than a queue here.
- **Idempotency is mandatory**, because at-least-once means duplicates will happen.
- **Eventual consistency is user-visible** and needs a UI decision.
- **DLQ and replay before first message**, not during the first incident.
- Debuggability cost is real and is why observability appears in the target state.

## Likely pushback

| Question | Answer |
|---|---|
| "Do we need Kafka? We already have MQ / SNS+SQS." | "Probably not, if what you have supports consumer groups and replay. What matters is independent consumers and the ability to replay from an offset. If the existing broker does that, use it — Kafka isn't the pattern, it's an implementation of it." |
| "How do we answer 'did it happen?' for a regulator?" | "The audit consumer has to be the one with the strongest guarantees — manual offset commit, DLQ with alerting, and a reconciliation job comparing published against persisted. If we can't commit to that, audit stays synchronous. That's a legitimate outcome for that one consumer." |
| "This adds a whole new failure surface." | "It does. It trades a synchronous failure — which the user sees — for an asynchronous one, which they don't see but which we have to detect. That's a good trade only if we invest in the detection. Without monitoring on consumer lag and DLQ depth, it's a bad trade." |
| "What's the ordering guarantee?" | "Per partition, and we're partitioning by customer. So all events for one customer are ordered; across customers there's no guarantee. That's usually what you want — check it holds for each consumer." |

## Do not

- Present async as free. It's the first pattern with a real semantic cost.
- Skip idempotency or DLQ to save time. Cut the architecture walkthrough instead.
- Let "eventual consistency" pass as jargon. Say what the user actually sees.

## Transition

"That covers what we call and when. The next question is how — the wire format between our own services."
