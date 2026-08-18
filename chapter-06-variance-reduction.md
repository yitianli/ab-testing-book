# Variance Reduction

Some experiments fail not because the idea is weak, but because the data is noisy.

Imagine a product change that increases revenue per user by 1%. If user spending varies wildly, that 1% lift may be hard to detect. A few unusually large purchases can dominate the metric. Some users may never purchase at all. Others may buy frequently regardless of treatment. The treatment effect is there, but it is buried inside natural user-to-user variation.

One solution is to get more traffic. More users reduce uncertainty.

But traffic is expensive. Experiments take time, and many products have limited eligible users. Variance reduction methods try to answer a practical question:

> Can we make the experiment more precise without increasing sample size?

The answer is often yes.

Variance reduction uses information we already know about users, markets, items, or time periods to remove predictable noise from the outcome. If a user's past behavior strongly predicts their future behavior, then adjusting for past behavior can make the treatment-control comparison sharper.

This chapter covers three common approaches:

- Stratification and blocking
- Regression adjustment
- CUPED

The methods differ in implementation, but the intuition is shared:

> Explain away variation that is unrelated to the treatment, so the remaining comparison has less noise.

## Why Variance Matters

For a simple difference in means, the standard error is roughly:

$$
\text{SE}(\bar{Y}_T - \bar{Y}_C)
=
\sqrt{\frac{\sigma_T^2}{n_T} + \frac{\sigma_C^2}{n_C}}
$$

If treatment and control have equal sample size $n$ and similar variance $\sigma^2$, this becomes:

$$
\text{SE} \approx \sqrt{\frac{2\sigma^2}{n}}
$$

There are two ways to reduce the standard error:

- Increase $n$, the sample size
- Decrease $\sigma^2$, the outcome variance

Most sample size discussions focus on increasing $n$. Variance reduction focuses on decreasing $\sigma^2$.

This matters because sample size requirements are proportional to variance:

$$
n \propto \frac{\sigma^2}{\delta^2}
$$

If a method reduces variance by 25%, the experiment may need about 25% less sample size for the same MDE and power.

## Predictable Noise

Not all variation in outcomes is equally useful.

Suppose an e-commerce experiment measures GMV per user. Some users are naturally heavy buyers. Others rarely purchase. If those differences existed before the experiment, they are not caused by treatment.

Pre-treatment behavior can be used to explain part of the outcome:

- Past purchase amount predicts future purchase amount
- Past watch time predicts future watch time
- Past session frequency predicts future retention
- Past order count predicts future order count
- Country or device type predicts baseline conversion

If treatment and control are randomized, these variables should be balanced on average. But in any finite experiment, imbalance and noise remain.

Variance reduction does not make randomization unnecessary. It makes randomized experiments more precise.

The methods in this chapter use predictable noise in two ways:

- Design-stage methods, such as stratification and blocking, use pre-treatment information before assignment to improve balance.
- Analysis-stage methods, such as regression adjustment and CUPED, use pre-treatment information after assignment to reduce residual noise.

## Stratification

Stratification means splitting users into groups before randomization, then randomizing within each group.

Suppose a subscription app knows that new users and returning users have very different baseline conversion rates. If the experiment randomizes everyone together, randomization should balance the two groups on average, but one variant may still accidentally get slightly more returning users.

With stratified randomization:

- New users are randomized 50/50 into control and treatment
- Returning users are randomized 50/50 into control and treatment

This guarantees balance on the stratification variable.

Common stratification variables include:

- Country or region
- Device type
- New versus returning user
- Acquisition channel
- User activity level
- Marketplace side or seller tier
- Pre-period purchase frequency

Stratification is most useful when the variable strongly predicts the outcome.

If mobile and desktop users behave differently, stratifying by device can reduce noise. If browser language barely relates to the outcome, stratifying by language may not help much.

The math intuition is that stratification reduces noise caused by imbalance between groups.

Let $S$ be the stratification variable, such as new versus returning user. The total outcome variance can be decomposed as:

$$
\text{Var}(Y)
=
E[\text{Var}(Y \mid S)]
+
\text{Var}(E[Y \mid S])
$$

The first term is variation within strata. The second term is variation between strata. If new and returning users have very different baseline conversion rates, then $\text{Var}(E[Y \mid S])$ is large.

Stratification does not make users within a stratum identical, and it does not remove all outcome variation. It helps because treatment and control are compared within the same stratum first. New users are compared with new users, returning users with returning users. The final estimate then averages these within-stratum comparisons. This reduces noise from one variant accidentally containing more high-converting users.

