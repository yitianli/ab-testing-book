# Concurrent Experiments

In a real product organization, experiments rarely run one at a time.

A large app may have dozens or hundreds of experiments running at the same time:

- Search ranking tests
- Checkout flow tests
- Pricing tests
- Notification tests
- Feed ranking tests
- Onboarding tests
- Ads experiments
- Infrastructure changes

This creates a natural question:

> Can a user be included in more than one experiment at the same time?

The answer is usually yes, but not always.

Concurrent experiments are not automatically invalid. In many cases, overlapping experiments are exactly how modern experimentation platforms scale. But overlap becomes risky when experiments affect the same user experience, the same metric, the same marketplace system, or the same mechanism.

This chapter explains how concurrent experiments work, when they are safe, when they are dangerous, and how companies manage overlapping experiments using layers, mutual exclusion, factorial designs, and interaction checks.

## The Basic Problem

Suppose a company runs two experiments at the same time.

Experiment A tests a new checkout button.

Experiment B tests a new free-shipping banner.

A user may be assigned to:

|  | Experiment B Control | Experiment B Treatment |
|---|---:|---:|
| Experiment A Control | A0B0 | A0B1 |
| Experiment A Treatment | A1B0 | A1B1 |

If the experiments are independent, this is usually fine. The effect of the checkout button can be estimated by comparing users in A treatment versus A control, averaging over both B groups.

In notation:

$$
\text{Effect of A}
=
E[Y \mid A=1] - E[Y \mid A=0]
$$

This comparison includes users from both B treatment and B control. If B is independently randomized, the B assignment should be balanced across A treatment and A control.

The same is true for B:

$$
\text{Effect of B}
=
E[Y \mid B=1] - E[Y \mid B=0]
$$

This is the reason concurrent experimentation can work. Independent randomization makes other experiments look like balanced noise.

But this logic relies on an important assumption:

> The effect of one experiment does not meaningfully depend on the assignment of another experiment.

When that assumption fails, the experiments interact.

## Main Effects and Interaction Effects

With two experiments, the outcome can be written as:

$$
Y_i
=
\alpha
+ \tau_A A_i
+ \tau_B B_i
+ \tau_{AB} A_iB_i
+ \epsilon_i
$$

Here:

- $\tau_A$ is the main effect of experiment A.
- $\tau_B$ is the main effect of experiment B.
- $\tau_{AB}$ is the interaction effect.

The interaction term asks:

> Is the effect of A different depending on whether B is also turned on?

For example, suppose:

- A new checkout button increases conversion by 2%.
- A free-shipping banner increases conversion by 3%.
- But when both are shown together, the page becomes visually crowded and conversion increases by only 1%.

The effects are not additive. The two changes interfere with each other at the user-experience level.

To describe this, define four average outcomes:

|  | B Control | B Treatment |
|---|---:|---:|
| A Control | $Y_{00}$ | $Y_{01}$ |
| A Treatment | $Y_{10}$ | $Y_{11}$ |

Here, $Y_{ab}$ means the average outcome for users with assignment $A=a$ and $B=b$. For example, $Y_{10}$ is the average outcome for users in A treatment and B control.

A simple way to calculate the interaction is:

$$
\text{Interaction}
=
(Y_{11} - Y_{01}) - (Y_{10} - Y_{00})
$$

Equivalently:

$$
\text{Interaction}
=
Y_{11} - Y_{10} - Y_{01} + Y_{00}
$$

This compares the effect of A when B is on versus the effect of A when B is off.

If the interaction is close to zero, the experiments are approximately additive. If the interaction is large, the combined product experience is different from what the separate experiment results suggest.

## When Overlap Is Usually Fine

Concurrent experiments are usually safer when they affect different parts of the product and different mechanisms.

Examples:

- An onboarding copy test and a backend latency improvement
- A profile page layout test and a search ranking test
- A notification timing test and a checkout button color test
- A help-center UI test and a feed recommendation test

