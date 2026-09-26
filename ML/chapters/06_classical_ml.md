# Chapter 6 — Classical ML: SVMs, KNN, Naive Bayes, Clustering, PCA

> *These algorithms round out the classical toolkit. Each embodies a distinct inductive bias. Understanding when each one is appropriate is as important as knowing how they work.*

---

## 6.1 Support Vector Machines (SVMs)

### The Idea: Maximum Margin

For linearly separable binary classification, there are infinitely many hyperplanes that correctly separate the classes. SVM picks the one with the **maximum margin** — the widest "road" between the two classes.

```
     Class -1          |margin|         Class +1
     ×  ×              |      |              o  o
        ×   ×  ×  — — — — — — — — — o   o
                       |      |         o
                   Decision Boundary
```

Intuition: larger margin → more robust to new points → better generalization.

### Hard-Margin SVM

For a hyperplane `wᵀx + b = 0`:
- Points on one side: `wᵀx + b > 0` (class +1)
- Points on other side: `wᵀx + b < 0` (class -1)

The margin width = `2/‖w‖`. Maximizing margin = minimizing `‖w‖²`.

**Optimization problem:**
```
minimize   (1/2) ‖w‖²
subject to yᵢ(wᵀxᵢ + b) ≥ 1  for all i
```

The training points closest to the decision boundary are **support vectors**. Only these matter — removing other points doesn't change the solution.

### Soft-Margin SVM (C-SVM)

Real data isn't linearly separable. Introduce slack variables ξᵢ ≥ 0 to allow misclassifications:

```
minimize   (1/2)‖w‖² + C Σᵢ ξᵢ
subject to yᵢ(wᵀxᵢ + b) ≥ 1 - ξᵢ
           ξᵢ ≥ 0
```

`C` controls the tradeoff:
- Large C: smaller margin, fewer misclassifications (low bias, high variance)
- Small C: larger margin, more misclassifications allowed (high bias, low variance)

### The Kernel Trick

SVMs naturally extend to non-linear boundaries via the **kernel trick**.

The dual form of SVM only involves dot products: `xᵢᵀxⱼ`. We can replace this with a kernel function `K(xᵢ, xⱼ) = φ(xᵢ)ᵀφ(xⱼ)` — which computes the dot product in a (potentially infinite) feature space without explicitly constructing `φ(x)`.

**Common kernels:**
```
Linear:          K(x, z) = xᵀz
Polynomial:      K(x, z) = (xᵀz + c)ᵈ
RBF (Gaussian):  K(x, z) = exp(-γ‖x - z‖²)
Sigmoid:         K(x, z) = tanh(κxᵀz + c)
```

The **RBF kernel** is the most common and powerful default. It maps data to an infinite-dimensional feature space where any boundary is achievable.

### SVMs in Practice

```python
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

# Feature scaling is CRITICAL for SVMs
svm_pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(
        kernel='rbf',    # 'linear', 'poly', 'rbf', 'sigmoid'
        C=1.0,           # regularization
        gamma='scale',   # 'scale' = 1/(n_features * X.var())
        probability=True # enable predict_proba (slower)
    ))
])

svm_pipe.fit(X_train, y_train)
```

**Hyperparameter tuning (C and gamma together):**
```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'svm__C': [0.1, 1, 10, 100],
    'svm__gamma': [1, 0.1, 0.01, 0.001]
}
grid = GridSearchCV(svm_pipe, param_grid, cv=5, scoring='accuracy')
grid.fit(X_train, y_train)
```

**SVMs scale as O(n²) to O(n³) in training** — not suitable for large datasets (n > 100k). Use `LinearSVC` or logistic regression instead.

### SVR (Support Vector Regression)

SVMs also work for regression. Instead of maximum margin, we fit a tube of width ε around the predictions. Points inside the tube have zero loss.

---

## 6.2 k-Nearest Neighbors (k-NN)

### Algorithm

The simplest non-parametric classifier:

1. Store all training data
2. For a new point x, find its k nearest training points by distance (usually L2)
3. Classify by majority vote (or weighted by distance)

