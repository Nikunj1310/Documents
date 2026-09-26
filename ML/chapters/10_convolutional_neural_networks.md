# Chapter 10 — Convolutional Neural Networks

> *CNNs are the dominant architecture for visual data. They work by exploiting spatial structure — nearby pixels are related, and patterns (edges, textures, shapes) appear at different locations. Understanding the convolution operation deeply is essential.*

---

## 10.1 Why Not Just Use an MLP?

For a 256×256 RGB image: 256 × 256 × 3 = 196,608 input features. A single hidden layer with 1,024 neurons would need 201 million parameters. This:
- Is extremely parameter-inefficient
- Doesn't exploit spatial structure (each pixel treated independently)
- Massively overfits on typical datasets

CNNs solve this by:
1. **Local connectivity:** Neurons connect to small local regions, not all inputs
2. **Weight sharing:** The same filter is applied at every location
3. **Hierarchical features:** Stack layers to detect progressively complex patterns

---

## 10.2 The Convolution Operation

### 2D Convolution

A **filter** (or kernel) `K` of size `f×f` is slid across the input `I` (of size `H×W`):
```
(I * K)[i, j] = Σ_{p=0}^{f-1} Σ_{q=0}^{f-1} I[i+p, j+q] · K[p, q]
```

Each position of the filter computes a dot product with the local patch it covers. The output is a **feature map**.

**For color images (H × W × C_in):** The filter has shape `f × f × C_in`. The dot product is over all spatial and channel dimensions. One filter produces one feature map.

**For multiple filters (C_out filters):** Output shape is `H_out × W_out × C_out`.

### Output Size

For input `H × W`, filter `f × f`, padding `p`, stride `s`:
```
H_out = ⌊(H + 2p - f) / s⌋ + 1
W_out = ⌊(W + 2p - f) / s⌋ + 1
```

**Parameter count for one conv layer:**
```
# params = f × f × C_in × C_out + C_out (biases)
```

vs. a fully-connected layer: `(H_in × W_in × C_in) × (H_out × W_out × C_out)`. Conv layers are vastly more parameter-efficient.

### Padding

**Valid padding (p=0):** No padding. Output is smaller than input.

**Same padding:** Pad input with zeros to keep output size equal to input size. For stride=1: `p = (f-1)/2`.

### Stride

Skip positions as we slide the filter. Stride=2 halves the spatial dimensions while covering the same area as stride=1 but only computing half the positions.

### Intuition: What Does a Filter Detect?

- A horizontal edge detector filter: `[[-1,-1,-1],[0,0,0],[1,1,1]]`
- A vertical edge detector filter: same transposed
- Early layers: edges, corners, colors
- Middle layers: textures, patterns
- Later layers: parts of objects, high-level features
- Final layers: object-level representations

This hierarchy builds automatically through training.

---

## 10.3 Pooling

Pooling reduces spatial dimensions while retaining important information.

**Max pooling:** Take the maximum value in each `f×f` window.
- Provides translation invariance (feature present, regardless of exact position)
- Reduces computation

**Average pooling:** Take the mean. Smoother than max pooling.

**Global Average Pooling (GAP):** Pool the entire feature map to a single value per channel. Extremely efficient. Used in modern architectures instead of fully-connected layers.

```python
# In PyTorch
nn.MaxPool2d(kernel_size=2, stride=2)      # halves H and W
nn.AvgPool2d(kernel_size=2, stride=2)
nn.AdaptiveAvgPool2d(output_size=(1, 1))   # global average pooling
```

---

## 10.4 A Complete CNN

