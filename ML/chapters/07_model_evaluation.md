# Chapter 7 — Model Evaluation & Metrics

> *A model is only as good as your ability to measure it. This chapter is what separates practitioners who ship reliable models from those who produce misleadingly optimistic results.*

---

## 7.1 The Evaluation Framework

Before training a single model, answer:
1. What metric aligns with the business/scientific goal?
2. How will you avoid data leakage in evaluation?
3. Is your data distribution representative of real-world deployment?

Never optimize for accuracy when accuracy is not what you care about.

---

## 7.2 Classification Metrics

### The Confusion Matrix

For binary classification:
```
                   Predicted Positive    Predicted Negative
Actual Positive:   True Positive (TP)    False Negative (FN)
Actual Negative:   False Positive (FP)   True Negative (TN)
```

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay

cm = confusion_matrix(y_true, y_pred)
disp = ConfusionMatrixDisplay(cm, display_labels=['Negative', 'Positive'])
disp.plot(cmap='Blues')
plt.title('Confusion Matrix')
plt.show()
```

### Core Metrics

**Accuracy:** `(TP + TN) / (TP + TN + FP + FN)`

Misleading when classes are imbalanced. If 99% are negative, predicting "always negative" gives 99% accuracy but is useless.

**Precision:** `TP / (TP + FP)` — "Of all predicted positives, how many are actually positive?"

High precision = few false alarms.

**Recall (Sensitivity):** `TP / (TP + FN)` — "Of all actual positives, how many did we catch?"

High recall = few missed cases.

**F1 Score:** Harmonic mean of precision and recall:
```
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

Use F1 when both false positives and false negatives matter equally.

**F-beta Score:** Weighted F score:
```
F_β = (1 + β²) × (Precision × Recall) / (β² × Precision + Recall)
```
- β > 1: emphasize recall (e.g., disease detection — missing cases is costly)
- β < 1: emphasize precision (e.g., spam filter — false alarms are costly)

```python
from sklearn.metrics import (accuracy_score, precision_score, recall_score,
                             f1_score, classification_report)

print(classification_report(y_true, y_pred, target_names=['neg', 'pos']))
```

### The Precision-Recall Tradeoff

There is always a tradeoff between precision and recall, controlled by the **classification threshold** (default 0.5).

- Higher threshold → higher precision, lower recall (conservative)
- Lower threshold → higher recall, lower precision (aggressive)

```python
from sklearn.metrics import precision_recall_curve
import matplotlib.pyplot as plt

probs = model.predict_proba(X_test)[:, 1]
precision, recall, thresholds = precision_recall_curve(y_test, probs)

plt.figure(figsize=(8, 6))
plt.plot(recall, precision, marker='.')
plt.xlabel('Recall')
plt.ylabel('Precision')
plt.title('Precision-Recall Curve')

# Show the effect of different thresholds
for thresh in [0.3, 0.5, 0.7]:
    preds = (probs >= thresh).astype(int)
    p = precision_score(y_test, preds)
    r = recall_score(y_test, preds)
    plt.scatter(r, p, zorder=5)
    plt.annotate(f'thresh={thresh}', (r, p))
plt.show()
```

**Average Precision (AP):** Area under the precision-recall curve. Useful for highly imbalanced datasets.

### ROC Curve and AUC

**ROC (Receiver Operating Characteristic):** plots True Positive Rate (Recall) vs. False Positive Rate across all thresholds.

```
TPR = TP / (TP + FN)   = Recall
FPR = FP / (FP + TN)   = 1 - Specificity
```

**AUC (Area Under the ROC Curve):**
- AUC = 1.0: perfect classifier
- AUC = 0.5: random (useless)
- AUC = 0.0: perfectly wrong (just flip predictions)

AUC has a beautiful probabilistic interpretation: the probability that a randomly chosen positive example ranks higher than a randomly chosen negative example.

