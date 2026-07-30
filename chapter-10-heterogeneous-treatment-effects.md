# Heterogeneous Treatment Effects

Most experiment reports begin with an average treatment effect.

For example:

> The new recommendation model increased watch time by 2%.

That number is useful. It summarizes the overall effect of the treatment. It is often the first number a decision maker wants to see.

But an average can hide very different realities:

- New users may benefit, while long-time users are harmed.
- High-intent users may convert more, while low-intent users ignore the change.
- One product category may improve, while another category declines.
- A treatment may look positive overall because a large segment improves, even though a smaller but important segment gets worse.

This is the motivation for heterogeneous treatment effect analysis, often shortened to HTE analysis.

HTE analysis asks:

> For whom, where, or under what conditions does the treatment work better or worse?

This chapter is about how to answer that question responsibly. The goal is not to search until a good story appears. The goal is to understand when the average effect is hiding meaningful product differences.

The chapter moves in four steps:

1. Why heterogeneous effects matter.
2. How to choose and test segments responsibly.
3. How machine learning methods estimate more complex heterogeneity.
4. How uplift modeling turns HTE estimation into targeting decisions.

## The Average Can Hide the Product Story

Suppose an e-commerce platform tests a new recommendation model.

The average result is:

- Purchase conversion rate: +1.0%

At first, this looks like a clear win.

But the segment results are:

| Segment | Treatment Effect |
|---|---:|
| Returning buyers | +3.0% |
| New visitors | -1.5% |

The average effect is positive, but the product interpretation is more subtle. The model may be good at using past purchase history, but less helpful for users with little behavioral data.

This changes the launch conversation. The team may decide to:

- Launch only to returning buyers
- Reduce the model weight for new visitors
- Add a fallback ranking for cold-start users
- Run a follow-up experiment focused on new visitors

The average effect answers whether the treatment worked overall. HTE analysis helps explain why it worked, where it worked, and where it may need modification.

## What Counts as Heterogeneity?

Heterogeneity can appear across users, items, markets, and contexts.

Common user dimensions include:

- New versus returning users
- High-frequency versus low-frequency users
- Paid versus free users
- High-intent versus low-intent users
- Different countries, languages, or regions
- Different acquisition channels

Common item or content dimensions include:

- Product category
- Seller type
- Creator size
- Listing quality
- Price band
- Inventory depth

Common context dimensions include:

- Device type
- Time of day
- Weekday versus weekend
- Marketplace supply-demand balance
- Network density
- Prior exposure to similar features

The key is that the segment should have a plausible relationship to the treatment mechanism.

If a recommendation model uses purchase history, it is natural to check prior-purchase segments. If a checkout change simplifies payment, it is natural to check device type or payment method. If a delivery treatment affects capacity, it is natural to check high-demand and low-demand city-hours.

Good HTE analysis starts with this question:

> Where do we expect the treatment mechanism to differ?

Bad HTE analysis starts with a different question:

> Where can we find the most exciting-looking lift?

That second question is how segment analysis becomes a false-positive machine.

## Choosing Segments Before Looking

Not all segment analysis has the same evidential strength.

There are two broad types.

**Pre-specified HTE analysis**

These segments are chosen before the experiment result is analyzed.

Examples:

- New versus returning users for an onboarding experiment
- Mobile versus desktop for a layout change
- High versus low historical purchase frequency for a recommendation model
- High-demand versus low-demand markets for a delivery experiment

Pre-specified segments are more credible because the team chose them based on product reasoning, not because they happened to show a large effect.

**Exploratory HTE analysis**

These segments are searched after seeing the data.

Examples:

- Checking dozens of countries, then highlighting the one with the largest lift
- Trying many age bands, price bands, device types, and categories
- Searching for the segment where the p-value is smallest

Exploratory analysis can be useful for generating hypotheses. But it should be treated as discovery, not confirmation.

A clean report separates the two:

> Pre-specified segment analysis showed stronger effects for returning users. Exploratory analysis suggested possible weakness in electronics, which should be validated in a follow-up experiment.

## Multiple Testing and Segment Uncertainty

HTE analysis creates a multiple testing problem.

If the team checks 50 segments, some segments may look significant by chance even if the treatment effect is the same everywhere.

This is the same issue discussed in Chapter 6, but segment analysis makes it especially tempting because every surprising segment can be turned into a story.

