# Pulsar Classification with Supervised Learning

**A comparative study of seven supervised classification methods on the imbalanced HTRU2 dataset.**

**[Read the full HTML report](https://simonedibella.github.io/htru2-pulsar-classification/pulsar_classification.html)**

**Author:** Simone Di Bella  
**Language:** R · **Analysis format:** R Markdown

## Overview

This project studies binary classification of pulsar candidates from the [High Time Resolution Universe (HTRU2) dataset](https://archive.ics.uci.edu/dataset/372/htru2). Real pulsars account for approximately **9.2%** of the observations, so model assessment focuses on the minority class rather than overall accuracy alone.

The analysis distinguishes **ranking performance** (ROC-AUC and PR-AUC) from **operational classification decisions**. Hyperparameters are selected by cross-validated PR-AUC. Separately, decision thresholds are chosen from out-of-fold (OOF) training predictions to maximize specificity subject to an empirical sensitivity target of at least 90%; the thresholds are then frozen for the independent test evaluation.

This is an **academic supervised-learning project**, not a novel classification algorithm or a peer-reviewed research publication.

## Data

| Property | Value |
| --- | ---: |
| Candidate observations | 17,898 |
| Predictors | 8 numeric features |
| Non-pulsars | 16,259 |
| Pulsars | 1,639 (9.16%) |
| Target | Binary: pulsar vs. non-pulsar |

The eight predictors summarize the *integrated pulse profile* and *DM–SNR curve* (mean, standard deviation, excess kurtosis, and skewness for each).

## Methods

The same high-level train/test strategy is used across **seven classifiers**:

- Linear Discriminant Analysis (LDA)
- Quadratic Discriminant Analysis (QDA)
- K-Nearest Neighbors (KNN)
- Classification Tree
- Bagging
- Random Forest
- Gradient Boosting

The data are divided into a **stratified 80% training / 20% test split**, with **10-fold cross-validation** inside training. The held-out test set is used for final evaluation only. For methods with tuned hyperparameters, the primary selection metric is **PR-AUC**.

### Operating-point selection

The sensitivity-constrained operating point is selected using OOF training predictions:

> Maximize specificity, subject to empirical sensitivity ≥ 90%.

This is a threshold selection rule rather than a guarantee on new observations. No method reached 90% sensitivity in the final test evaluation under its training-selected threshold.

## Selected results

### Ranking performance on the independent test set

| Model | ROC-AUC | PR-AUC |
| --- | ---: | ---: |
| LDA | 0.9725 | 0.9127 |
| QDA | 0.9611 | 0.8771 |
| KNN | 0.9611 | 0.9012 |
| Classification Tree | 0.9602 | 0.8740 |
| Bagging | 0.9634 | 0.9092 |
| Random Forest | 0.9660 | 0.9126 |
| **Gradient Boosting** | **0.9766** | **0.9153** |

Gradient Boosting has the highest observed test ROC-AUC and PR-AUC in this comparison, but differences in PR-AUC among some methods are small. No formal test of differences between AUC estimates is conducted.

### Threshold-adjusted test performance

| Model | Sensitivity | FPR | Precision |
| --- | ---: | ---: | ---: |
| LDA | 0.8960 | 0.0188 | 0.8277 |
| QDA | 0.8930 | 0.0498 | 0.6432 |
| KNN | **0.8991** | 0.0234 | 0.7946 |
| Classification Tree | 0.8838 | 0.0305 | 0.7448 |
| Bagging | 0.8838 | 0.0221 | 0.8006 |
| Random Forest | 0.8869 | 0.0169 | 0.8406 |
| Gradient Boosting | 0.8746 | **0.0123** | **0.8773** |

KNN comes closest to the 90% sensitivity target, whereas Gradient Boosting has the lowest FPR at the selected test operating point. These are different operational trade-offs, not a universal winner.

## Repository contents

```text
.
├── README.md
├── pulsar_classification.Rmd    # Original analysis source (renamed)
├── pulsar_classification.html   # Self-contained rendered report (renamed)
├── data/
│   ├── HTRU_2.csv
│   ├── HTRU2_original_readme.txt
│   └── README.md
└── .gitignore
```

## Running the analysis

1. Install **R** and **R Markdown** (and Pandoc / RStudio as applicable).
2. Install the packages used by the analysis:

   ```r
   install.packages(c(
     "rmarkdown", "knitr", "caret", "MASS", "class", "tree", "randomForest",
     "gbm", "pROC", "PRROC", "corrplot", "e1071", "ggplot2"
   ))
   ```

3. In an R session with the **repository root as the working directory**, render the document:

   ```r
   rmarkdown::render("pulsar_classification.Rmd")
   ```

The script reads `data/HTRU_2.csv` relative to the repository root. The report includes computationally expensive cross-validation and model tuning; a full fresh execution may take substantial time and requires a suitable R environment.

**Reproducibility status:** The input file has been checked against the dataset description (dimensions, class frequencies, missing values). The complete model-training pipeline has **not yet been rerun independently** in a fresh R environment. The package versions are not pinned in this draft.

## Limitations

- Performance is estimated using a single train/test split.
- Hyperparameter selection and OOF performance reporting are based on the same CV process; post-selection OOF metrics can be optimistic. A fully nested evaluation is not implemented.
- No confidence intervals, formal between-model comparisons, or probability calibration analysis are provided.
- The sensitivity threshold target is an illustrative operational choice, not an optimum for every application.
- Generalization beyond HTRU2-like data has not been established.

## Dataset attribution and license

Dataset: **Robert J. Lyon, HTRU2**, UCI Machine Learning Repository, DOI: [10.24432/C5DK6R](https://doi.org/10.24432/C5DK6R).

The HTRU2 dataset is distributed by UCI under the [Creative Commons Attribution 4.0 International license](https://creativecommons.org/licenses/by/4.0/). The unmodified original CSV and dataset README are included for reproducibility, with attribution to the dataset creator. The analyses and code are separate from the third-party dataset; **no independent license for the analysis code has yet been selected**.

Related publication: Lyon, R. J., Stappers, B. W., Cooper, S., Brooke, J. M., & Knowles, J. D. (2016). *Fifty years of pulsar candidate selection: From simple filters to a new principled real-time classification approach.* Monthly Notices of the Royal Astronomical Society, 459(1), 1104–1123. [DOI: 10.1093/mnras/stw656](https://doi.org/10.1093/mnras/stw656).
