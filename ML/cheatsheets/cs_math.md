# Cheat Sheet — Key Equations

> The most important formulas in ML. Organized by concept.

---

## Linear Algebra

```
Matrix multiply:       (AB)ᵢⱼ = Σₖ Aᵢₖ Bₖⱼ
Transpose rules:       (AB)ᵀ = BᵀAᵀ
Normal equation:       θ* = (XᵀX)⁻¹Xᵀy
SVD:                   A = UΣVᵀ
L1 norm:               ‖x‖₁ = Σ |xᵢ|
L2 norm:               ‖x‖₂ = √(Σ xᵢ²) = √(xᵀx)
Cosine similarity:     cos(θ) = (xᵀy) / (‖x‖ ‖y‖)
Eigenvector:           Av = λv
Trace:                 tr(A) = Σᵢ Aᵢᵢ = Σᵢ λᵢ
Determinant:           det(A) = Πᵢ λᵢ
```

---

## Calculus / Optimization

```
Chain rule:            d/dx[f(g(x))] = f'(g(x)) · g'(x)
Gradient:              ∇f(x) = [∂f/∂x₁, ..., ∂f/∂xₙ]ᵀ
Jacobian:              J_{ij} = ∂fᵢ/∂xⱼ
Hessian:               H_{ij} = ∂²f/(∂xᵢ∂xⱼ)
Taylor (2nd order):    f(x) ≈ f(x₀) + ∇f(x₀)ᵀΔx + ½ Δxᵀ H Δx
Gradient descent:      θ ← θ - η ∇L(θ)
Newton's method:       θ ← θ - H⁻¹ ∇L(θ)
```

---

## Probability

```
Bayes:                 P(A|B) = P(B|A)P(A) / P(B)
Chain rule:            P(A,B,C) = P(A) P(B|A) P(C|A,B)
Expectation:           E[X] = Σ x P(X=x)
Variance:              Var(X) = E[(X-μ)²] = E[X²] - μ²
Covariance:            Cov(X,Y) = E[(X-μₓ)(Y-μᵧ)]
MLE:                   θ̂_MLE = argmax_θ Σᵢ log p(xᵢ; θ)
MAP:                   θ̂_MAP = argmax_θ Σᵢ log p(xᵢ; θ) + log p(θ)
Gaussian PDF:          f(x) = exp(-(x-μ)²/2σ²) / √(2πσ²)
KL divergence:         KL(p‖q) = Σ p(x) log(p(x)/q(x)) ≥ 0
Cross-entropy:         H(p,q) = -Σ p(x) log q(x) = H(p) + KL(p‖q)
Entropy:               H(X) = -Σ P(x) log P(x)
```

---

## Loss Functions

```
MSE:          L = (1/n) Σ (ŷᵢ - yᵢ)²
MAE:          L = (1/n) Σ |ŷᵢ - yᵢ|
Huber:        L = ½(ŷ-y)² if |ŷ-y|≤δ, else δ(|ŷ-y| - δ/2)

Binary CE:    L = -(1/n) Σ [y log ŷ + (1-y) log(1-ŷ)]
Multiclass CE:L = -(1/n) Σᵢ Σₖ 1[yᵢ=k] log P(yᵢ=k|xᵢ)

Hinge (SVM):  L = max(0, 1 - yᵢ(wᵀxᵢ + b))

KL (VAE):     L_KL = -½ Σ(1 + log σ² - μ² - σ²)
ELBO (VAE):   ELBO = E[log p(x|z)] - KL(q(z|x) ‖ p(z))
```

---

## Regularization

```
L2 (Ridge):   L_reg = L + λ Σ wᵢ²
L1 (Lasso):   L_reg = L + λ Σ |wᵢ|
Elastic Net:  L_reg = L + λ₁ Σ|wᵢ| + λ₂ Σwᵢ²
Ridge solution: θ* = (XᵀX + λI)⁻¹Xᵀy
```

---

## Classification

```
Sigmoid:      σ(z) = 1/(1+e⁻ᶻ),   σ'(z) = σ(z)(1-σ(z))
Softmax:      p(k) = exp(zₖ) / Σⱼ exp(zⱼ)
Log-odds:     log(p/(1-p)) = wᵀx + b
```

---

## Evaluation Metrics

```
Accuracy:     (TP+TN) / (TP+TN+FP+FN)
Precision:    TP / (TP+FP)
Recall:       TP / (TP+FN)
F1:           2·P·R / (P+R)
F_β:          (1+β²)·P·R / (β²P + R)
TPR:          TP/(TP+FN)  = Recall = Sensitivity
FPR:          FP/(FP+TN)  = 1 - Specificity
AUC:          P(score(pos) > score(neg))
R²:           1 - Σ(ŷ-y)² / Σ(ȳ-y)²
Perplexity:   exp(- (1/n) Σ log P(xᵢ))
```

