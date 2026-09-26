# Chapter 5 — Tree-Based Methods

> *Decision trees are the building block of the most powerful classical ML models. Random forests and gradient boosting win most tabular competitions. Know them from first principles.*

---

## 5.1 Decision Trees

### The Idea

A decision tree recursively splits the feature space into rectangular regions, assigning a prediction to each region.

```
                  Age > 35?
                 /          \
               No            Yes
        Income > 50k?    Income > 80k?
        /       \          /       \
       No       Yes       No       Yes
     DENY     APPROVE   DENY    APPROVE
```

Each **internal node** tests a feature. Each **leaf node** makes a prediction.

### How to Build a Tree (CART Algorithm)

At each node, find the best split: choose feature `j` and threshold `t` that maximally reduces impurity.

**For regression:** minimize MSE
```
Impurity = (1/nL) Σ_{i∈L} (yᵢ - ȳ_L)² + (1/nR) Σ_{i∈R} (yᵢ - ȳ_R)²
```

**For classification:** minimize Gini impurity or entropy

**Gini impurity:**
```
Gini = 1 - Σₖ pₖ²
```
where pₖ = fraction of class k in the node. Gini = 0 means pure node (all one class). Gini = 0.5 means maximum impurity (50/50 split for binary).

**Entropy (information gain):**
```
Entropy = -Σₖ pₖ log₂(pₖ)
Information Gain = Entropy(parent) - [nL/n · Entropy(L) + nR/n · Entropy(R)]
```

In practice, Gini and Entropy give nearly identical results. Gini is faster to compute.

**Algorithm: CART**
```
function BuildTree(data, depth):
    if stopping criterion met:
        return Leaf(prediction)
    
    best_split = argmin_{j, t} Impurity(split data by feature j ≤ t)
    left_data, right_data = split(data, best_split)
    
    return Node(
        split=best_split,
        left=BuildTree(left_data, depth+1),
        right=BuildTree(right_data, depth+1)
    )
```

**Stopping criteria:**
- Max depth reached
- Node has fewer than `min_samples_split` samples
- Node is pure (Gini = 0)
- Impurity improvement < threshold

### Prediction at Leaf

- **Regression:** mean of training targets in leaf
- **Classification:** majority class (or class proportions for probability)

### Properties

**Advantages:**
- Interpretable — you can trace every prediction
- Handles mixed data types natively
- No feature scaling needed
- Captures non-linear relationships and interactions
- Automatically handles missing values (in some implementations)

