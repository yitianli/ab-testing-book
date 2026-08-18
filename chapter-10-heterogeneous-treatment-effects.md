# Heterogeneous Treatment Effects

Most experiment reports begin with an average treatment effect.

For example:

> The new recommendation model increased purchase conversion by 2%.

That number is useful. It tells the team what happened on average across the experiment population. But the average can hide the product story.

The same treatment may help one group, do nothing for another group, and harm a third group:

- Returning buyers may benefit, while new visitors are confused.
- High-intent users may convert more, while low-intent users ignore the change.
- One product category may improve, while another category declines.
- A ranking model may increase total GMV while reducing exposure for small sellers.

Heterogeneous treatment effect analysis, or HTE analysis, asks:

> For whom, where, or under what conditions does the treatment work better or worse?

The target is usually the conditional average treatment effect, or CATE:

$$
\tau(x)
=
E[Y(1)-Y(0) \mid X=x]
$$

Here, $X$ represents user, item, market, or context features.

This chapter is organized around the purpose of HTE analysis. We start with product decisions: targeted rollout, personalization, guardrail protection, and product learning. Then we discuss predefined segment analysis with experimental data. After that, we move to data-driven methods: interpretable methods for discovering segments, adjustment methods for observational data, and flexible methods for CATE prediction and uplift modeling.

## Why Study HTE?

HTE analysis is useful when the average treatment effect is not enough for the decision.

There are four common reasons.

**Targeted rollout**

Sometimes a treatment works well for some users and poorly for others. A company may not want to launch the same experience to everyone.

For example, a recommendation model may improve conversion for returning buyers because it can use purchase history, but it may hurt new users because it has little information about them. A targeted rollout may launch the model to returning buyers and use a fallback model for new users.

**Personalization**

Some treatments are naturally personalized decisions.

Examples:

- Which users should receive a coupon?
- Which riders should receive an incentive?
- Which sellers should receive extra exposure?
- Which users should receive a notification?
- Which ranking model should be used for this context?

In these cases, the question is not only whether the treatment works on average. The question is whether the platform can predict who is most likely to benefit.

**Guardrail protection**

The average primary metric may improve while harm is concentrated in a specific group.

Examples:

- A video ranking model increases watch time but increases reported content among teenage users.
- A marketplace ranking model increases GMV but reduces exposure for small sellers.
- A pricing change increases revenue but hurts low-income regions.
- An onboarding change increases signups but lowers paid conversion for users from broad paid acquisition channels.

If harm is concentrated in an important segment, the launch decision may change even when the average result looks good.

**Product learning**

HTE analysis can help explain why a treatment worked.

If a repeat-purchase ranking signal works mainly for users with prior purchase history, that supports the mechanism. If it works equally well for new users with no history, the team may need a different explanation.

This is why HTE analysis is not only a targeting tool. It is also a product-learning tool.

## Predefined Segment Analysis

The cleanest HTE analysis starts with segments chosen before looking at the experiment result.

Examples:

- New versus returning users for onboarding
- Mobile versus desktop for a layout change
- Prior purchasers versus non-purchasers for recommendations
- High-demand versus low-demand markets for delivery incentives
- High-repeat versus low-repeat categories for repeat-purchase ranking

Predefined segments are more credible because they are chosen from product reasoning, not because they happened to show an exciting result.

The guiding question is:

> Where should the treatment mechanism plausibly differ?

If a recommendation model uses purchase history, prior-purchase segments are natural. If a checkout change simplifies payment, device type or payment method may be natural. If a delivery treatment affects capacity, high-demand and low-demand city-hours may matter.

Bad HTE analysis starts with a different question:

> Which slice of the data gives us the most exciting story?

That is how segment analysis becomes a false-positive machine. But this does not mean all segments have to be predefined. Formal exploratory methods, such as causal trees, also search over possible segments, but they do so with explicit rules, complexity control, and validation.

### Estimating Segment Effects

With randomized experiment data, segment-level HTE analysis is conceptually simple. Within each segment, compare treatment and control:

$$
\widehat{\tau}_s
=
\bar{Y}_{T=1,S=s}
-
\bar{Y}_{T=0,S=s}
$$

For example:

| Segment | Control CVR | Treatment CVR | Estimated Difference |
|---|---:|---:|---:|
| All users | 10.0% | 11.0% | +1.0 pp |
| Prior purchase users | 15.0% | 18.0% | +3.0 pp |
| No prior purchase users | 8.0% | 8.2% | +0.2 pp |

The average effect is positive, but the product interpretation is richer. The treatment appears much more useful for users with prior purchase history.

A good segment table should include sample size and uncertainty:

| Segment | Users | Control Mean | Treatment Mean | Difference | Confidence Interval |
|---|---:|---:|---:|---:|---:|
| New users | 120,000 | 10.0% | 9.8% | -0.2 pp | [-0.5 pp, +0.1 pp] |
| Returning users | 380,000 | 18.0% | 18.7% | +0.7 pp | [+0.2 pp, +1.2 pp] |

