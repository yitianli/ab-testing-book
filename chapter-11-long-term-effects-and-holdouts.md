# Long-Term Effects and Holdouts

Many experiments are designed to answer a short-term question:

> Did the treatment improve the metric during the experiment window?

That question is useful, but it is not always enough.

Some product changes create effects that take time to appear. Others create short-term excitement that fades. Some changes look neutral in the first week but become valuable after users learn how to use them. Some changes increase immediate conversion but bring in lower-quality users who churn later.

This chapter is about the gap between short-term experiment results and long-term product value.

It covers:

1. Delayed metrics and long-term outcomes.
2. Novelty effects and learning effects.
3. How to detect whether effects change over time.
4. How to design long-term holdouts.
5. How to use long-term evidence in launch decisions.

The central idea is simple:

> A treatment effect is not only a number. It can also be a time path.

## Why Short-Term Results Can Mislead

Suppose a subscription app tests a new onboarding flow.

After seven days, the result looks strong:

- Signup completion: +8%
- Trial start: +5%

The team may want to launch immediately.

But the business goal is not only more trials. The business wants more valuable subscribers.

After 60 days, the result may look different:

- Paid conversion after trial: -4%
- Refund requests: +3%
- Net revenue per signup: no change

The short-term metrics were not wrong. They measured exactly what happened during the first week. The problem is that the first week did not capture the full business outcome.

Short-term experiments can mislead when:

- The true outcome is delayed.
- The treatment changes user composition.
- Users need time to learn the new experience.
- Users react temporarily because the feature is new.
- Supply, marketplace, or ecosystem effects accumulate slowly.
- The treatment affects retention, repeat purchase, or habit formation.

This is why experiment design should separate leading indicators from decision metrics.

**Leading indicators**

Metrics that move quickly and provide early evidence.

Examples:

- Signup completion
- Trial start
- First purchase
- First watch session
- First order completion

**Decision metrics**

Metrics closer to the product or business goal.

Examples:

- 30-day retention
- Paid conversion after trial
- Repeat purchase
- Refund-adjusted revenue
- Lifetime value
- Creator or seller long-term supply

Leading indicators are useful, but they should not automatically replace delayed decision metrics.

## Delayed Metrics

A delayed metric is a metric that cannot be fully observed during a short experiment window.

Examples:

| Product Area | Short-Term Metric | Delayed Metric |
|---|---|---|
| Subscription | Trial start | Paid conversion after trial |
| E-commerce | First purchase | Repeat purchase or refund-adjusted revenue |
| Video app | Watch time today | 30-day retention |
| Job platform | Applications submitted | Recruiter responses or hires |
| Marketplace | Orders | Repeat buyers, seller supply, dispute rate |

Delayed metrics matter because the treatment can change the quality of users, orders, or interactions.

For example, a one-click application feature may increase applications submitted. But if it mainly increases low-effort applications, recruiter response rate may fall. The immediate metric improves, while the downstream quality metric worsens.

The same pattern appears in many products:

- More signups, but lower paid conversion
- More purchases, but higher refund rate
- More watch time, but no retention improvement
- More messages, but more complaints
- More applications, but fewer meaningful matches

The design question is:

> How long does it take for the metric that really matters to become observable?

The answer should influence experiment duration, follow-up analysis, and whether the team needs a long-term holdout.

## Treatment Effects Over Time

A treatment effect can change over time.

Instead of reporting only one number, it is often useful to examine the effect by time since exposure:

| Time Since Exposure | Estimated Effect |
|---|---:|
| Day 1 | +5.0% |
| Day 7 | +3.0% |
| Day 30 | +0.5% |
| Day 60 | 0.0% |

This time path tells a different story from the day-1 effect alone.

There are several common patterns.

**Stable effect**

The effect appears quickly and remains similar over time.

This is common for simple friction reductions, such as fewer required fields in checkout.

**Decay**

The effect is large early but shrinks over time.

This may indicate novelty, temporary attention, or one-time behavior.

**Ramp-up**

The effect is small early but grows over time.

This may indicate learning, habit formation, network accumulation, or model feedback.

**Delayed harm**

The primary metric improves early, but guardrails or downstream metrics worsen later.

This may happen when the treatment reduces friction too much, attracts lower-intent users, or shifts traffic toward lower-quality interactions.

A useful report should show not only:

> What was the average effect?

but also:

> How did the effect evolve over time?

## Novelty Effects

A novelty effect happens when users react temporarily because an experience is new.

For example:

- A new feed layout may increase exploration in the first few days.
- A new badge may attract attention at first, then become ignored.
- A redesigned homepage may increase clicks because users are curious.
- A new notification style may work temporarily until users habituate.

