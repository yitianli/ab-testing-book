# From Result to Decision

An experiment result is not a decision.

It is evidence for a decision.

This distinction matters because product experiments rarely produce a perfectly clean answer. A primary metric may improve while a guardrail worsens. A result may be statistically significant but too small to matter. A treatment may work well for new users but not for existing users. A short-term metric may improve while the long-term effect is still uncertain.

The question at the end of an experiment is not only:

> Is the result statistically significant?

The better question is:

> Given the full pattern of evidence, what should the team do next?

This chapter is about that final step: translating experiment evidence into a launch, no-launch, iterate, or rollout decision.

## The Decision Comes After Trust

Before making a decision, the team should first know whether the experiment result is trustworthy.

Chapter 14 discussed experiment quality checks: sample ratio mismatch, logging validation, exposure counts, metric construction, missing data, bots, outliers, and ramp issues. If those checks fail badly, the decision should usually be:

> Fix the experiment before interpreting the treatment effect.

This chapter starts after that point. Assume the team has checked the experiment and believes the result is credible enough to interpret.

Now the decision problem begins.

The analyst should move through three questions:

1. What did the experiment estimate?
2. Is the estimated effect valuable enough to matter?
3. Is the tradeoff acceptable?

The first question is statistical. The second is business judgment. The third is product judgment.

Good launch decisions need all three.

## Statistical Significance Is Not the Decision

Statistical significance answers a narrow question:

> If the true treatment effect were zero, how surprising would this result be?

It does not answer:

- Is the effect large enough to matter?
- Does the benefit justify the cost?
- Are guardrails healthy?
- Is the result stable across important segments?
- Is the effect likely to persist?
- Should the feature launch to everyone, some users, or no one?

A small effect can be statistically significant if the experiment has enough traffic.

For example:

| Metric | Result |
|---|---:|
| Purchase conversion rate | +0.03 percentage points |
| p-value | 0.01 |
| Engineering maintenance cost | High |
| Customer support risk | Medium |

The effect is statistically significant, but the decision may still be no launch if the gain is too small relative to operational cost or product complexity.

The reverse can also happen. A result may not reach statistical significance but still be directionally promising, especially if the experiment is underpowered or the effect is important enough to justify more testing.

The decision should not be:

> Significant means launch. Not significant means stop.

The decision should be:

> What action is justified by the size, uncertainty, cost, risk, and strategic importance of the effect?

## Practical Significance

Practical significance asks whether the effect is large enough to matter.

Suppose a marketplace experiment increases order conversion by:

$$
\widehat{\tau} = 0.05\%
$$

This may be meaningful for a platform with millions of daily orders. It may be meaningless for a small product surface with low traffic.

The same statistical effect can imply different decisions depending on:

- Traffic volume
- Revenue per conversion
- User value
- Engineering cost
- Maintenance cost
- Operational complexity
- Strategic importance
- Risk to trust, safety, or fairness

A useful habit is to translate metric lift into business impact.

For example:

$$
\text{Incremental value}
=
\text{Traffic}
\times
\text{Baseline conversion}
\times
\text{Relative lift}
\times
\text{Value per conversion}
$$

This calculation does not need to be perfect. Its purpose is to turn an abstract metric movement into a decision-relevant estimate.

For example:

| Quantity | Value |
|---|---:|
| Monthly eligible users | 10,000,000 |
| Baseline conversion | 4% |
| Relative lift | 1% |
| Value per conversion | $20 |

Then:

$$
10{,}000{,}000
\times
0.04
\times
0.01
\times
20
=
80{,}000
$$

The estimated monthly incremental value is about $80,000 before considering costs and risks.

This kind of calculation helps the team decide whether the effect is worth launching, iterating on, or ignoring.

## Confidence Intervals and Decision Risk

A point estimate is only one number. A confidence interval shows the range of effects that are reasonably compatible with the data.

Suppose an experiment estimates:

$$
\widehat{\tau} = +2.0\%
$$

with a 95% confidence interval:

$$
[+0.2\%, +3.8\%]
$$

This result is positive and statistically significant, but the lower bound is small. If implementation cost is low and guardrails are healthy, launch may be reasonable. If the feature is expensive or risky, the team may want stronger evidence.

Now suppose the estimate is:

$$
\widehat{\tau} = +2.0\%
$$

with a 95% confidence interval:

$$
[-1.5\%, +5.5\%]
$$

The point estimate is the same, but uncertainty is much larger. This result does not rule out harm.

The decision should consider the downside risk:

> How bad would it be if the true effect were near the lower end of the interval?

This is especially important for high-risk launches. A small chance of a serious guardrail violation may matter more than a moderate chance of a primary metric lift.

## Metric Hierarchy

A decision is easier when the experiment has a clear metric hierarchy.