The confidence interval prevents the reader from overreacting to noisy small segments.

### Do Not Compare Significance Labels

A common mistake is to say:

> The treatment is significant for segment A but not significant for segment B, so the treatment effect is different between A and B.

This is not necessarily true.

For example:

| Segment | Estimated Lift | p-value |
|---|---:|---:|
| Segment A | +2.0% | 0.04 |
| Segment B | +1.8% | 0.08 |

The treatment may be statistically significant in A and not significant in B, but the two estimated effects are almost the same. The difference in p-values may come from different sample sizes or different variances.

The correct question is:

> Is the treatment effect meaningfully different between the two segments?

That requires an interaction test.

### Interaction Tests

Suppose $S_i$ indicates whether unit $i$ belongs to a segment.

- $S_i = 1$: the unit is in the segment
- $S_i = 0$: the unit is in the reference group

A simple regression is:

$$
Y_i
=
\alpha
+
\tau T_i
+
\beta S_i
+
\delta T_iS_i
+
\epsilon_i
$$

Here:

- $\tau$ is the treatment effect for the reference group, where $S_i=0$.
- $\delta$ is the additional treatment effect for the segment where $S_i=1$.
- $\tau+\delta$ is the treatment effect for the segment where $S_i=1$.

The interaction term $\delta$ answers:

> Is the treatment effect different across the two segments?

For example, suppose:

- Treatment effect for new users: -1%
- Treatment effect for returning users: +3%

The segment difference is:

$$
3\% - (-1\%) = 4\%
$$

The interaction test evaluates whether that 4 percentage point difference is larger than what would be expected from random noise.

This is usually better than testing each segment separately and comparing significance labels.

### Segment Analysis Is Not Free

HTE analysis creates a multiple testing problem.

If the team checks 50 segments, some segments may look significant by chance even if the treatment effect is the same everywhere. This is the same issue discussed in Chapter 5, but segment analysis makes it especially tempting because every surprising segment can be turned into a story.

For predefined segments:

- Limit the number of primary segments.
- Report confidence intervals, not only p-values.
- Use interaction tests for segment differences.
- Correct for multiple testing when many segment tests can independently change the decision.

For exploratory segments:

- Treat findings as hypotheses.
- Look for stable patterns across related metrics.
- Validate important findings in a follow-up experiment.
- Avoid launching a highly targeted policy based only on one noisy slice.

Segment analysis also reduces effective sample size. Even if the overall experiment is well powered, individual segments may not be. Small segments create noisy estimates, wide confidence intervals, and extreme-looking effects that may not replicate.

### Guardrail Heterogeneity

Heterogeneity is not only about finding upside.

Sometimes the average primary metric improves, but a guardrail worsens for a specific group.

For example:

- A ranking change increases overall watch time, but reported content increases among teenage users.
- A marketplace model increases GMV, but creator diversity declines for small creators.
- A checkout change increases conversion, but refund rate rises in one product category.

For important guardrails, teams should predefine critical slices.

Examples:

- New users for onboarding quality
- Small sellers for marketplace fairness
- High-risk content categories for safety
- Low-income regions for pricing changes
- Long-time subscribers for retention impact

The goal is not to inspect every possible slice. The goal is to protect segments where harm would materially change the decision.

## From HTE to Targeted Rollout

HTE analysis often leads to a tempting conclusion:

> Launch the treatment only to the segments where it works.

Sometimes this is right. But targeted rollout needs caution.

First, the segment effect must be credible. If the segment was discovered after searching many slices, it may not replicate.

Second, the treatment must remain effective when targeted. A segment effect measured inside a broad experiment may change after the product is launched only to that segment.

Third, targeting can create product or fairness concerns. If only some users receive a better experience, the company should understand whether that is acceptable.

A responsible path is:

1. Use HTE analysis to identify promising or risky segments.
2. Check whether the segment pattern matches the product mechanism.
3. Validate important exploratory findings in a follow-up experiment.
4. Launch selectively only when the evidence is strong enough.
5. Monitor whether the effect persists after targeting.

## Interpretable Exploratory HTE

Predefined segment analysis is strongest when the team already knows which segments matter. But sometimes the treatment effect may depend on combinations of features that are hard to specify in advance.

Examples:

- The effect of a recommendation model may depend on prior purchase history, category preference, and seller density.
- The effect of a coupon may depend on price sensitivity, order frequency, and acquisition channel.
- The effect of a delivery incentive may depend on time of day, rider supply, distance, and local demand.

In these cases, the team may use data-driven methods to discover possible HTE patterns.

The key distinction is interpretability. Some models estimate CATE in a way that produces product-readable segments. Others estimate CATE more flexibly, but their results are harder to explain.

