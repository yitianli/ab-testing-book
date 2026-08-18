# From Product Idea to Experiment Question

Most A/B tests look simple on the surface. We show version A to one group of users, version B to another group, wait for data, and compare the metrics.

But in practice, the hard part is often not the statistical test. The hard part is knowing what question the experiment is supposed to answer.

A product team may say, "We want to test a new recommendation algorithm." But that is not yet a hypothesis. A designer may say, "We want to test a new checkout page." That is not yet a success metric. A manager may say, "We want to increase engagement." That is not yet a launch decision.

A good experiment starts by translating a product idea into a decision-ready causal question.

The flow is:

> Product idea -> decision -> causal question -> metrics -> hypothesis -> evidence -> decision

This chapter walks through that flow.

## Start With the Product Decision

An experiment should begin with the decision the team needs to make.

Suppose a food delivery app currently shows a point estimate for delivery time:

> 30 minutes

The product team wants to test showing a range instead:

> 25-35 minutes

The vague version of the experiment is:

> Test whether users like the new estimate format.

This is not enough. It does not say what "like" means, which users matter, what risk the team is worried about, or what decision will follow.

A better starting point is:

> Should we launch delivery-time ranges because they set expectations better than point estimates?

Now the experiment is connected to a product decision, but it is still not precise enough to analyze.

## Translate the Decision Into a Causal Question

The decision becomes analyzable when it is turned into a causal question.

A causal question asks what would happen if the same eligible users received treatment instead of control.

For the delivery-time example:

> Does showing a delivery-time range reduce post-order cancellations or delivery-time complaints among users who see delivery estimates, compared with showing a point estimate?

This question names the main pieces of the experiment:

- Treatment: show a delivery-time range, such as "25-35 minutes"
- Control: show a point estimate, such as "30 minutes"
- Eligible population: users who see a restaurant or checkout page with an estimated delivery time
- Intended benefit: fewer cancellations or delivery-time complaints
- Comparison: treatment versus control

This is already much clearer than "test whether users like it."

The causal question also prevents a common mistake: testing a feature without knowing what success means. If the team only looks for any metric that improves, the analysis can become a search for a convenient story.

## Choose Metrics That Match the Question

With the causal question in place, the team can choose metrics.

Metrics usually play different roles.

| Metric Type | Role |
|---|---|
| Primary metric | The main metric used to judge whether the experiment succeeded |
| Secondary metrics | Metrics that explain how or why the primary metric moved |
| Guardrail metrics | Metrics that should not get worse, even if the primary metric improves |

For the delivery-time experiment, the metric set might be:

Primary metric:

- Post-order cancellation rate caused by delivery-time concerns

Secondary metrics:

- Delivery-time complaint rate
- Time from order placement to cancellation
- Share of cancellations related to delivery-time expectations
- Clicks on delivery details or tracking page
- Difference between estimated and actual delivery time
- Delivery-time satisfaction rating, if available

Guardrail metrics:

- Order conversion rate
- Checkout completion rate
- Refund requests
- Support contacts
- Repeat order rate
- Actual delivery time

The primary metric captures the intended upside and should be close to the business or user outcome the team cares about. In this example, cancellation rate is closer to the final business goal than complaint rate because a canceled order directly affects completed transactions. Secondary metrics explain the mechanism. Guardrails protect the product from winning in the wrong way.

For example, delivery-time ranges might reduce cancellations but also lower order conversion because some users dislike seeing uncertainty. That tradeoff matters. The treatment is not successful just because one metric improves.

## Define the Hypothesis

After the metric is chosen, the statistical hypothesis becomes easier to state.

In a standard A/B test, the null hypothesis is usually that the treatment has no effect.

For example:

> Null hypothesis: showing a delivery-time range does not change the post-order cancellation rate caused by delivery-time concerns.

The alternative hypothesis is the effect the team hopes to detect:

> Alternative hypothesis: showing a delivery-time range reduces the post-order cancellation rate caused by delivery-time concerns.

This distinction sounds basic, but it matters. In product conversations, people often say, "Our hypothesis is that this will improve the experience." Statistically, that is the alternative hypothesis. The null is the boring but important default: nothing changed.

The experiment is designed to collect enough evidence to reject the null hypothesis.

## Do Not Confuse Evidence With Decision

Rejecting the null hypothesis is not the same as making a launch decision.

Imagine the delivery-time experiment shows:

| Metric | Result |
|---|---:|
| Post-order cancellation rate caused by delivery-time concerns | -1.5%, statistically significant |
| Order conversion rate | -0.4%, statistically significant |
| Refund requests | No significant change |
| Repeat order rate | No significant change |

The experiment found evidence that delivery-time-related cancellations decreased. But it also found evidence that conversion decreased.

Should the team launch?

Not automatically.

Statistical significance tells us that an observed effect is unlikely to be random noise under the null hypothesis. It does not tell us whether the effect is large enough, close enough to the business goal, profitable enough, safe enough, or strategically useful.

A launch decision should consider:

- Statistical significance
- Practical significance
- Guardrail metrics
- Long-term effects
- Business tradeoffs
- Implementation cost
- Risk of user or ecosystem harm

In other words, an experiment result is evidence. It is not the decision itself.

Real experiments are often more complicated than this simple example. A primary metric may improve while a guardrail gets worse, or the top of a funnel may improve while the final business outcome gets worse.

We return to this distinction in Chapter 15, where we discuss practical significance, guardrail tradeoffs, rollout strategy, and decision-making under uncertainty in more detail.

## Practical Significance

Practical significance asks:

> Is the effect large enough to matter?

Suppose a large company runs an experiment on millions of users and finds that a button color change increases click-through rate by 0.03%, with a p-value below 0.01.

This may be statistically significant, but practically meaningless.

For business metrics, practical significance often means translating the lift into absolute impact:

- How many additional purchases?
- How much incremental revenue?
- How many retained users?
- How much engineering or operational cost?
- How much risk?

A tiny effect can be statistically significant with enough traffic. A large effect can be statistically insignificant if the sample size is too small. Good experiment analysis needs both statistical and practical thinking.

In the delivery-time example, a 1.5% reduction in cancellation rate may matter a lot if cancellations are frequent, costly, or damaging to user trust. But if delivery-time-related cancellations are already extremely rare, the same relative reduction may not justify a more complex user experience or a conversion loss. Practical significance depends on absolute impact, not only relative lift.

## The Decision-Ready Question

By the end of the setup, the original product idea should become a question specific enough that the team knows what result would support launch.

It is no longer:

> Should we show delivery time as a range?

It is:

> Does showing a delivery-time range reduce delivery-time-related cancellations among eligible users enough to justify launch, while keeping order conversion, refunds, support contacts, and repeat orders healthy?

This question is useful because it names the main pieces of the experiment:

- The treatment
- The control
- The eligible population
- The intended benefit
- The guardrails
- The decision

It also makes the decision rule clearer. A result that reduces cancellations while keeping guardrails healthy would support launch. A result that reduces cancellations only by sacrificing too much conversion would require iteration, segmentation, or no launch.

That is the standard this book will use. A good experiment is not just a comparison between treatment and control. It is a structured way to answer a product decision under uncertainty.

## Key Takeaways

A good experiment starts with a product decision, not just a feature idea.

The product decision should be translated into a causal question: what happens if eligible users receive treatment instead of control?

The primary metric should reflect the intended benefit.

Secondary metrics explain the mechanism.

Guardrail metrics protect against harmful side effects.

The null hypothesis is usually that treatment has no effect. The alternative hypothesis describes the effect the team hopes to detect.

Statistical significance is evidence, not a launch decision.

The final decision should consider the full pattern of evidence, including practical significance, guardrails, long-term effects, cost, and risk.
