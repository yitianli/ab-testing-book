# Quasi-Experiments

Randomized experiments are powerful because treatment assignment is controlled by design. If the experiment is well implemented, treatment and control groups should be comparable before the treatment happens. That makes the causal question relatively clean.

But in real product and business settings, randomization is not always possible.

Sometimes the product has already launched. Sometimes the change happens at the market, policy, creator, or platform level, so there is no clean user-level randomization. Sometimes a team cannot withhold a feature for ethical, legal, operational, or political reasons. Sometimes the decision was made outside the experimentation system, and the analyst is asked to evaluate it afterward.

Quasi-experiments are methods for causal analysis when treatment was not randomly assigned.

They do not magically turn observational data into an A/B test. Instead, they try to find a credible comparison by using structure in the data:

- A change over time
- A policy cutoff
- A staggered rollout
- A similar untreated group
- A source of variation that affects treatment but does not directly affect the outcome

The central question is always:

> In the absence of treatment, what would have happened to the treated group?

That missing counterfactual is the heart of quasi-experimental thinking.

## Why Quasi-Experiments Are Hard

Suppose an e-commerce platform launches a new ranking model in one country but not another. After launch, GMV increases in the treated country.

Can we conclude that the ranking model caused the increase?

Not immediately.

The treated country may have been growing faster already. It may have had a holiday promotion. It may have had different inventory, different users, different creator behavior, or different macroeconomic conditions. The untreated country may not be a good counterfactual.

The problem is selection.

Treatment assignment was not random. The treated units may differ from untreated units in ways that also affect the outcome.

In an A/B test, randomization helps make:

$$
E[Y(0) \mid T=1] \approx E[Y(0) \mid T=0]
$$

In words, the treated and control groups would have had similar outcomes under control.

In a quasi-experiment, this is no longer guaranteed. The method must argue why the comparison group is still credible.

This is why quasi-experiments depend heavily on assumptions. The math can be elegant, but the design logic matters more.

## The Design Mindset

A good quasi-experimental analysis usually starts before modeling.

The analyst should ask:

- What caused some units to receive treatment and others not?
- Was the timing of treatment related to expected performance?
- What would the treated units have done without treatment?
- Is there a comparison group with similar pre-treatment behavior?
- Are there other events happening at the same time?
- Can we test the core identifying assumption indirectly?

The word "identification" means the assumptions that allow us to interpret an estimate causally.

For example, a difference-in-differences design is not credible just because it uses a difference-in-differences formula. It is credible only if the comparison group gives a reasonable estimate of the treated group's counterfactual trend.

Quasi-experiments are therefore design-first methods. The model is only useful if the design story is believable.

## Difference-in-Differences

Difference-in-differences, often called DiD, is one of the most common quasi-experimental methods.

It is useful when:

- Some units receive treatment
- Other units do not
- We observe both groups before and after treatment

The basic idea is to compare changes, not levels.

Suppose one city receives a new delivery dispatch algorithm, and another city does not.

| Group | Before | After |
|---|---:|---:|
| Treated city | 10.0 | 13.0 |
| Control city | 8.0 | 9.0 |

The treated city increased by:

$$
13.0 - 10.0 = 3.0
$$

The control city increased by:

$$
9.0 - 8.0 = 1.0
$$

The difference-in-differences estimate is:

$$
(13.0 - 10.0) - (9.0 - 8.0) = 2.0
$$

The interpretation is:

> The treated city improved by 2.0 units more than the control city did.

More generally:

$$
\widehat{\tau}_{\text{DiD}}
=
(\bar{Y}_{T,\text{post}} - \bar{Y}_{T,\text{pre}})
-
(\bar{Y}_{C,\text{post}} - \bar{Y}_{C,\text{pre}})
$$

This removes two kinds of differences:

- Permanent level differences between treated and control groups
- Common time shocks that affect both groups

If the treated city always has higher GMV than the control city, DiD can still work as long as the gap would have stayed stable without treatment. If both cities are affected by the same holiday or seasonality, DiD can subtract that common shock.

## The Parallel Trends Assumption

The key assumption behind DiD is parallel trends.

Parallel trends means:

> Without treatment, the treated group and control group would have changed similarly over time.

It does not require treated and control groups to have the same level before treatment. It requires their trends to be comparable.

For example:

| Week | Treated City GMV | Control City GMV |
|---|---:|---:|
| -4 | 100 | 80 |
| -3 | 105 | 84 |
| -2 | 110 | 88 |
| -1 | 115 | 92 |

