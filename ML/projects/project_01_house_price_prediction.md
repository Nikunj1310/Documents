# Project 1 — House Price Prediction (End-to-End Tabular ML)

**Covers:** Chapters 1–8  
**Estimated time:** 1–2 weeks  
**Difficulty:** Beginner–Intermediate

---

## Project Goal

Build a production-quality regression pipeline to predict house prices. By the end, you will have:
- A fully reproducible ML pipeline with no data leakage
- A model beating the Kaggle baseline by a significant margin
- A REST API serving predictions
- An MLflow experiment log comparing multiple approaches

---

## Dataset

**Kaggle House Prices - Advanced Regression Techniques**
URL: `kaggle.com/competitions/house-prices-advanced-regression-techniques`

```bash
kaggle competitions download -c house-prices-advanced-regression-techniques
unzip house-prices-advanced-regression-techniques.zip -d data/
```

---

## Step 1: Exploratory Data Analysis

Spend 2–3 hours here before touching a model.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

train = pd.read_csv('data/train.csv')
test = pd.read_csv('data/test.csv')

print(f"Train shape: {train.shape}")    # (1460, 81)
print(f"Test shape: {test.shape}")      # (1459, 80)

# Target distribution
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
train['SalePrice'].hist(ax=axes[0], bins=50)
axes[0].set_title('SalePrice Distribution (raw)')
np.log1p(train['SalePrice']).hist(ax=axes[1], bins=50)
axes[1].set_title('SalePrice Distribution (log-transformed)')
plt.show()
# → Target is right-skewed → use log-transform

# Missing values
missing = (train.isnull().sum() / len(train) * 100).sort_values(ascending=False)
print(missing[missing > 0].head(20))

# Correlation with target
corr = train.select_dtypes(include=np.number).corr()
top_features = corr['SalePrice'].abs().sort_values(ascending=False).head(15)
print(top_features)

# Key findings to document:
# - PoolQC, MiscFeature, Alley, Fence are mostly NA (legitimate absence)
# - OverallQual has strongest correlation (0.79)
# - GrLivArea highly correlated (0.71) but has outliers
# - Neighborhood is important (categorical)
```

**EDA Deliverable:** Write a bullet-point summary of 10 key findings from the data.

---

## Step 2: Feature Engineering

This is where the most value comes from.

```python
from sklearn.base import BaseEstimator, TransformerMixin

class HousePriceFeatureEngineer(BaseEstimator, TransformerMixin):
    def fit(self, X, y=None):
        return self
    
    def transform(self, X):
        X = X.copy()
        
        # Handle "NA" = not applicable (not missing)
        na_means_none = ['PoolQC', 'MiscFeature', 'Alley', 'Fence',
                         'FireplaceQu', 'GarageType', 'GarageFinish',
                         'GarageQual', 'GarageCond', 'BsmtQual', 'BsmtCond',
                         'BsmtExposure', 'BsmtFinType1', 'BsmtFinType2',
                         'MasVnrType', 'MSSubClass']
        for col in na_means_none:
            if col in X.columns:
                X[col] = X[col].fillna('None')
        
        # Zero fill for these (NA = 0 count/area)
        zero_fill = ['GarageYrBlt', 'GarageArea', 'GarageCars',
                     'BsmtFinSF1', 'BsmtFinSF2', 'BsmtUnfSF', 'TotalBsmtSF',
                     'BsmtFullBath', 'BsmtHalfBath', 'MasVnrArea']
        for col in zero_fill:
            if col in X.columns:
                X[col] = X[col].fillna(0)
        
        # Mode fill for categoricals
        for col in ['MSZoning', 'Utilities', 'Functional', 'Electrical',
                    'Exterior1st', 'Exterior2nd', 'KitchenQual', 'SaleType']:
            if col in X.columns:
                X[col] = X[col].fillna(X[col].mode()[0])
        
        # ── New Features ─────────────────────────────────────
        # Total area
        X['TotalSF'] = (X['TotalBsmtSF'] + X['1stFlrSF'] + X['2ndFlrSF'])
        
        # House age at time of sale
        X['HouseAge'] = X['YrSold'] - X['YearBuilt']
        X['YearsAfterRemodel'] = X['YrSold'] - X['YearRemodAdd']
        
        # Has certain features
        X['HasPool'] = (X['PoolArea'] > 0).astype(int)
        X['HasGarage'] = (X['GarageArea'] > 0).astype(int)
        X['HasBsmt'] = (X['TotalBsmtSF'] > 0).astype(int)
        X['HasFireplace'] = (X['Fireplaces'] > 0).astype(int)
        
        # Total porch area
        X['TotalPorchSF'] = (X['OpenPorchSF'] + X['EnclosedPorch'] +
                              X['3SsnPorch'] + X['ScreenPorch'])
        
        # Bathroom count
        X['TotalBath'] = (X['FullBath'] + 0.5 * X['HalfBath'] +
                           X['BsmtFullBath'] + 0.5 * X['BsmtHalfBath'])
        
        # Quality × condition interaction
        X['OverallScore'] = X['OverallQual'] * X['OverallCond']
        
        # Log area (right-skewed distributions)
        X['LogLotArea'] = np.log1p(X['LotArea'])
        X['LogGrLivArea'] = np.log1p(X['GrLivArea'])
        
        return X
    
    def get_feature_names_out(self):
        return None  # required for sklearn compatibility