**No training phase!** The work is all at inference time — this is called a **lazy learner**.

**Time complexity:** O(1) training, O(n·d) per prediction (brute force). Use KD-trees or Ball trees for better scaling.

### Choosing k

```
k=1:        Extremely wiggly decision boundary, high variance (memorizes)
k=n:        Constant prediction (always predicts majority class), high bias
Optimal k:  Somewhere in between. Use CV, often 5–20 for medium datasets
```

Rule of thumb: start with `k = √n`.

As k increases: smoother boundary, lower variance, higher bias.

### Distance Metrics

```python
from sklearn.neighbors import KNeighborsClassifier

knn = KNeighborsClassifier(
    n_neighbors=5,
    metric='euclidean',    # 'manhattan', 'minkowski', 'cosine'
    weights='uniform',     # or 'distance' (weight by 1/distance)
    algorithm='auto'       # 'ball_tree', 'kd_tree', 'brute'
)
```

**Critical:** Feature scaling is essential. With unscaled features, high-range features dominate the distance calculation.

### When to Use k-NN

- Small to medium datasets
- Non-linear boundaries (adapts to any shape)
- When model interpretability isn't needed
- As a baseline or sanity check

**Curse of dimensionality:** In high dimensions, all points are approximately equidistant, making k-NN unreliable. Feature selection or dimensionality reduction is important.

---

## 6.3 Naive Bayes

### The Generative Approach

Naive Bayes is a **generative** classifier. It models the joint distribution `P(x, y)` and uses Bayes theorem to compute `P(y|x)`.

From Bayes theorem:
```
P(y | x) = P(x | y) · P(y) / P(x)
```

To classify, we pick: `ŷ = argmax_y P(y | x) = argmax_y P(x | y) · P(y)`

(P(x) is the same for all classes, so it cancels.)

### The "Naive" Assumption

Computing `P(x | y)` for high-dimensional x is intractable. The naive assumption: **features are conditionally independent given the class**.

```
P(x | y) = Π_j P(xⱼ | y)
```

This simplifies enormously — each feature's distribution is estimated independently.

**This is wrong!** Features are not truly independent. But Naive Bayes works surprisingly well anyway, because:
1. For classification, we just need relative probabilities (not accurate ones)
2. In high-dimensional sparse settings (text), the bias is small

### Variants

**Gaussian NB:** Assumes `P(xⱼ | y) = N(μⱼy, σ²ⱼy)` — best for continuous features.

**Multinomial NB:** Assumes multinomial distribution — best for text (word counts).

**Bernoulli NB:** Assumes binary features — best for text (word presence/absence).

```python
from sklearn.naive_bayes import GaussianNB, MultinomialNB, BernoulliNB
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline

# For text classification
text_clf = Pipeline([
    ('tfidf', TfidfVectorizer()),
    ('clf', MultinomialNB(alpha=1.0))  # alpha: Laplace smoothing
])
text_clf.fit(X_train_text, y_train)
```

### Laplace Smoothing

If a word never appears in the training data for class k, `P(word | class k) = 0`, and the entire product is 0. Laplace smoothing adds `α` pseudo-counts:
```
P(xⱼ = v | y = k) = (count(xⱼ=v, y=k) + α) / (count(y=k) + α·|V|)
```

**Strengths:** Very fast, works well for text, excels with small data, interpretable.
**Weaknesses:** Independence assumption, poor probability estimates.

---

## 6.4 Clustering

Clustering is unsupervised — we find structure without labels.

### K-Means

Partition n samples into K clusters by minimizing within-cluster sum of squared distances:
```
argmin_{C} Σₖ Σ_{x∈Cₖ} ‖x - μₖ‖²
```

**Lloyd's Algorithm:**
1. Initialize K centroids (randomly or with K-Means++)
2. Assign each point to its nearest centroid
3. Update centroids as mean of assigned points
4. Repeat 2-3 until convergence

**K-Means++ initialization:** Choose each new centroid with probability proportional to its squared distance from existing centroids. Dramatically reduces bad initializations.

