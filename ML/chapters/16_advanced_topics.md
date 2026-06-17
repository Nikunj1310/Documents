# Chapter 16 — Advanced Topics

> *This chapter surveys the frontier of ML: graph neural networks, large language model agents, multimodal learning, and efficient training. These are active research areas — the goal is a strong conceptual foundation, not mastery, so you can read papers and implement new ideas quickly.*

---

## 16.1 Graph Neural Networks (GNNs)

### When Graphs Arise

Not all data is tabular or sequential. Many real-world problems are naturally graphs:
- **Social networks:** users = nodes, friendships = edges
- **Molecules:** atoms = nodes, bonds = edges
- **Knowledge graphs:** entities = nodes, relations = edges
- **Code:** AST nodes = nodes, data/control flow = edges
- **Recommendation systems:** users + items as a bipartite graph

### The Challenge of Graph Data

CNNs exploit spatial regularity (fixed grid). RNNs exploit sequential structure (ordered chain). Graphs have neither:
- Nodes can have arbitrary numbers of neighbors
- There's no canonical ordering of nodes or edges
- The graph structure itself is part of the input

### The Message Passing Framework

Almost all GNN variants follow the same pattern:

```
For each node v at layer l:
  1. Aggregate messages from neighbors: m_v = AGG({h_u^(l) : u ∈ N(v)})
  2. Update own representation:         h_v^(l+1) = UPDATE(h_v^(l), m_v)
```

Different GNN architectures differ in how they define AGG and UPDATE.

### Graph Convolutional Network (GCN)

Kipf & Welling (2017) — the seminal GCN paper:

```
H^(l+1) = σ(D̃^(-1/2) Ã D̃^(-1/2) H^(l) W^(l))
```

Where:
- `Ã = A + I` — adjacency matrix with self-loops
- `D̃` — degree matrix of Ã
- `H^(l)` — node feature matrix at layer l
- `W^(l)` — learnable weight matrix

Intuitively: each node's representation is the mean of its neighbors' representations (linearly transformed). The `D̃^(-1/2)` normalization prevents high-degree nodes from dominating.

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class GCNLayer(nn.Module):
    def __init__(self, in_features, out_features):
        super().__init__()
        self.linear = nn.Linear(in_features, out_features, bias=False)
    
    def forward(self, x, adj_norm):
        # adj_norm: normalized adjacency matrix (A_hat = D^-1/2 * A_tilde * D^-1/2)
        return F.relu(adj_norm @ self.linear(x))

class GCN(nn.Module):
    def __init__(self, input_dim, hidden_dim, output_dim, dropout=0.5):
        super().__init__()
        self.conv1 = GCNLayer(input_dim, hidden_dim)
        self.conv2 = GCNLayer(hidden_dim, output_dim)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, adj_norm):
        x = self.conv1(x, adj_norm)
        x = self.dropout(x)
        x = adj_norm @ self.linear2(x)   # no activation at final layer
        return F.log_softmax(x, dim=1)

def normalize_adjacency(A):
    """Compute D^-1/2 * (A + I) * D^-1/2"""
    A_tilde = A + torch.eye(A.size(0))
    D = A_tilde.sum(dim=1)
    D_inv_sqrt = torch.diag(D ** -0.5)
    return D_inv_sqrt @ A_tilde @ D_inv_sqrt
```

### Graph Attention Networks (GAT)

Instead of averaging neighbors equally, learn attention weights:

```
α_uv = softmax_u( LeakyReLU(aᵀ [Wh_v ‖ Wh_u]) )
h_v^(l+1) = σ( Σ_{u∈N(v)} α_uv W h_u )
```

Multi-head attention (like in Transformers) is applied for stability.

```python
from torch_geometric.nn import GCNConv, GATConv, SAGEConv
import torch_geometric

