# Chapter 1 — Mathematical Foundations

> *Every ML algorithm is a mathematical object. Skipping this chapter means you will use ML as a black box forever. Work through the exercises before moving to Chapter 2.*

---

## 1.1 Linear Algebra

Linear algebra is the language of data. A dataset of n samples with d features is naturally a matrix of shape (n × d). Every operation — regression, PCA, neural networks — reduces to matrix operations under the hood.

### Scalars, Vectors, Matrices, Tensors

- **Scalar** — a single real number: `x ∈ ℝ`
- **Vector** — an ordered list of numbers: `x ∈ ℝⁿ`. Convention: column vector by default.
- **Matrix** — a 2D array: `A ∈ ℝᵐˣⁿ` (m rows, n columns)
- **Tensor** — generalization to arbitrary dimensions. An image is a 3-tensor (H × W × C).

### Key Operations

**Dot product** (inner product) of two vectors:
```
x · y = xᵀy = Σᵢ xᵢyᵢ
```
Geometric interpretation: `x · y = ‖x‖ ‖y‖ cos(θ)`. The dot product measures alignment. Orthogonal vectors have dot product 0.

**Matrix multiplication** `C = AB` where `A ∈ ℝᵐˣⁿ`, `B ∈ ℝⁿˣᵖ`:
```
Cᵢⱼ = Σₖ Aᵢₖ Bₖⱼ
```
Think of it as: each row of A dotted with each column of B.

**Transpose**: `(Aᵀ)ᵢⱼ = Aⱼᵢ`. Rows become columns.

**Properties:**
- `(AB)ᵀ = BᵀAᵀ` — order reverses
- Matrix multiplication is NOT commutative: `AB ≠ BA` in general
- `(AB)C = A(BC)` — associativity holds

### Norms

A norm measures the "size" or "length" of a vector.

**L1 norm:** `‖x‖₁ = Σᵢ |xᵢ|`  
**L2 norm (Euclidean):** `‖x‖₂ = √(Σᵢ xᵢ²) = √(xᵀx)`  
**Lp norm:** `‖x‖ₚ = (Σᵢ |xᵢ|ᵖ)^(1/p)`  
**L∞ norm:** `‖x‖∞ = maxᵢ |xᵢ|`

In ML, L2 norm appears in regularization (ridge), L1 in sparsity (lasso).

### Special Matrices

**Identity matrix** `I`: ones on diagonal, zeros elsewhere. `AI = IA = A`.

**Diagonal matrix**: non-zero only on diagonal. Efficient: `Dx = d ⊙ x` (elementwise multiply).

**Symmetric matrix**: `A = Aᵀ`. Covariance matrices are symmetric.

**Orthogonal matrix** `Q`: columns are orthonormal — `QᵀQ = QQᵀ = I`. Rotations are orthogonal. Key: `Q⁻¹ = Qᵀ` (very cheap to invert).

**Positive definite matrix**: `A` is PD if `xᵀAx > 0` for all nonzero `x`. Covariance matrices are PSD (positive semi-definite). Hessians of convex functions are PD.

### Matrix Inverse

`A⁻¹` exists iff `A` is square and full rank (det(A) ≠ 0).

`AA⁻¹ = A⁻¹A = I`

**Warning:** Never compute `A⁻¹b` directly in code. Use `np.linalg.solve(A, b)` — it's numerically stable and faster. Inverting is for math, not computation.

### Solving Linear Systems

`Ax = b` — given `A` and `b`, find `x`.

- If `A` is square and invertible: `x = A⁻¹b` (unique solution)
- If overdetermined (m > n, more equations than unknowns): least-squares solution `x = (AᵀA)⁻¹Aᵀb` — this IS linear regression
- If underdetermined (m < n): infinitely many solutions

### Eigenvalues and Eigenvectors

For square matrix `A`, an eigenvector `v` and eigenvalue `λ` satisfy:
```
Av = λv
```
`v` is the direction unchanged (only scaled) by `A`. `λ` is the scaling factor.

**Finding eigenvalues:** Solve `det(A - λI) = 0` (characteristic polynomial).

