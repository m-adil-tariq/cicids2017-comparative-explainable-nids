# Comparative and Explainable Network Intrusion Detection on CICIDS2017

A controlled, multi-model comparison study on CICIDS2017 that goes beyond training a single classifier — combining seven algorithms, consensus feature selection, SHAP explainability, and rigorous statistical significance testing.

Read Full Report here: [Report.pdf](./Report.pdf)

## Overview

This project builds a leakage-free experimental pipeline to compare seven classical ML classifiers on multiclass network intrusion detection (Normal, DoS, DDoS, Port Scanning, Brute Force, Web Attacks, Bots). It extends beyond a standard benchmark in three ways:

1. **Feature selection** — four techniques (Variance Threshold, Mutual Information, ANOVA F-test, RFE) are combined into consensus feature sets, and dimensionality reduction effects are measured per model.
2. **Explainability** — the strongest model is interpreted with SHAP, both globally and for individual correctly/incorrectly classified flows.
3. **Statistical rigor** — the top three models are re-evaluated across five random seeds, with Wilcoxon signed-rank tests (Holm-corrected) to check whether ranking differences are statistically meaningful.

## Dataset

- **Source:** Pre-processed CICIDS2017 CSV (Kaggle)
- **Size:** 2,520,751 records → 2,517,681 after cleaning (removed infinities, impossible negative values, 161 exact duplicates)
- **Classes:** Normal Traffic (83.11%), DoS, DDoS, Port Scanning, Brute Force, Web Attacks, Bots (three rarest classes together < 0.5%)

## Methodology

1. **Preprocessing & splitting** — 80/20 stratified split; scaling and feature selection fit only on training data to prevent leakage.
2. **Classifiers compared** — Logistic Regression, Decision Tree, Random Forest, Extra Trees, Gradient Boosting, KNN, Gaussian Naive Bayes.
3. **Feature selection** — Variance Threshold, Mutual Information, ANOVA F-test (k=10/20/30/50), RFE (20 features); combined into strict-consensus (3 features) and majority-vote (22 features) sets.
4. **Explainability** — SHAP TreeExplainer on Random Forest (majority-vote feature set), including global importance, class-specific beeswarm, and 4 local waterfall explanations.
5. **Repeated experiments** — top 3 models re-run across 5 seeds; Wilcoxon signed-rank test with Holm correction on pairwise macro-F1 differences.

## Key Findings

- **Tree-based ensembles dominate:** Random Forest, Decision Tree, and Extra Trees all achieve macro-F1 > 0.95 on the full 52-feature set, while Logistic Regression (0.69) and Naive Bayes (0.66) lag far behind — despite both exceeding 78% raw accuracy, illustrating why accuracy alone is misleading under class imbalance.
- **Bots is the hardest class for nearly every model** (recall 0.59–0.78), almost always misclassified as Normal Traffic — consistent with botnet traffic designed to blend in with benign activity, and consistent with findings from a related CICIDS2017 project.
- **Feature selection strategy matters more than feature count:** a 22-feature majority-vote consensus set (58% dimensionality reduction) preserved ensemble macro-F1 almost exactly (0.9659 vs. 0.9653 full-feature), while a single-method ANOVA top-20 selection of the same size scored notably worse (0.9238).
- **The benefit isn't universal:** Logistic Regression, Naive Bayes, and Gradient Boosting all lost meaningful macro-F1 after reduction to the majority-vote set — likely because the surviving features are dominated by correlated packet-length/size statistics, which suit tree splits but hurt linear/probabilistic models.
- **SHAP confirms the same feature cluster** (Destination Port, packet-length, window-size stats) drives predictions, and shows the model was essentially "unsure" (near-zero confidence margin) on a misclassified Bots flow rather than confidently wrong.
- **Model ranking is not statistically significant:** despite Random Forest scoring highest on average across 5 seeds, Holm-corrected Wilcoxon tests found no significant difference vs. Extra Trees or Decision Tree (all adjusted p ≈ 0.19).
- **Computational cost matters for practical deployment:** KNN's inference took ~40 minutes on ~500K test flows despite competitive accuracy, making it impractical for real-time use, whereas Random Forest combined top accuracy with inference in seconds.

## Limitations

- SHAP explainability was applied only to Random Forest, not to the other top competitors.
- Only the DoS class received a full SHAP beeswarm visualization.
- Uses a random stratified split rather than a temporal/session-aware split, so same-session flows could appear on both sides — a known characteristic of CICIDS2017.
- Feature importance/SHAP values reflect statistical association, not causal attacker behavior.

## Conclusion

Under a controlled, leakage-free pipeline, tree-based ensembles clearly outperform linear/probabilistic classifiers on CICIDS2017 once evaluation moves past raw accuracy. Consensus-based feature selection can cut dimensionality by more than half with negligible performance loss — but only if the selection strategy is chosen carefully, not just the feature count. SHAP corroborates these feature-selection findings, and computational cost proves to be a meaningful factor separate from raw accuracy when choosing a model for real-world deployment.

## References

- Sharafaldin, I., Lashkari, A. H., & Ghorbani, A. A. (2018). *Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization.* ICISSP.
- Lanvin, M., Gimenez, P. F., Han, Y., Majorczyk, F., Mé, L., & Totel, É. (2023). *Errors in the CICIDS2017 Dataset and the Significant Differences in Detection Performances It Makes.* CRiSIS 2022.
- Ribeiro, E. A. (2024). *CICIDS2017: Cleaned & Preprocessed.* Kaggle.

## Author

Muhammad Adil (Independent research project)
