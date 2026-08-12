# Quant_Credit_Scorecard
# Retail Lending Application Scorecard — End-to-End Credit Risk Project

An end-to-end credit scoring project built to demonstrate the full lifecycle of a retail lending
application scorecard: data preparation, variable selection, model development, score scaling,
validation, monitoring, and basic regulatory (IFRS 9) support — the core toolkit of a Quantitative
/ Credit Risk Analyst role.

## Overview

This project builds a real, working credit scorecard on genuine bank loan-application data, covering
the same process a retail/MFB/fintech lender would run when developing an application scorecard for a
new lending product:

**Data QC → EDA & Variable Selection (WOE/IV) → Logistic Regression + Decision Tree →
Score Scaling → AUC/Gini/KS Validation → Score Cut-off Strategy → PSI Monitoring →
Basic IFRS 9/ECL → Model Documentation**

All core techniques (WOE, IV, KS, PSI, PDO score scaling) are implemented from scratch in Python,
since these are scorecard-specific methods that aren't available in standard ML libraries like
scikit-learn.

## Dataset

**Source:** [UCI Statlog (German Credit Data)](https://archive.ics.uci.edu/dataset/144/statlog+german+credit+data)
Donated by Prof. Hans Hofmann, Institut für Statistik und Ökonometrie, Universität Hamburg (1994).
Citation: Hofmann, H. (1994). *Statlog (German Credit Data)* [Dataset]. UCI Machine Learning Repository.
[https://doi.org/10.24432/C5NC77](https://doi.org/10.24432/C5NC77)

- 1,000 real bank loan applications, 20 attributes + binary outcome (Good/Bad credit risk)
- Observed bad rate in this dataset: 30% (a known, fixed characteristic of this academic benchmark —
  see *Limitations* below; real lending portfolios typically run far lower)
- Raw categorical fields are coded (e.g. `A11`, `A34`) per the dataset's official codebook
  (`german.doc`) and decoded into readable labels as part of the data preparation step

No proprietary or employer loan-book data is used anywhere in this project — this is publicly
available, licensed data (CC BY 4.0).

## Methodology

| Step | What it does |
|---|---|
| **1. Data Preparation** | Load raw coded data, decode attribute codes, run data quality checks (missing values, duplicates), produce summary statistics |
| **2. EDA & Variable Selection** | Bin numeric variables into quantiles, compute Weight of Evidence (WOE) and Information Value (IV) per variable, select variables with IV > 0.02 |
| **3. Model Development** | WOE-transform selected variables, fit a Logistic Regression scorecard model (primary) and a Decision Tree (benchmark) |
| **4. Score Scaling** | Convert model output (probability of default) into a points-based score using standard PDO (Points to Double the Odds) scaling |
| **5. Model Evaluation** | Validate discriminatory power using AUC, Gini, and the KS statistic |
| **6. Score Cut-off Strategy** | Build an approval-rate vs. bad-rate tradeoff curve to support origination cutoff decisions |
| **7. Monitoring (PSI)** | Calculate the Population Stability Index to detect drift between a reference population and a new scoring batch |
| **8. Basic IFRS 9 / ECL** | Calculate 12-month Expected Credit Loss (PD × LGD × EAD) at portfolio and segment level |
| **9. Model Documentation** | Produce a condensed model documentation summary (methodology, assumptions, monitoring plan) |
| **10. Reusable Toolkit** | Package the WOE/IV, KS, PSI, and score-scaling logic into standalone, reusable functions |

## Key Results

| Metric | Result | Industry Rule of Thumb |
|---|---|---|
| AUC | 0.824 | 0.70–0.80 = decent, >0.80 = strong |
| Gini | 0.648 | Derived as 2×AUC − 1 |
| KS | 55.8% | >40% generally considered a strong scorecard |
| Variables selected | 15 of 20 (IV > 0.02) | — |
| Train / test split | 750 / 250, stratified | — |
| PSI (simulated monitoring batch) | 0.05 | <0.10 = stable population |
| 12-month portfolio ECL rate | 16.2% (test book) | LGD assumed flat at 45% |

The model performs above the "strong scorecard" threshold on all three discrimination metrics
(AUC, Gini, KS) on out-of-sample data.

## What This Project Demonstrates

- **Weight of Evidence / Information Value** built from scratch — the standard scorecard variable
  selection technique, not available in scikit-learn
- **Leakage screening** — the top variable (`checking_account_status`, IV = 0.666) exceeds the
  typical "check for leakage" threshold (IV > 0.5); the notebook documents the specific checks run
  (bad-rate monotonicity, adequate bin sample sizes, timing of data availability) before retaining it
- **PDO score scaling** — converting a raw probability into an actual points-based score, the format
  a loan officer or credit policy team would actually use
- **Full validation suite** — AUC, Gini, KS, plus a KS chart presented the way a model validation
  report would show it
- **Score cut-off / approval strategy analysis** — the approval-rate vs. bad-rate tradeoff a credit
  policy committee reviews before setting an origination cutoff
- **Population Stability Index (PSI)** — monitoring methodology for detecting portfolio drift after
  a model goes live
- **Basic IFRS 9 / Expected Credit Loss support** — the PD × LGD × EAD calculation underlying
  regulatory credit-loss reporting
- **Model documentation** — a condensed but realistic methodology write-up, generated
  programmatically from the actual model run so it can't drift out of sync with the results

## Repository Structure

```
.
├── README.md
└── Application_Scorecard_Project.ipynb   # Full analysis, code, and outputs
```

## How to Run

```bash
pip install numpy pandas matplotlib scikit-learn jupyter
jupyter notebook Application_Scorecard_Project.ipynb
```

The notebook downloads the dataset directly from a public source at runtime — no manual data
download required.

## Tech Stack

Python · pandas · NumPy · scikit-learn (Logistic Regression, Decision Tree) · Matplotlib

## Limitations

- The dataset's 30% bad rate is a fixed characteristic of this academic benchmark, not a real
  portfolio's natural default rate — a live MFB/fintech book would typically run far lower
- No application-date field exists in the source data, so the PSI monitoring section uses a
  deliberately reweighted resample of the same population to demonstrate the methodology, not a
  genuine second time period
- LGD is assumed flat at 45% for the ECL illustration; a real deployment would need a dedicated
  LGD model or segment-level assumptions
- This is a demonstration project on a small (n=1,000), 1994 academic dataset — a production
  deployment would require revalidation on a live, proprietary loan book

## License

Code in this repository is provided as-is for portfolio/demonstration purposes. The underlying
dataset is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/legalcode) and
must be credited to Prof. Hans Hofmann per the citation above.
