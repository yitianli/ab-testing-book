# Ranking Experiments and Interleaving

Many product experiments compare two different experiences. A checkout experiment may show one page to control users and another page to treatment users. A pricing experiment may show different prices to different markets.

Ranking experiments often compare something more specific:

> Given the same user context, which ranking algorithm orders the items better?

A ranking system may order search results, videos, products, creators, jobs, ads, restaurants, or posts in a feed. Two rankers may use the same page and the same candidate items, but arrange those items differently.

For example:

- Ranker A puts product X first and product Y second.
- Ranker B puts product Y first and product X second.

Sometimes the immediate question is not whether the whole product experience should change. The immediate question is:

> Which ranker makes better ordering decisions?

This chapter focuses on interleaving, a method designed for fast ranker comparison.

Interleaving does not prove that a ranker improves long-term retention, satisfaction, fairness, marketplace balance, or ecosystem health. Those questions still require careful metrics, guardrails, and often a full A/B test.

Its main advantage is narrower:

> Interleaving can compare two rankers faster and with higher sensitivity than a standard A/B test.

## Why Ranker A/B Tests Can Be Slow

A normal A/B test can compare two rankers:

- Control users see ranker A.
- Treatment users see ranker B.
- The analyst compares clicks, watch time, purchases, or another metric between groups.

This design is valid and often necessary for final launch decisions. But it can be inefficient when the team is still iterating on ranking models.

The reason is noise.

Suppose a video app compares two recommendation rankers. Some users watch for hours, while others open the app for one minute. Some users are loyal, some are new, some have strong preferences, and some are just browsing. Even with randomization, the treatment-control comparison contains a lot of user-level variation.

Search and recommendation requests also vary sharply. Some queries are easy, some are ambiguous, and some have no good answer. Some recommendation sessions happen when the user has strong intent, while others happen when the user is casually scrolling.

Because of this noise, a better ranker may need a large amount of traffic before a standard A/B test can detect the improvement. The ranking change may be real, but the signal can be small relative to natural variation across users and requests.

Interleaving is designed for this sensitivity problem.

## The Basic Idea of Interleaving

Interleaving turns a ranker comparison into a paired comparison.

Instead of showing ranker A to one group of users and ranker B to another group, the system asks both rankers to produce results for the same request. It then mixes their results into one list shown to the user.

Suppose two rankers return:

| Position | Ranker A | Ranker B |
|---:|---|---|
| 1 | A1 | B1 |
| 2 | A2 | B2 |
| 3 | A3 | B3 |
| 4 | A4 | B4 |

An interleaved list may look like this:

| Display Position | Item | Contributed By |
|---:|---|---|
| 1 | A1 | A |
| 2 | B1 | B |
| 3 | A2 | A |
| 4 | B2 | B |
| 5 | B3 | B |
| 6 | A3 | A |

The user sees one normal-looking result page. Behind the scenes, each item is labeled by the ranker that contributed it.

If the user clicks $B1$ and $B2$, ranker B gets credit. If the user clicks $A1$ and $A2$, ranker A gets credit. If the user clicks one item from each ranker, the request may be treated as a tie, depending on the credit rule.

Across many requests, the analyst checks whether one ranker wins more often.

One common construction is team draft interleaving. It mimics teams picking players in sports: ranker A and ranker B take turns choosing their highest-ranked available items for the final displayed list. Which ranker picks first is randomly chosen, so neither ranker always gets the top position.

The key advantage is that both rankers are evaluated under the same user, query, device, time, and immediate intent. Interleaving turns a noisy between-user comparison into a cleaner within-context comparison.

## Credit Assignment

Interleaving needs a credit rule.

The simplest click-based rule counts actions on items contributed by each ranker:

$$
C_{i,A} = \text{clicks on items contributed by ranker A in request } i
$$

$$
C_{i,B} = \text{clicks on items contributed by ranker B in request } i
$$

Then define the credit difference:

$$
D_i = C_{i,B} - C_{i,A}
$$

If $D_i > 0$, ranker B wins request $i$.

If $D_i < 0$, ranker A wins request $i$.

If $D_i = 0$, the request is a tie.

Credit does not have to be based only on clicks. Depending on the product, it may use:

- Clicks
- Long clicks
- Watch starts
- Watch time
- Purchases
- Add-to-cart events
- Saves
- Likes
- Query reformulation
- Skips, hides, or dislikes as negative credit

The credit rule should match the product question. For search, a long click may be more meaningful than a short click: a short click means the user quickly returns after clicking, while a long click means the user stays longer or does not return immediately. For video recommendations, watch time or completion may be more meaningful than a play start. For e-commerce ranking, purchase or add-to-cart may be more meaningful than a product click.

