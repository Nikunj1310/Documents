# Cheat Sheet — NumPy & Pandas

> Quick reference for the most-used operations. No explanations — use the chapter notes for those.

---

## NumPy

### Array Creation
```python
np.array([1, 2, 3])              # from list
np.zeros((m, n))                 # zeros
np.ones((m, n))                  # ones
np.eye(n)                        # identity
np.arange(start, stop, step)     # range
np.linspace(start, stop, n)      # evenly spaced
np.random.randn(m, n)            # N(0,1)
np.random.rand(m, n)             # uniform [0,1)
np.random.randint(low, high, (m,n))

rng = np.random.default_rng(42)  # seeded RNG (preferred)
rng.standard_normal((m, n))
```

### Shape & Info
```python
a.shape       # (m, n)
a.ndim        # 2
a.size        # m*n
a.dtype       # dtype('float64')
a.T           # transpose
```

### Indexing
```python
a[0]           # first row
a[-1]          # last row
a[:, 1]        # second column
a[1:3, 0:2]    # slice rows 1-2, cols 0-1
a[::2]         # every other row
a[[0,2,4]]     # fancy indexing
a[a > 0]       # boolean mask
np.where(a > 0, a, 0)  # conditional
```

### Reshaping
```python
a.reshape(m, n)      # new shape (must match size)
a.reshape(-1, n)     # infer first dim
a.ravel()            # flatten (view when possible)
a.flatten()          # flatten (always copy)
a[:, None]           # add axis → (m,) to (m,1)
a[None, :]           # add axis → (n,) to (1,n)
np.expand_dims(a, 0) # add axis at position 0
np.squeeze(a)        # remove size-1 dimensions
```

### Math Operations
```python
a + b / a * b / a - b   # elementwise
a @ b                    # matrix multiply
np.dot(a, b)             # dot product / matmul
a.T                      # transpose
np.linalg.inv(A)         # inverse
np.linalg.solve(A, b)    # Ax=b (prefer over inv)
np.linalg.norm(a)        # L2 norm
np.linalg.norm(a, ord=1) # L1 norm
np.linalg.eig(A)         # eigenvalues, eigenvectors
np.linalg.svd(A)         # SVD: U, s, Vt

np.sqrt(a)  np.exp(a)  np.log(a)  np.log1p(a)
np.abs(a)   np.sign(a) np.clip(a, lo, hi)
```

### Reductions
```python
a.sum()          a.sum(axis=0)    # sum all / per column
a.mean()         a.mean(axis=1)   # mean all / per row
a.std()          a.var()
a.min()          a.max()
a.argmin()       a.argmax()
a.cumsum(axis=0)                  # cumulative sum
np.unique(a)
np.sort(a)       np.argsort(a)    # values / indices
```

### Stacking & Splitting
```python
np.concatenate([a, b], axis=0)   # vertical stack
np.concatenate([a, b], axis=1)   # horizontal stack
np.vstack([a, b])                # same as axis=0
np.hstack([a, b])                # same as axis=1
np.stack([a, b], axis=0)         # new axis
np.split(a, 3)                   # split into 3 equal parts
np.split(a, [2, 5])              # split at indices
```

### Useful Tricks
```python
np.c_[a, b]              # column-wise concat (adds bias col: np.c_[np.ones(n), X])
np.r_[a, b]              # row-wise concat
np.tile(a, (m, n))       # repeat array m×n times
np.repeat(a, 3, axis=0)  # repeat each element 3 times

# Pairwise distances (efficient, no loops)
diff = X[:, None, :] - X[None, :, :]  # (n, n, d)
dists = np.linalg.norm(diff, axis=-1)  # (n, n)

# Or: ||xi - xj||² = ||xi||² + ||xj||² - 2 xi·xj
sq = (X**2).sum(1)
dists_sq = sq[:, None] + sq[None, :] - 2 * X @ X.T
```

---

## Pandas

