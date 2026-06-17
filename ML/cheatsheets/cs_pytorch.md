# Cheat Sheet — PyTorch

> The operations you reach for most often, in one place.

---

## Tensors

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

# Creation
torch.tensor([1.0, 2.0])              # from data
torch.zeros(3, 4)
torch.ones(3, 4)
torch.eye(4)
torch.arange(0, 10, 2)
torch.linspace(0, 1, 5)
torch.randn(3, 4)                     # N(0,1)
torch.rand(3, 4)                      # uniform [0,1)
torch.randint(0, 10, (3, 4))
torch.full((3, 4), fill_value=5.0)
torch.empty(3, 4)                     # uninitialized

# From NumPy (shared memory!)
t = torch.from_numpy(arr)             # shares memory
arr = t.numpy()                       # shares memory (CPU only)
t = torch.tensor(arr)                 # copy

# Properties
t.shape      t.size()       t.dtype
t.device     t.requires_grad
t.numel()    t.ndim

# Device
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
t = t.to(device)
t = t.cuda()    t = t.cpu()
```

---

## Tensor Operations

```python
# Math
a + b    a - b    a * b    a / b    a ** 2
torch.matmul(a, b)    a @ b         # matrix multiply
torch.dot(a, b)                      # 1D dot product
torch.mm(a, b)                       # 2D matrix multiply
torch.bmm(a, b)                      # batch matrix multiply (B, m, k) × (B, k, n)

# Elementwise
torch.exp(t)   torch.log(t)   torch.sqrt(t)   torch.abs(t)
torch.sigmoid(t)  torch.tanh(t)  F.relu(t)  F.softmax(t, dim=-1)
torch.clamp(t, min=0, max=1)

# Reductions
t.sum()          t.sum(dim=0)        t.sum(dim=1, keepdim=True)
t.mean()         t.mean(dim=-1)
t.max()          t.max(dim=1)        # returns (values, indices)
t.min()          t.argmax(dim=1)     # just indices
t.norm()         t.norm(p=1)

# Comparison
t > 0            t == other          torch.allclose(a, b, atol=1e-5)
```

---

## Reshaping

```python
t.view(3, 4)           # must be contiguous
t.reshape(3, 4)        # works always (may copy)
t.flatten()            # → 1D
t.squeeze()            # remove all size-1 dims
t.squeeze(0)           # remove dim 0 if size-1
t.unsqueeze(0)         # add dim at position 0
t.permute(2, 0, 1)     # reorder dims (like transpose for N-D)
t.transpose(0, 1)      # swap two dims
t.contiguous()         # make memory contiguous

# Stacking
torch.cat([a, b], dim=0)     # concat along existing dim
torch.stack([a, b], dim=0)   # new dim
torch.chunk(t, chunks=3, dim=0)   # split into equal chunks
torch.split(t, split_size=2, dim=0)
```

---

## Autograd

```python
x = torch.tensor(3.0, requires_grad=True)
y = x ** 2 + 2*x + 1
y.backward()
print(x.grad)   # dy/dx = 2x+2 = 8

# Disable gradient computation (inference)
with torch.no_grad():
    output = model(x)

# Or: detach from computation graph
z = x.detach()

# Manual gradient zeroing (MUST do before backward)
optimizer.zero_grad()
loss.backward()
optimizer.step()

# Gradient clipping
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
torch.nn.utils.clip_grad_value_(model.parameters(), clip_value=0.5)
```

---

## nn.Module

```python
class MyModel(nn.Module):
    def __init__(self, input_dim, hidden_dim, output_dim):
        super().__init__()
        # All nn.Module and nn.Parameter attributes are auto-registered
        self.fc1 = nn.Linear(input_dim, hidden_dim)
        self.bn  = nn.BatchNorm1d(hidden_dim)
        self.dropout = nn.Dropout(0.3)
        self.fc2 = nn.Linear(hidden_dim, output_dim)
    
    def forward(self, x):
        x = F.relu(self.bn(self.fc1(x)))
        x = self.dropout(x)
        return self.fc2(x)

