# Metrics Are Not Just Numbers

Metrics look objective. They appear in dashboards, tables, scorecards, and experiment reports. They have precise names and decimal places. It is tempting to treat them as facts.

But a metric is not just a number. A metric is a decision about what matters.

Every metric contains assumptions:

- What outcome should the product optimize?
- Which users or events should be counted?
- What time window matters?
- What denominator should be used?
- What tradeoffs are acceptable?
- What behavior might the metric accidentally encourage?

Good experiment analysis depends on understanding those assumptions. A/B testing does not only compare treatment and control. It compares treatment and control through the lens of the metrics we choose.

This chapter focuses on several metric problems that appear again and again in real experiments:

- Ratio metrics
- Conditional metrics
- Delayed metrics
- Proxy metrics
- Metric decomposition
- Metric tradeoffs

The goal is not to memorize a list of metric types. The goal is to learn how to ask what a metric is really measuring.

## Metrics Represent Product Judgment

Suppose a video app tests a new feed ranking model. Possible primary metrics include:

- Watch time per user
- 7-day retention
- Sessions per user
- Like rate
- User satisfaction survey score
- Reported content rate

Each metric tells a different story.

Watch time emphasizes engagement. Retention emphasizes durable usage. Like rate emphasizes explicit positive feedback. Reported content emphasizes safety. No single metric fully captures user value.

This is why metric choice is a product judgment, not only a statistical choice. The metric determines what kind of win the experiment is allowed to claim.

If the ranking model increases watch time by recommending addictive but low-quality content, watch time may say the experiment succeeded while user trust says it failed. If the model increases retention but reduces creator diversity, the user-side metric may miss ecosystem damage.

The first question for any metric is:

> If this metric improves, what kind of product behavior are we rewarding?

The second question is:

> What could get worse while this metric improves?

These two questions lead naturally to primary metrics, secondary metrics, and guardrails.

## Ratio Metrics

Many common experiment metrics are ratios:

$$
\text{Metric} = \frac{\text{Numerator}}{\text{Denominator}}
$$

Examples include:

- Click-through rate = clicks / impressions
- Conversion rate = purchases / visitors
- Average order value = revenue / orders
- Revenue per visitor = revenue / visitors
- Like rate = likes / views
- GPM = GMV / (impressions / 1,000)
- Recruiter response rate = recruiter responses / applications

Ratio metrics are useful because they normalize for scale. A product with more impressions will naturally have more clicks, so clicks alone may not be comparable. CTR adjusts clicks by impressions.

But ratio metrics can be tricky because the treatment may affect the numerator, the denominator, or both.

Suppose a recommendation update changes CTR:

$$
\text{CTR} = \frac{\text{Clicks}}{\text{Impressions}}
$$

CTR can increase because users click more. It can also increase because the system shows fewer impressions to low-intent users. CTR can decrease because clicks fall, or because impressions increase faster than clicks.

The ratio alone does not explain the mechanism.

Good analysis should inspect:

- Numerator movement
- Denominator movement
- Ratio movement
- User-level distribution
- Whether the denominator was affected by treatment

## The Unit of Analysis for Ratio Metrics

The randomization unit matters for ratio metrics.

Suppose users are randomized into treatment and control, and the metric is CTR. One approach is to compute:

$$
\text{Overall CTR} = \frac{\text{Total clicks}}{\text{Total impressions}}
$$

This is often the number shown in dashboards. But for inference, treating every impression as independent can be misleading because impressions from the same user are correlated. A heavy user may contribute hundreds of impressions, while a light user contributes one.

If the experiment randomizes users, the safest principle is:

> Analyze the metric at the same level as randomization whenever possible.

For user-level randomization, this means aggregating clicks and impressions by user first.

For each user \(i\):

$$
\text{CTR}_i = \frac{\text{Clicks}_i}{\text{Impressions}_i}
$$

Then compare user-level outcomes between treatment and control.

This does not mean every ratio metric must literally be reported as an average of user-level ratios. Some business questions are better answered by a ratio of totals, as the next section discusses. But even then, the analysis should remember that the experiment randomized users, not impressions.

The main point is:

> The analysis unit should respect the randomization unit.

## Ratio of Means and Mean of Ratios

There are two similar-looking but different ways to compute a ratio metric.

The first is a ratio of totals:

$$
\frac{\sum_i \text{Clicks}_i}{\sum_i \text{Impressions}_i}
$$

The second is an average of user-level ratios:

$$
\frac{1}{N}\sum_i \frac{\text{Clicks}_i}{\text{Impressions}_i}
$$

