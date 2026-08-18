# Designing a Clean Experiment

Chapter 1 explained how to turn a product idea into a decision-ready experiment question. Chapter 2 starts after that point.

Once the hypothesis is clear, the next question is design:

> What experiment would allow us to test this hypothesis credibly?

A good experiment is mostly designed before the first user enters it. Once the data arrives, the analysis may feel like the main event: p-values, confidence intervals, dashboards, and launch decisions. But many experiment failures are already built into the design. If the wrong users enter the test, if treatment and control are not clearly defined, if the randomization unit is unstable, or if exposure is ambiguous, no statistical method can fully rescue the result later.

The important idea in this chapter is that a hypothesis is not only a statistical statement. It is also a design anchor. It tells the team what treatment must be created, who should enter the experiment, when exposure happens, what unit should be randomized, what metrics should be measured, and what validity checks matter.

Suppose a food delivery app has the following hypothesis:

> Showing a delivery-time range, such as "25-35 minutes," instead of a point estimate, such as "30 minutes," will reduce post-order cancellations caused by delivery-time uncertainty, without reducing order conversion.

This hypothesis already implies most of the design:

- What treatment and control mean in the product
- Who is eligible to enter the experiment
- What counts as exposure
- What unit is randomized
- When randomization happens
- Which metrics and guardrails will be analyzed
- How much sample size and duration are needed
- What validity checks should be run before trusting the result

The rest of this chapter shows how to turn that hypothesis into a clean experiment design.

## Treatment and Control

The first design question is what exactly changes.

In the delivery-time example, the treatment is not simply "a better delivery estimate." That is too vague. The experiment needs a precise comparison:

- Control users see the current point estimate, such as "30 minutes"
- Treatment users see a delivery-time range, such as "25-35 minutes"

Treatment and control should differ in exactly the intended way. If treatment also changes the color, placement, wording, or backend estimation model, then the experiment no longer isolates the effect of showing a range instead of a point estimate.

The design question is:

> What exactly is the treatment?

A good experiment document should specify:

- What control users see
- What treatment users see
- Where the change appears
- Whether the change is consistent across surfaces
- Whether any backend logic changes
- What version or configuration is being tested

This level of detail prevents confusion later, especially when results are surprising.

## Eligibility: Who Enters the Experiment?

Eligibility defines who can enter the experiment population before treatment is assigned.

This sounds simple, but it is one of the most important design choices. Eligibility should usually be based on information that exists before treatment exposure: country, platform, app version, login status, account type, device type, or whether a feature is technically available to the user.

Suppose a food delivery app tests a new delivery-time display, but the feature is available only in cities where the logistics system can provide reliable time ranges. Users outside those cities should not enter the experiment. Including them would dilute the measured effect, because they could never receive the intended treatment.

A cleaner eligibility rule would be:

> Logged-in users in cities where delivery-time ranges are supported, using app versions that can display the new format.

Eligibility should be defined before randomization whenever possible. Defining it after treatment exposure can introduce bias.

Good eligibility rules answer:

- Who can actually receive the treatment?
- What pre-treatment information determines eligibility?
- Are there users or contexts that should be excluded?
- At what moment does an eligible user enter the randomization system?

## Exposure and Triggered Users

Eligibility is not the same as exposure.

A user may be eligible for the experiment but never actually reach the product moment where the treatment could matter. For example:

- A user eligible for a checkout experiment never reaches checkout
- A user eligible for a recommendation experiment does not open the feed
- A user eligible for a job application experiment never opens an application page
- A user eligible for a pricing experiment visits only free pages

This creates two related populations:

**Assigned users**

Users included in the experiment assignment.

**Triggered users**

Users who actually had a chance to experience the treatment.

Analyzing all assigned users gives an intent-to-treat estimate. It answers:

> What is the effect of assigning users to this product experience?

Analyzing triggered users can be more sensitive, because it focuses on users who had a real opportunity to be affected. But it must be done carefully. If triggering is affected by the treatment, analyzing only triggered users can introduce bias.