**Key properties:**
- A symmetric matrix has real eigenvalues and orthogonal eigenvectors
- A PD matrix has all positive eigenvalues
- Trace(A) = Σᵢ λᵢ
- det(A) = Πᵢ λᵢ

**Why it matters in ML:**
- PCA finds eigenvectors of the covariance matrix
- The eigenspectrum of the Hessian tells you about loss landscape curvature
- PageRank is an eigenvector problem

### Singular Value Decomposition (SVD)

Any matrix `A ∈ ℝᵐˣⁿ` can be decomposed as:
```
A = UΣVᵀ
```
- `U ∈ ℝᵐˣᵐ` — orthogonal, left singular vectors
- `Σ ∈ ℝᵐˣⁿ` — diagonal, singular values σ₁ ≥ σ₂ ≥ ... ≥ 0
- `V ∈ ℝⁿˣⁿ` — orthogonal, right singular vectors

**Truncated SVD** keeps only the top k singular values — this is the math behind PCA, matrix factorization in recommender systems, and data compression.

**Relationship to eigendecomposition:** The singular values of `A` are the square roots of the eigenvalues of `AᵀA`.

---

## 1.2 Calculus

### Derivatives

The derivative of `f(x)` at `x` measures instantaneous rate of change:
```
f'(x) = df/dx = lim_{h→0} [f(x+h) - f(x)] / h
```

**Common derivatives:**
```
d/dx [xⁿ]    = nxⁿ⁻¹
d/dx [eˣ]    = eˣ
d/dx [ln x]  = 1/x
d/dx [sin x] = cos x
d/dx [cos x] = -sin x
```

**Chain rule** — essential for backpropagation:
```
d/dx [f(g(x))] = f'(g(x)) · g'(x)
```

In Leibniz notation: `dz/dx = (dz/dy)(dy/dx)` where `z = f(y)`, `y = g(x)`.

### Partial Derivatives

For `f(x₁, x₂, ..., xₙ)`, the partial derivative with respect to `xᵢ` treats all other variables as constants:
```
∂f/∂xᵢ = lim_{h→0} [f(..., xᵢ+h, ...) - f(..., xᵢ, ...)] / h
```

**Example:** `f(x, y) = x²y + 3xy²`
```
∂f/∂x = 2xy + 3y²
∂f/∂y = x² + 6xy
```

### Gradient

The gradient collects all partial derivatives into a vector:
```
∇f(x) = [∂f/∂x₁, ∂f/∂x₂, ..., ∂f/∂xₙ]ᵀ
```

**Key property:** The gradient points in the direction of steepest ascent. Gradient descent subtracts it to move downhill:
```
x ← x - η ∇f(x)
```

### Jacobian

For a vector-valued function `f: ℝⁿ → ℝᵐ`, the Jacobian is an m×n matrix:
```
J_{ij} = ∂fᵢ/∂xⱼ
```
This generalizes the gradient to multi-output functions. Critical in backpropagation through layers.

### Hessian

The Hessian is the matrix of second-order partial derivatives:
```
H_{ij} = ∂²f / (∂xᵢ ∂xⱼ)
```

The Hessian tells us about curvature. If the Hessian at a critical point is:
- Positive definite → local minimum
- Negative definite → local maximum
- Indefinite → saddle point

Most ML loss surfaces are non-convex with many saddle points.

### Taylor Series

Any smooth function can be approximated near a point `x₀`:
```
f(x) ≈ f(x₀) + f'(x₀)(x - x₀) + ½f''(x₀)(x - x₀)² + ...
```

In multiple dimensions (vector Taylor expansion):
```
f(x) ≈ f(x₀) + ∇f(x₀)ᵀ(x - x₀) + ½(x - x₀)ᵀH(x - x₀) + ...
```