The short-term lift may be real, but not durable.

Novelty effects are especially plausible when:

- The change is visually obvious.
- Users interact with the surface repeatedly.
- The metric depends on attention or curiosity.
- The treatment does not fundamentally improve underlying value.
- The early lift is much larger than the later lift.

Novelty is not always bad. A launch can benefit from attention. But if the business decision depends on durable value, the team needs to know whether the effect persists.

## Detecting Novelty Effects

One simple diagnostic is to compare new users and existing users.

Existing users have a prior experience. When the product changes, they may react partly because the experience feels new or surprising.

New users do not have the same before-versus-after comparison. They enter the product for the first time, so they are less exposed to novelty from the change itself.

This creates a useful diagnostic:

> If the treatment effect is strong for existing users but weak or absent for new users, the lift may partly reflect novelty or change reaction.

For example:

| Segment | Day-1 Lift | Day-30 Lift |
|---|---:|---:|
| Existing users | +6.0% | +1.0% |
| New users | +1.5% | +1.2% |

This pattern suggests that much of the early lift among existing users may be temporary. New users show a smaller but more stable effect, which may be closer to the durable value of the new experience.

This comparison is not perfect. New users and existing users differ in many ways besides novelty. Existing users may be more engaged, have richer histories, or use different product surfaces. So the comparison should be treated as evidence, not proof.

Other novelty diagnostics include:

- Plotting treatment effect by days since first exposure
- Comparing first-time exposure versus repeated exposure
- Checking whether the effect decays after users have seen the feature many times
- Looking for large early movement in attention-based metrics but smaller movement in value metrics
- Running a longer experiment when the decision depends on durable behavior

The practical question is:

> Would the treatment still look good after users stop reacting to the fact that it is new?

## Learning Effects

A learning effect is the opposite pattern: the treatment becomes more valuable as users learn how to use it.

Examples:

- A new analytics dashboard may be confusing at first, then valuable later.
- A new creator tool may require time before creators adjust their workflow.
- A recommendation feature may improve as users give more feedback.
- A seller tool may affect inventory quality only after sellers learn the new process.

Learning effects are especially plausible when:

- The treatment changes workflow.
- Users need to build a habit.
- The product requires repeated use.
- The value depends on content, data, or marketplace adaptation.
- Supply-side participants need time to respond.

In this case, a short experiment may underestimate the treatment effect.

For example:

| Time Since Exposure | Estimated Effect on Weekly Active Use |
|---|---:|
| Week 1 | -1.0% |
| Week 2 | +0.5% |
| Week 4 | +3.0% |
| Week 8 | +4.0% |

A one-week test might reject the feature. A longer test might show that users needed time to learn the new workflow.

The key is to distinguish learning from noise. A growing effect should make product sense, appear in relevant intermediate metrics, and ideally be supported by cohort analysis.

## Cohort Analysis

Cohort analysis is one of the most useful tools for long-term effects.

Instead of mixing all users together, the team tracks users based on when they first entered the experiment or first saw the treatment.

For example:

| Cohort | Day 1 Retention Lift | Day 7 Retention Lift | Day 30 Retention Lift |
|---|---:|---:|---:|
| Week 1 entrants | +2.0% | +1.5% | +0.3% |
| Week 2 entrants | +2.1% | +1.4% | +0.4% |
| Week 3 entrants | +1.8% | +1.2% | pending |

This helps separate:

- Time since exposure
- Calendar time
- User acquisition mix
- Product or market shocks

The distinction matters. Suppose the treatment effect falls in week 3. Is that because users are losing interest after repeated exposure? Or because week 3 had a holiday, outage, promotion, or different acquisition channel mix?

Cohort analysis helps answer that question.

## Retention Curves and Survival

Retention is naturally a long-term metric.

A single retention number, such as 30-day retention, is useful but incomplete. A retention curve gives more information:

| Day | Control Retention | Treatment Retention | Difference |
|---|---:|---:|---:|
| 1 | 45% | 47% | +2 pp |
| 7 | 28% | 29% | +1 pp |
| 30 | 16% | 16% | 0 pp |
| 60 | 10% | 9% | -1 pp |

This curve shows whether the treatment changes early activation, medium-term habit, or long-term retention.

For subscription products, a similar idea applies to churn or survival:

- How long does the user remain subscribed?
- When does churn occur?
- Does treatment delay churn or only shift it?
- Does treatment increase trial starts but reduce paid survival?

The analysis should match the business question. If the treatment affects trial starts, the team may need both:

- Conversion from trial to paid
- Retention among paid subscribers

Otherwise, the experiment may reward signups that do not become durable customers.

## LTV and Long-Term Value

Lifetime value, or LTV, is the total value a user is expected to generate over time.

In practice, true lifetime value is rarely fully observed during an experiment. Teams often use expected LTV:

$$
\text{Expected LTV}
=
\sum_{t=0}^{T}
P(\text{active at }t)
\times
E(\text{value at }t \mid \text{active})
$$

This formula is not meant to be a universal LTV model. It shows the basic idea: long-term value depends on both retention and value conditional on retention.

A treatment can improve one component while hurting another.

Examples:

- More users start trials, but fewer become paid subscribers.
- More buyers make a first purchase, but repeat purchase falls.
- More orders happen today, but refund-adjusted revenue declines.
- More sellers receive traffic, but high-quality sellers become less active.

For experiment decisions, LTV should be decomposed into interpretable pieces:

| Component | Example Metric |
|---|---|
| Acquisition or activation | Signup, trial start, first purchase |
| Conversion quality | Paid conversion, first-order cancellation |
| Repeat behavior | Repeat purchase, repeat session, repeat order |
| Retention | 7-day, 30-day, 90-day retention |
| Monetary value | Net revenue, contribution margin, GMV |
| Quality adjustment | Refunds, returns, complaints, disputes |

This decomposition prevents the team from hiding a tradeoff inside one modeled number.

## Long-Term Holdouts

A long-term holdout keeps a small group of users, markets, sellers, or traffic on the old experience after the main experiment ends.

The purpose is to answer questions that a short A/B test cannot answer:

- Does the effect persist?
- Does the effect decay after novelty fades?
- Does the effect grow as users learn?
- Does the treatment change retention or repeat behavior?
- Does the treatment affect ecosystem health over time?
- Does a model update improve long-term value or only short-term optimization?

For example, a recommendation team may launch a new ranking model to 95% of traffic but keep 5% on the old model for several months. This makes it possible to compare long-term retention, repeat purchase, content diversity, or seller ecosystem outcomes.

Long-term holdouts are especially useful when:

- The treatment affects habit formation.
- The treatment changes ranking, recommendation, pricing, or incentives.
- The primary goal is LTV or retention.
- The product has marketplace or ecosystem feedback loops.
- The cost of a wrong full rollout is high.

## Designing a Long-Term Holdout

A long-term holdout should be planned carefully. It is not just an experiment that accidentally runs for a long time.

Important design choices include:

**Holdout unit**

The holdout unit should match the level where the treatment is stable and where interference is manageable.

Common choices include:

- User-level holdout
- Account-level holdout
- Seller or creator holdout
- Market or geo holdout
- Traffic slice holdout

For example, if a ranking model affects shared seller exposure, a pure user-level holdout may not capture seller ecosystem effects. A market-level or seller-level holdout may be more appropriate.

**Holdout size**

The holdout should be large enough to estimate long-term metrics but small enough to limit opportunity cost.

A 1% holdout may be enough for a high-traffic consumer app. A smaller marketplace or subscription product may need a larger holdout or longer duration.

**Duration**

Duration should match the time scale of the decision metric.

Examples:

- 30 days for early retention
- 60-90 days for paid subscription survival
- Several months for repeat purchase
- Longer for seller ecosystem or creator supply effects

**Eligibility**

The holdout population should be clearly defined.

For example:

- All existing users at launch date
- New users entering after launch
- Sellers active during the pre-period
- Markets eligible for the new dispatch policy

Without a clear eligibility rule, long-term comparisons can become difficult to interpret.

**Consistency**

Holdout users should receive a consistent experience unless the experiment explicitly allows switching.

If users move in and out of the holdout, the long-term comparison becomes contaminated.

**Governance**

Long-term holdouts create product and business cost. Teams should document why the holdout exists, what metrics it protects, who owns it, and when it will be reviewed.

## The Cost of Holdouts

Holdouts are valuable because they preserve a counterfactual.

But they are not free.

Costs include:

- Opportunity cost from withholding a beneficial feature
- User experience inconsistency
- Engineering complexity
- Product debt from maintaining old code paths
- Lower short-term revenue or engagement
- Internal pressure to remove the holdout

Because of these costs, not every feature needs a long-term holdout.

Long-term holdouts are most justified when:

- The launch affects a large share of product value.
- The treatment changes long-term incentives or behavior.
- The short-term metric is an imperfect proxy.
- The downside risk is large.
- The team expects novelty, learning, or ecosystem feedback.

For small UI changes with clear short-term outcomes, a standard experiment may be enough.

