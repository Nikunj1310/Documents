# Chapter 8 — Feature Engineering

> *Feature engineering is where domain knowledge meets machine learning. The best algorithm on bad features will lose to a mediocre algorithm on good features. This chapter is what separates competitive practitioners from beginners.*

---

## 8.1 The Role of Features

A **feature** is a measurable property of the phenomenon you are modeling. Your model can only learn patterns that exist (at least implicitly) in the features you provide.

The three fundamental problems:
1. **Missing information:** Features that would help don't exist in your data
2. **Wrong representation:** Features exist but aren't in the right form
3. **Noise:** Features contain irrelevant variation that confuses the model

Feature engineering addresses all three.

---

## 8.2 Handling Missing Values

Missing data is one of the most common real-world problems.

### Types of Missingness

- **MCAR (Missing Completely At Random):** Missing values have no relationship with any variable. Safe to impute or drop.
- **MAR (Missing At Random):** Missingness depends on other observed variables (e.g., women less likely to report income). Can impute using those variables.
- **MNAR (Missing Not At Random):** Missingness depends on the missing value itself (e.g., very high earners don't report income). Hard — may need domain knowledge or model the missingness.

### Imputation Strategies

```python
import pandas as pd
import numpy as np
from sklearn.impute import SimpleImputer, KNNImputer
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

# Mean/median/mode imputation
simple_imputer = SimpleImputer(strategy='mean')    # numerical
mode_imputer = SimpleImputer(strategy='most_frequent')  # categorical
const_imputer = SimpleImputer(strategy='constant', fill_value=-1)

# K-NN imputation (uses nearby samples to estimate)
knn_imputer = KNNImputer(n_neighbors=5)

# MICE (Multiple Imputation by Chained Equations) — most sophisticated
mice_imputer = IterativeImputer(max_iter=10, random_state=42)

# Apply
X_imputed = knn_imputer.fit_transform(X_train)
```

### Missing Indicator Feature

Always consider adding a binary "was_missing" feature. The fact that a value is missing may itself be informative.

```python
from sklearn.impute import MissingIndicator

indicator = MissingIndicator()
missing_flags = indicator.fit_transform(X)  # binary array of is_missing
X_with_flags = np.hstack([X_imputed, missing_flags])
```

**Best practice:** Impute + add missing indicator together via `IterativeImputer` or `Pipeline`. Always impute using statistics computed on training data only.

---

## 8.3 Encoding Categorical Features

ML algorithms need numbers. Categorical features need encoding.

### Ordinal Encoding

Use when the categories have a natural order (small < medium < large).

```python
from sklearn.preprocessing import OrdinalEncoder

enc = OrdinalEncoder(categories=[['low', 'medium', 'high']])
X_enc = enc.fit_transform(X[['size']])
```

### One-Hot Encoding

Use when categories have no natural order. Creates binary columns for each category.

```python
from sklearn.preprocessing import OneHotEncoder

ohe = OneHotEncoder(
    drop='first',         # drop one category to avoid perfect collinearity
    sparse_output=False,  # return dense array
    handle_unknown='ignore'  # unseen categories → all zeros
)
X_enc = ohe.fit_transform(X[['city']])
```

**Problem with high cardinality:** A feature with 1,000 unique values becomes 999 binary columns — very sparse, slow, and noisy.

### Target Encoding (Mean Encoding)

Replace each category with the mean of the target variable for that category.

```python
# Manual implementation (always do this within CV to prevent leakage)
def target_encode(train, val, col, target, smoothing=10):
    global_mean = train[target].mean()
    counts = train.groupby(col)[target].agg(['sum', 'count'])
    smooth = (counts['sum'] + smoothing * global_mean) / (counts['count'] + smoothing)
    val_encoded = val[col].map(smooth).fillna(global_mean)
    return val_encoded

# Or use category_encoders library
from category_encoders import TargetEncoder
enc = TargetEncoder(smoothing=10)
X_train['city_encoded'] = enc.fit_transform(X_train['city'], y_train)
X_test['city_encoded'] = enc.transform(X_test['city'])
```

**Caution:** Target encoding leaks label information — ALWAYS compute within cross-validation folds.

### Frequency Encoding

Replace each category with its frequency in the training data. Good for high-cardinality:

```python
freq_map = X_train['city'].value_counts(normalize=True).to_dict()
X_train['city_freq'] = X_train['city'].map(freq_map)
X_test['city_freq'] = X_test['city'].map(freq_map).fillna(0)
```

### Hashing

For very high-cardinality (millions of categories), use feature hashing:
```python
from sklearn.feature_extraction import FeatureHasher

hasher = FeatureHasher(n_features=256, input_type='string')
X_hashed = hasher.transform(X['city'].apply(lambda x: [x]))
```

---

## 8.4 Feature Scaling

Distance-based algorithms (k-NN, SVM, neural networks) are sensitive to feature scale. Tree-based methods are not.

### Standardization (Z-score)

```
z = (x - μ) / σ
```
Centers around 0, unit variance. Assumes approximately normal distribution.

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_train)  # fit on train only
X_test_scaled = scaler.transform(X_test)   # transform test without fitting
```

### Min-Max Scaling

```
x_scaled = (x - x_min) / (x_max - x_min)  → [0, 1]
```
Sensitive to outliers (one extreme value compresses everything else).

```python
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler(feature_range=(0, 1))
```

### Robust Scaling

Uses median and IQR instead of mean and std. Robust to outliers.
```python
from sklearn.preprocessing import RobustScaler
scaler = RobustScaler(quantile_range=(25, 75))
```

### Log Transformation

For right-skewed distributions (income, population, counts):
```python
X['log_income'] = np.log1p(X['income'])  # log1p = log(1+x), handles zeros
```

### Power Transformations

Yeo-Johnson or Box-Cox transformations to make distributions more Gaussian:
```python
from sklearn.preprocessing import PowerTransformer
pt = PowerTransformer(method='yeo-johnson')  # works with negative values
X_transformed = pt.fit_transform(X)
```

**When to scale:** Standardize for linear models, SVMs, k-NN, and neural networks. Not needed for tree-based methods.

---

## 8.5 Feature Creation

Creating new features from existing ones is often where the biggest performance gains come from.

### Polynomial Features

```python
from sklearn.preprocessing import PolynomialFeatures

