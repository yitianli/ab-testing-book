# Multiple Testing

One experiment rarely produces one number.

A checkout experiment may report conversion rate, revenue per visitor, average order value, payment error rate, refund rate, page latency, and support contacts. A recommendation experiment may report watch time, retention, likes, hides, reports, creator diversity, and revenue. Analysts often slice these metrics by country, device, user tenure, acquisition channel, and product category.

Each comparison is a test.

The more tests we run, the more chances we have to find something that looks significant by random noise alone.

This is the multiple testing problem.

It appears whenever a team tests:

- Many metrics
- Many variants
- Many segments
- Many time windows
- Many model versions
- Many guardrails
- Many exploratory hypotheses

Multiple testing does not mean the analysis is wrong. It means the interpretation needs discipline.

## The Basic Problem

Suppose the treatment has no real effect on any metric. For one test with:

$$
\alpha = 0.05
$$

the probability of a false positive is 5%.

Now suppose the team tests 20 independent metrics, each at \(\alpha = 0.05\). The probability of seeing at least one false positive is:

$$
1 - (1 - 0.05)^{20}
$$

which is:

$$
1 - 0.95^{20} \approx 64\%
$$

Even when nothing is real, at least one metric is likely to look significant.

This is why a dashboard with many p-values can be dangerous. If the team searches long enough, the data may offer a story.

## Family-Wise Error Rate

One way to define the multiple testing problem is through the family-wise error rate, or FWER.

FWER is the probability of making at least one false positive within a family of tests.

A family might be:

- All primary comparisons in one experiment
- All variants compared against control
- All metrics used for a launch decision
- All segment tests in a predefined subgroup analysis

The word "family" matters because correction depends on what set of tests should be controlled together.

If a team tests one primary metric and ten secondary diagnostics, it may not need to adjust the primary metric for all diagnostic tests if the primary metric was clearly pre-specified. But if the team is willing to launch based on any of the eleven metrics, then those metrics form a decision family.

The practical question is:

> Across which tests are we allowing ourselves to claim a win?

That set is the family that needs error control.

## Bonferroni Correction

The simplest multiple testing correction is Bonferroni.

If the team runs \(m\) tests and wants the family-wise error rate to be at most \(\alpha\), Bonferroni tests each individual hypothesis at:

$$
\frac{\alpha}{m}
$$

For example, if:

$$
\alpha = 0.05,\quad m = 10
$$

then each test uses:

$$
0.05 / 10 = 0.005
$$

The Bonferroni rule is:

> Declare a result significant only if \(p_i \le \alpha/m\).

Bonferroni is easy to understand and works under very general conditions. It does not require tests to be independent.

The downside is that it can be conservative, especially when many tests are correlated. If metrics move together, Bonferroni may reduce false positives at the cost of missing real effects.

## Holm Correction

Holm's method is a step-down version of Bonferroni. It also controls the family-wise error rate, but it is usually less conservative.

The procedure is:

1. Sort the \(m\) p-values from smallest to largest:

$$
p_{(1)} \le p_{(2)} \le \cdots \le p_{(m)}
$$

2. Compare the smallest p-value to:

$$
\frac{\alpha}{m}
$$

3. Compare the second-smallest p-value to:

$$
\frac{\alpha}{m-1}
$$

4. Continue this way. At rank \(k\), compare:

$$
p_{(k)} \le \frac{\alpha}{m-k+1}
$$

5. Once a p-value fails the test, stop. All larger p-values are not significant.

For example, suppose five metrics have p-values:

$$
0.003,\quad 0.012,\quad 0.020,\quad 0.041,\quad 0.20
$$

With \(\alpha = 0.05\), Holm compares:

| Rank | P-Value | Threshold |
|---:|---:|---:|
| 1 | 0.003 | 0.010 |
| 2 | 0.012 | 0.0125 |
| 3 | 0.020 | 0.0167 |
| 4 | 0.041 | 0.025 |
| 5 | 0.200 | 0.050 |

The first two pass. The third fails, so the procedure stops. Only the first two are significant under Holm.

Holm is often a good default when the team wants family-wise error control but does not want Bonferroni to be unnecessarily strict.

## False Discovery Rate

Family-wise error rate controls the chance of any false positive. This can be too strict when the goal is discovery.

False discovery rate, or FDR, controls the expected share of false discoveries among all discoveries.

This is useful when a team runs many exploratory tests and expects several true effects, such as:

- Testing many user segments
- Testing many product categories
- Screening many model features
- Comparing many creative variants
- Looking for heterogeneous treatment effects

The most common FDR method is Benjamini-Hochberg.

