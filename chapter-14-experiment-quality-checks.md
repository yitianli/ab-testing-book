# Experiment Quality Checks

An experiment can have a clean hypothesis, a good randomization design, and enough sample size, but still produce a misleading result if the data pipeline is broken.

This chapter is about the unglamorous but essential part of experimentation: quality checks.

Before asking:

> Did treatment improve the metric?

the analyst should first ask:

> Can I trust the experiment data?

This order matters. If the assignment is wrong, exposure is logged incorrectly, users are missing from one group, or a metric changed definition during the test, then a statistically significant result may only be a measurement artifact.

Experiment quality checks are not a formality. They are the difference between measuring a product change and measuring a data problem.

## Why Quality Checks Matter

A/B testing depends on a simple promise:

> Treatment and control are comparable except for the treatment.

Randomization is supposed to make that promise plausible. But production systems can break it in many ways:

- Some users are assigned but never eligible to see the feature.
- Some treatment users are not logged correctly.
- Some control users accidentally receive treatment.
- Assignment probabilities are not what the experiment platform expected.
- A tracking bug affects only one variant.
- A metric denominator is missing events from one client version.
- Bots or internal users enter one group more often than the other.
- Revenue or GMV has a few extreme outliers.

When these problems happen, the observed treatment-control difference may not represent a causal effect.

The danger is especially high because many quality problems look like real product effects. A logging bug can look like a conversion lift. A missing denominator can look like better engagement. A bot imbalance can look like worse retention.

That is why experiment analysis should usually follow this sequence:

1. Check assignment and sample size.
2. Check eligibility and exposure.
3. Check logging and metric construction.
4. Check baseline balance and pre-treatment behavior.
5. Check outliers, missing data, bots, and unusual segments.
6. Only then interpret primary and secondary metrics.

The goal is not to find a perfect dataset. Production data is rarely perfect. The goal is to understand whether remaining imperfections are small enough, symmetric enough, and explainable enough that the result can still support a decision.

## Sample Ratio Mismatch

Sample ratio mismatch, often called SRM, is one of the most important experiment health checks.

SRM means the observed number of users in each group does not match the intended allocation.

For example, suppose an experiment is designed as a 50/50 split. After one week, the analyst expects about half of users in treatment and half in control. Instead, the data shows:

| Group | Expected Share | Observed Users |
|---|---:|---:|
| Control | 50% | 530,000 |
| Treatment | 50% | 470,000 |

This may look close at first glance, but with large sample sizes, even a small percentage difference can be suspicious.

SRM is dangerous because it suggests that assignment, logging, eligibility, or data extraction may differ by group. If users are missing from treatment, or if some users are more likely to be assigned to one group, the comparison may no longer be randomized.

### Testing for SRM

For a two-group experiment, the expected counts are:

$$
E_C = p_C N
$$

$$
E_T = p_T N
$$

where $p_C$ and $p_T$ are the intended allocation probabilities and $N$ is the total observed sample size.

A common SRM test uses a chi-square statistic:

$$
\chi^2
=
\sum_g
\frac{(O_g - E_g)^2}{E_g}
$$

where $O_g$ is the observed count for group $g$ and $E_g$ is the expected count.

The statistic should include all experiment groups. If there are $k$ groups, then under the null hypothesis that the sample ratio is correct:

$$
\chi^2 \sim \chi^2_{k-1}
$$

The degree of freedom is $k - 1$ because the group counts must add up to the total sample size. Once the total $N$ is fixed, only $k - 1$ group counts are free to vary; the last count is determined by the others. For a 50/50 A/B test, $k = 2$, so the SRM test uses one degree of freedom.

The null hypothesis is:

> The observed group counts are consistent with the planned allocation.

If the SRM p-value is very small, the experiment has a health problem.

In practice, many teams use a stricter threshold for SRM than for treatment effects, such as $p < 0.001$. The reason is that SRM tests often run on very large sample sizes, and the cost of investigating a suspicious assignment problem is usually much lower than the cost of launching based on broken data.

### Common Causes of SRM

SRM can come from many places.

Randomization problems:

- The hash function is not stable.
- The bucketing range is incorrect.
- A user can be assigned multiple times.
- Some platforms or countries use a different assignment path.

Eligibility problems:

- Treatment users become eligible only after an event that control users do not need.
- One group is filtered differently in the analysis query.
- Triggering logic is applied after assignment in a biased way.

Logging problems:

- Assignment events are missing for one variant.
- Exposure events are logged only when the feature successfully loads.
- One client version fails to send treatment events.

Data processing problems:

- A join drops more users from one group.
- Duplicate users are deduplicated differently across variants.
- Time-zone logic includes one group for a different window.

Product problems:

- Treatment changes whether users continue far enough to be counted.
- Treatment redirects users to a different page where tracking differs.
- A feature rollout system fails for one platform.

SRM does not automatically tell the analyst which problem happened. It is an alarm, not a diagnosis.

### What To Do When SRM Appears

When SRM appears, the analyst should not immediately interpret the experiment result.

A useful debugging path is:

1. Check assignment counts directly from the experiment platform.
2. Compare assignment counts with exposure counts.
3. Break SRM down by platform, country, app version, browser, and time.
4. Check whether SRM starts at launch or appears after a specific deployment.
5. Rebuild the analysis sample step by step and identify where users are lost.
6. Confirm whether control users and treatment users pass through the same logging path.

If SRM exists at assignment time, the randomization itself may be broken. The experiment usually needs to be fixed and restarted.

If assignment counts are healthy but exposure counts show SRM, the issue may be downstream eligibility, loading, or logging. The analyst should decide whether the estimand is assignment-level intent-to-treat, exposure-level effect, or something else.

If SRM appears only after a data join or metric filter, the experiment may still be valid, but the analysis pipeline needs repair.

The safest rule is:

> Do not explain away SRM with product intuition until the data path has been checked.

## Assignment, Eligibility, and Exposure

Many experiment bugs come from confusing three related but different concepts:

| Concept | Meaning |
|---|---|
| Assignment | The user, session, item, or market is randomized to treatment or control |
| Eligibility | The unit qualifies for the experiment based on product rules |
| Exposure | The unit actually experiences the treatment or has the opportunity to experience it |

These concepts answer different questions.

Assignment answers:

> Who was randomized?

Eligibility answers:

> Who could have been affected?

Exposure answers:

> Who actually reached the part of the product where the treatment could matter?

For example, consider a checkout page experiment. All site visitors may be assigned to treatment or control, but only users who add an item to cart are eligible to see checkout, and only users who reach the checkout page are exposed.

If the analysis includes all assigned users, the estimate is close to an intent-to-treat effect. It measures the effect of being assigned to the new checkout experience.

If the analysis includes only exposed users, the estimate may be larger, but it can also become biased if treatment changes who reaches exposure.

This issue is discussed in more detail in Chapter 8. For quality checks, the key point is simpler:

> Assignment, eligibility, and exposure counts should be reconciled before metric interpretation.

A healthy experiment dashboard should make it possible to compare:

- Assigned users by variant
- Eligible users by variant
- Exposed users by variant
- Metric denominator users by variant
- Missing or invalid event counts by variant

If the counts diverge sharply, the analyst should understand why.

## Logging Validation

Experiment analysis is only as good as the events that feed it.

Logging validation asks whether the product events needed for the experiment were captured correctly and consistently across variants.

Important checks include:

- Does every assigned user have one stable variant assignment?
- Are assignment events logged before exposure events?
- Are exposure events logged only when the user could actually see the treatment?
- Are key metric events present on all platforms?
- Are event timestamps in the expected order?
- Are event names, schemas, and parameters consistent across variants?
- Are client-side and server-side logs reconciled where possible?

One common problem is treatment-dependent logging. This happens when the treatment changes the logging path itself.

For example, suppose treatment replaces a checkout button with a new component. If the new component logs clicks differently from the old component, then a change in click-through rate may reflect tracking differences rather than user behavior.

Another common problem is success-only exposure logging. If exposure is logged only after a treatment module successfully loads, then users who fail to load the treatment may disappear from the exposed treatment sample. This can make treatment look better than it is, because failed experiences are excluded.

Good exposure logging should usually happen when the system attempts to show the treatment, not only when the treatment succeeds perfectly.

## Metric Construction Checks

Even when raw logging is correct, metrics can still be wrong.

Metric construction checks ask whether the numerator, denominator, aggregation level, and time window match the intended definition.

For a ratio metric such as CTR:

$$
\text{CTR} = \frac{\text{clicks}}{\text{impressions}}
$$

the analyst should check:

- What counts as a click?
- What counts as an impression?
- Are clicks and impressions measured on the same surface?
- Are repeated clicks counted once or multiple times?
- Are bot impressions excluded from both numerator and denominator?
- Is the metric computed at the user level, impression level, or event level?

For a revenue metric such as GMV per user:

$$
\text{GMV per user} = \frac{\text{GMV}}{\text{users}}
$$

the analyst should check:

- Is GMV based on order creation, payment, shipment, or completion?
- Are canceled orders included?
- Are refunds subtracted?
- Are coupons and subsidies treated consistently?
- Is currency conversion handled correctly?
- Is the user denominator assignment-based, exposure-based, or buyer-based?