This is used to derive second-order optimization methods (Newton's method).

### Convexity

A function `f` is convex if the line segment between any two points on its graph lies above or on the graph:
```
f(λx + (1-λ)y) ≤ λf(x) + (1-λ)f(y)  for λ ∈ [0,1]
```

**Why convexity matters:** Convex functions have a single global minimum. Gradient descent is guaranteed to find it. MSE loss for linear regression is convex. Neural network losses are not.

---

## 1.3 Probability and Statistics

### Sample Space and Events

A **probability space** consists of:
- Ω — sample space (all possible outcomes)
- F — event space (subsets of Ω)
- P — probability measure: P(A) ∈ [0,1], P(Ω) = 1

### Random Variables

A random variable `X` maps outcomes to real numbers. Characterized by:
- **PMF** (discrete): `P(X = x)` — probability mass function
- **PDF** (continuous): `f(x)` where `P(a ≤ X ≤ b) = ∫ₐᵇ f(x) dx`
- **CDF**: `F(x) = P(X ≤ x)`

### Expectation and Variance

**Expectation** (mean):
```
E[X] = Σₓ x·P(X=x)     (discrete)
E[X] = ∫ x·f(x) dx     (continuous)
```

**Variance:**
```
Var(X) = E[(X - E[X])²] = E[X²] - (E[X])²
```

**Standard deviation:** `σ = √Var(X)`

**Key properties:**
- `E[aX + b] = aE[X] + b`
- `Var(aX + b) = a²Var(X)`
- For independent X, Y: `Var(X + Y) = Var(X) + Var(Y)`

### Key Distributions

**Bernoulli(p):** X ∈ {0,1}, P(X=1) = p. Models a single coin flip.

**Binomial(n, p):** Number of successes in n independent Bernoulli trials.
```
P(X = k) = C(n,k) pᵏ (1-p)ⁿ⁻ᵏ
```

**Gaussian (Normal) N(μ, σ²):**
```
f(x) = (1/√(2πσ²)) exp(-(x-μ)²/2σ²)
```
The bell curve. Central limit theorem: sums of independent variables converge to Gaussian. Used everywhere in ML due to convenient math properties.

**Multivariate Gaussian N(μ, Σ):**
```
f(x) = (1/√((2π)ᵈ|Σ|)) exp(-½(x-μ)ᵀΣ⁻¹(x-μ))
```
μ is the mean vector, Σ is the covariance matrix. This is the foundation of Gaussian processes and many generative models.

**Categorical:** Generalization of Bernoulli to K classes. Output of a softmax classifier.

**Exponential/Poisson:** Modeling time between events, count data.

### Conditional Probability and Independence

```
P(A | B) = P(A ∩ B) / P(B)    (conditional probability)
```

Events A and B are **independent** if `P(A ∩ B) = P(A)P(B)`, equivalently `P(A|B) = P(A)`.

**Chain rule of probability:**
```
P(A, B, C) = P(A) P(B|A) P(C|A,B)
```

This factorization is the foundation of probabilistic graphical models.

### Bayes' Theorem

```
P(A | B) = P(B | A) · P(A) / P(B)
```

In ML terms, for a model with parameters θ and data D:
```
P(θ | D) = P(D | θ) · P(θ) / P(D)
  posterior   likelihood   prior   evidence
```

- **Prior** `P(θ)`: your belief about θ before seeing data
- **Likelihood** `P(D | θ)`: how probable is the data under θ?
- **Posterior** `P(θ | D)`: updated belief after seeing data
- **Evidence** `P(D)`: normalizing constant (often intractable)

Bayesian ML uses the full posterior. Frequentist ML maximizes the likelihood.

### Maximum Likelihood Estimation (MLE)

Given data `x₁, ..., xₙ` assumed i.i.d. from `p(x; θ)`, find θ that maximizes:
```
L(θ) = Πᵢ p(xᵢ; θ)          (likelihood)
ℓ(θ) = Σᵢ log p(xᵢ; θ)     (log-likelihood, easier to optimize)
```

We maximize log-likelihood (or equivalently minimize negative log-likelihood).

**Example:** For Gaussian data, MLE gives:
```
μ̂ = (1/n) Σᵢ xᵢ      (sample mean)
σ̂² = (1/n) Σᵢ (xᵢ - μ̂)²   (biased, but MLE estimate)
```

**Connection to ML losses:**
- Assuming Gaussian noise → minimizing MSE is MLE
- Assuming Bernoulli outputs → minimizing cross-entropy is MLE

### Information Theory

**Entropy** measures uncertainty:
```
H(X) = -Σₓ P(x) log P(x)
```
Maximum entropy: uniform distribution. Zero entropy: deterministic.

**Cross-entropy** between true distribution p and predicted q:
```
H(p, q) = -Σₓ p(x) log q(x)
```
This is the standard classification loss.

**KL divergence** — measures how much q diverges from p:
```
KL(p ‖ q) = Σₓ p(x) log(p(x) / q(x)) ≥ 0
```
Not symmetric: `KL(p‖q) ≠ KL(q‖p)`.

`H(p, q) = H(p) + KL(p ‖ q)` — cross-entropy = true entropy + divergence.

### Covariance and Correlation

**Covariance** between X and Y:
```
Cov(X, Y) = E[(X - E[X])(Y - E[Y])] = E[XY] - E[X]E[Y]
```

**Covariance matrix** for a random vector x:
```
Σ = E[(x - μ)(x - μ)ᵀ]    where μ = E[x]
```
Σᵢⱼ = covariance between xᵢ and xⱼ. Diagonal entries are variances.

**Correlation:** `ρ(X, Y) = Cov(X,Y) / (σ_X σ_Y)` — normalized to [-1, 1].

---

## 1.4 Code: Math in NumPy

```python
import numpy as np

# Vectors and matrices
x = np.array([1, 2, 3])       # shape (3,)
A = np.array([[1,2],[3,4]])    # shape (2, 2)

# Dot product
x_dot = np.dot(x, x)          # or x @ x

# Matrix multiply
B = np.array([[5,6],[7,8]])
C = A @ B                      # preferred over np.dot for matrices

# Transpose
A_T = A.T

# Inverse
A_inv = np.linalg.inv(A)

# Eigendecomposition
eigenvalues, eigenvectors = np.linalg.eig(A)

# SVD
U, sigma, Vt = np.linalg.svd(A)

# Norms
l2_norm = np.linalg.norm(x)           # L2 by default
l1_norm = np.linalg.norm(x, ord=1)

# Gradient numerical approximation
def numerical_gradient(f, x, h=1e-5):
    grad = np.zeros_like(x)
    for i in range(len(x)):
        x_plus = x.copy(); x_plus[i] += h
        x_minus = x.copy(); x_minus[i] -= h
        grad[i] = (f(x_plus) - f(x_minus)) / (2 * h)
    return grad

# Covariance matrix
data = np.random.randn(100, 5)  # 100 samples, 5 features
cov = np.cov(data.T)            # shape (5, 5)
```

---

## Exercises

1. **Matrix algebra:** Let A = [[2,1],[1,3]] and b = [4,5]. Solve Ax = b by hand using the inverse, then verify with `np.linalg.solve`.

2. **Gradient derivation:** For `f(x, y) = 3x²y + xy³ - 2y`, compute ∇f at the point (1, 2) by hand. Verify numerically using the `numerical_gradient` function above.

3. **SVD compression:** Load any 256×256 grayscale image as a matrix. Compute its SVD. Reconstruct it using only the top k=5, 20, 50 singular values. At what k does it look acceptable? What is the compression ratio?

4. **Bayes' theorem:** A disease affects 1% of the population. A test is 95% sensitive (true positive rate) and 90% specific (true negative rate). You test positive. What is the probability you have the disease? (Answer: ~8.7% — surprising but correct)

5. **MLE:** You flip a coin 100 times and get 63 heads. Using MLE, what is your estimate of p (the probability of heads)? Show the derivation by taking the derivative of the log-likelihood and setting it to zero.

6. **Eigenvalues:** What are the eigenvalues of the 2×2 covariance matrix of two perfectly correlated random variables? What does this tell you about PCA?

---

## Further Reading

- *Mathematics for Machine Learning* — Deisenroth, Faisal, Ong (Chapters 2–6 cover everything here)
- *Linear Algebra Done Right* — Axler (for deeper linear algebra theory)
- *Pattern Recognition and Machine Learning* — Bishop, Appendix B (probability review)
- 3Blue1Brown "Essence of Linear Algebra" series (YouTube) — superb geometric intuition
- 3Blue1Brown "Essence of Calculus" series (YouTube)