But richer credit rules also create risk. If the credit metric is noisy, delayed, or too narrow, interleaving may favor a ranker that wins the credit rule but not the real product objective.

## Duplicates and Shared Items

Credit assignment is simple when every displayed item clearly belongs to only one ranker. In practice, ranking algorithms often return overlapping items.

For example:

| Position | Ranker A | Ranker B |
|---:|---|---|
| 1 | Product X | Product Y |
| 2 | Product Y | Product X |
| 3 | Product Z | Product W |

Both rankers like Product X and Product Y, but in different orders.

Interleaving needs a rule for duplicates. Usually, an item appears only once in the final list. If ranker A already added Product X, ranker B cannot add the same Product X again.

In team draft interleaving, credit is usually deterministic: the ranker that added the item to the displayed list gets the credit.

Other methods, such as probabilistic interleaving and multileaving, use softer credit rules. Instead of saying that a clicked item belongs completely to ranker A or completely to ranker B, they ask:

> Given the displayed list, which ranker was more likely to have produced the clicked item in that position?

If both rankers placed Product X near the top, then a click on Product X may not be strong evidence for either ranker. But if ranker A placed Product X first and ranker B placed it tenth, then a click on Product X gives more evidence to ranker A.

The intuition is that probabilistic credit uses more information from the two original rankings, especially when the rankers return many of the same items.

## Formal Inference With Interleaving

Interleaving produces paired comparison data.

For each request, session, query, or user, the analyst observes whether ranker A won, ranker B won, or the result was tied.

The null hypothesis is:

> Users do not prefer ranker A or ranker B.

In a win-loss framework, after excluding ties:

$$
H_0: P(\text{B wins}) = P(\text{A wins}) = 0.5
$$

Suppose the experiment produces:

| Result | Count |
|---|---:|
| B wins | 5,800 |
| A wins | 4,200 |
| Ties | 3,000 |

After excluding ties, there are $10,000$ non-tied comparisons.

Under the null hypothesis:

$$
W_B \sim \text{Binomial}(10000, 0.5)
$$

where $W_B$ is the number of B wins.

If the binomial p-value is small, ranker B wins significantly more often than expected by chance. This is essentially a sign test.

Another approach keeps the numeric credit difference:

$$
D_i = C_{i,B} - C_{i,A}
$$

and tests:

$$
H_0: E[D_i] = 0
$$

This can be done with:

- A paired t-test on $D_i$
- A bootstrap confidence interval
- A permutation or randomization test
- Cluster-robust inference when repeated observations come from the same user

The sign test is simple and robust. The credit-difference test can use more information, especially when a request can generate multiple actions or weighted engagement.

## The Unit of Inference

Interleaving is often logged at the request level, but requests from the same user are not independent.

If one highly active user sends hundreds of requests, treating all requests as independent can make the result look more precise than it really is.

A safer approach is to aggregate credit at the user level:

$$
D_u = \sum_{i \in u} D_i
$$

Then test:

$$
H_0: E[D_u] = 0
$$

This respects the fact that users, not requests, may be the independent experimental units.

Another option is a user-level bootstrap:

1. Sample users with replacement.
2. Keep all interleaved requests for each sampled user.
3. Recompute the win rate or mean credit difference.
4. Repeat many times.
5. Build a confidence interval from the bootstrap distribution.

In search, the analyst may also worry about query-level dependence. Some queries appear many times and may dominate the result. In that case, it is useful to check robustness by aggregating at both the user level and the query level.

The principle is the same as in ratio metrics and clustered experiments:

> The logging unit is not always the inference unit.

## What Interleaving Estimates

Interleaving estimates relative preference between rankers under the interleaving design.

It can support statements like:

> Ranker B receives more user credit than ranker A in interleaved comparisons.

Or:

> Among non-tied interleaved requests, ranker B wins 58% of comparisons.

It does not directly estimate:

> Ranker B increases retention by 2%.

Or:

> Ranker B increases GMV by 3%.

The reason is that interleaving shows users a mixed list. The observed behavior comes from a list partly produced by ranker A and partly produced by ranker B, not from a world where ranker B fully controls the page.

So interleaving is good at answering:

> Which ranker appears better for ordering results in this immediate context?

It is weaker at answering:

> What is the full product impact if we launch this ranker to all users?

For that reason, interleaving is usually a fast screening method, not the final launch decision.

## Guardrails in Interleaving

Guardrails in interleaving need a different interpretation from guardrails in a standard A/B test.

In a standard A/B test, the comparison is usually:

> Compare the guardrail metric between users assigned to ranker A and users assigned to ranker B.

In interleaving, the user sees a mixed list. There is no pure "ranker A user experience" or pure "ranker B user experience" inside the interleaved request. Because of that, not every guardrail can be compared as group A versus group B.

