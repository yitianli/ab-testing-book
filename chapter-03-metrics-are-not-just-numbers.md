# Metrics Are Not Just Numbers

Chapter 1 started with the experiment question. Chapter 2 turned that question into an experiment design. This chapter takes the next step: measurement.

A hypothesis is still abstract until it becomes a metric plan. Suppose the hypothesis is that showing a delivery-time range reduces user frustration. The experiment cannot measure "frustration" directly. The team has to decide what observable behavior will stand in for it: post-order cancellation, delivery-time complaints, support contacts, refunds, repeat orders, or satisfaction ratings.

Those choices are not interchangeable. Cancellation rate is close to business impact, but it may miss users who are unhappy and still complete the order. Complaint rate is closer to frustration, but it depends on whether users bother to complain. Repeat order rate may capture long-term trust, but it is delayed and affected by many other factors.

This is why metrics are not just numbers. A metric is a product judgment made measurable. It defines what counts as success, whose behavior matters, over what time window, and what tradeoffs the team is willing to accept.

Every metric carries assumptions:

- What outcome should the product optimize?
- Which users or events should be counted?
- What time window matters?
- What denominator should be used?
- What tradeoffs are acceptable?
- What behavior might the metric accidentally encourage?

Good experiment analysis depends on making those assumptions explicit. A/B testing does not only compare treatment and control. It compares treatment and control through the lens of the metrics chosen before the test begins.

The goal of this chapter is not to memorize metric types. The goal is to learn how to ask what a metric is really measuring. A useful order is:

1. What outcome are we trying to measure?
2. What do we do when the true outcome is delayed or hard to observe?
3. Who or what enters the metric?
4. If the metric is a ratio, how are numerator and denominator defined?
5. How do we diagnose why a metric moved?

Before discussing specific metric problems, two questions are useful for almost any experiment:

> If this metric improves, what kind of product behavior are we rewarding?

> What could get worse while this metric improves?

These questions lead naturally to primary metrics, secondary metrics, guardrails, proxy metrics, denominator choices, and metric decomposition.

## From Hypothesis to Metric Plan

A metric plan should follow from the hypothesis.

If the hypothesis is:

> Showing a delivery-time range reduces post-order cancellations caused by delivery-time uncertainty, without reducing order conversion.

then the metric plan should reflect all parts of that sentence:

- The primary metric should measure the main expected benefit.
- Secondary metrics should explain the mechanism.
- Guardrails should capture important ways the treatment could cause harm.

For the delivery-time example:

- Primary metric: post-order cancellation rate caused by delivery-time concerns
- Secondary metrics: delivery-time complaints, delivery-time satisfaction, tracking-page clicks
- Guardrails: restaurant page to checkout conversion, order conversion, refund rate, support contact rate, repeat order rate

The primary metric should match the launch decision. Secondary metrics help explain why the primary metric moved. Guardrails protect against false wins.

A good metric set is small enough to interpret, but broad enough to prevent the team from declaring victory on a narrow number while missing obvious harm elsewhere.

## What Outcome Are We Measuring?

The first metric question is what outcome the experiment is trying to measure.

Some outcomes are direct and observable. A checkout experiment can measure purchases. A pricing experiment can measure revenue. A notification experiment can measure opens.

Other outcomes are conceptual. A recommendation system may want to improve satisfaction, trust, discovery, marketplace health, or lifetime value. These outcomes matter, but they are not directly observed as a single clean event.

When the true outcome is hard to observe, the team needs to decide whether the experiment will use:

- A direct decision metric
- A delayed metric
- A proxy metric
- A predicted metric
- A combination of metrics and guardrails

This choice is part of the product judgment. The metric defines what kind of improvement the experiment is allowed to claim.

### Delayed Metrics

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

### Proxy Metrics

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

### Choosing a Proxy Metric

A proxy metric should not be chosen only because it is easy to measure. It should be chosen because there is a credible reason to believe it moves with the true outcome.

A practical process is:

1. Start from the true outcome.
2. Map the path from product behavior to that outcome.
3. Choose the closest observable metric on that path.
4. Validate the proxy against downstream outcomes when historical data is available.
5. Add guardrails for the ways the proxy could fail.
6. Decide whether the proxy is strong enough to be a decision metric or only a leading metric.

Suppose a subscription app wants to increase long-term paid subscriber value, but 6-month retention is too slow to observe in every experiment. Possible early metrics include:

- Signup completion
- Trial start
- Paid conversion after trial
- First-month retention
- Expected lifetime value based on early behavior

These are not equally strong proxies. Signup completion and trial start are early, but they may mostly measure reduced friction. Paid conversion after trial is closer to revenue. First-month retention is delayed, but closer to long-term value. Expected lifetime value may be useful if the prediction model has been validated against real downstream retention and revenue.