The solution is not to forbid segment analysis. The solution is to interpret segment results with the right level of caution.

For pre-specified segments:

- Limit the number of primary segments.
- Adjust for multiple testing when many confirmatory segment tests are used.
- Report confidence intervals, not only p-values.
- Prefer interaction tests over separate within-segment significance tests.

For exploratory segments:

- Treat findings as hypotheses.
- Look for stable patterns across related metrics.
- Validate important findings in a follow-up experiment.
- Avoid launching a highly targeted policy based only on one noisy slice.

Segment analysis also reduces sample size.

Even if the overall experiment is well powered, individual segments may not be. Suppose an experiment has 1,000,000 users. If only 2% of users belong to a segment, that segment has 20,000 users before splitting treatment and control. If the outcome is rare, the number of actual outcome events may be much smaller.

Small segments create three problems:

- Estimates are noisy.
- Confidence intervals are wide.
- Extreme segment effects are more likely to be noise.

A good segment table shows effect size and uncertainty:

| Segment | Users | Control Mean | Treatment Mean | Lift | Confidence Interval |
|---|---:|---:|---:|---:|---:|
| New users | 120,000 | 10.0% | 9.8% | -2.0% | [-4.5%, +0.7%] |
| Returning users | 380,000 | 18.0% | 18.7% | +3.9% | [+1.5%, +6.4%] |

The confidence interval prevents the reader from overreacting to noisy small segments.

## Do Not Compare Significance Labels

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

## Interaction Tests

Suppose $S_i$ indicates whether user $i$ belongs to a segment.

- $S_i = 1$: user is in the segment
- $S_i = 0$: user is not in the segment

A simple regression is:

$$
Y_i
=
\alpha
+ \tau T_i
+ \beta S_i
+ \delta T_iS_i
+ \epsilon_i
$$

Here:

- $\tau$ is the treatment effect for the reference group, users with $S_i = 0$.
- $\delta$ is the additional treatment effect for users with $S_i = 1$.
- $\tau + \delta$ is the treatment effect for users with $S_i = 1$.

This is different from a simple regression without the interaction term. Without the interaction, the treatment coefficient summarizes the average treatment effect across the analyzed population. Once the interaction term is added, the coefficient on $T_i$ becomes the treatment effect for the reference segment.

The interaction term $\delta$ answers:

> Is the treatment effect different for this segment?

For example, suppose:

- Treatment effect for new users: -1%
- Treatment effect for returning users: +3%

The segment difference is:

$$
3\% - (-1\%) = 4\%
$$

The interaction test evaluates whether that 4 percentage point difference is larger than what would be expected from random noise.

This is usually better than testing each segment separately and comparing significance labels.

## Guardrail Heterogeneity

Heterogeneity is not only about upside.

Sometimes the average primary metric improves, but a guardrail worsens for a specific group.

For example:

- A ranking change increases overall watch time.
- But reported content increases among teenage users.
- Or creator diversity decreases for small creators.
- Or refund rate rises in a specific product category.

This matters because the launch decision may depend on harm concentration.

A small average guardrail movement can hide a large problem in a vulnerable or strategically important segment.

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

## Machine Learning for HTE

Simple segment analysis works well when the team has a small number of plausible segments.

But some products have many possible moderators:

- User history
- Device
- Country
- Product category
- Price sensitivity
- Acquisition channel
- Prior exposure
- Marketplace state

The treatment effect may also be nonlinear. It may depend on combinations of features rather than one segment at a time.

Machine learning methods try to estimate the conditional average treatment effect, or CATE:

$$
\tau(x)
=
E[Y(1) - Y(0) \mid X=x]
$$

Here, $X$ represents user, item, market, or context features. Instead of asking only, "What is the average treatment effect?", the model asks:

> What is the treatment effect for units with these characteristics?

There are several common families of methods for estimating or predicting $\tau(x)$.

### Causal Trees

A causal tree is a tree-based method for finding interpretable segments with different treatment effects.

It is helpful to compare it with an ordinary decision tree. A standard prediction tree searches for splits that improve outcome prediction. For example, it may split users by country, device type, or purchase history because those features help predict conversion.

A causal tree searches for a different kind of split:

> Does this split create groups where the treatment effect is different?

The important subtlety is that the tree does not observe each user's individual treatment effect. For one user, the data contains only one of two potential outcomes:

- If the user is treated, we observe $Y_i(1)$.
- If the user is in control, we observe $Y_i(0)$.

The individual treatment effect would be:

$$
Y_i(1) - Y_i(0)
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

For example, suppose the overall experiment result is:

| Group | Control CVR | Treatment CVR | Estimated Lift |
|---|---:|---:|---:|
| All users | 10.0% | 11.0% | +1.0% |

The tree may ask whether prior purchase history explains variation in the treatment effect:

| Group | Control CVR | Treatment CVR | Estimated Lift |
|---|---:|---:|---:|
| Prior purchase users | 15.0% | 18.0% | +3.0% |
| No prior purchase users | 8.0% | 8.2% | +0.2% |

This split may be useful because the estimated average treatment effect differs across the two groups. The tree has not learned that a specific user's treatment effect is exactly +3.0% or +0.2%. It has learned that, on average, the treatment appears more effective among users with prior purchase history.

In an experiment, this leaf-level comparison is straightforward because treatment is randomized. Within each candidate leaf, treated and control users should be comparable on average, so the tree can estimate:

$$
\hat{\tau}_{leaf}
=
\bar Y_{T=1,leaf}
-
\bar Y_{T=0,leaf}
$$

This is the core version of a causal tree: search for product-readable segments where the randomized treated-control difference is meaningfully different.

Causal trees are useful when interpretability matters. They can turn a complex HTE problem into a small number of readable segment rules.

The main risk is instability. If the tree searches many possible splits, it can find patterns that are partly noise. Small changes in the data can sometimes produce a different tree.

For this reason, causal trees are often used with honest estimation. One part of the data chooses the tree structure, and another part estimates the treatment effect inside each leaf. This reduces the risk that the same noise is used both to find the segment and to estimate its effect.

#### When the Data Is Observational

The explanation above assumes experimental data. With observational data, the same tree idea can be used, but the leaf-level effect estimate needs more care.

In observational data, treatment was not randomly assigned. Treated and control users may differ before treatment.

For example, suppose a platform gives retention coupons mostly to users who look likely to churn. If coupon users have lower retention than non-coupon users, a naive comparison may make the coupon look harmful even if it helped. The treated users were already different.

So with observational data, a causal tree should not split on raw treated-control differences. It should split on adjusted estimated treatment effects.

The common identifying assumption is conditional ignorability:

$$
(Y(1), Y(0)) \perp T \mid X
$$

This means that, after controlling for observed features $X$, treatment assignment is as good as random. This is a strong assumption because it rules out important unobserved confounders.

Another required condition is overlap:

$$
0 < P(T=1 \mid X=x) < 1
$$

This means that, for the types of users being compared, the data must contain both treated and control examples. If every high-risk user received a coupon and no similar high-risk user was untreated, the tree cannot reliably estimate what would have happened without the coupon.

Two common ways to adjust the leaf-level treatment effect are inverse propensity weighting and doubly robust estimation.

**Inverse propensity weighting**

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

The weighting tries to make treated and control groups more comparable in observed covariates. If a treated user was unlikely to be treated, that user receives a larger weight because they represent an underrepresented type of treated user. The same logic applies to control users.

For an average treatment effect, a simple IPW estimator is:

$$
\hat{\tau}_{IPW}
=
\frac{1}{n}
\sum_i
\left[
\frac{T_iY_i}{e(X_i)}
-
\frac{(1-T_i)Y_i}{1-e(X_i)}
\right]
$$

Inside a causal tree, the same idea can be applied within candidate leaves. Instead of comparing raw treated and control means inside a leaf, the tree compares weighted treated and control outcomes.

The main weakness is instability. If $e(X_i)$ is close to 0 or 1, the inverse weight can become very large. A few observations can then dominate the estimate. In practice, analysts often check covariate balance after weighting, trim extreme propensity scores, cap very large weights, use stabilized weights, and avoid estimating effects in leaves with poor overlap.

**Doubly robust estimation**

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

Together, these models describe both expected outcomes and treatment-selection patterns.

A common doubly robust estimator is:

$$
\hat{\tau}_{DR}
=
\frac{1}{n}
\sum_i
\left[
\hat{\mu}_1(X_i)
-
\hat{\mu}_0(X_i)
+
\frac{T_i(Y_i-\hat{\mu}_1(X_i))}{\hat e(X_i)}
-
\frac{(1-T_i)(Y_i-\hat{\mu}_0(X_i))}{1-\hat e(X_i)}
\right]
$$

