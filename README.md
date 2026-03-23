# Metabolomics Sex Classification using Machine Learning

This project is part of the course Statistical Learning and focuses on reproducing a complete and statistically rigorous metabolomics analysis pipeline using the dataset from Fan et al. (2018):
*Sex-associated differences in baseline urinary metabolites of healthy adults* (Scientific Reports, 2018).

The analysis is based on the supplementary file MOESM2 (ESM), which contains GC–TOF–MS metabolomic measurements for 120 urine samples (60 males, 60 females).

**Objective:** classify biological sex using metabolomics data and identify relevant metabolic patterns using statistical learning methods.

---

## Context

This work was developed as part of a group project in the Statistical Learning course.
This repository highlights my contribution and understanding of the full analysis pipeline.

---

## 1. Import Data

- Loaded GC–TOF–MS metabolomics dataset from `41598_2018_29592_MOESM2_ESM.xlsx`.
- Extracted sex labels (Male/Female) from metadata rows.
- Removed metadata columns and retained only metabolite intensities.
- Ensured all metabolite values were numeric and stored in matrix format.
- Final dataset contains 120 samples (60 males, 60 females) and several hundred metabolites.

---

## 2. Exploratory Data Analysis (EDA)

### 2.1 Summary Statistics
- Inspected metabolite distributions (mean, median, range).
- Counted zero values and missing values across the dataset.

### 2.2 Distribution Assessment
- Raw metabolite intensities showed strong right-skewness.
- QQ-plots confirmed non-Gaussian distributions.

### 2.3 Zero Handling
- Zero values replaced by half of the minimum non-zero value per metabolite, allowing log10 transformation without introducing infinite values.

### 2.4 Log Transformation
- Applied log10(x) transformation to stabilize variance and reduce skewness.

### 2.5 Autoscaling
- All metabolites mean-centered and scaled to unit variance to ensure comparability in multivariate analyses.

### 2.6 Outlier Detection
- Performed PCA combined with Hotelling’s T² ellipse (95% CI).
- Detected 9 potential outliers.
- No samples were removed to avoid biological bias and to remain consistent with the original study, which analyzed all subjects.

---

## 3. Principal Component Analysis (PCA)

### 3.1 PCA Computation
- Performed PCA on log-transformed and autoscaled data with samples as rows and metabolites as columns.

### 3.2 PCA Score Plot
- PC1 vs PC2 revealed moderate sex-related trends.
- Outliers highlighted using Mahalanobis distance.

### 3.3 Scree Plot
- Typical metabolomics behavior observed.
- PC1–PC2 explain approximately 20–30% of the total variance.

### 3.4 Interpretation
- Although unsupervised, PCA already suggests subtle sex-related structure.
- Variability is considered biologically meaningful rather than noise.

---

## 4. Supervised Analysis (PLS-DA & Validation)

### 4.1 Train/Test Split
- Performed a stratified 70/30 split preserving Male/Female proportions.
- Training set used for model building and feature selection.
- Test set reserved exclusively for external validation.

### 4.2 Initial PLS-DA Model
- Trained an initial PLS-DA model with 10 latent variables.
- Score plot showed clear discrimination between males and females.

### 4.3 Internal Validation (5-Fold Cross-Validation)
- Evaluated models with 1–5 latent variables.
- AUC-ROC used as the primary performance metric.
- Optimal number of components: 3 (highest mean AUC and best generalization).

### 4.4 Final PLS-DA Model
- Retrained the PLS-DA model using 3 latent variables.
- Reduced model complexity while maintaining discriminatory power.

### 4.5 External Validation (Test Set)
- Predicted sex labels for the held-out test set.
- Computed:
  - Confusion matrix
  - Accuracy
  - Balanced Error Rate (BER)
  - ROC curve and AUC
- Results confirmed a strong and reproducible sex-related metabolic signal.

---

## 5. Univariate Analysis

### 5.1 Statistical Testing
- Applied Welch’s t-tests to each metabolite on the training set only.
- Corrected for multiple testing using FDR (Benjamini–Hochberg).

### 5.2 Effect Size
- Computed mean differences (Male – Female) for each metabolite.

### 5.3 Volcano Plot
- Generated volcano plot displaying:
  - Effect size vs statistical significance
  - Metabolites with FDR < 0.05
  - Direction of sex-associated changes

### 5.4 Interpretation
- Identified 9 statistically significant metabolites.
- These features represent candidate biomarkers and inputs for subsequent feature selection.

---

## 6. Feature Selection

### 6.1 VIP Score Ranking
- Computed Variable Importance in Projection (VIP) scores from the PLS-DA model.
- Ranked metabolites according to their contribution to class discrimination.
- Visualized the top-ranked metabolites and applied the conventional VIP > 1 threshold.

### 6.2 Recursive Feature Elimination (RFE)
- Implemented RFE using a k-NN classifier with repeated 5-fold cross-validation.
- Evaluated classification accuracy across different feature subset sizes.
- Selected the optimal subset of metabolites maximizing cross-validated accuracy.

---

## 7. Final Model with Optimal Features

### 7.1 Model Training
- Subset the training and test sets to the optimal RFE-selected features.
- Trained a final PLS-DA model using:
  - Optimal number of latent variables (3)
  - Optimal feature subset

### 7.2 External Validation
- Evaluated the final model on the independent test set.
- Computed:
  - Confusion matrix
  - Accuracy
  - Balanced Error Rate (BER)
  - Cohen’s Kappa
  - ROC curve and AUC

---

## 8. Figure of Merit Uncertainty

### 8.1 Bootstrap Analysis
- Quantified uncertainty of performance metrics using nonparametric bootstrap resampling (n = 1000) applied to the test set.
- Metrics evaluated:
  - Accuracy
  - BER
  - Cohen’s Kappa
  - Sensitivity and specificity for both sexes
  - AUC-ROC

### 8.2 Confidence Intervals
- Reported mean estimates and 95% bootstrap confidence intervals for all figures of merit.
- Results demonstrate robust performance and stable generalization of the final model.