Blocking is the same basic idea under a different name. The term is often used when the groups are natural experimental contexts or matched sets, such as stores, cities, schools, restaurants, creators, or time windows. For example, a food delivery platform running a geo experiment might pair cities with similar historical order volume and cancellation rate, then randomize one city in each pair to treatment. The goal is still to compare treatment and control within similar groups, then combine the within-block estimates.

### Stratified Analysis

After stratified randomization, the treatment effect can be estimated within each stratum and then combined.

Let $h$ index strata. The treatment effect in stratum $h$ is:

$$
\widehat{\tau}_h = \bar{Y}_{T,h} - \bar{Y}_{C,h}
$$

The overall treatment effect is a weighted average:

$$
\widehat{\tau} = \sum_h w_h \widehat{\tau}_h
$$

where $w_h$ is the share of the experiment population in stratum $h$.

This approach has two benefits:

- It preserves the intended population mix
- It reduces noise from random imbalance across strata

For example, if treatment accidentally has more high-value returning users, a naive comparison may overestimate the treatment effect. Stratified analysis compares treatment and control within new users and within returning users, then combines the comparisons using fixed weights.

The result is often more stable.

## Regression Adjustment

Regression adjustment uses pre-treatment variables to improve precision after randomization.

A simple experiment analysis estimates:

$$
Y_i = \alpha + \tau T_i + \epsilon_i
$$

where:

- $Y_i$ is the outcome
- $T_i$ is the treatment indicator
- $\tau$ is the treatment effect

Regression adjustment adds covariates:

$$
Y_i = \alpha + \tau T_i + \beta^\top X_i + \epsilon_i
$$

where $X_i$ contains pre-treatment variables such as past activity, country, device, or acquisition channel.

Because treatment is randomized, adding pre-treatment covariates is not needed for unbiasedness. The simple difference in means is already unbiased.

The purpose is precision.

If $X_i$ explains variation in $Y_i$, then the residual noise $\epsilon_i$ becomes smaller. A smaller residual variance leads to a smaller standard error for $\tau$.

Intuitively, regression adjustment compares the part of the outcome that is not predictable from pre-treatment covariates. If past activity, country, or device type explains some of the outcome, the treatment effect is estimated after accounting for that predictable part.

The covariates should be measured before treatment assignment or at least not affected by treatment. Adjusting for post-treatment variables can bias the effect estimate.

### Choosing Pre-Treatment Covariates

Good variance reduction depends on good covariates.

A useful covariate should be:

- Predictive of the outcome
- Measured before treatment
- Available for both treatment and control
- Not affected by treatment
- Stable and reliably logged

Common choices include:

- Pre-period value of the same metric
- Historical activity level
- Historical purchase frequency
- Country or region
- Device type
- Acquisition channel
- User tenure
- Seller or creator tier

The best covariate is often the pre-period version of the outcome metric.

For example:

- Use last week's watch time to adjust this week's watch time
- Use last month's GMV to adjust this month's GMV
- Use previous order frequency to adjust experiment-period order frequency

But not every metric has a useful pre-period version. New users may have no history. New product surfaces may have no historical data. In those cases, stratification by available attributes or regression adjustment with other pre-treatment variables may help.

## CUPED

CUPED stands for Controlled-experiment Using Pre-Experiment Data.

The method is widely used in online experiments because many user outcomes are strongly correlated over time. A user who watched many videos last week is likely to watch many videos this week. A user who spent a lot last month is more likely to spend this month.

CUPED uses a pre-period metric to reduce variance in the experiment-period outcome.

CUPED is closely related to regression adjustment. In fact, it can be viewed as a special case where the covariate is usually a pre-period version of the outcome metric.

The difference is mostly presentation:

- Regression adjustment is usually written as a model that adds covariates.
- CUPED is usually written as a transformation of the outcome, followed by a simple treatment-control comparison.

Both methods use pre-treatment information to remove predictable variation from the outcome. CUPED is popular in online experiments because the pre-period version of the same metric is often one of the strongest predictors of the experiment-period metric.

Let:

- $Y_i$ be the experiment-period outcome
- $X_i$ be the pre-period version of the same or related metric

CUPED creates an adjusted outcome:

$$
Y_i^{\text{CUPED}} = Y_i - \theta (X_i - \bar{X})
$$

where:

$$
\theta = \frac{\text{Cov}(Y, X)}{\text{Var}(X)}
$$

Then the experiment compares the adjusted outcome between treatment and control.

The CUPED estimate can differ slightly from the raw treatment-control difference. The adjusted effect is:

$$
\widehat{\tau}_{\text{CUPED}}
=
(\bar{Y}_T - \bar{Y}_C)
-
\theta(\bar{X}_T - \bar{X}_C)
$$

If treatment and control are perfectly balanced on the pre-period covariate $X$, then $\bar{X}_T - \bar{X}_C = 0$ and the adjusted estimate equals the raw difference. In a finite experiment, the two groups may not be perfectly balanced, so CUPED can slightly adjust the point estimate while reducing variance.

The intuition is simple. If a user had unusually high activity before the experiment, some of their high activity during the experiment was predictable. CUPED subtracts out the predictable part and compares what remains.

The variance of the CUPED-adjusted outcome is:

$$
\text{Var}(Y^{\text{CUPED}})
=
\text{Var}(Y)(1 - \rho^2)
$$

where $\rho$ is the correlation between the pre-period metric $X$ and the experiment-period outcome $Y$.

This equation is the heart of CUPED.

If $X$ and $Y$ are uncorrelated, then $\rho = 0$, and CUPED does not reduce variance.

If $X$ and $Y$ are highly correlated, CUPED can reduce variance substantially.

| Correlation Between Pre-Period and Experiment Outcome | Variance Remaining After CUPED | Variance Reduction |
|---:|---:|---:|
| 0.0 | 100% | 0% |
| 0.3 | 91% | 9% |
| 0.5 | 75% | 25% |
| 0.7 | 51% | 49% |
| 0.9 | 19% | 81% |

This is why pre-period metrics are so valuable in experimentation platforms. They can turn noisy user behavior into a more precise treatment estimate.

Suppose a video app tests a new recommendation model and the outcome is watch time per user. Some users usually watch for two minutes, while others usually watch for two hours. If pre-period watch time strongly predicts experiment-period watch time, CUPED compares users after adjusting for their own baseline behavior. The treatment-control comparison becomes less about differences between heavy and light users and more about changes relative to what each user was expected to do.

## What Variance Reduction Cannot Fix

Variance reduction makes estimates more precise. It does not make a biased experiment unbiased.

The distinction is important:

- Bias is about whether the estimate is centered on the right answer.
- Variance is about how spread out the estimate is.

Variance reduction mainly addresses variance. It does not fix broken randomization, post-treatment adjustment, logging differences, or interference.

Common failure modes include:

**Broken randomization**

If treatment assignment is not random or the traffic split is wrong, CUPED or regression adjustment cannot fully rescue the experiment.

**Post-treatment covariates**

If a covariate is affected by treatment, adjusting for it can remove part of the treatment effect or introduce bias. For example, if a recommendation change affects click-through rate, then adjusting for experiment-period clicks when measuring purchases can bias the estimated purchase effect.

**Logging differences**

If treatment and control have different exposure or outcome logging, variance reduction may make the wrong estimate look more precise.

**Interference**

If treatment users affect control users, blocking may reduce noise but does not restore a clean no-treatment comparison.

**Weak or unstable predictors**

If pre-period behavior weakly predicts experiment-period behavior, methods such as CUPED will help little. CUPED works best when past and future outcomes are strongly correlated.

## Practical Workflow

A practical variance reduction workflow looks like this:

1. Choose the primary metric and randomization unit.
2. Identify pre-treatment variables that predict the primary metric.
3. Check correlation between covariates and the outcome using historical data.
4. Decide before the experiment which adjustment method will be used.
5. Run the experiment and verify randomization, logging, and sample ratio.
6. Estimate both the unadjusted and adjusted effects.
7. Use the adjusted estimate for the primary analysis if it was preplanned.
8. Report how much variance reduction was achieved.

Reporting both unadjusted and adjusted estimates is useful. If they point in very different directions, that may indicate imbalance, logging problems, model instability, or an analysis mistake.

## Key Takeaways

Variance reduction improves precision without requiring more traffic.

It works by removing predictable variation unrelated to treatment.

Stratification and blocking are design-stage methods: they randomize within important groups to improve balance.

Regression adjustment and CUPED are analysis-stage methods: they use pre-treatment covariates to reduce residual variance.

CUPED is a special case of regression adjustment that usually uses a pre-period version of the outcome metric.

Adjusted estimates can differ slightly from raw treatment-control differences when treatment and control are imbalanced on pre-treatment covariates in the realized sample.

Covariates should be measured before treatment and should not be affected by treatment.

Variance reduction does not fix broken randomization, bad logging, post-treatment conditioning, or interference.

The goal is a sharper answer to the same causal question.
