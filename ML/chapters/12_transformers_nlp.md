# Chapter 12 — Transformers & Modern NLP

> *The Transformer architecture (2017) changed everything. It abolished recurrence, relied entirely on attention, and enabled training at unprecedented scale. All modern LLMs — GPT, BERT, LLaMA — are Transformers. This chapter derives it from first principles.*

---

## 12.1 Motivation: Limitations of RNNs

RNNs have two fundamental problems:
1. **Sequential computation:** You must process `x₁, x₂, ..., xₙ` in order. Can't parallelize across time steps. Slow to train.
2. **Long-range dependencies:** Even LSTMs struggle with dependencies spanning hundreds of tokens.

**The Transformer solution:** Remove recurrence entirely. Replace it with **self-attention** — a mechanism that allows every position to attend to every other position in a single operation. The entire sequence is processed in parallel.

---

## 12.2 Self-Attention

Self-attention is attention where the query, key, and value all come from the same sequence. Each position attends to all other positions.

### Scaled Dot-Product Attention

Given input matrix `X ∈ ℝ^(n×d)` (n positions, d dimensions each):

1. **Project** X to queries, keys, values:
```
Q = XWQ    (n × dₖ)
K = XWK    (n × dₖ)
V = XWV    (n × dᵥ)
```

2. **Compute attention scores** (how much should each position attend to each other?):
```
A = softmax(QKᵀ / √dₖ)    (n × n matrix)
```

3. **Weighted sum of values:**
```
Output = A · V    (n × dᵥ)
```

**Full formula:**
```
Attention(Q, K, V) = softmax(QKᵀ / √dₖ) · V
```

**The scaling factor `√dₖ`:** Without it, dot products grow large for high-dimensional vectors, pushing softmax into regions with tiny gradients.

### Intuition

Think of it as a soft database query:
- **Query (Q):** "What am I looking for?"
- **Key (K):** "What information do I have to offer?"
- **Value (V):** "What information will I give if selected?"

The attention score `QᵢKⱼᵀ/√dₖ` measures how much position i should attend to position j. Softmax normalizes scores to probabilities. The output is a weighted mixture of values.

For a language example: the word "it" attends strongly to the noun it refers to. The model learns to make these connections automatically.

### Computational Complexity

Self-attention is `O(n²d)` in time and `O(n²)` in memory (the n×n attention matrix). For very long sequences, this is expensive. Various efficient attention variants (Longformer, FlashAttention) address this.

---

## 12.3 Multi-Head Attention

Instead of one attention operation, use h parallel attention operations (heads), each with smaller dimension `dₖ = d/h`:

```
head_i = Attention(Q·W_i^Q, K·W_i^K, V·W_i^V)
MultiHead(Q,K,V) = Concat(head_1, ..., head_h) · W^O
```

**Why multiple heads?** Different heads learn to attend to different types of relationships simultaneously:
- One head might learn syntax (subject-verb agreement)
- Another might learn coreference (pronoun to noun)
- Another might learn semantic similarity

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, n_heads):
        super().__init__()
        assert d_model % n_heads == 0
        
        self.d_model = d_model
        self.n_heads = n_heads
        self.d_k = d_model // n_heads
        
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
    
    def attention(self, Q, K, V, mask=None):
        scores = Q @ K.transpose(-2, -1) / math.sqrt(self.d_k)  # (B, h, n, n)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        weights = F.softmax(scores, dim=-1)
        return weights @ V, weights
    
    def forward(self, Q, K, V, mask=None):
        B, n, _ = Q.shape
        
        # Project and split into heads
        Q = self.W_q(Q).view(B, n, self.n_heads, self.d_k).transpose(1, 2)
        K = self.W_k(K).view(B, -1, self.n_heads, self.d_k).transpose(1, 2)
        V = self.W_v(V).view(B, -1, self.n_heads, self.d_k).transpose(1, 2)
        # Now: (B, n_heads, seq_len, d_k)
        
        x, self.attn_weights = self.attention(Q, K, V, mask)
        
        # Concatenate heads
        x = x.transpose(1, 2).contiguous().view(B, n, self.d_model)
        return self.W_o(x)