Typical CNN architecture:
```
Input: (N, 3, H, W)
    ↓
Conv(32 filters, 3×3, padding=1) → ReLU → (N, 32, H, W)
MaxPool(2×2) → (N, 32, H/2, W/2)
    ↓
Conv(64 filters, 3×3, padding=1) → ReLU → (N, 64, H/2, W/2)
MaxPool(2×2) → (N, 64, H/4, W/4)
    ↓
Conv(128 filters, 3×3, padding=1) → ReLU → (N, 128, H/4, W/4)
GlobalAvgPool → (N, 128)
    ↓
Linear(128 → 10) → Softmax
```

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SimpleCNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        
        self.features = nn.Sequential(
            # Block 1
            nn.Conv2d(3, 32, kernel_size=3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Dropout2d(0.1),
            
            # Block 2
            nn.Conv2d(32, 64, kernel_size=3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Dropout2d(0.2),
            
            # Block 3
            nn.Conv2d(64, 128, kernel_size=3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.AdaptiveAvgPool2d((1, 1))  # global average pool
        )
        
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(128, 256),
            nn.ReLU(inplace=True),
            nn.Dropout(0.5),
            nn.Linear(256, num_classes)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x
```

---

## 10.5 Classic Architectures

### LeNet-5 (1998)

The first successful CNN. Used for handwritten digit recognition.
- 2 conv layers, 2 pooling layers, 2 fully-connected layers
- Used tanh/sigmoid activations
- Only ~60K parameters

### AlexNet (2012)

Won ImageNet competition, triggered the deep learning revolution.
- 5 conv layers, 3 fully-connected layers
- First use of ReLU in a deep CNN
- First use of dropout (p=0.5) in classification network
- ~60M parameters

### VGG (2014)

Very deep networks using only 3×3 convolutions throughout. VGG16 (16 layers) achieved excellent results. Key insight: small filters + depth > large filters + shallow.

### GoogLeNet / Inception (2014)

Introduced **Inception modules**: apply multiple filter sizes in parallel and concatenate:
```
Input → [Conv 1×1 | Conv 3×3 | Conv 5×5 | MaxPool 3×3] → Concatenate
```

Also introduced global average pooling instead of fully-connected layers, reducing parameters by 10×.

### ResNet (2015) — The Breakthrough

Introduced **residual connections** (skip connections):
```
Output = F(x, W) + x    ← the "shortcut"
```

Without residuals, networks deeper than ~20 layers got *worse* (degradation problem, related to vanishing gradients). Residuals allow gradients to flow directly backward, enabling 50, 101, 152+ layer networks.

```python
class ResidualBlock(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.block = nn.Sequential(
            nn.Conv2d(channels, channels, 3, padding=1),
            nn.BatchNorm2d(channels),
            nn.ReLU(inplace=True),
            nn.Conv2d(channels, channels, 3, padding=1),
            nn.BatchNorm2d(channels)
        )
        self.relu = nn.ReLU(inplace=True)
    
    def forward(self, x):
        return self.relu(x + self.block(x))   # skip connection
```

**Why residuals work:** The network only needs to learn the *residual* `F(x) = H(x) - x` instead of the full mapping. If the optimal function is close to the identity, the residual is close to zero — easy to learn.

---

## 10.6 Transfer Learning

Training a CNN from scratch requires millions of labeled images. Transfer learning lets you reuse a pre-trained network's learned features.

**Concept:** ImageNet-pretrained ResNet has learned to detect edges, textures, shapes, and object parts. These features are useful for almost any visual task.

### Feature Extraction

Freeze all pretrained weights, replace only the final classifier:
```python
import torchvision.models as models

# Load pretrained ResNet
model = models.resnet50(weights='IMAGENET1K_V2')

# Freeze all parameters
for param in model.parameters():
    param.requires_grad = False

# Replace the final layer (ResNet50 outputs 2048 features)
num_classes = 10
model.fc = nn.Linear(2048, num_classes)   # only this is trained

optimizer = torch.optim.Adam(model.fc.parameters(), lr=1e-3)
```

### Fine-tuning

Unfreeze some or all layers and train with a low learning rate:
```python
# Unfreeze all layers
for param in model.parameters():
    param.requires_grad = True

# Lower learning rate for pretrained layers, higher for new layer
optimizer = torch.optim.Adam([
    {'params': model.layer4.parameters(), 'lr': 1e-4},
    {'params': model.fc.parameters(), 'lr': 1e-3},
], lr=1e-5)
```

**When to use what:**
| Dataset Size | Similarity to ImageNet | Strategy |
|---|---|---|
| Small (< 1000) | High | Feature extraction only |
| Small | Low | Fine-tune last few layers |
| Large (> 10k) | High | Fine-tune all layers |
| Large | Low | Train from scratch or fine-tune all |

---

## 10.7 Data Augmentation

Artificially expand the dataset by applying label-preserving transformations:

```python
from torchvision import transforms

train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224, scale=(0.7, 1.0)),
    transforms.RandomHorizontalFlip(),
    transforms.ColorJitter(brightness=0.3, contrast=0.3, saturation=0.3),
    transforms.RandomRotation(15),
    transforms.RandomGrayscale(p=0.1),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],   # ImageNet stats
                         std=[0.229, 0.224, 0.225])
])