**Disadvantages:**
- High variance — small data changes → very different trees
- Prone to overfitting (deep trees memorize training data)
- Axis-aligned splits (can't capture diagonal decision boundaries efficiently)
- Not competitive with ensemble methods on most problems

### Hyperparameters

```python
from sklearn.tree import DecisionTreeClassifier

tree = DecisionTreeClassifier(
    max_depth=5,              # most important regularizer
    min_samples_split=20,     # min samples to split a node
    min_samples_leaf=10,      # min samples in a leaf
    max_features=None,        # features to consider per split
    criterion='gini',         # or 'entropy'
    ccp_alpha=0.01            # cost-complexity pruning (post-pruning)
)
```

**Tuning max_depth:** The primary regularizer. Too deep = overfit. Too shallow = underfit. Use CV to find the sweet spot (typically 3–10).

### Visualization

```python
from sklearn.tree import plot_tree, export_text
import matplotlib.pyplot as plt

plt.figure(figsize=(20, 10))
plot_tree(tree, feature_names=feature_names, class_names=class_names, 
          filled=True, rounded=True, max_depth=3)
plt.show()

# Text representation
print(export_text(tree, feature_names=feature_names))
```

---

## 5.2 Ensemble Methods

The core insight: combine many weak models to get a strong model.

**Why it works:**
- If individual models have error ε each and are independent, the average error goes to 0 as the ensemble grows
- In practice, models are correlated, so gains are real but limited
- Diversity between models is the key — same model same data = no gain

### Bagging (Bootstrap Aggregating)

Train multiple models on different bootstrap samples (random samples with replacement) of the training data. Average their predictions.

```
Bootstrap sample 1 → Model 1
Bootstrap sample 2 → Model 2
      ...
Bootstrap sample B → Model B

Prediction = average(Model 1, ..., Model B)  [regression]
           = majority vote(...)              [classification]
```

Benefits:
- Reduces variance (individual models overfit less when averaged)
- Out-of-Bag (OOB) error: each sample is out-of-bag for ~37% of trees → free validation estimate

---

## 5.3 Random Forests

Random forests = bagging + random feature subsets at each split.

At each node, instead of searching all d features for the best split, only consider a random subset of `m = √d` features (for classification) or `m = d/3` (for regression).

**Why random feature subsets?** It de-correlates the trees. If one feature is very strong, without this trick all trees would look similar (just using that feature at the root). With random subsets, trees are forced to explore other patterns → more diverse ensemble → lower variance.

### Out-of-Bag Error

Each tree is trained on ~63% of the data (bootstrap). The remaining ~37% (OOB samples) act as a validation set for that tree. Averaging OOB predictions across all trees gives a nearly unbiased estimate of generalization error — without needing a separate validation set.

```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(
    n_estimators=100,        # number of trees (more is better, diminishing returns)
    max_depth=None,          # let trees grow deep (bootstrap handles overfitting)
    min_samples_leaf=1,
    max_features='sqrt',     # √d features per split
    oob_score=True,          # compute OOB error
    n_jobs=-1,               # use all CPU cores
    random_state=42
)
rf.fit(X_train, y_train)
print(f"OOB Score: {rf.oob_score_:.4f}")
```

### Feature Importance

Random forests provide a natural feature importance measure: the total reduction in impurity (Gini) across all trees attributable to each feature.

```python
importances = pd.Series(rf.feature_importances_, index=feature_names)
importances.sort_values(ascending=True).plot(kind='barh', figsize=(8, 6))
plt.title('Random Forest Feature Importances')
plt.show()
```

**Caveat:** This impurity-based importance is biased toward high-cardinality features. Prefer **permutation importance** for unbiased estimates:

```python
from sklearn.inspection import permutation_importance

result = permutation_importance(rf, X_test, y_test, n_repeats=10)
perm_imp = pd.Series(result.importances_mean, index=feature_names)
```

### Tuning Random Forests

Random forests are famously robust — even with default settings they perform well. Key parameters:
- `n_estimators`: more is better, ~100–500 usually sufficient
- `max_features`: default is usually good; try `'sqrt'`, `'log2'`, or specific numbers
- `min_samples_leaf`: 1–4 is typical; increase to smooth predictions

---

## 5.4 Gradient Boosting

While bagging reduces variance by averaging, **boosting** reduces bias by building models sequentially — each new model focuses on the errors of the previous ensemble.

### AdaBoost (Adaptive Boosting)

The first successful boosting algorithm:

1. Initialize equal weights for all samples: `wᵢ = 1/n`
2. For t = 1, ..., T:
   - Train weak learner hₜ on weighted data
   - Compute weighted error `εₜ = Σᵢ wᵢ 1[hₜ(xᵢ) ≠ yᵢ]`
   - Compute model weight `αₜ = (1/2) log((1-εₜ)/εₜ)`
   - Update sample weights: increase weights for misclassified, decrease for correct
3. Final prediction: `H(x) = sign(Σₜ αₜ hₜ(x))`

AdaBoost is elegant but sensitive to noisy data (outliers get extreme weights).

### Gradient Boosting Machines (GBM)

A more general and powerful framework. Key insight: **fit each new tree to the residuals (negative gradient) of the current ensemble**.

For any differentiable loss `L`:

1. Initialize `F₀(x) = argmin_γ Σᵢ L(yᵢ, γ)` (e.g., the mean for MSE)
2. For t = 1, ..., T:
   - Compute pseudo-residuals: `rᵢₜ = -∂L(yᵢ, Fₜ₋₁(xᵢ)) / ∂Fₜ₋₁(xᵢ)`
   - Fit a tree hₜ to the pseudo-residuals `{(xᵢ, rᵢₜ)}`
   - Find optimal step size: `γₜ = argmin_γ Σᵢ L(yᵢ, Fₜ₋₁(xᵢ) + γ hₜ(xᵢ))`
   - Update: `Fₜ(x) = Fₜ₋₁(x) + η · γₜ · hₜ(x)`

**For MSE loss:** The pseudo-residuals are exactly the regular residuals: `rᵢ = yᵢ - Fₜ₋₁(xᵢ)`. This gives the intuitive interpretation: each tree fits the current errors.

**Learning rate (shrinkage)** `η ∈ (0, 1]`: Scales the contribution of each tree. Lower `η` → more trees needed, but better generalization.

### XGBoost

The algorithm that dominated Kaggle competitions from 2015 onward. XGBoost adds several improvements over vanilla GBM:

1. **Regularized objective:**
```
L = Σᵢ L(yᵢ, ŷᵢ) + Σₖ Ω(fₖ)    where Ω(f) = γT + ½λ‖w‖²
```
T = number of leaves, w = leaf weights. Regularization is built in.

2. **Second-order Taylor expansion:** Uses both gradient (gᵢ) and Hessian (hᵢ) for better updates.

3. **Approximate split finding:** Efficient histogram-based split enumeration.

4. **Sparsity awareness:** Handles missing values natively.

5. **Column/row subsampling:** Like random forests, reduces overfitting.

6. **Parallelization & cache awareness:** Much faster than scikit-learn's GBM.

```python
import xgboost as xgb

model = xgb.XGBClassifier(
    n_estimators=500,
    max_depth=6,
    learning_rate=0.05,
    subsample=0.8,          # row sampling per tree
    colsample_bytree=0.8,   # column sampling per tree
    reg_alpha=0.1,          # L1 regularization
    reg_lambda=1.0,         # L2 regularization
    use_label_encoder=False,
    eval_metric='logloss',
    random_state=42
)

# Train with early stopping
eval_set = [(X_val, y_val)]
model.fit(X_train, y_train, 
          eval_set=eval_set, 
          early_stopping_rounds=50,
          verbose=100)
```

### LightGBM

Microsoft's GBM implementation, often faster and better than XGBoost for large datasets.

Key innovations:
- **GOSS (Gradient-based One-Side Sampling):** Keep large-gradient samples, randomly sample small-gradient ones
- **EFB (Exclusive Feature Bundling):** Bundle mutually exclusive sparse features
- **Leaf-wise growth:** Grows the leaf with highest loss reduction, not level-by-level like XGBoost

```python
import lightgbm as lgb

model = lgb.LGBMClassifier(
    n_estimators=1000,
    max_depth=-1,           # no depth limit (control via num_leaves)
    num_leaves=31,          # key complexity parameter
    learning_rate=0.05,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_alpha=0.1,
    reg_lambda=1.0
)
model.fit(X_train, y_train, 
          eval_set=[(X_val, y_val)],
          callbacks=[lgb.early_stopping(50), lgb.log_evaluation(100)])
```

### When to Use What

| Algorithm | Best For |
|-----------|----------|
| Decision Tree | Interpretability, small datasets, rule extraction |
| Random Forest | General purpose, fast training, good default |
| XGBoost | Competitive tabular ML, when you have tuning time |
| LightGBM | Very large datasets, many features, fastest training |
| CatBoost | High-cardinality categorical features |

**In practice:** Start with Random Forest as baseline. If more performance needed, try XGBoost/LightGBM with hyperparameter tuning.

---

## 5.5 Hyperparameter Tuning for Boosting

The most important hyperparameters and their interactions:

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

param_dist = {
    'n_estimators': randint(100, 1000),
    'max_depth': randint(3, 10),
    'learning_rate': uniform(0.01, 0.3),
    'subsample': uniform(0.6, 0.4),
    'colsample_bytree': uniform(0.6, 0.4),
    'reg_alpha': uniform(0, 1),
    'reg_lambda': uniform(0.5, 5),
}

search = RandomizedSearchCV(
    xgb.XGBClassifier(eval_metric='logloss'),
    param_distributions=param_dist,
    n_iter=50,
    cv=5,
    scoring='roc_auc',
    n_jobs=-1,
    random_state=42
)
search.fit(X_train, y_train)
print(f"Best params: {search.best_params_}")
```

**General tuning strategy:**
1. Fix `n_estimators=500` with `learning_rate=0.1` and tune other params
2. Then lower `learning_rate` to 0.01–0.05 and raise `n_estimators` proportionally
3. Lower learning rate → more trees → better performance, slower training

---

## Exercises

1. **Implement CART from scratch:** Write a `DecisionTreeClassifier` class that builds a tree using Gini impurity. Test it on the Iris dataset. Verify it matches sklearn's output.

2. **Visualize decision boundaries:** Plot the decision boundary of a decision tree with max_depth=1, 3, 5, and unlimited on a 2D dataset. What do you observe?

3. **Random forest OOB:** Generate a dataset of 500 samples. Train a random forest with `oob_score=True`. Compare the OOB score to the test set score from a proper train/test split.

4. **Gradient boosting stages:** Train a GBM on a regression problem. Plot the training and validation RMSE as a function of number of trees. Identify the optimal number of trees (where test error is minimized). Use early stopping.

5. **Feature importance comparison:** On a dataset with 20 features, compare impurity-based importance vs. permutation importance from a random forest. Do they rank features the same? Which do you trust more, and why?

---

## Further Reading

- *The Elements of Statistical Learning* — Hastie et al. — Chapter 9 (Additive Models, Trees) and Chapter 10 (Boosting)
- *XGBoost: A Scalable Tree Boosting System* — Chen & Guestrin (2016) (original paper, very readable)
- Friedman (2001) "Greedy Function Approximation: A Gradient Boosting Machine" (derivation of GBM)
