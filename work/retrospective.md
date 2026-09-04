# FlyRank ML Internship — Retrospective

## A Note to My Week 1 Self

When I started this track, I wanted to become better at turning machine learning knowledge into something that could actually be shipped, explained, and evaluated. I knew how to build models and work with Python, but I wanted to become more disciplined about the part that comes before and after the model: defining a useful question, understanding the data, checking for leakage, validating honestly, communicating results, and turning predictions into decisions.

The biggest change was that I stopped thinking of the model as the main product. My capstone started with a research question: can historical search, traffic, engagement, and content-freshness signals identify content that is currently showing a declining trend, and which signals provide useful evidence for prioritizing content review? That question forced me to make decisions before training anything. I defined the target, documented the data, excluded target-derived fields such as `trend_direction` and `trend_pct`, and treated client identity as a grouping variable rather than a predictive feature.

One of the most important things that changed in how I work was my approach to validation. A random train/test split initially produced stronger-looking results, with a ROC-AUC of 0.9124 and F1 of 0.8339. Instead of presenting those numbers as the final result, I asked whether the split reflected the way the data could actually be reused across clients. I introduced a client-grouped split with zero client overlap between train and test. Performance dropped to a ROC-AUC of 0.8497 and F1 of 0.7422. That lower number was more useful because it gave me a more conservative estimate of generalization. I learned that a weaker-looking result can be a stronger research result when the evaluation design is more defensible.

The second major change was in how I interpret model outputs. The final Logistic Regression model achieved 0.8497 ROC-AUC on the grouped test set, compared with 0.5000 for the majority baseline. I also inspected standardized coefficients rather than treating the model as a black box. Recent and previous-period impression signals were among the strongest coefficients, which gave me a useful direction for interpretation. At the same time, I learned to describe these coefficients as associations rather than causal explanations.

The third change was learning to connect ML results to an actual workflow. A prediction alone is not an action plan. I translated the analysis into a review-prioritization playbook using transparent reason codes such as impression decline, session decline, low search position, stale content, and lack of recent impressions. The intended use is to help prioritize human review, not to automatically change or remove content.

If I were starting Week 1 again, I would spend even more time defining the evaluation design before touching the model. I would also plan the final artifacts and deployment earlier, because a strong analysis becomes much more valuable when another person can reproduce it and see it running.

My next step is to build on this workflow with another real case study. I plan to add CreditGuard ML, my credit-risk classification project, as the next portfolio case. I want to apply the same discipline: clear problem definition, reproducible preprocessing, comparable baselines, honest validation, useful visualizations, and a practical interpretation of the results.

The three most transferable things I learned are:

1. **Validation design is part of the model.** The split determines what the evaluation actually means, so it should reflect the real generalization problem.

2. **Good ML work includes the full evidence chain.** Question → data → assumptions → preprocessing → model → baseline → validation → interpretation → decision is more valuable than a model score by itself.

3. **Shipping changes the standard of quality.** A project becomes much stronger when the notebook, report, artifacts, deployment, and explanation all connect and another person can inspect the work without needing the original author beside them.

The goal I had in Week 1 was to become better at building ML systems. I am leaving the track with a broader goal: building ML work that is not only technically functional, but also reproducible, honestly evaluated, explainable, and useful to someone making a decision.