The team should choose the proxy based on the decision risk. If the change is small and low-risk, a validated early proxy may be enough for rollout. If the change affects pricing, user trust, marketplace incentives, or recommendation quality, the team should wait for stronger downstream metrics or launch with a long-term holdout.

### Example: Recommendation Ranking and Lifetime Value

Consider an e-commerce recommendation system that already optimizes short-term GMV. The team wants to add a ranking signal that predicts repeat-purchase value between the same user and creator.

The true goal is user lifetime value, but true LTV may take months to observe. The team therefore needs a practical decision metric or proxy.

Possible candidates include:

- Short-term GMV per user
- 30-day GMV per user
- Repeat purchase rate
- Repeat-purchase GMV
- Orders per buyer
- Buyer retention
- Predicted LTV

These candidates are not equally strong. Short-term GMV is easy to observe, but it may be too shallow. It can increase because the model pushes impulse purchases, discounts, or already high-converting creators. Repeat-purchase GMV is closer to the hypothesis because it measures whether the same user returns to buy again from the same creator. Predicted LTV may be useful if the model has been validated against longer-term retention and revenue.

The proxy should be protected by guardrails such as refund rate, return rate, complaint rate, negative feedback, retention, and creator exposure concentration.

A reasonable metric plan might use repeat-purchase GMV or predicted 30-day or 90-day LTV as the decision metric, short-term GMV and GPM as secondary metrics, and refund rate, return rate, negative feedback, retention, and concentration as guardrails.

## Who or What Enters the Metric?

After choosing the outcome, the next question is who or what gets counted.

This is often a denominator question. A metric can change not only because behavior changes, but also because the population entering the denominator changes.

### Conditional Metrics

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

For example, suppose 10,000 users enter onboarding.

In control:

- 40% start a trial, producing 4,000 trial starters
- 20% of trial starters become paid, producing 800 paid subscribers
- Paid subscribers per trial starter = 800 / 4,000 = 20%
- Paid subscribers per onboarding entrant = 800 / 10,000 = 8.0%

In treatment:

- 45% start a trial, producing 4,500 trial starters
- 18.8% of trial starters become paid, producing about 846 paid subscribers
- Paid subscribers per trial starter = 846 / 4,500 = 18.8%
- Paid subscribers per onboarding entrant = 846 / 10,000 = 8.46%

In this example, the conditional paid conversion rate decreases from 20% to 18.8%, but the full-funnel paid subscriber rate increases from 8.0% to 8.46%. The treatment brought more users into the trial funnel, and the larger top-of-funnel gain more than offset the lower conversion rate among trial starters.

The general rule is:

> Be careful when conditioning on a post-treatment event.

If treatment affects whether users enter the denominator, the conditional metric mixes product impact with population composition.

## Ratio Metrics: How Numerators and Denominators Shape the Result

After deciding what outcome to measure and who enters the metric, many experiments still need one more layer of care: how numerator and denominator are combined.

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

The ratio alone does not explain the mechanism. Good analysis should inspect numerator movement, denominator movement, ratio movement, user-level distribution, and whether the denominator was affected by treatment.

### Ratio of Totals and Average of User Ratios

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

### Unit of Analysis for Ratio Metrics

The randomization unit matters for ratio metrics.

Suppose users are randomized into treatment and control, and the metric is CTR. One approach is to compute:

$$
\text{Overall CTR} = \frac{\text{Total clicks}}{\text{Total impressions}}
$$

This is often the number shown in dashboards. But for inference, treating every impression as independent can be misleading because impressions from the same user are correlated. A heavy user may contribute hundreds of impressions, while a light user contributes one.

If the experiment randomizes users, the safest principle is:

> Analyze the metric at the same level as randomization whenever possible.

For user-level randomization, this often means aggregating clicks and impressions by user first.

This does not mean every ratio metric must be reported as an average of user-level ratios. Some business questions are better answered by a ratio of totals. But even then, the analysis should remember that the experiment randomized users, not impressions.

The main point is:

> The analysis unit should respect the randomization unit.

### Uncertainty for Ratio Metrics

Ratio metrics also create a separate inference problem. The experiment needs not only an estimated lift, but also a valid estimate of uncertainty around that lift.

This subsection is more technical than the earlier metric-design discussion. The key idea is simple: if users were randomized, uncertainty should usually be estimated in a way that treats users, not lower-level events, as the independent units.

For a metric such as CTR, uncertainty comes from several sources:

- Clicks vary across users
- Impressions vary across users
- Clicks and impressions are related within the same user
- The denominator may itself be affected by treatment

This is why ratio metrics often need more care than simple binary metrics. If users are randomized, a confidence interval should not be calculated as if every impression were an independent observation.

There are three common ways to estimate uncertainty for ratio metrics.

#### User-Level Bootstrap

The user-level bootstrap is often the easiest method to explain and implement.

