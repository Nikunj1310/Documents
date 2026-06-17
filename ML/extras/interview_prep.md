# Interview Preparation — ML Theory & Coding

> Covers the questions you'll face in ML engineer, data scientist, and research scientist interviews. Each answer here is a starting point — expand by studying the relevant chapter.

---

## Part 1 — Theory Questions

### Fundamentals

**Q: What is the bias-variance tradeoff?**

Bias is systematic error from model assumptions too simple to capture the true function. Variance is sensitivity to fluctuations in training data — the model memorizes noise. Total error = Bias² + Variance + Irreducible Noise. You cannot minimize both simultaneously: simpler models have high bias and low variance, complex models have low bias and high variance. The goal is to minimize the sum, usually found via cross-validation to choose model complexity.

---

**Q: What is overfitting? How do you detect and fix it?**

Overfitting is when a model learns the training data (including noise) too well and fails to generalize. Detected by: train error << test error, or learning curve showing large gap between training and validation scores. Fixes: more data, regularization (L1/L2/dropout), simpler model, early stopping, data augmentation.

---

**Q: Explain the difference between L1 and L2 regularization.**

Both add a penalty to the loss. L2 (Ridge) adds `λ‖w‖²` — pushes all weights toward zero but keeps them all non-zero, giving a dense solution. L1 (Lasso) adds `λ‖w‖₁` — due to the sharp corners of the L1 ball, solutions tend to be sparse (many weights exactly zero), acting as automatic feature selection. L1 is preferred when you believe most features are irrelevant. ElasticNet combines both.

---

**Q: What is the curse of dimensionality?**

In high dimensions: (1) the volume of space grows exponentially, so data becomes sparse. (2) All points become approximately equidistant, destroying the notion of "nearby" needed by k-NN and clustering. (3) Exponentially more data is needed to cover the space adequately. Solutions: dimensionality reduction (PCA), feature selection, kernel methods that implicitly operate in high-dim space without materializing it.

---

**Q: Why does gradient descent work? What can go wrong?**

Gradient descent exploits the fact that the negative gradient is the direction of steepest decrease in loss. Under mild conditions (smooth loss, small enough step size), iterating improves the solution. Problems: (1) learning rate too large → diverges or oscillates; (2) saddle points → gradient is zero, but not a minimum; (3) flat regions → tiny gradients → slow progress; (4) non-convex losses → converges to local minimum. Practical fixes: Adam (adaptive rates), learning rate schedules, momentum, careful initialization.

---

**Q: What is the difference between generative and discriminative models?**

Discriminative models learn `P(y|x)` — the decision boundary. Examples: logistic regression, SVM, BERT-classifier. Faster to train, usually better accuracy on the classification task. Generative models learn `P(x,y) = P(x|y)P(y)` — the full joint distribution. Examples: Naive Bayes, GMM, VAE. Can generate new data, handle missing features naturally, and are better with small data (priors help). Logistic regression vs Naive Bayes is the classic comparison.

---

**Q: Explain cross-validation. Why not just use a train/test split?**

k-Fold CV splits data into k folds, trains on k-1, validates on the remaining fold, repeats k times. Advantages over single split: uses all data for both training and validation (important with small datasets), provides variance estimate of generalization error, less sensitive to how data happens to be split. Disadvantage: k× more computation. Use stratified k-Fold for classification to preserve class ratios.

---

**Q: What is data leakage? Give an example.**

Data leakage occurs when information from outside the training set illegitimately flows into the model during training, causing inflated performance metrics that don't reflect real-world performance. Example: fitting a StandardScaler on the full dataset before splitting — the scaler has "seen" the test set statistics, which are technically unknowable at training time. Fix: always fit preprocessing within the training fold only, using sklearn Pipelines.

---

### Decision Trees & Ensembles

**Q: How does a decision tree split? What criterion?**

For classification: minimize Gini impurity or entropy (information gain). For regression: minimize MSE. At each node, search all features and all thresholds to find the split that maximally reduces the chosen criterion. CART (Classification And Regression Trees) is the standard algorithm.

---

**Q: Why do random forests outperform single decision trees?**