Small definition differences can completely change interpretation.

For example, if treatment increases paid orders but also increases refunds, then order-created GMV may look good while net revenue does not. If the metric definition does not specify the transaction state, the result may be misleading.

## Missing Data

Missing data is not automatically fatal. But missing data becomes dangerous when missingness differs by treatment group or is related to outcomes.

Suppose a mobile experiment tests a heavier recommendation module. If the module increases app crashes, then some treatment sessions may never send later engagement events. The observed treatment group may look less active because users truly disengaged, or because their events were lost after crashes.

Both are important, but they mean different things.

Useful missing-data checks include:

- Missing event rates by variant
- Missing event rates by platform and app version
- Crash rates and client error rates by variant
- Time from assignment to first event
- Share of users with partial event sequences
- Share of users with delayed outcome measurement

The analyst should be careful with imputation. Filling missing outcomes with zeros, averages, or model predictions can be reasonable in some settings, but it changes the estimand. The book-level rule is:

> Do not hide missing data inside the metric. Show it as a quality check.

If missingness is itself caused by the treatment, it may be a product effect, not only a data issue. For example, if treatment causes users to abandon the app before purchase, then missing purchase events are part of the treatment effect. But if treatment causes the purchase event logger to fail, that is a measurement problem.

The difference is not always obvious. That is why event sequence checks are useful.

## Baseline Balance Checks

Randomization should make treatment and control comparable before treatment.

Baseline balance checks compare pre-treatment user characteristics across variants:

- Country
- Platform
- App version
- Traffic source
- Historical activity
- Historical purchase behavior
- Historical value
- Tenure
- Device type

For a user-level experiment, these variables should be measured before assignment. If a variable can be affected by treatment, it should not be used as a baseline balance check.

Balance checks are useful because they can reveal assignment or data extraction problems. For example, if treatment has many more iOS users than control, then a platform-specific assignment issue may exist.

However, balance checks should be interpreted carefully. If the analyst checks hundreds of baseline variables, some differences will be statistically significant by chance.

The question is not:

> Is every baseline variable perfectly equal?

The better question is:

> Are there systematic, meaningful, or surprising imbalances that suggest randomization or logging is broken?

A good balance check focuses on variables that strongly predict the primary metric or reflect assignment paths.

## A/A Tests

An A/A test assigns users into two or more groups that receive the same product experience.

The purpose is not to estimate a product effect. The purpose is to test the experiment system.

In a healthy A/A test:

- Sample ratios should match the planned allocation.
- Baseline characteristics should be balanced.
- Primary metrics should not show systematic differences.
- False positive rates should be close to expected levels over many repeated tests.
- Logging and exposure events should behave symmetrically.

A/A tests are especially useful when:

- A new experimentation platform is launched.
- A new randomization unit is introduced.
- A metric pipeline is redesigned.
- A new client logging system is released.
- A high-stakes experiment depends on a complex triggering rule.

One A/A test cannot prove that the system is perfect. But repeated A/A tests can reveal whether the platform produces suspiciously many false positives, SRMs, or metric differences.

## Outliers and Heavy-Tailed Metrics

Many business metrics are heavy-tailed.

Revenue, GMV, watch time, delivery cost, and seller income often have a small number of extreme observations.

For example, in an e-commerce experiment, most users may spend nothing, many users may spend a small amount, and a few users may place very large orders. A single large enterprise order or fraud event can move the average.

This does not mean the average is wrong. Sometimes the average is exactly the business metric. But it does mean the estimate can be noisy and sensitive.

Useful outlier checks include:

- Distribution plots by variant
- Top users or orders contributing to metric movement
- Percentile comparisons, such as p50, p90, p99, and p99.9
- Winsorized or capped sensitivity analyses
- User-level aggregation before treatment-control comparison
- Separate analysis for consumer and enterprise segments, if relevant

The important rule is that outlier handling should not be chosen after seeing which version makes treatment look better.

If the team plans to cap GMV at the 99.9th percentile, winsorize session duration, or exclude fraud users, the rule should ideally be defined before the experiment is analyzed.

When outlier handling is exploratory, the analysis should say so clearly:

> The primary result uses the pre-specified metric. The capped analysis is a sensitivity check.

This keeps the decision honest.

## Bots, Fraud, and Internal Traffic

Some experiments are affected by traffic that does not represent real users.

Examples include:

- Search engine crawlers
- Automated scraping
- Fraud traffic
- Load testing
- Internal employees
- QA accounts
- Seller self-testing
- Repeated refreshes from monitoring systems

