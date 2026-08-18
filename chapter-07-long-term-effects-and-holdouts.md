# Long-Term Effects and Holdouts

Many experiments are designed around a short observation window:

> Did the treatment improve the metric during the experiment?

That question is useful, but it is not always the question the business needs to answer. A product decision may depend on retention, repeat purchase, refund-adjusted revenue, lifetime value, seller supply, creator health, or marketplace quality. These outcomes often take longer to observe than clicks, signups, orders, or first sessions.

This chapter is about the gap between short-term experiment evidence and long-term product value.

The central idea is simple:

> A treatment effect is not only a number. It can also be a time path.

The chapter moves in three steps:

1. Explain why short-term results can diverge from long-term value.
2. Show how to read or preserve evidence when effects change over time.
3. Decompose long-term value so the launch decision is not based on one opaque modeled number.

## Why Short-Term Results Can Mislead

Suppose a subscription app tests a new onboarding flow.

After seven days, the result looks strong:

- Signup completion: +8%
- Trial start: +5%

The team may want to launch immediately. But the business goal is not only more trials. The business wants more valuable subscribers.

After 60 days, the result may look different:

- Paid conversion after trial: -4%
- Refund requests: +3%
- Net revenue per signup: no change

The short-term metrics were not wrong. They measured what happened early in the funnel. The problem is that the experiment window did not capture the full business outcome.

Chapter 3 discussed one version of this problem: when the true decision metric is delayed or hard to observe, teams often use leading indicators or proxy metrics. Those metrics can be useful, but they can also mislead if their relationship with the delayed outcome breaks.

This chapter focuses on the time dimension of that problem. Even when the metric plan is reasonable, the treatment effect itself may change over time. Short-term results can disagree with long-term value for several reasons.

### Novelty Effects

A novelty effect happens when users react temporarily because an experience is new.

Examples:

- A new feed layout increases exploration for a few days.
- A new badge attracts attention at first, then becomes ignored.
- A redesigned homepage increases clicks because users are curious.
- A new notification style works temporarily until users habituate.

The short-term lift may be real, but not durable. Novelty effects are especially plausible when the change is visually obvious, users interact with the surface repeatedly, or the metric depends heavily on attention.

Novelty is not always bad. A launch can benefit from attention. But if the decision depends on durable value, the team needs to know whether the effect persists after the experience stops feeling new.

### Learning Effects

A learning effect is the opposite pattern: the treatment becomes more valuable as users learn how to use it.

Examples:

- A new analytics dashboard may be confusing at first, then valuable later.
- A new creator tool may require time before creators adjust their workflow.
- A recommendation feature may improve as users give more feedback.
- A seller tool may affect inventory quality only after sellers learn the new process.

In this case, a short experiment may underestimate the treatment effect.

For example:

| Time Since Exposure | Estimated Effect on Weekly Active Use |
|---|---:|
| Week 1 | -1.0% |
| Week 2 | +0.5% |
| Week 4 | +3.0% |
| Week 8 | +4.0% |

A one-week test might reject the feature. A longer test might show that users needed time to learn the new workflow.

### Accumulation and Ecosystem Effects

Some effects grow slowly because users, sellers, creators, models, or marketplaces adapt.

Examples:

- A recommendation model changes which sellers receive traffic, and seller supply changes over time.
- A creator tool changes creator behavior, which later changes content supply.
- A pricing or incentive change affects user expectations and market equilibrium.
- A ranking model improves as it collects better feedback data.

These effects may not appear during the first few days. They can accumulate through repeated behavior, feedback loops, or marketplace responses.

### Composition Effects and Delayed Harm

Sometimes the short-term metric improves because the treatment brings more users, orders, applications, or interactions into the funnel. But the added volume may be lower quality, or the harm may only appear later in refunds, complaints, churn, retention, or ecosystem quality.

One pattern is composition-driven harm. The treatment changes who enters the downstream denominator.

For example, a one-click application feature may increase applications submitted within one day. If applications are the only metric, the treatment looks successful. But recruiter responses, interviews, or hires may take weeks. If the extra applications are low effort, the leading indicator improves while the decision metric worsens.

