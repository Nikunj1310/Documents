# Chapter 13 — Generative Models

> *Discriminative models learn P(y|x) — to classify. Generative models learn P(x) — to generate. Understanding VAEs, GANs, and diffusion models opens the door to image synthesis, text generation, and data augmentation.*

---

## 13.1 Generative vs Discriminative

**Discriminative models:** Learn the decision boundary between classes. Examples: logistic regression, SVM, BERT for classification.

**Generative models:** Learn the full data distribution P(x). Can:
- Generate new samples
- Estimate density (is this point likely under the data distribution?)
- Impute missing data

**Two approaches:**
1. **Explicit density estimation:** Directly learn and evaluate P(x) (VAEs, autoregressive models)
2. **Implicit density:** Learn to sample from P(x) without explicit density (GANs)

---

## 13.2 Variational Autoencoders (VAEs)

### Autoencoders

An **autoencoder** learns to compress data into a latent code and reconstruct it:
```
x → Encoder → z → Decoder → x̂
```

The bottleneck forces z to capture essential information. Trained to minimize `‖x - x̂‖²`.

Problem: The latent space is unstructured — you can't randomly sample z and get a meaningful x.

### The Variational Extension

A **VAE** forces the latent space to be smooth and structured, suitable for sampling.

Instead of encoding x to a single point z, the encoder outputs distribution parameters:
```
Encoder: x → (μ, σ²)    [parameters of q_φ(z|x) = N(μ, σ²)]
Sample:  z ~ N(μ, σ²)    [using reparameterization]
Decoder: z → x̂           [p_θ(x|z)]
```

### The ELBO (Evidence Lower Bound)

The VAE optimizes the ELBO:
```
ELBO = E_{q_φ(z|x)}[log p_θ(x|z)] - KL(q_φ(z|x) ‖ p(z))
```

- **Reconstruction term:** `E[log p_θ(x|z)]` — how well does the decoder reconstruct x?
- **KL term:** `KL(q_φ(z|x) ‖ N(0,I))` — how close is the posterior to a standard Gaussian?

The KL term regularizes the latent space to be smooth (close to N(0,I)). This ensures nearby latent points decode to similar outputs.

For Gaussian q and prior N(0,I):
```
KL = ½ Σⱼ (μⱼ² + σⱼ² - log σⱼ² - 1)
```

### The Reparameterization Trick

Sampling `z ~ N(μ, σ²)` is not differentiable (the randomness blocks the gradient). Solution:

```
z = μ + σ ⊙ ε,    where ε ~ N(0, I)
```

Now z is a deterministic function of ε (which has no parameters). Gradients flow through μ and σ normally.

### VAE Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class VAE(nn.Module):
    def __init__(self, input_dim=784, hidden_dim=400, latent_dim=20):
        super().__init__()
        
        # Encoder
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, hidden_dim),
            nn.ReLU()
        )
        self.fc_mu = nn.Linear(hidden_dim, latent_dim)
        self.fc_logvar = nn.Linear(hidden_dim, latent_dim)
        
        # Decoder
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, input_dim),
            nn.Sigmoid()   # output is [0,1] pixel values
        )
    
    def encode(self, x):
        h = self.encoder(x)
        return self.fc_mu(h), self.fc_logvar(h)
    
    def reparameterize(self, mu, logvar):
        std = torch.exp(0.5 * logvar)
        eps = torch.randn_like(std)
        return mu + eps * std
    
    def decode(self, z):
        return self.decoder(z)
    
    def forward(self, x):
        mu, logvar = self.encode(x)
        z = self.reparameterize(mu, logvar)
        return self.decode(z), mu, logvar
    
    def loss(self, x_recon, x, mu, logvar, beta=1.0):
        # Reconstruction loss (binary cross-entropy for image pixels)
        BCE = F.binary_cross_entropy(x_recon, x, reduction='sum')
        # KL divergence
        KLD = -0.5 * torch.sum(1 + logvar - mu.pow(2) - logvar.exp())
        return BCE + beta * KLD

# Usage
model = VAE()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

for batch in dataloader:
    x = batch[0].view(-1, 784)
    x_recon, mu, logvar = model(x)
    loss = model.loss(x_recon, x, mu, logvar)
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

# Generate new samples
with torch.no_grad():
    z = torch.randn(16, 20)   # sample from prior N(0,I)
    samples = model.decode(z)
    