This section focuses on interpretable exploratory methods.

### Linear Regression With Interactions

Linear regression can estimate heterogeneous treatment effects if it includes treatment-feature interactions.

A regression without interactions,

$$
Y_i
=
\alpha
+
\tau T_i
+
X_i^\top\beta
+
\epsilon_i
$$

estimates one constant treatment effect, $\tau$.

To allow the treatment effect to vary with features, add interactions:

$$
Y_i
=
\alpha
+
\tau T_i
+
X_i^\top\beta
+
T_iX_i^\top\gamma
+
\epsilon_i
$$

Then the treatment effect is:

$$
\tau(x)
=
\tau
+
x^\top\gamma
$$

This is the same idea as the earlier interaction test, but written for many features. The vector $\gamma$ plays the role of multiple interaction effects; with one binary segment, it reduces to the earlier $\delta$.

This model is interpretable when $X$ contains a small number of meaningful features. For example, the coefficient on $T_i \times \text{PriorPurchase}_i$ tells the team whether the treatment effect differs for users with prior purchase history.

The strength of interaction regression is clarity. The weakness is that the analyst must choose the interactions. If many interactions are tested after seeing the result, the analysis becomes exploratory and faces the same multiple testing problem as segment search.

### Causal Trees

A causal tree is a tree-based method for finding interpretable segments with different treatment effects. Athey and Imbens (2016) formalize this idea as recursive partitioning for heterogeneous causal effects.

Unlike a linear interaction model, a causal tree is nonparametric: it does not require the analyst to specify a linear form for $\tau(x)$ or choose interactions manually. This makes it more flexible, but also more vulnerable to instability if the tree is allowed to search too freely.

It is helpful to compare it with an ordinary decision tree. A standard prediction tree searches for splits that improve outcome prediction. A causal tree searches for a different kind of split:

> Does this split create groups where the treatment effect is different?

The important subtlety is that the tree does not observe each user's individual treatment effect. For one user, the data contains only one of two potential outcomes:

- If the user is treated, we observe $Y_i(1)$.
- If the user is in control, we observe $Y_i(0)$.

The individual treatment effect would be:

$$
Y_i(1)-Y_i(0)
$$

But this quantity is never directly observed for a single user.

So a causal tree does not split users based on known individual treatment effects. It splits users based on estimated average treatment effects within candidate groups.

Conceptually, the procedure is:

1. Start with all users.
2. Try possible splits, such as prior purchase history, device type, category, or country.
3. Estimate the treatment effect on each side of the split.
4. Prefer splits where the estimated treatment effects differ meaningfully.
5. Continue splitting until the leaves are useful and still have enough data.
6. Estimate the treatment effect inside each final leaf.

For example:

| Group | Control CVR | Treatment CVR | Estimated Difference |
|---|---:|---:|---:|
| Prior purchase users | 15.0% | 18.0% | +3.0 pp |
| No prior purchase users | 8.0% | 8.2% | +0.2 pp |

The tree has not learned that a specific user's treatment effect is exactly +3.0 pp or +0.2 pp. It has learned that, on average, the treatment appears more effective among users with prior purchase history.

In an experiment, this leaf-level comparison is straightforward because treatment is randomized:

$$
\widehat{\tau}_{\text{leaf}}
=
\bar{Y}_{T=1,\text{leaf}}
-
\bar{Y}_{T=0,\text{leaf}}
$$

A fitted causal tree can also be used for CATE prediction. A new unit is assigned to a leaf, and the leaf's estimated treatment effect becomes the prediction:

$$
\widehat{\tau}(x)
=
\widehat{\tau}_{\text{leaf}(x)}
$$

The prediction is interpretable but coarse. All units in the same leaf receive the same predicted treatment effect.

Causal trees are useful when interpretability matters. They can turn a complex HTE problem into a small number of readable segment rules.

The main risk is instability. If the tree searches many possible splits, it can find patterns that are partly noise. Small changes in the data can sometimes produce a different tree. For this reason, causal trees are often used with honest estimation: one part of the data chooses the tree structure, and another part estimates the treatment effect inside each leaf.

## HTE With Observational Data

Everything above is easiest with randomized experiment data. With observational data, treatment was not randomly assigned. Treated and control users may differ before treatment.

For example, suppose a platform gives retention coupons mostly to users who look likely to churn. If coupon users have lower retention than non-coupon users, a naive comparison may make the coupon look harmful even if it helped. The treated users were already different.

With observational data, HTE analysis needs adjustment before it can be interpreted causally.

The common identifying assumption is conditional ignorability:

$$
(Y(1),Y(0)) \perp T \mid X
$$

This means that, after controlling for observed features $X$, treatment assignment is as good as random. This is a strong assumption because it rules out important unobserved confounders.

Another required condition is overlap:

$$
0 < P(T=1 \mid X=x) < 1
$$