The first part, $\hat{\mu}_1(X_i) - \hat{\mu}_0(X_i)$, is the model-predicted treatment effect for a unit with features $X_i$. The remaining terms are residual corrections. They compare the observed outcome with the predicted outcome and weight that correction by the propensity score.

The intuition is:

- The outcome model says, "Based on this unit's features, here is what I predict under treatment and control."
- The propensity model says, "Because treatment assignment was not random, correct the prediction errors using how likely this unit was to receive treatment."

This is called doubly robust because the estimator can still be consistent if either the propensity model is correct or the outcome model is correct. It does not require both to be perfect. But if both are badly wrong, the estimate can still be biased.

Inside a causal tree, doubly robust scores can be used as adjusted treatment-effect signals when choosing splits and estimating leaf effects.

The comparison is:

| Method | Uses Propensity Model? | Uses Outcome Model? | Main Strength | Main Weakness |
|---|---:|---:|---|---|
| IPW | Yes | No | Reweights treated and control groups to improve observed comparability | Can be unstable with extreme propensity scores |
| Doubly robust | Yes | Yes | Can work if either the propensity model or outcome model is correct | More complex and still fails with hidden confounding |

The distinction is simple:

- With experimental data, a causal tree splits based on randomized treated-control differences inside candidate groups.
- With observational data, a causal tree should split based on adjusted estimated treatment effects.

Observational causal trees can be useful, but they are only causal if the adjustment variables are sufficient and there is enough overlap. Otherwise, the tree may discover where selection bias differs, not where the treatment effect differs.

### Causal Forests

A causal forest extends the causal tree idea by building many trees and averaging them.

This is similar to how a random forest improves on a single decision tree. Instead of relying on one tree, the forest combines many trees built from different samples and feature splits.

The output is usually an estimated treatment effect for each unit:

$$
\hat{\tau}(X_i)
$$

This can be used to:

- Rank users by predicted incremental impact
- Group users into high-, medium-, and low-effect segments
- Study which features are associated with stronger treatment effects
- Design a targeted rollout strategy

Causal forests are more flexible than simple segment tables, but they are less transparent. They also require careful validation because flexible models can find noisy treatment-effect patterns.

### Meta-Learners

Meta-learners estimate treatment effects by combining standard prediction models.

They are called "meta" learners because the framework is separate from the underlying model. The base model could be a linear model, random forest, gradient boosted tree, neural network, or another supervised learning method.

Three common examples are T-learners, S-learners, and X-learners.

**T-learner**

The T-learner fits two separate outcome models:

$$
\hat{\mu}_1(x) = E[Y \mid T=1, X=x]
$$

$$
\hat{\mu}_0(x) = E[Y \mid T=0, X=x]
$$

Then it estimates the treatment effect as:

$$
\hat{\tau}(x)
=
\hat{\mu}_1(x) - \hat{\mu}_0(x)
$$

The T-learner is intuitive: predict the treated outcome, predict the control outcome, then take the difference.

It works best when both treatment and control groups have enough data and the outcome patterns under treatment and control may be quite different.

Its main strength is flexibility. Because the treated and control outcome models are fit separately, each model can learn a different response surface. For example, if a promotion changes not only the level of purchase probability but also which user features matter, a T-learner can capture that difference.

Its main weakness is instability when one group is small. If the treatment group has far fewer users than the control group, the treated outcome model may be noisy. It can also struggle when the two models learn incompatible patterns, because the final treatment effect is the difference between two separately estimated functions.

**S-learner**

The S-learner fits one model using both treatment and control data:

$$
\hat{\mu}(x,t) = E[Y \mid X=x, T=t]
$$

Then it estimates:

$$
\hat{\tau}(x)
=
\hat{\mu}(x,1) - \hat{\mu}(x,0)
$$

The S-learner is simple because it uses one model. But if treatment effects are small relative to baseline outcome differences, the model may focus on predicting outcomes and understate treatment heterogeneity.

It works best when treatment and control outcome functions are mostly similar, and the treatment effect is relatively smooth or simple.

Its main strength is data sharing. Because one model is trained on both treated and control units, it can be more stable when data is limited.