# PyTorch Geometric: the standard library for GNNs
class GAT(nn.Module):
    def __init__(self, in_channels, hidden_channels, out_channels, heads=8, dropout=0.6):
        super().__init__()
        self.conv1 = GATConv(in_channels, hidden_channels, heads=heads, dropout=dropout)
        self.conv2 = GATConv(hidden_channels * heads, out_channels, heads=1, concat=False, dropout=dropout)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, edge_index):
        x = self.dropout(x)
        x = F.elu(self.conv1(x, edge_index))
        x = self.dropout(x)
        x = self.conv2(x, edge_index)
        return F.log_softmax(x, dim=1)

# GraphSAGE: scales to large graphs via neighbor sampling
class GraphSAGE(nn.Module):
    def __init__(self, in_channels, hidden_channels, out_channels):
        super().__init__()
        self.conv1 = SAGEConv(in_channels, hidden_channels)
        self.conv2 = SAGEConv(hidden_channels, out_channels)
    
    def forward(self, x, edge_index):
        x = F.relu(self.conv1(x, edge_index))
        x = F.dropout(x, p=0.5, training=self.training)
        return self.conv2(x, edge_index)
```

### Graph Tasks

**Node classification:** Predict label for each node (e.g., paper topic in citation network).

**Link prediction:** Predict whether an edge exists (e.g., friendship prediction, drug-protein interaction).

**Graph classification:** Predict label for entire graph (e.g., molecule is toxic or not).

```python
# Link prediction: score edge (u, v) as dot product of embeddings
def link_predict(model, x, edge_index, edge_label_index):
    z = model.encode(x, edge_index)
    src, dst = edge_label_index
    return (z[src] * z[dst]).sum(dim=-1)
```

### Graph Transformers

The latest direction: apply Transformer self-attention to graphs, using graph structure to define which nodes can attend to each other or as positional encodings.

---

## 16.2 Large Language Model Agents

### From Generation to Agency

A language model generates text. An **LLM agent** uses an LLM as the core reasoning engine but also takes actions in the world: searching the web, writing code, calling APIs, managing files.

### The ReAct Pattern (Reason + Act)

```
Thought: I need to find the current temperature in London.
Action: search("current temperature London")
Observation: The current temperature in London is 15°C.
Thought: Now I can answer the question.
Answer: It is currently 15°C in London.
```

The model alternates between reasoning (Thought) and actions (Action), using observations to update its plan.

```python
from anthropic import Anthropic

client = Anthropic()

def react_agent(question, tools, max_steps=10):
    messages = [{"role": "user", "content": question}]
    
    for step in range(max_steps):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )
        
        # If model wants to use a tool
        if response.stop_reason == "tool_use":
            tool_use = next(b for b in response.content if b.type == "tool_use")
            tool_result = execute_tool(tool_use.name, tool_use.input)
            
            messages.append({"role": "assistant", "content": response.content})
            messages.append({
                "role": "user",
                "content": [{"type": "tool_result", "tool_use_id": tool_use.id, 
                              "content": str(tool_result)}]
            })
        else:
            # Model gave a final answer
            return response.content[0].text
    
    return "Max steps reached"
```

### Tool Use / Function Calling

Define tools the model can call:

```python
tools = [
    {
        "name": "search_web",
        "description": "Search the web for current information",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "The search query"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "execute_python",
        "description": "Execute Python code and return the output",
        "input_schema": {
            "type": "object", 
            "properties": {
                "code": {"type": "string", "description": "Python code to execute"}
            },
            "required": ["code"]
        }
    }
]
```

### Retrieval-Augmented Generation (RAG)

Instead of relying solely on training data, RAG retrieves relevant documents at inference time:

```
Query → Embed → Search vector store → Retrieve top-k docs → Prompt = context + query → LLM → Answer
```

```python
from sentence_transformers import SentenceTransformer
import faiss
import numpy as np

# Build index
embed_model = SentenceTransformer('all-MiniLM-L6-v2')
documents = load_documents()   # your corpus
doc_embeddings = embed_model.encode(documents)