poly = PolynomialFeatures(degree=2, include_bias=False, interaction_only=False)
X_poly = poly.fit_transform(X)
# For 3 features, degree=2 adds: x₁², x₂², x₃², x₁x₂, x₁x₃, x₂x₃
```

**Caution:** For d features and degree k, produces O(dᵏ) features. Quickly explodes. Combine with regularization.

### Interaction Features

Manually create interactions based on domain knowledge:
```python
df['bmi'] = df['weight'] / df['height']**2
df['age_x_income'] = df['age'] * df['income']
df['rooms_per_person'] = df['total_rooms'] / df['population']
```

### Date/Time Features

```python
df['datetime'] = pd.to_datetime(df['timestamp'])
df['hour'] = df['datetime'].dt.hour
df['day_of_week'] = df['datetime'].dt.dayofweek
df['month'] = df['datetime'].dt.month
df['is_weekend'] = df['day_of_week'].isin([5, 6]).astype(int)
df['days_since_epoch'] = (df['datetime'] - pd.Timestamp('2020-01-01')).dt.days

# Cyclical encoding for periodic features
df['hour_sin'] = np.sin(2 * np.pi * df['hour'] / 24)
df['hour_cos'] = np.cos(2 * np.pi * df['hour'] / 24)
```

**Cyclical encoding:** For cyclical features (hour, day of week, month), using sin/cos preserves the cyclical relationship. Without it, hour 23 and hour 0 appear far apart.

### Aggregation Features (Group Statistics)

For datasets with groupings (user-level, item-level, store-level):
```python
# Add per-user aggregate features
user_stats = df.groupby('user_id')['purchase_amount'].agg(['mean', 'std', 'count', 'max'])
user_stats.columns = ['user_mean_spend', 'user_std_spend', 'user_purchase_count', 'user_max_spend']
df = df.merge(user_stats, on='user_id', how='left')
```

### Text Features

For raw text:
```python
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer

# TF-IDF (term frequency × inverse document frequency)
tfidf = TfidfVectorizer(
    max_features=5000,
    ngram_range=(1, 2),  # unigrams and bigrams
    strip_accents='unicode',
    min_df=2,            # ignore terms in fewer than 2 documents
    max_df=0.95,         # ignore terms in more than 95% of documents
    sublinear_tf=True    # apply log normalization to tf
)
X_tfidf = tfidf.fit_transform(text_data)  # sparse matrix
```

---

## 8.6 Feature Selection

Not all features help. Irrelevant features add noise and increase overfitting. Feature selection improves accuracy and speed.

### Filter Methods (Independent of Model)

```python
from sklearn.feature_selection import (
    SelectKBest, SelectPercentile,
    f_classif, f_regression, mutual_info_classif, chi2,
    VarianceThreshold
)

# Remove low-variance features (often near-constant)
selector = VarianceThreshold(threshold=0.01)
X_selected = selector.fit_transform(X)

# Select by statistical test
selector_kbest = SelectKBest(score_func=f_classif, k=20)
X_selected = selector_kbest.fit_transform(X, y)

# Mutual information (captures non-linear relationships)
selector_mi = SelectKBest(score_func=mutual_info_classif, k=20)
X_selected = selector_mi.fit_transform(X, y)
```

### Wrapper Methods (Model-Based)

```python
from sklearn.feature_selection import RFE, RFECV
from sklearn.linear_model import LogisticRegression

# Recursive Feature Elimination
rfe = RFE(
    estimator=LogisticRegression(max_iter=1000),
    n_features_to_select=15,
    step=1
)
X_rfe = rfe.fit_transform(X_train, y_train)
selected_features = np.array(feature_names)[rfe.support_]

