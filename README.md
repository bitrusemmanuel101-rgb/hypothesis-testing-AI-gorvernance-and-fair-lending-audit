# Age Fairness Audit of a Bank Account Fraud Risk Score: Hypothesis Testing on the BAF Dataset

*Last updated: November 2026*

---

## Summary

This project tests whether a bank's automated account-opening risk process behaves differently for older and younger applicants, using a public benchmark of 1,000,000 synthetic applications. Older applicants (50 and over) had a fraud rate of 2.34% against 0.83% for younger applicants, about 2.8 times higher. Among legitimate applicants, older applicants also received higher average risk scores, by about 20 points (roughly 0.3 standard deviations).

Both gaps are statistically significant. The score gap is modest in size and does not on its own show unfair treatment, because fraud really is more common in the older group in this data.

---

## Context

Automated credit and risk decisions face growing regulatory scrutiny. In the EU, AI systems used to assess creditworthiness are classed as high-risk under the AI Act, with those obligations now due from 2 December 2027 following the 2026 Digital Omnibus changes. AI used only to detect financial fraud is excluded from that category. In the US, the Equal Credit Opportunity Act prohibits discrimination in credit, including on the basis of age.

This project is a statistical illustration on a public fraud benchmark. It is not a legal compliance test.

---

## Research Questions and Hypotheses

**Hypothesis 1:** Does the fraud rate differ between applicants aged 50 and over and applicants under 50?
- H0: the fraud rates are equal.
- H1: the fraud rates differ.

**Hypothesis 2:** Among legitimate applicants (`fraud_bool == 0`), does `credit_risk_score` differ between the two age groups?
- H0: the mean scores are equal.
- H1: the mean scores differ.

Significance level: 5%. Higher scores are read as higher risk.

---

## Data

**Source:** the Bank Account Fraud (BAF) dataset suite, Base variant (Jesus et al., NeurIPS 2022).

- 1,000,000 bank account applications, 32 columns, about 1.1% fraudulent (11,029 cases).
- The data are synthetic. They were generated from a real bank's fraud data using a GAN, with noise added and age binned into decades to protect privacy.
- Age is recorded in decades (10 to 90). "Older" means 50 and over, the convention used in other work on this dataset.
- No duplicate rows. Some columns use -1 as a missing-value code.

| Group | Applications | Share |
|---|---:|---:|
| Older (50 and over) | 183,013 | 18.3% |
| Younger (under 50) | 816,987 | 81.7% |

---

## Methodology

1. Loaded and explored the data in Python (pandas): types, duplicates, missing values, distributions.
2. **Hypothesis 1:** two-proportion z-test on fraud rate, with a 95% confidence interval for the difference and the relative risk.
3. **Hypothesis 2:** Welch's t-test on `credit_risk_score` among legitimate applicants, with a 95% confidence interval for the difference in means and Cohen's d as the effect size. Welch's test does not assume equal variances.

---

## Results

### Hypothesis 1: fraud rate

| Group | Applications | Fraudulent | Fraud rate |
|---|---:|---:|---:|
| Older (50 and over) | 183,013 | 4,285 | 2.34% |
| Younger (under 50) | 816,987 | 6,744 | 0.83% |

- Difference: **+1.52 percentage points** (95% CI 1.01 to 2.01)
- Relative risk: **2.84** (older applicants about 2.8 times as likely to be fraudulent)
- z = 56.1, p < 0.001. The exact p-value is too small for Python to store.
- Decision: reject H0.

### Hypothesis 2: risk score among legitimate applicants

| Group | Legitimate applicants | Mean risk score |
|---|---:|---:|
| Older (50 and over) | 178,728 | 146.8 |
| Younger (under 50) | 810,243 | 126.9 |

- Difference: **+20.0 points** (95% CI 17.2 to 22.7)
- Effect size: Cohen's d about 0.3 (small to moderate)
- Welch t = 104.1, p < 0.001
- Decision: reject H0.

---

## Interpretation

- With one million rows almost any gap is statistically significant, so the effect sizes matter more than the p-values.
- The fraud gap is large and is likely to be a real feature of this dataset.
- The score gap is consistent with the score penalising age. It is equally consistent with the score using features that are correlated with age and that really do predict fraud. These tests cannot separate the two.
- A higher average score among legitimate older applicants means they are more likely to be wrongly flagged at a given alert threshold. That is the practical concern, and it is what the next steps test.

---

## Conclusion

Older applicants had a much higher fraud rate and also higher average risk scores among legitimate applicants. The data cannot show whether the score treats older applicants unfairly. Answering that needs a comparison of error rates at the alert threshold.

## Next Steps

- Compare the false positive rate between age groups at a fixed alert rate (predictive equality).
- Check calibration by group: does a given score mean the same fraud probability for older and younger applicants?
- Plot fraud rate by age decade to see whether the 50 cutoff matches the pattern.
- Control for other features, such as income and employment status, with a regression model.

---

## Limitations

- The data are synthetic, so results describe the benchmark and not real customers.
- Age is binned into decades, and the groups are unequal in size.
- The tests show association, not cause.
- The analysis uses a score, not actual approval decisions.
- Missing values coded as -1 were not treated, because they do not affect the variables used.

---

## Tools

Python (pandas, NumPy, SciPy, statsmodels), Jupyter Notebook, Git and GitHub.

## Repository Structure

```
age-fairness-audit-baf/
├── data/
│   └── raw/             Base.csv (download from Kaggle, not committed)
├── notebooks/
│   └── hypothesis-testing-ai-governance-fairness-audit.ipynb
├── visuals/
├── requirements.txt
└── README.md
```

## How to Run

1. Install the packages: `pip install -r requirements.txt`
2. Download `Base.csv` from the BAF dataset on Kaggle and place it in `data/raw/`.
3. Open the notebook and choose **Run All**.

## Sources

- Jesus et al. (2022). *Turning the Tables: Biased, Imbalanced, Dynamic Tabular Datasets for ML Evaluation.* NeurIPS 2022.
- BAF dataset: https://github.com/feedzai/bank-account-fraud

---

*Author: Emmanuel Bitrus*