This means that, for the types of users being compared, the data must contain both treated and control examples. If every high-risk user received a coupon and no similar high-risk user was untreated, the model cannot reliably estimate what would have happened without the coupon.

Two common adjustment tools are inverse propensity weighting and doubly robust estimation.

### Inverse Propensity Weighting

Inverse propensity weighting, or IPW, starts by estimating the propensity score:

$$
e(x) = P(T=1 \mid X=x)
$$

This is the probability that a unit receives treatment given its observed features.

Then each observation is weighted by the inverse of the probability of receiving the treatment it actually received.

For treated units:

$$
w_i = \frac{1}{e(X_i)}
$$

For control units:

$$
w_i = \frac{1}{1-e(X_i)}
$$

The weighting tries to make treated and control groups more comparable in observed covariates.

For an average treatment effect, a simple IPW estimator is:

$$
\widehat{\tau}_{\text{IPW}}
=
\frac{1}{n}
\sum_i
\left[
\frac{T_iY_i}{\widehat{e}(X_i)}
-
\frac{(1-T_i)Y_i}{1-\widehat{e}(X_i)}
\right]
$$

The same idea can be written as a pseudo-treatment-effect signal:

$$
\widetilde{\tau}^{\text{IPW}}_i
=
\frac{T_iY_i}{\widehat{e}(X_i)}
-
\frac{(1-T_i)Y_i}{1-\widehat{e}(X_i)}
$$

The main weakness is instability. If $\widehat{e}(X_i)$ is close to 0 or 1, the inverse weight can become very large. A few observations can then dominate the estimate. In practice, analysts often check covariate balance after weighting, trim extreme propensity scores, cap very large weights, use stabilized weights, and avoid estimating effects where overlap is poor.

### Doubly Robust Signals

Doubly robust methods combine two kinds of models:

- A propensity model, $e(x)$
- Outcome models, $\mu_1(x)$ and $\mu_0(x)$

The outcome models predict expected outcomes under treatment and control:

$$
\mu_1(x) = E[Y \mid T=1, X=x]
$$

$$
\mu_0(x) = E[Y \mid T=0, X=x]
$$

A common doubly robust pseudo-treatment-effect signal is:

$$
\widetilde{\tau}^{\text{DR}}_i
=
\widehat{\mu}_1(X_i)
-
\widehat{\mu}_0(X_i)
+
\frac{T_i(Y_i-\widehat{\mu}_1(X_i))}{\widehat{e}(X_i)}
-
\frac{(1-T_i)(Y_i-\widehat{\mu}_0(X_i))}{1-\widehat{e}(X_i)}
$$

The first part, $\widehat{\mu}_1(X_i)-\widehat{\mu}_0(X_i)$, is the model-predicted treatment effect for a unit with features $X_i$. The remaining terms are residual corrections. They compare the observed outcome with the predicted outcome and weight that correction by the propensity score.

The intuition is:

- The outcome model says, "Based on this unit's features, here is what I predict under treatment and control."
- The propensity model says, "Because treatment assignment was not random, correct the prediction errors using how likely this unit was to receive treatment."

This is called doubly robust because the estimator can still be consistent if either the propensity model is correct or the outcome model is correct. It does not require both to be perfect. But if both are badly wrong, the estimate can still be biased.

If the analyst averages this signal across all units, the result is a doubly robust estimate of the average treatment effect:

$$
\widehat{\tau}_{\text{DR}}
=
\frac{1}{n}
\sum_i
\widetilde{\tau}^{\text{DR}}_i
$$

If the analyst averages it inside a segment or leaf, the result is a doubly robust estimate of the segment-level CATE:

$$
\widehat{\tau}_{\text{segment}}
=
\frac{1}{n_{\text{segment}}}
\sum_{i \in \text{segment}}
\widetilde{\tau}^{\text{DR}}_i
$$

This connects observational adjustment to the methods above. With observational data, a causal tree should search over adjusted treatment-effect signals rather than raw treated-control differences.

The important warning is that these methods adjust only for observed confounders. If important unobserved factors affect both treatment assignment and the outcome, observational HTE analysis can still be biased.

## Flexible CATE Prediction

Interpretable methods can also be used for CATE prediction. A linear interaction model predicts treatment effects through its regression formula, and a causal tree predicts treatment effects by assigning a new unit to a leaf.

But these predictions are usually simple: linear in selected features, or piecewise constant by leaf. When the goal is targeting or personalization, and the treatment effect may depend on many nonlinear feature interactions, more flexible methods can be useful.

For example:

- A promotion should be sent only to users with high predicted incremental purchase probability.
- A notification should be sent only when predicted incremental engagement is high enough to justify annoyance risk.
- A ranking system should use the model that has the highest predicted incremental value for a context.

These decisions require a CATE prediction function:

$$
\widehat{\tau}(x)
$$

