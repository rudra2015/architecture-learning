# Slide 07 — Cache Strategy & TTL Policy

**Slide type:** Governance artefact / decision framework
**Time:** 3 min
**The one thing:** TTL is a statement about acceptable staleness, and staleness is a business risk — so engineering shouldn't be setting these numbers alone.

## How to explain it

This slide is a different mode from the rest of the deck. Signal the shift: *"This one isn't an architecture slide, it's a governance artefact. It's the table I'd want to exist before we cache anything."*

1. **Explain the reframe.** "A TTL looks like a config value. It's actually a statement that says: we accept the risk of showing data this stale. Written that way, it's obviously not a decision engineering should make on its own."

2. **Walk the gradient, not the rows.** The table is designed to be read; narrate the shape instead. "Top to bottom, this runs from 24 hours to never. What changes as you go down isn't how often the data changes — it's how much harm stale data does."

3. **Contrast the two ends deliberately.**
   - *Profile at 24 hours:* "Changes are rare, and when they do change we get an event. Long TTL plus event invalidation gives us both properties."
   - *Payments at never:* "Not slow, not risky — never. Correctness and auditability outrank latency without exception. And crucially, this is a decision we should be able to point at during an audit."

4. **Use transaction history to show it's not binary.** "Page one is volatile — new transactions land constantly. Page fourteen is immutable and will never change again. Same entity, different caching policy by page. Most entities have a split like this if you look."

5. **Close on governance.** "Named owner, documented staleness tolerance, quarterly review. TTLs rot — a value that was right when the business had one campaign a quarter is wrong when marketing goes weekly."

## Worked example

The balance example makes the 30-second row concrete and shows the reasoning is about harm, not volatility:

*"Why 30 seconds on balance? A customer transfers money and immediately checks the balance. Thirty seconds of staleness reads as slightly laggy. Five minutes reads as 'my money is missing' — and that's a call to the contact centre, possibly a complaint. The TTL isn't derived from how often balances change. It's derived from what a stale balance costs us."*

That reframe — cost of staleness, not rate of change — is the most transferable idea on the slide.

## Key points

- TTL = **accepted staleness risk**, so it's a joint decision with the business.
- The gradient tracks **harm from staleness**, not rate of change.
- Long TTL + event invalidation gives **both** freshness and load reduction.
- Caching policy can vary **within** one entity — transaction history proves it.
- Governance: **owner, rationale, quarterly review.**

## Likely pushback

| Question | Answer |
|---|---|
| "This is a lot of process for a cache." | "It's five rows, decided once, reviewed quarterly. The alternative is TTLs picked ad hoc in PRs, and then nobody can answer 'why is this 60 seconds?' during an incident." |
| "Who actually owns these decisions?" | "Product or the domain owner sets the tolerance; engineering implements it. If nobody will own the tolerance for an entity, that's the signal not to cache it yet." |
| "Payments never cached — not even idempotency keys or a read-only receipt?" | "Fair distinction. 'Never' applies to anything on the authorisation path. An immutable receipt after settlement is a different object with different rules, and the table should say so. Good catch — I'll split that row." (Accept this one; it's a legitimate refinement and accepting it visibly costs you nothing.) |
| "What about GDPR / data residency in the cache?" | "Real constraint and not on this table. Cached PII is still PII — it needs the same residency, encryption, and deletion guarantees as the source. A deletion request has to purge the cache too. That should be a column here." |

## Do not

- Read the table row by row. It's a reference artefact; the audience reads faster than you talk.
- Defend the specific TTL values as correct. They're illustrative of the *method*.
- Skip the governance line. Without it this is a table nobody maintains.

## Transition

"Caching removes reads from the request path. The next pattern removes writes."
