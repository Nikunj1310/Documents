# Chapter 4 — Linear Models

> *Linear models are not "simple" models — they are the foundation of everything that comes after. Understanding their derivation, geometry, and failure modes prepares you for regularization, logistic regression, and even the linear layers of neural networks.*

---

## 4.1 Linear Regression

### The Model

Given input `x ∈ ℝⁿ`, predict a continuous output:
```
ŷ = wᵀx + b = w₁x₁ + w₂x₂ + ... + wₙxₙ + b
```

Using the **bias trick** (append 1 to x): `x̃ = [x₁, ..., xₙ, 1]ᵀ`, then:
```
ŷ = θᵀx̃     where θ = [w₁, ..., wₙ, b]ᵀ
```

This simplifies notation: one weight vector θ encodes everything.

### Loss Function: Mean Squared Error

```
L(θ) = (1/n) Σᵢ (ŷᵢ - yᵢ)² = (1/n) ‖Xθ - y‖²
```

where X is the design matrix (each row is one training example).

**Why MSE?** It is the MLE solution under the assumption that the noise is Gaussian:
```
y = θᵀx + ε,    ε ~ N(0, σ²)
```
Maximizing the Gaussian likelihood → minimizing MSE.

### The Normal Equation (Closed-Form Solution)

For linear regression, we can solve for the optimal θ exactly:
```
∂L/∂θ = (2/n)(XᵀX)θ - (2/n)Xᵀy = 0
→  θ* = (XᵀX)⁻¹Xᵀy
```

`(XᵀX)⁻¹Xᵀ` is the **Moore-Penrose pseudoinverse** of X.

**Derivation:**
```
L(θ) = (1/n)(Xθ - y)ᵀ(Xθ - y)
      = (1/n)(θᵀXᵀXθ - 2yᵀXθ + yᵀy)
∇_θ L = (2/n)(XᵀXθ - Xᵀy) = 0
XᵀXθ = Xᵀy       ← Normal Equations
θ* = (XᵀX)⁻¹Xᵀy  ← if XᵀX is invertible
```

**When to use:** Small datasets (n < 10,000). Exact solution, no hyperparameters.

**Complexity:** O(n·d²) for the matrix multiply + O(d³) for the inversion. Becomes slow for many features.

**When NOT to use:** Large datasets (use gradient descent), highly correlated features (XᵀX near-singular, solution unstable).

### Geometric Interpretation

The prediction `ŷ = Xθ` is a linear combination of the columns of X. The optimal θ projects `y` onto the column space of X. The residual `y - ŷ` is orthogonal to the column space.

This is why it's called "least squares" — we are finding the closest point in the column space to y.

### Implementation

```python
import numpy as np
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score

# From scratch
class LinearRegressionScratch:
    def fit(self, X, y):
        # Add bias column
        X_b = np.c_[np.ones(len(X)), X]
        # Normal equation
        self.theta = np.linalg.pinv(X_b.T @ X_b) @ X_b.T @ y
        return self
    
    def predict(self, X):
        X_b = np.c_[np.ones(len(X)), X]
        return X_b @ self.theta
    
    @property
    def intercept_(self):
        return self.theta[0]
    
    @property
    def coef_(self):
        return self.theta[1:]

# Generate data
X, y = make_regression(n_samples=200, n_features=5, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

model = LinearRegressionScratch()
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
print(f"R²: {r2_score(y_test, y_pred):.4f}")
print(f"RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")

# Using scikit-learn
from sklearn.linear_model import LinearRegression
sklearn_model = LinearRegression()
sklearn_model.fit(X_train, y_train)
```

---

## 4.2 Regularization

Regularization adds a penalty term to the loss to prevent overfitting by discouraging large weights.

```
L_reg(θ) = L(θ) + λ · Penalty(θ)
```

`λ` (lambda) is the **regularization strength** hyperparameter.
- `λ = 0`: no regularization (pure ERM)
- `λ → ∞`: all weights pushed to zero

### Ridge Regression (L2 Regularization)

```
L_ridge(θ) = (1/n) ‖Xθ - y‖² + λ ‖w‖²
```

Note: regularize weights w, NOT the bias b.

**Closed-form solution:**
```
θ* = (XᵀX + λI)⁻¹Xᵀy
```

Adding `λI` makes `(XᵀX + λI)` invertible even when `XᵀX` is singular. This is called **Tikhonov regularization**.

**Effect:** Weights are shrunk toward zero, but rarely exactly zero. All features remain in the model.

