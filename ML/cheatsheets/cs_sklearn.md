# Cheat Sheet — Scikit-learn

> Unified API reference. All estimators follow the same pattern.

---

## The Core API

```python
# Every estimator:
model.fit(X_train, y_train)         # learn from data
model.predict(X_test)               # predict labels
model.predict_proba(X_test)         # predict probabilities (classifiers)
model.score(X_test, y_test)         # default metric (accuracy/R²)
model.transform(X)                  # apply transformation (transformers)
model.fit_transform(X)              # fit + transform in one step
```

---

## Data Splitting

```python
from sklearn.model_selection import train_test_split, StratifiedKFold, KFold

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y)  # classification

kf  = KFold(n_splits=5, shuffle=True, random_state=42)
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

from sklearn.model_selection import cross_val_score
scores = cross_val_score(model, X, y, cv=5, scoring='roc_auc', n_jobs=-1)
print(f"{scores.mean():.4f} ± {scores.std():.4f}")
```

---

## Preprocessing

```python
from sklearn.preprocessing import (
    StandardScaler,       # (x - mean) / std
    MinMaxScaler,         # (x - min) / (max - min) → [0,1]
    RobustScaler,         # uses median/IQR, robust to outliers
    PowerTransformer,     # Yeo-Johnson / Box-Cox → more Gaussian
    LabelEncoder,         # target: str → int
    OrdinalEncoder,       # features: ordered categories → int
    OneHotEncoder,        # features: nominal categories → binary
)
from sklearn.impute import SimpleImputer, KNNImputer

# ALWAYS fit on train, transform on both
scaler = StandardScaler()
X_tr_s = scaler.fit_transform(X_tr)   # fit + transform
X_te_s = scaler.transform(X_te)       # transform only (!)

# Imputation
imp = SimpleImputer(strategy='median')   # 'mean', 'most_frequent', 'constant'
X_imp = imp.fit_transform(X_tr)
```

---

## Pipelines

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer

num_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

cat_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='constant', fill_value='unknown')),
    ('ohe', OneHotEncoder(handle_unknown='ignore', sparse_output=False))
])

preprocessor = ColumnTransformer([
    ('num', num_transformer, num_cols),
    ('cat', cat_transformer, cat_cols)
])

pipe = Pipeline([
    ('preproc', preprocessor),
    ('model', RandomForestClassifier())
])

pipe.fit(X_tr, y_tr)
pipe.predict(X_te)
cross_val_score(pipe, X, y, cv=5)   # pipeline used inside CV safely
```

---

## Models Quick Reference

```python
# ── Classification ────────────────────────────────────────────
from sklearn.linear_model    import LogisticRegression
from sklearn.tree            import DecisionTreeClassifier
from sklearn.ensemble        import RandomForestClassifier, GradientBoostingClassifier, AdaBoostClassifier
from sklearn.svm             import SVC
from sklearn.neighbors       import KNeighborsClassifier
from sklearn.naive_bayes     import GaussianNB, MultinomialNB

# ── Regression ────────────────────────────────────────────────
from sklearn.linear_model    import LinearRegression, Ridge, Lasso, ElasticNet
from sklearn.tree            import DecisionTreeRegressor
from sklearn.ensemble        import RandomForestRegressor, GradientBoostingRegressor
from sklearn.svm             import SVR

# ── Clustering ────────────────────────────────────────────────
from sklearn.cluster         import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.mixture         import GaussianMixture

# ── Dimensionality Reduction ──────────────────────────────────
from sklearn.decomposition   import PCA, TruncatedSVD, NMF, FastICA
from sklearn.manifold        import TSNE, LocallyLinearEmbedding

# ── Feature Selection ─────────────────────────────────────────
from sklearn.feature_selection import SelectKBest, RFE, RFECV, SelectFromModel
from sklearn.feature_selection import f_classif, mutual_info_classif
```

---

## Metrics

```python
# ── Classification ────────────────────────────────────────────
from sklearn.metrics import (
    accuracy_score,
    precision_score, recall_score, f1_score,
    roc_auc_score, average_precision_score,
    confusion_matrix, classification_report,
    log_loss, brier_score_loss,
    cohen_kappa_score
)

# Probabilities needed for some:
y_proba = model.predict_proba(X_te)[:, 1]
roc_auc = roc_auc_score(y_te, y_proba)
ap = average_precision_score(y_te, y_proba)

# Multiclass averaging:
f1 = f1_score(y_te, y_pred, average='macro')     # equal class weight
f1 = f1_score(y_te, y_pred, average='weighted')  # weighted by support

# Full report:
print(classification_report(y_te, y_pred))

# ── Regression ────────────────────────────────────────────────
from sklearn.metrics import (
    mean_absolute_error,
    mean_squared_error,
    root_mean_squared_error,
    r2_score,
    mean_absolute_percentage_error
)

# ── Clustering ────────────────────────────────────────────────
from sklearn.metrics import silhouette_score, adjusted_rand_score

# Curve objects:
from sklearn.metrics import roc_curve, precision_recall_curve
fpr, tpr, thresh = roc_curve(y_te, y_proba)
prec, rec, thresh = precision_recall_curve(y_te, y_proba)
```

---

## Hyperparameter Tuning

```python
from sklearn.model_selection import GridSearchCV, RandomizedSearchCV
from scipy.stats import loguniform, randint, uniform

# Grid search (exhaustive)
param_grid = {'max_depth': [3, 5, 7], 'n_estimators': [100, 300]}
grid = GridSearchCV(model, param_grid, cv=5, scoring='roc_auc', n_jobs=-1, refit=True)
grid.fit(X_tr, y_tr)
print(grid.best_params_, grid.best_score_)

# Random search (faster)
param_dist = {
    'max_depth': randint(3, 15),
    'learning_rate': loguniform(1e-3, 0.3),    # log-uniform for rates
    'subsample': uniform(0.5, 0.5)             # uniform [0.5, 1.0]
}
search = RandomizedSearchCV(model, param_dist, n_iter=50, cv=5, scoring='roc_auc', n_jobs=-1)
search.fit(X_tr, y_tr)

# Access results
results_df = pd.DataFrame(grid.cv_results_)
```

---

## Scoring Strings (use in cross_val_score)

| Task | Common scorers |
|------|---------------|
| Binary classification | `'accuracy'`, `'roc_auc'`, `'f1'`, `'average_precision'` |
| Multiclass | `'accuracy'`, `'f1_macro'`, `'f1_weighted'` |
| Regression | `'r2'`, `'neg_root_mean_squared_error'`, `'neg_mean_absolute_error'` |

Note: all scorers are **greater-is-better** in sklearn — negative metrics are negated.

---

## Useful Utilities

```python
from sklearn.utils.class_weight import compute_class_weight
weights = compute_class_weight('balanced', classes=np.unique(y), y=y)
class_weight_dict = dict(enumerate(weights))
model = LogisticRegression(class_weight=class_weight_dict)
# Or: class_weight='balanced' directly in most classifiers

from sklearn.inspection import permutation_importance
result = permutation_importance(model, X_te, y_te, n_repeats=10)
perm_imp = pd.Series(result.importances_mean, index=feature_names).sort_values()

from sklearn.calibration import CalibratedClassifierCV
cal_model = CalibratedClassifierCV(model, method='isotonic', cv=5)

from sklearn.dummy import DummyClassifier, DummyRegressor
baseline = DummyClassifier(strategy='most_frequent')   # always predict majority class
baseline.fit(X_tr, y_tr); baseline.score(X_te, y_te)
```