index = faiss.IndexFlatIP(doc_embeddings.shape[1])  # inner product (cosine if normalized)
faiss.normalize_L2(doc_embeddings)
index.add(doc_embeddings)

# Retrieve
def retrieve(query, k=5):
    query_emb = embed_model.encode([query])
    faiss.normalize_L2(query_emb)
    scores, indices = index.search(query_emb, k)
    return [documents[i] for i in indices[0]]

# Augmented generation
def rag_answer(question):
    relevant_docs = retrieve(question)
    context = "\n\n".join(relevant_docs)
    
    prompt = f"""Context:\n{context}\n\nQuestion: {question}\n\nAnswer based on the context:"""
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=512,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.content[0].text
```

### Multi-Agent Systems

Multiple specialized agents collaborating:

```python
# Orchestrator routes tasks to specialist agents
orchestrator_prompt = """You are an orchestrator. Given a task, decide which agent to call:
- researcher: for information gathering
- coder: for writing and running code
- critic: for reviewing and improving outputs
"""

# Agents can call each other, share context, work in parallel
```

### Agentic Considerations

**Planning:** Complex tasks require decomposition. Techniques: chain-of-thought, tree-of-thought, LLM-guided search.

**Memory:** Agents need both short-term (conversation context) and long-term (vector store) memory.

**Evaluation:** Measuring agent performance is hard. Often requires task-specific benchmarks (WebArena, SWE-Bench, GAIA).

**Safety:** Agents with real-world actions can cause harm. Key concerns: authorization, sandboxing, human oversight, action reversibility.

---

## 16.3 Multimodal Learning

### The Idea

Models that understand and generate multiple modalities (text, images, audio, video) simultaneously.

### CLIP (Contrastive Language-Image Pre-Training)

Trains image and text encoders to align their embedding spaces:
- Positive pairs: (image, its caption) should have similar embeddings
- Negative pairs: (image, other captions) should have dissimilar embeddings

**Loss:** InfoNCE (contrastive loss):
```
L = -log(exp(sim(I, T) / τ) / Σⱼ exp(sim(I, Tⱼ) / τ))
```

τ is a temperature parameter. This is cross-entropy over batch of text descriptions.

```python
import torch
import clip

model, preprocess = clip.load("ViT-B/32", device=device)

# Zero-shot image classification
image = preprocess(Image.open("cat.jpg")).unsqueeze(0).to(device)
text = clip.tokenize(["a photo of a cat", "a photo of a dog", "a photo of a car"]).to(device)

with torch.no_grad():
    image_features = model.encode_image(image)
    text_features = model.encode_text(text)
    
    logits = (image_features @ text_features.T).softmax(dim=-1)
    print(logits)   # which description matches best?
```

CLIP enables zero-shot classification — just describe the classes in text.

### Vision-Language Models (VLMs)

Models that can both see and generate text: LLaVA, GPT-4V, Claude (Vision), Gemini.

Architecture: visual encoder (ViT) → projection layer → language model decoder

```python
from anthropic import Anthropic
import base64

client = Anthropic()

def analyze_image(image_path, question):
    with open(image_path, "rb") as f:
        image_data = base64.standard_b64encode(f.read()).decode("utf-8")
    
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {"type": "base64", "media_type": "image/jpeg", "data": image_data}
                },
                {"type": "text", "text": question}
            ]
        }]
    )
    return response.content[0].text
```

### Image Generation with Text Guidance

**Stable Diffusion:** Diffusion model conditioned on CLIP text embeddings.
- Text prompt → CLIP encoder → text embeddings → guide the denoising process
- The denoising U-Net is conditioned on text at each step via cross-attention

```python
from diffusers import StableDiffusionPipeline
import torch

pipe = StableDiffusionPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5",
    torch_dtype=torch.float16
).to("cuda")

image = pipe(
    prompt="A photorealistic cat astronaut floating in space, 4K, detailed",
    negative_prompt="cartoon, low quality, blurry",
    num_inference_steps=50,
    guidance_scale=7.5   # how much to weight the text conditioning
).images[0]