Suppose each user has a numerator $X_i$, such as clicks, and a denominator $Y_i$, such as impressions. To estimate uncertainty for overall CTR:

$$
\widehat{R} = \frac{\sum_i X_i}{\sum_i Y_i}
$$

the bootstrap repeatedly resamples users with replacement, recomputes the ratio in each resampled dataset, and uses the distribution of those bootstrap ratios to form a confidence interval.

The important detail is the resampling unit:

> If users were randomized, resample users, not impressions.

This preserves the relationship between a user's clicks and impressions. Heavy users, light users, and users with unusual behavior are resampled as whole units.

The bootstrap is flexible and works well for skewed metrics, but it can be computationally expensive for very large experiments.

#### Delta Method

The delta method gives an analytic approximation for the variance of a ratio.

Again let $X_i$ be the numerator and $Y_i$ be the denominator for user $i$. The ratio of means is:

$$
\widehat{R} = \frac{\bar{X}}{\bar{Y}}
$$

where:

$$
\bar{X} = \frac{1}{n}\sum_i X_i,\quad \bar{Y} = \frac{1}{n}\sum_i Y_i
$$

The approximate variance of $\widehat{R}$ is:

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

#### Event-Level Regression and Cluster-Robust Standard Errors

Sometimes the data is stored at the event level rather than the user level. For CTR, each impression might be one row, with a binary outcome indicating whether that impression received a click.

In this setup, an event-level regression estimates the probability that an impression receives a click:

$$
\text{Clicked}_{ij} = \alpha + \tau \text{Treatment}_i + \epsilon_{ij}
$$

where $j$ indexes impressions for user $i$.

With one row per impression and no additional covariates, the treatment-control difference from this regression is equivalent to comparing ratio-of-sums CTR:

$$
\text{CTR} = \frac{\sum_i \text{clicks}_i}{\sum_i \text{impressions}_i}
$$

This is an impression-weighted estimand. Users with more impressions contribute more to the estimate. It is not the same as average user CTR, which would first calculate each user's CTR and then average across users.

The problem is that impressions from the same user are usually correlated. A user who tends to click more may click more across many impressions. If the analysis ignores this correlation, the standard error can be too small and the experiment can look more precise than it really is.

Cluster-robust standard errors address this by allowing observations within the same user to be correlated. The treatment effect is still estimated using event-level data, but uncertainty is calculated as if users, not impressions, are the independent units.

This approach is useful when the product question is naturally impression-level, such as:

> Did treatment increase the probability that an impression receives a click?

It is less appropriate when the product question is user-level, such as:

> Did treatment improve the average user's experience?

The method should follow the estimand. First decide what ratio answers the product question. Then choose an uncertainty method that respects the randomization unit.

## Why Did the Metric Move?

When a metric moves, or does not move as expected, the next question is why.

Metric decomposition breaks a high-level metric into components. It is often used for metric movement attribution, or root-cause analysis of why a metric changed, but it is also useful before the experiment starts because it helps choose secondary metrics.

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

### Example: Checkout Conversion and Average Order Value

Consider an e-commerce website testing a new checkout page. The treatment group shows:

- Purchase conversion rate: +2%, statistically significant
- Average order value: -4%, statistically significant
- Revenue per visitor: no significant change
- Checkout error rate: no significant change

This is not a simple win. The checkout page converts more users, but the average order is smaller. Revenue per visitor, which combines conversion and order value, does not significantly change.

The decomposition makes the pattern visible:

$$
\text{Revenue per visitor}
=
\text{Purchase conversion rate}
\times
\text{Average order value}
$$

If purchase conversion increases while average order value decreases, the two effects can offset each other. Revenue per visitor may remain flat even though both components changed significantly.

One possible explanation is that the new checkout reduces friction for lower-value carts. Users who were previously on the margin now complete their purchases, increasing conversion but lowering average order value. To verify this, the team can segment users by pre-checkout cart value and check whether the conversion lift is concentrated among low-value carts.

Another possibility is that the new checkout unintentionally reduces add-ons, bundles, shipping upgrades, or other high-value choices, meaning optional paid choices that usually increase order value. To verify this, the team can check add-on attachment rate, promo code usage, shipping option selection, and category mix.

The point of decomposition is to turn a flat high-level metric into a clearer diagnosis: conversion improved, average order value declined, and the next analysis should explain which user or order segments drove each movement.

## Common Metric Failure Modes

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

The first metric question is what outcome the experiment is trying to measure.

Delayed or hard-to-observe outcomes often require carefully validated proxy metrics.

Conditional metrics can be misleading when treatment changes who enters the denominator.

Ratio metrics can move because of the numerator, the denominator, or both.

For ratio metrics, inference should respect the randomization unit.

Metric decomposition helps explain why a high-level metric moved or failed to move as expected.

Good experiment decisions come from the full metric pattern, not a single dashboard number.
