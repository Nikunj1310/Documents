# Chapter 2 — Python & Data Science Tools

> *This chapter is a practical reference. Work through every code block. If you already know NumPy/Pandas well, skim for gaps and move to Chapter 3.*

---

## 2.1 The Python ML Stack

```
NumPy       — numerical arrays, linear algebra
Pandas      — tabular data manipulation
Matplotlib  — 2D plotting
Seaborn     — statistical visualization (built on Matplotlib)
Scikit-learn — classical ML algorithms and utilities
PyTorch     — deep learning (covered in Ch 9+)
```

Install everything:
```bash
pip install numpy pandas matplotlib seaborn scikit-learn jupyter
```

---

## 2.2 NumPy

NumPy's core is the `ndarray` — a fixed-type, contiguous-memory array that is 10–1000× faster than Python lists for numerical operations.

### Creating Arrays

```python
import numpy as np

# From lists
a = np.array([1, 2, 3])                 # 1D, shape (3,)
A = np.array([[1, 2], [3, 4]])          # 2D, shape (2,2)

# Utility constructors
np.zeros((3, 4))        # 3×4 of zeros
np.ones((2, 5))         # 2×5 of ones
np.eye(4)               # 4×4 identity matrix
np.arange(0, 10, 2)     # [0, 2, 4, 6, 8]
np.linspace(0, 1, 5)    # [0., 0.25, 0.5, 0.75, 1.]

# Random arrays (always set a seed for reproducibility)
rng = np.random.default_rng(42)
rng.random((3, 3))       # uniform [0,1)
rng.standard_normal((3, 3))  # N(0,1)
rng.integers(0, 10, size=(4,))  # random ints
```

### Array Properties

```python
a = np.array([[1., 2., 3.], [4., 5., 6.]])

a.shape    # (2, 3)
a.ndim     # 2
a.size     # 6
a.dtype    # dtype('float64')
```

### Indexing and Slicing

```python
a = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

a[0]        # first row: [1, 2, 3]
a[:, 1]     # second column: [2, 5, 8]
a[1:, :2]   # rows 1+, cols 0-1: [[4,5],[7,8]]
a[::2]      # every other row

# Boolean indexing
a[a > 4]    # array([5, 6, 7, 8, 9])
mask = np.array([True, False, True])
a[mask]     # rows 0 and 2
```

### Broadcasting

Broadcasting allows operations on arrays of different shapes without copying data. NumPy "stretches" the smaller array virtually.

```python
# Add a scalar to every element
np.array([1, 2, 3]) + 10       # [11, 12, 13]

# Add a (3,) vector to every row of a (4, 3) matrix
A = np.ones((4, 3))
b = np.array([1, 2, 3])
A + b  # shape (4, 3) — b is broadcast across rows

# Outer product via broadcasting
x = np.array([1, 2, 3])           # shape (3,)
y = np.array([10, 20])[:, None]   # shape (2, 1)
x + y  # shape (2, 3) — both broadcast
```

**Broadcasting rules:**
1. Dimensions are compared from the right
2. Dimensions match if equal or one of them is 1
3. The smaller array is stretched along size-1 dimensions

### Vectorized Operations

```python
x = np.array([1., 4., 9.])

np.sqrt(x)          # [1., 2., 3.]
np.exp(x)           # elementwise e^x
np.log(x)           # natural log

# Reductions
x.sum()             # scalar
x.mean()
x.std()
A.sum(axis=0)       # sum along rows (result per column)
A.sum(axis=1)       # sum along columns (result per row)

# Linear algebra
np.dot(A, B)        # matrix multiply (or A @ B)
np.linalg.norm(x)   # L2 norm
np.linalg.solve(A, b)  # solve Ax = b
```

### Reshaping

```python
a = np.arange(12)        # shape (12,)
a.reshape(3, 4)          # shape (3, 4)
a.reshape(2, -1)         # shape (2, 6) — -1 infers the dim
a[:, None]               # shape (12, 1) — add axis

# Flatten
A.flatten()              # always returns copy
A.ravel()                # returns view when possible (faster)
```

### Memory Efficiency: Views vs. Copies

```python
a = np.array([1, 2, 3, 4])
b = a[1:3]   # b is a VIEW — shares memory with a
b[0] = 99    # modifies a too!

c = a[1:3].copy()   # c is an independent copy
```

Understanding this prevents subtle bugs in ML pipelines.

---

## 2.3 Pandas

Pandas provides two main structures:
- `Series` — 1D labeled array (like a column)
- `DataFrame` — 2D labeled table (like a spreadsheet)