Bagging reduces variance by averaging multiple high-variance models. Random forests add an additional de-correlation step: at each split, only a random subset of features (√d for classification) is considered. This forces diversity among trees — different trees specialize in different patterns. The ensemble's error is the average error of individual trees, and correlations between trees limit how much diversity helps, so the random feature subsets are crucial.

---

**Q: What is gradient boosting? How does it differ from bagging?**

Bagging trains models in parallel on random bootstrap samples and averages. Boosting trains sequentially — each new model focuses on the errors of the current ensemble. Gradient boosting specifically fits each new tree to the pseudo-residuals (negative gradient of the loss). The ensemble improves iteratively, reducing both bias and variance. Gradient boosting tends to outperform random forests on tabular data but is more sensitive to hyperparameters and doesn't parallelize as naturally.

---

### Neural Networks

**Q: Explain backpropagation.**

Backpropagation efficiently computes gradients of the loss with respect to all parameters using the chain rule. In the forward pass, intermediate values are computed and cached. In the backward pass, gradients flow from output to input: the error signal `δ[l] = (W[l+1]ᵀ δ[l+1]) ⊙ f'(z[l])` is propagated backward, and parameter gradients `∂L/∂W[l] = δ[l] (a[l-1])ᵀ` are computed efficiently. The key insight: gradients at earlier layers can be computed by reusing gradients from later layers.

---

**Q: Why do we need activation functions? What is the vanishing gradient problem?**

Without non-linear activations, stacking layers is equivalent to a single linear transformation (composition of linear functions is linear). Activations introduce non-linearity needed to represent complex functions. Vanishing gradients: with sigmoid/tanh activations, gradients are multiplied by `σ'(z) ≤ 0.25` at each layer. After many layers, gradients become exponentially small, and early layers learn extremely slowly. ReLU alleviates this (gradient = 1 for positive inputs) but can cause dying ReLU (neurons stuck at 0 forever).

---

**Q: What is batch normalization and why does it help?**

Batch norm normalizes layer inputs within each mini-batch: `x̂ = (x - μ_B) / √(σ²_B + ε)`, then rescales with learnable parameters `y = γx̂ + β`. Benefits: (1) smooths the loss landscape, allowing higher learning rates and faster training; (2) reduces dependence on initialization; (3) acts as mild regularization. The debate: why exactly it works is still somewhat open (the original "internal covariate shift" explanation is disputed).

---

**Q: What is the difference between BERT and GPT?**

Both are Transformers. BERT uses an encoder-only architecture with bidirectional (masked) self-attention — each token can attend to all others. Trained with masked language modeling (MLM). Best for understanding tasks (classification, NER, QA). GPT uses a decoder-only architecture with causal (left-to-right) self-attention — each token only attends to previous tokens. Trained with next-token prediction. Best for generation tasks. Modern LLMs are all decoder-only (GPT-style).

---

**Q: What is attention and why is it important?**

Attention allows a model to selectively weight different parts of the input when computing each output element. Scaled dot-product attention: `Attention(Q,K,V) = softmax(QKᵀ/√dₖ)V`. Importance: (1) handles variable-length sequences with O(1) distance between any two positions; (2) replaces recurrence in RNNs, enabling full parallelization; (3) learns which parts of the input are relevant for each output; (4) interpretable (attention weights show what the model attended to).

---

### Evaluation & Practical

**Q: When would you use AUC-ROC vs Precision-Recall?**

AUC-ROC is appropriate when the class distribution is roughly balanced. It measures the model's ability to rank positives above negatives. PR-AUC (average precision) is better for highly imbalanced datasets — where the rare positive class is what matters. In fraud detection (0.1% positive rate), PR-AUC reveals performance differences that AUC-ROC masks. If you care about the minority class, use PR.

---

**Q: How do you handle imbalanced datasets?**

In order of preference: (1) Use the right metric (not accuracy). (2) Tune the classification threshold using the ROC or PR curve. (3) Use `class_weight='balanced'` to scale the loss by class frequency. (4) SMOTE (synthetic minority oversampling) — only on training data to avoid leakage. (5) Undersample majority class. (6) Ensemble with balanced subsamples. The root cause also matters: if imbalance reflects reality, the model should reflect it.

---

**Q: Your model performs well in offline evaluation but poorly in production. What could be wrong?**