model = MyModel(64, 128, 10)
model.to(device)
model.train()       # enables dropout, batchnorm training mode
model.eval()        # disables dropout, uses running stats for BN

# Inspect
sum(p.numel() for p in model.parameters())                     # total params
sum(p.numel() for p in model.parameters() if p.requires_grad)  # trainable params
```

---

## Common Layers

```python
nn.Linear(in, out, bias=True)
nn.Conv2d(in_ch, out_ch, kernel_size, stride=1, padding=0, groups=1)
nn.ConvTranspose2d(in_ch, out_ch, kernel_size, stride=1, padding=0)
nn.MaxPool2d(kernel_size, stride=None)
nn.AvgPool2d(kernel_size, stride=None)
nn.AdaptiveAvgPool2d(output_size)      # e.g., (1,1) for global avg pool
nn.BatchNorm1d(num_features)
nn.BatchNorm2d(num_features)
nn.LayerNorm(normalized_shape)
nn.Dropout(p=0.5)
nn.Dropout2d(p=0.5)
nn.Embedding(num_embeddings, embedding_dim, padding_idx=0)
nn.LSTM(input_size, hidden_size, num_layers, batch_first=True, bidirectional=False)
nn.GRU(input_size, hidden_size, num_layers, batch_first=True)
nn.MultiheadAttention(embed_dim, num_heads, dropout=0.0, batch_first=True)
nn.Transformer(d_model, nhead, num_encoder_layers, num_decoder_layers)
```

---

## Loss Functions

```python
nn.CrossEntropyLoss()                    # combines LogSoftmax + NLLLoss
                                          # input: (N, C) logits, target: (N,) class indices
nn.CrossEntropyLoss(weight=class_weights) # for imbalanced data
nn.CrossEntropyLoss(label_smoothing=0.1)  # label smoothing
nn.BCEWithLogitsLoss()                   # binary CE with numerical stability
nn.MSELoss()                             # mean squared error
nn.L1Loss()                              # mean absolute error
nn.SmoothL1Loss()                        # Huber loss (robust regression)
nn.KLDivLoss(reduction='batchmean')      # KL divergence (input must be log-probs)
nn.NLLLoss()                             # negative log likelihood

# Contrastive / embedding losses
nn.CosineEmbeddingLoss()
nn.TripletMarginLoss()
```

---

## Optimizers

```python
torch.optim.SGD(params, lr=0.01, momentum=0.9, weight_decay=1e-4)
torch.optim.Adam(params, lr=1e-3, betas=(0.9, 0.999), weight_decay=0)
torch.optim.AdamW(params, lr=1e-3, betas=(0.9, 0.999), weight_decay=0.01)  # preferred
torch.optim.RMSprop(params, lr=1e-3, alpha=0.99)
torch.optim.LBFGS(params, lr=1, max_iter=20)   # for small problems, very fast

# Per-layer learning rates:
optimizer = torch.optim.AdamW([
    {'params': model.backbone.parameters(), 'lr': 1e-4},
    {'params': model.head.parameters(), 'lr': 1e-3}
], lr=1e-3, weight_decay=0.01)
```

---

## Learning Rate Schedulers

```python
# Attach after optimizer
sched = torch.optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.5)
sched = torch.optim.lr_scheduler.MultiStepLR(optimizer, milestones=[30,80], gamma=0.1)
sched = torch.optim.lr_scheduler.ExponentialLR(optimizer, gamma=0.95)
sched = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=100)
sched = torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer, patience=10, factor=0.5)
sched = torch.optim.lr_scheduler.OneCycleLR(optimizer, max_lr=1e-2,
    steps_per_epoch=len(loader), epochs=50)   # warmup + cosine annealing
sched = torch.optim.lr_scheduler.CosineAnnealingWarmRestarts(optimizer, T_0=10)

