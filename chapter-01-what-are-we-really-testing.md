# What Are We Really Testing?

Most A/B tests look simple on the surface. We show version A to one group of users, version B to another group, wait for data, and compare the metrics.

But in practice, the hard part is often not the statistical test. The hard part is knowing what question we are actually trying to answer.

A product team may say, "We want to test a new recommendation algorithm." But that is not yet a hypothesis. A designer may say, "We want to test a new checkout page." That is not yet a success metric. A manager may say, "We want to increase engagement." That is not yet a launch decision.

A good experiment starts by translating a product change into a causal question.

For example:

> If we show users a new recommendation algorithm, does it increase meaningful user value compared with the current algorithm?

This already contains the core ingredients of an experiment:

- A treatment: the new recommendation algorithm
- A control: the current algorithm
- A population: the users who may receive recommendations
- An outcome: meaningful user value
- A comparison: treatment versus control

The phrase "meaningful user value" is still vague, so the next step is to define metrics.

## From Business Goal to Metric

Suppose we work on a video app. The team wants to update the recommendation algorithm. The business goal might be to increase user engagement. A natural primary metric could be watch time per user.

But watch time alone can be dangerous. A model may increase watch time by recommending sensational, repetitive, or low-quality content. Users may watch more today but feel worse about the product tomorrow.

So we separate metrics into three types.

**Primary metric**

The main metric used to decide whether the experiment succeeds.

**Secondary metrics**

Metrics that help explain why the primary metric moved.

**Guardrail metrics**

Metrics that should not get worse, even if the primary metric improves.

For the video recommendation example:

Primary metric:
- Watch time per user

Secondary metrics:
- Video click-through rate
- Completion rate
- Like rate
- Follow rate

Guardrail metrics:
- Reported content
- Not-interested rate
- 7-day retention
- Content diversity
- Creator ecosystem health

The primary metric captures the intended upside. Secondary metrics explain the mechanism. Guardrails protect us from winning in the wrong way.

## The Null Hypothesis

In a standard A/B test, the null hypothesis is usually that the treatment has no effect.

For example:

> Null hypothesis: the new recommendation algorithm does not change watch time per user.

The alternative hypothesis is what we hope to find:

> Alternative hypothesis: the new recommendation algorithm increases watch time per user.

This distinction sounds basic, but it matters. In product conversations, people often say, "Our hypothesis is that this will improve retention." Statistically, that is the alternative hypothesis. The null is the boring but important default: nothing changed.

The experiment is designed to collect enough evidence to reject the null hypothesis.

## Statistical Significance Is Not the Decision

A common mistake is treating statistical significance as the launch decision.

Imagine a checkout experiment:

- Purchase conversion rate: +0.2%, statistically significant
- Average order value: -3%, statistically significant
- Revenue per visitor: no significant change
- Refund rate: slightly higher

Should we launch?

Not necessarily.

Statistical significance tells us that the observed effect is unlikely to be random noise under the null hypothesis. It does not tell us whether the effect is large enough, profitable enough, safe enough, or strategically useful.

A launch decision should consider:

- Statistical significance
- Practical significance
- Guardrail metrics
- Long-term effects
- Business tradeoffs
- Implementation cost
- Risk of user or ecosystem harm

In other words, an experiment result is evidence. It is not the decision itself.

## Example: When Engagement Improves but Safety Gets Worse

Suppose we run an A/B test for a new recommendation algorithm on a video app. The treatment group shows:

- Watch time per user: +4%, statistically significant
- Like rate: -2%
- 7-day retention: no significant change
- Reported content: +8%, statistically significant

At first glance, the experiment looks positive because the primary engagement metric improved. Users in the treatment group are watching more.

But the full metric pattern tells a more complicated story. Like rate decreased, retention did not improve, and reported content increased significantly. This suggests that the extra watch time may be coming from lower-quality, more controversial, or potentially harmful content rather than genuinely better recommendations.

This is not a clean win. Reported content is a trust and safety guardrail, and an 8% increase is a serious warning sign. Before making a launch decision, the team should understand whether the increase is large in absolute terms, what types of reports increased, and whether the effect is concentrated in certain content categories, user segments, or creator groups.

The next layer of analysis should look at deeper indicators of user value and platform health: long-term retention, user satisfaction, hides or dislikes, content diversity, moderation burden, and repeated exposure to reported content. Since 7-day retention did not improve, the watch-time gain may not represent durable user value.

A broad launch would be difficult to justify from this result alone. A better next step would be to iterate on the algorithm with safety constraints, such as downranking borderline content, adding content-quality filters, or optimizing a ranking objective that balances watch time with satisfaction and safety. If some segments show higher watch time without increased reports, a limited rollout may be reasonable there.