Its main weakness is that the model may ignore treatment. If the outcome is driven much more by user features than by treatment, the model may use most of its capacity to predict baseline outcomes and treat $T$ as a minor feature. In that case, it can under-detect heterogeneous treatment effects.

**X-learner**

The X-learner is designed for settings where treatment and control groups are imbalanced or treatment effects vary a lot.

At a high level, it:

1. Fits outcome models for treatment and control.
2. Uses those models to impute individual-level treatment-effect signals.
3. Fits models to predict those imputed effects.
4. Combines the results using weights based on treatment probability or model reliability.

The X-learner is more complex, but it can work well when one group is much larger than the other.

It works best when treatment assignment is imbalanced or when the treated and control groups have very different sizes.

Its main strength is that it uses information from the larger group to improve treatment-effect estimation for the smaller group. For example, if only 10% of users received a coupon and 90% did not, the X-learner can use the large control group to build a strong control outcome model, then use that model to help infer treatment-effect signals for treated users.

Its main weakness is complexity. There are more modeling steps, more chances for error, and more implementation choices. It also needs careful validation because imputed treatment-effect signals can amplify mistakes from the first-stage outcome models.

A practical comparison is:

| Learner | Basic Idea | Works Best When | Main Strength | Main Weakness |
|---|---|---|---|---|
| T-learner | Fit separate treated and control outcome models | Both groups have enough data; treatment and control patterns may differ | Flexible response surfaces | Noisy when one group is small; difference of two models can be unstable |
| S-learner | Fit one outcome model with treatment as a feature | Treatment effect is simple; data is limited | Stable and easy to implement | May understate treatment heterogeneity |
| X-learner | Impute treatment-effect signals, then model them | Treatment and control group sizes are imbalanced | Uses the larger group efficiently | More complex and sensitive to first-stage model errors |

There is no universally best meta-learner. The choice depends on sample size, treatment-control balance, expected complexity of treatment effects, and how much validation data is available.

## From HTE Estimation to Uplift Modeling

HTE analysis and uplift modeling are closely related, but they emphasize different goals.

HTE analysis is often explanatory. It asks:

> Where is the treatment effect different?

Uplift modeling is decision-oriented. It asks:

> Can we predict which users will be changed most by treatment?

The target is still the conditional average treatment effect, or CATE:

$$
\tau(x) = E[Y(1) - Y(0) \mid X=x]
$$

If a model can predict this quantity for a new user,

$$
\hat{\tau}(X_i)
$$

then the platform can rank users by predicted incremental impact and decide whom to treat.

This is why uplift modeling is not one specific algorithm. It is a use case for predicted treatment effects. Causal trees, causal forests, meta-learners, and other models can all support uplift modeling if they produce reliable out-of-sample predictions of $\tau(x)$.

The key distinction is between predicting outcomes and predicting incremental effects.

A normal conversion model asks:

> Who is most likely to purchase?

An uplift model asks:

> Whose purchase probability increases the most because of the treatment?

These are not the same question.

For example, loyal users may have high purchase probability whether or not they receive a promotion. A standard conversion model may rank them highly. But the promotion may not be incremental for them because they would have purchased anyway.

Another group may have medium baseline purchase probability but respond strongly to the promotion. That group can have higher uplift even though its raw conversion probability is lower.

This matters most when treatment has a cost or downside:

- Promotions cost money.
- Notifications can annoy users.
- Ads retargeting can waste budget.
- Sales outreach uses team capacity.
- Personalized recommendations can reduce diversity if targeted poorly.

In these cases, the platform may not want to treat everyone. The goal is to treat users who are likely to be changed by the treatment, not users who would take the desired action anyway.

A useful way to think about targeting is to divide users into four conceptual groups:

| Segment Type | Behavior |
|---|---|
| Persuadables | Convert only if treated |
| Sure things | Convert whether treated or not |
| Lost causes | Do not convert either way |
| Do-not-disturbs | Convert less if treated |

The most valuable target group is usually the persuadables. The model should avoid sure things because treatment is not incremental for them. It should also avoid do-not-disturbs because treatment may cause harm.

This framing also explains why ordinary model accuracy is not enough. A model that predicts purchases well may mostly identify sure things. That can be useful for forecasting, but it is not necessarily useful for targeting a promotion.

Once uplift modeling is used for targeting, the next question is whether the model can rank future users well. It is not enough for the model to explain patterns in the data used to train it. The model needs to identify high-uplift users it has not seen before.