**Geometry:** L2 penalty is a sphere. The optimal θ is where the elliptical loss contour first touches the sphere.

### Lasso Regression (L1 Regularization)

```
L_lasso(θ) = (1/n) ‖Xθ - y‖² + λ ‖w‖₁
```

**No closed-form** — the L1 penalty is not differentiable at 0. Solved via coordinate descent or subgradient methods.

**Effect:** Lasso produces **sparse** solutions — many weights become exactly zero. Acts as feature selection.

**Geometry:** L1 penalty is a diamond (polytope). Loss contours tend to touch the diamond at corners, which are on the axes (one weight zero, others non-zero).

**When to use Lasso vs Ridge:**
- Lasso: you believe many features are irrelevant. Want automatic feature selection.
- Ridge: most features contribute something. Want to shrink all uniformly.

### ElasticNet

Combines both:
```
L_elastic(θ) = (1/n) ‖Xθ - y‖² + λ₁‖w‖₁ + λ₂‖w‖²
```

Best of both worlds. The `l1_ratio` hyperparameter controls the mix.

### Choosing λ

Always choose λ via cross-validation:
```python
from sklearn.linear_model import RidgeCV, LassoCV
from sklearn.model_selection import cross_val_score

# RidgeCV automatically searches over alphas
ridge = RidgeCV(alphas=[0.001, 0.01, 0.1, 1, 10, 100], cv=5)
ridge.fit(X_train, y_train)
print(f"Best λ: {ridge.alpha_}")

# Or manually
from sklearn.linear_model import Ridge
lambdas = np.logspace(-3, 3, 50)
cv_scores = [cross_val_score(Ridge(alpha=l), X, y, cv=5).mean() for l in lambdas]
best_lambda = lambdas[np.argmax(cv_scores)]
```

---

## 4.3 Logistic Regression

Despite the name, logistic regression is a **classification** algorithm. It models the probability that an input belongs to a class.

### The Sigmoid Function

```
σ(z) = 1 / (1 + e⁻ᶻ)
```

Maps any real number to (0, 1). Has nice properties:
```
σ(-z) = 1 - σ(z)
σ'(z) = σ(z)(1 - σ(z))
```

### The Model

```
P(y = 1 | x) = σ(wᵀx + b) = 1 / (1 + exp(-(wᵀx + b)))
```

The model outputs a probability. Threshold at 0.5 for binary classification.

**Decision boundary:** The set of x where P(y=1|x) = 0.5 → where wᵀx + b = 0 → a **hyperplane** in feature space.

### Log-Odds Interpretation

Taking the log of the odds ratio:
```
log(P(y=1|x) / P(y=0|x)) = wᵀx + b
```

Logistic regression models the **log-odds** as a linear function of x. Each feature changes the log-odds by a fixed amount.

### Loss Function: Binary Cross-Entropy

Derived from maximum likelihood under a Bernoulli model:
```
L(w, b) = -(1/n) Σᵢ [yᵢ log(ŷᵢ) + (1-yᵢ) log(1-ŷᵢ)]
```

No closed-form solution. Optimized via gradient descent:
```
∂L/∂w = (1/n) Σᵢ (ŷᵢ - yᵢ) xᵢ
∂L/∂b = (1/n) Σᵢ (ŷᵢ - yᵢ)
```

Remarkably clean gradients — the same form as linear regression but with the sigmoid nonlinearity absorbed.

### Multiclass: Softmax Regression

For K classes, use the **softmax** function:
```
P(y = k | x) = exp(wₖᵀx) / Σⱼ exp(wⱼᵀx)
```

Outputs K probabilities that sum to 1. Loss = multiclass cross-entropy:
```
L = -(1/n) Σᵢ Σₖ 1[yᵢ = k] log P(yᵢ = k | xᵢ)
```

### Regularization for Logistic Regression

Same story as linear regression:
```python
from sklearn.linear_model import LogisticRegression

# C = 1/λ (inverse of regularization strength)
model = LogisticRegression(
    penalty='l2',    # 'l1', 'l2', 'elasticnet', 'none'
    C=1.0,           # smaller C = more regularization
    solver='lbfgs',  # 'liblinear', 'saga', etc.
    max_iter=1000
)
```

### Implementation from Scratch