The tradeoff is that flexible methods may improve prediction while making the treatment-effect pattern harder to explain.

### Causal Forests

A causal forest extends the causal tree idea by building many trees and averaging them. Wager and Athey (2018) develop causal forests as a random-forest-style method for estimating heterogeneous treatment effects.

This is similar to how a random forest improves on a single prediction tree. Instead of relying on one tree, the forest combines many trees built from different samples and feature splits.

The output is usually an estimated treatment effect for each unit:

$$
\widehat{\tau}(X_i)
$$

Causal forests are more stable and flexible than a single causal tree, but they are less transparent. They usually do not naturally produce a small set of product-readable segment rules. Analysts can inspect feature importance, uplift quantiles, partial dependence plots, or fit a simpler surrogate tree to the forest predictions, but these are post-hoc interpretation tools.

### Meta-Learners

Meta-learners estimate CATE by combining standard prediction models.

They are called "meta" learners because the framework is separate from the underlying model. The base model could be a linear model, random forest, gradient boosted tree, neural network, or another supervised learning method.

Common examples include S-learners, T-learners, X-learners, R-learners, and DR-learners. Künzel et al. (2019) present a unified meta-learner framework and introduce the X-learner. Nie and Wager (2021) develop the R-learner, and Kennedy (2023) studies doubly robust estimation for heterogeneous causal effects.

**S-learner**

The S-learner fits one model using both treatment and control data:

$$
\widehat{\mu}(x,t)
=
\widehat{E}[Y \mid X=x, T=t]
$$

Then it estimates:

$$
\widehat{\tau}(x)
=
\widehat{\mu}(x,1)
-
\widehat{\mu}(x,0)
$$

In an S-learner, treatment assignment $T$ is an input feature. The method is simple and stable, but it may understate treatment heterogeneity if the model focuses mostly on predicting baseline outcomes and treats $T$ as a minor feature.

The linear regression model with treatment interactions discussed earlier can be viewed as a simple parametric S-learner. In practice, the same S-learner idea can be implemented with more flexible prediction models such as random forests, gradient boosted trees, or neural networks.

**T-learner**

The T-learner fits two separate outcome models.

For treated units:

$$
\widehat{\mu}_1(x)
=
\widehat{E}[Y \mid T=1, X=x]
$$

For control units:

$$
\widehat{\mu}_0(x)
=
\widehat{E}[Y \mid T=0, X=x]
$$

Then it estimates:

$$
\widehat{\tau}(x)
=
\widehat{\mu}_1(x)
-
\widehat{\mu}_0(x)
$$

In a T-learner, treatment assignment $T$ is used to split the data. It does not appear as a feature inside each separate model.

The T-learner works best when both treatment and control groups have enough data and the outcome patterns under treatment and control may be quite different. Its main weakness is instability when one group is small.

**X-learner**

The X-learner is designed for settings where treatment and control groups are imbalanced or treatment effects vary a lot.

The idea is to first learn good outcome models, then use those models to impute treatment-effect signals for the units where the missing potential outcome is not observed.

First, fit the same two outcome models as a T-learner:

$$
\widehat{\mu}_1(x)
=
\widehat{E}[Y \mid T=1, X=x]
$$

$$
\widehat{\mu}_0(x)
=
\widehat{E}[Y \mid T=0, X=x]
$$

For treated units, the observed outcome is $Y_i(1)$. The missing outcome is what would have happened under control. The X-learner imputes a treatment-effect signal for treated units as:

$$
\widetilde{\tau}_{1i}
=
Y_i
-
\widehat{\mu}_0(X_i)
\quad
\text{for } T_i=1
$$

For control units, the observed outcome is $Y_i(0)$. The missing outcome is what would have happened under treatment. The X-learner imputes a treatment-effect signal for control units as:

$$
\widetilde{\tau}_{0i}
=
\widehat{\mu}_1(X_i)
-
Y_i
\quad
\text{for } T_i=0
$$

Then it fits two treatment-effect models:

$$
\widehat{\tau}_1(x)
\approx
E[\widetilde{\tau}_{1i} \mid X_i=x, T_i=1]
$$

$$
\widehat{\tau}_0(x)
\approx
E[\widetilde{\tau}_{0i} \mid X_i=x, T_i=0]
$$

Finally, it combines the two predictions, often using weights based on the propensity score:

$$
\widehat{\tau}(x)
=
g(x)\widehat{\tau}_0(x)
+
\left(1-g(x)\right)\widehat{\tau}_1(x)
$$

where $g(x)$ is often chosen to reflect the probability of treatment or the relative reliability of the two models.

The X-learner can work well when one group is much larger than the other. For example, if only 10% of users received a coupon and 90% did not, the large control group can help build a strong estimate of $\widehat{\mu}_0(x)$, which then helps impute treatment-effect signals for the treated users.