```python
from sklearn.metrics import roc_curve, roc_auc_score

fpr, tpr, thresholds = roc_curve(y_test, probs)
auc = roc_auc_score(y_test, probs)

plt.figure(figsize=(6, 6))
plt.plot(fpr, tpr, label=f'AUC = {auc:.3f}')
plt.plot([0,1], [0,1], 'k--', label='Random')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate (Recall)')
plt.title('ROC Curve')
plt.legend()
plt.show()
```

**ROC vs PR curve:**
- Use ROC when classes are balanced
- Use PR when positive class is rare (PR more informative in that case)

### Multiclass Metrics

For K > 2 classes:

```python
# Averaging strategies:
# 'macro': average per-class (treats all classes equally, sensitive to rare classes)
# 'weighted': average weighted by class support (reflects real-world class balance)
# 'micro': aggregate TP/FP/FN then compute (identical to accuracy for balanced)

from sklearn.metrics import f1_score
f1_macro = f1_score(y_test, y_pred, average='macro')
f1_weighted = f1_score(y_test, y_pred, average='weighted')
```

**Cohen's Kappa:** Accounts for chance agreement. Better than accuracy for imbalanced multiclass:
```python
from sklearn.metrics import cohen_kappa_score
kappa = cohen_kappa_score(y_test, y_pred)
# kappa = 1: perfect, kappa = 0: chance, kappa < 0: worse than chance
```

---

## 7.3 Regression Metrics

**MAE (Mean Absolute Error):**
```
MAE = (1/n) Σ |ŷᵢ - yᵢ|
```
Robust to outliers. In the same units as y. Easier to interpret.

**MSE (Mean Squared Error):**
```
MSE = (1/n) Σ (ŷᵢ - yᵢ)²
```
Penalizes large errors more. Differentiable (good for optimization). Not in same units as y.

**RMSE (Root MSE):** `√MSE` — same units as y, emphasizes large errors.

**R² (Coefficient of Determination):**
```
R² = 1 - Σ(ŷᵢ - yᵢ)² / Σ(ȳ - yᵢ)²
```
How much variance does the model explain? R² = 1: perfect. R² = 0: same as predicting the mean. R² < 0: worse than the mean.

**MAPE (Mean Absolute Percentage Error):**
```
MAPE = (1/n) Σ |ŷᵢ - yᵢ| / |yᵢ|
```
Scale-free, but problematic when yᵢ ≈ 0.

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import numpy as np

mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, y_pred)
print(f"MAE: {mae:.3f} | RMSE: {rmse:.3f} | R²: {r2:.3f}")
```

**Choosing the right regression metric:**
- Target on absolute scale: RMSE or MAE
- Target varies in scale: MAPE or MSLE
- Comparing models to a baseline: R²
- Outliers present, don't want them to dominate: MAE or Huber

---

## 7.4 Cross-Validation Strategies

### k-Fold Cross-Validation

Split data into k folds. Train on k-1, validate on 1. Repeat k times.

```python
from sklearn.model_selection import KFold, StratifiedKFold, cross_val_score

# For regression
kf = KFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=kf, scoring='neg_root_mean_squared_error')
print(f"CV RMSE: {-scores.mean():.3f} ± {scores.std():.3f}")

# For classification (stratified = preserves class balance in each fold)
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=skf, scoring='roc_auc')
print(f"CV AUC: {scores.mean():.3f} ± {scores.std():.3f}")
```

### Time Series Cross-Validation

Regular k-Fold would leak future information into the past. Use TimeSeriesSplit:

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
for fold, (train_idx, val_idx) in enumerate(tscv.split(X)):
    X_train, X_val = X[train_idx], X[val_idx]
    y_train, y_val = y[train_idx], y[val_idx]
    # Train and evaluate...
```

Each fold: training set grows, validation is always in the future of training.

### Leave-One-Out CV (LOOCV)

k=n (one sample per fold). Unbiased but computationally expensive. Only for very small datasets.

### Repeated CV

