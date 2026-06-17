# Chapter 11 — Sequence Models & RNNs

> *Sequential data — text, speech, time series, video — has a structure that MLPs and CNNs don't exploit: the order of elements matters, and context from earlier positions is relevant to later ones. This chapter builds the machinery that culminates in Transformers.*

---

## 11.1 The Sequence Modeling Problem

Given a sequence `x₁, x₂, ..., xₜ`, we might want to:
- **Sequence classification:** Predict one label for the whole sequence (sentiment)
- **Sequence labeling:** Predict one label per element (named entity recognition)
- **Sequence-to-sequence:** Map one sequence to another (translation)
- **Language modeling:** Predict `P(xₜ | x₁, ..., xₜ₋₁)` (next token)

**Key challenge:** Sequences have variable length, and the relevant context can span many steps.

---

## 11.2 Vanilla RNN

A Recurrent Neural Network processes sequences by maintaining a **hidden state** that summarizes the history seen so far.

### The Update Equations

```
hₜ = tanh(Wₕhₜ₋₁ + Wₓxₜ + b)     # hidden state update
yₜ = Wyʰhₜ + bᵧ                     # output at time t
```

- `hₜ ∈ ℝᵈ` — hidden state (memory) at time t
- `xₜ ∈ ℝⁿ` — input at time t
- `Wₕ, Wₓ, Wᵧ` — weight matrices **shared across all time steps** (parameter efficiency)

The critical insight: **Wₕ is the same matrix at every time step.** The RNN is a function applied repeatedly with shared parameters.

### Unrolling

We can visualize an RNN as a chain of identical operations:
```
x₁ → [h₁] → y₁
x₂ → [h₂] → y₂   (h₁ flows into h₂)
x₃ → [h₃] → y₃
...
```

Backpropagation through this chain is called **Backpropagation Through Time (BPTT)**.

### The Vanishing/Exploding Gradient Problem

Gradients flow through the chain: `∂h_T/∂h₁ = Πₜ (∂hₜ/∂hₜ₋₁)`.

Each factor is `Wₕ · diag(tanh'(zₜ))`. Repeated matrix multiplication:
- `‖Wₕ‖ < 1` → gradients shrink exponentially → model can't learn long-range dependencies
- `‖Wₕ‖ > 1` → gradients grow exponentially → loss diverges

For sequences of length 100+, vanilla RNNs effectively cannot propagate gradients from the output to the beginning of the sequence.

**Solutions:**
- LSTMs (architectural gating)
- GRUs (simpler gating)
- Gradient clipping (for exploding)
- Truncated BPTT

---

## 11.3 Long Short-Term Memory (LSTM)

The LSTM explicitly separates short-term memory (hidden state h) from long-term memory (cell state c). The cell state c flows through time with only linear operations — gradients can flow unimpeded.

### The Four Gates

```
fₜ = σ(Wf[hₜ₋₁, xₜ] + bf)    # forget gate: what to discard from cell state
iₜ = σ(Wi[hₜ₋₁, xₜ] + bi)    # input gate: what new info to add
c̃ₜ = tanh(Wc[hₜ₋₁, xₜ] + bc) # candidate cell state
oₜ = σ(Wo[hₜ₋₁, xₜ] + bo)    # output gate: what to output
```

### The State Updates

```
cₜ = fₜ ⊙ cₜ₋₁ + iₜ ⊙ c̃ₜ    # cell state: forget old + add new
hₜ = oₜ ⊙ tanh(cₜ)           # hidden state: filtered cell state
```

Where `⊙` is elementwise multiplication (Hadamard product).

**Gate intuitions:**
- **Forget gate:** "Should we keep this information or forget it?"
- **Input gate:** "Is this new information worth remembering?"
- **Output gate:** "What should we expose from the cell state?"

The cell state `c` is like a "conveyor belt" — the forget and input gates selectively write and erase information. Crucially, the gradient flows through `cₜ` via elementwise operations (no vanishing!).

### LSTM Gradient Flow

The gradient of the loss w.r.t. `cₜ₋₁` passes through the forget gate `fₜ`:
```
∂cₜ/∂cₜ₋₁ = fₜ    (elementwise, not matrix multiply)
```

If `fₜ ≈ 1` (remember everything), gradients flow unchanged. The LSTM can *choose* to preserve gradients by keeping forget gates near 1.

---

## 11.4 Gated Recurrent Unit (GRU)

GRU simplifies LSTM to two gates and one state vector:

```
zₜ = σ(Wz[hₜ₋₁, xₜ])          # update gate (combines input + forget)
rₜ = σ(Wr[hₜ₋₁, xₜ])          # reset gate
h̃ₜ = tanh(W[rₜ ⊙ hₜ₋₁, xₜ])  # candidate hidden state
hₜ = (1 - zₜ) ⊙ hₜ₋₁ + zₜ ⊙ h̃ₜ   # interpolate old and new
```

GRU has fewer parameters than LSTM. Performance is similar — preference depends on the task.

---

## 11.5 RNNs in PyTorch

