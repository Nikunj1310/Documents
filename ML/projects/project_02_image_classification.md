# Project 2 — Image Classification with CNNs

**Covers:** Chapters 9–10  
**Estimated time:** 1–2 weeks  
**Difficulty:** Intermediate

---

## Project Goal

Build a high-accuracy image classifier using:
1. A CNN from scratch on CIFAR-10
2. Transfer learning (fine-tuned ResNet) on a custom dataset
3. Analysis of what the model actually learned

By the end, you will understand the full computer vision pipeline: data → augmentation → model → training → evaluation → visualization.

---

## Part A: CNN from Scratch on CIFAR-10

### Dataset

CIFAR-10: 60,000 32×32 color images in 10 classes.

```python
from torchvision import datasets, transforms
from torch.utils.data import DataLoader
import torch
import torch.nn as nn

# Transforms
train_transform = transforms.Compose([
    transforms.RandomCrop(32, padding=4),
    transforms.RandomHorizontalFlip(),
    transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2),
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010))
])

test_transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010))
])

train_dataset = datasets.CIFAR10('data/', train=True, download=True, transform=train_transform)
test_dataset = datasets.CIFAR10('data/', train=False, download=True, transform=test_transform)

train_loader = DataLoader(train_dataset, batch_size=128, shuffle=True, num_workers=4, pin_memory=True)
test_loader = DataLoader(test_dataset, batch_size=256, shuffle=False, num_workers=4)
```

### Build a VGG-style CNN

```python
class ConvBlock(nn.Module):
    def __init__(self, in_ch, out_ch, dropout=0.0):
        super().__init__()
        self.block = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.Conv2d(out_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Dropout2d(dropout)
        )
    def forward(self, x):
        return self.block(x)

class CIFAR10CNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        
        self.features = nn.Sequential(
            ConvBlock(3, 64, dropout=0.1),      # 32×32 → 16×16
            ConvBlock(64, 128, dropout=0.2),    # 16×16 → 8×8
            ConvBlock(128, 256, dropout=0.3),   # 8×8 → 4×4
            nn.Conv2d(256, 512, 3, padding=1),
            nn.BatchNorm2d(512),
            nn.ReLU(inplace=True),
            nn.AdaptiveAvgPool2d((1, 1))        # → 1×1
        )
        
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(512, 512),
            nn.BatchNorm1d(512),
            nn.ReLU(inplace=True),
            nn.Dropout(0.5),
            nn.Linear(512, num_classes)
        )
    
    def forward(self, x):
        return self.classifier(self.features(x))

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = CIFAR10CNN().to(device)
print(f"Parameters: {sum(p.numel() for p in model.parameters()):,}")
```

### Training with OneCycleLR

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)
scheduler = torch.optim.lr_scheduler.OneCycleLR(
    optimizer, max_lr=1e-2,
    steps_per_epoch=len(train_loader),
    epochs=100
)
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)

def train_epoch(model, loader, optimizer, criterion, scheduler):
    model.train()
    correct = total = running_loss = 0
    for images, labels in loader:
        images, labels = images.to(device), labels.to(device)
        
        optimizer.zero_grad()
        logits = model(images)
        loss = criterion(logits, labels)
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        optimizer.step()
        scheduler.step()
        
        running_loss += loss.item()
        correct += (logits.argmax(1) == labels).sum().item()
        total += labels.size(0)
    
    return running_loss / len(loader), correct / total

@torch.no_grad()
def evaluate(model, loader):
    model.eval()
    correct = total = 0
    all_preds, all_labels = [], []
    for images, labels in loader:
        images, labels = images.to(device), labels.to(device)
        logits = model(images)
        preds = logits.argmax(1)
        correct += (preds == labels).sum().item()
        total += labels.size(0)
        all_preds.extend(preds.cpu().numpy())
        all_labels.extend(labels.cpu().numpy())
    return correct / total, all_preds, all_labels

# Training loop
history = {'train_loss': [], 'train_acc': [], 'test_acc': []}
best_acc = 0

for epoch in range(100):
    train_loss, train_acc = train_epoch(model, train_loader, optimizer, criterion, scheduler)
    test_acc, preds, labels = evaluate(model, test_loader)
    
    history['train_loss'].append(train_loss)
    history['train_acc'].append(train_acc)
    history['test_acc'].append(test_acc)
    
    if test_acc > best_acc:
        best_acc = test_acc
        torch.save(model.state_dict(), 'best_cnn.pth')
    
    if (epoch + 1) % 10 == 0:
        print(f"Epoch {epoch+1}: loss={train_loss:.3f}, train={train_acc:.3f}, test={test_acc:.3f}")

print(f"\nBest test accuracy: {best_acc:.4f}")
# Target: > 90% accuracy
```

---

## Part B: Transfer Learning on Custom Dataset

Use any image dataset with 3+ classes. Good options:
- Download from Kaggle (dogs vs cats, flowers, etc.)
- Intel Image Classification (natural scenes)
- Food-101

```bash
# Structure your data as:
# data/
#   train/
#     class_a/  img1.jpg img2.jpg ...
#     class_b/  ...
#   val/
#     class_a/  ...
#     class_b/  ...
```

```python
import torchvision.models as models