If this traffic is evenly distributed across variants and small relative to real traffic, it may not matter much. But if it is concentrated in one group or affects the metric strongly, it can distort the result.

Bot and internal traffic checks often include:

- Known bot user-agent filters
- Internal IP ranges
- Employee account lists
- Abnormally high event frequency
- Repeated identical event sequences
- Suspiciously high conversion or click rates
- Very short session durations with many events

The analyst should be careful not to over-filter. Removing users after seeing the result can create bias. The best practice is to use stable, pre-defined filters that are applied consistently across treatment and control.

## Time and Ramp Checks

Experiment results should be inspected over time before final interpretation.

This does not mean peeking for significance. It means checking whether the experiment behaved normally.

Useful time checks include:

- Assignment counts by hour or day
- Exposure counts by hour or day
- Metric denominators by hour or day
- Error rates by hour or day
- Treatment effects by day as a diagnostic plot
- Deployment or incident markers

These checks can reveal problems such as:

- The experiment started later for one variant.
- A logging bug began after a client release.
- A server outage affected only treatment.
- Ramp-up changed the user mix over time.
- A daily batch job produced duplicated events.

Time checks are especially important for experiments with staged rollouts. A treatment that starts at 1%, ramps to 10%, and then ramps to 50% may have different users at each stage. Early ramp users may be internal, low-risk, or geographically limited.

If the ramp strategy changes the experiment population, the analyst should not blindly pool all periods without checking whether the estimand remains the same.

## Segment-Level Debugging

Segment analysis is often discussed as a way to find heterogeneous treatment effects. But before that, it is also a debugging tool.

When a quality check fails, segment breakdowns can help locate the source.

Useful segments include:

- Platform
- App version
- Country or region
- Traffic source
- New versus returning users
- Logged-in versus logged-out users
- Browser
- Device type
- Experiment entry surface
- Time since assignment

For example, if SRM exists only on Android version 12.4, the issue may be a client-side assignment or logging bug. If missing purchase events appear only in one country, the issue may be a payment provider or local data pipeline.

Segment debugging should be disciplined. The goal is not to search for a segment where the treatment effect is significant. The goal is to find whether the experiment machinery behaves differently across known product paths.

## Practical Debugging Table

The table below summarizes common experiment health symptoms and likely causes.

| Symptom | Possible Causes | First Checks |
|---|---|---|
| SRM at assignment | Broken randomization, unstable user ID, incorrect allocation | Assignment logs, bucketing code, hash stability |
| SRM at exposure but not assignment | Biased triggering, load failures, treatment-only logging | Exposure logic, page load errors, eligibility rules |
| Metric denominator differs sharply | Missing events, changed eligibility, query filter bug | Funnel counts, event schema, analysis query |
| Treatment has more missing outcomes | Crashes, logging failure, delayed events, user abandonment | Error logs, event sequence, delayed metric windows |
| Huge metric movement from few users | Outliers, fraud, enterprise users, duplicated events | Distribution, top contributors, capped sensitivity |
| Effect appears only after a deployment | Logging change, product bug, incident | Time series, release notes, incident logs |
| Baseline covariates imbalanced | Randomization issue, data join issue, segment rollout | Pre-treatment covariates, assignment path |
| Control users receive treatment | Contamination, caching, shared devices, account switching | Exposure logs, treatment rendering logs |

This table is not a substitute for product knowledge. It is a starting map.

## When To Restart an Experiment

Not every issue requires restarting an experiment.

Some issues can be fixed in analysis:

- A known bot filter was missing but can be applied symmetrically.
- A metric query used the wrong time zone but can be corrected.
- A duplicated event source can be removed consistently.
- A delayed outcome window was too short and can be extended.

Some issues require caution but may not invalidate the experiment:

- A small amount of missing data affects both variants equally.
- A short logging incident affected both groups symmetrically.
- A few outliers move noisy secondary metrics but not the primary decision.

Other issues often require restarting:

- SRM exists at assignment and cannot be explained.
- Treatment and control were assigned using different rules.
- Control users were exposed to treatment at meaningful rates.
- The primary metric was not logged for one group.
- A product bug affected only one variant for a large share of users.
- The analysis population was changed after seeing the result.

Restarting is painful, but launching from corrupted evidence is worse.

The practical decision is:

> Can the team explain the problem, repair it symmetrically, and still answer the original causal question?

If not, the experiment should usually be rerun.

## Product Example: Checkout Experiment

Suppose an e-commerce team tests a new checkout page.

The planned allocation is 50/50. The primary metric is purchase conversion rate among users who enter checkout. Guardrails include payment error rate, page load latency, refund rate, and support complaints.