Overlap is more acceptable when:

- The experiments touch different surfaces.
- The user experiences do not conflict.
- The primary metrics are not driven by the same narrow funnel step.
- The treatments do not compete for the same scarce resource.
- The engineering changes do not modify the same underlying system.

In this case, other experiments mainly add variance. They may make the metric noisier, but they do not necessarily bias the treatment effect.

## When Overlap Is Risky

Concurrent experiments become risky when two treatments affect the same mechanism.

### Same Product Surface

Two experiments may both change the same page or flow.

Examples:

- Two checkout page experiments
- Two onboarding flow experiments
- Two search results page experiments
- Two feed layout experiments

The risk is user-experience conflict. One treatment may hide, weaken, or amplify the other.

For example, a checkout simplification may improve conversion, while a new upsell module may reduce conversion. If both run together, the final page may not represent either experiment cleanly.

### Same Metric Path

Experiments can interact even if they touch different surfaces, as long as they affect the same user journey.

For example:

- A product detail page experiment increases add-to-cart rate.
- A checkout experiment increases purchase completion.

Both affect purchase conversion. If the product detail page sends a different mix of users into checkout, the checkout experiment may see a changed population.

This is not always a problem, but it should be understood. The measured effect of the checkout experiment may depend on what upstream experiments are doing.

### Same Marketplace Resource

In marketplaces, experiments may compete through shared supply, demand, or attention.

Examples:

- A buyer promotion and a seller ranking experiment both affect order allocation.
- A courier incentive and a delivery-fee test both affect supply-demand balance.
- Two ads experiments both affect auction prices.
- Two feed ranking experiments both affect creator exposure.

This overlaps with the interference problems from Chapter 8. The difference is that Chapter 8 focused on interference between treatment and control units inside one experiment. Concurrent experiments add another layer: different experiments can interfere with each other.

### Same Machine Learning System

Many modern products use shared ranking, recommendation, or pricing models.

Two experiments may look separate in the UI but still affect the same model inputs or feedback loops.

Examples:

- One experiment changes what users click.
- Another experiment trains or updates a ranking model using those clicks.
- One experiment changes creator exposure.
- Another experiment measures creator content supply.

The risk is that one experiment changes the data-generating process for another experiment.

## Orthogonal Randomization

The simplest way to run concurrent experiments is independent, or orthogonal, randomization.

Suppose experiment A assigns 50% of users to treatment, and experiment B also assigns 50% of users to treatment. If the assignments are independent, then approximately:

| Group | Share of Users |
|---|---:|
| A0B0 | 25% |
| A0B1 | 25% |
| A1B0 | 25% |
| A1B1 | 25% |

This allows the company to estimate both main effects at the same time.

For experiment A:

$$
E[Y \mid A=1] - E[Y \mid A=0]
$$

For experiment B:

$$
E[Y \mid B=1] - E[Y \mid B=0]
$$

Because A and B are independently randomized, B is balanced across A's treatment and control groups, and A is balanced across B's treatment and control groups.

This is the core idea behind large-scale experimentation platforms. They allow many experiments to run at once as long as the assignment system keeps experiments statistically independent and operationally compatible.

## Experiment Layers

Many companies use experiment layers to manage overlap.

A layer is a part of the product or system where experiments are allowed to overlap with experiments in other layers, but not necessarily with experiments inside the same layer.

Example layers:

- Search ranking layer
- Checkout layer
- Notifications layer
- Pricing layer
- Ads auction layer
- Feed ranking layer
- Infrastructure layer

The idea is:

> Experiments in different layers can often overlap. Experiments inside the same layer may need mutual exclusion.

For example, a checkout button experiment and a checkout form experiment may be placed in the same checkout layer. A user should usually not receive both treatments at the same time if the team wants clean interpretation.

But a checkout experiment and a notification timing experiment may be placed in different layers. A user can be part of both if the two changes are unlikely to interact meaningfully.