## Reading Long-Term Results

Long-term analysis should avoid a common mistake:

> Looking only at the final cumulative number.

Cumulative metrics can hide time patterns.

For example:

| Window | Treatment Effect |
|---|---:|
| Days 1-7 | +5.0% |
| Days 8-30 | +1.0% |
| Days 31-60 | -0.5% |

The cumulative 60-day effect might still be positive, but the time path suggests the early lift is fading.

A strong long-term report should include:

- Cumulative effect
- Period-by-period effect
- Cohort analysis
- Retention or survival curves when relevant
- Guardrail trends
- Segment-level long-term effects
- Interpretation of novelty, learning, or delayed harm

Long-term results should also be checked against external events:

- Holidays
- Marketing campaigns
- Seasonality
- Competitor actions
- Pricing changes
- Product launches
- Logging changes

The longer the experiment runs, the more likely external changes become.

## Example: Recommendation Ranking

Suppose an e-commerce platform updates its recommendation ranking model. The old model optimizes short-term GMV. The new model adds a signal for repeat-purchase GMV between the same user and seller.

A short experiment may show:

- GMV per user: +2.0%
- GPM: +3.0%
- Repeat-purchase GMV: +4.5%

This is encouraging, but the team should still ask whether the change creates durable value.

Long-term metrics might include:

- 30-day and 90-day repeat purchase
- Refund-adjusted GMV
- Return rate
- User retention
- Seller concentration
- Cold-start seller exposure
- Active seller supply

The treatment-control gap over time is especially important. If the GMV gap widens over time, it suggests the model is identifying durable user-seller relationships rather than only reallocating short-term traffic.

A long-term holdout may be justified because ranking changes can reshape user behavior, seller incentives, and marketplace supply.

## Example: New Onboarding Flow

Suppose a subscription app tests a new onboarding flow.

Short-term results:

- Signup completion: +8%
- Trial start: +5%

These are good leading indicators.

But downstream results may show:

- Paid conversion after trial: -4%
- 30-day retention among paid users: no change
- Refund requests: +2%

This suggests the new onboarding flow may be reducing friction for lower-intent users. It gets more users into trial, but not necessarily more valuable subscribers.

The team should analyze:

- Paid conversion by acquisition channel
- Retention among users who convert to paid
- Refund and cancellation behavior
- Support contacts
- Net revenue per onboarding entrant

The launch decision may not be a simple yes or no. The team could keep the new flow for high-intent channels, add better expectation-setting for low-intent channels, or run a follow-up experiment with a more qualified trial start.

## Practical Workflow

A practical workflow for long-term experiment analysis is:

1. Define the long-term business outcome before the experiment.
2. Separate leading indicators from decision metrics.
3. Decide how long the key delayed metrics need to mature.
4. Plot treatment effects by time since exposure.
5. Compare new users and existing users when novelty effects are plausible.
6. Use cohort analysis to separate time since exposure from calendar time.
7. Track retention curves or survival curves when relevant.
8. Decompose LTV into interpretable components.
9. Use long-term holdouts when short-term results cannot answer the decision question.
10. Monitor guardrails and ecosystem metrics over time.
11. Review whether the long-term effect persists, decays, grows, or reverses.

## Common Mistakes

**Treating early lift as permanent**

A large first-week effect may reflect novelty, attention, or temporary curiosity.

**Ignoring delayed quality metrics**

More signups, applications, purchases, or messages may not mean more durable value.

**Stopping before users can learn**

Some treatments need time before users, sellers, creators, or models adapt.

**Using LTV as a black box**

LTV should be decomposed into retention, monetization, repeat behavior, and quality components.

**Keeping holdouts without governance**

Long-term holdouts have cost. They need owners, metrics, review dates, and a reason to exist.

**Ignoring external shocks**

Longer experiments are more exposed to holidays, campaigns, outages, seasonality, and other product launches.

## Key Takeaways

Short-term experiment results may not capture long-term product value.

Delayed metrics matter when the treatment affects retention, repeat behavior, quality, or ecosystem health.

Treatment effects can be stable, decay, ramp up, or reverse over time.

Novelty effects happen when users react temporarily because an experience is new.

Comparing existing users and new users can help diagnose novelty, because existing users have a before-versus-after experience while new users do not.

Learning effects happen when users or ecosystems need time to adapt.

Cohort analysis helps separate time since exposure from calendar time.

LTV should be decomposed into interpretable components instead of treated as a single magic number.

Long-term holdouts preserve a counterfactual, but they have opportunity cost and should be governed carefully.

The best long-term analysis shows both the cumulative effect and the effect path over time.