(1) **Data leakage** during offline eval — fix: audit pipeline for future data usage. (2) **Distribution shift (data drift)** — real-world distribution has changed since training; fix: monitor feature distributions, retrain periodically. (3) **Concept drift** — the mapping from features to labels has changed; fix: monitor prediction accuracy over time. (4) **Feedback loops** — model predictions affect future data, creating a non-stationary distribution. (5) **Training-serving skew** — preprocessing in training differs from serving; fix: shared preprocessing pipeline.

---

## Part 2 — Coding Questions

### Question 1: Implement Gradient Descent

```python
def gradient_descent(f, grad_f, x_init, lr=0.01, n_iter=1000, tol=1e-6):
    """
    Minimize f using gradient descent.
    Returns: (x_optimal, history_of_loss)
    """
    x = np.array(x_init, dtype=float)
    history = []
    
    for i in range(n_iter):
        loss = f(x)
        history.append(loss)
        
        grad = grad_f(x)
        x = x - lr * grad
        
        if np.linalg.norm(grad) < tol:
            print(f"Converged at iteration {i}")
            break
    
    return x, history

# Test: minimize f(x,y) = (x-3)^2 + (y+1)^2
f = lambda x: (x[0]-3)**2 + (x[1]+1)**2
gf = lambda x: np.array([2*(x[0]-3), 2*(x[1]+1)])
x_opt, hist = gradient_descent(f, gf, [0.0, 0.0])
# Expected: x_opt ≈ [3, -1]
```

---

### Question 2: Implement k-Fold Cross-Validation from Scratch

```python
def kfold_cv(X, y, model_fn, k=5, metric_fn=None):
    """
    k-Fold CV without sklearn.
    model_fn: function that returns a new model instance
    metric_fn: function(y_true, y_pred) -> float
    Returns: list of k scores
    """
    n = len(y)
    indices = np.random.permutation(n)
    fold_size = n // k
    scores = []
    
    for fold in range(k):
        val_start = fold * fold_size
        val_end = val_start + fold_size if fold < k-1 else n
        
        val_idx = indices[val_start:val_end]
        train_idx = np.concatenate([indices[:val_start], indices[val_end:]])
        
        X_tr, X_val = X[train_idx], X[val_idx]
        y_tr, y_val = y[train_idx], y[val_idx]
        
        model = model_fn()
        model.fit(X_tr, y_tr)
        y_pred = model.predict(X_val)
        
        score = metric_fn(y_val, y_pred) if metric_fn else model.score(X_val, y_val)
        scores.append(score)
    
    return scores
```

---

### Question 3: Implement Logistic Regression from Scratch

```python
class LogisticRegressionScratch:
    def __init__(self, lr=0.1, n_iter=1000, lambda_=0.0):
        self.lr, self.n_iter, self.lambda_ = lr, n_iter, lambda_
    
    @staticmethod
    def sigmoid(z):
        return 1 / (1 + np.exp(-np.clip(z, -500, 500)))
    
    def fit(self, X, y):
        n, d = X.shape
        self.w = np.zeros(d)
        self.b = 0.0
        
        for _ in range(self.n_iter):
            y_hat = self.sigmoid(X @ self.w + self.b)
            dw = (X.T @ (y_hat - y)) / n + self.lambda_ * self.w
            db = np.mean(y_hat - y)
            self.w -= self.lr * dw
            self.b -= self.lr * db
        return self
    
    def predict_proba(self, X):
        return self.sigmoid(X @ self.w + self.b)
    
    def predict(self, X, threshold=0.5):
        return (self.predict_proba(X) >= threshold).astype(int)
```

---

### Question 4: Implement SMOTE