val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225])
])
```

**Advanced augmentations:**
- **CutOut/Random Erasing:** Randomly mask rectangular regions
- **Mixup:** Blend two images and their labels
- **CutMix:** Replace a patch from one image with a patch from another
- **RandAugment:** Randomly select from a large augmentation space

---

## 10.8 Training Pipeline

```python
from torch.utils.data import DataLoader
from torchvision.datasets import ImageFolder
import torch

# Data
train_dataset = ImageFolder('data/train', transform=train_transform)
val_dataset = ImageFolder('data/val', transform=val_transform)

train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True, 
                          num_workers=4, pin_memory=True)
val_loader = DataLoader(val_dataset, batch_size=64, shuffle=False, num_workers=4)

# Model
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = models.resnet50(weights='IMAGENET1K_V2')
model.fc = nn.Linear(2048, num_classes)
model = model.to(device)

# Optimizer + scheduler
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)
scheduler = torch.optim.lr_scheduler.OneCycleLR(
    optimizer, max_lr=1e-2, steps_per_epoch=len(train_loader), epochs=50
)
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)

def train_epoch(model, loader, optimizer, criterion):
    model.train()
    correct = total = 0
    running_loss = 0
    
    for X, y in loader:
        X, y = X.to(device), y.to(device)
        
        optimizer.zero_grad()
        logits = model(X)
        loss = criterion(logits, y)
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        optimizer.step()
        scheduler.step()
        
        running_loss += loss.item()
        correct += (logits.argmax(1) == y).sum().item()
        total += y.size(0)
    
    return running_loss / len(loader), correct / total

@torch.no_grad()
def evaluate(model, loader):
    model.eval()
    correct = total = 0
    for X, y in loader:
        X, y = X.to(device), y.to(device)
        logits = model(X)
        correct += (logits.argmax(1) == y).sum().item()
        total += y.size(0)
    return correct / total
```

---

## Exercises

1. **Convolution by hand:** Apply a 3×3 edge-detection filter (Sobel operator) to a 5×5 grayscale image manually. Compute the output size for padding=0, stride=1. Verify with PyTorch's `F.conv2d`.

2. **Visualize filters:** Train a small CNN on CIFAR-10. Visualize the first-layer filters as 3×3 images. Do they look like edge detectors? Visualize the feature maps produced by those filters for a sample image.

3. **Transfer learning experiment:** Take ResNet18 pretrained on ImageNet. Fine-tune it on CIFAR-10 (only 10 classes). Compare to: (a) training from scratch, (b) feature extraction only. Plot accuracy and training time for all three.

4. **Receptive field calculation:** For a network with three 3×3 conv layers (all stride=1, padding=1) followed by a 2×2 max pool (stride=2), calculate the receptive field of a neuron in the final layer on a 32×32 input.

5. **Augmentation ablation:** Train the same CNN three times: (a) no augmentation, (b) random flip + crop, (c) full augmentation suite. Report validation accuracy for each. How much does augmentation help?

---

## Further Reading

- CS231n Stanford — Image Recognition Course (excellent notes at cs231n.github.io)
- *Deep Learning* — Goodfellow et al. — Chapter 9 (Convolutional Networks)
- *Dive into Deep Learning* — Zhang et al. (free at d2l.ai) — Chapters 6–8
- *He et al. (2016) Deep Residual Learning for Image Recognition* (ResNet paper)
