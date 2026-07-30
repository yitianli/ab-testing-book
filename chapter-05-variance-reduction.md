# Variance Reduction

Some experiments fail not because the idea is weak, but because the data is noisy.

Imagine a product change that increases revenue per user by 1%. If user spending varies wildly, that 1% lift may be hard to detect. A few unusually large purchases can dominate the metric. Some users may never purchase at all. Others may buy frequently regardless of treatment. The treatment effect is there, but it is buried inside natural user-to-user variation.

One solution is to get more traffic. More users reduce uncertainty.

But traffic is expensive. Experiments take time, and many products have limited eligible users. Variance reduction methods try to answer a practical question:

> Can we make the experiment more precise without increasing sample size?

The answer is often yes.

Variance reduction uses information we already know about users, markets, items, or time periods to remove predictable noise from the outcome. If a user's past behavior strongly predicts their future behavior, then adjusting for past behavior can make the treatment-control comparison sharper.

This chapter covers four common approaches:

- Stratification
- Blocking
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

## Stratified Analysis

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

## Blocking

Blocking is closely related to stratification. The terms are sometimes used interchangeably.

In practice, blocking often refers to creating small groups of similar units and randomizing within each block.

For example:

- Pair similar cities and assign one to treatment, one to control
- Group restaurants by order volume and randomize within each group
- Match creators by historical GMV and randomize within each matched set
- Randomize time windows within each day of week and hour of day

Blocking is especially useful when the randomization unit is large and the number of units is small.

Suppose a food delivery platform runs a geo experiment across 20 cities. Cities differ greatly in order volume, courier supply, weather, and restaurant density. A simple random assignment of 10 cities to treatment and 10 to control may create imbalance.

Blocking can improve the design:

- Pair cities with similar historical order volume and cancellation rate
- Randomize one city in each pair to treatment
- Compare treatment and control within pairs

This reduces the risk that the treatment group contains mostly large, high-growth cities while the control group contains smaller or slower markets.

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

The covariates should be measured before treatment assignment or at least not affected by treatment. Adjusting for post-treatment variables can bias the effect estimate.

## CUPED

CUPED stands for Controlled-experiment Using Pre-Experiment Data.

The method is widely used in online experiments because many user outcomes are strongly correlated over time. A user who watched many videos last week is likely to watch many videos this week. A user who spent a lot last month is more likely to spend this month.

CUPED uses a pre-period metric to reduce variance in the experiment-period outcome.

CUPED can be viewed as a special case of regression adjustment where the covariate is usually a pre-period version of the outcome metric, and the adjustment coefficient is chosen to minimize variance.

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

The intuition is simple. If a user had unusually high activity before the experiment, some of their high activity during the experiment was predictable. CUPED subtracts out the predictable part and compares what remains.

## Why CUPED Reduces Variance

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

For example:

| Correlation Between Pre-Period and Experiment Outcome | Variance Remaining After CUPED | Variance Reduction |
|---:|---:|---:|
| 0.0 | 100% | 0% |
| 0.3 | 91% | 9% |
| 0.5 | 75% | 25% |
| 0.7 | 51% | 49% |
| 0.9 | 19% | 81% |

This is why pre-period metrics are so valuable in experimentation platforms. They can turn noisy user behavior into a more precise treatment estimate.

## CUPED Example

Suppose a video app tests a new recommendation model. The outcome is watch time per user during the experiment.

Users vary a lot:

- Some users usually watch for two minutes
- Some users usually watch for two hours
- The treatment effect may be small compared with these baseline differences

Pre-period watch time is strongly predictive of experiment-period watch time. CUPED uses each user's pre-period watch time to adjust their experiment-period watch time.

If a heavy user watches a lot in treatment, CUPED asks:

> Is this user watching more than expected, given their own past behavior?

If a light user watches little in control, CUPED asks:

> Is this user watching less than expected, given their own past behavior?

The treatment-control comparison becomes less about differences between heavy and light users and more about changes relative to each user's baseline.

