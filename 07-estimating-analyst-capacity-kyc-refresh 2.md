# How Many Analysts Does It Take to Refresh KYC for 7,000 Customers on a Closed Product?

*An estimation walk-through, top-down and bottom-up, that ends where good estimates often end: with the number telling you not to build the expensive thing.*

**Status:** Worked estimation. All figures are illustrative assumptions, stated explicitly, not measured results.  
**Note:** Original idea was enriched using AI. 

## Abstract

A bank has closed a legacy product to new customers but continues to support about 7,000 existing customers until they close it themselves. Their KYC information must be refreshed, and no refresh capability exists. How many analysts would a manual program need? The top-down estimate lands at about one analyst. The bottom-up estimate, which counts non-response and exceptions, lands at about 1.5, or roughly 2 with quality review and oversight. The sanity check says this is easily staffed. That is the finding: the population is too small, and shrinking, to justify building an automated refresh journey. The estimate supports a lightweight, risk-tiered manual program using existing channels, and it points to the real risks, which are non-responders and the effect of outreach on customer attrition.

## Clarify the Question

- **What is a refresh?** Contacting the customer, collecting updated identity and profile information, verifying it, and documenting the outcome. Not a full re-onboarding.
- **What population?** 7,000 customers holding a product that is closed to new business. No new customers will enter the population. Assume individuals; any small business customers would be sized separately.
- **What happens to the population?** It shrinks. Assume about 10 percent close the product each year, and that about 1,000 of the 7,000 close or exit before their turn comes up, leaving roughly 6,000 refreshes in the window.
- **What window?** 24 months, ordered by risk so the highest-risk customers are refreshed first.
- **What counts as an analyst?** A trained KYC operations analyst with about 6.5 productive hours a day, roughly 500 working days in 24 months (about 3,250 productive hours).

## Top-Down Estimate

| Step | Calculation | Result |
| --- | --- | --- |
| Refreshes required | 7,000 less about 1,000 expected closures | About 6,000 |
| Refreshes per working day | 6,000 / 500 days | About 12 per day |
| Refreshes one analyst completes per day | 6.5 hours at 30 minutes each, less exceptions | About 12 per day |
| Total, top-down | 12 / 12 | **About 1 analyst** |

The 30-minute assumption is the one that drives the result, and it is tested next.

## Bottom-Up Estimate

| Activity per customer | Assumed time | Note |
| --- | --- | --- |
| Prepare and send outreach | 5 minutes | Templates help; confirming contact details on legacy records does not |
| Handle the response | 10 minutes | Matching what came back to the request, chasing gaps |
| Verify updated information | 10 minutes | Against identity vendors, bureaus, or documents supplied |
| Document and close | 8 minutes | The audit record |
| Base case total | **33 minutes** | Close to the top-down assumption |

Two adjustments, both harsher than for a modern product because these customers are long-tenured and less digitally engaged:

- **Non-response.** Assume 50 percent respond to the first outreach. The other half need at least one more attempt, adding about 10 minutes each, or about 5 minutes across the population. The weighted time is about 38 minutes.
- **Exceptions.** Assume 20 percent hit an exception (outdated records, name or address changes, documents that do not verify). Exceptions take about 90 minutes instead of 33, adding about 11 minutes across the population. The average is about 49 minutes.

| Step | Calculation | Result |
| --- | --- | --- |
| Weighted time per customer | 33 minutes plus loading | About 49 minutes |
| Total effort | 6,000 x 49 minutes | About 4,900 hours |
| Analyst capacity over 24 months | 500 days x 6.5 hours | About 3,250 hours per analyst |
| Analysts required | 4,900 / 3,250 | About 1.5 |
| With quality review and supervision (about 20 percent) | | **About 2 analysts** |

## Sanity Check

The two routes land at 1 and 1.5 analysts, and the gap has the same explanation as before: the bottom-up route counts non-response and exceptions. Does the number feel right? Yes. About 12 refreshes a day across the program is a modest queue for a single small team, and the workload is a fraction of a typical KYC operations team. Is it achievable? It can be absorbed by existing staff or covered by one or two contractors or a temporary secondment, with cost not mounting as much as the software development costs. 

That is the answer to the estimation question. A digital self-service refresh journey costs far more to build than it would save for this population, and the population declines every year, so the payback period is questionable. The manual program appears to be the better option. 

## Run-Off: The Effort Declines Over Time

With no new customers, workload follows the population. Assuming 10 percent annual attrition, 7,000 customers become about 6,300 after one year and about 5,700 after two.

Beyond the initial backlog, ongoing refreshes follow risk-based cycles. Assume 10 percent of customers are high risk (annual), 20 percent medium (every 3 years) and 70 percent low (every 5 years). At the start that is about 2,150 refreshes a year, or about 1,750 hours, roughly 1 analyst. It falls in proportion as the population runs off.

| Point in time | Approximate population | Approximate annual refresh effort |
| --- | --- | --- |
| Start | 7,000 | About 1.1 analysts |
| Year 2 | 5,700 | About 0.9 analysts |
| Year 5 | About 4,100 | About 0.6 analysts |

This should be staffed as a shrinking, part-time responsibility, not a standing team. The planning question is who owns the tail once the backlog is cleared.

## What the Estimate Tells You Not to Build

| Where the minutes are | Share of the weighted 49 minutes | Recommended treatment |
| --- | --- | --- |
| Outreach and response handling, including chasing non-responders | About 20 minutes | Reuse existing email, letter, and phone channels with templates. Batch the outreach in risk order. Do not build a new journey. |
| Verification | About 10 minutes | Use vendor checks the bank already licenses. Manual document review for the rest. |
| Exceptions | About 11 minutes of loading | A simple risk-based queue, even a spreadsheet, so high-risk exceptions get analyst time first |
| Documentation | About 8 minutes | Standardize the documentation template. |

The one design worth investing in is sequencing: start with the highest-risk customers, since they are the population an examiner would ask about first and they take priority within the two-year window.

## What to Instrument

| Assumption | Measure instead |
| --- | --- |
| 33 minutes base handle time | Actual handle time per refresh, by channel and by analyst |
| 50 percent first-attempt response | Response rate by channel and by reminder count |
| 20 percent exception rate | Exception rate by cause |
| 10 percent annual attrition | Actual closure rate, and closures that follow refresh outreach |
| 6.5 productive hours | Actual, and what consumes the rest |

## Open Questions

- **Non-responders:** what is the treatment for customers who never respond? Restricting or exiting an account on a closed product may need customer communication and legal review, and it needs to be sized like any other work. This group may be the real cost driver, not the analyst hours.
- **Outreach-driven attrition:** does the refresh request accelerate closures? If customers close rather than respond, the workload falls, is this good?
- **Segmentation:** should the estimate be redone by segment if the customers differ in tenure, risk rating, or contact-data quality?
- **Ownership of the tail:** who owns refreshes once the backlog is cleared and the population is a few thousand and falling?

## Status

The figures in this paper are illustrative assumptions chosen to show the method, not measurements from any program. The method is the same as for a large-scale program: estimate the manual effort honestly, then let the result decide what to build. For 7,000 customers on a closed product it says staff it modestly, sequence by risk, and spend the effort on non-responders and on tracking attrition. For a bigger customer base, the solution would be to automate. 
