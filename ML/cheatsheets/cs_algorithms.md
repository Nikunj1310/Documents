# Cheat Sheet — Algorithm Selection Guide

> When to use which algorithm. No hedging — direct recommendations.

---

## By Problem Type

### Tabular Data (Structured)

```
Regression:
  Default start:      XGBoost / LightGBM
  Interpretability:   Linear Regression (after feature engineering)
  Small dataset:      Ridge / Lasso
  Noisy features:     Random Forest (less sensitive)

Classification:
  Default start:      XGBoost / LightGBM
  Interpretability:   Logistic Regression or Decision Tree (depth ≤ 5)
  Small dataset:      Logistic Regression + careful regularization
  High cardinality:   CatBoost or LightGBM with target encoding
  Imbalanced:         XGBoost with scale_pos_weight, or class_weight='balanced'
  Very large (>10M):  LightGBM (fastest on big data)

Always:
  1. Start with a DummyClassifier/DummyRegressor baseline
  2. Then Logistic Regression / Linear Regression as second baseline
  3. Then tree ensemble
  4. Feature engineering usually beats algorithm choice
```

### Image Data

```
< 1000 training images:   Transfer learning (freeze backbone, train head only)
1k–50k images:            Transfer learning (fine-tune last 2+ blocks)
> 50k images:             Train from scratch or full fine-tune
  Architecture choices:
    General purpose:      ResNet50 / EfficientNetB3 (balanced accuracy/speed)
    Best accuracy:        ConvNeXt / EfficientNetV2
    Speed-critical:       MobileNetV3 / EfficientNet-Lite
    Detection:            YOLOv8 / DETR
    Segmentation:         U-Net / SAM (Segment Anything)
```

### Text / NLP Data

```
< 1000 examples:          TF-IDF + Logistic Regression (surprisingly good)
1k–10k examples:          Fine-tune small BERT (bert-base) or DistilBERT
> 10k examples:           Fine-tune large BERT or domain-specific model
Generation tasks:         Fine-tune GPT-2 or use larger LLM with prompting
Multilingual:             XLM-RoBERTa
Domain-specific text:     Find a pretrained model for that domain (PubMedBERT, LegalBERT)

For production latency:
  Fastest inference:      DistilBERT (40% smaller, 60% faster, 97% of BERT accuracy)
  Very fast:              TF-IDF + linear model (microseconds)
```

### Sequence / Time Series

```
Tabular time series:      LightGBM with lag features + rolling statistics
Long sequences:           Transformer (if full context needed)
Short sequences (<100):   LSTM / GRU
Forecasting:              N-BEATS, TimesNet, or classical ARIMA/Prophet
Anomaly detection:        Isolation Forest, LSTM autoencoder
```

### Graphs

```
Node classification:      GCN / GraphSAGE (start here)
Link prediction:          GAE (Graph Autoencoder)
Graph classification:     GIN (Graph Isomorphism Network)
Large graphs (>1M nodes): GraphSAGE with neighbor sampling, or ClusterGCN
Heterogeneous graphs:     HAN / HGT
```

### Clustering

```
Known K, spherical:       K-Means
Unknown K:                DBSCAN (if density-based) or Agglomerative (hierarchical)
High-dimensional:         Reduce with PCA first, then K-Means
Soft/probabilistic:       Gaussian Mixture Model
Very large:               MiniBatchKMeans
```

### Dimensionality Reduction

```
For ML features:          PCA (fast, reproducible, linear)
For visualization:        UMAP > t-SNE (faster, preserves more structure)
Sparse data:              TruncatedSVD (like PCA but for sparse matrices)
Non-linear features:      UMAP or Autoencoder
```

### Reinforcement Learning

```
Discrete actions:         DQN, Double DQN, Dueling DQN
Continuous actions:       PPO (on-policy), SAC (off-policy, more sample-efficient)
Simple environments:      Q-Learning (tabular)
Atari / pixels:           DQN with CNN (use framestack + preprocessing)
Robotics / physics:       PPO or SAC with domain randomization
LLM alignment:            PPO with reward model (RLHF)
```

---

## By Constraint

### Speed (inference must be fast)