# Interpolate in latent space
z1, z2 = model.encode(x1)[0], model.encode(x2)[0]
interp = torch.lerp(z1, z2, torch.linspace(0, 1, 8).unsqueeze(1))
interpolated = model.decode(interp)
```

---

## 13.3 Generative Adversarial Networks (GANs)

### The Idea

Train two networks in a game:
- **Generator G:** Takes random noise z → generates fake data G(z)
- **Discriminator D:** Takes real or fake data → outputs probability that it's real

The generator tries to fool the discriminator. The discriminator tries to tell real from fake. They push each other to improve.

### The Minimax Objective

```
min_G max_D V(D, G) = E_{x~p_data}[log D(x)] + E_{z~p_z}[log(1 - D(G(z)))]
```

- D maximizes: correctly classify real (D(x) → 1) and fake (D(G(z)) → 0)
- G minimizes: fool D into classifying fakes as real (D(G(z)) → 1)

In practice, train G to maximize `log D(G(z))` (not minimize `log(1-D(G(z)))`) for stronger gradients early in training.

### Training Loop

```python
class Generator(nn.Module):
    def __init__(self, z_dim=100, img_dim=784):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(z_dim, 256), nn.LeakyReLU(0.2),
            nn.Linear(256, 512),   nn.LeakyReLU(0.2),
            nn.Linear(512, 1024),  nn.LeakyReLU(0.2),
            nn.Linear(1024, img_dim), nn.Tanh()   # output in [-1, 1]
        )
    def forward(self, z): return self.net(z)

class Discriminator(nn.Module):
    def __init__(self, img_dim=784):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(img_dim, 1024), nn.LeakyReLU(0.2), nn.Dropout(0.3),
            nn.Linear(1024, 512),     nn.LeakyReLU(0.2), nn.Dropout(0.3),
            nn.Linear(512, 256),      nn.LeakyReLU(0.2), nn.Dropout(0.3),
            nn.Linear(256, 1),        nn.Sigmoid()
        )
    def forward(self, x): return self.net(x)

G = Generator().to(device)
D = Discriminator().to(device)

opt_G = torch.optim.Adam(G.parameters(), lr=2e-4, betas=(0.5, 0.999))
opt_D = torch.optim.Adam(D.parameters(), lr=2e-4, betas=(0.5, 0.999))
criterion = nn.BCELoss()
z_dim = 100

for epoch in range(num_epochs):
    for real_imgs, _ in dataloader:
        real_imgs = real_imgs.view(-1, 784).to(device)
        batch_size = real_imgs.size(0)
        real_labels = torch.ones(batch_size, 1).to(device)
        fake_labels = torch.zeros(batch_size, 1).to(device)
        
        # ── Train Discriminator ──────────────────────────────
        z = torch.randn(batch_size, z_dim).to(device)
        fake_imgs = G(z).detach()   # detach: don't update G here
        
        loss_D = (criterion(D(real_imgs), real_labels) +
                  criterion(D(fake_imgs), fake_labels)) / 2
        
        opt_D.zero_grad()
        loss_D.backward()
        opt_D.step()
        
        # ── Train Generator ──────────────────────────────────
        z = torch.randn(batch_size, z_dim).to(device)
        fake_imgs = G(z)
        
        loss_G = criterion(D(fake_imgs), real_labels)  # want D to say "real"
        
        opt_G.zero_grad()
        loss_G.backward()
        opt_G.step()
```

### GAN Problems

**Mode collapse:** The generator learns to produce only a few types of samples (the modes D has most trouble detecting) instead of the full diversity of the real data.

**Training instability:** The two-player game is hard to balance. If D is too strong, G gets no gradient. If G is too strong, D provides no useful signal.

**Solutions:**
- **Wasserstein GAN (WGAN):** Replace binary cross-entropy with Earth Mover's Distance. More stable training, meaningful loss metric.
- **Spectral normalization:** Constrain D's Lipschitz constant. Stabilizes training.
- **Progressive growing (ProGAN):** Start with 4×4 images, gradually grow to full resolution.
- **StyleGAN:** Separate "style" and "content" in the latent space. Produces remarkably high-quality, controllable images.

### Conditional GAN (cGAN)

Generate samples conditioned on a class label:
```python
# Feed label embeddings to both G and D
class ConditionalGenerator(nn.Module):
    def __init__(self, z_dim, num_classes, img_dim):
        super().__init__()
        self.label_emb = nn.Embedding(num_classes, num_classes)
        self.net = nn.Sequential(
            nn.Linear(z_dim + num_classes, 256),
            nn.LeakyReLU(0.2),
            # ...
            nn.Linear(256, img_dim),
            nn.Tanh()
        )
    def forward(self, z, labels):
        label_emb = self.label_emb(labels)
        inp = torch.cat([z, label_emb], dim=1)
        return self.net(inp)
