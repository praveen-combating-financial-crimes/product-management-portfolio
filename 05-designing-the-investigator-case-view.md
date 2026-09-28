# Designing the Investigator's Case View on Top of Entity Resolution

*A product-design walk-through, using the CIRCLES method, of the screen a financial crime investigator opens first, and why building it on a resolved view of the customer cut investigation time from about two hours to about thirty minutes.*

**Author:** Praveen Sridharan  
**Status:** Design case study from a production implementation.

## Abstract

A bank has to watch for suspicious activity in its customers' accounts, and when something looks off, an investigator has to review it and decide whether to report it to regulators. Before an investigator can judge whether an activity is suspicious, they first have to understand who the customer actually is: what accounts they hold, who they're connected to, and what they've done across every product, not just the one that triggered the alert. That customer picture is usually scattered across several separate computer systems, so pulling it together today is a slow, manual task the investigator repeats on every case, and it is where most of an investigator's time actually goes. This paper walks through, step by step, how a single screen was designed to assemble that picture automatically, using a design method called CIRCLES: understand the situation, identify who the design is for, understand what they need, decide what to build first, weigh different approaches, and summarize a recommendation. The resulting design shows an investigator one customer, their connections, their accounts, and their activity, on a single page, and deliberately leaves off anything they don't need to make a decision. In practice, it cut average investigation time from about two hours to about thirty minutes.

## Comprehend the Situation

The bank runs lending and deposit products on separate originations and servicing systems, with identity verification, sanctions screening, transaction monitoring, and case management as further systems on top. Each one holds only part of a customer's record. An investigator picking up an alert has to work out who the customer is before they can work out what the customer did.

Clarifying questions, and the answers that shaped the design:

- **What is the goal?** Reduce time per investigation without reducing the quality of the decision or the report that follows it.
- **What are the constraints?** Regulatory clocks on filing decisions, a full audit trail of what the investigator saw and when, entitlements that differ by system, and an engineering team that is shared with other compliance priorities.
- **What already exists?** Entity Resolution, a single resolved view of a customer across systems, does not exist yet. Investigators, and every other team that needs a customer's full picture, pull information directly from whichever siloed system they have access to. Those siloed systems support more than 95% of use cases without any trouble on their own; the trouble concentrates in the edge cases, predominantly customers who hold more than one product or have more than one kind of relationship in how they use those products.
- **What is out of scope?** Alert generation, rule tuning, and the filing workflow itself. The case view feeds those; it does not replace them.

## Identify the Customer

Four groups need a complete picture of a customer. Only one of them sits inside the financial crime program.

| User | What they are trying to do | What slows them down today |
| --- | --- | --- |
| AML and Fraud Investigators (one group) | Decide whether activity is suspicious, or whether a customer or a third party is behind it, and document why | Assembling the customer across systems by hand, redone on every related case, against a regulatory or loss-prevention clock |
| Customer Service Operations | Answer a customer's question about the full relationship they have with the bank | No single place to see everything a customer holds |
| Marketing Operations | Understand a customer's existing relationship before targeting them with an offer | Same fragmentation, seen from the other direction: is this someone we already serve, and how? |
| Collections Operations | Decide how to work a delinquent account in the context of the customer's other relationships | Cannot see whether a customer struggling on one product is a strong relationship elsewhere |

Entity Resolution and a single customer view is a bank-wide need, not a compliance-specific one; Customer Service, Marketing, and Collections would each benefit from the same underlying capability. This paper narrows scope deliberately from here on: the needs, priorities, and design decisions that follow are for the financial crime use case only, the AML and Fraud Investigator group. The same pattern is exportable to the other three groups, but designing for them is not what this paper covers.

## Report Customer Needs

Watching investigators work produced a short list of things every investigation does, in roughly this order:

1. Establish who the customer is, including every account, product, and identifier they hold, across lending and deposits.
2. Identify related parties: joint holders, beneficial owners, authorized signers, and anyone sharing an address, phone, or device.
3. See the activity in question in the context of the customer's whole history, not just the product that alerted.
4. Check what has already been looked at: prior alerts, prior cases, prior decisions, and any earlier reports filed.
5. Record findings in a form that a reviewer can follow and a filing can be built from.

Steps one and two consumed most of the two hours. Steps three through five were comparatively quick once the picture existed.

## Cut and Prioritize

Scored on impact against effort, one need stood out.