```python
import torch
import torch.nn as nn

# Built-in RNN/LSTM/GRU
lstm = nn.LSTM(
    input_size=64,     # dimension of each input vector xₜ
    hidden_size=128,   # dimension of hidden state hₜ
    num_layers=2,      # stacked LSTMs
    batch_first=True,  # (batch, seq_len, features) vs (seq_len, batch, features)
    dropout=0.2,       # between layers (not last layer)
    bidirectional=False
)

# Input: (batch, seq_len, input_size)
x = torch.randn(32, 20, 64)   # batch=32, seq_len=20, features=64

# Initial state (default: zeros)
h0 = torch.zeros(2, 32, 128)  # (num_layers, batch, hidden_size)
c0 = torch.zeros(2, 32, 128)

# Forward pass
output, (hn, cn) = lstm(x, (h0, c0))
# output: (32, 20, 128)  — hidden states at every time step
# hn: (2, 32, 128)       — final hidden state per layer
# cn: (2, 32, 128)       — final cell state per layer
```

### Packing Variable-Length Sequences

Real sequences have different lengths. PyTorch's `pack_padded_sequence` handles this efficiently:

```python
from torch.nn.utils.rnn import pack_padded_sequence, pad_packed_sequence

# Pad sequences to the same length
# x: (batch, max_seq_len, features)  — padded
# lengths: list of actual lengths

packed_input = pack_padded_sequence(x, lengths, batch_first=True, enforce_sorted=False)
packed_output, (hn, cn) = lstm(packed_input)
output, output_lengths = pad_packed_sequence(packed_output, batch_first=True)
```

### Sequence Classification Example

```python
class SentimentLSTM(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes, n_layers=2, dropout=0.3):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, n_layers,
                            batch_first=True, dropout=dropout, bidirectional=True)
        self.dropout = nn.Dropout(dropout)
        self.fc = nn.Linear(hidden_dim * 2, num_classes)  # *2 for bidirectional
    
    def forward(self, x, lengths):
        # x: (batch, seq_len) token ids
        embedded = self.dropout(self.embedding(x))   # (batch, seq_len, embed_dim)
        
        packed = pack_padded_sequence(embedded, lengths, batch_first=True, enforce_sorted=False)
        _, (hn, _) = self.lstm(packed)
        
        # Concatenate final states from both directions
        # hn: (n_layers*2, batch, hidden_dim)
        hn = torch.cat([hn[-2], hn[-1]], dim=1)  # (batch, hidden_dim*2)
        
        return self.fc(self.dropout(hn))
```

---

## 11.6 Bidirectional RNNs

A standard RNN only reads the sequence left to right. A **BiRNN** runs two RNNs — one forward, one backward — and concatenates the hidden states:

```
→ h₁ → h₂ → h₃ → h₄    (forward)
← h₁ ← h₂ ← h₃ ← h₄    (backward)
output at t: [hₜ_forward, hₜ_backward]
```