### Reading & Writing
```python
df = pd.read_csv('file.csv')
df = pd.read_csv('file.csv', sep='\t', encoding='utf-8', 
                  usecols=['a','b'], dtype={'a': float}, parse_dates=['date'])
df = pd.read_parquet('file.parquet')    # faster than CSV
df = pd.read_json('file.json')

df.to_csv('out.csv', index=False)
df.to_parquet('out.parquet')
```

### Inspection
```python
df.head()        df.tail(10)
df.shape         df.dtypes       df.info()
df.describe()    df.describe(include='all')
df.columns.tolist()
df.index
```

### Selection
```python
df['col']                        # Series
df[['col1', 'col2']]            # DataFrame
df.loc[row_label]               # by label (inclusive both ends)
df.iloc[0:5]                    # by integer position (exclusive end)
df.loc[0:3, 'col1':'col3']      # label slice
df.iloc[0:3, 0:3]               # position slice

# Boolean
df[df['age'] > 30]
df.query('age > 30 and income < 50000')
df[df['city'].isin(['NYC', 'LA'])]
df[~df['col'].isnull()]
```

### Missing Values
```python
df.isnull().sum()
df.isnull().sum() / len(df) * 100   # % missing per column
df.dropna()                          # drop rows with any NaN
df.dropna(subset=['col'])            # drop only if 'col' is NaN
df.dropna(thresh=5)                  # keep rows with at least 5 non-NaN
df.fillna(0)
df['col'].fillna(df['col'].median())
df.fillna(method='ffill')            # forward fill (time series)
```

### Manipulation
```python
# New/modify columns
df['new_col'] = df['a'] + df['b']
df.assign(ratio=df['a']/df['b'])         # method-chaining friendly
df.apply(lambda x: x**2)                 # apply function to each col
df['col'].apply(lambda x: x.lower())     # apply to series

# Renaming
df.rename(columns={'old': 'new'})
df.columns = [c.lower() for c in df.columns]

# Sorting
df.sort_values('col', ascending=False)
df.sort_values(['col1', 'col2'], ascending=[True, False])

# Drop
df.drop(columns=['col1', 'col2'])
df.drop_duplicates()
df.drop_duplicates(subset=['user_id'])
```

### GroupBy
```python
df.groupby('city')['sales'].mean()
df.groupby('city')['sales'].agg(['mean', 'std', 'count'])
df.groupby(['city', 'year'])['sales'].sum()
df.groupby('city').transform('mean')   # broadcast back to original index
df.groupby('city').apply(custom_func)
```

### Merging & Reshaping
```python
pd.merge(left, right, on='key', how='inner')  # how: inner/left/right/outer
pd.merge(left, right, left_on='a', right_on='b')
pd.concat([df1, df2], axis=0, ignore_index=True)  # stack rows
pd.concat([df1, df2], axis=1)                      # stack columns

df.pivot(index='date', columns='city', values='sales')
df.melt(id_vars=['date'], var_name='city', value_name='sales')
df.pivot_table(index='city', columns='year', values='sales', aggfunc='mean')
```

### Type Conversion
```python
df['age'] = df['age'].astype(int)
df['date'] = pd.to_datetime(df['date'])
df['category'] = df['category'].astype('category')  # memory efficient

# Extract from datetime
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day_of_week'] = df['date'].dt.dayofweek
df['hour'] = df['date'].dt.hour
```

### Useful Utilities
```python
df['col'].value_counts()
df['col'].value_counts(normalize=True)   # proportions
df['col'].nunique()
df['col'].unique()

df.sample(100, random_state=42)           # random sample
df.sample(frac=0.1, random_state=42)      # 10% sample

pd.get_dummies(df, columns=['city'])      # one-hot encoding
df['col'].map({'yes': 1, 'no': 0})        # value mapping
df['col'].str.contains('pattern')         # string operations
df['col'].str.lower().str.strip()

df.corr()                                  # correlation matrix
df.nlargest(10, 'col')                     # top 10 by column
df.nsmallest(5, 'col')
```