| Need | Impact on investigation time | Effort | Decision |
| --- | --- | --- | --- |
| Assemble the whole customer, with related parties | Very high: the bulk of the two hours | Medium: new capability and a new view | Build first |
| Cross-product activity timeline | High | Medium | Build first, same release |
| Prior alerts, cases, decisions | Medium | Low: case management already holds them | Build first, cheap |
| Investigator workload analytics | Low for the investigator | Low | Instrument now, surface later |

## List Solutions

Three ways to give an investigator the whole customer were considered.

- **Federated search.** Query each source system live from the case screen and show the results side by side. No merge, no new data store. The investigator still does the matching, but from one screen instead of several logins.
- **Nightly consolidated customer table.** Build a denormalized customer record in the warehouse each night, joined on the identifiers the systems share, and read from it.
- **Entity Resolution-backed case view.** Build Entity Resolution: a resolved identifier and relationship graph linking every representation of a customer across systems. Render the customer, related parties, products, and activity from that single identifier, with links back to source records for evidence.

## Evaluate Trade-offs

| Option | Strengths | Weaknesses |
| --- | --- | --- |
| Federated search | Fastest to build; always current; no new data store to govern | Does not solve the matching problem, only relocates it; several sets of entitlements to honor live; slow when any source is slow; no related-party discovery |
| Nightly consolidated table | Simple joins; familiar warehouse tooling | Joins on shared identifiers miss exactly the cases that matter, where identifiers differ; a day stale; no relationship graph; becomes a second, unofficial source of truth |
| ER-backed case view | Matching is solved once, upstream, and governed; related parties are first-class; one identifier drives the whole page; evidence links preserve provenance | Depends on getting entity matching right; needs stewardship for uncertain matches; a wrong merge is visible to the investigator and must be easy to flag |

The federated option would have shipped soonest and changed the least. The nightly table would have quietly recreated the fragmentation problem in a new place. The ER-backed view was the only option that removed the work rather than moving it.

## Summarize the Recommendation

Build the case view on a resolved entity. One page, six panels, in the order an investigation proceeds.

| Panel | Contents | Design note |
| --- | --- | --- |
| Header | Resolved entity name, stable identifier, aliases, customer since, current risk rating, active flags | Aliases are shown so the investigator understands why a record with a different name belongs here |
| Relationships | Related parties with relationship type and the evidence for the link: shared address, joint account, beneficial ownership | Each link is one click from the related entity's own view |
| Products and accounts | Every account across lending and deposits, with status and open date | Grouped by line of business; closed accounts collapsed by default |
| Activity timeline | Transactions, logins, and profile changes across all products on one timeline, with the alerting activity highlighted | The single most-used panel; filters by product, amount, and counterparty |
| History | Prior alerts, cases, decisions, and filings on this entity and its related parties | Answers 'has anyone looked at this before' without a search |
| Evidence and findings | Links to source records, and the investigator's structured notes | Source records open in place; nothing is copied into the case that cannot be traced back |

Deliberately left off the page: raw source-system records by default, marketing and servicing data unrelated to the decision, and any field the investigator cannot act on. Every panel earns its place by answering one of the five needs.

One design decision worth calling out: a **flag this merge** control in the header. Entity Resolution will occasionally link records that should not be linked. The investigator is the person most likely to notice, so the view makes it one click to send the merge back to the steward, and the case continues on the records the investigator confirms.

## Metrics

| Metric | Why it matters |
| --- | --- |
| Average investigation time, by alert type | The headline outcome: about two hours to about thirty minutes in production |
| Share of investigations requiring a manual lookup in a source system | Whether the page is complete enough; the target trends toward zero |

## Open Questions

- How much of the activity timeline should be pre-analyzed, for example flagging round-number structuring patterns, versus left raw for the investigator to interpret? Pre-analysis speeds the common case and risks anchoring the investigator.
- When confidence on a link is below the auto-merge threshold, should the case view show the candidate link as a suggestion, or hide it until a steward confirms? Showing it helps investigations and risks showing the wrong person.
- What is the right default depth for the relationship panel? One hop is clear; two hops finds networks and clutters the page.

## Status

Implemented in production on top of an Entity Resolution capability spanning originations and servicing systems for lending and deposit products. Average investigation time fell from about two hours to about thirty minutes once investigators started from a resolved entity rather than assembling one.

**About the author.** Praveen Sridharan is a product leader in financial crimes compliance with close to 20 years in technology and financial services, and 10+ years building KYC, sanctions, transaction monitoring, and investigations platforms for global payments and banking. linkedin.com/in/sridharanpraveen