## CUPED and Difference-in-Differences

CUPED is related in spirit to difference-in-differences, but it is not the same.

A simple pre-post difference would compare:

$$
Y_i - X_i
$$

CUPED instead compares:

$$
Y_i - \theta X_i
$$

with:

$$
\theta = \frac{\text{Cov}(Y, X)}{\text{Var}(X)}
$$

If $\theta = 1$, CUPED resembles a pre-post difference. But usually $\theta$ is estimated from the data and may be less than or greater than 1.

CUPED chooses the adjustment that minimizes variance under a linear relationship between $X$ and $Y$.

## Choosing Covariates

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

## What Can Go Wrong

Variance reduction can be powerful, but it has failure modes.

**Using post-treatment covariates**

If a covariate is affected by treatment, adjusting for it can remove part of the treatment effect or introduce bias.

For example, if a recommendation change affects click-through rate, then adjusting for experiment-period clicks when measuring purchases can bias the estimated purchase effect.

**Overfitting covariates**

Adding many weak covariates can make analysis harder to explain and may introduce instability, especially in small experiments.

**Changing the estimand**

Some adjustments estimate a covariate-adjusted treatment effect. This is usually fine when the target is the average treatment effect in the experiment population, but the weighting and interpretation should be clear.

**Using covariates with missing or inconsistent logging**

If pre-period data is missing for some users, the missingness strategy should be defined before analysis. Dropping users with missing covariates can change the experiment population.

**Applying CUPED to unstable users**

If user behavior changes rapidly for reasons unrelated to treatment, pre-period behavior may be a weak predictor. CUPED helps most when past and future behavior are correlated.

## Variance Reduction Does Not Fix Bias

Variance reduction makes estimates more precise. It does not make a biased experiment unbiased.

If randomization is broken, CUPED does not fix it.

If treatment and control have different logging behavior, regression adjustment does not fix it.

If the analysis conditions on a post-treatment variable, stratification does not automatically fix it.

If interference contaminates control users, blocking may reduce noise but does not restore a clean no-treatment comparison.

This distinction matters:

- Bias is about whether the estimate is centered on the right answer
- Variance is about how spread out the estimate is

Variance reduction mainly addresses the second problem.

## A Product Example

Suppose an e-commerce platform tests a new recommendation ranking model. The primary metric is GMV per user over a two-week experiment.

GMV per user is noisy. Many users buy nothing, some users buy small items, and a few users make large purchases.

A naive experiment analysis compares:

$$
\bar{Y}_T - \bar{Y}_C
$$

where $Y_i$ is experiment-period GMV per user.

To reduce variance, the team uses last month's GMV per user as a CUPED covariate:

$$
Y_i^{\text{adj}} =
Y_i - \theta(X_i - \bar{X})
$$

where $X_i$ is pre-period GMV per user.

If pre-period GMV and experiment-period GMV have correlation 0.6, then the approximate variance reduction is:

$$
\rho^2 = 0.6^2 = 0.36
$$

So the adjusted metric has about 36% less variance:

$$
1 - \rho^2 = 64\%
$$

This does not change the product question. The experiment still asks whether the ranking model increases GMV per user. CUPED simply makes the estimate more precise by removing predictable user-level spending differences.

The team should still monitor guardrails such as refund rate, return rate, negative feedback, retention, and creator exposure concentration. A more precise estimate of GMV does not remove the need to understand business tradeoffs.

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

Stratification randomizes within important groups to improve balance.

Blocking is especially useful when randomization units are large or limited in number.

Regression adjustment controls for pre-treatment covariates to reduce residual variance.

CUPED adjusts outcomes using pre-period behavior and can substantially reduce variance when past and future outcomes are correlated.

Covariates should be measured before treatment and should not be affected by treatment.

Variance reduction does not fix broken randomization, bad logging, post-treatment conditioning, or interference.

The goal is a sharper answer to the same causal question.