Another pattern is experience-driven delayed harm. The same users may respond positively at first, but later outcomes worsen after repeated exposure.

Examples:

- More signups, but fewer paid subscribers.
- More purchases, but higher refund rate.
- More watch time, but worse long-term satisfaction.
- More messages, but more complaints.
- More applications, but fewer meaningful matches.
- More orders, but worse supply quality.

This pattern is especially important because the early experiment can look clearly positive while the long-term decision is more complicated.

## Reading Long-Term Effects Over Time

Once long-term mismatch is possible, the best solution is to observe the long-term outcome directly. If the decision metric needs 30 days to mature and the experiment can safely run for 30 days, the report should include the final result and the time path.

But real experiments are not always that clean. Sometimes the experiment runs long enough but the trend is hard to interpret. Sometimes the business cannot wait for the delayed metric before shipping. This section follows that practical order:

1. Run long enough and report the time path when feasible.
2. Use targeted diagnostics when the trend is ambiguous.
3. Use follow-up readouts, holdouts, or predicted long-term metrics when the experiment cannot run long enough.

### Run Long Enough to See the Time Path

When the experiment can run long enough, plot the treatment effect by time since exposure.

| Time Since Exposure | Estimated Effect |
|---|---:|
| Day 1 | +5.0% |
| Day 7 | +3.0% |
| Day 30 | +0.5% |
| Day 60 | 0.0% |

This time path tells a different story from the day-1 effect alone.

Common patterns include:

- **Stable effect**: the effect appears quickly and remains similar over time.
- **Decay**: the effect is large early but shrinks, suggesting novelty or temporary attention.
- **Ramp-up**: the effect is small early but grows, suggesting learning, habit formation, network accumulation, or model feedback.
- **Reversal**: the effect is positive early but negative later, suggesting delayed harm.

A strong report should show both cumulative and period-by-period effects.

| Window | Treatment Effect |
|---|---:|
| Days 1-7 | +5.0% |
| Days 8-30 | +1.0% |
| Days 31-60 | -0.5% |

The cumulative 60-day effect might still be positive, but the period-by-period view shows whether the treatment is still helping, fading, growing, or reversing.

When the delayed metric is retention or churn, the same idea becomes a retention or survival curve. The key question is not whether every experiment must wait 60 days before making a decision. The question is which retention horizon matters for the decision, and whether earlier signals have been validated as reasonable proxies.

### When the Trend Is Ambiguous

Sometimes the time path does not clearly show decay, ramp-up, or reversal. In that case, the team should use diagnostics that match the suspected mechanism.

If novelty is plausible, compare new users and existing users.

Existing users have a prior experience. When the product changes, they may react partly because the experience feels new or surprising. New users do not have the same before-versus-after comparison. They enter the product for the first time, so they are less exposed to novelty from the change itself.

For example:

| Segment | Day-1 Lift | Day-30 Lift |
|---|---:|---:|
| Existing users | +6.0% | +1.0% |
| New users | +1.5% | +1.2% |

This pattern suggests that much of the early lift among existing users may be temporary. New users show a smaller but more stable effect, which may be closer to the durable value of the new experience.

This comparison is not proof. New users and existing users differ in many ways besides novelty. Existing users may be more engaged, have richer histories, or use different product surfaces. The comparison should be treated as evidence, not as a complete identification strategy.

If calendar effects or user mix are plausible, use cohort analysis. Cohort analysis tracks users based on when they first entered the experiment or first saw the treatment.

| Cohort | Day 1 Retention Lift | Day 7 Retention Lift | Day 30 Retention Lift |
|---|---:|---:|---:|
| Week 1 entrants | +2.0% | +1.5% | +0.3% |
| Week 2 entrants | +2.1% | +1.4% | +0.4% |
| Week 3 entrants | +1.8% | +1.2% | pending |

Cohort analysis helps separate time since exposure, calendar time, acquisition mix, and product or market shocks. Suppose the treatment effect falls in week 3. Is that because users are losing interest after repeated exposure? Or because week 3 had a holiday, outage, promotion, or different acquisition channel mix?

Other useful diagnostics include:

- Plotting treatment effect by days since first exposure
- Comparing first-time exposure versus repeated exposure
- Checking whether the effect decays after users have seen the feature many times
- Looking for large early movement in attention metrics but smaller movement in value metrics

The practical question is:

> Would the treatment still look good after users stop reacting to the fact that it is new?

### When the Experiment Cannot Run Long Enough

Sometimes the decision metric needs more time than the main experiment can reasonably run. The team then has two immediate design options: predefine a follow-up readout, or roll out with a long-term holdout. A third option, predicting the long-term metric, is important enough to discuss separately in the next section.

**Predefine a follow-up readout**

The team can make an initial decision using mature short-term evidence, then revisit delayed metrics when they become observable. For example, a team may read out 7-day results first, then revisit 30-day retention, refund-adjusted revenue, or repeat purchase later. The important point is to define the follow-up before the team sees the results. Otherwise, long-term analysis can become selective storytelling.

**Roll out with a long-term holdout**

If the short-term result is strong enough to ship, the team can launch the treatment to most traffic while keeping a small long-term holdout on the old experience. This preserves a randomized comparison while allowing most users to receive the new version.

The purpose of the holdout is to answer questions that a short A/B test cannot answer:

- Does the effect persist?
- Does the effect decay after novelty fades?
- Does the effect grow as users learn?
- Does the treatment change retention or repeat behavior?
- Does the treatment affect ecosystem health over time?
- Does a model update improve long-term value or only short-term optimization?

For example, a recommendation team may launch a new ranking model to 95% of traffic but keep 5% on the old model for several months. This makes it possible to compare long-term retention, repeat purchase, content diversity, or seller ecosystem outcomes.

Long-term holdouts are most useful when the treatment affects habit formation, ranking, recommendation, pricing, incentives, marketplace feedback loops, or other parts of the product where a wrong full rollout would be costly.

The holdout should be planned carefully. Key design choices include:

- **Unit**: the holdout unit should match the level where treatment is stable and where interference is manageable, such as users, accounts, sellers, creators, markets, geos, or traffic slices.
- **Size**: the holdout should be large enough to estimate long-term metrics but small enough to limit opportunity cost.
- **Duration**: the duration should match the time scale of the decision metric, such as 30 days for early retention, 60-90 days for paid subscription survival, or several months for repeat purchase and ecosystem effects.
- **Eligibility**: the holdout population should be clearly defined.
- **Consistency**: holdout units should receive a consistent experience unless the design explicitly allows switching.
- **Governance**: teams should document why the holdout exists, what metrics it protects, who owns it, and when it will be reviewed.

Holdouts preserve a counterfactual, but they are not free. They can create opportunity cost, product inconsistency, engineering complexity, maintenance of old code paths, and pressure to remove the holdout. For small UI changes with clear short-term outcomes, a standard experiment may be enough.

## Predicting Long-Term Metrics

When the true outcome is delayed, another option is to predict the long-term metric from early behavior. LTV is the most common business example. Many teams care about user value over time, but true lifetime value is usually not observable during an experiment.

Teams may use a windowed realized metric, such as 30-day or 90-day LTV, or a predicted LTV model based on early behavior and historical cohorts. A windowed metric is still an observed outcome, just with a fixed horizon.

For a fixed window:

$$
\text{90-day LTV}
=
\frac{\text{Total net value from a cohort over 90 days}}
{\text{Number of users in the cohort}}
$$

A predicted LTV model is faster. For example, a model may predict one-year value using the first seven days of behavior. The model can then produce a predicted long-term value for each user in the experiment, and the team can compare the average predicted value between treatment and control.

This idea is useful, but it changes the question. The experiment is no longer only measuring an observed outcome. It is also relying on a model that connects early behavior to future value.

### Common Prediction Methods

There are several common ways to predict long-term value. They differ in how much structure they impose, how much data they need, and how easily the result can be interpreted.

**RFM and customer-base models**

Classic customer lifetime value models often start from recency, frequency, and monetary value, or RFM. Recency measures how recently a customer purchased, frequency measures how often they purchased, and monetary value measures how much they spent.

