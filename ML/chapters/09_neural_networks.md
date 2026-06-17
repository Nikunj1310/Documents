# Chapter 9 — Neural Networks from Scratch

> *This is the most important chapter in the deep learning sequence. If you understand backpropagation at the level in this chapter, the rest of deep learning is "just" adding more layers. Do not skip the from-scratch implementation.*

---

## 9.1 The Biological Metaphor (and Why It's Mostly Irrelevant)

Artificial neural networks were loosely inspired by the brain. In practice, they are more precisely understood as:
- **Parameterized functions** composed of simple operations
- **Universal function approximators** (given enough capacity)
- **Differentiable programs** optimized by gradient descent

Forget the biology. Think about composition of functions.

---

## 9.2 The Perceptron

The simplest neural network "neuron":

```
Inputs:   x₁, x₂, ..., xₙ
Weights:  w₁, w₂, ..., wₙ, bias b
Output:   a = activation(w₁x₁ + w₂x₂ + ... + wₙxₙ + b)
        = activation(wᵀx + b)
```

The **activation function** adds non-linearity. Without it, stacking layers is equivalent to a single linear layer (composition of linear functions is linear).

A single perceptron with sigmoid activation = logistic regression. Same math, different framing.

---

## 9.3 Activation Functions

### Sigmoid

```
σ(z) = 1 / (1 + e⁻ᶻ)
σ'(z) = σ(z)(1 - σ(z))
```

**Problems:**
- Saturates for large |z| → gradients near 0 (vanishing gradients)
- Not zero-centered (outputs always positive) → zig-zag gradient updates
- Expensive to compute

**Use:** Output layer for binary classification.

### Tanh

```
tanh(z) = (eᶻ - e⁻ᶻ) / (eᶻ + e⁻ᶻ)
tanh'(z) = 1 - tanh²(z)
```

Zero-centered, saturates like sigmoid. Better than sigmoid for hidden layers. Same vanishing gradient problem.

### ReLU (Rectified Linear Unit)

```
ReLU(z) = max(0, z)
ReLU'(z) = 1 if z > 0, else 0
```

**Why ReLU revolutionized deep learning:**
- No vanishing gradient for positive inputs
- Very cheap to compute
- Sparse activations (negative inputs become 0)
- Trains faster than sigmoid/tanh

**Problem — Dying ReLU:** If a neuron outputs 0 for all training examples, its gradient is 0 forever. Fixed by:
- Leaky ReLU: `max(0.01z, z)`
- ELU: negative side uses exponential
- Careful weight initialization
- Lower learning rates

### GELU (Gaussian Error Linear Unit)

```
GELU(z) = z · Φ(z)    where Φ is the CDF of N(0,1)
```

Used in Transformers (BERT, GPT). Smooth approximation of ReLU, empirically better for language models.

### Softmax (output layer for multiclass)

```
softmax(z)ₖ = exp(zₖ) / Σⱼ exp(zⱼ)
```

Converts a vector of arbitrary scores to probabilities that sum to 1. Always use for the final layer in multiclass classification.

**Numerical stability:** `softmax(z) = softmax(z - max(z))` — subtract max before exponentiating to prevent overflow.

```python
def softmax(z):
    z_stable = z - np.max(z, axis=-1, keepdims=True)
    exp_z = np.exp(z_stable)
    return exp_z / exp_z.sum(axis=-1, keepdims=True)
```

---

## 9.4 The Multi-Layer Perceptron (MLP)

An MLP is a sequence of layers. Each layer applies an affine transformation + activation:

```
Layer l:  z[l] = W[l] a[l-1] + b[l]
          a[l] = f(z[l])
```

For a 2-hidden-layer network:
```
Input:    a[0] = x
Layer 1:  z[1] = W[1]x + b[1],    a[1] = ReLU(z[1])
Layer 2:  z[2] = W[2]a[1] + b[2], a[2] = ReLU(z[2])
Output:   z[3] = W[3]a[2] + b[3], ŷ = softmax(z[3])
```

**Universal Approximation Theorem:** A neural network with one hidden layer and enough neurons can approximate any continuous function on a compact domain to arbitrary precision. Deep networks can do this more efficiently than wide ones.

---

## 9.5 Backpropagation

Backpropagation is an efficient algorithm for computing the gradient of the loss with respect to all parameters using the **chain rule**.

### Forward Pass