After one week, the treatment group shows:

- Purchase conversion: +6%
- Average order value: no significant change
- Payment error rate: no significant change
- Sample ratio: 54% control, 46% treatment

The conversion lift looks promising, but the SRM is a major warning sign.

The analyst should first identify where the ratio changes:

- Are assignment counts 50/50?
- Are checkout entry counts 50/50?
- Are page exposure events 50/50?
- Are purchase metric denominator users 50/50?

Suppose assignment is 50/50, but exposure is 54/46. Further investigation shows that treatment users on older Android versions often fail to load the new checkout page and are redirected to the old page without an exposure event.

Now the observed treatment group excludes many failed treatment experiences. The exposed-user conversion metric is biased upward because users with bad treatment experiences are missing from the treatment denominator.

The team should not launch based on the exposed-user lift. A better analysis would include all assigned checkout entrants as an intent-to-treat population, count load failures as part of treatment performance, and fix the Android issue before rerunning or continuing the test.

The key lesson is that quality checks change interpretation. The question is not only whether conversion increased. The question is whether the measured lift came from better checkout design or from missing failed sessions.

## Product Example: Recommendation Experiment

Consider a recommendation ranking experiment in an e-commerce feed.

The treatment model is designed to improve long-term GMV by promoting creators with higher repeat-purchase potential.

After two weeks, the treatment group shows:

- GMV per thousand impressions: +3%
- Repeat-purchase GMV: +5%
- Negative feedback: no significant change
- Total impressions per user: -4%

The metric result looks positive, but the impression decline needs a quality check.

Possible explanations include:

- The new ranking model makes the feed slower, so users see fewer impressions.
- The treatment changes impression logging.
- Treatment users spend less time in the feed.
- The model shows fewer but more commercially relevant items.
- One app version fails to log some impressions.

The analyst should compare:

- Feed entry counts
- Page load latency
- Impression logging rates
- Session duration
- App version breakdowns
- Clicks and orders per feed entry

If treatment truly shows fewer impressions but higher GMV per impression, the business interpretation may be positive or negative depending on total GMV, user experience, and long-term retention.

But if the impression denominator is missing because of a logging bug, then GPM is inflated mechanically. The experiment cannot support a launch decision until the denominator is repaired.

This example shows why metric decomposition and data quality checks belong together.

## A Practical Quality Checklist

Before interpreting an experiment result, the analyst should be able to answer:

1. Did the observed sample ratio match the planned allocation?
2. Are assignment, eligibility, exposure, and metric denominator counts reconciled?
3. Are treatment and control balanced on important pre-treatment characteristics?
4. Are assignment and exposure events logged consistently across variants?
5. Are primary metric numerator and denominator definitions correct?
6. Are missing data rates similar across variants?
7. Are bots, fraud, internal users, and test traffic handled consistently?
8. Are outliers understood and handled according to a pre-defined rule?
9. Are time trends stable, except for explainable ramps or incidents?
10. Do segment breakdowns reveal a platform, country, or version-specific issue?
11. Were any product launches, outages, or logging changes concurrent with the experiment?
12. Can the team still answer the original causal question?

This checklist should usually be completed before reading too much into statistical significance.

## Common Mistakes

The first common mistake is treating SRM as a minor warning. SRM is often the first visible sign that something deeper is wrong.

The second mistake is analyzing only exposed users without asking whether treatment affects exposure. This can turn a randomized experiment into a selected comparison.

The third mistake is trusting a metric without checking its construction. A ratio can move because the numerator changed, the denominator changed, or both changed for different reasons.

The fourth mistake is removing outliers after seeing the result. Outlier rules should be stable and defensible.

The fifth mistake is using segment analysis only to find exciting effects. Segment analysis is also a debugging tool, and often that is its first job.

The sixth mistake is treating data quality as separate from product interpretation. In real experiments, the two are connected. A load failure can be both a logging issue and a user experience issue. A missing purchase event can be either a tracking bug or evidence that users abandoned the flow.

## Key Takeaways

Experiment analysis starts with trust, not treatment effects.

SRM is one of the most important warning signs. If group counts do not match the planned allocation, the analyst should investigate before interpreting results.

Assignment, eligibility, exposure, and metric denominator are different concepts. Confusing them can create biased analysis.

Logging validation and metric construction checks are part of causal inference. If the measurement system differs by treatment group, the estimated effect may be a measurement artifact.

Missing data, outliers, bots, and time-based incidents should be examined before making a launch decision.

A good experiment report should not only say what changed. It should explain why the data is trustworthy enough to support that conclusion.