The treated city is always higher, but both cities grow by about 5% each week. This makes the control city more plausible as a counterfactual.

Now compare:

| Week | Treated City GMV | Control City GMV |
|---|---:|---:|
| -4 | 100 | 80 |
| -3 | 110 | 82 |
| -2 | 125 | 84 |
| -1 | 145 | 86 |

Here, the treated city was already accelerating before treatment. A simple DiD estimate after launch may falsely attribute that pre-existing growth to the treatment.

Parallel trends cannot be proven, because it is about what would have happened after treatment without treatment. But it can be assessed indirectly.

Common checks include:

- Plot pre-treatment trends
- Estimate placebo effects before the treatment date
- Use multiple pre-periods instead of only one
- Compare several candidate control groups
- Check whether other outcomes move before treatment
- Look for other events that happen at the same time as treatment

If the pre-trends are very different, DiD becomes hard to defend.

## DiD Regression

The two-by-two DiD estimate can also be written as a regression.

Let:

- $T_i = 1$ if unit $i$ belongs to the treated group
- $Post_t = 1$ after treatment starts
- $Y_{it}$ be the outcome for unit $i$ at time $t$

Then:

$$
Y_{it}
=
\alpha
+ \beta T_i
+ \gamma Post_t
+ \tau (T_i \times Post_t)
+ \epsilon_{it}
$$

The coefficient $\tau$ is the DiD estimate.

The terms have intuitive meanings:

- $\beta$ captures baseline differences between treated and control units
- $\gamma$ captures common post-period shocks
- $\tau$ captures the extra post-period change for treated units

In many real applications, DiD is considered as a special case of a two-way fixed effect model, i.e., with unit fixed effects and time fixed effects:

$$
Y_{it}
=
\alpha_i
+ \lambda_t
+ \tau \text{Treatment}_{it}
+ \epsilon_{it}
$$

Here:

- $\alpha_i$ controls for stable differences across units
- $\lambda_t$ controls for shocks common to all units at time $t$
- $\text{Treatment}_{it}$ indicates whether unit $i$ is treated at time $t$

This form is common when there are many cities, users, stores, products, or markets observed over time.

## Staggered Rollouts

Many product launches are staggered. One city receives a feature in January, another city in February, and another in March.

This looks attractive for DiD because treatment timing varies across units.

But staggered rollouts require care.

The timing of rollout may not be random. Teams often launch first in markets where:

- Engineering support is easier
- The business opportunity is larger
- The market is more stable
- The team expects stronger results
- Local partners are ready

If rollout timing is related to expected outcomes, the comparison is biased.

Another issue is that, when estimated using a two-way fixed effect model, already-treated units may become inappropriate controls for later-treated units, especially when treatment effects change over time.

The practical advice is:

- Understand how rollout order was chosen
- Plot event-time effects around treatment
- Check pre-trends for early and late treated units
- Avoid blindly trusting a single two-way fixed effects estimate
- Consider modern staggered DiD estimators when rollout timing varies

For a product analyst, the most important point is not the estimator name. The important point is whether untreated or not-yet-treated units are a believable counterfactual.

## Synthetic Control

Synthetic control is useful when there is one treated unit or a small number of treated units.

For example:

- A new policy launches in one country
- A pricing change affects one region
- A marketplace intervention affects one city
- A major creator program launches in one vertical

Instead of choosing one control unit, synthetic control builds a weighted combination of untreated units.

The goal is to create a synthetic version of the treated unit that matches its pre-treatment behavior.

Suppose the treated market is Canada. A synthetic control might be:

$$
0.35 \times \text{Australia}
+ 0.25 \times \text{United Kingdom}
+ 0.20 \times \text{Germany}
+ 0.20 \times \text{France}
$$

The weights are chosen so that the synthetic control resembles Canada before treatment.

After treatment, the estimated effect is the gap between the treated unit and its synthetic control:

$$
\widehat{\tau}_t
=
Y_{\text{treated},t}
-
Y_{\text{synthetic},t}
$$

Synthetic control is especially useful when a single untreated unit is not comparable enough.

Inference with synthetic control is different from inference in a large user-level A/B test. Because the design often has one treated unit and a limited donor pool, ordinary standard errors are usually not the main tool. A common approach is placebo inference. The analyst repeatedly pretends that each untreated donor unit was treated, builds a synthetic control for that placebo unit, and estimates its post-treatment gap. The actual treated-unit gap is then compared with the distribution of placebo gaps. If the real gap is much larger than the gaps produced by fake treatment dates or fake treated units, the evidence is stronger.