### Creating DataFrames

```python
import pandas as pd

# From dict
df = pd.DataFrame({
    'age': [25, 30, 35, 40],
    'income': [40000, 60000, 80000, 120000],
    'purchased': [0, 1, 1, 1]
})

# From CSV (the most common)
df = pd.read_csv('data.csv')

# Quick overview
df.head()           # first 5 rows
df.tail(10)         # last 10
df.shape            # (rows, cols)
df.dtypes           # column data types
df.info()           # types + null counts
df.describe()       # summary statistics
```

### Selecting Data

```python
# Column selection
df['age']           # Series
df[['age', 'income']]  # DataFrame with two columns

# Row selection by label (.loc) and by position (.iloc)
df.loc[0]           # row with label 0
df.loc[0:2]         # rows 0, 1, 2 (inclusive!)
df.iloc[0]          # first row (position-based)
df.iloc[0:2]        # rows 0, 1 (exclusive end)

# Boolean filtering
df[df['age'] > 28]
df[(df['age'] > 28) & (df['income'] > 50000)]

# .query() — readable string-based filtering
df.query('age > 28 and income > 50000')
```

### Handling Missing Data

```python
df.isnull().sum()           # count NaNs per column
df.dropna()                 # drop rows with any NaN
df.dropna(subset=['age'])   # only if 'age' is NaN
df.fillna(0)                # fill NaN with 0
df['age'].fillna(df['age'].mean())  # fill with mean

# More sophisticated: sklearn's SimpleImputer (see Ch 8)
```

### Data Manipulation

```python
# New column
df['income_k'] = df['income'] / 1000

# Apply a function
df['age_group'] = df['age'].apply(lambda x: 'young' if x < 30 else 'senior')

# GroupBy — split, apply, combine
df.groupby('purchased')['income'].mean()
df.groupby('age_group').agg({'income': ['mean', 'std'], 'age': 'count'})

# Sorting
df.sort_values('income', ascending=False)

# Merging (like SQL JOIN)
pd.merge(df1, df2, on='user_id', how='inner')
pd.concat([df1, df2], axis=0)   # stack vertically
```

### Encoding and Type Conversion

```python
# Convert string to category (memory efficient)
df['city'] = df['city'].astype('category')

# One-hot encoding
pd.get_dummies(df, columns=['city'])

# Converting to NumPy for ML
X = df[['age', 'income']].values   # numpy array
y = df['purchased'].values
```

---

## 2.4 Matplotlib & Seaborn

Visualization is how you understand your data before modeling.

### Matplotlib Basics

```python
import matplotlib.pyplot as plt

# Line plot
x = np.linspace(0, 10, 100)
plt.figure(figsize=(8, 4))
plt.plot(x, np.sin(x), label='sin(x)', color='blue', linewidth=2)
plt.plot(x, np.cos(x), label='cos(x)', color='red', linestyle='--')
plt.xlabel('x')
plt.ylabel('y')
plt.title('Trig Functions')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()

# Scatter plot
plt.scatter(df['age'], df['income'], c=df['purchased'], cmap='RdYlGn', alpha=0.7)
plt.colorbar(label='purchased')

# Histogram
plt.hist(df['income'], bins=20, edgecolor='black')

# Subplots
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(x, np.sin(x))
axes[1].hist(np.random.randn(1000), bins=30)
```

### Seaborn for Statistical Visualization

```python
import seaborn as sns

# Distribution
sns.histplot(df['income'], kde=True)        # histogram + density

# Relationships
sns.scatterplot(data=df, x='age', y='income', hue='purchased')
sns.pairplot(df, hue='purchased')           # all pairs at once

# Correlation heatmap (essential for EDA)
corr = df.corr()
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm', center=0)

# Box plots
sns.boxplot(data=df, x='purchased', y='income')

# Regression line
sns.regplot(data=df, x='age', y='income')
```

---

## 2.5 Scikit-learn Overview

Scikit-learn has a consistent, unified API. Understanding the pattern means you can use any algorithm with minimal new syntax.

### The Estimator API

```python
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# 1. Split data
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 2. Preprocess
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # fit + transform
X_test_scaled = scaler.transform(X_test)         # transform ONLY (never fit on test!)

# 3. Train
model = LogisticRegression(max_iter=1000)
model.fit(X_train_scaled, y_train)

# 4. Evaluate
y_pred = model.predict(X_test_scaled)
print(accuracy_score(y_test, y_pred))
```

**Critical rule:** Never `fit` any preprocessor on the test set. It leaks information about the test distribution into your model.

### Pipelines

