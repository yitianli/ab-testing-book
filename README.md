# Beyond Simple A/B Tests

## A Practical Guide to Modern Experimentation

## Contents

### Part I: Foundations

**Chapter 1: From Product Idea to Experiment Question**  
Hypotheses, treatment effects, metrics, guardrails, and the difference between statistical significance and business decisions.

**Chapter 2: Designing a Clean Experiment**  
Randomization units, eligibility, exposure, sample size, MDE, duration, and experiment validity checks.

**Chapter 3: Metrics Are Not Just Numbers**  
Ratio metrics, delayed metrics, proxy metrics, metric decomposition, and why "more clicks" may not mean "more value."

### Part II: Inference and Evidence Problems

**Chapter 4: The Peeking Problem**  
Why checking results repeatedly breaks fixed-horizon tests, and how sequential testing, alpha spending, and Bayesian monitoring help.

**Chapter 5: Multiple Testing**  
What happens when we test many metrics, segments, or variants, and how to control false positives.

**Chapter 6: Variance Reduction**  
CUPED, stratification, blocking, regression adjustment, and how to get more power without more traffic.

**Chapter 7: Long-Term Effects and Holdouts**  
Delayed metrics, novelty effects, learning effects, LTV, and when to keep long-term control groups.

### Part III: Real-World Experiment Complications

**Chapter 8: Triggered Analysis and Noncompliance**  
Intent-to-treat, treatment-on-treated, exposure bias, and why analyzing only exposed users can be dangerous.

**Chapter 9: Interference and Network Effects**  
When one user's treatment affects another user's outcome, with switchback tests, geo experiments, and marketplace examples.

### Part IV: Learning and Optimization

**Chapter 10: Heterogeneous Treatment Effects**  
Why the average effect can hide important segment differences, and how to explore treatment effects responsibly.

**Chapter 11: Bandits and Adaptive Experiments**  
Multi-armed bandits, exploration versus exploitation, regret, and when optimization is more important than clean measurement.

**Chapter 12: Ranking Experiments and Interleaving**  
Why ranking experiments are different, how interleaving compares rankers, and how to interpret paired preference results.

### Part V: Experiment Platforms and Operations

**Chapter 13: Concurrent Experiments**  
How overlapping experiments interact and how companies manage experiment layers.

**Chapter 14: Experiment Quality Checks**  
Sample ratio mismatch, logging validation, missing data, bots, outliers, and practical debugging.

**Chapter 15: From Result to Decision**  
Metric tradeoffs, guardrail violations, rollout strategy, and how to make a launch decision under uncertainty.

### Part VI: When Randomization Is Hard

**Chapter 16: Quasi-Experiments**  
Difference-in-differences, synthetic control, regression discontinuity, matching, and instrumental variables.

## Chapter Files

### Part I: Foundations

- [Part I: Foundations](part-01-foundations.md)
- [Chapter 1: From Product Idea to Experiment Question](chapter-01-what-are-we-really-testing.md)
- [Chapter 2: Designing a Clean Experiment](chapter-02-designing-a-clean-experiment.md)
- [Chapter 3: Metrics Are Not Just Numbers](chapter-03-metrics-are-not-just-numbers.md)

### Part II: Inference and Evidence Problems

- [Part II: Inference and Evidence Problems](part-02-inference-problems.md)
- [Chapter 4: The Peeking Problem](chapter-04-the-peeking-problem.md)
- [Chapter 5: Multiple Testing](chapter-05-multiple-testing.md)
- [Chapter 6: Variance Reduction](chapter-06-variance-reduction.md)
- [Chapter 7: Long-Term Effects and Holdouts](chapter-07-long-term-effects-and-holdouts.md)

### Part III: Real-World Experiment Complications

- [Part III: Real-World Experiment Complications](part-03-real-world-experiment-complications.md)
- [Chapter 8: Triggered Analysis and Noncompliance](chapter-08-triggered-analysis-and-noncompliance.md)
- [Chapter 9: Interference and Network Effects](chapter-09-interference-and-network-effects.md)

### Part IV: Learning and Optimization

- [Part IV: Learning and Optimization](part-04-learning-and-optimization.md)
- [Chapter 10: Heterogeneous Treatment Effects](chapter-10-heterogeneous-treatment-effects.md)
- [Chapter 11: Bandits and Adaptive Experiments](chapter-11-bandits-and-adaptive-experiments.md)
- [Chapter 12: Ranking Experiments and Interleaving](chapter-12-ranking-experiments-and-interleaving.md)

### Part V: Experiment Platforms and Operations

- [Part V: Experiment Platforms and Operations](part-05-experiment-platforms-and-operations.md)
- [Chapter 13: Concurrent Experiments](chapter-13-concurrent-experiments.md)
- [Chapter 14: Experiment Quality Checks](chapter-14-experiment-quality-checks.md)
- [Chapter 15: From Result to Decision](chapter-15-from-result-to-decision.md)

### Part VI: When Randomization Is Hard

- [Part VI: When Randomization Is Hard](part-06-when-randomization-is-hard.md)
- [Chapter 16: Quasi-Experiments](chapter-16-quasi-experiments.md)

## Website

The published book website is here:

[https://yitianli.github.io/ab-testing-book/](https://yitianli.github.io/ab-testing-book/)

The source repository is here:

[https://github.com/yitianli/ab-testing-book](https://github.com/yitianli/ab-testing-book)

This book is built with Quarto and published with GitHub Pages through GitHub Actions.

To preview the book locally:

```bash
quarto preview
```

To render the static site locally:

```bash
quarto render
```

The rendered website will be created in `_book/`.

To update the published website:

1. Edit the Markdown or Quarto source files.
2. Commit the changes.
3. Push to `main`.

GitHub Actions will rebuild and publish the website automatically.