Fader, Hardie, and Lee (2005a) connect RFM-style customer histories to CLV through iso-value curves, and Fader, Hardie, and Lee (2005b) introduce the BG/NBD model as a practical alternative to the Pareto/NBD framework. These models are especially useful for repeat-purchase businesses because they explicitly model future purchasing behavior from past purchasing patterns.

The strength of these models is interpretability. The weakness is that they rely on assumptions about purchase and dropout behavior, and they may be less useful for new users with little or no history.

**Supervised prediction models**

Many industry teams frame LTV prediction as a supervised learning problem. The label may be 30-day revenue, 90-day contribution margin, one-year value, or another delayed outcome. The features may include acquisition channel, first purchase, first-week activity, subscription plan, product category, refunds, device, geography, and engagement frequency.

This approach is flexible and can use many signals. It is often useful when the product has rich behavioral data or when many users have limited purchase history. The main risk is that predictive accuracy does not automatically imply causal validity in an experiment.

**Probabilistic LTV models**

LTV is often difficult to model because many users have zero future value, while a small number of users have very large value. Wang, Liu, and Miao (2019) propose a zero-inflated lognormal model for LTV prediction. The zero-inflated part models whether future value is zero, while the lognormal part models positive future value.

This type of model is useful because it matches two common facts about LTV: many users churn or never purchase again, and the positive-value distribution is heavy-tailed.

**Uncertainty-aware prediction models**

Some prediction methods also estimate uncertainty, not only a point prediction. Cao, Xu, and Yang (2024) use Monte Carlo Dropout to produce LTV predictions with uncertainty estimates in a mobile game setting. Their application predicts future purchasing amount after observing a new user's first week of behavior.

This is useful for business decisions because two users may have the same predicted LTV but very different uncertainty. A team may treat a high-predicted-value, high-confidence user differently from a high-predicted-value, low-confidence user. For experiments, uncertainty also reminds the team that predicted LTV is a modeled quantity, not a directly observed outcome.

**Surrogate index and delayed reward forecasting**

A formal version of this idea is the surrogate index approach in Athey et al. (2026). Instead of relying on one short-term proxy, the method combines multiple short-term outcomes into a predicted long-term outcome. In an experiment, the team can compare treatment and control on this surrogate index to estimate the long-term treatment effect faster.

Wayfair's delayed reward forecasting system is an industry example of the same idea (Wang 2021). Wayfair describes training models on historical data where both short-term customer activity and delayed rewards are observed, then applying those models to experiment users using early leading indicators and pre-treatment customer context. The predicted delayed reward can then be averaged by treatment group to estimate the likely long-term impact before the full reward window matures.

This is the most directly relevant approach for experiment analysis, because it is designed around treatment-control comparisons rather than only individual prediction.

Two newer extensions are worth knowing. Huang et al. (2026) study long-term treatments, where the product change is not a one-time intervention but a treatment that continues over time. Their longitudinal surrogate framework uses short-term experimental data to estimate future treatment effects, and they validate the approach using two large-scale WeChat experiments.

Saito et al. (2024a) study long-term off-policy evaluation for algorithms. Their setting is especially relevant for recommendation and ranking systems, where short-term rewards such as clicks may differ from long-term outcomes such as engagement or retention. The related Spotify Research article explains the same idea in product language: combine historical data, short-term experiment data, and long-term rewards to estimate long-term algorithm performance faster (Saito et al. 2024b).

A common way to think about LTV is:

$$
\text{Expected LTV}
=
\sum_{t=0}^{T}
P(\text{active at }t)
\times
E(\text{value at }t \mid \text{active})
$$

This formula is not meant to be a universal LTV model. It shows the basic idea: long-term value depends on both retention and value conditional on retention.

### Using Predicted Metrics in Experiments

Predicted long-term metrics can be useful as early decision-support metrics, but prediction accuracy alone is not enough. For experiment analysis, the model must preserve the treatment-control comparison.

The main risk is that the treatment changes the relationship between early behavior and long-term value. Suppose a recommendation model increases first-week purchases by showing aggressive discounts. A historical model may interpret those purchases as a strong signal of future value. But if the discounts attract low-loyalty buyers, the predicted LTV lift may overstate the true long-term effect.