A good design defines the trigger event in advance.

For example:

> A user enters the checkout experiment when they land on the checkout page.

This is cleaner than assigning all site visitors and later filtering to only those who reached checkout, especially if the treatment could affect whether users reach checkout.

The distinction from eligibility is timing. Eligibility defines who can enter the experiment before treatment exposure. Triggering defines when an assigned user reaches the product moment where treatment could matter.

Before choosing the randomization unit and timing, the team should already know the primary metric. This does not require a full metric plan yet, but it does require knowing where the main outcome sits in the product funnel. A cancellation metric, a conversion metric, and a revenue-per-user metric may imply different assignment timing.

## Randomization Unit

The randomization unit is the level at which treatment assignment happens.

The hypothesis helps choose the unit because it defines both the treatment and the outcome of interest. In the delivery-time example, the treatment changes the user's experience across restaurant pages, checkout, and possibly order tracking. If the same user saw a point estimate in one session and a range in another, the experience would be confusing and the comparison would be contaminated. User-level randomization is therefore a natural choice.

Common choices include:

- User
- Session
- Device
- Page view
- Item or listing
- Creator or seller
- Store or restaurant
- City or region
- Time block

In general, the right unit depends on how the treatment works, whose outcome the primary metric measures, whose behavior can be affected, and where the product experience needs to remain stable.

User-level randomization is common because it gives a consistent experience. If a user sees the new checkout page today, they should usually see it again tomorrow. This avoids confusion and contamination.

Session-level randomization may be useful when the experience is short-lived and users rarely return, but it can create inconsistency for returning users.

Item-level or creator-level randomization may be needed when the treatment is attached to content rather than users. For example, if a marketplace tests a new seller badge, the right unit depends on the business question. If the goal is to measure buyer-side outcomes such as GMV per user or purchase conversion, user-level randomization may be reasonable: some buyers see badges and some do not. If the goal is to measure seller-side outcomes such as seller revenue, exposure, or conversion, seller-level randomization is cleaner. Otherwise the same seller may be partially treated, making seller-level outcomes hard to interpret.

Geo-level or time-level randomization is often used when individual users are not independent, such as in ride-sharing, food delivery, ad auctions, and marketplaces. Chapter 9 discusses these designs in more detail.

The rule of thumb is:

> Choose a randomization unit that matches the treatment, the primary metric, and the level where contamination or interference may occur.

### When Randomization Happens

The randomization unit defines who receives a stable assignment. The randomization timing defines when that assignment is created.

This timing matters because it affects the experiment population.

In the delivery-time example, the team could randomize users at several moments:

- When an eligible user opens the app
- When an eligible user views a restaurant page
- When an eligible user enters checkout

Randomizing too early can dilute the effect. If all eligible app users are randomized, many assigned users may never see a delivery estimate. The intent-to-treat estimate is still valid for the population of eligible app users, but it may be much smaller than the effect among users who actually reach the delivery estimate.

Randomizing too late can create bias. If the delivery-time range appears on the restaurant page, but users are randomized only after reaching checkout, the treatment may have already affected who reaches checkout. In that case, the checkout population is partly shaped by the treatment itself.

The practical rule is:

> Randomization should happen before treatment exposure, but as close as practical to the first moment when treatment could affect behavior.

The right timing also depends on the primary metric and where that metric sits in the funnel.

Consider a simplified food ordering funnel:

```text
Open app -> View restaurant page -> Enter checkout -> Place order -> Cancel or complete order
```

If the primary metric is post-order cancellation rate, randomizing users when they place an order can make sense. The experiment would answer:

> Among users who placed an order, does showing the delivery-time range reduce cancellation?

But if the primary metric is orders per user, restaurant page to checkout conversion, or GMV per user, randomizing only after order placement would be too late. The treatment may already have affected whether the user continued through the funnel and placed an order. In that case, randomization should happen earlier, before the treatment can affect order creation.

So a more complete rule is:

> Randomization should happen before the earliest point in the funnel where the treatment can affect the primary metric.