```
< 1ms:       Linear/logistic regression, naive Bayes, simple heuristic
< 10ms:      Gradient boosting (compiled, e.g., XGBoost), SVM (linear kernel)
< 100ms:     Random forest, small neural net, DistilBERT with batch_size=1
< 1s:        Large neural net, full BERT
```

### Small Data (< 1000 labeled examples)

```
Best choices:
  - Logistic Regression / SVM (generalizes better than trees with small data)
  - Random Forest (low variance via bagging)
  - Transfer learning (use pretrained features, train only a small head)
  - Data augmentation (especially for images)
  - k-NN (non-parametric, no overfitting risk for small k)
  
Avoid:
  - Gradient boosting from scratch (overfits easily)
  - Deep neural networks from scratch
  - Large parameter counts relative to data size
```

### Interpretability Required

```
Highest interpretability:
  Linear/Logistic Regression — coefficients directly give feature impact
  Single Decision Tree (depth ≤ 4) — trace every prediction

Good interpretability with post-hoc tools:
  Random Forest + SHAP
  XGBoost + SHAP
  Any model + LIME for local explanations

Use SHAP for:
  - Feature importance (global)
  - Individual prediction explanation (local)
  - Feature interaction detection

Use Grad-CAM for:
  - CNN image classification (which region mattered)
```

### Imbalanced Classes

```
Step 1: Use the right metric (F1, AUC, PR-AUC — NOT accuracy)
Step 2: Try these in order:
  1. class_weight='balanced' in the model
  2. Threshold tuning (ROC curve → pick optimal threshold)
  3. Oversample minority: SMOTE (but only in training set!)
  4. Undersample majority
  5. Cost-sensitive learning (scale loss by class frequency)
  6. Ensemble of classifiers trained on balanced sub-samples

Do NOT:
  - Oversample before splitting (causes leakage)
  - Report accuracy as your metric
```

---

## Quick Parameter Starting Points

### XGBoost / LightGBM
```python
# Safe starting point
n_estimators=500, learning_rate=0.05, max_depth=6,
subsample=0.8, colsample_bytree=0.8,
early_stopping_rounds=50

# For LightGBM: replace max_depth with num_leaves=31
```

### Neural Network (Tabular)
```python
# Architecture: input → [256 → 128 → 64] → output
# Optimizer: AdamW(lr=1e-3, weight_decay=1e-4)
# Batch size: 256-1024
# Dropout: 0.2-0.3 after each hidden layer
# BN before activation
```

### Fine-tuning Transformers
```python
# Learning rate: 2e-5 to 5e-5 for BERT-base
# Batch size: 16-32 (accumulate gradients if GPU is small)
# Epochs: 3-5 for most tasks
# Warmup: 6% of total steps
# Weight decay: 0.01
```

### Random Forest
```python
# n_estimators: 100-300 (more rarely hurts)
# max_features: 'sqrt' (classification), 'log2' or 1/3 (regression)
# min_samples_leaf: 1-5 (increase to reduce overfitting)
# Usually works well with defaults
```

### CNN (from scratch)
```python
# Architecture: Conv(32) → Conv(64) → Conv(128) → GAP → Dense → Softmax
# Optimizer: AdamW(lr=1e-3, weight_decay=1e-4) + OneCycleLR
# Augmentation: random crop + flip + color jitter (minimum)
# Batch norm before every activation
# Dropout(0.3-0.5) before final Dense
```

---

## Common Mistakes to Avoid

| Mistake | What to Do Instead |
|---------|-------------------|
| Scaling after split | Fit scaler on train only, transform both |
| Using accuracy for imbalanced data | Use F1, AUC, or PR-AUC |
| Tuning on test set | Use a validation set; touch test only once |
| Not having a baseline | Always start with DummyClassifier |
| Normalizing before feature selection | Normalize, then select, not the other way |
| Grid search before feature engineering | Engineer features first, then tune |
| Too complex model with small data | Use simpler model + regularization |
| Forgetting model.eval() at inference | Always call model.eval() before inference |
| Using MSE for skewed targets | Use log transform on target + MAE/Huber |
| Assuming more data always helps | With high bias, more data won't help — need better features or model |