# Call after each epoch (or each step for OneCycleLR):
sched.step()                           # most schedulers
sched.step(val_loss)                   # ReduceLROnPlateau
```

---

## Training Loop Template

```python
def train_epoch(model, loader, optimizer, criterion, device, scheduler=None):
    model.train()
    total_loss = correct = total = 0
    
    for batch in loader:
        X, y = batch[0].to(device), batch[1].to(device)
        
        optimizer.zero_grad()
        
        with torch.autocast(device_type='cuda', dtype=torch.float16):  # mixed precision
            logits = model(X)
            loss = criterion(logits, y)
        
        scaler.scale(loss).backward()
        scaler.unscale_(optimizer)
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        scaler.step(optimizer)
        scaler.update()
        
        if scheduler is not None and not isinstance(scheduler, 
                                                      torch.optim.lr_scheduler.ReduceLROnPlateau):
            scheduler.step()
        
        total_loss += loss.item()
        if logits.dim() == 2:
            correct += (logits.argmax(1) == y).sum().item()
            total += y.size(0)
    
    metrics = {'loss': total_loss / len(loader)}
    if total > 0:
        metrics['accuracy'] = correct / total
    return metrics

@torch.no_grad()
def evaluate(model, loader, criterion, device):
    model.eval()
    total_loss = correct = total = 0
    all_preds, all_targets = [], []
    
    for batch in loader:
        X, y = batch[0].to(device), batch[1].to(device)
        logits = model(X)
        loss = criterion(logits, y)
        
        total_loss += loss.item()
        preds = logits.argmax(1)
        correct += (preds == y).sum().item()
        total += y.size(0)
        all_preds.extend(preds.cpu().numpy())
        all_targets.extend(y.cpu().numpy())
    
    return {'loss': total_loss/len(loader), 'accuracy': correct/total,
            'preds': all_preds, 'targets': all_targets}

# Initialization
scaler = torch.cuda.amp.GradScaler()    # for mixed precision
best_val_loss = float('inf')

for epoch in range(n_epochs):
    train_metrics = train_epoch(model, train_loader, optimizer, criterion, device, scheduler)
    val_metrics = evaluate(model, val_loader, criterion, device)
    
    if isinstance(scheduler, torch.optim.lr_scheduler.ReduceLROnPlateau):
        scheduler.step(val_metrics['loss'])
    
    if val_metrics['loss'] < best_val_loss:
        best_val_loss = val_metrics['loss']
        torch.save(model.state_dict(), 'best_model.pth')
    
    print(f"Epoch {epoch+1:3d} | "
          f"Train Loss: {train_metrics['loss']:.4f} | "
          f"Val Loss: {val_metrics['loss']:.4f} | "
          f"Val Acc: {val_metrics['accuracy']:.4f}")
```

---

## Saving & Loading

```python
# Save
torch.save(model.state_dict(), 'weights.pth')
torch.save({'epoch': epoch, 'model': model.state_dict(),
            'optimizer': optimizer.state_dict(), 'loss': best_loss}, 'checkpoint.pth')

# Load
model.load_state_dict(torch.load('weights.pth', map_location=device))
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model'])
optimizer.load_state_dict(checkpoint['optimizer'])

model.eval()  # don't forget!
```

---

## DataLoader

```python
from torch.utils.data import DataLoader, Dataset, random_split, Subset, WeightedRandomSampler

class CustomDataset(Dataset):
    def __init__(self, X, y, transform=None):
        self.X = torch.tensor(X, dtype=torch.float32)
        self.y = torch.tensor(y, dtype=torch.long)
        self.transform = transform
    def __len__(self): return len(self.y)
    def __getitem__(self, idx):
        x = self.X[idx]
        if self.transform: x = self.transform(x)
        return x, self.y[idx]

loader = DataLoader(dataset, batch_size=64, shuffle=True,
                    num_workers=4, pin_memory=True, drop_last=False)

# Imbalanced dataset: oversample minority class
sample_weights = compute_sample_weights(y_train)  # weight ∝ 1/class_count
sampler = WeightedRandomSampler(sample_weights, num_samples=len(sample_weights), replacement=True)
loader = DataLoader(dataset, batch_size=64, sampler=sampler)

# Split dataset
n = len(dataset)
train_ds, val_ds = random_split(dataset, [int(0.8*n), n - int(0.8*n)])
```