For triggered experiments, this means the trigger event should be defined before the experiment starts. In the delivery-time example, a reasonable trigger could be:

> The first time an eligible user views a restaurant page or checkout page where a delivery estimate is shown.

That timing keeps assignment before exposure while avoiding a large population of assigned users who never had a chance to be affected.

### Hash-Based User Randomization

Once the design chooses user-level assignment, the platform still needs a stable way to implement it.

When the randomization unit is the user, many experimentation systems assign users with a deterministic hash.

The basic idea is simple:

1. Take a stable user identifier, such as `user_id`.
2. Combine it with an experiment identifier, such as `checkout_redesign_v1`.
3. Hash the combined string into a number.
4. Map that number into a bucket.
5. Assign buckets to control or treatment.

For example:

```text
hash_input = user_id + ":" + experiment_id
bucket = hash(hash_input) % 10000

if bucket < 5000:
    assignment = "control"
else:
    assignment = "treatment"
```

This creates a stable 50/50 split. The same user will receive the same assignment every time the experiment is checked, because the hash input does not change.

The experiment identifier is important. If the platform hashes only `user_id`, then the same users may repeatedly fall into treatment across many experiments. By hashing `user_id` together with `experiment_id`, each experiment gets a fresh random split while keeping assignment stable inside that experiment.

A more general version is:

$$
\text{bucket}_i
=
h(\texttt{user\_id}_i, \texttt{experiment\_id}) \bmod B
$$

where $B$ is the number of available buckets. If $B = 10{,}000$, then each bucket represents roughly 0.01% of eligible users.

Hash-based assignment has several advantages:

- It is deterministic.
- It does not require storing every assignment in a database.
- It gives users a consistent experience across visits.
- It supports gradual rollout by changing which buckets receive treatment.
- It makes the expected traffic split easy to audit.

The implementation details matter because randomization must be stable and auditable. If the identifier changes across sessions, the same user may receive different variants. If assignment happens after exposure, the experiment population may already be selected by post-treatment behavior. If the observed traffic split differs from the planned split, the team should investigate before interpreting results.

Hash-based randomization is not the only possible implementation. The principle is more important than the exact code:

> Use a stable identifier and a deterministic randomization rule so assignment is consistent, auditable, and independent of user behavior.

## Metrics

Chapter 1 introduced primary, secondary, and guardrail metrics. Chapter 3 will discuss metric design in more detail. Here, the design task is to complete the metric plan before the experiment starts.

The primary metric should match the hypothesis and the launch decision. In the delivery-time example, the hypothesis is not merely that users will notice the range. The business question is whether clearer expectations reduce cancellations. A natural primary metric is therefore post-order cancellation rate caused by delivery-time concerns.

Delivery-time-related complaint rate can still be useful as a secondary metric, because it helps explain whether users are less frustrated even when they do not cancel.

The phrase "without reducing order conversion" also matters. It implies guardrails. If the range makes delivery uncertainty more visible and fewer users place orders, the treatment may reduce cancellations only because fewer risky orders happen in the first place.

Secondary metrics explain the mechanism. Guardrails protect against unintended harm.

A useful metric set has balance:

- One primary metric for the main decision
- A small number of secondary metrics for interpretation
- A small number of guardrails for safety and product health

Too many metrics create noise and make it easier to tell a convenient story after the fact. Too few metrics make it hard to understand what happened.

## Sample Size, MDE, and Duration

Before launching an experiment, the team should estimate how much data is needed.

This is called power analysis. Its goal is to answer a practical question:

> If the treatment has an effect large enough to matter, how many users do we need in order to detect it reliably?

The hypothesis determines which metric this calculation should use. In the delivery-time example, sample size should be planned around the primary outcome, such as post-order cancellation rate caused by delivery-time concerns, not around a convenient high-volume metric such as page clicks.

This matters because cancellation may be a relatively rare event. If the baseline cancellation rate is low, detecting a meaningful reduction may require a large number of orders or a longer experiment.

The key inputs are:

- Baseline metric value
- Minimum detectable effect, or MDE
- Significance level, usually called alpha
- Power, which equals 1 - beta
- Metric variance
- Daily eligible traffic

### Alpha, Beta, and Power

Alpha is the false positive rate. If alpha is 0.05, the experiment allows a 5% chance of incorrectly detecting an effect when the treatment actually has no effect.

Beta is the false negative rate. If beta is 0.20, the experiment has a 20% chance of missing the effect size it was designed to detect.

Power is:

$$
\text{Power} = 1 - \beta
$$

So if beta is 0.20, power is 80%.

A common setup is:

$$
\alpha = 0.05,\quad \beta = 0.20,\quad \text{power} = 80\%
$$

### Minimum Detectable Effect

The minimum detectable effect is the smallest effect the experiment is designed to detect with the chosen power.

If the baseline conversion rate is 10%, and the team wants to detect a 5% relative lift, then:

$$
\text{MDE} = 10\% \times 5\% = 0.5\text{ percentage points}
$$

So the experiment is designed to detect a change from 10.0% to 10.5%.

This distinction matters:

- Relative lift: 5%
- Absolute lift: 0.5 percentage points

Sample size calculations usually use the absolute lift.

### Sample Size for a Difference in Means

For a continuous metric such as revenue per user, session duration, or GMV per user, a simple approximation for equal-sized treatment and control groups is:

$$
n \approx \frac{2(z_{1-\alpha/2} + z_{1-\beta})^2 \sigma^2}{\delta^2}
$$

Where:

- $n$ is the required sample size per group
- $\sigma^2$ is the metric variance
- $\delta$ is the minimum detectable effect in absolute units
- $z_{1-\alpha/2}$ comes from the significance level
- $z_{1-\beta}$ comes from the desired power

For a two-sided test with alpha = 0.05 and power = 80%:

$$
z_{1-\alpha/2} \approx 1.96,\quad z_{1-\beta} \approx 0.84
$$

So:

$$
(z_{1-\alpha/2} + z_{1-\beta})^2 \approx (1.96 + 0.84)^2 = 7.84
$$

The formula becomes approximately:

$$
n \approx \frac{15.68\sigma^2}{\delta^2}
$$

### Sample Size for a Conversion Rate

For a binary metric such as conversion rate, signup rate, or click-through rate, the metric variance is tied to the baseline rate.

If the control conversion rate is $p_C$, the treatment conversion rate is $p_T$, and the absolute MDE is:

$$
\delta = p_T - p_C
$$

Then an approximate sample size per group is:

$$
n \approx \frac{(z_{1-\alpha/2} + z_{1-\beta})^2[p_C(1-p_C) + p_T(1-p_T)]}{\delta^2}
$$

When the expected effect is small, $p_T$ is close to $p_C$, so a simpler approximation is:

$$
n \approx \frac{2(z_{1-\alpha/2} + z_{1-\beta})^2p(1-p)}{\delta^2}
$$

where $p$ is the baseline conversion rate.

For example, suppose:

- Baseline conversion rate: 10%
- Desired relative lift: 5%
- Absolute MDE: 0.5 percentage points
- Alpha: 0.05
- Power: 80%

Then:

$$
p = 0.10,\quad \delta = 0.005
$$

Using the approximation:

$$
n \approx \frac{2(1.96 + 0.84)^2(0.10)(0.90)}{0.005^2}
$$

$$
n \approx 56{,}448
$$

So the experiment needs about 56,000 users per group, or about 112,000 users total.

### From Sample Size to Duration

Once the sample size is estimated, duration depends on eligible traffic.

If the experiment needs 112,000 users total and the product has 20,000 eligible users per day, then the minimum duration is:

$$
\text{Duration} = \frac{112{,}000}{20{,}000} = 5.6\text{ days}
$$

But experiments should usually cover full business cycles. If user behavior differs between weekdays and weekends, the test should run for at least one full week, even if the required sample size is reached earlier.

For metrics with delayed outcomes, duration also needs to include observation time. A trial conversion experiment may need one week to collect users and another week or month to observe whether they convert.