```

---

## 12.4 Positional Encoding

Self-attention has no notion of order (it treats the sequence as a set). We need to inject positional information.

**Sinusoidal positional encoding** (original Transformer):
```
PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
```

Properties:
- Unique encoding for each position
- Smooth interpolation between nearby positions
- Works for sequences longer than seen in training

**Learned positional embeddings** (BERT, GPT): Learn a lookup table `pos_emb[pos]`. Simpler, often as good.

**Relative positional encoding** (modern models): Encode the *distance* between positions rather than absolute positions. More generalizable.

```python
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000, dropout=0.1):
        super().__init__()
        self.dropout = nn.Dropout(dropout)
        
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        
        self.register_buffer('pe', pe.unsqueeze(0))  # (1, max_len, d_model)
    
    def forward(self, x):
        return self.dropout(x + self.pe[:, :x.size(1)])
```

---

## 12.5 The Feed-Forward Sublayer

Each Transformer layer also has a position-wise feed-forward network:
```
FFN(x) = ReLU(xW₁ + b₁)W₂ + b₂
```

Applied identically at each position independently. Typically `d_ff = 4 × d_model`.

---

## 12.6 The Full Transformer Layer

Each layer applies:
1. Multi-head self-attention with residual connection
2. Layer normalization
3. Feed-forward network with residual connection
4. Layer normalization

```
x = x + MultiHeadAttention(LayerNorm(x))   # Pre-LN (modern standard)
x = x + FFN(LayerNorm(x))
```

**Pre-LN vs Post-LN:** Pre-LayerNorm (normalize before the sublayer) trains more stably than the original Post-LN formulation.

```python
class TransformerBlock(nn.Module):
    def __init__(self, d_model, n_heads, d_ff, dropout=0.1):
        super().__init__()
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.attention = MultiHeadAttention(d_model, n_heads)
        self.ff = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.GELU(),
            nn.Dropout(dropout),
            nn.Linear(d_ff, d_model),
            nn.Dropout(dropout)
        )
    
    def forward(self, x, mask=None):
        x = x + self.attention(self.norm1(x), self.norm1(x), self.norm1(x), mask)
        x = x + self.ff(self.norm2(x))
        return x
```

---

## 12.7 The Original Transformer (Encoder-Decoder)

For sequence-to-sequence (translation):

**Encoder:** Stack of L Transformer blocks with bidirectional self-attention (each position can attend to all others).

**Decoder:** Stack of L blocks with:
- Masked self-attention (can only attend to past positions — for autoregressive generation)
- Cross-attention over encoder outputs (each decoder position attends to all encoder positions)

**Causal masking:** Prevents attending to future tokens:
```python
def causal_mask(seq_len):
    mask = torch.tril(torch.ones(seq_len, seq_len))  # lower triangular
    return mask.bool()
```

---

## 12.8 BERT (Bidirectional Encoder Representations from Transformers)

BERT pre-trains a Transformer encoder on two tasks:

**1. Masked Language Model (MLM):** Randomly mask 15% of tokens. Predict the masked tokens.
```
Input:  "The cat [MASK] on the [MASK]"
Target: "The cat sat on the mat"
```

**2. Next Sentence Prediction (NSP):** Given two sentences, predict if B follows A in the original text. (Later found to be less useful; dropped in RoBERTa.)

**Key point:** Bidirectional — each token can attend to all other tokens (past AND future). This is only possible in pre-training; at inference, you encode the full input.

### Fine-tuning BERT

```python
from transformers import BertForSequenceClassification, BertTokenizer
import torch

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertForSequenceClassification.from_pretrained('bert-base-uncased', num_labels=2)

# Tokenize
text = ["This movie was great!", "Terrible film, wasted time."]
inputs = tokenizer(
    text,
    padding=True,
    truncation=True,
    max_length=128,
    return_tensors='pt'
)

# Forward pass
outputs = model(**inputs)
logits = outputs.logits

# Fine-tuning
optimizer = torch.optim.AdamW(model.parameters(), lr=2e-5, weight_decay=0.01)
# Train on your classification dataset
```

**HuggingFace Trainer API:**
```python
from transformers import Trainer, TrainingArguments