```

---

## 13.4 Diffusion Models

Diffusion models (2020–present) are the current state-of-the-art for image generation, outperforming GANs on quality and diversity.

### The Forward Process (Adding Noise)

Gradually add Gaussian noise to data over T steps:
```
q(xₜ | xₜ₋₁) = N(xₜ; √(1-βₜ) xₜ₋₁, βₜI)
```

After enough steps (T ≈ 1000), xT ≈ N(0, I) — pure noise.

Using the reparameterization with αₜ = 1 - βₜ and ᾱₜ = Πₛ₌₁ᵗ αₛ:
```
q(xₜ | x₀) = N(xₜ; √ᾱₜ x₀, (1-ᾱₜ)I)
```

So xₜ = √ᾱₜ x₀ + √(1-ᾱₜ) ε,  ε ~ N(0,I) — can directly sample xₜ given x₀.

### The Reverse Process (Denoising)

A neural network (U-Net) learns to denoise:
```
p_θ(xₜ₋₁ | xₜ) = N(xₜ₋₁; μ_θ(xₜ, t), Σ_θ(xₜ, t))
```

The network is trained to predict the noise ε added at step t:
```
L = E_{t, x₀, ε}[‖ε - ε_θ(√ᾱₜ x₀ + √(1-ᾱₜ) ε, t)‖²]
```

### Generation

Start with xT ~ N(0, I). Iteratively denoise:
```
For t = T, T-1, ..., 1:
    ε_pred = ε_θ(xₜ, t)
    xₜ₋₁ = denoising_step(xₜ, ε_pred, t)  [plus a bit of noise except at t=1]
```

### Why Diffusion Models Work Well

- **Stable training:** The objective is simple MSE on noise prediction. No adversarial game.
- **Coverage of full data distribution:** Each denoising step is probabilistic → explores diverse modes.
- **Guided generation:** Easy to condition on text, class, or other signals via classifier-free guidance.

### Classifier-Free Guidance

Train the same model with and without conditioning (randomly drop condition during training). At inference, extrapolate:
```
ε_guided = ε_uncond + w * (ε_cond - ε_uncond)
```

Guidance scale `w > 1` amplifies the conditioning signal → more faithful to condition, less diverse.

---

## 13.5 Evaluating Generative Models

Unlike discriminative models, evaluating generative quality is hard.

**Inception Score (IS):** High quality images → high confidence predictions (sharp). Diverse images → varied predictions. IS = exp(E[KL(p(y|x) ‖ p(y))]).

**Fréchet Inception Distance (FID):** Compare statistics (mean and covariance) of features extracted from real and generated images using InceptionNet. Lower FID = better.

```python
# Using torchmetrics
from torchmetrics.image.fid import FrechetInceptionDistance

fid = FrechetInceptionDistance(feature=2048)
fid.update(real_imgs, real=True)
fid.update(generated_imgs, real=False)
print(f"FID: {fid.compute():.2f}")
```

**LPIPS (Learned Perceptual Image Patch Similarity):** Perceptual similarity between images. Lower = more similar to reference.

---

## Exercises

1. **VAE latent space:** Train a VAE on MNIST. Visualize the 2D latent space (use 2D latent dim) with colors for each digit class. Does the latent space cluster nicely? Now traverse a path between two clusters — what do the decoded images look like?

2. **β-VAE:** Train a VAE with `β=1` and `β=4` (upweight KL term). Compare the reconstructions and the disentanglement of the latent space. What does higher β do to the latent space structure?

3. **GAN training instability:** Train a GAN on MNIST. Plot the generator and discriminator losses over training. Identify if and when mode collapse occurs. Try adding label noise to the discriminator — does training stabilize?

4. **Conditional generation:** Implement a conditional GAN. After training, generate samples from each class label. Can you interpolate between classes in the latent space?

5. **FID comparison:** Generate 1,000 samples from your VAE and your GAN. Compute FID for both against the real MNIST test set. Which generates higher quality samples?

---

## Further Reading

- Kingma & Welling (2013) "Auto-Encoding Variational Bayes" — original VAE paper
- Goodfellow et al. (2014) "Generative Adversarial Networks" — original GAN paper
- Ho et al. (2020) "Denoising Diffusion Probabilistic Models" — DDPM paper
- Lilian Weng's blog posts on VAEs, GANs, and Diffusion Models (excellent summaries)
- *Deep Learning* — Goodfellow et al. — Chapters 14, 20