For this reason, predicted long-term metrics should be validated with backtests on mature historical experiments whenever possible. The team should check whether the model would have predicted the long-term treatment effect correctly using only early experiment data.

Predicted long-term metrics should be decomposed into interpretable pieces:

| Component | Example Metric |
|---|---|
| Acquisition or activation | Signup, trial start, first purchase |
| Conversion quality | Paid conversion, first-order cancellation |
| Repeat behavior | Repeat purchase, repeat session, repeat order |
| Retention | 7-day, 30-day, 90-day retention |
| Monetary value | Net revenue, contribution margin, GMV |
| Quality adjustment | Refunds, returns, complaints, disputes |

This matters because a treatment can improve one component while hurting another. For example, more users may start trials, but fewer become paid subscribers. More buyers may make a first purchase, but repeat purchase may fall. More orders may happen today, but refund-adjusted revenue may decline.

If an experiment reports a predicted LTV lift, the team should ask:

- Is the metric realized or predicted?
- What time horizon does it use?
- What costs are included?
- Which component moved?
- Could the treatment change the relationship between early behavior and future value?

If the team launches without a long-term holdout, it should still monitor delayed metrics after rollout. This monitoring is not as clean as a randomized comparison, because there may no longer be a control group. But it can detect warning signs such as retention decay, refund increases, repeat purchase decline, seller concentration, ecosystem quality degradation, or delayed support burden.

## Product Example: Recommendation Ranking and Long-Term Value

Suppose an e-commerce platform updates its recommendation ranking model. The old model optimizes short-term GMV. The new model adds a signal for repeat-purchase GMV between the same user and seller.

A short experiment may show:

- GMV per user: +2.0%
- GPM: +3.0%
- Repeat-purchase GMV: +4.5%

This is encouraging, but the team should still ask whether the change creates durable value.

Fast signals:

- More clicks on recommended products
- More first purchases
- Higher short-term GMV
- Higher GPM

Long-term decision metrics:

- 30-day and 90-day repeat purchase
- Refund-adjusted GMV
- Return rate
- User retention
- Seller concentration
- Cold-start seller exposure
- Active seller supply

The treatment-control gap over time is especially important. If the GMV gap widens over time, it suggests the model is identifying durable user-seller relationships rather than only reallocating short-term traffic.

The team should also look for failure modes:

- Novelty: users initially explore the new ranking but stop engaging later.
- Composition: the model increases low-quality purchases or returns.
- Accumulation: traffic shifts toward a smaller group of sellers over time.
- Delayed harm: refund rate, return rate, or seller concentration worsens after the initial readout.

A long-term holdout may be justified because ranking changes can reshape user behavior, seller incentives, and marketplace supply.

## Practical Workflow

A practical workflow for long-term experiment analysis is:

1. Define the long-term business outcome before the experiment.
2. Identify likely mismatch patterns: novelty, learning, accumulation, composition, or delayed harm.
3. Decide whether the main experiment can run long enough for the decision metric to mature.
4. If the trend is visible, report treatment effects over time, including cumulative and period-by-period effects.
5. If the experiment cannot run long enough, predefine follow-up readouts, keep a long-term holdout, or use predicted long-term metrics.
6. If the trend is ambiguous, use diagnostics that match the suspected mechanism, such as cohort analysis or new-versus-existing-user comparison.
7. Validate predicted long-term metrics with backtests, then decompose them into interpretable components.
8. Monitor delayed metrics after launch, especially when the launch relied on leading indicators.

## Common Mistakes

**Treating early lift as permanent**

A large first-week effect may reflect novelty, attention, or temporary curiosity.

**Ignoring delayed quality metrics**

More signups, applications, purchases, or messages may not mean more durable value.

**Stopping before users can learn**

Some treatments need time before users, sellers, creators, or models adapt.

**Reading only the cumulative effect**

The cumulative effect can look positive even when the treatment is fading or reversing in later periods.

**Using predicted LTV as one opaque number**

Predicted LTV should be validated against mature outcomes and decomposed into retention, monetization, repeat behavior, and quality components.