Layering is a practical compromise. It does not prove that experiments cannot interact, but it creates a controlled structure for managing risk.

### Hash-Based Assignment with Layers

Experiment layers are often implemented with hash buckets.

A bucket is a numbered slot created by hashing a stable identifier. For example, if a system has 10,000 buckets, each eligible user is deterministically mapped into one bucket from 0 to 9,999.

Chapter 2 introduced this idea for a single experiment. With layers, the same idea is extended in two steps:

1. Use a stable identifier, such as `user_id`.
2. Use a layer-specific hash to decide where the user falls inside that layer.

For example:

```text
layer_bucket = hash(user_id + ":" + layer_id) % 10000
```

Here, `layer_id` could be `checkout_layer`, `search_layer`, or `notification_layer`.

Each layer has its own bucket space. A user can be in bucket 1,250 in the checkout layer and bucket 7,820 in the notification layer. These are separate assignments because the hash includes different layer identifiers.

This allows experiments in different layers to overlap.

For example:

```text
checkout_bucket = hash(user_id + ":checkout_layer") % 10000
notification_bucket = hash(user_id + ":notification_layer") % 10000
```

A user can be assigned to a checkout experiment and a notification experiment at the same time, because the two layers are independent.

Inside one layer, buckets can be reserved for mutually exclusive experiments.

For example, suppose the checkout layer has 10,000 buckets:

| Bucket Range | Assignment |
|---|---|
| 0-1999 | Checkout experiment A |
| 2000-3999 | Checkout experiment B |
| 4000-5999 | Checkout experiment C |
| 6000-9999 | No checkout-layer experiment |

A user whose checkout-layer bucket is 1,250 can enter experiment A. A user whose bucket is 3,100 can enter experiment B. No user can enter both A and B because the bucket ranges do not overlap.

Then each experiment can randomize treatment versus control within its reserved range.

For example:

```text
if 0 <= checkout_bucket < 2000:
    experiment = "checkout_experiment_a"
    variant_bucket = hash(user_id + ":checkout_experiment_a") % 10000

    if variant_bucket < 5000:
        variant = "control"
    else:
        variant = "treatment"
```

This creates two levels of assignment:

- Layer assignment decides which experiment, if any, the user can enter.
- Experiment assignment decides which variant the user receives inside that experiment.

The layer hash and experiment hash should use different identifiers. The layer hash controls traffic allocation across experiments. The experiment hash controls treatment-control balance within a specific experiment.

This design has several benefits:

- Experiments in different layers can overlap.
- Experiments inside a mutually exclusive layer do not overlap.
- Traffic allocation can be audited by checking bucket ranges.
- New experiments can be added by reserving unused buckets.
- Users keep stable assignments as long as the hash identifiers and bucket rules do not change.

There are also important cautions.

First, changing bucket ranges during an experiment can change the eligible population. If experiment A starts with buckets 0-999 and later expands to 0-1999, the later analysis includes users who were not eligible at the start. That can be fine for staged rollout, but the analysis should clearly define the population and time window.

Second, layers only manage planned overlap. They do not guarantee that two experiments are causally independent. A checkout experiment and a notification experiment may live in different layers, but they can still interact if notifications change the type of users who reach checkout.

Third, assignment should be checked after launch. If a layer reserves 20% of users for an experiment but the observed traffic is very different, the team may have an eligibility, logging, or assignment problem.

## Mutual Exclusion

Mutual exclusion means a user, market, seller, or other unit can enter only one experiment from a specified group.

It is useful when:

- Experiments modify the same UI surface.
- Treatments cannot be shown together.
- Experiments change the same ranking or pricing logic.
- The interaction would be hard to interpret.
- The team needs a clean estimate for launch decisions.

For example, suppose three teams are testing different checkout page layouts. If the same user can receive multiple layout changes, the experiment result may be hard to understand. A mutually exclusive layer assigns each eligible user to at most one checkout experiment.