---

## Trees & Ensembles

```
Gini:         1 - Σₖ pₖ²
Entropy:      -Σₖ pₖ log pₖ
Info Gain:    H(parent) - Σ (nₗ/n)·H(leaf)

GBM update:   Fₜ = Fₜ₋₁ + η · hₜ     where hₜ fits pseudo-residuals rᵢ = -∂L/∂F(xᵢ)
Pseudo-resid: rᵢ = yᵢ - Fₜ₋₁(xᵢ)    (for MSE loss)
```

---

## Neural Networks

```
Forward:      z[l] = W[l] a[l-1] + b[l],    a[l] = f(z[l])
Backprop:     δ[L] = ∇L ⊙ f'(z[L])
              δ[l] = (W[l+1]ᵀ δ[l+1]) ⊙ f'(z[l])
Gradients:    ∂L/∂W[l] = δ[l] (a[l-1])ᵀ,   ∂L/∂b[l] = δ[l]

Activations:
  ReLU:       f(z) = max(0, z),     f'(z) = 1[z>0]
  Sigmoid:    f(z) = σ(z),          f'(z) = σ(z)(1-σ(z))
  Tanh:       f(z) = tanh(z),       f'(z) = 1-tanh²(z)
  GELU:       f(z) = z·Φ(z)

He init:      W ~ N(0, 2/fan_in)      (ReLU)
Xavier init:  W ~ N(0, 1/fan_in)      (sigmoid/tanh)

Batch Norm:   x̂ = (x-μ_B)/√(σ²_B+ε),   y = γx̂ + β
```

---

## Attention & Transformers

```
Scaled dot-product:
  Attention(Q,K,V) = softmax(QKᵀ/√dₖ) · V

Multi-head:
  MultiHead = Concat(head₁,...,headₕ) Wᴼ
  headᵢ = Attention(Q Wᵢᴼ, K Wᵢᴷ, V Wᵢᵛ)

Positional encoding:
  PE(pos, 2i)   = sin(pos / 10000^(2i/d))
  PE(pos, 2i+1) = cos(pos / 10000^(2i/d))

Language model loss:  L = -(1/T) Σₜ log P(xₜ | x₁,...,xₜ₋₁)
```

---

## Optimization Algorithms

```
SGD:          θ ← θ - η g
Momentum:     v ← βv + (1-β)g,   θ ← θ - ηv
RMSProp:      v ← β₂v + (1-β₂)g²,   θ ← θ - η g/√(v+ε)
Adam:         m ← β₁m + (1-β₁)g          (1st moment)
              v ← β₂v + (1-β₂)g²         (2nd moment)
              m̂ = m/(1-β₁ᵗ),  v̂ = v/(1-β₂ᵗ)
              θ ← θ - η m̂/(√v̂ + ε)
Defaults:     β₁=0.9, β₂=0.999, ε=1e-8, η=3×10⁻⁴
```

---

## SVMs

```
Objective:    min ½‖w‖² + C Σ ξᵢ    s.t.  yᵢ(wᵀxᵢ+b) ≥ 1-ξᵢ,  ξᵢ≥0
Margin:       2/‖w‖
Kernel trick: K(xᵢ,xⱼ) = φ(xᵢ)ᵀφ(xⱼ)
RBF kernel:   K(x,z) = exp(-γ‖x-z‖²)
```

---

## Reinforcement Learning

```
Return:         Gₜ = Σₖ γᵏ rₜ₊ₖ
Bellman V:      V^π(s) = Σₐ π(a|s) Σₛ' P(s'|s,a)[R + γV^π(s')]
Bellman Q:      Q^π(s,a) = Σₛ' P(s'|s,a)[R + γ Σₐ' π(a'|s') Q^π(s',a')]
Q-learning:     Q(s,a) ← Q(s,a) + α[r + γ max_a' Q(s',a') - Q(s,a)]
Policy gradient:∇J(θ) = E[Σₜ ∇ log π_θ(aₜ|sₜ) · Gₜ]
TD error:       δₜ = rₜ + γV(sₜ₊₁) - V(sₜ)
```

---

## Generative Models

```
VAE ELBO:       ELBO = E[log p_θ(x|z)] - KL(q_φ(z|x) ‖ N(0,I))
KL (Gaussian):  KL = -½ Σ(1 + log σ² - μ² - σ²)
Reparam trick:  z = μ + σ ⊙ ε,  ε ~ N(0,I)

GAN objective:  min_G max_D  E[log D(x)] + E[log(1-D(G(z)))]

Diffusion forward:  q(xₜ|x₀) = N(√ᾱₜ x₀, (1-ᾱₜ)I)
Diffusion loss:     L = E[‖ε - ε_θ(xₜ,t)‖²]
```
