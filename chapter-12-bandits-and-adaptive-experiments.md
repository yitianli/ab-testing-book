# Bandits and Adaptive Experiments

Most A/B tests use fixed allocation.

For example:

- 50% of users see control
- 50% of users see treatment

The allocation does not change while the experiment is running. This makes the experiment easy to analyze. Treatment and control groups are comparable because assignment was randomized and fixed by design.

But fixed allocation can feel wasteful when there are many variants and some are clearly worse.

Suppose a product team tests four notification messages:

| Variant | True Click Rate |
|---|---:|
| A | 2.0% |
| B | 2.1% |
| C | 3.5% |
| D | 1.4% |

A fixed 25/25/25/25 experiment keeps sending users to weak variants while it waits for enough data.

A bandit experiment asks a different question:

> Can we learn which option is better while gradually sending more traffic to better-performing options?

This chapter explains multi-armed bandits, exploration versus exploitation, regret, common bandit algorithms, and when adaptive experiments are useful.

The central distinction is:

> A/B tests are usually designed for clean measurement. Bandits are usually designed for learning while optimizing.

## The Basic Bandit Problem

The name comes from the idea of several slot machines, sometimes called "one-armed bandits." Each machine has an unknown payout rate. The player wants to earn as much reward as possible while learning which machine is best.

In product experimentation:

- Each machine is a variant.
- Pulling an arm means assigning a user, session, impression, or request to a variant.
- The reward is the observed outcome, such as click, purchase, revenue, or retention proxy.

The challenge is that the platform faces two goals at the same time:

**Exploration**

Try different variants to learn their performance.

**Exploitation**

Send more traffic to the variant that currently looks best.

Pure exploration learns well but may waste traffic on bad variants. Pure exploitation earns more short-term reward but may get stuck on a variant that looked good early by chance.

Bandit algorithms manage this tradeoff.

## Regret

Regret measures the cost of not always choosing the best variant.

Suppose the best variant has expected reward $\mu^*$, and the algorithm chooses variant $A_t$ at time $t$ with expected reward $\mu_{A_t}$.

The expected regret after $T$ assignments is:

$$
R_T
=
\sum_{t=1}^{T}
(\mu^* - \mu_{A_t})
$$

In words:

> Regret is the reward lost because the algorithm sometimes chose a non-best option.

For a fixed A/B test, regret can be high because the experiment keeps assigning traffic to weak variants at the planned rate.

For a bandit, regret can be lower because the algorithm shifts traffic toward better variants as evidence accumulates.

This is the main reason bandits are attractive in product settings where every assignment has business cost.

## A/B Tests and Bandits Answer Different Questions

A standard A/B test usually asks:

> What is the treatment effect, and how uncertain is it?

A bandit usually asks:

> Which option should receive more traffic as we learn?

These questions overlap, but they are not identical.

A fixed A/B test is often better when the goal is causal measurement:

- Estimate a launch effect
- Compare treatment and control cleanly
- Measure guardrails
- Produce a final confidence interval
- Understand downstream effects

A bandit is often better when the goal is adaptive optimization:

- Choose the best headline
- Tune creative variants
- Allocate traffic among promotions
- Personalize recommendations
- Reduce traffic sent to weak options

The difference matters because adaptive allocation changes the data distribution. Later users are more likely to receive variants that looked good earlier. This can complicate inference if the team tries to analyze the bandit as if it were a fixed A/B test.

## Epsilon-Greedy

Epsilon-greedy is one of the simplest bandit algorithms.

At each decision:

- With probability $1-\epsilon$, choose the variant with the highest observed average reward.
- With probability $\epsilon$, choose a random variant.

For example, if $\epsilon = 0.1$, the algorithm exploits 90% of the time and explores 10% of the time.

The intuition is simple:

> Mostly use the current best option, but keep trying other options so the algorithm can recover from early noise.

Epsilon-greedy is easy to understand and implement.

But it has weaknesses:

- It explores randomly, even when some variants are clearly bad.
- It requires choosing $\epsilon$.
- If $\epsilon$ is too small, the algorithm may get stuck early.
- If $\epsilon$ is too large, the algorithm wastes too much traffic exploring.

Some versions reduce $\epsilon$ over time, exploring more early and exploiting more later.

## Upper Confidence Bound

Upper Confidence Bound, or UCB, chooses variants based on both estimated reward and uncertainty.

The algorithm gives each variant a score:

$$
\text{UCB}_a
=
\hat{\mu}_a
+
c
\sqrt{
\frac{\log t}{n_a}
}
$$

where:

- $\hat{\mu}_a$ is the observed average reward for variant $a$
- $n_a$ is the number of times variant $a$ has been tried
- $t$ is the total number of assignments so far
- $c$ controls how much the algorithm values exploration

The first term rewards variants that have performed well. The second term rewards variants with high uncertainty.

A variant that has been tried only a few times may get a high UCB score because the algorithm is still uncertain about it. As the variant receives more traffic, its uncertainty term shrinks.

The intuition is:

> Choose the option with the best optimistic estimate.

UCB is more directed than epsilon-greedy. It does not explore purely at random. It explores variants that could plausibly be best.

## Thompson Sampling

Thompson sampling is a Bayesian-style bandit algorithm.

Instead of keeping only a point estimate for each variant, it keeps a probability distribution over the variant's reward rate.

At each decision:

1. Draw one possible reward rate from each variant's current distribution.
2. Choose the variant with the highest draw.
3. Observe the reward.
4. Update the distribution for that variant.

For a binary outcome, such as click or no click, a common model is:

$$
p_a \sim \text{Beta}(\alpha_a, \beta_a)
$$

where $p_a$ is the click probability for variant $a$.

After observing a click, update:

$$
\alpha_a \leftarrow \alpha_a + 1
$$

After observing no click, update:

$$
\beta_a \leftarrow \beta_a + 1
$$

The algorithm naturally balances exploration and exploitation:

- Variants with high estimated reward are sampled often.
- Variants with high uncertainty still get chances.
- Clearly bad variants are sampled less over time.

Thompson sampling is popular because it is intuitive, flexible, and often performs well in practice.

## Contextual Bandits

So far, the algorithms treat every user as exchangeable. They try to find the single best variant overall.

But in many products, the best variant depends on context.

For example:

- A discount may work better for price-sensitive users.
- A notification may work better at different times of day.
- A recommendation module may work better for returning users than new users.
- A creative message may work differently by country or device.

A contextual bandit uses features $X_i$ when choosing the action.

Instead of learning:

$$
E[Y \mid A=a]
$$

it learns:

$$
E[Y \mid A=a, X=x]
$$

The policy becomes:

$$
\pi(x) = \text{action chosen for context }x
$$

This connects bandits to HTE and uplift modeling from Chapter 10. Both are concerned with treatment effects that vary by user, item, or context.

The difference is that a contextual bandit adapts while data is being collected. It is not only estimating heterogeneity after the experiment; it is using the estimated heterogeneity to decide what to show next.

Contextual bandits are useful for:

- Personalized recommendations
- Notification timing
- Creative selection
- Search ranking modules
- Promotion targeting
- Dynamic content selection

They are powerful, but they are also harder to evaluate because the data is generated by an adaptive policy.

## Guardrails in Bandit Experiments

Bandits optimize a reward.

That reward may be too narrow.

For example:

- Optimizing clicks may increase low-quality clicks.
- Optimizing watch time may increase regretful consumption.
- Optimizing orders may increase refunds.
- Optimizing notifications may increase opt-outs.
- Optimizing short-term GMV may reduce repeat purchase or seller diversity.

A bandit should not be launched with only one reward metric and no guardrails.

Practical guardrails include:

- User complaints
- Opt-outs or unsubscribes
- Refunds and returns
- Latency
- Retention
- Diversity or concentration metrics
- Long-term value proxies

There are several ways to use guardrails:

- Stop the bandit if a guardrail crosses a threshold.
- Penalize the reward when guardrails worsen.
- Restrict the eligible action set to safe variants.
- Run a fixed holdout to monitor long-term effects.
- Review reward and guardrail tradeoffs before full automation.

The key question is:

> Is the bandit optimizing the behavior we actually want?

## Delayed Rewards

Bandits are easiest when rewards are observed quickly.

Examples:

- Click
- Like
- Open
- Add to cart
- Immediate conversion

But many important outcomes are delayed:

- 7-day retention
- Paid conversion after trial
- Repeat purchase
- Refund-adjusted revenue
- Long-term satisfaction

Delayed rewards create a problem. If the bandit updates based on fast metrics, it may optimize the wrong thing. If it waits for long-term metrics, learning becomes slow.

Common responses include:

- Use a fast proxy reward, but validate it against long-term metrics.
- Use delayed reward updates when the delay is manageable.
- Combine short-term reward with long-term guardrails.
- Keep a long-term holdout to measure delayed effects.
- Avoid bandits when the decision metric is too delayed or too complex.

For example, a subscription app should be careful about using trial start as the only bandit reward. The algorithm may learn to attract low-intent users who start trials but do not become durable subscribers.

## Nonstationarity

Bandits assume that learning from the past helps future decisions.

But product environments change.

Examples:

- User behavior changes by season.
- A competitor launches a promotion.
- A recommendation model is updated.
- Inventory changes.
- A creative message becomes stale.
- The user population shifts after a marketing campaign.

This is called nonstationarity.

If the best action changes over time, a bandit needs to keep exploring. Otherwise, it may overcommit to an option that used to be best.

Practical responses include:

- Keep a minimum exploration rate.
- Use recent data more heavily than old data.
- Reset or refresh the bandit periodically.
- Monitor performance by calendar time and segment.
- Keep a fixed control or benchmark policy.

Nonstationarity is one reason bandits require ongoing monitoring. They are not "set it and forget it" systems.

## Inference After Adaptive Allocation

A common mistake is to run a bandit and then analyze the result like a fixed A/B test.

This is risky because assignment probabilities changed over time.

Suppose variant A looked strong early, so the bandit sent more traffic to A later. If later traffic differs from early traffic, the observed average for A may reflect both:

- The quality of variant A
- The types of users and time periods when A received more traffic

This does not mean bandits cannot be analyzed. It means the analysis must account for the adaptive assignment process.

Useful practices include:

- Log assignment probabilities for every decision.
- Log the context available at the time of assignment.
- Keep a small fixed-randomization holdout when clean measurement matters.
- Use inverse-propensity-weighted or doubly robust evaluation for policies.
- Avoid claiming a simple fixed-horizon p-value unless the analysis supports it.

The most important logging field is the probability that the chosen action was assigned:

$$
P(A_i=a \mid X_i, \text{history before }i)
$$

Without this probability, later policy evaluation becomes much harder.

## Bandits Versus Sequential Testing

Bandits and sequential tests both look at data while the experiment is running, but they solve different problems.

Sequential testing asks:

> Can we stop early while preserving valid inference?

Bandits ask:

> Can we allocate more traffic to better options while learning?

Sequential testing changes the stopping rule. Bandits change the assignment rule.

This distinction matters:

| Design | Main Goal | What Adapts? | Typical Output |
|---|---|---|---|
| Fixed A/B test | Clean measurement | Nothing | Treatment effect and confidence interval |
| Sequential test | Early stopping with valid inference | Stopping rule | Decision to stop or continue |
| Bandit | Reduce regret and optimize allocation | Traffic assignment | Learned policy or best arm |

If the goal is a clean launch estimate, a fixed or sequential A/B test is often better. If the goal is ongoing optimization among many options, a bandit may be better.

## When to Use Bandits

Bandits are useful when:

- There are multiple plausible variants.
- The reward is observed quickly.
- The cost of sending traffic to bad variants is meaningful.
- The goal is optimization, not only measurement.
- The treatment can be changed or personalized frequently.
- Guardrails can be monitored reliably.