Sample size planning is not just a statistical exercise. It is also a business decision. A good experiment plan should make the tradeoff explicit:

- What effect size would be meaningful for the business?
- How long would it take to detect that effect?
- Is the required duration realistic?
- Are there ways to reduce variance, such as CUPED or stratification?
- Are delayed metrics or seasonality likely to require a longer test?

The goal is not to calculate a perfect number. The goal is to avoid running an experiment that is too small to answer the question it was designed to answer.

## Validity Checks

Before interpreting experiment results, basic validity checks should pass. These checks do not prove that the experiment is perfect, but they can reveal problems that make the result hard to trust.

Important checks include:

**Sample ratio mismatch**

Did traffic split as expected? If the experiment was designed as 50/50, but the result is 55/45, something may be wrong with assignment, logging, or eligibility.

**Logging correctness**

Were exposure and outcome events recorded correctly for both groups?

**Baseline balance**

Do treatment and control look similar on pre-treatment variables?

**Contamination**

Could users see both treatment and control experiences?

**Interference**

Could one user's treatment affect another user's outcome?

**Novelty effects**

Are users reacting temporarily because the experience is new?

**External shocks**

Did marketing campaigns, outages, holidays, or operational issues affect one group more than the other?

Some of these issues, such as interference and novelty effects, are discussed in later chapters. Here, the main point is that they should be considered before the team treats the result as decision-ready.

## Putting the Design Together

The delivery-time hypothesis can now be written as a compact experiment design:

Hypothesis:
- Showing a delivery-time range instead of a point estimate will reduce post-order cancellations caused by delivery-time uncertainty, without reducing order conversion.

Treatment and control:
- Control users see a point estimate, such as "30 minutes"
- Treatment users see a range, such as "25-35 minutes"

Eligible users:
- Logged-in users in cities where delivery-time ranges are supported, using app versions that can display the new format

Exposure:
- The treatment can affect users when they view a restaurant page or checkout page where a delivery estimate is shown

Triggered users:
- Eligible users who reach the first page where a delivery estimate is shown

Randomization unit:
- User level, because the treatment changes the user's experience and the primary metric is based on user/order behavior. User-level assignment also keeps the delivery-time format consistent across sessions and surfaces.

Randomization timing:
- Assign users at the first qualifying restaurant page or checkout page view, before the delivery estimate is shown
- This keeps assignment close to exposure while still happening before the treatment can affect behavior

Primary metric:
- Post-order cancellation rate caused by delivery-time concerns

Secondary metrics:
- Delivery-time-related complaint rate
- Time from order placement to cancellation
- Clicks on delivery tracking or delivery details
- Difference between estimated and actual delivery time
- Delivery-time satisfaction rating

Guardrail metrics:
- Restaurant page to checkout conversion
- Checkout completion rate
- Order conversion rate
- Refund or credit request rate
- Support contact rate
- Repeat order rate

Duration:
- Based on baseline delivery-time-related cancellation rate, MDE, alpha, power, and daily eligible traffic
- Run for at least full weekly cycles to cover weekday and weekend ordering patterns

Validity risks:
- Users may see a point estimate on one surface and a range on another
- Weather or courier supply shocks may affect delivery experience
- Logging may capture estimate exposure inconsistently
- The range may affect order conversion before it affects cancellations
- Users may initially react differently because the format is new

This design is not complicated, but it is precise. It starts from the hypothesis, then defines what changes, who enters, when treatment can matter, how assignment happens, what success means, what harm to watch for, and what could make the result untrustworthy.

## Key Takeaways

A clean experiment starts from a clear hypothesis.

The hypothesis should imply the treatment, eligible population, exposure moment, randomization unit, and metric plan.

Treatment and control should differ in the intended way.

Eligibility should be based on pre-treatment information, while exposure defines when treatment can actually matter.

The randomization unit and timing should match the treatment, the primary metric, and the main contamination risks.

Sample size and duration should be planned around MDE, power, traffic, and time patterns.

Validity checks are essential before trusting the result.