## Benjamini-Hochberg Procedure

Benjamini-Hochberg controls FDR at a chosen level \(q\), such as 0.05 or 0.10.

The procedure is:

1. Sort the \(m\) p-values:

$$
p_{(1)} \le p_{(2)} \le \cdots \le p_{(m)}
$$

2. Find the largest rank \(k\) such that:

$$
p_{(k)} \le \frac{k}{m}q
$$

3. Declare \(p_{(1)}, \ldots, p_{(k)}\) significant.

For example, suppose ten segment tests have p-values:

$$
0.002,\ 0.006,\ 0.011,\ 0.018,\ 0.029,\ 0.041,\ 0.090,\ 0.20,\ 0.40,\ 0.70
$$

With \(q = 0.10\), the Benjamini-Hochberg thresholds are:

| Rank | P-Value | Threshold \((k/m)q\) |
|---:|---:|---:|
| 1 | 0.002 | 0.010 |
| 2 | 0.006 | 0.020 |
| 3 | 0.011 | 0.030 |
| 4 | 0.018 | 0.040 |
| 5 | 0.029 | 0.050 |
| 6 | 0.041 | 0.060 |
| 7 | 0.090 | 0.070 |
| 8 | 0.200 | 0.080 |
| 9 | 0.400 | 0.090 |
| 10 | 0.700 | 0.100 |

The largest passing rank is 6, so the first six p-values are discoveries.

This does not mean all six are true effects. It means the procedure controls the expected false discovery rate at the chosen level, under its assumptions.

FDR is less conservative than FWER. It is often appropriate for exploration, but less appropriate when a single false positive would create a costly launch decision.

## Primary Metrics and Secondary Metrics

Multiple testing is easier to manage when the experiment has a clear metric hierarchy.

A common structure is:

- One primary metric for the launch decision
- Secondary metrics for mechanism
- Guardrails for harm
- Exploratory metrics for learning

The primary metric should be chosen before the experiment starts. If there is only one primary metric, its p-value usually does not need correction for all secondary and exploratory metrics.

This is because the primary metric is the pre-specified decision test.

But the team should not quietly switch primary metrics after seeing the results. If the original primary metric is flat and one secondary metric is significant, that secondary metric should not be treated as a clean confirmatory win unless the multiple testing issue is addressed.

The language matters:

- "The pre-specified primary metric improved significantly" is confirmatory.
- "One of several exploratory metrics improved" is suggestive.

Both can be useful. They should not be presented as the same level of evidence.

## Guardrails and Multiple Testing

Guardrails create a subtle multiple testing problem.

Suppose an experiment has one primary metric and twelve guardrails. If one guardrail shows a significant degradation at \(p < 0.05\), is that enough to block launch?

The answer depends on context.

For severe safety, privacy, payment, or reliability guardrails, the team may not want to apply a strict correction before acting. A large degradation in a critical guardrail may justify investigation or rollback even if many guardrails were monitored.

For softer guardrails, such as small movements in secondary engagement metrics, multiple testing correction may be appropriate.

The practical approach is to classify guardrails by severity:

**Hard guardrails**

Metrics where degradation is unacceptable or risky, such as crashes, payment failures, safety reports, severe latency, or fraud. These are monitored conservatively, and practical harm can matter more than corrected significance.

**Soft guardrails**

Metrics that provide context but do not automatically block launch, such as small changes in likes, time spent, or optional feature usage. These should be interpreted with the broader metric pattern.

The purpose of guardrails is risk control, not discovery. A guardrail warning should often trigger investigation before it triggers a final decision.

## Many Variants

Multiple testing also appears when testing more than two variants.

Suppose a team tests four onboarding flows:

- Control
- Variant A
- Variant B
- Variant C

If each variant is compared with control at \(\alpha = 0.05\), the chance of at least one false positive increases.

If the goal is to choose any winning variant against control, the comparisons form a family.

For three treatment-control comparisons, Bonferroni would use:

$$
0.05 / 3 \approx 0.0167
$$

for each comparison.

Another option is to first run an omnibus test, such as an ANOVA or chi-square test, asking:

> Is there any difference among the variants?

If the omnibus test is significant, the team can then examine pairwise comparisons. This approach is common in classical statistics, though product experiment platforms often prefer direct pairwise comparisons with correction.

The right method depends on the decision:

- Choose the best of many variants
- Confirm that one pre-selected variant beats control
- Learn which design elements matter
- Screen many ideas before a later confirmatory test

Different goals require different error-control strategies.

## Segment Analysis

Segment analysis is one of the easiest places to fool ourselves.

Suppose the overall treatment effect is flat. The analyst then checks:

- New users
- Returning users
- iOS users
- Android users
- Desktop users
- Paid acquisition users
- Organic users
- High-activity users
- Low-activity users
- Ten product categories
- Twenty countries

Eventually, some segment may show a significant lift.

This does not mean the segment effect is real. It may be random noise from many comparisons.

Segment analysis should be separated into two types:

**Pre-specified segment analysis**

Segments chosen before the experiment because there is a strong prior reason to expect different effects. These can be interpreted more seriously, especially if the number of segments is limited.

**Exploratory segment analysis**

Segments searched after seeing the result. These are useful for generating hypotheses, but they need confirmation in future experiments or correction for multiple testing.

For heterogeneous treatment effects, the cleanest evidence often comes from:

- Pre-specified segments
- Interaction tests
- Correction for multiple comparisons
- Replication in follow-up experiments

## Interaction Tests

A common mistake is comparing significance across segments incorrectly.

Suppose treatment is significant for iOS users but not significant for Android users. This does not automatically mean the treatment effect is different between iOS and Android.

The correct question is:

> Is the treatment effect significantly different across segments?

This is an interaction test.

A regression model might include:

$$
Y_i = \alpha + \tau T_i + \gamma S_i + \delta(T_i \times S_i) + \epsilon_i
$$

where:

- \(T_i\) is treatment assignment
- \(S_i\) is a segment indicator
- \(T_i \times S_i\) is the interaction
- \(\delta\) measures whether the treatment effect differs by segment

The segment difference is tested through \(\delta\), not by checking whether one segment has \(p < 0.05\) and another segment has \(p > 0.05\).

This distinction matters because one segment may be noisier or smaller than another. A result can be significant in one group and not significant in another even when the estimated effects are similar.

## Practical Significance Still Matters

Multiple testing corrections control statistical error, but they do not decide whether an effect matters.

With enough traffic, a tiny corrected-significant effect may still be practically irrelevant. With limited traffic, a meaningful effect may not survive a strict correction.

The experiment report should separate:

- Effect size
- Confidence interval
- Statistical significance
- Multiple testing adjustment
- Business importance

For example:

> Revenue per visitor increased by 0.15%, with a 95% confidence interval from 0.03% to 0.27%. This result remains significant after correcting for three primary variant comparisons. The expected annualized impact is meaningful relative to launch cost.

This is much more informative than:

> The p-value is significant.

## A Product Example

Suppose a video app tests a new recommendation algorithm. The pre-specified primary metric is 7-day retention. The team also monitors:

- Watch time
- Completion rate
- Like rate
- Follow rate
- Reported content
- Not-interested rate
- Creator diversity
- Revenue

The result is:

- 7-day retention: no significant change
- Watch time: +3%, \(p = 0.03\)
- Completion rate: +2%, \(p = 0.04\)
- Like rate: no significant change
- Reported content: +5%, \(p = 0.06\)
- Other metrics: no clear movement

If the team treats every metric as a possible launch metric, the watch-time and completion-rate p-values are less convincing because many metrics were checked.

If retention was truly the pre-specified primary metric, the clean interpretation is:

> The primary metric did not improve. Some engagement metrics moved positively, but those results are secondary and should be interpreted cautiously in light of multiple testing and the reported-content warning.

A reasonable decision might be to iterate, run a follow-up experiment with engagement as a pre-specified objective, or investigate whether the watch-time lift comes from healthy content. The experiment is not a clear launch win.

## Practical Workflow

A practical workflow for handling multiple testing is:

1. Define the primary metric before launch.
2. Decide which tests form the main decision family.
3. Separate confirmatory metrics from exploratory diagnostics.
4. Choose the correction method before looking at results.
5. Use FWER control when false positives are costly.
6. Use FDR control when the goal is discovery across many hypotheses.
7. Treat post-hoc segment wins as hypotheses unless corrected or replicated.
8. Report effect sizes and confidence intervals, not only adjusted p-values.

This workflow keeps the analysis honest without preventing useful exploration.

## Key Takeaways

Multiple testing creates more chances for false positives.

The family-wise error rate is the probability of at least one false positive within a family of tests.

Bonferroni is simple and conservative.

Holm controls family-wise error and is usually less conservative than Bonferroni.

Benjamini-Hochberg controls false discovery rate and is useful for exploratory analysis.

A pre-specified primary metric reduces ambiguity.

Secondary and exploratory metrics should not quietly become the launch metric after results are known.

Segment analysis is especially vulnerable to multiple testing.

An effect significant in one segment but not another does not prove heterogeneity; use an interaction test.

Statistical correction does not replace business judgment.