**Keeping holdouts without governance**

Long-term holdouts have cost. They need owners, metrics, review dates, and a reason to exist.

## Key Takeaways

- Short-term experiment results may not capture long-term product value.
- Short-term results can mislead because of novelty, learning, accumulation effects, composition effects, or delayed harm.
- If the experiment runs long enough, plot the treatment effect over time and report both cumulative and period-by-period effects.
- If the experiment cannot run long enough, use predefined follow-up readouts, cautious rollout, long-term holdouts, or predicted long-term metrics.
- If the trend is ambiguous, diagnostics such as cohort analysis and new-versus-existing-user comparison can test specific hypotheses.
- Predicted long-term metrics should be validated, then decomposed into interpretable components instead of treated as one opaque number.
- Long-term holdouts preserve a counterfactual, but they have opportunity cost and should be governed carefully.

## References

- Athey, Susan, Raj Chetty, Guido W. Imbens, and Hyunseung Kang. "The Surrogate Index: Combining Short-Term Proxies to Estimate Long-Term Treatment Effects More Rapidly and Precisely." *The Review of Economic Studies* 93, no. 4 (2026): 2284-2312. [https://doi.org/10.1093/restud/rdaf087](https://doi.org/10.1093/restud/rdaf087).
- Cao, Xinzhe, Yadong Xu, and Xiaofeng Yang. "Customer Lifetime Value Prediction with Uncertainty Estimation Using Monte Carlo Dropout." arXiv preprint arXiv:2411.15944 (2024). [https://arxiv.org/abs/2411.15944](https://arxiv.org/abs/2411.15944).
- Fader, Peter S., Bruce G. S. Hardie, and Ka Lok Lee. "RFM and CLV: Using Iso-Value Curves for Customer Base Analysis." *Journal of Marketing Research* 42, no. 4 (2005a): 415-430. [https://doi.org/10.1509/jmkr.2005.42.4.415](https://doi.org/10.1509/jmkr.2005.42.4.415).
- Fader, Peter S., Bruce G. S. Hardie, and Ka Lok Lee. "'Counting Your Customers' the Easy Way: An Alternative to the Pareto/NBD Model." *Marketing Science* 24, no. 2 (2005b): 275-284. [https://doi.org/10.1287/mksc.1040.0098](https://doi.org/10.1287/mksc.1040.0098).
- Huang, Shan, Chen Wang, Yuan Yuan, Jinglong Zhao, and Brocco (Jingjing) Zhang. "Estimating Effects of Long-Term Treatments." *Management Science* (2026). [https://doi.org/10.1287/mnsc.2023.02575](https://doi.org/10.1287/mnsc.2023.02575).
- Saito, Yuta, Himan Abdollahpouri, Jesse Anderton, Ben Carterette, and Mounia Lalmas. "Long-Term Off-Policy Evaluation and Learning." In *Proceedings of the ACM Web Conference 2024*, 3432-3443. 2024a. [https://doi.org/10.1145/3589334.3645446](https://doi.org/10.1145/3589334.3645446).
- Saito, Yuta, Himan Abdollahpouri, Jesse Anderton, Ben Carterette, and Mounia Lalmas. "Estimating Long-Term Outcome of Algorithms." Spotify Research, May 9, 2024b. [https://research.atspotify.com/2024/5/estimating-long-term-outcome-of-algorithms](https://research.atspotify.com/2024/5/estimating-long-term-outcome-of-algorithms).
- Wang, Xiaojing, Tianqi Liu, and Jingang Miao. "A Deep Probabilistic Model for Customer Lifetime Value Prediction." arXiv preprint arXiv:1912.07753 (2019). [https://arxiv.org/abs/1912.07753](https://arxiv.org/abs/1912.07753).
- Wang, Le. "Speeding up A/B Tests with Delayed Reward Forecasting." Wayfair Tech Blog, October 29, 2021. [https://www.aboutwayfair.com/careers/tech-blog/speeding-up-a-b-tests-with-delayed-reward-forecasting](https://www.aboutwayfair.com/careers/tech-blog/speeding-up-a-b-tests-with-delayed-reward-forecasting).