One simple way to summarize uncertainty is to compute the variance of placebo effects at each post-treatment time:

$$
\widehat{\text{Var}}(\widehat{\tau}_t)
=
\frac{1}{J-1}
\sum_{j=1}^{J}
(\widehat{\tau}_{j,t}^{\text{placebo}} - \bar{\tau}_t^{\text{placebo}})^2
$$

where $J$ is the number of placebo donor units. This can be used to form rough uncertainty bands around the estimated effect. In practice, analysts often also compare the actual post-treatment gap to the ratio of post-treatment error to pre-treatment error for each placebo unit. This matters because a placebo unit with poor pre-treatment fit is not a very useful comparison. The goal is not just to attach a mechanical standard error, but to ask whether the treated unit looks unusual relative to units that should not have been affected.

This is why the first diagnostic is always the pre-treatment fit. Before interpreting the post-treatment gap, the analyst should ask whether the synthetic control actually tracked the treated unit before the intervention. If the synthetic Canada was already far away from the real Canada before treatment, then a post-treatment gap is not very convincing.

After checking pre-treatment fit, the next question is whether the treated unit looks unusual relative to placebo units. If many untreated markets also show large post-treatment gaps when they are treated as fake treated units, then the Canada result may not be special. But if Canada has a much larger and more persistent gap than most placebo units, the evidence becomes stronger.

The final step is sensitivity analysis. A good synthetic control result should not depend entirely on one donor market, one exact pre-treatment window, or one arbitrary set of predictors. The analyst should explain why the donor units are eligible controls, test whether the result changes when important donors are removed, and check whether other major events happened around the treatment date.

So synthetic control is not only a weighting method. It is a discipline for constructing and defending a credible comparison unit.

## Regression Discontinuity

Regression discontinuity, or RD, is useful when treatment is assigned by a cutoff.

Examples:

- Users with score above 80 receive a promotion
- Sellers above a quality threshold receive a badge
- Loans above a risk threshold are rejected
- Creators above a follower threshold enter a program
- Orders above a price threshold get free shipping

The key idea is that units just below and just above the cutoff may be very similar.

If users with score 79.9 do not receive treatment and users with score 80.1 do receive treatment, the comparison near the cutoff may be close to random.

Let $X_i$ be the running variable, such as score, and $c$ be the cutoff. Treatment is assigned when:

$$
X_i \ge c
$$

RD estimates the jump in the outcome at the cutoff:

$$
\tau_{\text{RD}}
=
\lim_{x \downarrow c} E[Y \mid X=x]
-
\lim_{x \uparrow c} E[Y \mid X=x]
$$

In words:

> How much does the outcome jump right at the threshold?

The identifying assumption is continuity:

> Without treatment, the outcome would have changed smoothly through the cutoff.

If there is a sudden jump exactly at the cutoff, and no other rule changes at that same cutoff, the jump can be interpreted as the treatment effect near the cutoff.

### RD Diagnostics

RD depends heavily on the cutoff being hard to manipulate.

If users, sellers, or teams can precisely manipulate the running variable, the design becomes less credible.

For example:

- Sellers may game a quality score to cross the badge threshold
- Users may split orders to fall below a price cutoff
- Account managers may manually push strategic creators above an eligibility score

Useful RD checks include:

- Plot outcomes around the cutoff
- Check whether the number of units jumps at the cutoff
- Check covariate balance just above and below the cutoff
- Try different bandwidths around the cutoff
- Check whether other policies change at the same cutoff

RD estimates are local. They estimate the treatment effect for units near the cutoff, not necessarily for all users.

This is a strength and a limitation. The comparison near the cutoff can be credible, but the result may not generalize to units far away from the cutoff.

## Matching

Matching tries to construct a comparison group that looks similar to the treated group based on observed covariates.

Suppose a company gives a retention offer to some users. Treated users are compared with untreated users who look similar before the offer.

Covariates might include:

- Country
- Device type
- Tenure
- Past purchase frequency
- Past engagement
- Acquisition channel
- Baseline subscription status

The idea is simple:

> Compare treated users with untreated users who had similar observed characteristics before treatment.

Matching can be done in many ways:

- Exact matching
- Nearest-neighbor matching
- Propensity score matching
- Coarsened exact matching
- Weighting based on propensity scores

A propensity score is:

$$
e(x) = P(T=1 \mid X=x)
$$

It is the probability of receiving treatment given observed features.

Users with similar propensity scores have similar observed likelihood of treatment. Matching or weighting by propensity score can make treated and control groups more balanced on observed covariates.

The key phrase is "observed covariates."

Matching cannot fix unobserved confounding. If high-motivation users are more likely to receive treatment and motivation is not measured, matching on observed variables may still be biased.

This is why matching is often less convincing than designs based on timing, cutoffs, or external variation. It can be useful, but it relies on the assumption that all important confounders are observed and measured well.

### Inference with Matching

Inference after matching needs care because the comparison group is constructed from the data. The analyst should not match treated and control units, run a standard two-sample t-test, and ignore how the matched sample was created.

In the simplest one-to-one matched-pair design, each treated unit is matched to one control unit. The analysis can use pair-level differences:

$$
d_i
=
Y_i^{T}
-
Y_i^{C}
$$

Then the treatment effect is:

$$
\widehat{\tau}
=
\frac{1}{N}
\sum_i d_i
$$

and the standard error can be based on the variation in paired differences:

$$
SE(\widehat{\tau})
=
\frac{sd(d_i)}{\sqrt{N}}
$$

This paired-difference approach works best when the matched pairs are reasonably independent and the matching structure is simple.

For more general matching designs, inference is harder. One-to-many matching, matching with replacement, and propensity-score matching can create dependence across observations because the same control unit may be used multiple times or because the matched sample is selected by an estimated score.

Common approaches include:

- Analytic standard errors designed for nearest-neighbor matching, such as Abadie-Imbens standard errors
- Regression after matching, using robust or cluster-robust standard errors on the matched sample
- Randomization inference when the matched design is intended to approximate paired random assignment
- Sensitivity analysis for hidden confounding

The ordinary bootstrap should be used carefully. For nearest-neighbor matching, the matching rule can change discontinuously when the sample changes, so the usual bootstrap may not give valid uncertainty estimates.

The practical lesson is:

> Matching improves comparability, but it does not create randomization. Inference should respect the matched design, and the causal interpretation still depends on balance in observed covariates and the absence of important unobserved confounders.

## Instrumental Variables

Instrumental variables, or IV, are used when treatment is confounded but there is some external source of variation that shifts treatment exposure.

An instrument $Z$ should satisfy three ideas:

**Relevance**

The instrument affects treatment.

$$
Z \rightarrow T
$$

**Exclusion restriction**

The instrument affects the outcome only through treatment.

$$
Z \rightarrow T \rightarrow Y
$$

There should not be a direct path:

$$
Z \rightarrow Y
$$

**As-if random variation**

The instrument is not related to unobserved factors that also affect the outcome.

Examples that sometimes act like instruments:

- Random encouragement to use a feature
- Distance to a service location
- Assignment to a salesperson
- Eligibility rules
- System outages that affect exposure
- Queue position or capacity constraints

But instruments are often controversial because the exclusion restriction is hard to prove.

For example, suppose a notification outage reduces feature usage for some users. The outage might be used as an instrument for feature usage. But if the outage also annoys users directly or changes their trust in the product, then it affects the outcome through more than feature usage. The instrument is invalid.

IV estimates are also local. They estimate effects for users whose treatment status was changed by the instrument, often called compliers. This may not equal the average treatment effect for all users.

## Interrupted Time Series

Interrupted time series uses a long time series before and after a treatment.

It is useful when:

- One unit is treated
- There is no obvious control group
- There are many observations over time
- The treatment happens at a clear date

For example, a platform changes its search ranking algorithm globally on March 1. There is no untreated market, but the team has daily search conversion data for many months before and after launch.

The analysis asks:

> Did the level or trend of the metric change sharply after the intervention?

A simple model might be:

$$
Y_t
=
\alpha
+ \beta t
+ \tau Post_t
+ \gamma (t \times Post_t)
+ \epsilon_t
$$

Here:

- $\beta$ is the pre-treatment trend
- $\tau$ is the immediate level change after treatment
- $\gamma$ is the change in trend after treatment

Interrupted time series is weaker than DiD when other things happen at the same time. If a promotion, seasonality change, competitor event, or logging change occurs near the intervention date, attribution becomes difficult.

It is strongest when the outcome is stable, the intervention date is sharp, the time series is long, and there are no other plausible explanations for the break.

## Choosing Among Quasi-Experimental Designs

Different methods fit different kinds of variation.

