# Beyond Simple A/B Tests

## A Practical Guide to Modern Experimentation

## Contents

### Part I: Foundations

**Chapter 1: What Are We Really Testing?**  
Hypotheses, treatment effects, metrics, guardrails, and the difference between statistical significance and business decisions.

**Chapter 2: Designing a Clean Experiment**  
Randomization units, eligibility, exposure, sample size, MDE, duration, and experiment validity checks.

**Chapter 3: Metrics Are Not Just Numbers**  
Ratio metrics, delayed metrics, proxy metrics, metric decomposition, and why "more clicks" may not mean "more value."

### Part II: Inference Problems

**Chapter 4: The Peeking Problem**  
Why checking results repeatedly breaks fixed-horizon tests, and how sequential testing, alpha spending, and Bayesian monitoring help.

**Chapter 5: Variance Reduction**  
CUPED, stratification, blocking, regression adjustment, and how to get more power without more traffic.

**Chapter 6: Multiple Testing**  
What happens when we test many metrics, segments, or variants, and how to control false positives.

### Part III: Real-World Experiment Complications

**Chapter 7: Triggered Analysis and Noncompliance**  
Intent-to-treat, treatment-on-treated, exposure bias, and why analyzing only exposed users can be dangerous.

**Chapter 8: Interference and Network Effects**  
When one user's treatment affects another user's outcome, with switchback tests, geo experiments, and marketplace examples.

**Chapter 9: Concurrent Experiments**  
How overlapping experiments interact and how companies manage experiment layers.

### Part IV: Learning Beyond the Average

**Chapter 10: Heterogeneous Treatment Effects**  
Why the average effect can hide important segment differences, and how to explore treatment effects responsibly.

**Chapter 11: Long-Term Effects and Holdouts**  
Delayed metrics, novelty effects, learning effects, LTV, and when to keep long-term control groups.

**Chapter 12: Bandits and Adaptive Experiments**  
Multi-armed bandits, exploration versus exploitation, regret, and when optimization is more important than clean measurement.

### Part V: When Randomization Is Hard

**Chapter 13: Quasi-Experiments**  
Difference-in-differences, synthetic control, regression discontinuity, matching, and instrumental variables.

**Chapter 14: Experiment Quality Checks**  
Sample ratio mismatch, logging validation, missing data, bots, outliers, and practical debugging.

**Chapter 15: From Result to Decision**  
Metric tradeoffs, guardrail violations, rollout strategy, and how to make a launch decision under uncertainty.

## Chapter Files

- [Chapter 1: What Are We Really Testing?](chapter-01-what-are-we-really-testing.md)
- [Chapter 2: Designing a Clean Experiment](chapter-02-designing-a-clean-experiment.md)
- [Chapter 3: Metrics Are Not Just Numbers](chapter-03-metrics-are-not-just-numbers.md)
- [Chapter 4: The Peeking Problem](chapter-04-the-peeking-problem.md)
- [Chapter 5: Variance Reduction](chapter-05-variance-reduction.md)
- [Chapter 6: Multiple Testing](chapter-06-multiple-testing.md)
- [Chapter 7: Triggered Analysis and Noncompliance](chapter-07-triggered-analysis-and-noncompliance.md)
- [Chapter 8: Interference and Network Effects](chapter-08-interference-and-network-effects.md)
- [Chapter 9: Concurrent Experiments](chapter-09-concurrent-experiments.md)
- [Chapter 10: Heterogeneous Treatment Effects](chapter-10-heterogeneous-treatment-effects.md)
- [Chapter 11: Long-Term Effects and Holdouts](chapter-11-long-term-effects-and-holdouts.md)
- [Chapter 12: Bandits and Adaptive Experiments](chapter-12-bandits-and-adaptive-experiments.md)

## Website

This book is set up as a Quarto book website.

To preview it locally after installing Quarto:

```bash
quarto preview
```

To render the static site:

```bash
quarto render
```

The rendered website will be created in `_book/`.

To publish with GitHub Pages:

1. Create a GitHub repository for this folder.
2. Push the book files to the repository.
3. In GitHub, go to `Settings > Pages`.
4. Set the source to `GitHub Actions`.
5. Push to `main` or run the `Publish Quarto Book` workflow manually.