The benefit is cleaner interpretation.

The cost is traffic.

If experiments are mutually exclusive, they must share the same pool of users. This can slow down learning because each experiment receives less traffic.

## Factorial Designs

Sometimes interaction is not a nuisance. It is the thing the team wants to learn.

A factorial design deliberately randomizes multiple factors together.

For example, an e-commerce team may want to test:

- Checkout layout: old versus new
- Free-shipping banner: hidden versus visible

This creates four cells:

| Cell | Checkout Layout | Banner |
|---|---|---|
| A0B0 | Old | Hidden |
| A0B1 | Old | Visible |
| A1B0 | New | Hidden |
| A1B1 | New | Visible |

The team can estimate:

- The main effect of the checkout layout
- The main effect of the banner
- The interaction between layout and banner

Factorial designs are useful when the team believes treatments may interact and wants to measure the combined product experience.

The cost is sample size. Estimating interactions requires enough observations in each cell. With many factors, the number of cells grows quickly.

For two binary factors, there are $2^2 = 4$ cells.

For five binary factors, there are $2^5 = 32$ cells.

This is why factorial designs should be used deliberately. They are powerful when the interaction matters, but expensive when the goal is only to estimate separate main effects.

## How to Check Interaction

When two overlapping experiments may interact, the analysis should compare the four cells.

For two binary experiments A and B:

|  | B Control | B Treatment |
|---|---:|---:|
| A Control | $Y_{00}$ | $Y_{01}$ |
| A Treatment | $Y_{10}$ | $Y_{11}$ |

The effect of A when B is off is:

$$
Y_{10} - Y_{00}
$$

The effect of A when B is on is:

$$
Y_{11} - Y_{01}
$$

The interaction is:

$$
(Y_{11} - Y_{01}) - (Y_{10} - Y_{00})
$$

This is the same as:

$$
Y_{11} - Y_{10} - Y_{01} + Y_{00}
$$

A regression version is:

$$
Y_i
=
\alpha
+ \tau_A A_i
+ \tau_B B_i
+ \tau_{AB} A_iB_i
+ \epsilon_i
$$

The coefficient $\tau_{AB}$ estimates the interaction.

But interaction tests often have low power. A non-significant interaction does not prove there is no interaction. It may simply mean the experiment does not have enough data to detect it.

In practice, teams should combine:

- Product judgment
- Surface ownership
- Metric overlap
- Statistical interaction checks
- Guardrail monitoring

## Concurrent Experiments and Metrics

The risk of concurrent experiments depends heavily on metrics.

If two experiments both optimize the same primary metric, they may still be fine if they operate through different mechanisms. But if both affect the same metric through the same funnel step, overlap needs more care.

For example:

- An onboarding experiment increases signup completion.
- A pricing page experiment increases trial start.

Both may affect paid conversion, but they operate at different points in the funnel.

By contrast:

- A pricing page discount experiment increases trial start.
- A pricing page copy experiment also increases trial start.

These two experiments share the same surface and the same metric path. Overlap is riskier.

Guardrails are also important. Two experiments may each look safe individually but jointly violate a guardrail.

For example:

- A notification experiment increases daily active users but slightly increases opt-outs.
- A promotion experiment increases purchases but slightly increases unsubscribes.

Each experiment may pass guardrails alone. But together, they may create too much user fatigue.

This is why experiment review should consider the experiment portfolio, not only one test at a time.

## Launches During Experiments

Concurrent experimentation becomes harder when one experiment launches while another is still running.

Suppose experiment A is running for four weeks. In week two, experiment B launches to everyone.

Now the environment changed halfway through experiment A. If B affects A's metrics, A's before-and-after periods are not comparable.

There are several possible responses:

- Pause or restart A if B affects the same surface or mechanism.
- Add a time indicator for the launch of B.
- Analyze A separately before and after B launched.
- Exclude the transition period if logging or user experience changed sharply.
- Avoid launching major related changes during critical experiment windows.