One useful structure is:

| Metric Type | Role |
|---|---|
| Primary metric | Main metric used to judge success |
| Secondary metrics | Explain mechanism and supporting behavior |
| Guardrail metrics | Protect user experience, trust, quality, cost, or ecosystem health |
| Diagnostic metrics | Help debug why the result happened |
| Long-term metrics | Validate whether the short-term decision remains good |

The primary metric should answer the main business question.

Secondary metrics should explain how the treatment worked.

Guardrails should define what the team is not willing to sacrifice.

Diagnostic metrics should help interpret the result, but they should not quietly become new success metrics after the fact.

The launch decision should follow the hierarchy. If the primary metric is flat but a random secondary metric improves, the team should be careful about declaring success. If the primary metric improves but a critical guardrail worsens, the team should not ignore the guardrail.

The hierarchy prevents the team from choosing whichever metric tells the most convenient story.

## A Simple Decision Matrix

A practical launch decision often falls into one of several patterns.

| Primary Metric | Guardrails | Decision Pattern |
|---|---|---|
| Improves clearly | Healthy | Launch or ramp |
| Improves clearly | Critical guardrail worsens | Do not broadly launch; investigate or redesign |
| Flat | Healthy | Usually no launch unless there is strategic value or cost reduction |
| Flat | Guardrail improves | Consider launch if guardrail is strategically important |
| Worsens | Healthy | Usually no launch |
| Mixed or uncertain | Mixed or uncertain | Segment, rerun, extend, or iterate |

This table is not a mechanical rule. It is a starting point.

The real decision depends on effect size, uncertainty, costs, reversibility, user risk, and strategic context.

## Guardrail Violations

Guardrails are not decorative.

They are constraints on what the team is willing to trade away.

For example, a video recommendation experiment may show:

| Metric | Result |
|---|---:|
| Watch time per user | +4%, statistically significant |
| Like rate | -2% |
| 7-day retention | No significant change |
| Reported content | +8%, statistically significant |

This is not a clean win.

The primary metric improves, but reported content is a trust and safety guardrail. The team should not say:

> Watch time improved, so launch.

A better interpretation is:

> The model increases short-term engagement, but it may also increase exposure to problematic content. We should investigate what types of reports increased, which content categories or user segments are affected, and whether safety constraints can reduce the harm.

The decision may be:

- Do not launch broadly.
- Launch only in low-risk segments if the guardrail is healthy there.
- Add safety filters or ranking constraints.
- Rerun the experiment after redesign.
- Use a longer test to measure retention and satisfaction.

The central idea is:

> A guardrail violation changes the decision, even when the primary metric wins.

## Metric Tradeoffs

Many experiments produce tradeoffs rather than simple wins.

Consider a checkout experiment:

| Metric | Result |
|---|---:|
| Purchase conversion rate | +3%, statistically significant |
| Average order value | -2%, statistically significant |
| Revenue per visitor | No significant change |
| Refund rate | No significant change |

The treatment increases conversion but lowers average order value.

This does not automatically mean launch or no launch. The team should decompose the result.

Possible explanations include:

- The new checkout flow helps low-intent users complete smaller purchases.
- The treatment shifts users toward cheaper products.
- Discounts or payment options encourage smaller orders.
- High-value users are unaffected, while low-value users convert more.
- The change improves first purchase activation but not immediate revenue.

The decision metric may be revenue per visitor, contribution margin per visitor, first-time buyer activation, or long-term buyer value, depending on the business goal.

If revenue per visitor is flat but first-time buyer activation improves, a launch may still be reasonable for new users. If the company cares mainly about short-term revenue, the result may not justify launch.

Tradeoffs require the team to return to the original business goal.

## Segment-Level Decisions

Sometimes the average treatment effect hides important differences.

For example:

| Segment | Treatment Effect |
|---|---:|
| New users | +5% activation |
| Existing users | 0% |
| High-value users | -2% revenue |
| Low-value users | +4% revenue |

A single global launch decision may be too crude.

Possible decisions include:

- Launch to new users only.
- Exclude high-value users.
- Launch in countries where guardrails are healthy.
- Keep the feature behind an eligibility rule.
- Run a follow-up experiment focused on the promising segment.

Segment decisions should be made carefully. If the segment was pre-specified, the evidence is stronger. If the segment was discovered after searching many cuts of the data, it should be treated as exploratory.

A useful rule is:

> Segment-based launch is more credible when the segment has a clear product rationale, enough sample size, and healthy guardrails.

If the segment result is surprising or discovered after heavy exploration, a follow-up test is usually better than immediate targeting.

## Short-Term Versus Long-Term Effects

Some experiments have short-term effects that do not persist.

Examples include:

- Novelty effects
- Learning effects
- Delayed purchases
- Delayed churn
- Creator or seller adaptation
- Marketplace equilibrium changes

Suppose a recommendation experiment shows:

| Metric | Week 1 | Week 4 |
|---|---:|---:|
| Click-through rate | +5% | +1% |
| Watch time | +4% | +0.5% |
| Negative feedback | +2% | +6% |

The week-1 result looks promising. The week-4 result is much less attractive.

For product changes that may affect habits, satisfaction, marketplace balance, or user trust, the team should ask:

> Is the observed lift likely to persist?

If the answer is uncertain, the decision may be:

- Extend the experiment.
- Use a staged rollout.
- Keep a long-term holdout.
- Launch only if leading metrics and guardrails remain stable.

Not every experiment needs long-term validation. A copy change on a low-risk page may not need weeks of follow-up. But changes to ranking, pricing, notifications, incentives, or marketplace allocation often do.

## Reversibility and Risk

The same evidence can justify different decisions depending on how reversible the launch is.

If a feature is easy to roll back, affects a low-risk surface, and has healthy guardrails, the team may accept more uncertainty.

If a feature affects trust, safety, payments, privacy, pricing, creator income, or marketplace allocation, the team should require stronger evidence.

A useful distinction is:

| Launch Type | Risk Level | Evidence Standard |
|---|---|---|
| Easy rollback, low user risk | Lower | Directionally positive may be acceptable |
| Hard rollback or high user risk | Higher | Strong evidence and healthy guardrails needed |
| Irreversible user or ecosystem impact | Very high | Extra validation, staged rollout, and monitoring needed |

Decision-making is not only about expected value. It is also about downside risk.

## Rollout Strategy

Launch does not have to mean 100% rollout immediately.

Common options include:

- No launch
- Continue experiment
- Rerun experiment
- Iterate and retest
- Launch to a segment
- Ramp gradually
- Launch with monitoring
- Launch with a long-term holdout
- Roll back

A gradual rollout is useful when the result is positive but risk remains.

For example:

1. Ramp from 5% to 10%.
2. Monitor guardrails.
3. Ramp to 25%.
4. Monitor operational metrics and user feedback.
5. Ramp to 50%.
6. Keep a small holdout if long-term effects matter.

The ramp plan should define stopping rules before rollout:

- What metric movement triggers a pause?
- What guardrail movement triggers rollback?
- Who owns the decision?
- How often will the rollout be reviewed?
- What population remains as a holdout?

Rollout is part of the experiment decision, not an afterthought.

## When To Rerun or Extend

Sometimes the correct decision is not launch or no launch. It is to gather better evidence.

Rerun or extend the experiment when:

- The result is underpowered.
- The confidence interval includes both meaningful benefit and meaningful harm.
- The experiment missed important seasonal patterns.
- A logging or ramp issue affected part of the test.
- A key guardrail is delayed.
- A strategic segment needs more sample.
- The result is surprising and high-stakes.

But rerunning should not become a way to keep testing until the team gets the answer it wants.

The team should state what the new experiment is meant to clarify:

> We are rerunning because the confidence interval is too wide to rule out harm.

Or:

> We are extending because the primary metric is immediate, but the guardrail is delayed.

The goal is better evidence, not repeated chances at significance.

## Product Example: Onboarding Flow

Suppose a subscription app tests a new onboarding flow.

The experiment shows:

| Metric | Result |
|---|---:|
| Signup completion rate | +8%, statistically significant |
| Trial start rate | +5%, statistically significant |
| Paid conversion after trial | -6%, statistically significant |
| 30-day retention among paid users | No significant change |

At first glance, the onboarding flow looks successful because more users sign up and start trials.

But the downstream paid conversion rate falls. This suggests that the new flow may be attracting or encouraging more low-intent users into trials. The top of the funnel improves, but the quality of trial starts declines.

The decision should depend on the full funnel, not on each step in isolation.

For example, suppose 10,000 users enter onboarding.

In the control group:

- 40% start a trial, producing 4,000 trials.
- 20% of trials convert to paid, producing 800 paid subscribers.

In the treatment group:

- Trial starts increase by 5%, so the trial start rate becomes 42%.
- Paid conversion after trial decreases by 6%, so the paid conversion rate becomes 18.8%.
- The result is $10{,}000 \times 42\% \times 18.8\%$, or about 790 paid subscribers.

In this simplified example, the new onboarding flow creates more trials but slightly fewer paid subscribers. The top of the funnel improved, but the end outcome did not.

The opposite result is also possible. If the increase in trial starts is large enough, total paid subscribers may still increase even though the conditional paid conversion rate falls. This is why funnel metrics should be interpreted together, not as separate wins and losses.

The team should also check:

- Revenue per eligible visitor
- Trial cost or subsidy cost
- Cancellation or refund behavior
- Support tickets
- Payment failure
- Acquisition channel mix
- Segment-level effects for new versus returning users

If total paid subscribers and revenue per visitor increase without higher cost or worse retention, launch may be reasonable. If the extra trial starts do not convert and create support, payment, or subsidy cost, the team should iterate.

The right conclusion is:

> The new onboarding flow improves early funnel metrics, but the decision depends on whether it increases valuable paid users, not just signups and trials.

This example also shows why the denominator matters. "Paid conversion after trial" is conditional on users who started a trial. If treatment changes who enters that denominator, the metric can move because the product experience changed, because the composition of trial users changed, or both.

## Product Example: E-Commerce Ranking

Suppose an e-commerce recommendation experiment shows:

| Metric | Result |
|---|---:|
| Repeat-purchase GMV | +5%, statistically significant |
| GMV per thousand impressions | +3%, statistically significant |
| Total GMV per user | +1%, not statistically significant |
| Negative feedback | No significant change |
| Refund rate | No significant change |
| Seller concentration | +6% |

The treatment appears to improve repeat-purchase behavior and monetization efficiency. But total GMV per user is uncertain, and seller concentration increases.

The decision should not be based only on repeat-purchase GMV.

The team should ask:

- Is the total GMV confidence interval compatible with meaningful lift?
- Is seller concentration a strategic concern?
- Is the effect stronger in categories where repeat purchase is naturally important?
- Are smaller sellers losing exposure?
- Does the treatment-control GMV gap widen over time?

A reasonable decision may be a targeted rollout:

- Increase the signal weight in high-repeat categories.
- Limit the weight in categories where concentration worsens.
- Monitor seller diversity and refund rate.
- Keep a long-term holdout to validate LTV.

This is not a simple yes or no. It is a launch strategy shaped by the evidence.

## The Decision Memo

A good experiment decision should be written down.

The memo does not need to be long, but it should make the reasoning explicit.

A useful structure is:

1. Decision: launch, no launch, iterate, extend, rerun, or targeted rollout.
2. Experiment goal and hypothesis.
3. Primary metric result with uncertainty.
4. Secondary metric pattern.
5. Guardrail results.
6. Experiment quality checks.
7. Segment findings.
8. Practical significance and expected impact.
9. Risks and unresolved questions.
10. Rollout or follow-up plan.

The memo should separate evidence from judgment.

Evidence:

> Purchase conversion increased by 3.2% with a 95% confidence interval of $[1.1\%, 5.3\%]$.

Judgment:

> Because refund rate and payment errors were stable, and the estimated monthly value is larger than maintenance cost, we recommend a gradual rollout.

This separation makes the decision easier to review later.

## Common Mistakes

The first mistake is treating statistical significance as the launch rule. Significance is evidence, not a decision.

The second mistake is ignoring practical significance. A tiny statistically significant lift may not justify cost, complexity, or risk.

The third mistake is declaring success based on a secondary metric after the primary metric fails.

The fourth mistake is treating guardrails as optional. A critical guardrail violation can override a primary metric win.

The fifth mistake is launching globally when the evidence supports only a segment-level decision.

The sixth mistake is ignoring uncertainty. A positive point estimate with a wide confidence interval may still include meaningful harm.

The seventh mistake is using reruns as a way to search for significance rather than to answer a clearly stated uncertainty.

The eighth mistake is launching without a monitoring and rollback plan.

## Practical Checklist

Before making a launch decision, the team should answer:

1. Is the experiment data trustworthy enough to interpret?
2. What was the pre-specified primary metric?
3. Did the primary metric improve, worsen, or remain inconclusive?
4. Is the effect practically meaningful?
5. What does the confidence interval imply about upside and downside risk?
6. Did secondary metrics support the expected mechanism?
7. Did any guardrail worsen?
8. Are delayed or long-term metrics needed before launch?
9. Are there credible segment differences?
10. Is the launch reversible?
11. What rollout strategy matches the risk?
12. What monitoring or holdout is needed after launch?

This checklist helps keep the decision disciplined.

## Key Takeaways

An experiment result is evidence for a decision, not the decision itself.

Statistical significance does not imply practical significance.

A good decision considers effect size, uncertainty, metric hierarchy, guardrails, business value, risk, reversibility, and long-term effects.

Mixed results are normal. The right response may be targeted rollout, iteration, extension, or rerun rather than simple launch or no launch.

Guardrails are real constraints. A primary metric win with a serious guardrail violation is not a clean win.

Rollout strategy is part of the decision. Teams should define monitoring, stopping rules, and rollback plans before scaling a change.

The best experiment decisions are explicit: they state what the data showed, what judgment was applied, and what action follows.