# RFECV: finds optimal n automatically via CV
rfecv = RFECV(estimator=LogisticRegression(max_iter=1000), cv=5, scoring='roc_auc')
rfecv.fit(X_train, y_train)
print(f"Optimal feature count: {rfecv.n_features_}")
```

### Embedded Methods (Feature Importance from Model)

```python
from sklearn.feature_selection import SelectFromModel
from sklearn.ensemble import RandomForestClassifier

# Select features above mean importance
rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

selector = SelectFromModel(rf, threshold='mean', prefit=True)
X_selected = selector.transform(X_train)

# Lasso for linear feature selection
from sklearn.linear_model import LassoCV
lasso = LassoCV(cv=5, random_state=42)
lasso.fit(X_train, y_train)
selected = np.where(lasso.coef_ != 0)[0]
```

### Correlation-Based Removal

Remove highly correlated features (they add no information but add variance):
```python
def remove_correlated_features(df, threshold=0.95):
    corr_matrix = df.corr().abs()
    upper = corr_matrix.where(np.triu(np.ones(corr_matrix.shape), k=1).astype(bool))
    to_drop = [col for col in upper.columns if any(upper[col] > threshold)]
    return df.drop(columns=to_drop), to_drop

X_uncorrelated, dropped = remove_correlated_features(X_df, threshold=0.95)
```

---

## 8.7 Pipelines for Feature Engineering

All preprocessing must happen inside a pipeline to prevent leakage:

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer

numerical_cols = ['age', 'income', 'amount']
categorical_cols = ['city', 'category']

numerical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler()),
])

categorical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='constant', fill_value='unknown')),
    ('ohe', OneHotEncoder(handle_unknown='ignore', sparse_output=False)),
])

preprocessor = ColumnTransformer([
    ('num', numerical_transformer, numerical_cols),
    ('cat', categorical_transformer, categorical_cols),
])

full_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(n_estimators=100))
])

full_pipeline.fit(X_train, y_train)
y_pred = full_pipeline.predict(X_test)
```

This is the correct, leak-proof way to build ML pipelines.

---

## 8.8 Feature Engineering Checklist

Before modeling, work through this list:

```
NUMERICAL FEATURES
[ ] Check for missing values → impute or add indicator
[ ] Check for outliers → cap at percentiles or use Robust scaler
[ ] Check distribution → log-transform if right-skewed
[ ] Scale → StandardScaler for linear/neural models
[ ] Create interactions/ratios if domain knowledge suggests it
[ ] Create polynomial features (for linear models with non-linear patterns)

CATEGORICAL FEATURES
[ ] Check cardinality of each feature
[ ] Low cardinality (<10) → One-hot encoding
[ ] High cardinality (>100) → Target encoding or frequency encoding
[ ] Ordinal → Ordinal encoding with correct order
[ ] Handle unseen categories in test set

DATE/TIME FEATURES
[ ] Extract hour, day, month, year, day of week
[ ] Create cyclical encodings for periodic features
[ ] Compute time since reference (days since signup, etc.)
[ ] Create business-logic features (is_weekend, is_holiday)

TEXT FEATURES
[ ] Clean text (lowercase, remove punctuation, stop words)
[ ] TF-IDF for classical ML
[ ] Embeddings (BERT etc.) for deep learning

DOMAIN-SPECIFIC
[ ] Create aggregation features (user-level stats, store-level stats)
[ ] Encode domain knowledge as features
[ ] Create ratio/rate features

SELECTION & VALIDATION
[ ] Remove near-zero variance features
[ ] Remove highly correlated features
[ ] Run model-based feature importance
[ ] Validate that features don't leak target information
```

---

## Exercises

1. **Missing value strategies:** Create a dataset with 20% MCAR missing values. Compare 5 imputation strategies (mean, median, constant, KNN, MICE) using 5-fold CV RMSE. Which performs best?

2. **Encoding benchmark:** On a dataset with high-cardinality categoricals, compare OHE, target encoding, and frequency encoding. How does each affect model performance and training speed?

3. **Date feature engineering:** Use a dataset with timestamps (e.g., NYC taxi rides). Create hour, day, month, cyclical, and "time since midnight" features. Which features most improve prediction?

4. **Correlation filtering:** Generate 50 features where features 1–45 are random noise and 46–50 are the actual signal. Use correlation-based removal, random forest importance, and RFECV to select features. Which method best identifies the 5 signal features?

5. **End-to-end pipeline:** Build a complete `ColumnTransformer` + `Pipeline` for a dataset with mixed types (numerical + categorical + date). Use cross-validation to evaluate. Make sure there is zero data leakage.

---

## Further Reading

- *Feature Engineering for Machine Learning* — Alice Zheng & Amanda Casari
- *Feature Engineering and Selection* — Kuhn & Johnson (free online at `feat.engineering`)
- Fast.ai lesson on tabular data — excellent practical discussion