| Situation | Useful Design |
|---|---|
| Treated and untreated groups observed before and after | Difference-in-differences |
| One treated market and many potential controls | Synthetic control |
| Treatment assigned by a cutoff | Regression discontinuity |
| Treated users can be matched to similar untreated users | Matching or weighting |
| External variation shifts treatment exposure | Instrumental variables |
| One treated unit with a long pre/post time series | Interrupted time series |

The choice should follow the assignment mechanism.

Do not start by asking, "Which model should I use?"

Start by asking:

> Why did some units get treated and others not?

The answer points toward the design.

## Product Example: Marketplace Fee Change

Suppose a marketplace changes seller fees in one country. The change is expected to improve platform revenue, but it may reduce seller participation.

Randomization is not feasible because the fee policy applies at the country level.

A possible quasi-experimental approach is synthetic control.

The treated unit is the country with the fee change. The donor pool contains similar countries that did not change fees. The analyst builds a weighted combination of donor countries that matches the treated country before the policy change.

Primary metrics might include:

- Platform revenue
- Seller active rate
- Listing supply
- Buyer conversion rate
- GMV

Guardrails might include:

- Seller churn
- Buyer retention
- Complaint rate
- Refund rate

The core evidence would be:

- The synthetic control tracks the treated country well before the change
- After the change, platform revenue rises relative to synthetic control
- Seller supply and buyer experience guardrails do not degrade
- Placebo countries do not show similarly large fake effects

Even then, the conclusion should be cautious. The estimate is credible only if no other country-specific shock occurred at the same time.

## Product Example: Seller Badge Threshold

Suppose sellers receive a "trusted seller" badge if their quality score is at least 90. The team wants to know whether the badge increases conversion.

This is a natural regression discontinuity setting.

Sellers with scores just below 90 and just above 90 may be similar, except that one group receives the badge.

The analysis would compare conversion near the cutoff.

Key checks:

- Is there a visible jump in conversion at score 90?
- Are seller characteristics smooth around score 90?
- Is there a suspicious pile-up just above 90?
- Are there other benefits triggered at the same score?
- Does the estimated effect change dramatically with bandwidth?

The conclusion should be local:

> The badge effect is estimated for sellers near the quality threshold, not necessarily for very low-score or very high-score sellers.

## Practical Workflow

When evaluating a non-randomized change, use a workflow like this:

1. Define the causal question.
2. Describe exactly how treatment was assigned.
3. Identify the candidate counterfactual.
4. Choose the design that matches the assignment mechanism.
5. Plot pre-treatment outcomes.
6. Check whether the core identifying assumption is plausible.
7. Estimate the effect.
8. Run placebo and sensitivity checks.
9. Inspect guardrails and secondary outcomes.
10. State the conclusion with assumptions and limitations.

The last step matters. Quasi-experimental conclusions should not sound as certain as clean randomized experiments unless the design is exceptionally strong.

## Common Mistakes

**Comparing treated and untreated levels**

If treated units are naturally larger, more engaged, or faster growing, level differences can be misleading.

**Using DiD without checking pre-trends**

The DiD formula is not enough. The counterfactual trend must be plausible.

**Ignoring why rollout happened**

If treatment timing was chosen based on expected performance, the design may be biased.

**Overtrusting matching**

Matching balances observed covariates. It does not balance unobserved motivation, intent, quality, or market pressure.

**Using an invalid instrument**

An instrument must affect the outcome only through treatment. This is a strong assumption and should be argued carefully.

**Forgetting that RD and IV are local**

RD estimates effects near the cutoff. IV estimates effects for units whose treatment was shifted by the instrument. These may not generalize.

**Ignoring simultaneous shocks**

Promotions, holidays, logging changes, competitor actions, and operational incidents can all create fake treatment effects.

## Key Takeaways

Quasi-experiments estimate causal effects when randomization is not available.

The core question is always: what would have happened to the treated group without treatment?

Difference-in-differences relies on parallel trends.

Synthetic control builds a weighted comparison unit that matches the treated unit before treatment.

Regression discontinuity compares units just above and below a treatment cutoff.

Matching compares treated units with similar untreated units, but only on observed covariates.

Instrumental variables use external variation that shifts treatment exposure, but the assumptions are strong.

Interrupted time series looks for a break after an intervention, but it is vulnerable to simultaneous shocks.

The credibility of a quasi-experiment comes less from the formula and more from the assignment mechanism, the counterfactual, and the diagnostics.