image.save("cat_astronaut.png")
```

**Guidance scale:** Higher = more faithful to prompt but less diverse. 7–12 is typical.

---

## 16.4 Efficient ML

### The Problem

Large models are expensive: training GPT-3 (175B params) cost ~$5M. Inference for a 70B model requires multiple high-end GPUs. Deploying on edge devices is even harder.

### Quantization

Reduce numerical precision of weights and activations:
- **FP32 → FP16:** 2× memory reduction, minimal accuracy loss
- **FP16 → INT8:** 4× memory reduction vs FP32, ~1% accuracy loss
- **INT8 → INT4:** 8× memory reduction vs FP32, larger accuracy loss

```python
# Post-training quantization with PyTorch
model_int8 = torch.quantization.quantize_dynamic(
    model, {nn.Linear}, dtype=torch.qint8
)

# GGUF/llama.cpp: run quantized LLMs on CPU
# ollama run llama3:8b-q4  # 4-bit quantized Llama3 8B, runs on laptop
```

### Pruning

Remove unimportant weights (set to zero):

**Unstructured pruning:** Zero out individual weights. Creates sparse tensors. Hard to speed up on standard hardware.

**Structured pruning:** Remove entire neurons, attention heads, or layers. Creates smaller dense models. Easier to accelerate.

```python
import torch.nn.utils.prune as prune

# Prune 20% of weights in a linear layer (magnitude-based)
prune.l1_unstructured(model.fc, name='weight', amount=0.2)

# Global pruning: prune 30% of all weights across the network
parameters_to_prune = [(layer, 'weight') for layer in model.children() 
                       if isinstance(layer, nn.Linear)]
prune.global_unstructured(parameters_to_prune, pruning_method=prune.L1Unstructured, amount=0.3)
```

### Knowledge Distillation

Train a small "student" model to mimic a large "teacher" model:

```python
class DistillationLoss(nn.Module):
    def __init__(self, temperature=4.0, alpha=0.7):
        super().__init__()
        self.T = temperature
        self.alpha = alpha
        self.kl_loss = nn.KLDivLoss(reduction='batchmean')
        self.ce_loss = nn.CrossEntropyLoss()
    
    def forward(self, student_logits, teacher_logits, true_labels):
        # Soft targets from teacher (high temperature = softer distribution)
        soft_loss = self.kl_loss(
            F.log_softmax(student_logits / self.T, dim=1),
            F.softmax(teacher_logits / self.T, dim=1)
        ) * (self.T ** 2)
        
        # Hard targets from true labels
        hard_loss = self.ce_loss(student_logits, true_labels)
        
        return self.alpha * soft_loss + (1 - self.alpha) * hard_loss
```

The temperature `T > 1` softens the teacher's distribution, revealing "dark knowledge" — which wrong classes the teacher considers plausible.

### Low-Rank Adaptation (LoRA)

For fine-tuning large models efficiently (also covered in Ch 12):

Instead of updating the full weight matrix W ∈ ℝ^(d×k):
```
ΔW = BA    where B ∈ ℝ^(d×r), A ∈ ℝ^(r×k), r << min(d,k)
```

Only A and B are trained. With r=8, this reduces trainable parameters by 100-1000×.

### FlashAttention

The standard attention algorithm is memory-bound — it reads/writes the O(n²) attention matrix to GPU HBM (slow). FlashAttention reorders the computation to stay in the fast SRAM:

- **No O(n²) memory:** Processes in tiles, no full attention matrix materialized
- **2-4× speedup** in practice
- **Exact attention** (not approximate) — identical results

```python
# In PyTorch 2.0+, FlashAttention is used automatically with:
with torch.backends.cuda.sdp_kernel(enable_flash=True):
    output = F.scaled_dot_product_attention(q, k, v, is_causal=True)