Its main weakness is complexity: there are more modeling steps, more chances for error, and more implementation choices. Mistakes in the first-stage outcome models can also affect the imputed treatment-effect signals.

**R-learner**

The R-learner starts by removing the part of the outcome and treatment assignment that can already be predicted from user features.

It first estimates:

$$
\widehat{m}(x)=E[Y \mid X=x]
$$

and:

$$
\widehat{e}(x)=P(T=1 \mid X=x)
$$

Then it learns the treatment effect from the residualized relationship:

$$
Y_i-\widehat{m}(X_i)
\approx
\left(T_i-\widehat{e}(X_i)\right)\tau(X_i)
$$

It may look like we can simply divide both sides and estimate an individual treatment-effect signal:

$$
\widetilde{\tau}^{R}_i
=
\frac{Y_i-\widehat{m}(X_i)}
{T_i-\widehat{e}(X_i)}
$$

This ratio is the right intuition, but it can be unstable. If $T_i-\widehat{e}(X_i)$ is close to zero, the denominator is small and the implied treatment-effect signal can become very noisy.

For this reason, the R-learner usually does not use the raw ratio directly. Instead, it estimates the function $\tau(x)$ by solving a weighted prediction problem:

$$
\min_{\tau(\cdot)}
\sum_i
\left[
Y_i-\widehat{m}(X_i)
-
\left(T_i-\widehat{e}(X_i)\right)\tau(X_i)
\right]^2
$$

This can be understood as fitting the ratio-like signal above on $X_i$, but with observations weighted by how informative their residualized treatment variation is. Observations with larger $\left(T_i-\widehat{e}(X_i)\right)^2$ receive more weight.

The intuition is close to regression adjustment. The model asks: after removing baseline outcome differences and treatment-assignment differences explained by $X$, how does the remaining treatment variation relate to the remaining outcome variation?

The R-learner is useful when flexible models are used for the baseline outcome and propensity score. Its main weakness is that it is less intuitive than S-, T-, or X-learners and requires careful implementation, usually with sample splitting or cross-fitting.

**DR-learner**

The DR-learner uses the doubly robust pseudo-treatment-effect signal from the previous section as the target in a prediction problem.

The signal is:

$$
\widetilde{\tau}^{\text{DR}}_i
=
\widehat{\mu}_1(X_i)
-
\widehat{\mu}_0(X_i)
+
\frac{T_i(Y_i-\widehat{\mu}_1(X_i))}{\widehat{e}(X_i)}
-
\frac{(1-T_i)(Y_i-\widehat{\mu}_0(X_i))}{1-\widehat{e}(X_i)}
$$

It learns:

$$
\tau(x)
=
E[\widetilde{\tau}^{\text{DR}}_i \mid X_i=x]
$$

Although $\widetilde{\tau}^{\text{DR}}_i$ contains functions of $X_i$, it is not itself a prediction rule for new users. After plugging in the observed $T_i$ and $Y_i$, it becomes one noisy scalar for unit $i$. For a new user, $T_i$ and $Y_i$ are not observed, so the learner needs a second-stage model that maps $X$ to the expected pseudo-treatment-effect signal.

This is the key difference from using the same signal inside a causal tree. A causal tree averages $\widetilde{\tau}^{\text{DR}}_i$ inside leaves to discover interpretable segments. A DR-learner regresses $\widetilde{\tau}^{\text{DR}}_i$ on $X_i$ to predict CATE for new or fine-grained units.

The DR-learner is useful because the pseudo-treatment-effect signal combines outcome prediction with propensity weighting. Its main weakness is complexity: it requires several nuisance models, careful cross-fitting, and validation to avoid overfitting noisy treatment-effect signals.

A practical comparison is:

| Learner | Basic Idea | Works Best When | Main Strength | Main Weakness |
|---|---|---|---|---|
| S-learner | Fit one outcome model with treatment as a feature | Treatment effect is simple; data is limited | Stable and easy to implement | May understate treatment heterogeneity |
| T-learner | Fit separate treated and control outcome models | Both groups have enough data; treatment and control patterns may differ | Flexible response surfaces | Noisy when one group is small |
| X-learner | Impute treatment-effect signals, then model them | Treatment and control group sizes are imbalanced | Uses the larger group efficiently | More complex and sensitive to first-stage model errors |
| R-learner | Learn treatment effects from residualized outcomes and treatment assignment | Strong baseline outcome prediction is possible; flexible CATE modeling is needed | Focuses on residual treatment variation | Less intuitive; needs cross-fitting |
| DR-learner | Learn CATE from doubly robust pseudo-treatment-effect signals | Outcome and propensity models can be estimated reasonably well | Robust if either nuisance model is correct | Complex; pseudo-treatment-effect signals can be noisy |

There is no universally best meta-learner. The choice depends on sample size, treatment-control balance, expected complexity of treatment effects, the need for interpretability, and how much validation data is available.