training_args = TrainingArguments(
    output_dir='./results',
    num_train_epochs=3,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=64,
    warmup_steps=500,
    weight_decay=0.01,
    learning_rate=2e-5,
    evaluation_strategy='epoch',
    save_strategy='epoch',
    load_best_model_at_end=True,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=val_dataset,
    compute_metrics=compute_metrics,
)
trainer.train()
```

---

## 12.9 GPT Architecture (Decoder-Only)

While BERT is an encoder (bidirectional), GPT is a **decoder-only** model (unidirectional/causal).

- **Pre-training:** Autoregressive language modeling — predict next token given all previous tokens
- **Inference:** Generate token by token

```
Input:  "The cat"
GPT:    "sat"  →  "on"  →  "the"  →  "mat"  (generated one token at a time)
```

GPT models scale remarkably with size. GPT-3 (175B params) showed emergent capabilities (few-shot learning) not seen in smaller models. GPT-4 and beyond are multimodal.

### Text Generation with GPT

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer

tokenizer = GPT2Tokenizer.from_pretrained('gpt2')
model = GPT2LMHeadModel.from_pretrained('gpt2')

input_text = "Once upon a time"
input_ids = tokenizer.encode(input_text, return_tensors='pt')

# Greedy decoding
output = model.generate(input_ids, max_length=100)

# Sampling (more diverse)
output = model.generate(
    input_ids,
    max_length=200,
    do_sample=True,
    temperature=0.8,
    top_p=0.92,           # nucleus sampling: sample from top 92% probability mass
    top_k=50,             # sample from top 50 tokens
    repetition_penalty=1.2,
    no_repeat_ngram_size=3
)

print(tokenizer.decode(output[0], skip_special_tokens=True))
```

---

## 12.10 Fine-tuning vs Prompting

### Full Fine-tuning

Update all model weights on task-specific data. Best results, but:
- Requires labeled data
- Expensive for large models
- Risk of catastrophic forgetting

### Parameter-Efficient Fine-tuning (PEFT)

Update only a small fraction of parameters:

**LoRA (Low-Rank Adaptation):** Insert trainable low-rank matrices into attention layers. Only ~0.1–1% of parameters updated.

```python
from peft import LoraConfig, get_peft_model

config = LoraConfig(
    r=8,              # rank of the update matrices
    lora_alpha=32,
    target_modules=['q_proj', 'v_proj'],
    lora_dropout=0.1,
    bias='none',
    task_type='SEQ_CLS'
)

peft_model = get_peft_model(model, config)
peft_model.print_trainable_parameters()
# trainable params: 294,912 || all params: 184,997,914 || trainable%: 0.16
```

### Prompt Engineering

Guide model behavior via the input prompt. No training required. Works best for large models.

**Zero-shot:** Just describe the task.
**Few-shot:** Provide examples in the prompt.
**Chain-of-thought:** Ask the model to reason step by step.

---

## 12.11 Tokenization

Models don't work with raw text — text is tokenized into subword units.

**BPE (Byte-Pair Encoding):** Start with characters. Repeatedly merge the most frequent pair of adjacent tokens. Used by GPT.

**WordPiece:** Similar to BPE but merges based on likelihood. Used by BERT.

**SentencePiece:** Language-agnostic tokenization, works on bytes. Used by LLaMA, T5.

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained('bert-base-uncased')
tokens = tokenizer.tokenize("Transformer architecture")
# ['transformer', 'architecture']

ids = tokenizer.encode("Hello world!")
# [101, 7592, 2088, 999, 102]   (with [CLS] and [SEP] tokens)
```

---

## Exercises

1. **Self-attention from scratch:** Implement scaled dot-product attention in NumPy. Verify that for a random 4-token sequence, the output has the same shape as the input and the attention weights sum to 1 across the sequence dimension.

2. **Positional encoding visualization:** Plot the positional encodings for positions 0–99 and dimensions 0–63 as a heatmap. What patterns do you see? Why does the encoding at position t and t+k look similar for large k?

3. **BERT fine-tuning:** Fine-tune `bert-base-uncased` on the IMDb sentiment dataset (available via HuggingFace datasets). Report test accuracy. Compare to a TF-IDF + logistic regression baseline.

4. **Attention head visualization:** Take a fine-tuned BERT model. For a sample sentence, visualize the attention weights of each head in the last layer. Do different heads attend to different things?

5. **Build a mini-GPT:** Implement a small decoder-only Transformer from scratch in PyTorch. Train it on a small text corpus (Shakespeare). Generate text. Count parameters. How does perplexity change as you increase depth?

---

## Further Reading

- Vaswani et al. (2017) "Attention Is All You Need" — original Transformer paper
- Devlin et al. (2018) "BERT: Pre-training of Deep Bidirectional Transformers" — BERT paper
- Radford et al. (2018, 2019) GPT and GPT-2 papers
- Karpathy "nanoGPT" — minimal GPT implementation on GitHub (essential study)
- Illustrated Transformer — Jay Alammar's blog (`jalammar.github.io`) — superb visual explanation