It is useful to separate three cases.

First, some guardrails can be attributed to contributed items:

- Negative feedback on an item
- Content report on an item
- Hide or "not interested" action
- Return or refund associated with a clicked product
- Very short watch after clicking a contributed video

For these outcomes, the analysis can use the same logic as credit assignment. If a reported item was contributed by ranker B, ranker B receives negative credit. The analyst can then compare whether ranker B receives more negative credit than ranker A across interleaved requests.

Second, some guardrails belong to the mixed page or session, not to one ranker:

- Page load latency
- Empty result rate
- Abandonment rate
- Query reformulation rate
- Session-level satisfaction
- 7-day retention

These cannot be cleanly attributed to ranker A or ranker B within the same interleaved list. For example, if the user abandons the page, was it because of ranker A's items, ranker B's items, the mixture, latency, or something else? The interleaving data alone usually cannot answer that.

For these guardrails, the practical check is usually at the interleaving-bucket level:

> Does traffic exposed to interleaving look healthy compared with normal production traffic?

This protects users from a bad mixed-list experience, but it does not tell us whether ranker B would be safer than ranker A under full rollout.

Third, some guardrails must be checked in the final A/B test. Long-term retention, marketplace concentration, creator diversity, seller fairness, and ecosystem health are usually full-rollout questions. Interleaving may provide early warning signals, but the final comparison should be:

> Users fully served by ranker B versus users fully served by ranker A.

So the short version is:

> In interleaving, item-level guardrails can sometimes be credited to rankers, but page-level and long-term guardrails usually cannot. Those should be monitored for the interleaving bucket and then formally tested in a full A/B experiment.

## Interleaving Versus A/B Testing

Interleaving and A/B testing answer different questions.

| Method | Main Question | Typical Unit | Strength | Limitation |
|---|---|---|---|---|
| A/B test | What happens if users receive version B instead of version A? | User, session, market | Measures full product impact | May need large sample and long duration |
| Interleaving | Which ranker do users prefer within the same context? | Request, session, user | Very sensitive for ranker comparison | Does not measure full rollout impact |

For ranking systems, a common workflow is:

1. Use offline evaluation to remove clearly bad candidates.
2. Use interleaving to compare promising rankers quickly.
3. Use a full A/B test to validate business metrics and guardrails.
4. Use long-term holdouts if the change may affect retention, marketplace balance, creator ecosystem, or user trust.

This workflow is especially useful when teams iterate on many ranking models. Interleaving helps avoid spending full A/B test traffic on every candidate.

Parks, Aurisset, and Ramm (2017) describe how Netflix used interleaving to evaluate personalization algorithms faster before moving promising ideas into larger online experiments. The practical lesson is not that interleaving replaces A/B testing. The lesson is that interleaving can make the experimentation pipeline faster by acting as an early ranking-quality filter.

## Beyond Team Draft Interleaving

Team draft interleaving is not the only method.

Other approaches include:

- Balanced interleaving
- Probabilistic interleaving
- Multileaving for more than two rankers

Balanced interleaving tries to place results from both rankers in a balanced way while respecting their rankings.

Probabilistic interleaving samples rankings from probability distributions induced by each ranker. Instead of asking only which ranker contributed an item, it can estimate how likely each ranker was to generate the observed list.

Multileaving extends the idea to compare more than two rankers at once.

For most product analysts, the details of each algorithm are less important than the design principles:

- The displayed list should be fair enough for comparison.
- Contribution labels and credit rules should be logged clearly.
- The inference method should match the interleaving method.
- The result should be interpreted as relative ranking preference.

## Product Example: Search Ranking

Suppose an e-commerce marketplace is testing a new search ranking model.

The old model ranks products mainly by short-term purchase probability. The new model also considers return rate, seller quality, and repeat-purchase behavior.

The team could run a normal A/B test:

- Control users see old search ranking.
- Treatment users see new search ranking.
- The team measures clicks, add-to-cart, purchases, refunds, and retention.

This is useful, but it may take time. Search traffic is noisy, and many queries have low purchase intent.

As an earlier screening step, the team can run interleaving:

1. For each search query, generate results from the old and new rankers.
2. Build an interleaved result list.
3. Randomize which ranker picks first.
4. Log which ranker contributed each product.
5. Assign credit based on clicks, add-to-cart, purchases, or long clicks.
6. Test whether the new ranker wins significantly more often.

If the new ranker wins interleaving comparisons and does not create obvious item-level guardrail concerns, the team may promote it to a full A/B test.

In the full A/B test, the team would measure broader outcomes:

- Conversion rate
- GMV per search
- Refund rate
- Repeat purchase
- Seller concentration
- User retention
- Search reformulation

The interleaving result tells the team whether users prefer the new ranking in direct comparison. The A/B test tells the team whether launching the new ranking improves the product.

## Product Example: Video Recommendations

Consider a video app testing a new recommendation ranker.

The current ranker optimizes short-term watch time. The new ranker tries to improve satisfaction by considering completion rate, dislikes, surveys, and long-term retention signals.

Interleaving can compare the two rankers within the same recommendation request. The final row or feed contains videos from both rankers, and user actions are credited back to the contributing ranker.

Possible credit signals include:

- Play starts
- Watch time
- Completion
- Likes
- Dislikes as negative credit
- Skips as negative credit
- "Not interested" feedback as negative credit

The team should be careful not to define the credit rule too narrowly. If the rule is only watch time, the method may favor videos that maximize time spent but hurt satisfaction.

So the interleaving credit rule should match the ranking objective as closely as possible, and the final A/B test should still check broader metrics:

- 7-day retention
- Satisfaction survey scores
- Reported content
- Negative feedback
- Creator diversity
- Long-term watch behavior

This is why interleaving is powerful but incomplete. It can speed up ranker comparison, but it cannot fully answer the long-term product question.

## Common Mistakes

The first mistake is treating interleaving as a full replacement for A/B testing. Interleaving estimates relative ranker preference in a mixed list. It does not directly estimate full rollout impact.

The second mistake is using a credit metric that does not match the product objective. If the product cares about successful purchases, pure clicks may be too shallow. If the product cares about satisfaction, pure watch time may be too narrow.

The third mistake is ignoring dependence. Request-level data can be highly correlated within users, queries, markets, or sessions. Inference should account for clustering or use aggregation at a credible independent unit.

The fourth mistake is allowing one ranker to receive better positions systematically. The interleaving method should give both rankers fair opportunities in visible positions.

The fifth mistake is ignoring ties. Many interleaved requests produce no clicks or equal credit. The analysis should state how ties are handled.

The sixth mistake is forgetting that the mixed list is its own product experience. If the interleaved list is unnatural, confusing, or lower quality than either original list, the result may not generalize to a full launch.

## Practical Checklist

Before using interleaving, the team should answer:

1. Is the treatment really a ranking change?
2. Can both rankers generate lists for the same request?
3. Will the mixed list still be a reasonable user experience?
4. How will the interleaving method balance position opportunities?
5. How will duplicate items be handled?
6. What user actions will assign credit?
7. What is the unit of inference: request, session, query, user, or something else?
8. How will repeated users or repeated queries be handled?
9. Which item-level guardrails can be credited to rankers?
10. Which page-level or long-term guardrails require A/B validation?
11. What result is strong enough to move the ranker to a full A/B test?
12. What final business metrics still require A/B validation?

This checklist keeps interleaving in its proper role: a fast and sensitive ranker-comparison tool.

## Key Takeaways

Ranking A/B tests can be slow because user behavior and request intent are noisy.

Interleaving compares two rankers inside the same user context by mixing their results into one displayed list.

Team draft interleaving is a simple method where rankers take turns choosing items, with randomized first pick.

Credit assignment turns user actions into ranker wins, losses, ties, or credit differences.

Formal inference often treats each request, session, query, or user as a paired comparison. Analysts can use sign tests, binomial tests, credit-difference tests, bootstrap confidence intervals, or cluster-aware methods.

The result of interleaving is a relative preference measure, not a direct estimate of full business impact.

Interleaving is most useful as a fast screening method before a full A/B test.

## References

- Chapelle, Olivier, Thorsten Joachims, Filip Radlinski, and Yisong Yue. "Large-Scale Validation and Analysis of Interleaved Search Evaluation." *ACM Transactions on Information Systems* 30, no. 1 (2012): 6:1-6:41. [https://doi.org/10.1145/2094072.2094078](https://doi.org/10.1145/2094072.2094078).
- Parks, Joshua, Juliette Aurisset, and Michael Ramm. "Innovating Faster on Personalization Algorithms at Netflix Using Interleaving." Netflix Technology Blog, November 29, 2017. [https://netflixtechblog.com/interleaving-in-online-experiments-at-netflix-a04ee392ec55](https://netflixtechblog.com/interleaving-in-online-experiments-at-netflix-a04ee392ec55).
- Radlinski, Filip, Madhu Kurup, and Thorsten Joachims. "How Does Clickthrough Data Reflect Retrieval Quality?" In *Proceedings of the 17th ACM Conference on Information and Knowledge Management*, 43-52. 2008. [https://doi.org/10.1145/1458082.1458092](https://doi.org/10.1145/1458082.1458092).