The best answer depends on how related the two changes are.

If B is unrelated infrastructure work with no metric impact, A may remain valid. If B changes the same funnel or marketplace system, A may need to be redesigned or interpreted with caution.

## Example: Checkout and Free Shipping

Suppose an e-commerce company runs two experiments:

Experiment A:

- New checkout page layout
- Primary metric: purchase conversion rate

Experiment B:

- Free-shipping banner on the product detail page
- Primary metric: purchase conversion rate

These experiments touch different pages, but they share the same purchase funnel.

If run concurrently with independent randomization, the company should check:

- Does A's effect differ when B is on versus off?
- Does B change the type of users or orders reaching checkout?
- Does the combined experience reduce average order value?
- Do guardrails such as refund rate, cancellation rate, or customer support contacts worsen?

If interaction is small, both experiments can be interpreted mostly as independent main effects.

If interaction is large, the company should make a combined decision. It may not be valid to launch A and B separately based only on their individual average effects.

## Example: Ranking Experiments in a Shared Feed

Suppose a content platform runs two feed ranking experiments:

Experiment A boosts posts from new creators.

Experiment B boosts posts predicted to increase watch time.

Both experiments change the same feed ranking system and allocate the same scarce attention. If they overlap naively, they may interact strongly:

- New creators may receive more or less exposure depending on the watch-time model.
- Watch time may increase while creator diversity decreases.
- Control creators may lose impressions because two experiments are both reallocating feed space.
- The logging system may make it hard to know which ranking change caused which outcome.

This is a case where mutual exclusion or a deliberately designed factorial experiment may be better than uncontrolled overlap.

If the company mainly wants separate clean estimates, use mutual exclusion in the feed-ranking layer.

If the company wants to understand how the two ranking changes work together, use a factorial design and make the interaction an explicit estimand.

## Practical Workflow

A practical workflow for concurrent experiments is:

1. List other experiments that may overlap with the new experiment.
2. Identify whether they touch the same surface, metric path, model, or shared resource.
3. Decide whether overlap is safe, risky, or intentionally useful.
4. If overlap is safe, use independent randomization.
5. If overlap is risky and separate estimates are needed, use mutual exclusion.
6. If interaction is important, use a factorial design.
7. Preserve the assignment structure in the analysis.
8. Check interaction effects for high-risk overlaps.
9. Monitor shared guardrails at the portfolio level.
10. Document launches or major product changes that occur during the experiment.

## Common Mistakes

**Assuming all overlap is bad**

Concurrent experiments are often valid when they are independently randomized and affect different mechanisms. Avoiding all overlap can make experimentation unnecessarily slow.

**Assuming all overlap is safe**

Independent randomization balances assignments statistically, but it does not prevent product or marketplace interactions.

**Ignoring the combined user experience**

Two treatments may each look reasonable alone but confusing, crowded, or harmful together.

**Testing interactions without enough power**

Interaction effects often require much larger samples than main effects. A non-significant interaction is not strong evidence that interaction is impossible.

**Forgetting launches during experiments**

A full rollout of another feature can change the environment during an experiment and make the result harder to interpret.

**Using mutual exclusion too broadly**

Mutual exclusion improves interpretability but reduces traffic. It should be reserved for experiments likely to interact or conflict.

## Key Takeaways

Concurrent experiments are normal in mature experimentation systems.

Overlap is usually acceptable when experiments are independently randomized and affect different mechanisms.

The main risk is interaction: the effect of one experiment may depend on another experiment's assignment.

Experiment layers help manage overlap by separating product areas or systems.

Mutual exclusion gives cleaner interpretation when experiments affect the same surface, model, or resource.

Factorial designs are useful when the interaction itself is important to measure.

Interaction tests can be underpowered, so product judgment still matters.

Guardrails should sometimes be monitored across the experiment portfolio, not only within each individual experiment.

Launches during an experiment can change the environment and should be documented or handled in the analysis.