These two quantities answer different questions.

The ratio of totals gives more weight to users with many impressions. It answers:

> What fraction of all impressions received a click?

The average of user-level ratios gives each user equal weight. It answers:

> What was the average user's click-through rate?

Neither is always correct. The right choice depends on the decision.

If the business cares about monetization per impression, the ratio of totals may be appropriate. If the product cares about the average user experience, the mean of user-level ratios may be better.

The danger is using one while believing it means the other.

## Uncertainty for Ratio Metrics

Ratio metrics also create a separate inference problem. The experiment needs not only an estimated lift, but also a valid estimate of uncertainty around that lift.

For a metric such as CTR, uncertainty comes from several sources:

- Clicks vary across users
- Impressions vary across users
- Clicks and impressions are related within the same user
- The denominator may itself be affected by treatment

This is why ratio metrics often need more care than simple binary metrics. If users are randomized, a confidence interval should not be calculated as if every impression were an independent observation.

There are three common ways to estimate uncertainty for ratio metrics.

### User-Level Bootstrap

The user-level bootstrap is often the easiest method to explain and implement.

Suppose each user has a numerator \(X_i\), such as clicks, and a denominator \(Y_i\), such as impressions. To estimate uncertainty for overall CTR:

$$
\widehat{R} = \frac{\sum_i X_i}{\sum_i Y_i}
$$

the bootstrap repeatedly resamples users with replacement, recomputes the ratio in each resampled dataset, and uses the distribution of those bootstrap ratios to form a confidence interval.

The important detail is the resampling unit:

> If users were randomized, resample users, not impressions.

This preserves the relationship between a user's clicks and impressions. Heavy users, light users, and users with unusual behavior are resampled as whole units.

The bootstrap is flexible and works well for skewed metrics, but it can be computationally expensive for very large experiments.

### Delta Method

The delta method gives an analytic approximation for the variance of a ratio.

Again let \(X_i\) be the numerator and \(Y_i\) be the denominator for user \(i\). The ratio of means is:

$$
\widehat{R} = \frac{\bar{X}}{\bar{Y}}
$$

where:

$$
\bar{X} = \frac{1}{n}\sum_i X_i,\quad \bar{Y} = \frac{1}{n}\sum_i Y_i
$$

The approximate variance of \(\widehat{R}\) is:

$$
\text{Var}(\widehat{R}) \approx
\frac{1}{n\bar{Y}^2}
\left[
\text{Var}(X_i)
- 2\widehat{R}\text{Cov}(X_i,Y_i)
+ \widehat{R}^2\text{Var}(Y_i)
\right]
$$

This formula shows why ratio metrics are not just ordinary averages. The variance depends on:

- Variation in the numerator
- Variation in the denominator
- Covariance between numerator and denominator

For an A/B test, compute this variance separately for treatment and control:

$$
\text{Var}(\widehat{R}_T - \widehat{R}_C)
\approx
\text{Var}(\widehat{R}_T) + \text{Var}(\widehat{R}_C)
$$

Then the standard error is:

$$
\text{SE} = \sqrt{\text{Var}(\widehat{R}_T - \widehat{R}_C)}
$$

The delta method is fast and common in experimentation platforms. Its weakness is that it is an approximation, so it may be less reliable for very small samples, very skewed denominators, or ratios with unstable denominators.

### Regression and Cluster-Robust Standard Errors

Sometimes the data is stored at the event level rather than the user level. For CTR, each impression might be one row, with a binary outcome indicating whether that impression received a click.

A naive event-level regression would treat every impression as independent:

$$
\text{Clicked}_{ij} = \alpha + \tau \text{Treatment}_i + \epsilon_{ij}
$$

where \(j\) indexes impressions for user \(i\).

The problem is that impressions from the same user are usually correlated. A user who tends to click more may click more across many impressions. If the analysis ignores this correlation, the standard error can be too small and the experiment can look more precise than it really is.

Cluster-robust standard errors address this by allowing observations within the same user to be correlated. The treatment effect is still estimated using event-level data, but uncertainty is calculated as if users, not impressions, are the independent units.

This approach is useful when the product question is naturally impression-level, such as:

> Did treatment increase the probability that an impression receives a click?

It is less appropriate when the product question is user-level, such as:

> Did treatment improve the average user's experience?

The method should follow the estimand. First decide what ratio answers the product question. Then choose an uncertainty method that respects the randomization unit.

## Conditional Metrics

Some metrics are conditional on reaching a later stage in the funnel:

- Paid conversion after trial = paid subscribers / trial starters
- Recruiter response rate = responses / applications
- Refund rate = refunds / orders
- Average order value = revenue / orders
- 30-day retention among paid users = retained paid users / paid users

Conditional metrics are useful, but they can be hard to interpret when treatment changes who enters the denominator.

Consider a subscription app testing a simpler onboarding flow:

- Signup completion increases
- Trial starts increase
- Paid conversion after trial decreases

The decrease in paid conversion after trial may not mean the payment experience got worse. It may mean the new onboarding flow brought more low-intent users into the trial population. The denominator changed.

In this case, the cleaner decision metric may be:

$$
\text{Paid subscribers per onboarding entrant}
$$

rather than:

$$
\text{Paid subscribers per trial starter}
$$

The general rule is:

> Be careful when conditioning on a post-treatment event.

If treatment affects whether users enter the denominator, the conditional metric mixes product impact with population composition.

## Delayed Metrics

Some outcomes appear quickly:

- Clicks
- Page views
- Trial starts
- Add-to-cart actions
- Session duration

Other outcomes take time:

- Paid conversion after trial
- Refunds and returns
- Repeat purchases
- 30-day retention
- Churn
- Lifetime value
- Hire rate after job application

Delayed metrics create a design problem. The experiment may collect enough users quickly, but the outcome is not observable yet.

Suppose a subscription app tests a new onboarding flow. Signup completion can be measured immediately. Trial-to-paid conversion may require 7 or 14 days. Retention may require 30 days or more.

Stopping the experiment after three days because signup completion improved may be premature. The metric that matters may not have had time to appear.

A useful way to classify metrics is:

- Leading metrics: move early and indicate possible direction
- Decision metrics: determine whether the experiment should launch
- Long-term metrics: validate whether the decision remains good over time

Leading metrics are helpful, but they should not replace delayed decision metrics when the product goal is long-term value.

## Proxy Metrics

A proxy metric is a metric used because the true outcome is hard or slow to measure.

Examples:

- Watch time as a proxy for user satisfaction
- Trial start as a proxy for subscription growth
- Add-to-cart as a proxy for purchase intent
- Recruiter response as a proxy for application quality
- Repeat-purchase GMV as a proxy for lifetime value

Proxy metrics are often necessary. The true outcome may take months, or it may be difficult to measure directly. But proxies are dangerous when they can be optimized in ways that break their relationship with the true goal.

Watch time may be a good proxy for satisfaction when users watch content they enjoy. It becomes a worse proxy if a model increases watch time by recommending content users later regret watching.

Trial starts may be a good proxy for subscription growth when trial quality stays stable. It becomes a worse proxy if a new onboarding flow attracts many low-intent trial users who never pay.

A good proxy metric should be:

- Directionally aligned with the true goal
- Hard to manipulate in harmful ways
- Validated against downstream outcomes
- Protected by guardrail metrics

The key question is:

> Under what conditions does this proxy stop being trustworthy?

## Metric Decomposition

When a metric moves, the next question is why.

Metric decomposition breaks a high-level metric into components.

For e-commerce:

$$
\text{Revenue per visitor} = \text{Purchase conversion rate} \times \text{Average order value}
$$

For marketplace GMV:

$$
\text{GMV} = \text{Buyers} \times \text{Orders per buyer} \times \text{Average order value}
$$

For ad revenue:

$$
\text{Revenue} = \text{Impressions} \times \text{CTR} \times \text{Conversion rate} \times \text{Value per conversion}
$$

For a subscription funnel:

$$
\text{Paid subscribers per visitor} =
\text{Signup rate} \times
\text{Trial start rate} \times
\text{Paid conversion rate}
$$

These decompositions help distinguish different stories.

Suppose revenue per visitor does not change, but purchase conversion increases and average order value decreases. The treatment may be converting more low-value orders. That might be acceptable if the goal is first-time buyer activation, but not if the goal is near-term revenue.

Suppose GMV increases because average order value increases, but buyer count falls. The product may be pushing high-value purchases while discouraging smaller buyers.

Metric decomposition turns a result into a diagnosis.

## Example: Checkout Conversion and Average Order Value

Consider an e-commerce website testing a new checkout page. The treatment group shows:

- Purchase conversion rate: +2%, statistically significant
- Average order value: -4%, statistically significant
- Revenue per visitor: no significant change
- Checkout error rate: no significant change

This is not a simple win. The checkout page converts more users, but the average order is smaller. Revenue per visitor, which combines conversion and order value, does not significantly change.