```python
def smote(X, y, minority_class=1, k=5, ratio=1.0):
    """
    Synthetic Minority Over-sampling Technique.
    Generates synthetic samples for the minority class.
    """
    from sklearn.neighbors import NearestNeighbors
    
    X_min = X[y == minority_class]
    n_to_generate = int(ratio * (np.sum(y != minority_class) - len(X_min)))
    
    if n_to_generate <= 0:
        return X, y
    
    # Find k nearest neighbors for each minority sample
    knn = NearestNeighbors(n_neighbors=k+1).fit(X_min)
    _, neighbor_indices = knn.kneighbors(X_min)
    
    synthetic = []
    for _ in range(n_to_generate):
        # Pick a random minority sample
        idx = np.random.randint(0, len(X_min))
        # Pick one of its neighbors (exclude itself at index 0)
        neighbor_idx = neighbor_indices[idx, np.random.randint(1, k+1)]
        # Interpolate
        alpha = np.random.random()
        synthetic.append(X_min[idx] + alpha * (X_min[neighbor_idx] - X_min[idx]))
    
    X_synthetic = np.array(synthetic)
    y_synthetic = np.ones(len(X_synthetic), dtype=y.dtype) * minority_class
    
    X_aug = np.vstack([X, X_synthetic])
    y_aug = np.hstack([y, y_synthetic])
    
    # Shuffle
    idx = np.random.permutation(len(y_aug))
    return X_aug[idx], y_aug[idx]
```

---

### Question 5: Implement Precision-Recall Curve

```python
def precision_recall_curve_scratch(y_true, y_scores):
    """
    Compute precision-recall curve at all unique thresholds.
    Returns: precision, recall arrays (no sklearn).
    """
    thresholds = np.sort(np.unique(y_scores))[::-1]  # descending
    precisions, recalls = [1.0], [0.0]   # starts at (0 recall, 1 precision)
    
    for thresh in thresholds:
        y_pred = (y_scores >= thresh).astype(int)
        tp = np.sum((y_pred == 1) & (y_true == 1))
        fp = np.sum((y_pred == 1) & (y_true == 0))
        fn = np.sum((y_pred == 0) & (y_true == 1))
        
        prec = tp / (tp + fp) if (tp + fp) > 0 else 0.0
        rec = tp / (tp + fn) if (tp + fn) > 0 else 0.0
        
        precisions.append(prec)
        recalls.append(rec)
    
    return np.array(precisions), np.array(recalls), thresholds

# Average Precision: area under PR curve
def average_precision_scratch(y_true, y_scores):
    prec, rec, _ = precision_recall_curve_scratch(y_true, y_scores)
    # Trapezoidal integration
    return -np.sum(np.diff(rec) * prec[1:])
```

---

### Question 6: Implement a Mini Neural Network Training Loop

```python
import torch
import torch.nn as nn

class MiniMLP(nn.Module):
    def __init__(self, input_dim, hidden_dims, output_dim, dropout=0.2):
        super().__init__()
        layers = []
        prev = input_dim
        for h in hidden_dims:
            layers += [nn.Linear(prev, h), nn.BatchNorm1d(h), nn.ReLU(), nn.Dropout(dropout)]
            prev = h
        layers.append(nn.Linear(prev, output_dim))
        self.net = nn.Sequential(*layers)
    
    def forward(self, x): return self.net(x)

def train_and_evaluate(X_train, y_train, X_val, y_val, config):
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    
    # Convert to tensors
    X_tr = torch.tensor(X_train, dtype=torch.float32).to(device)
    y_tr = torch.tensor(y_train, dtype=torch.long).to(device)
    X_v  = torch.tensor(X_val, dtype=torch.float32).to(device)
    y_v  = torch.tensor(y_val, dtype=torch.long).to(device)
    
    loader = torch.utils.data.DataLoader(
        torch.utils.data.TensorDataset(X_tr, y_tr),
        batch_size=config.get('batch_size', 64), shuffle=True
    )
    
    model = MiniMLP(X_train.shape[1], config.get('hidden', [128, 64]), 
                    config.get('n_classes', 2)).to(device)
    optimizer = torch.optim.AdamW(model.parameters(), lr=config.get('lr', 1e-3))
    criterion = nn.CrossEntropyLoss()
    
    best_val_acc = 0
    for epoch in range(config.get('epochs', 100)):
        model.train()
        for Xb, yb in loader:
            optimizer.zero_grad()
            criterion(model(Xb), yb).backward()
            optimizer.step()
        
        model.eval()
        with torch.no_grad():
            val_acc = (model(X_v).argmax(1) == y_v).float().mean().item()
            if val_acc > best_val_acc:
                best_val_acc = val_acc
    
    return model, best_val_acc
```

---

## Part 3 — System Design

### "Design a recommendation system for a music streaming app."

**Clarify scope first:** How many users/songs? Real-time or batch? Cold start handling?