Compute and cache all intermediate values:
```
For l = 1, ..., L:
    z[l] = W[l] a[l-1] + b[l]
    a[l] = f(z[l])
Loss = L(a[L], y)
```

### Backward Pass

Compute gradients in reverse, reusing cached values:

Define the **error signal** at layer l:
```
δ[l] = ∂L/∂z[l]
```

**Output layer (L):**
```
δ[L] = ∂L/∂z[L] = (∂L/∂a[L]) · (∂a[L]/∂z[L]) = ∇_{a[L]}L ⊙ f'(z[L])
```

For cross-entropy loss + softmax: `δ[L] = a[L] - y` (remarkably clean!)

**Hidden layer (propagating backward):**
```
δ[l] = (W[l+1]ᵀ δ[l+1]) ⊙ f'(z[l])
```

**Parameter gradients:**
```
∂L/∂W[l] = δ[l] (a[l-1])ᵀ
∂L/∂b[l] = δ[l]
```

### Backprop Intuition

Think of δ[l] as "how much the loss would change if z[l] changed." Backprop computes this by multiplying two things:
1. How much z[l+1] would change if z[l] changed (the weight matrix W[l+1])
2. How much the loss changed due to z[l+1] (the upstream gradient δ[l+1])

This is the chain rule applied recursively, from output to input.

---

## 9.6 MLP from Scratch in NumPy

```python
import numpy as np

class MLP:
    def __init__(self, layer_dims, activation='relu'):
        """
        layer_dims: [input_dim, hidden1, hidden2, ..., output_dim]
        """
        self.layer_dims = layer_dims
        self.L = len(layer_dims) - 1  # number of layers
        self.params = {}
        
        # He initialization for ReLU
        for l in range(1, self.L + 1):
            self.params[f'W{l}'] = np.random.randn(layer_dims[l], layer_dims[l-1]) * np.sqrt(2. / layer_dims[l-1])
            self.params[f'b{l}'] = np.zeros((layer_dims[l], 1))
    
    def relu(self, z):
        return np.maximum(0, z)
    
    def relu_grad(self, z):
        return (z > 0).astype(float)
    
    def softmax(self, z):
        z_shifted = z - np.max(z, axis=0, keepdims=True)
        exp_z = np.exp(z_shifted)
        return exp_z / exp_z.sum(axis=0, keepdims=True)
    
    def forward(self, X):
        """
        X: shape (n_features, n_samples)
        Returns: predictions and cache for backprop
        """
        cache = {'a0': X}
        a = X
        
        for l in range(1, self.L):
            z = self.params[f'W{l}'] @ a + self.params[f'b{l}']
            a = self.relu(z)
            cache[f'z{l}'] = z
            cache[f'a{l}'] = a
        
        # Output layer (no activation, softmax applied to loss)
        z = self.params[f'W{self.L}'] @ a + self.params[f'b{self.L}']
        a_out = self.softmax(z)
        cache[f'z{self.L}'] = z
        cache[f'a{self.L}'] = a_out
        
        return a_out, cache
    
    def compute_loss(self, y_pred, y_true):
        """Cross-entropy loss. y_true: one-hot (K, n) or class indices."""
        n = y_pred.shape[1]
        # Clip to prevent log(0)
        y_pred_clipped = np.clip(y_pred, 1e-10, 1.)
        if y_true.ndim == 1:
            # class indices
            loss = -np.sum(np.log(y_pred_clipped[y_true, np.arange(n)])) / n
        else:
            # one-hot
            loss = -np.sum(y_true * np.log(y_pred_clipped)) / n
        return loss
    
    def backward(self, y_true, cache):
        n = cache['a0'].shape[1]
        grads = {}
        
        # Output layer gradient (cross-entropy + softmax = ŷ - y)
        a_L = cache[f'a{self.L}']
        if y_true.ndim == 1:
            y_one_hot = np.zeros_like(a_L)
            y_one_hot[y_true, np.arange(n)] = 1
            y_true = y_one_hot
        
        delta = (a_L - y_true) / n  # shape (K, n)
        
        for l in range(self.L, 0, -1):
            a_prev = cache[f'a{l-1}']
            grads[f'dW{l}'] = delta @ a_prev.T
            grads[f'db{l}'] = delta.sum(axis=1, keepdims=True)
            
            if l > 1:
                dA_prev = self.params[f'W{l}'].T @ delta
                delta = dA_prev * self.relu_grad(cache[f'z{l-1}'])
        
        return grads
    
    def update(self, grads, lr):
        for l in range(1, self.L + 1):
            self.params[f'W{l}'] -= lr * grads[f'dW{l}']
            self.params[f'b{l}'] -= lr * grads[f'db{l}']
    
    def fit(self, X, y, lr=0.01, epochs=100, batch_size=64, verbose=True):
        n = X.shape[1]
        self.loss_history = []
        
        for epoch in range(epochs):
            # Shuffle
            idx = np.random.permutation(n)
            X_shuf, y_shuf = X[:, idx], y[idx]
            
            epoch_loss = 0
            for i in range(0, n, batch_size):
                X_batch = X_shuf[:, i:i+batch_size]
                y_batch = y_shuf[i:i+batch_size]
                
                y_pred, cache = self.forward(X_batch)
                loss = self.compute_loss(y_pred, y_batch)
                grads = self.backward(y_batch, cache)
                self.update(grads, lr)
                epoch_loss += loss
            
            self.loss_history.append(epoch_loss)
            if verbose and (epoch + 1) % 10 == 0:
                print(f"Epoch {epoch+1}/{epochs} — Loss: {epoch_loss:.4f}")
    
    def predict(self, X):
        y_pred, _ = self.forward(X)
        return np.argmax(y_pred, axis=0)
    
    def predict_proba(self, X):
        y_pred, _ = self.forward(X)
        return y_pred.T


# Test on MNIST (or any classification dataset)
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

digits = load_digits()
X, y = digits.data, digits.target
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train).T   # (features, samples)
X_test_s = scaler.transform(X_test).T

mlp = MLP([64, 128, 64, 10])
mlp.fit(X_train_s, y_train, lr=0.01, epochs=100)

y_pred = mlp.predict(X_test_s)
print(f"Test Accuracy: {(y_pred == y_test).mean():.4f}")
```

