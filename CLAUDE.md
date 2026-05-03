# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Binary classification task: identifying Head and Neck Squamous Cell Carcinoma (HNSC) from RNA-seq gene expression data using the TCGA-HNSC dataset.

- **Dataset:** 563 patient samples × 35,957 gene features, severely imbalanced (92% cancerous, 8% healthy)
- **Data files:** `G2-TCGA-HNSC_EX.csv` (expression matrix) and `G2-TCGA-HNSC_Label.csv` (labels) — excluded from git via `.gitignore`

## Running the Project

All work lives in `project.ipynb`. Run it with:

```bash
jupyter notebook project.ipynb
```

Cells must be run sequentially — each builds on the previous. Required packages: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`.

## Architecture & Pipeline

The notebook is organized into four sequential phases:

1. **EDA** — Class distribution, gene correlation analysis, PCA visualization. Key finding: negatively correlated genes (tumor suppressors silenced) are more diagnostic than overexpressed ones.

2. **Preprocessing pipeline** (order matters):
   - Log1p transform (normalize right-skewed raw counts)
   - Variance threshold filter: 35,957 → 28,950 genes
   - Correlation-based feature selection: → top 500 genes by absolute correlation to label
   - Stratified 80/20 train/test split (preserves 92/8 imbalance)
   - StandardScaler fit on train, applied to both splits

3. **Model training** — Currently Logistic Regression (`penalty='l2'`, `solver='lbfgs'`, `C=1.0`, `max_iter=1000`). Evaluation uses F1-score and ROC-AUC (not accuracy) given class imbalance.

4. **Evaluation** — Confusion matrix, classification report, ROC curve.

## Key Design Decisions

- **Stratified split** is required to preserve the 92/8 class ratio in both train and test sets.
- **Scaler must be fit only on training data** to prevent data leakage.
- Feature selection is done before splitting in the current implementation — worth flagging if refactoring.
- Perfect test performance (100% accuracy, AUC=1.0) is achieved with current Logistic Regression; be cautious of overfitting if adding more complex models.