```python
from sklearn.cluster import KMeans

kmeans = KMeans(
    n_clusters=5,
    init='k-means++',   # default, better than 'random'
    n_init=10,          # number of restarts (take best)
    max_iter=300,
    random_state=42
)
labels = kmeans.fit_predict(X)
centers = kmeans.cluster_centers_
inertia = kmeans.inertia_   # within-cluster sum of squares
```

**Choosing K — The Elbow Method:**
```python
inertias = []
K_range = range(1, 11)
for k in K_range:
    km = KMeans(n_clusters=k, n_init=10, random_state=42)
    km.fit(X)
    inertias.append(km.inertia_)

plt.plot(K_range, inertias, marker='o')
plt.xlabel('Number of clusters K')
plt.ylabel('Inertia')
plt.title('Elbow Method for Optimal K')
```

Look for the "elbow" where inertia stops decreasing rapidly. Supplemented by **Silhouette score** (higher is better):
```python
from sklearn.metrics import silhouette_score
score = silhouette_score(X, labels)
```

**Silhouette score:** For each point, measures how similar it is to its own cluster vs. the nearest other cluster. Range [-1, 1]. Higher = better-separated clusters.

**Weaknesses of K-Means:**
- Must specify K in advance
- Assumes spherical, equal-sized clusters
- Sensitive to outliers
- Fails on non-convex shapes

### DBSCAN

Density-Based Spatial Clustering of Applications with Noise.

**Key concepts:**
- **Core point:** Has at least `min_samples` points within `eps` radius
- **Border point:** Within eps of a core point, but not itself a core point
- **Noise/Outlier:** Not a core point or reachable from one

```python
from sklearn.cluster import DBSCAN

db = DBSCAN(
    eps=0.5,          # neighborhood radius
    min_samples=5     # min points to be a core point
)
labels = db.fit_predict(X)
# Labels: -1 = outlier, 0,1,2,... = cluster IDs
```

**Advantages over K-Means:**
- No need to specify K
- Can find arbitrarily shaped clusters
- Identifies outliers explicitly
- Robust to noise

**Weaknesses:** Struggles with varying density clusters, sensitive to `eps` and `min_samples`.

### Hierarchical Clustering

Builds a tree (dendrogram) of cluster merges. No need to specify K — choose the level of the tree.

**Agglomerative (bottom-up):** Each point starts as its own cluster, then merge closest pairs repeatedly.

**Linkage criteria:**
- Single: min distance between any two points in the clusters
- Complete: max distance (makes compact clusters)
- Average: mean distance
- Ward: minimize increase in total within-cluster variance (usually best)

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage

Z = linkage(X, method='ward')
plt.figure(figsize=(12, 5))
dendrogram(Z, truncate_mode='level', p=5)
plt.title('Dendrogram')
plt.show()

# Cut to get K clusters
agg = AgglomerativeClustering(n_clusters=5, linkage='ward')
labels = agg.fit_predict(X)
```

---

## 6.5 Principal Component Analysis (PCA)

PCA is the most widely used dimensionality reduction technique.

### The Goal

Find a low-dimensional linear subspace that captures maximum variance in the data.

Project d-dimensional data onto k dimensions (k << d) while retaining as much information as possible.

### The Math

1. Center the data: `X_centered = X - mean(X, axis=0)`
2. Compute the covariance matrix: `C = (1/n) X_centeredᵀ X_centered`
3. Compute eigenvectors of C: `Cv = λv`
4. Sort eigenvectors by eigenvalue (descending)
5. Project: `X_pca = X_centered @ V_k` where V_k = top k eigenvectors

**Equivalently (and numerically better):** Use SVD on the centered data:
```
X_centered = UΣVᵀ
X_pca = X_centered @ V_k = U_k Σ_k
```

The **principal components** (columns of V_k) are the directions of maximum variance.

### Explained Variance Ratio

```python
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt

pca = PCA()  # compute all components first
pca.fit(X_train)