---

## 9.7 Weight Initialization

Bad initialization can prevent the network from learning at all.

**Zero initialization:** All neurons compute the same gradient, learn the same thing. Never do this.

**Random normal:** Too large → gradients explode. Too small → gradients vanish.

**Xavier/Glorot initialization (sigmoid/tanh):**
```
W ~ N(0, 1/fan_in)   or    Uniform(-√(6/(fan_in+fan_out)), √(6/(fan_in+fan_out)))
```

**He initialization (ReLU):**
```
W ~ N(0, 2/fan_in)
```
The factor of 2 compensates for the expected 50% of activations being 0 after ReLU.

```python
# PyTorch initialization examples
import torch.nn as nn

linear = nn.Linear(256, 128)
nn.init.kaiming_normal_(linear.weight, mode='fan_in', nonlinearity='relu')  # He
nn.init.xavier_uniform_(linear.weight)   # Xavier
nn.init.zeros_(linear.bias)
```

---

## 9.8 Optimization Algorithms

### Momentum

Accumulates a velocity in the gradient direction:
```
v ← β·v + (1-β)·∇L(θ)
θ ← θ - η·v
```

`β ≈ 0.9` is typical. Smooths oscillations, accelerates progress in the right direction.

### Adam (Adaptive Moment Estimation)

The default optimizer for most tasks:
```
m ← β₁·m + (1-β₁)·g          # first moment (mean)
v ← β₂·v + (1-β₂)·g²         # second moment (uncentered variance)
m̂ = m/(1-β₁ᵗ)                # bias-corrected
v̂ = v/(1-β₂ᵗ)
θ ← θ - η·m̂/(√v̂ + ε)
```

Typical hyperparameters: `β₁ = 0.9, β₂ = 0.999, ε = 1e-8, η = 3e-4`.

Adam adapts the learning rate per parameter — large gradients → small effective step, small gradients → larger effective step. Very robust.

**AdamW:** Adam with weight decay decoupled from the gradient update. Better regularization than L2 in the gradient. Default in most modern frameworks.

```python
import torch
optimizer = torch.optim.AdamW(model.parameters(), lr=3e-4, weight_decay=0.01)
```

---

## 9.9 Regularization Techniques

### Dropout

During training, randomly zero out each neuron with probability p:

```python
nn.Dropout(p=0.5)  # 50% of neurons zeroed each forward pass
```

At test time, scale by (1-p) or equivalently set `training=False` (PyTorch handles this automatically).