```python
class LogisticRegressionScratch:
    def __init__(self, lr=0.1, n_iter=1000, lambda_=0.01):
        self.lr = lr
        self.n_iter = n_iter
        self.lambda_ = lambda_
    
    def sigmoid(self, z):
        return 1 / (1 + np.exp(-np.clip(z, -250, 250)))  # clip for stability
    
    def fit(self, X, y):
        n, d = X.shape
        self.w = np.zeros(d)
        self.b = 0.0
        self.losses = []
        
        for _ in range(self.n_iter):
            z = X @ self.w + self.b
            y_hat = self.sigmoid(z)
            
            # Gradients
            dw = (1/n) * X.T @ (y_hat - y) + self.lambda_ * self.w
            db = (1/n) * np.sum(y_hat - y)
            
            # Update
            self.w -= self.lr * dw
            self.b -= self.lr * db
            
            # Loss
            eps = 1e-10
            loss = -(1/n) * np.sum(y*np.log(y_hat+eps) + (1-y)*np.log(1-y_hat+eps))
            self.losses.append(loss)
        
        return self
    
    def predict_proba(self, X):
        return self.sigmoid(X @ self.w + self.b)
    
    def predict(self, X, threshold=0.5):
        return (self.predict_proba(X) >= threshold).astype(int)
```

---

## 4.4 Assumptions and Diagnostics

### Assumptions of Linear Regression

1. **Linearity:** True relationship is linear in the features
2. **Independence:** Data points are i.i.d.
3. **Homoscedasticity:** Residuals have constant variance
4. **Normality:** Residuals are normally distributed (needed for inference, not prediction)
5. **No multicollinearity:** Features are not perfectly correlated

Violating these doesn't necessarily break your model — it just means the model is suboptimal or your confidence intervals are wrong.

### Residual Analysis

```python
import matplotlib.pyplot as plt

y_pred = model.predict(X_test)
residuals = y_test - y_pred

# Residuals vs fitted values (check homoscedasticity)
plt.scatter(y_pred, residuals, alpha=0.5)
plt.axhline(0, color='red', linestyle='--')
plt.xlabel('Fitted Values')
plt.ylabel('Residuals')
plt.title('Residuals vs Fitted')
plt.show()

# QQ plot (check normality of residuals)
from scipy import stats
stats.probplot(residuals, plot=plt)
plt.show()
```

### Variance Inflation Factor (VIF)

Detects multicollinearity:
```python
from statsmodels.stats.outliers_influence import variance_inflation_factor

vif_data = pd.DataFrame()
vif_data["feature"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X.values, i) for i in range(len(X.columns))]
# VIF > 10 indicates severe multicollinearity
```

---

## 4.5 Interpretability

Linear models are the gold standard for interpretability.

**Coefficients for scaled features:** After standardizing, `|wᵢ|` tells you the relative importance of feature i.

**Odds ratios (logistic regression):** `exp(wᵢ)` is the multiplicative change in odds for a unit increase in feature i.

**Feature importance plot:**
```python
import matplotlib.pyplot as plt

coefs = pd.Series(model.coef_[0], index=feature_names)
coefs.sort_values().plot(kind='barh', figsize=(8, 6))
plt.title('Feature Coefficients')
plt.axvline(0, color='black', linestyle='-', linewidth=0.8)
plt.tight_layout()
plt.show()
```

---

## Exercises

1. **Normal equation derivation:** Starting from `L(θ) = (1/n) ‖Xθ - y‖²`, take the gradient and set it to zero. Derive the normal equation step-by-step. No skipping steps.

2. **Ridge vs Lasso sparsity:** Generate a dataset with 20 features, but only 5 are actually correlated with the target. Train a Lasso and Ridge model. Compare the coefficients. Which one zeros out the irrelevant features?

3. **Logistic regression boundary:** Generate a 2D classification dataset. Train logistic regression. Plot the data points (colored by class) and the decision boundary. Show how the boundary changes as you adjust the regularization strength C.

4. **From-scratch gradient descent:** Implement linear regression using gradient descent (NOT the normal equation). Compare the solution to the normal equation. They should match.

5. **Multicollinearity experiment:** Create features x₁, x₂ = x₁ + noise. Add them both to a linear regression. What happens to the coefficients? Now apply Ridge regularization. What changes?

---

## Further Reading

- *The Elements of Statistical Learning* — Hastie et al. — Chapter 3 (Linear Methods for Regression) and Chapter 4 (Linear Methods for Classification)
- *Pattern Recognition and Machine Learning* — Bishop — Chapter 3 (Linear Models)
- *An Introduction to Statistical Learning* — James et al. — Chapters 2–4 (excellent practical coverage, free PDF)