# Load ResNet18 with pretrained weights
model = models.resnet18(weights='IMAGENET1K_V1')

# Freeze backbone
for param in model.parameters():
    param.requires_grad = False

# Replace classifier
num_classes = len(train_dataset.classes)
model.fc = nn.Sequential(
    nn.Dropout(0.2),
    nn.Linear(model.fc.in_features, num_classes)
)
model = model.to(device)

# Stage 1: Train classifier head only
optimizer = torch.optim.Adam(model.fc.parameters(), lr=1e-3)
# ... train for 10 epochs

# Stage 2: Unfreeze last block + classifier, fine-tune
for param in model.layer4.parameters():
    param.requires_grad = True
optimizer = torch.optim.Adam([
    {'params': model.layer4.parameters(), 'lr': 1e-4},
    {'params': model.fc.parameters(), 'lr': 1e-3}
])
# ... train for 20 more epochs
```

---

## Part C: Visualization & Interpretability

### Visualize What the Network Learned

```python
# 1. Filter visualization (what patterns activate each filter)
def visualize_filters(model):
    first_layer = model.features[0][0]  # first conv layer
    filters = first_layer.weight.detach().cpu()
    filters = (filters - filters.min()) / (filters.max() - filters.min())
    
    fig, axes = plt.subplots(4, 8, figsize=(16, 8))
    for i, ax in enumerate(axes.flatten()):
        if i < filters.shape[0]:
            ax.imshow(filters[i].permute(1, 2, 0).clamp(0, 1))
        ax.axis('off')
    plt.title('First Layer Filters')
    plt.show()

# 2. Grad-CAM: visualize which regions the model attended to for a prediction
class GradCAM:
    def __init__(self, model, target_layer):
        self.model = model
        self.gradients = None
        self.activations = None
        
        target_layer.register_forward_hook(self._save_activation)
        target_layer.register_full_backward_hook(self._save_gradient)
    
    def _save_activation(self, module, input, output):
        self.activations = output
    
    def _save_gradient(self, module, grad_input, grad_output):
        self.gradients = grad_output[0]
    
    def generate(self, x, class_idx=None):
        self.model.eval()
        output = self.model(x)
        if class_idx is None:
            class_idx = output.argmax().item()
        
        self.model.zero_grad()
        output[0, class_idx].backward()
        
        pooled_grads = self.gradients.mean(dim=[0, 2, 3])
        cam = (self.activations[0] * pooled_grads[:, None, None]).sum(dim=0)
        cam = torch.relu(cam)
        cam = (cam - cam.min()) / (cam.max() - cam.min() + 1e-8)
        
        return cam.detach().cpu().numpy()

# Apply Grad-CAM
cam = GradCAM(model, model.features[-2])  # last conv layer
image, label = test_dataset[42]
cam_map = cam.generate(image.unsqueeze(0).to(device))

# Overlay on image
import cv2
cam_resized = cv2.resize(cam_map, (32, 32))
heatmap = cv2.applyColorMap(np.uint8(255 * cam_resized), cv2.COLORMAP_JET)
img_np = (image.permute(1,2,0).numpy() * 0.25 + 0.5).clip(0,1)
overlay = cv2.addWeighted(np.uint8(img_np * 255), 0.5, heatmap, 0.5, 0)

plt.figure(figsize=(8, 3))
plt.subplot(1,3,1); plt.imshow(img_np); plt.title('Original')
plt.subplot(1,3,2); plt.imshow(cam_resized, cmap='hot'); plt.title('Grad-CAM')
plt.subplot(1,3,3); plt.imshow(overlay); plt.title('Overlay')
plt.show()
```

### Confusion Matrix Analysis

```python
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay

classes = test_dataset.classes
cm = confusion_matrix(all_labels, all_preds)
disp = ConfusionMatrixDisplay(cm, display_labels=classes)
fig, ax = plt.subplots(figsize=(10, 10))
disp.plot(ax=ax, cmap='Blues', xticks_rotation='vertical')
plt.title('CIFAR-10 Confusion Matrix')
plt.show()

# Which classes are most confused?
# What do the confused images look like?
```

---

## Deliverables

- [ ] CNN from scratch achieving > 88% test accuracy on CIFAR-10
- [ ] Learning curves (train and test accuracy vs epochs)
- [ ] Confusion matrix with analysis of common errors
- [ ] Transfer learning model achieving > 90% on custom dataset
- [ ] Grad-CAM visualization for at least 5 correctly and 5 incorrectly classified images
- [ ] Ablation study: accuracy with no augmentation, basic augmentation, full augmentation
- [ ] Report: What patterns do the first-layer filters detect?

## Questions to Answer

1. Why does label smoothing improve accuracy?
2. What is the effect of removing batch normalization?
3. Why does the learning rate warm up in OneCycleLR?
4. Which classes are most often confused, and why?
5. Do the Grad-CAM maps indicate the model is looking at the right parts of the image?

## Extension Ideas

- Implement CutMix augmentation from scratch
- Try EfficientNet or ConvNeXt instead of ResNet
- Implement knowledge distillation (train a small "student" network to mimic the large "teacher")
- Export to ONNX and measure speedup vs PyTorch inference