Run k-Fold multiple times with different shuffles. More stable estimate:
```python
from sklearn.model_selection import RepeatedStratifiedKFold

rskf = RepeatedStratifiedKFold(n_splits=5, n_repeats=10, random_state=42)
scores = cross_val_score(model, X, y, cv=rskf, scoring='roc_auc')
```

---

## 7.5 Hyperparameter Tuning

### Grid Search

Exhaustive search over a specified grid:
```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'max_depth': [3, 5, 7, 10],
    'min_samples_leaf': [1, 5, 10],
    'max_features': ['sqrt', 'log2']
}

grid = GridSearchCV(
    estimator=RandomForestClassifier(n_estimators=100),
    param_grid=param_grid,
    cv=StratifiedKFold(5, shuffle=True, random_state=42),
    scoring='roc_auc',
    n_jobs=-1,
    verbose=1,
    refit=True  # refit best model on full train set
)
grid.fit(X_train, y_train)

print(f"Best params: {grid.best_params_}")
print(f"Best CV AUC: {grid.best_score_:.4f}")
print(f"Test AUC: {roc_auc_score(y_test, grid.predict_proba(X_test)[:,1]):.4f}")
```

**Problem:** Scales as O(product of grid sizes × CV folds). Quickly becomes intractable.

### Randomized Search

Sample n_iter combinations randomly. Often finds equally good results in a fraction of the time:
```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform, loguniform

param_dist = {
    'n_estimators': randint(50, 500),
    'max_depth': randint(3, 15),
    'min_samples_leaf': randint(1, 20),
    'learning_rate': loguniform(0.001, 0.3),  # log-uniform for learning rates
    'subsample': uniform(0.5, 0.5)
}

search = RandomizedSearchCV(
    estimator=XGBClassifier(),
    param_distributions=param_dist,
    n_iter=50,        # number of random combinations to try
    cv=5,
    scoring='roc_auc',
    n_jobs=-1,
    random_state=42
)
```

### Bayesian Optimization

The smartest approach: use the results of previous trials to choose the next hyperparameter combination to try. Builds a probabilistic model (surrogate) of the objective function.

```python
# Using Optuna (highly recommended)
import optuna
from optuna.integration import OptunaSearchCV

def objective(trial):
    params = {
        'max_depth': trial.suggest_int('max_depth', 3, 10),
        'learning_rate': trial.suggest_float('learning_rate', 1e-3, 0.3, log=True),
        'n_estimators': trial.suggest_int('n_estimators', 100, 1000),
        'subsample': trial.suggest_float('subsample', 0.5, 1.0),
        'colsample_bytree': trial.suggest_float('colsample_bytree', 0.5, 1.0),
    }
    model = xgb.XGBClassifier(**params, eval_metric='logloss', use_label_encoder=False)
    score = cross_val_score(model, X_train, y_train, cv=5, scoring='roc_auc').mean()
    return score

study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=100, n_jobs=-1)
print(f"Best value: {study.best_value:.4f}")
print(f"Best params: {study.best_params}")
```

---

## 7.6 Learning Curves

Diagnose bias vs. variance by plotting performance as a function of training set size.

```python
from sklearn.model_selection import learning_curve

train_sizes, train_scores, val_scores = learning_curve(
    estimator=model,
    X=X, y=y,
    train_sizes=np.linspace(0.1, 1.0, 10),
    cv=5,
    scoring='roc_auc',
    n_jobs=-1
)

train_mean = train_scores.mean(axis=1)
val_mean = val_scores.mean(axis=1)

plt.figure(figsize=(8, 6))
plt.plot(train_sizes, train_mean, 'o-', label='Training AUC')
plt.plot(train_sizes, val_mean, 's-', label='Validation AUC')
plt.fill_between(train_sizes, 
                 train_scores.mean(1)-train_scores.std(1),
                 train_scores.mean(1)+train_scores.std(1), alpha=0.1)
plt.fill_between(train_sizes, 
                 val_scores.mean(1)-val_scores.std(1),
                 val_scores.mean(1)+val_scores.std(1), alpha=0.1)
plt.xlabel('Training Set Size')
plt.ylabel('AUC')
plt.legend()
plt.title('Learning Curves')
plt.grid(True, alpha=0.3)
plt.show()
```