**Why it works:** Forces the network to learn redundant representations. Prevents co-adaptation between neurons. Equivalent to training an exponential ensemble of networks.

**Where to use:** After each fully-connected layer. Typically p=0.2–0.5.

### Batch Normalization

Normalizes layer inputs within a mini-batch:
```
μ_B = (1/m) Σ xᵢ
σ²_B = (1/m) Σ (xᵢ - μ_B)²
x̂ᵢ = (xᵢ - μ_B) / √(σ²_B + ε)
yᵢ = γ x̂ᵢ + β            # learnable scale and shift
```

γ and β are learned parameters. During inference, uses running statistics from training.

**Benefits:**
- Smooths the loss landscape → allows higher learning rates
- Acts as mild regularization
- Reduces sensitivity to initialization

```python
nn.Sequential(
    nn.Linear(256, 128),
    nn.BatchNorm1d(128),
    nn.ReLU(),
)
```

### L2 Weight Decay

Add `λ‖W‖²` to the loss. In optimizers, implemented as:
```python
optimizer = torch.optim.SGD(model.parameters(), lr=0.01, weight_decay=1e-4)
```

### Early Stopping

Monitor validation loss. Stop training when it starts increasing:
```python
from torch.optim.lr_scheduler import ReduceLROnPlateau

scheduler = ReduceLROnPlateau(optimizer, patience=10, factor=0.5)
# Training loop:
for epoch in range(max_epochs):
    train_loss = train_one_epoch(...)
    val_loss = evaluate(...)
    scheduler.step(val_loss)
    
    if val_loss < best_val_loss:
        best_val_loss = val_loss
        torch.save(model.state_dict(), 'best_model.pth')
    elif epochs_without_improvement > patience:
        break  # early stop
```

---

## 9.10 Neural Networks in PyTorch

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class MLP(nn.Module):
    def __init__(self, input_dim, hidden_dims, output_dim, dropout=0.2):
        super().__init__()
        
        layers = []
        in_dim = input_dim
        for h_dim in hidden_dims:
            layers.extend([
                nn.Linear(in_dim, h_dim),
                nn.BatchNorm1d(h_dim),
                nn.ReLU(),
                nn.Dropout(dropout)
            ])
            in_dim = h_dim
        layers.append(nn.Linear(in_dim, output_dim))
        
        self.network = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.network(x)


# Training loop
def train(model, loader, optimizer, criterion, device):
    model.train()
    total_loss = 0
    for X_batch, y_batch in loader:
        X_batch, y_batch = X_batch.to(device), y_batch.to(device)
        
        optimizer.zero_grad()
        output = model(X_batch)
        loss = criterion(output, y_batch)
        loss.backward()
        
        # Gradient clipping (prevents exploding gradients)
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        
        optimizer.step()
        total_loss += loss.item()
    return total_loss / len(loader)


# Setup
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = MLP(input_dim=64, hidden_dims=[256, 128], output_dim=10).to(device)
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)
criterion = nn.CrossEntropyLoss()
```

---

## Exercises

1. **Backprop by hand:** For a network with one hidden layer (sigmoid), compute the gradients of the MSE loss for a single training example. Verify against the autograd result in PyTorch.

2. **Vanishing gradient experiment:** Build a 10-layer network with sigmoid activations. Initialize randomly. On a random input, compute the gradient of the loss w.r.t. the first layer. How does it compare to the gradient of the last layer? Now repeat with ReLU. What do you observe?

3. **Adam vs SGD:** Train the same MLP on MNIST using SGD (with and without momentum) and Adam. Plot the learning curves. Which converges faster? Which achieves better final accuracy?

4. **Effect of dropout:** Train an MLP on a small dataset (200 samples, 50 features) with and without dropout. Plot train and test accuracy over epochs. What happens without dropout?

5. **From-scratch gradient check:** Implement `numerical_gradient` (from Ch 1) and compute gradients numerically for your MLP. Compare to your backprop implementation. They should match to within 1e-5.

---

## Further Reading

- *Deep Learning* — Goodfellow, Bengio, Courville — Chapters 6–8
- CS231n Notes (Stanford) — Neural Networks Parts 1, 2, 3 (excellent practical notes)
- Karpathy "A Hacker's Guide to Neural Networks" — builds backprop step by step
- PyTorch documentation — `pytorch.org/docs`