The lesson is that a primary metric can improve while the product gets worse. Guardrails are not secondary because they are unimportant. They are secondary because they define the conditions under which the primary metric is allowed to win.

## Example: When the Funnel Gets Wider but Lower Intent

Now consider a subscription app testing a new onboarding flow. The treatment group shows:

- Signup completion rate: +8%, statistically significant
- Trial start rate: +5%, statistically significant
- Paid conversion after trial: -6%, statistically significant
- 30-day retention among paid users: no significant change

This result has a common funnel pattern. The new onboarding flow reduces friction, so more users complete signup and start a trial. But the users added by the easier flow may have lower purchase intent, so the conversion rate from trial to paid subscription falls.

The key is not to judge each step of the funnel in isolation. A lower paid conversion rate after trial is not automatically bad if the total number of paid subscribers increases. The right question is whether the new flow increases paid subscribers, revenue, or expected lifetime value per user entering onboarding.

For example, suppose 10,000 users enter onboarding.

In the control group:

- 40% start a trial, producing 4,000 trials
- 20% of trials convert to paid, producing 800 paid subscribers

In the treatment group:

- Trial starts increase by 5%, so the trial start rate becomes 42%
- Paid conversion after trial decreases by 6%, so the paid conversion rate becomes 18.8%
- The result is 10,000 x 42% x 18.8%, or about 790 paid subscribers

In this simplified example, the new onboarding flow creates more trials but slightly fewer paid subscribers. The top of the funnel improved, but the end outcome did not.

The opposite result is also possible. If the increase in trial starts is large enough, total paid subscribers may still increase even though the conditional paid conversion rate falls. This is why funnel metrics should be interpreted together, not as separate wins and losses.

Additional checks are still needed. The team should look at trial abuse, cancellation rate, refund rate, support tickets, payment failure, acquisition channel mix, and longer-term retention. Segment analysis may show that the new onboarding helps high-intent organic users but attracts low-intent users from certain paid channels, or that it works better on mobile than desktop.

This example shows why the denominator matters. "Paid conversion after trial" is conditional on users who started a trial. If the treatment changes who enters that denominator, the metric can move for two reasons: the product experience changed, or the composition of trial users changed. Good experiment analysis separates these two stories.

## Practical Significance

Suppose a large company runs an experiment on millions of users and finds that a button color change increases click-through rate by 0.03%, with a p-value below 0.01.

This may be statistically significant, but practically meaningless.

Practical significance asks:

> Is the effect large enough to matter?

For business metrics, this often means converting the lift into absolute impact.

For example:

- How many additional purchases?
- How much incremental revenue?
- How many retained users?
- How much engineering or operational cost?
- How much risk?

A tiny effect can be statistically significant with enough traffic. A large effect can be statistically insignificant if the sample size is too small. Good experiment analysis needs both statistical and practical thinking.

## A Simple Product Example

Suppose a food delivery app wants to test showing delivery time as a range, such as "25-35 minutes," instead of a point estimate, such as "30 minutes."

A weak experiment setup would be:

> Test whether users like the new estimate format.

A stronger setup would be:

> We want to test whether showing a delivery time range improves expectation-setting and reduces post-order cancellations or delivery-time complaints, without reducing order conversion.

Now the experiment becomes clearer.

Eligible users:
- Users who view a restaurant or checkout page with an estimated delivery time

Randomization unit:
- User level, so the same user consistently sees one format

Primary metric:
- Post-order cancellation rate or delivery-time complaint rate

Secondary metrics:
- Time from order placement to cancellation
- Share of cancellations caused by delivery-time concerns
- User clicks on delivery details or tracking page
- Difference between estimated and actual delivery time
- Delivery-time satisfaction rating, if available

Guardrail metrics:
- Order conversion rate
- Checkout completion rate
- Refund requests
- Support contacts
- Repeat order rate
- Actual delivery time

This structure turns a vague product idea into a testable causal question.

## The Real Question

At the end of an experiment, the question is rarely just:

> Did the metric move?

The better question is:

> Did the treatment create real value, and do we understand the tradeoffs well enough to make a decision?

That is the mindset behind good experimentation.

A/B testing is not just about detecting differences. It is about making decisions under uncertainty.

## Key Takeaways

A good experiment starts with a clear causal question.

The primary metric should reflect the main business or user goal.

Secondary metrics explain the mechanism.

Guardrail metrics protect against harmful side effects.

Statistical significance is not the same as business significance.

The final decision should consider the full pattern of evidence, not one metric in isolation.