Examples:

- Choosing among headlines
- Selecting notification copy
- Ranking promotional creatives
- Allocating traffic among recommendation modules
- Personalizing coupons
- Choosing ad creatives

Bandits are less appropriate when:

- The primary outcome is delayed.
- Guardrails are hard to measure.
- The treatment affects long-term ecosystem behavior.
- A clean causal estimate is required.
- The product experience must remain stable.
- The organization cannot monitor the algorithm continuously.

The practical decision is:

> Are we trying to measure an effect, or are we trying to optimize an allocation?

## Example: Notification Copy

Suppose a music app wants to choose among five notification messages encouraging users to listen to a new playlist.

The reward is notification open rate within one hour.

A fixed A/B test would allocate 20% of traffic to each message until the experiment ends.

A bandit could start with equal traffic and gradually send more users to better-performing messages.

This is a reasonable bandit use case because:

- There are multiple variants.
- The reward is observed quickly.
- The downside of weak variants is real but limited.
- The creative can be changed frequently.

Guardrails should still be monitored:

- Notification opt-outs
- App uninstalls
- Session quality after open
- Repeat engagement

If one message gets many opens but also increases opt-outs, the reward is too narrow.

## Example: Subscription Onboarding

Now suppose a subscription app wants to choose among three onboarding flows.

The short-term reward is trial start.

A bandit could quickly send more users to the flow that starts the most trials. But this may be dangerous if the true goal is paid subscription value.

The better flow is not necessarily the one with the highest trial start rate. It may be the one with the best combination of:

- Trial start
- Paid conversion
- Refund rate
- 30-day paid retention
- Net revenue per entrant

If these outcomes are delayed, a fixed A/B test or long-term holdout may be more appropriate than a bandit. Another option is to use a bandit only after validating that the short-term reward is aligned with long-term value.

## Practical Workflow

A practical workflow for bandit experiments is:

1. Decide whether the goal is measurement or optimization.
2. Define the action set.
3. Choose the reward metric.
4. Check whether the reward is fast enough and aligned with long-term value.
5. Define guardrails and stopping rules.
6. Choose the bandit algorithm.
7. Log assignment probabilities and contexts.
8. Monitor reward, guardrails, and traffic allocation over time.
9. Keep a fixed holdout if clean long-term measurement matters.
10. Evaluate the learned policy out of sample before broad automation.

## Common Mistakes

**Using a bandit when the goal is clean measurement**

Bandits optimize allocation. They are not automatically the best design for estimating a launch effect.

**Optimizing a narrow reward**

Clicks, opens, and short-term conversion can be poor proxies for long-term value.

**Forgetting guardrails**

Adaptive systems can exploit metric loopholes quickly.

**Not logging assignment probabilities**

Without assignment probabilities, offline evaluation and unbiased policy analysis become much harder.

**Stopping exploration too early**

Early noise can cause the algorithm to overcommit to a variant that is not actually best.

**Ignoring delayed rewards**

If the important outcome arrives weeks later, a fast bandit reward may optimize the wrong behavior.

**Treating the final traffic allocation as proof**

The variant with the most traffic at the end is not automatically proven best by fixed A/B test standards.

## Key Takeaways

Bandits adapt traffic allocation while learning.

The core tradeoff is exploration versus exploitation.

Regret measures the cost of assigning traffic to non-best options.

Epsilon-greedy, UCB, and Thompson sampling are common bandit algorithms.

Contextual bandits choose actions based on user, item, or context features.

Bandits are useful when rewards are fast, variants are many, and optimization matters more than clean measurement.

Bandits are risky when outcomes are delayed, guardrails are weak, or a clear causal estimate is required.

Adaptive allocation complicates inference, so assignment probabilities and context must be logged.

Bandits are not the same as sequential tests. Sequential tests adapt stopping; bandits adapt assignment.

The best bandit systems optimize rewards that are aligned with long-term product value and constrained by meaningful guardrails.
