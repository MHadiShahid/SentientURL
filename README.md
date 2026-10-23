# SentinelURL — Phishing URL Detection Using Machine Learning

**A comparative analysis of machine learning models for identifying phishing URLs using structural and lexical URL features.**

SentinelURL is a machine learning project that investigates multiple approaches to phishing URL detection, maliciousness scoring, threat clustering, and sequential decision-making. Using the PhiUSIIL Phishing URL Dataset, the project implements and compares eight models across supervised learning, regression, unsupervised learning, reinforcement learning, and ensemble methods.

## Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Dataset](#dataset)
- [Technology Stack](#technology-stack)
- [Data Preprocessing and Feature Engineering](#data-preprocessing-and-feature-engineering)
- [Machine Learning Models](#machine-learning-models)
- [Results and Evaluation](#results-and-evaluation)
- [Comparative Analysis](#comparative-analysis)
- [Key Findings](#key-findings)
- [Limitations](#limitations)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Team Members](#team-members)
- [References](#references)

## Overview

Phishing attacks use deceptive URLs to trick users into visiting malicious websites, potentially exposing credentials, financial information, and other sensitive data. Traditional blacklist-based approaches can struggle with newly generated or short-lived phishing domains.

SentinelURL explores a feature-based machine learning approach that analyzes URL characteristics to distinguish phishing URLs from legitimate ones. Instead of relying solely on historical blacklist matches, the project evaluates models using structural, lexical, domain-related, and page-related features.

The project also explores complementary approaches, including continuous risk scoring, unsupervised threat clustering, and reinforcement learning for sequential URL inspection.

> **Project status:** Academic machine learning project. The models were evaluated on a labeled dataset; a live URL-scanning service or production deployment is not claimed.

## Objectives

- Develop machine learning models for phishing URL classification.
- Perform exploratory data analysis and feature engineering.
- Compare traditional classifiers with ensemble learning methods.
- Explore regression-based maliciousness scoring.
- Investigate unsupervised clustering of URLs into threat categories.
- Implement a reinforcement learning agent for sequential URL inspection.
- Evaluate model performance using appropriate metrics and compare potential use cases.

## Dataset

The project uses the **PhiUSIIL Phishing URL Dataset** from the UCI Machine Learning Repository.

| Property | Value |
|---|---:|
| Original URL instances | 235,795 |
| Phishing URLs | 134,850 (57.2%) |
| Legitimate URLs | 100,945 (42.8%) |
| Extracted features | 51 |
| Missing values reported | 0 |
| Duplicate rows removed | 808 |
| Rows after deduplication | 234,987 |

### Feature Categories

The dataset includes features representing different aspects of a URL and its associated webpage:

- **URL structure:** URL length, dots, hyphens, and slashes.
- **Domain information:** HTTPS status, IP-address usage, and top-level-domain legitimacy probability.
- **Character statistics:** Letter ratio, digit ratio, and obfuscation ratio.
- **Page content:** Page title, favicon presence, and JavaScript-related features.
- **Derived features:** Dot density and digit density.

**Dataset source:** [PhiUSIIL Phishing URL Dataset — UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+dataset)

## Technology Stack

- **Language:** Python
- **Development environment:** Google Colab / Jupyter Notebook
- **Data processing:** Pandas, NumPy
- **Visualization:** Matplotlib, Seaborn
- **Machine learning:** Scikit-learn
- **Gradient boosting:** XGBoost

The notebook's actual imports should be used to create the final dependency list.

## Data Preprocessing and Feature Engineering

The report describes the following preprocessing pipeline:

1. **Data loading:** Loaded the dataset and examined its 51 numeric columns.
2. **Missing-value analysis:** Checked for missing values; none were reported.
3. **Duplicate removal:** Removed 808 duplicate rows.
4. **Stratified sampling:** Preserved the class distribution during sampling.
5. **Variance filtering:** Applied `VarianceThreshold(0.01)`, reducing the feature count from 51 to 44.
6. **Correlation filtering:** Removed highly correlated features using a correlation threshold of 0.95, reducing the count from 44 to 43.
7. **Feature engineering:** Created `DotDensity` and `DigitDensity` features.
8. **Train-test splitting:** Applied a stratified split as described in the report.
9. **Feature scaling:** Used `StandardScaler`, fitted on the training data.
10. **Dimensionality reduction:** Used PCA with two components for visualization and 33 components for clustering and reinforcement learning.

The engineered features were defined as:

- `DotDensity = NoOfSubDomain / (URLLength + 1)`
- `DigitDensity = NoOfDegitsInURL / (URLLength + 1)`

The final reported feature count was 45.

**Reproducibility note:** The report describes an 80/10 split and gives 23,999 training rows and 6,000 test rows, which do not account for all 234,987 deduplicated rows. The exact sampling and splitting implementation should be checked in the notebook before treating these counts as a complete description of the training procedure.

## Machine Learning Models

SentinelURL evaluates eight models across different learning paradigms.

### 1. K-Nearest Neighbors (KNN)

A supervised classification model that predicts a URL's class based on its nearest training examples.

- Neighbors: `7`
- Distance metric: Euclidean
- Search algorithm: Ball tree
- Primary consideration: Prediction cost can increase with training-set size.

### 2. Support Vector Machine (SVM)

A linear support vector classifier used to separate phishing URLs from legitimate URLs. A calibrated classifier wrapper was used to support probability estimation.

- Classifier: `LinearSVC`
- Regularization parameter: `C = 1.0`
- Maximum iterations: `2000`

### 3. Decision Tree

An interpretable classifier that learns decision rules from URL features.

- Criterion: Gini impurity
- Maximum depth: `10`
- Minimum samples per leaf: `20`
- Important features reported: `URLSimilarityIndex`, `TLDLegitimateProb`, and `CharContinuationRate`

### 4. Linear Regression

A regression approach that estimates a continuous maliciousness score between 0 and 1, based on an engineered target.

The report defines the target using weighted URL characteristics, including URL length, obfuscated characters, IP-address usage, subdomain count, and digit ratio.

This score is an experimental risk-scoring approach and should not be interpreted as a calibrated probability of maliciousness without additional validation.

### 5. K-Means Clustering

An unsupervised learning approach used to group URLs without using their class labels during clustering.

- Input: 33 PCA components
- Number of clusters: `2`
- Selection approach: Elbow method and silhouette score

The resulting clusters were subsequently compared with the ground-truth labels to examine their relationship with phishing activity.

### 6. Q-Learning

A reinforcement learning agent was implemented using a custom `URLInspectorEnv` environment.

The agent selects among three actions:

- `0` — Inspect more
- `1` — Flag as phishing
- `2` — Clear as legitimate

The state space contains 36 states based on URL-length buckets, HTTPS status, subdomain buckets, and IP-address usage.

**Reported parameters:**

- Learning rate: `0.15`
- Discount factor: `0.90`
- Exploration rate decay: `1.0` to `0.05`
- Training episodes: `2,000`

The reward scheme assigns positive rewards for correct decisions, negative rewards for incorrect decisions, and a small penalty for additional inspection steps.

### 7. Random Forest

A bagging-based ensemble classifier that aggregates predictions from multiple decision trees.

- Number of estimators: `50`
- Maximum depth: `10`
- Minimum samples per leaf: `10`
- Parallel processing: `n_jobs = -1`

### 8. XGBoost

A gradient-boosted tree classifier that builds trees sequentially to improve on errors made by earlier trees.

- Number of estimators: `100`
- Maximum depth: `4`
- Learning rate: `0.1`
- Subsample: `0.8`
- Tree method: `hist`

## Results and Evaluation

The following results are reproduced from the project report.

### Classification Performance

| Model | Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|
| KNN | 99.78% | 99.65% | 99.97% | 99.81% |
| SVM | 99.85% | 99.94% | 100.00% | 99.97% |
| Decision Tree | 99.74% | 99.71% | 99.59% | 99.65% |
| Random Forest | 100.00% | 100.00% | 100.00% | 100.00% |
| XGBoost | 100.00% | 100.00% | 100.00% | 100.00% |

The report also records ROC-AUC scores of `1.000` for the classification models other than KNN, which achieved `0.999`.

### Regression, Clustering, and Reinforcement Learning

| Model | Evaluation Metric | Reported Result |
|---|---|---:|
| Linear Regression | RMSE | 0.0219 |
| Linear Regression | MAE | 0.0094 |
| Linear Regression | R² | 0.3935 |
| K-Means | Silhouette score | 0.2022 |
| K-Means | Davies–Bouldin index | 1.8731 |
| Q-Learning | Accuracy | 81.90% |
| Q-Learning | F1-score | 85.41% |
| Q-Learning | Average reward | 6.37 |

The clustering analysis reported that one cluster contained 99.41% phishing URLs, while the other was predominantly legitimate.

### Interpreting the Results

Random Forest and XGBoost achieved the strongest reported classification metrics. SVM also produced near-perfect results, while KNN and Decision Tree performed competitively.

The regression model provides a continuous score rather than a binary class prediction. K-Means explores the structure of URL features without supervised labels, while Q-Learning investigates sequential inspection decisions.

These results reflect the reported experimental evaluation. Perfect classification scores should be interpreted cautiously until the evaluation pipeline has been checked for data leakage, preprocessing leakage, duplicate overlap, and independent test-set contamination.

## Comparative Analysis

| Model | Main Strength | Main Limitation | Potential Use Case |
|---|---|---|---|
| KNN | Simple, competitive classification | Inference cost can grow with dataset size | Smaller datasets |
| SVM | Strong classification performance | Limited interpretability | Feature-based classification |
| Decision Tree | Interpretable decision rules | Can overfit if insufficiently constrained | Security analysis and audits |
| Random Forest | Ensemble robustness | Less interpretable than a single tree | General-purpose classification |
| XGBoost | Strong reported predictive performance | Requires tuning and validation | Candidate primary classifier |
| Linear Regression | Continuous score estimation | Limited fit to the engineered target | Experimental risk scoring |
| K-Means | Unsupervised grouping | Clusters do not inherently represent threat labels | Exploratory threat categorization |
| Q-Learning | Sequential decision-making | Lower reported accuracy and dependence on reward design | Experimental sequential inspection |

## Key Findings

- **Ensemble methods performed strongly:** Random Forest and XGBoost achieved 100% reported accuracy on the evaluation data.
- **SVM was competitive:** It achieved 99.85% accuracy and a reported F1-score of 99.97%.
- **Interpretability remains important:** A Decision Tree provides more accessible decision rules for analysis than a complex ensemble.
- **Unsupervised learning revealed structure:** K-Means produced two clusters with different phishing concentrations.
- **Regression offered an alternative output:** The engineered maliciousness score enabled continuous risk estimation.
- **Reinforcement learning explored a different problem formulation:** Q-Learning modeled sequential inspection rather than ordinary one-step classification.

Based on the reported results, XGBoost is a candidate for further deployment-oriented evaluation, while Decision Tree offers a more interpretable alternative. Neither should be considered production-ready solely on the basis of the reported scores.

## Limitations and Future Work

Potential next steps include:

- Validate the train-test split and confirm that no duplicate or related examples leak between partitions.
- Fit every preprocessing operation, including feature selection and PCA, exclusively on training data when evaluating supervised generalization.
- Evaluate the final model on a separate, independently collected dataset.
- Measure precision, recall, false-positive rate, and false-negative rate under realistic phishing-detection conditions.
- Test performance on URLs from domains and time periods not represented in the training data.
- Compare model latency, memory usage, and model size under consistent conditions.
- Validate the regression score against an explicitly defined and independently justified risk target.
- Evaluate the reinforcement learning agent against simple baseline policies.
- Develop an inference interface or web application if real-time scanning is a future goal.

These steps would help establish whether the reported results generalize beyond the experimental dataset.

## Getting Started

### Prerequisites

- Python
- Google Colab or Jupyter Notebook
- The dataset and libraries required by the notebook

### Run in Google Colab

1. Open the notebook in this repository.
2. Open it in Google Colab.
3. Download the PhiUSIIL dataset from the UCI Machine Learning Repository or from here.
4. Upload the dataset to the notebook.
5. Execute the cells in order to reproduce the preprocessing, training, and evaluation steps.