# Plot explained variance
plt.figure(figsize=(8, 4))
plt.plot(np.cumsum(pca.explained_variance_ratio_))
plt.xlabel('Number of Components')
plt.ylabel('Cumulative Explained Variance')
plt.axhline(0.95, color='red', linestyle='--', label='95%')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()

# Choose k to explain 95% variance
pca_95 = PCA(n_components=0.95)  # sklearn accepts fraction!
X_reduced = pca_95.fit_transform(X_train)
print(f"Reduced from {X_train.shape[1]} to {X_reduced.shape[1]} dimensions")
```

### Visualizing High-Dimensional Data

Use PCA to 2D for visualization:
```python
pca_2d = PCA(n_components=2)
X_2d = pca_2d.fit_transform(X)

plt.scatter(X_2d[:, 0], X_2d[:, 1], c=y, cmap='viridis', alpha=0.7)
plt.xlabel('PC1')
plt.ylabel('PC2')
plt.colorbar(label='Class')
plt.title('PCA — 2D Projection')
plt.show()
```

### PCA for Noise Reduction

Reconstruct data after keeping only the top k components:
```python
pca_k = PCA(n_components=50)
X_reconstructed = pca_k.inverse_transform(pca_k.fit_transform(X))
```

### t-SNE (t-Distributed Stochastic Neighbor Embedding)

t-SNE is a non-linear dimensionality reduction technique, almost exclusively used for visualization (2D/3D).

**Key idea:** Preserve local structure — points close in high-dim space should be close in 2D. Uses Student's t-distribution in the low-dim space (fat tails prevent crowding).

```python
from sklearn.manifold import TSNE

tsne = TSNE(
    n_components=2,
    perplexity=30,     # effective number of neighbors (~5–50)
    learning_rate=200,
    n_iter=1000,
    random_state=42
)
X_tsne = tsne.fit_transform(X)
```

**Caution:** t-SNE is NOT suitable for:
- New data points (must re-run on full dataset)
- Distance between clusters (distances are not meaningful)
- Dimensionality reduction before ML (use PCA instead)

Use t-SNE only for visualization. Use PCA for feature reduction before ML.

### UMAP

UMAP (Uniform Manifold Approximation and Projection) is a modern alternative to t-SNE:
- Faster than t-SNE
- Preserves more global structure
- Can be applied to new data (has a `transform()` method)
- Also excellent for visualization

```python
import umap
reducer = umap.UMAP(n_components=2, n_neighbors=15, min_dist=0.1)
X_umap = reducer.fit_transform(X)
```

---

## Exercises

1. **SVM kernel visualization:** Generate a dataset with two concentric rings (not linearly separable). Show that a linear SVM fails, then apply RBF SVM. Plot the decision boundaries for both. Explain how the kernel trick enables the non-linear boundary.

2. **K-Means from scratch:** Implement K-Means in NumPy. Test on the Iris dataset. Compute purity (fraction of cluster that belongs to the majority class) for each cluster.

3. **Clustering comparison:** On a dataset with moon-shaped clusters, run K-Means, DBSCAN, and hierarchical clustering. Plot the results. Which algorithm finds the true clusters?

4. **PCA reconstruction:** Apply PCA to MNIST digit images. Show the first 10 principal components as 28×28 images. Reconstruct a digit using 5, 20, 50, 100, and 784 components. At what k does it look like a digit?

5. **t-SNE visualization:** Apply t-SNE to MNIST (or CIFAR-10). Color by class label. Do digits cluster together? Try perplexity values of 5, 30, and 100. How does perplexity affect the visualization?

---

## Further Reading

- *The Elements of Statistical Learning* — Hastie et al. — Chapter 12 (SVMs), Chapter 13 (Nearest Neighbors), Chapter 14 (Unsupervised Learning)
- *A Tutorial on Support Vector Machines for Pattern Recognition* — Burges (1998) — excellent SVM tutorial
- *Visualizing Data using t-SNE* — van der Maaten & Hinton (2008) — original t-SNE paper
- *UMAP: Uniform Manifold Approximation and Projection* — McInnes et al. (2018)