**Reading learning curves:**

| Pattern | Diagnosis | Solution |
|---------|-----------|----------|
| High train score, low val score, large gap | High variance (overfit) | More data, regularization, simpler model |
| Both low, small gap | High bias (underfit) | More complex model, more features |
| Val plateaus near train as size increases | Well-fitted | You've maximized what this model can do |
| Gap shrinks as size increases | Variance problem, more data helps | Collect more data |

---

## 7.7 Error Analysis

Don't just look at aggregate metrics — look at the mistakes.

```python
# Find misclassified examples
y_pred = model.predict(X_test)
wrong_idx = np.where(y_pred != y_test)[0]

# For text/tabular: inspect them
print(X_test_df.iloc[wrong_idx].head(20))

# For images: show the images
fig, axes = plt.subplots(4, 5, figsize=(12, 10))
for i, idx in enumerate(wrong_idx[:20]):
    axes[i//5, i%5].imshow(X_test[idx].reshape(28, 28), cmap='gray')
    axes[i//5, i%5].set_title(f"True: {y_test[idx]}, Pred: {y_pred[idx]}")
    axes[i//5, i%5].axis('off')
plt.tight_layout()
plt.show()
```

**Error analysis checklist:**
- Is there a pattern in the errors (specific classes, feature ranges)?
- Are the mistakes "hard" examples (humans would also struggle) or "easy" (suggests a bug)?
- Are errors concentrated in specific subgroups (fairness concern)?

---

## 7.8 Statistical Tests for Model Comparison

When comparing two models, their CV score difference may be due to chance.

**Corrected paired t-test (Dietterich, 1998):**
```python
from scipy import stats

scores_A = cross_val_score(model_A, X, y, cv=10, scoring='accuracy')
scores_B = cross_val_score(model_B, X, y, cv=10, scoring='accuracy')

t_stat, p_value = stats.ttest_rel(scores_A, scores_B)
print(f"t = {t_stat:.3f}, p = {p_value:.4f}")
if p_value < 0.05:
    print("Significant difference at 5% level")
```

**Wilcoxon signed-rank test (non-parametric alternative):**
```python
t_stat, p_value = stats.wilcoxon(scores_A, scores_B)
```

For a single meaningful comparison, require `p < 0.05`. For multiple comparisons, use Bonferroni correction or Benjamini-Hochberg FDR control.

---

## Exercises

1. **Imbalanced dataset:** Create a dataset with 95% class 0, 5% class 1. Train a logistic regression. What is the accuracy? What is the F1 score? What does this tell you about using accuracy for imbalanced datasets?

2. **Threshold optimization:** Using the precision-recall curve, find the optimal threshold that maximizes F1 score. Write a function that takes `y_true` and `y_prob` and returns the optimal threshold.

3. **CV strategy comparison:** On a time-series dataset (or simulate one with temporal dependence), compare random k-Fold CV vs. TimeSeriesSplit. Does the random CV give an optimistic estimate?

4. **Learning curve analysis:** Generate two datasets: one with 100 samples and one with 10,000 samples. For each, train a random forest and plot learning curves. At what sample size does the forest stop improving? Is bias or variance the bottleneck for each?

5. **Bayesian optimization vs grid search:** On a real dataset, run 50 iterations of Optuna and a full grid search with 50 combinations. Plot the best score found as a function of number of trials. Which finds a better result given the same budget?

---

## Further Reading

- *Machine Learning Yearning* — Andrew Ng (free PDF) — excellent practical evaluation guidance
- Kohavi (1995) "A Study of Cross-Validation and Bootstrap for Accuracy Estimation and Model Selection"
- *Evaluating Machine Learning Models* — Alice Zheng (O'Reilly)