**Components:**
1. **Data:** User-song play history, song metadata, user demographics
2. **Model:** Two-tower neural network (user embedding + song embedding, trained with contrastive loss)
3. **Offline:** Train weekly on all play data
4. **Online:** At query time, retrieve top-k songs using approximate nearest neighbor (FAISS or ScaNN)
5. **Re-ranking:** Apply a fast scoring model (XGBoost) using context (time of day, recent plays, explicit skips)
6. **Cold start:** For new users, use popularity + content-based; for new songs, use audio embedding (CNN on spectrogram)
7. **Evaluation:** A/B test; offline metrics: NDCG@10, Recall@50

---

### "Design a fraud detection system that operates in real-time."

**Constraints:** < 100ms latency, high recall (can't miss fraud), accept some false positives

**Pipeline:**
1. **Features:** Transaction features + user/merchant aggregates (from Feature Store, precomputed hourly)
2. **Model:** LightGBM for primary classifier (fast inference, interpretable)
3. **Threshold:** Tuned to recall=0.95 (catch 95% of fraud), accept ~5% false positive rate
4. **Serving:** REST API (FastAPI), model loaded in memory, < 5ms per prediction
5. **Fallback:** If model unavailable, use simple rules (transaction > $10k = flag)
6. **Monitoring:** Track positive rate, feature distributions daily; retrain monthly or on drift alert
7. **Human review:** Flagged transactions reviewed by fraud analysts; labels feed back into next training set

---

### "How would you debug a model that is performing worse in production than in offline evaluation?"

1. **Confirm the problem:** Is it the metric? Serving infrastructure? Data pipeline?
2. **Check for train/serve skew:** Log production features and compare distributions to training data (feature drift)
3. **Check for label drift:** If labels are available from recent production data, is the error rate actually higher?
4. **Look at error cases:** Sample 100 misclassified examples from production. Any pattern?
5. **Check temporal alignment:** Is the model using features computed from future data in training? (leakage)
6. **A/B test:** If possible, run old model alongside new model to isolate
7. **Reproduce locally:** Can you replicate production inputs and get wrong predictions?

---

## Part 4 — ML Breadth Questions (1-minute answers)

**Q: What is the difference between parametric and non-parametric models?**
A: Parametric models have a fixed number of parameters (linear regression, logistic regression, neural networks — fixed architecture). Non-parametric models grow with data (k-NN, decision trees, SVMs with kernels). Non-parametric are more flexible but need more data.

**Q: What is a kernel in SVMs?**
A: A kernel function `K(x,z) = φ(x)ᵀφ(z)` computes the dot product in a (possibly infinite) feature space without explicitly constructing `φ(x)`. This allows SVMs to find non-linear decision boundaries while still solving a convex optimization problem.

**Q: What is the difference between dropout and L2 regularization?**
A: L2 adds a penalty on the magnitude of weights, shrinking them toward zero. Dropout randomly zeros out neurons during training, forcing the network to learn redundant representations. They address different failure modes: L2 prevents large weights, dropout prevents co-adaptation between neurons.

**Q: What is transfer learning? When does it work well?**
A: Using weights pre-trained on one task as initialization for another task. Works best when: (1) source and target tasks are similar (both involve visual features, or both are NLP tasks), (2) target dataset is small (pretrained features provide useful structure), (3) the pre-training corpus is large and diverse.

**Q: What is the difference between Type I and Type II error?**
A: Type I (false positive): reject the null hypothesis when it's true (e.g., classify benign transaction as fraud). Type II (false negative): fail to reject null when it's false (miss actual fraud). In ML: precision penalizes Type I, recall penalizes Type II.

**Q: What is regularization and why is it needed?**
A: Regularization adds a penalty for model complexity to the loss function. Needed because models with too many parameters can memorize the training data instead of learning generalizable patterns (overfitting). The regularization term provides an inductive bias toward simpler solutions that generalize better.

**Q: Explain the attention mechanism in one minute.**
A: Given a query vector and a set of key-value pairs, attention computes a weighted combination of values, where the weight for each value depends on the similarity between the query and its corresponding key. `Attention(Q,K,V) = softmax(QKᵀ/√d)V`. This lets the model dynamically focus on relevant parts of the input for each output step.