One possible explanation is that the new checkout reduces friction for lower-value carts. Users who were previously on the margin now complete their purchases, increasing conversion but lowering average order value. Another possibility is that the new checkout unintentionally reduces add-ons, bundles, shipping upgrades, or other high-value choices.

The right analysis decomposes the result:

- Did low-value cart conversion increase?
- Did high-value cart conversion change?
- Did add-on attachment rate decrease?
- Did promo code usage change?
- Did shipping option selection change?
- Did product category mix shift?

The launch decision depends on the product goal. If revenue per visitor is flat and guardrails are neutral, the new checkout may not create clear monetization value. But if it improves first-time buyer activation, reduces user frustration, or performs well for a strategic segment, a targeted rollout may still be reasonable.

The metric pattern matters more than any single number.

## Example: Recommendation Ranking and Long-Term Value

Consider an e-commerce recommendation system that already predicts short-term GMV. The team adds a new ranking signal that predicts repeat-purchase GMV between the same user and creator. The goal is to identify user-creator relationships with higher long-term commercial value.

Several metrics are relevant:

- GMV per user
- Repeat-purchase GMV
- GPM, or GMV per thousand impressions
- Treatment-control GMV gap over time
- Refund rate
- Return rate
- Negative feedback
- Retention
- Creator exposure concentration

Repeat-purchase GMV is closer to the long-term goal than immediate GMV, but it is still a proxy for lifetime value. It should be interpreted with guardrails. A model could increase repeat-purchase GMV by overexposing a small set of high-converting creators, reducing diversity or harming smaller creators. It could also increase gross GMV while increasing refunds or returns.

Metric decomposition helps here too:

$$
\text{GMV per user} =
\text{Impressions per user} \times
\text{GPM} / 1000
$$

and:

$$
\text{GMV} =
\text{Number of buyers} \times
\text{Orders per buyer} \times
\text{Average order value}
$$

If GMV increases, the team should ask whether the lift comes from more buyers, more frequent purchases, larger orders, or repeated purchases between the same user and creator. Each story has a different product meaning.

The strongest result would show that repeat-purchase GMV and GMV per user improve, the treatment-control gap widens over time, and guardrails such as retention, negative feedback, refunds, returns, and ecosystem concentration remain healthy.

## Practical and Statistical Metric Problems

Metric interpretation can fail in several common ways.

**The metric is too shallow**

Click rate improves, but purchase or retention does not.

**The metric is too delayed**

Lifetime value is the true goal, but the experiment cannot wait six months.

**The metric is conditional on post-treatment behavior**

Paid conversion among trial users changes because the treatment changed who started a trial.

**The metric is sensitive to outliers**

Revenue per user or GMV per user may be dominated by a small number of very large buyers.

**The metric can be optimized in harmful ways**

Watch time can increase through low-quality content. GMV can increase through aggressive discounts or poor-quality orders.

**The metric hides distributional effects**

The average user improves, but new users or small sellers are harmed.

These problems do not mean the metrics are useless. They mean that every metric needs context.

## Choosing a Metric Set

A good experiment metric set usually contains:

- A primary metric that matches the decision
- Secondary metrics that explain the mechanism
- Guardrails that protect against known risks
- A time window that matches the expected effect
- A denominator that is not misleading
- A plan for decomposing surprising movements

The metric set should be small enough to interpret, but broad enough to prevent a false win.

For a checkout experiment:

- Primary metric: revenue per visitor
- Secondary metrics: conversion rate, average order value, add-on attachment rate
- Guardrails: payment error rate, refund rate, support contact rate

For a recommendation experiment:

- Primary metric: long-term engagement or GMV per user, depending on the product goal
- Secondary metrics: CTR, completion rate, repeat purchase, GPM
- Guardrails: retention, negative feedback, reports, refunds, ecosystem diversity

For an onboarding experiment:

- Primary metric: paid subscribers or expected LTV per onboarding entrant
- Secondary metrics: signup completion, trial start, trial-to-paid conversion
- Guardrails: refund rate, cancellation rate, support tickets, trial abuse

The exact metrics differ by product, but the structure is similar.

## Key Takeaways

Metrics are product judgments, not just numbers.

A ratio metric can move because of the numerator, the denominator, or both.

For ratio metrics, inference should respect the randomization unit.

Conditional metrics can be misleading when treatment changes who enters the denominator.

Delayed metrics require longer observation windows or carefully validated proxies.

Proxy metrics are useful only when they remain aligned with the true goal.

Metric decomposition helps explain why a high-level metric moved.

Good experiment decisions come from the full metric pattern, not a single dashboard number.