## From CATE Prediction to Uplift Modeling

The methods above estimate or predict CATE:

$$
\widehat{\tau}(X_i)
$$

Uplift modeling is what happens when those predictions are used for decisions. The question changes from:

> How does the treatment effect vary?

to:

> Who should receive the treatment?

This is why uplift modeling is not one specific algorithm. It is a decision use case for predicted treatment effects. Causal trees, causal forests, meta-learners, and other CATE models can all support uplift modeling if they produce reliable out-of-sample predictions of $\tau(x)$.

The key distinction is between predicting outcomes and predicting incremental effects. A normal conversion model asks, "Who is most likely to purchase?" An uplift model asks, "Whose purchase probability increases because of the treatment?"

These are not the same question. Loyal users may have high purchase probability whether or not they receive a promotion. A standard conversion model may rank them highly. But the promotion may not be incremental for them because they would have purchased anyway.

Another group may have medium baseline purchase probability but respond strongly to the promotion. That group can have higher uplift even though its raw conversion probability is lower.

This distinction matters most when treatment has a cost or downside:

- Promotions cost money.
- Notifications can annoy users.
- Ads retargeting can waste budget.
- Sales outreach uses team capacity.
- Personalized recommendations can reduce diversity if targeted poorly.

In these cases, the platform may not want to treat everyone. The goal is to treat users who are likely to be changed by the treatment, not users who would take the desired action anyway.

One useful targeting framework divides users into four conceptual groups:

| Segment Type | Behavior |
|---|---|
| Persuadables | Convert only if treated |
| Sure things | Convert whether treated or not |
| Lost causes | Do not convert either way |
| Do-not-disturbs | Convert less if treated |

The most valuable target group is usually the persuadables. The model should avoid sure things because treatment is not incremental for them. It should also avoid do-not-disturbs because treatment may cause harm.

Because uplift modeling is used for future targeting, validation must be out of sample. It is not enough for the model to explain patterns in the data used to train it. The model needs to identify high-uplift users it has not seen before.

In randomized experiments, a common validation method is uplift by decile:

1. Train an uplift model using experiment data.
2. Apply it to a holdout or validation sample.
3. Rank users by predicted uplift.
4. Split users into deciles from highest to lowest predicted uplift.
5. Estimate the actual treatment effect within each decile.
6. Check whether the highest-uplift deciles show larger incremental effects.

If the model is useful, the top deciles should have meaningfully higher observed treatment effects than the bottom deciles.

This validation evaluates incremental lift directly. It asks whether the model can rank users by treatment effect, not merely whether it can predict the outcome.

## Example: Recommendation Ranking

Suppose an e-commerce platform tests a new recommendation ranking model.

Average results:

- GMV per user: +2.0%
- Repeat-purchase GMV: +4.5%
- Refund rate: no significant change

The average result is positive. But because the model is designed to identify durable user-creator or user-seller relationships, the team should expect heterogeneity.

Natural predefined segments include:

- Users with prior purchase history versus no purchase history
- High-frequency buyers versus low-frequency buyers
- Product categories with high versus low repeat-purchase behavior
- Sellers or creators with enough historical data versus sparse history

Suppose the segment analysis shows:

| Segment | GMV per User Lift | Repeat-Purchase GMV Lift |
|---|---:|---:|
| Prior purchase history | +3.5% | +7.0% |
| No purchase history | +0.2% | +0.5% |
| High-repeat categories | +4.0% | +8.5% |
| Low-repeat categories | -0.5% | +0.1% |

This pattern supports the mechanism. The model works best where repeat-purchase signals are informative. It works less well for cold-start users and categories where repeat purchase is naturally rare.

The launch decision may be:

- Give the signal higher weight for users and categories with reliable repeat-purchase history.
- Use a weaker weight or fallback model for cold-start cases.
- Continue monitoring guardrails such as refund rate, return rate, seller concentration, and user retention.

If the team wants a more personalized strategy, it may train a CATE model to predict which user-seller contexts have the highest incremental repeat-purchase value. But if the goal is product understanding, the predefined segment table or a causal tree may be easier to explain.

## Example: Onboarding Flow

Suppose a subscription app tests a new onboarding flow.

Average results:

- Signup completion: +6%
- Trial start: +4%
- Paid conversion after trial: -3%

The average result is ambiguous. The new onboarding flow brings more people into trial, but the additional users may be lower intent.

HTE analysis can ask:

- Does the effect differ by acquisition channel?
- Does paid conversion decline mainly among low-intent channels?
- Does the effect differ for users who selected a strong preference during onboarding?
- Does retention differ among users who converted to paid?

Suppose the treatment improves signup and trial start for all channels, but paid conversion declines mainly among users from broad social ads. That suggests the new flow may reduce friction too much for low-intent users.

The product decision may be:

- Keep the new onboarding for high-intent acquisition channels.
- Add more qualification or expectation-setting for low-intent channels.
- Track downstream retention and refund behavior before full rollout.

The value of HTE analysis is not only finding where the lift is largest. It is understanding how the treatment changes the composition and quality of users.

## Practical Workflow

A practical workflow for HTE analysis is:

1. Start with the product reason why effects might differ.
2. Define a small number of predefined segments before reading the result.
3. For each segment, report effect size, sample size, baseline metric, and confidence interval.
4. Use interaction tests to compare treatment effects across predefined segments.
5. Separate confirmatory segment analysis from exploratory segment discovery.
6. Use interpretable exploratory methods, such as interaction regressions or causal trees, when the goal is product-readable segmentation.
7. Use flexible CATE prediction methods, such as causal forests or meta-learners, when the goal is targeting or personalization.
8. With observational data, adjust for confounding and check overlap before interpreting HTE causally.
9. Validate important exploratory or model-based findings out of sample.
10. Consider guardrail heterogeneity, not only primary metric heterogeneity.
11. Validate important targeting decisions in follow-up experiments when the cost or risk is high.

## Common Mistakes

**Treating the average as the whole story**

The average treatment effect can hide meaningful differences across users, items, markets, and contexts.

**Searching too many segments and telling a story afterward**

Exploratory segment analysis can generate useful ideas, but it should not be treated as confirmatory evidence.

**Comparing significance labels**

"Significant in one segment but not significant in another" does not prove the treatment effects are different. Use an interaction test.

**Ignoring sample size**

Small segments often produce noisy and extreme estimates. Always show uncertainty.

**Looking only for upside**

HTE analysis should also search for concentrated harm in important guardrail segments.

**Confusing pseudo-treatment-effect signals with true individual effects**

IPW and doubly robust signals are noisy scalars computed from observed data. They are useful because their conditional averages estimate CATE under assumptions. They are not true individual treatment effects.

**Using complex models without validation**

Flexible CATE models can overfit. They need honest validation, not just attractive uplift charts.

**Treating uplift as ordinary prediction**

A model that predicts conversion well may not predict incremental lift well. Uplift models should be evaluated on treatment-control differences, not only outcome prediction.

## Key Takeaways

- HTE analysis asks where and for whom the treatment works better or worse.
- The broad target is CATE, $\tau(x)=E[Y(1)-Y(0)\mid X=x]$.
- Predefined segment analysis is the cleanest starting point with experiment data.
- Do not compare significance labels across segments. Test the interaction directly.
- Guardrail heterogeneity matters because harm may be concentrated in specific groups.
- Interpretable methods, such as linear regressions with interactions and causal trees, are useful for discovering product-readable segments.
- Flexible methods, such as causal forests and meta-learners, are useful for CATE prediction, targeting, and personalization.
- IPW and doubly robust methods can create pseudo-treatment-effect signals, but those signals are not true individual treatment effects.
- With observational data, HTE methods need adjustment for confounding and require assumptions such as conditional ignorability and overlap.
- Uplift modeling uses predicted treatment effects for targeting or decision-making.
- The best method depends on the goal: explanation, targeting, personalization, or safer rollout.

## References

- Athey, Susan, and Guido Imbens. "Recursive Partitioning for Heterogeneous Causal Effects." *Proceedings of the National Academy of Sciences of the United States of America* 113, no. 27 (2016): 7353-7360. [https://doi.org/10.1073/pnas.1510489113](https://doi.org/10.1073/pnas.1510489113).
- Kennedy, Edward H. "Towards Optimal Doubly Robust Estimation of Heterogeneous Causal Effects." *Electronic Journal of Statistics* 17, no. 2 (2023): 3008-3049. [https://doi.org/10.1214/23-EJS2157](https://doi.org/10.1214/23-EJS2157).
- Künzel, Sören R., Jasjeet S. Sekhon, Peter J. Bickel, and Bin Yu. "Metalearners for Estimating Heterogeneous Treatment Effects Using Machine Learning." *Proceedings of the National Academy of Sciences of the United States of America* 116, no. 10 (2019): 4156-4165. [https://doi.org/10.1073/pnas.1804597116](https://doi.org/10.1073/pnas.1804597116).
- Nie, Xinkun, and Stefan Wager. "Quasi-Oracle Estimation of Heterogeneous Treatment Effects." *Biometrika* 108, no. 2 (2021): 299-319. [https://doi.org/10.1093/biomet/asaa076](https://doi.org/10.1093/biomet/asaa076).
- Wager, Stefan, and Susan Athey. "Estimation and Inference of Heterogeneous Treatment Effects Using Random Forests." *Journal of the American Statistical Association* 113, no. 523 (2018): 1228-1242. [https://doi.org/10.1080/01621459.2017.1319839](https://doi.org/10.1080/01621459.2017.1319839).