Pipelines chain preprocessing + model into one object. They prevent data leakage automatically.

```python
from sklearn.pipeline import Pipeline

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])

pipe.fit(X_train, y_train)
pipe.predict(X_test)   # automatically scales then predicts
```

Pipelines are the correct way to build ML workflows. Use them always.

### Cross-Validation

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(pipe, X, y, cv=5, scoring='accuracy')
print(f"CV accuracy: {scores.mean():.3f} ± {scores.std():.3f}")
```

### Quick Model Survey

```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso, LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.cluster import KMeans
from sklearn.decomposition import PCA
```

All share `.fit()`, `.predict()`, and (for classifiers) `.predict_proba()`.

---

## 2.6 Jupyter Notebooks

Jupyter is the standard environment for ML experimentation.

```bash
jupyter notebook    # classic interface
jupyter lab         # modern interface (recommended)
```

**Key shortcuts (essential):**
| Shortcut | Action |
|----------|--------|
| `Shift+Enter` | Run cell, move to next |
| `Ctrl+Enter` | Run cell, stay |
| `A` (command mode) | Insert cell above |
| `B` (command mode) | Insert cell below |
| `DD` (command mode) | Delete cell |
| `M` (command mode) | Convert to Markdown |
| `Y` (command mode) | Convert to Code |
| `Esc` | Enter command mode |

**Good notebook habits:**
- One notebook per experiment
- Restart and run all before sharing
- Use markdown cells to document your thinking
- Keep cells short and focused
- Name notebooks with dates: `2024-01-15_feature_experiment.ipynb`

---

## 2.7 Exploratory Data Analysis (EDA) Template

EDA is the first step with any new dataset. Use this template:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv('your_data.csv')

# ── Shape and types ──────────────────────────────────────────
print(df.shape)
print(df.dtypes)
print(df.head())

# ── Missing values ───────────────────────────────────────────
missing = df.isnull().sum()
missing_pct = (missing / len(df) * 100).sort_values(ascending=False)
print(missing_pct[missing_pct > 0])

# ── Target distribution ──────────────────────────────────────
target = 'your_target_column'
df[target].value_counts(normalize=True).plot(kind='bar')
plt.title('Target Distribution')
plt.show()

# ── Numerical features ───────────────────────────────────────
num_cols = df.select_dtypes(include=np.number).columns.tolist()
df[num_cols].describe()

fig, axes = plt.subplots(len(num_cols), 2, figsize=(12, 4*len(num_cols)))
for i, col in enumerate(num_cols):
    df[col].hist(ax=axes[i,0], bins=30)
    axes[i,0].set_title(col)
    sns.boxplot(data=df, y=col, ax=axes[i,1])
plt.tight_layout()
plt.show()

# ── Correlation matrix ───────────────────────────────────────
corr = df[num_cols].corr()
plt.figure(figsize=(10, 8))
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm', center=0)
plt.title('Feature Correlations')
plt.show()

# ── Categorical features ─────────────────────────────────────
cat_cols = df.select_dtypes(include='object').columns.tolist()
for col in cat_cols:
    print(f"\n{col}: {df[col].nunique()} unique values")
    print(df[col].value_counts().head(10))
```

---

## Exercises

1. **NumPy broadcasting:** Without loops, compute the pairwise Euclidean distance matrix between all rows of a matrix `X ∈ ℝ^(100×5)`. The result should be 100×100. Hint: `‖xᵢ - xⱼ‖² = ‖xᵢ‖² + ‖xⱼ‖² - 2xᵢᵀxⱼ`.

2. **Pandas pipeline:** Download the Titanic dataset from Kaggle. Perform full EDA: missing values, distributions, correlations, survival rate by feature. Write a 5-bullet summary of what you find.

3. **Data leakage experiment:** Fit a `StandardScaler` on the entire dataset X (before splitting), then split. Compare test accuracy to the correct approach (fit scaler only on train). Do you see any difference? Why or why not?

4. **Visualization:** Using any dataset, create a figure with 4 subplots: histogram, boxplot, scatter plot, and heatmap. Make it publication-quality (proper labels, titles, font sizes).

5. **Pipeline benchmark:** Take any classification dataset. Build two identical pipelines (scaler + LogisticRegression) — one with a Pipeline object, one manually. Verify they give identical predictions.

---

## Further Reading

- NumPy documentation — `numpy.org/doc` (especially the broadcasting section)
- Pandas documentation — `pandas.pydata.org` (user guide)
- *Python for Data Analysis* — Wes McKinney (creator of Pandas)
- Matplotlib gallery — for plot inspiration and copy-paste templates