That is why validation should be out of sample.

In randomized experiments, a common validation method is uplift by decile:

1. Train an uplift model using experiment data.
2. Apply it to a holdout or validation sample.
3. Rank users by predicted uplift.
4. Split users into deciles from highest to lowest predicted uplift.
5. Estimate the actual treatment effect within each decile.
6. Check whether the highest-uplift deciles show larger incremental effects.

If the model is useful, the top deciles should have meaningfully higher observed treatment effects than the bottom deciles.

This validation evaluates incremental lift directly. It asks whether the model can rank users by treatment effect, not merely whether it can predict the outcome.

Machine-learning-based HTE and uplift methods are most useful when:

- There are many possible moderators.
- The treatment effect is likely nonlinear.
- The goal is targeting, personalization, or segment discovery.
- The team has enough data to validate the model.

They also increase the risk of overfitting. A model can find impressive-looking treatment differences that do not hold up later.

Practical safeguards include:

- Split the data into discovery and validation samples.
- Use cross-fitting or honest estimation.
- Evaluate calibration of predicted treatment effects.
- Compare model-based segments with simple business segments.
- Validate targeting rules in a new experiment.

For most product experiment reports, simple pre-specified segment analysis is often more useful than a complex uplift model. Machine learning becomes more valuable when the business truly plans to personalize treatment or target a costly intervention.

## Example: Recommendation Ranking

Suppose an e-commerce platform tests a new recommendation ranking model.

Average results:

- GMV per user: +2.0%
- Repeat-purchase GMV: +4.5%
- Refund rate: no significant change

The average result is positive. But because the model is designed to identify durable user-creator or user-seller relationships, the team should expect heterogeneity.

Natural segments include:

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

This is a good use of HTE analysis because the segments are connected to the treatment mechanism.

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

Again, the value of HTE analysis is not only finding where the lift is largest. It is understanding how the treatment changes the composition and quality of users.

## Practical Workflow

A practical workflow for HTE analysis is:

1. Start with the overall treatment effect.
2. Identify the treatment mechanism.
3. Define a small number of pre-specified segments based on that mechanism.
4. For each segment, report effect size, sample size, baseline metric, and confidence interval.
5. Use interaction tests to compare treatment effects across segments.
6. Separate confirmatory segment analysis from exploratory analysis.
7. Adjust for multiple testing when many confirmatory segments are tested.
8. Treat surprising exploratory findings as hypotheses.
9. Check whether segment patterns are consistent across related metrics.
10. Consider guardrail heterogeneity, not only primary metric heterogeneity.
11. Validate important targeting decisions in follow-up experiments.

## Common Mistakes

**Comparing significance labels**

"Significant in one segment but not significant in another" does not prove the treatment effects are different. Use an interaction test.

**Searching too many segments and telling a story afterward**

Exploratory segment analysis can generate useful ideas, but it should not be treated as confirmatory evidence.

**Ignoring sample size**

Small segments often produce noisy and extreme estimates. Always show uncertainty.

**Looking only for upside**

HTE analysis should also search for concentrated harm in important guardrail segments.

**Launching targeted policies too quickly**

A segment discovered after the fact may not replicate. Important targeted rollouts should be validated.

**Treating uplift as ordinary prediction**

A model that predicts conversion well may not predict incremental lift well. Uplift models should be evaluated on treatment-control differences, not only outcome prediction.

**Using complex models without validation**

Machine learning methods for HTE can overfit. They need honest validation, not just attractive segment charts.

## Key Takeaways

The average treatment effect can hide meaningful differences across users, items, markets, and contexts.

HTE analysis asks where and for whom the treatment works better or worse.

Good HTE segments should be connected to the treatment mechanism.

Pre-specified segments are more credible than segments discovered after looking at the data.

Segment analysis creates a multiple testing problem and requires uncertainty reporting.

Do not compare significance labels across segments. Test the interaction directly.

Guardrail heterogeneity matters because harm may be concentrated in specific groups.

Causal trees, causal forests, and meta-learners can estimate richer forms of heterogeneity, but they need careful validation.

With observational data, HTE methods need adjustment for confounding and require assumptions such as conditional ignorability and overlap.

Uplift modeling uses predicted treatment effects for targeting or decision-making.

HTE analysis is most useful when it leads to better product understanding, better targeting, or safer rollout decisions.