```

---

## Step 3: Full Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OrdinalEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import GradientBoostingRegressor
import xgboost as xgb

# Define feature groups
numerical_features = [
    'LotArea', 'OverallQual', 'OverallCond', 'YearBuilt', 'YearRemodAdd',
    'MasVnrArea', 'BsmtFinSF1', 'BsmtUnfSF', 'TotalBsmtSF', '1stFlrSF',
    '2ndFlrSF', 'GrLivArea', 'BsmtFullBath', 'FullBath', 'HalfBath',
    'BedroomAbvGr', 'KitchenAbvGr', 'TotRmsAbvGrd', 'Fireplaces',
    'GarageYrBlt', 'GarageCars', 'GarageArea', 'WoodDeckSF', 'OpenPorchSF',
    'EnclosedPorch', 'ScreenPorch', 'PoolArea', 'YrSold',
    # Engineered
    'TotalSF', 'HouseAge', 'YearsAfterRemodel', 'TotalBath', 'TotalPorchSF',
    'HasPool', 'HasGarage', 'HasBsmt', 'HasFireplace', 'OverallScore',
    'LogLotArea', 'LogGrLivArea'
]

categorical_features = [
    'MSSubClass', 'MSZoning', 'LotShape', 'LandContour', 'Utilities',
    'LotConfig', 'LandSlope', 'Neighborhood', 'Condition1', 'BldgType',
    'HouseStyle', 'RoofStyle', 'Exterior1st', 'Exterior2nd', 'MasVnrType',
    'ExterQual', 'ExterCond', 'Foundation', 'BsmtQual', 'BsmtExposure',
    'BsmtFinType1', 'Heating', 'CentralAir', 'Electrical', 'KitchenQual',
    'Functional', 'FireplaceQu', 'GarageType', 'GarageFinish', 'SaleType',
    'SaleCondition'
]

numerical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

categorical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='constant', fill_value='None')),
    ('encoder', OrdinalEncoder(handle_unknown='use_encoded_value', unknown_value=-1))
])

preprocessor = ColumnTransformer([
    ('num', numerical_transformer, numerical_features),
    ('cat', categorical_transformer, categorical_features)
])

# Target transform (log1p)
y_train_log = np.log1p(y_train)

# XGBoost model
xgb_model = xgb.XGBRegressor(
    n_estimators=2000,
    max_depth=4,
    learning_rate=0.03,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_alpha=0.05,
    reg_lambda=0.1,
    early_stopping_rounds=50,
    eval_metric='rmse',
    random_state=42
)

pipeline = Pipeline([
    ('features', HousePriceFeatureEngineer()),
    ('preprocessor', preprocessor),
    ('model', xgb_model)
])
```

---

## Step 4: Training and Evaluation

```python
from sklearn.model_selection import KFold, cross_val_score

# Cross-validation (5-fold)
kf = KFold(n_splits=5, shuffle=True, random_state=42)
cv_scores = cross_val_score(pipeline, X_train, y_train_log, 
                            cv=kf, scoring='neg_root_mean_squared_error')
print(f"CV RMSE (log): {-cv_scores.mean():.4f} ± {cv_scores.std():.4f}")

# Fit on full training data
pipeline.fit(X_train, y_train_log, 
             model__eval_set=[(preprocessor.transform(X_val_features), y_val_log)],
             model__verbose=200)

# Predictions (remember to inverse-transform!)
y_pred_log = pipeline.predict(X_test)
y_pred = np.expm1(y_pred_log)
```

---

## Step 5: Model Stacking (Bonus)

Blend multiple models for better performance:

```python
from sklearn.linear_model import Ridge

# Train multiple models
models = {
    'xgb': XGBRegressor(...),
    'lgb': LGBMRegressor(...),
    'ridge': Ridge(alpha=10)
}

# Level-1 predictions (out-of-fold)
oof_preds = np.zeros((len(X_train), len(models)))
for i, (name, model) in enumerate(models.items()):
    for fold, (train_idx, val_idx) in enumerate(kf.split(X_train)):
        model.fit(X_tr[train_idx], y_tr[train_idx])
        oof_preds[val_idx, i] = model.predict(X_tr[val_idx])

# Level-2 model (meta-learner)
meta_model = Ridge(alpha=1.0)
meta_model.fit(oof_preds, y_train_log)

# Test predictions: average base model predictions
test_preds = np.column_stack([m.predict(X_test) for m in models.values()])
final_preds = meta_model.predict(test_preds)
```

---

## Step 6: Submit to Kaggle

```python
submission = pd.DataFrame({
    'Id': test['Id'],
    'SalePrice': y_pred
})
submission.to_csv('submission.csv', index=False)

# kaggle competitions submit -c house-prices-advanced-regression-techniques -f submission.csv -m "XGBoost with feature engineering"
```

---

## Deliverables

- [ ] EDA notebook with documented findings
- [ ] Feature engineering class implementing `BaseEstimator` and `TransformerMixin`
- [ ] Full sklearn pipeline (no data leakage)
- [ ] 5-fold CV RMSE < 0.12 (log-scale RMSE)
- [ ] Kaggle submission in top 30%
- [ ] MLflow experiment log comparing ≥ 3 model variants
- [ ] Optional: Stacked ensemble
- [ ] Optional: FastAPI endpoint serving predictions

## Learning Outcomes

After this project you will:
- Have experience with a real messy dataset
- Know how to handle 80 features including categoricals, missing values, and distributions
- Know that feature engineering often matters more than model selection
- Have a reusable pipeline template for future tabular projects