Useful when the full context (past AND future) is available. Cannot be used for autoregressive generation (can't read future tokens for prediction).

```python
nn.LSTM(input_size, hidden_size, bidirectional=True)
# Output is (batch, seq_len, hidden_size * 2)
```

---

## 11.7 Sequence-to-Sequence (Seq2Seq)

Maps one sequence to another of potentially different length (e.g., translation).

### Encoder-Decoder Architecture

```
Encoder: reads the source sequence → produces context vector c
Decoder: generates the target sequence conditioned on c
```

```python
class Seq2Seq(nn.Module):
    def __init__(self, encoder, decoder, device):
        super().__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.device = device
    
    def forward(self, src, trg, teacher_forcing_ratio=0.5):
        batch_size = trg.shape[0]
        trg_len = trg.shape[1]
        trg_vocab_size = self.decoder.output_dim
        
        outputs = torch.zeros(batch_size, trg_len, trg_vocab_size).to(self.device)
        
        # Encode source
        enc_output, hidden = self.encoder(src)
        
        # Start decoding with <SOS> token
        decoder_input = trg[:, 0]
        
        for t in range(1, trg_len):
            output, hidden = self.decoder(decoder_input, hidden, enc_output)
            outputs[:, t] = output
            
            # Teacher forcing: use real target or model's prediction?
            use_teacher_forcing = torch.rand(1) < teacher_forcing_ratio
            decoder_input = trg[:, t] if use_teacher_forcing else output.argmax(1)
        
        return outputs
```

### The Information Bottleneck Problem

The encoder compresses the entire source sequence into a fixed-size vector. For long sequences, this bottleneck causes information loss. The solution: **attention**.

---

## 11.8 Attention Mechanism

**Core idea:** Instead of compressing everything into one vector, let the decoder attend to different parts of the encoder's output at each decoding step.

### Bahdanau Attention (Additive Attention)

At each decoding time step `t`, compute an attention score for each encoder position `s`:

```
eₜₛ = vᵀ tanh(Wₐhₜ₋₁ + Uₐhₛ)    # alignment score (energy)
αₜₛ = softmax(eₜₛ)                  # attention weights (sum to 1 over s)
cₜ = Σₛ αₜₛ hₛ                     # context vector (weighted sum of encoder states)
```

The decoder then uses `cₜ` (context vector) + `hₜ₋₁` (previous decoder state) to generate the next token.

**What's learned:** The model learns when decoding position t, which source positions s to attend to. For translation, this often aligns with linguistic structure.

```python
class BahdanauAttention(nn.Module):
    def __init__(self, enc_dim, dec_dim, attn_dim):
        super().__init__()
        self.W = nn.Linear(dec_dim, attn_dim)
        self.U = nn.Linear(enc_dim, attn_dim)
        self.v = nn.Linear(attn_dim, 1)
    
    def forward(self, decoder_hidden, encoder_outputs):
        # decoder_hidden: (batch, dec_dim)
        # encoder_outputs: (batch, src_len, enc_dim)
        
        H = self.W(decoder_hidden).unsqueeze(1)  # (batch, 1, attn_dim)
        E = self.U(encoder_outputs)              # (batch, src_len, attn_dim)
        
        energy = self.v(torch.tanh(H + E)).squeeze(2)   # (batch, src_len)
        attention = F.softmax(energy, dim=1)              # (batch, src_len)
        
        context = torch.bmm(attention.unsqueeze(1), encoder_outputs).squeeze(1)
        # (batch, enc_dim)
        return context, attention
```

### Scaled Dot-Product Attention

The attention used in Transformers (Chapter 12). Faster than additive attention:

```
Attention(Q, K, V) = softmax(QKᵀ / √dₖ) V
```

- Q (queries), K (keys), V (values): linear projections of the input
- `√dₖ`: scaling factor to prevent too-small gradients in softmax

This is the foundation of self-attention and the Transformer.

---

## 11.9 Language Modeling

A **language model** estimates `P(x₁, x₂, ..., xₙ)` = probability of a sequence.

By the chain rule:
```
P(x₁, ..., xₙ) = Πₜ P(xₜ | x₁, ..., xₜ₋₁)
```

Train a model to predict the next token given all previous tokens. Loss = cross-entropy over the vocabulary.

```python
class RNNLanguageModel(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_layers=2, dropout=0.3):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, num_layers, 
                            batch_first=True, dropout=dropout)
        self.dropout = nn.Dropout(dropout)
        self.fc = nn.Linear(hidden_dim, vocab_size)
    
    def forward(self, x, hidden=None):
        # x: (batch, seq_len)
        embedded = self.dropout(self.embedding(x))
        output, hidden = self.lstm(embedded, hidden)
        logits = self.fc(self.dropout(output))   # (batch, seq_len, vocab_size)
        return logits, hidden
    
    @torch.no_grad()
    def generate(self, start_token, max_length=100, temperature=1.0):
        self.eval()
        tokens = [start_token]
        hidden = None
        
        for _ in range(max_length):
            x = torch.tensor([[tokens[-1]]])
            logits, hidden = self(x, hidden)
            probs = F.softmax(logits[0, 0] / temperature, dim=-1)
            next_token = torch.multinomial(probs, 1).item()
            if next_token == EOS_TOKEN:
                break
            tokens.append(next_token)
        
        return tokens
```

**Perplexity:** Standard metric for language models:
```
Perplexity = exp(average cross-entropy loss)
```
Lower perplexity = better language model. A model with perplexity P is as uncertain as a uniform distribution over P options at each step.

---

## Exercises

1. **RNN from scratch:** Implement a vanilla RNN cell in NumPy. Apply it to classify sequences of 0s and 1s (e.g., "does sequence contain at least three 1s?"). Manually implement BPTT for a sequence of length 5.

2. **Vanishing gradient experiment:** Implement a vanilla RNN and an LSTM in PyTorch. Train both to remember information from the first element of a long sequence (e.g., sequence starts with 1 or 0, predict at end). At what sequence length does the RNN fail? At what length does the LSTM fail?

3. **LSTM language model:** Train an LSTM language model on a small corpus (Shakespeare, or any text file). Sample from the model with temperature 0.5, 1.0, and 2.0. What changes? What does temperature control?

4. **Attention visualization:** Train a seq2seq model with Bahdanau attention on a simple task (e.g., reversing a sequence, or simple translation). Plot the attention weights as a heatmap. Do the attention patterns make sense?

5. **Sentiment analysis:** Using the IMDB dataset, train a BiLSTM sentiment classifier. Compare to: (a) bag-of-words + logistic regression, (b) TF-IDF + gradient boosting. Which performs best?

---

## Further Reading

- *Deep Learning* — Goodfellow et al. — Chapter 10 (Sequence Modeling)
- Olah, Christopher "Understanding LSTM Networks" — `colah.github.io` (essential reading)
- Bahdanau et al. (2015) "Neural Machine Translation by Jointly Learning to Align and Translate" — original attention paper
- Karpathy (2015) "The Unreasonable Effectiveness of Recurrent Neural Networks" — motivating examples