```

### Mixed Precision Training

Use FP16 for forward/backward passes, FP32 for optimizer state:

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for batch in dataloader:
    optimizer.zero_grad()
    
    with autocast():   # runs forward pass in FP16
        output = model(batch)
        loss = criterion(output, target)
    
    scaler.scale(loss).backward()   # scale loss to prevent underflow
    scaler.unscale_(optimizer)
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
    scaler.step(optimizer)          # unscales gradients before optimizer step
    scaler.update()
```

Typically 1.5-2× speedup with minimal accuracy loss.

---

## 16.5 Trustworthy ML

### Fairness

ML models can perpetuate or amplify societal biases present in training data.

**Common fairness metrics:**
- **Demographic parity:** Positive prediction rate equal across groups
- **Equalized odds:** Equal TPR and FPR across groups
- **Individual fairness:** Similar individuals treated similarly

```python
# Measure fairness with fairlearn
from fairlearn.metrics import MetricFrame
from sklearn.metrics import accuracy_score

metric_frame = MetricFrame(
    metrics=accuracy_score,
    y_true=y_test,
    y_pred=y_pred,
    sensitive_features=X_test['gender']
)
print(metric_frame.by_group)
print(f"Accuracy gap: {metric_frame.difference():.4f}")
```

There is no single fairness criterion — different definitions conflict with each other (Chouldechova's impossibility theorem). Choosing a fairness criterion is a value judgment.

### Interpretability

Understanding why a model makes a prediction is critical for trust, debugging, and regulation.

**SHAP (SHapley Additive exPlanations):** Game-theoretic attribution of prediction to each feature:

```python
import shap

explainer = shap.TreeExplainer(xgb_model)
shap_values = explainer.shap_values(X_test)

# Summary plot: feature importance and direction
shap.summary_plot(shap_values, X_test, feature_names=feature_names)

# Explain a single prediction
shap.force_plot(explainer.expected_value, shap_values[0], X_test.iloc[0])

# For neural networks:
explainer = shap.DeepExplainer(neural_model, X_background)
shap_values = explainer.shap_values(X_test[:10])
```

**LIME (Local Interpretable Model-agnostic Explanations):** Fit a simple linear model locally around a prediction.

### Calibration

A model is **calibrated** if its confidence scores reflect true probabilities. "When the model says 80% confidence, it should be right 80% of the time."

```python
from sklearn.calibration import calibration_curve, CalibratedClassifierCV
import matplotlib.pyplot as plt

prob_true, prob_pred = calibration_curve(y_test, y_proba, n_bins=10)

plt.plot([0, 1], [0, 1], 'k--', label='Perfect calibration')
plt.plot(prob_pred, prob_true, 's-', label='Model')
plt.xlabel('Mean predicted probability')
plt.ylabel('Fraction of positives')
plt.title('Calibration Curve')
plt.legend()
plt.show()

# Recalibrate via Platt scaling or isotonic regression
calibrated_model = CalibratedClassifierCV(model, method='isotonic', cv=5)
calibrated_model.fit(X_train, y_train)
```

### Robustness

Models can fail dramatically on inputs slightly different from the training distribution.

**Adversarial examples:** Imperceptible perturbations to images that fool classifiers:
```python
# FGSM (Fast Gradient Sign Method) attack
def fgsm_attack(model, x, y, epsilon=0.03):
    x_adv = x.clone().requires_grad_(True)
    output = model(x_adv)
    loss = criterion(output, y)
    loss.backward()
    perturbation = epsilon * x_adv.grad.sign()
    return (x + perturbation).clamp(0, 1)
```

**Distribution shift:** Test performance drops when deployment distribution differs from training. Solutions: domain adaptation, data augmentation, domain randomization.

---

## 16.6 ML Systems at Scale

### Data Pipelines

Production ML requires reliable, scalable data pipelines:

```python
# Apache Beam / Dataflow for large-scale data processing
import apache_beam as beam

def preprocess(element):
    return {
        'features': normalize(element['raw_features']),
        'label': element['target']
    }

with beam.Pipeline() as p:
    (p
     | beam.ReadFromBigQuery(query='SELECT * FROM my_dataset')
     | beam.Map(preprocess)
     | beam.io.tfrecordio.WriteToTFRecord('output/train'))
```

### Distributed Training

For models too large for a single GPU:

**Data parallelism:** Each GPU has a full model copy, processes different batches. Gradients averaged across GPUs.

```python
# PyTorch DDP (DistributedDataParallel)
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP

dist.init_process_group("nccl")
model = DDP(model.to(device), device_ids=[local_rank])

# Each process handles a different subset of the data
sampler = torch.utils.data.DistributedSampler(dataset)
```

**Model parallelism:** Different layers on different GPUs. Needed when a single model layer doesn't fit in memory.

**Tensor parallelism:** Split individual weight matrices across GPUs. Used in Megatron-LM for training 100B+ parameter models.

### Model Serving at Scale

```yaml
# Kubernetes deployment for ML inference
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-inference
spec:
  replicas: 5
  selector:
    matchLabels:
      app: ml-inference
  template:
    spec:
      containers:
      - name: inference
        image: my-registry/ml-model:v2.1
        resources:
          requests:
            nvidia.com/gpu: 1
          limits:
            nvidia.com/gpu: 1
        env:
        - name: MODEL_PATH
          value: /models/fraud_model_v2.1
```

**Load balancing, autoscaling, A/B testing, canary deployments** — production ML requires the same engineering rigor as any large-scale software system.

---

## 16.7 Active Research Directions

These are open, fast-moving areas as of 2025. Worth tracking:

| Area | What's Happening |
|------|-----------------|
| **Scaling Laws** | How do loss, compute, and data relate? When does scaling stop helping? |
| **Long Context** | Models handling 100k+ tokens (Mamba, RWKV, extended attention) |
| **Reasoning** | Models that can do multi-step math and logic reliably (o1-style) |
| **Multimodal** | Unified models for text + image + audio + video + code |
| **LLM Agents** | Reliable planning, tool use, multi-agent coordination |
| **Efficient Fine-tuning** | Better PEFT methods, continual learning without forgetting |
| **AI Safety** | Alignment, interpretability, robustness, corrigibility |
| **Physical AI** | Robots learning from video/simulation + foundation models |
| **Synthetic Data** | Using models to generate training data for models |
| **Test-time Compute** | Spending more compute at inference for better results |

---

## Exercises

1. **GNN on Cora:** Use PyTorch Geometric. Load the Cora citation dataset. Train a GCN for node classification. Compare performance to MLP (ignoring graph structure) and the same GCN but with random graph structure.

2. **Build a RAG system:** Take any document set (your own notes, a textbook chapter). Build an embedding index with FAISS. Implement a retrieval-augmented QA system. What happens when the answer isn't in the documents?

3. **Quantization experiment:** Take a ResNet18 trained on CIFAR-10. Apply dynamic INT8 quantization. Measure: model size, inference latency, accuracy. What is the accuracy-efficiency tradeoff?

4. **Fairness audit:** Train a classifier on the Adult income dataset. Measure accuracy separately for men and women. Is the model fair? Now apply fairness constraints and retrain — what happened to overall accuracy?

5. **SHAP analysis:** On Project 01 (house prices), compute SHAP values for your XGBoost model. Which features have the biggest positive and negative impact on price? Do any findings surprise you?

---

## Further Reading

- *Graph Representation Learning* — Hamilton (free at cs.mcgill.ca/~wlh/grl_book)
- *Trustworthy Machine Learning* — Kearns & Roth
- *Designing Machine Learning Systems* — Chip Huyen (production scale)
- Vaswani et al. (2017) — Attention Is All You Need
- Papers With Code (paperswithcode.com) — track state-of-the-art with reproducible code
- The Batch (deeplearning.ai) — weekly newsletter on ML research
- Lilian Weng's blog (lilianweng.github.io) — deep-dives on most topics in this chapter
