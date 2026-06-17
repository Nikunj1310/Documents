# Project 3 — NLP Text Classification (Sentiment Analysis)

**Covers:** Chapters 11–12  
**Estimated time:** 1–2 weeks  
**Difficulty:** Intermediate

---

## Project Goal

Build progressively more powerful text classifiers, from bag-of-words to BERT fine-tuning:

1. **Baseline:** TF-IDF + Logistic Regression
2. **BiLSTM:** Sequential model that reads text token by token
3. **BERT fine-tuning:** State-of-the-art pre-trained Transformer

You will understand the dramatic improvements and why they happen.

---

## Dataset: IMDb Movie Reviews

```python
from datasets import load_dataset

dataset = load_dataset("imdb")
# train: 25,000 labeled reviews (positive/negative)
# test: 25,000 reviews

# Preview
print(dataset['train'][0])
# {'text': "I rented I AM CURIOUS-YELLOW from my video store...", 'label': 0}
# 0 = negative, 1 = positive

# Sample sizes
print(f"Train: {len(dataset['train'])}")
print(f"Test: {len(dataset['test'])}")
```

---

## Approach 1: TF-IDF Baseline

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline
from sklearn.metrics import accuracy_score, classification_report

# Extract text and labels
train_texts = dataset['train']['text']
train_labels = dataset['train']['label']
test_texts = dataset['test']['text']
test_labels = dataset['test']['label']

# Pipeline
baseline = Pipeline([
    ('tfidf', TfidfVectorizer(
        max_features=50000,
        ngram_range=(1, 2),
        min_df=2,
        max_df=0.95,
        sublinear_tf=True,
        strip_accents='unicode'
    )),
    ('clf', LogisticRegression(C=1.0, max_iter=1000, n_jobs=-1))
])

baseline.fit(train_texts, train_labels)
preds = baseline.predict(test_texts)

print(f"TF-IDF + LogReg accuracy: {accuracy_score(test_labels, preds):.4f}")
print(classification_report(test_labels, preds, target_names=['Negative', 'Positive']))
# Expected: ~89-90% accuracy

# What features did the model learn?
tfidf = baseline.named_steps['tfidf']
clf = baseline.named_steps['clf']
feature_names = tfidf.get_feature_names_out()
coef = clf.coef_[0]

top_positive = [(feature_names[i], coef[i]) for i in coef.argsort()[-20:][::-1]]
top_negative = [(feature_names[i], coef[i]) for i in coef.argsort()[:20]]

print("\nTop words predicting POSITIVE:")
for word, coef in top_positive:
    print(f"  {word}: {coef:.3f}")

print("\nTop words predicting NEGATIVE:")
for word, coef in top_negative:
    print(f"  {word}: {coef:.3f}")
```

---

## Approach 2: BiLSTM Classifier

```python
import torch
import torch.nn as nn
from torch.utils.data import Dataset, DataLoader
from collections import Counter

# ── Tokenization and Vocabulary ──────────────────────────────
def tokenize(text):
    return text.lower().split()

def build_vocab(texts, max_vocab=30000, min_freq=2):
    counter = Counter()
    for text in texts:
        counter.update(tokenize(text))
    vocab = {'<pad>': 0, '<unk>': 1}
    for word, freq in counter.most_common(max_vocab):
        if freq >= min_freq:
            vocab[word] = len(vocab)
    return vocab

vocab = build_vocab(train_texts)
print(f"Vocabulary size: {len(vocab)}")

# ── Dataset ───────────────────────────────────────────────────
class TextDataset(Dataset):
    def __init__(self, texts, labels, vocab, max_len=256):
        self.labels = labels
        self.sequences = [
            [vocab.get(t, vocab['<unk>']) for t in tokenize(text)[:max_len]]
            for text in texts
        ]
    
    def __len__(self):
        return len(self.labels)
    
    def __getitem__(self, idx):
        return self.sequences[idx], self.labels[idx]

def collate_fn(batch):
    sequences, labels = zip(*batch)
    lengths = torch.tensor([len(s) for s in sequences])
    padded = nn.utils.rnn.pad_sequence(
        [torch.tensor(s) for s in sequences],
        batch_first=True, padding_value=0
    )
    return padded, lengths, torch.tensor(labels)

train_dataset = TextDataset(train_texts, train_labels, vocab)
test_dataset = TextDataset(test_texts, test_labels, vocab)
train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True, collate_fn=collate_fn)
test_loader = DataLoader(test_dataset, batch_size=256, shuffle=False, collate_fn=collate_fn)

# ── BiLSTM Model ──────────────────────────────────────────────
class BiLSTMClassifier(nn.Module):
    def __init__(self, vocab_size, embed_dim=128, hidden_dim=256, 
                 n_layers=2, num_classes=2, dropout=0.3):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
        self.lstm = nn.LSTM(
            embed_dim, hidden_dim, n_layers,
            batch_first=True, bidirectional=True, dropout=dropout
        )
        self.dropout = nn.Dropout(dropout)
        self.fc = nn.Linear(hidden_dim * 2, num_classes)
    
    def forward(self, x, lengths):
        embedded = self.dropout(self.embedding(x))
        
        packed = nn.utils.rnn.pack_padded_sequence(
            embedded, lengths.cpu(), batch_first=True, enforce_sorted=False
        )
        output, (hidden, _) = self.lstm(packed)
        
        # Max pooling over sequence
        output, _ = nn.utils.rnn.pad_packed_sequence(output, batch_first=True)
        pooled = output.max(dim=1)[0]   # (batch, hidden*2)
        
        return self.fc(self.dropout(pooled))

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
lstm_model = BiLSTMClassifier(len(vocab)).to(device)

optimizer = torch.optim.AdamW(lstm_model.parameters(), lr=1e-3, weight_decay=1e-4)
scheduler = torch.optim.lr_scheduler.OneCycleLR(
    optimizer, max_lr=1e-2, steps_per_epoch=len(train_loader), epochs=10
)
criterion = nn.CrossEntropyLoss()

def train_epoch_lstm(model, loader, optimizer, criterion, scheduler):
    model.train()
    correct = total = 0
    for sequences, lengths, labels in loader:
        sequences, labels = sequences.to(device), labels.to(device)
        
        optimizer.zero_grad()
        logits = model(sequences, lengths)
        loss = criterion(logits, labels)
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        optimizer.step()
        scheduler.step()
        
        correct += (logits.argmax(1) == labels).sum().item()
        total += labels.size(0)
    return correct / total

@torch.no_grad()
def evaluate_lstm(model, loader):
    model.eval()
    correct = total = 0
    for sequences, lengths, labels in loader:
        sequences, labels = sequences.to(device), labels.to(device)
        logits = model(sequences, lengths)
        correct += (logits.argmax(1) == labels).sum().item()
        total += labels.size(0)
    return correct / total

for epoch in range(10):
    train_acc = train_epoch_lstm(lstm_model, train_loader, optimizer, criterion, scheduler)
    test_acc = evaluate_lstm(lstm_model, test_loader)
    print(f"Epoch {epoch+1}: train={train_acc:.4f}, test={test_acc:.4f}")
# Expected: ~91-93% accuracy
```

---

## Approach 3: BERT Fine-tuning

```python
from transformers import (BertTokenizerFast, BertForSequenceClassification,
                           Trainer, TrainingArguments)
from torch.utils.data import Dataset
import evaluate

tokenizer = BertTokenizerFast.from_pretrained('bert-base-uncased')

class IMDbDataset(Dataset):
    def __init__(self, texts, labels, tokenizer, max_length=256):
        self.encodings = tokenizer(
            texts, truncation=True, padding='max_length',
            max_length=max_length, return_tensors='pt'
        )
        self.labels = labels
    
    def __len__(self):
        return len(self.labels)
    
    def __getitem__(self, idx):
        return {
            'input_ids': self.encodings['input_ids'][idx],
            'attention_mask': self.encodings['attention_mask'][idx],
            'labels': torch.tensor(self.labels[idx])
        }

# Use subset for speed (or full dataset if GPU available)
n_train = 5000
bert_train = IMDbDataset(train_texts[:n_train], train_labels[:n_train], tokenizer)
bert_test = IMDbDataset(test_texts[:1000], test_labels[:1000], tokenizer)

# Model
bert_model = BertForSequenceClassification.from_pretrained(
    'bert-base-uncased',
    num_labels=2
)

# Metrics
accuracy_metric = evaluate.load('accuracy')
def compute_metrics(eval_pred):
    logits, labels = eval_pred
    preds = logits.argmax(-1)
    return accuracy_metric.compute(predictions=preds, references=labels)

# Training
training_args = TrainingArguments(
    output_dir='./bert_imdb',
    num_train_epochs=3,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=32,
    learning_rate=2e-5,
    weight_decay=0.01,
    warmup_ratio=0.1,
    evaluation_strategy='epoch',
    save_strategy='epoch',
    load_best_model_at_end=True,
    metric_for_best_model='accuracy',
    fp16=torch.cuda.is_available(),   # Mixed precision for speed
    logging_steps=50,
    report_to='none'
)

trainer = Trainer(
    model=bert_model,
    args=training_args,
    train_dataset=bert_train,
    eval_dataset=bert_test,
    compute_metrics=compute_metrics
)

trainer.train()
results = trainer.evaluate()
print(f"BERT accuracy: {results['eval_accuracy']:.4f}")
# Expected: ~94-95% on full dataset
```

---

## Analysis: Understanding the Models

### Error Analysis

```python
# Find examples where BERT is wrong
bert_model.eval()
wrong_examples = []

with torch.no_grad():
    for i, (text, label) in enumerate(zip(test_texts[:500], test_labels[:500])):
        inputs = tokenizer(text, return_tensors='pt', truncation=True, 
                           max_length=256).to(device)
        logits = bert_model(**inputs).logits
        pred = logits.argmax().item()
        confidence = logits.softmax(-1).max().item()
        
        if pred != label:
            wrong_examples.append({
                'text': text[:200],
                'true': label,
                'pred': pred,
                'confidence': confidence
            })

# Inspect the mistakes — common error patterns?
for ex in sorted(wrong_examples, key=lambda x: -x['confidence'])[:5]:
    print(f"True: {'POS' if ex['true']==1 else 'NEG'}, "
          f"Pred: {'POS' if ex['pred']==1 else 'NEG'}, "
          f"Confidence: {ex['confidence']:.3f}")
    print(f"Text: {ex['text'][:150]}...")
    print()
```

### Compare All Three Approaches

```python
results = {
    'TF-IDF + LogReg': 0.896,   # fill in your numbers
    'BiLSTM': 0.923,
    'BERT': 0.944
}

plt.figure(figsize=(8, 5))
plt.bar(results.keys(), results.values(), color=['#2196F3', '#4CAF50', '#FF9800'])
plt.ylim(0.85, 1.0)
plt.ylabel('Test Accuracy')
plt.title('IMDb Sentiment: Model Comparison')
for i, (name, acc) in enumerate(results.items()):
    plt.text(i, acc + 0.002, f'{acc:.3f}', ha='center')
plt.tight_layout()
plt.show()
```

---

## Extension: Build Your Own Text Classifier

Now apply these techniques to a domain of your choice:
- News category classification (AG News dataset)
- Toxic comment classification (Kaggle competition)
- Email spam/ham classification
- Product review classification (Amazon reviews)

Build all three approaches and compare. Document:
- Which approach is worth the extra complexity?
- How does fine-tuning time scale with data size?
- Are there cases where TF-IDF + LogReg is good enough?

---

## Deliverables

- [ ] TF-IDF baseline with accuracy and top-weight visualization
- [ ] BiLSTM model achieving > 91% on IMDb
- [ ] BERT model achieving > 93% on IMDb (or > 90% on 5000 samples)
- [ ] Error analysis: 10 examples BERT gets wrong, pattern identification
- [ ] Comparison table: accuracy, training time, inference time per 1000 samples
- [ ] Custom dataset application (at least one of the three approaches)

## Questions to Answer

1. BERT uses subword tokenization. What happens to unknown words? How does this compare to vocab-based tokenization?
2. What is the effect of truncating long reviews to 256 tokens? Do the important signals come from the beginning, middle, or end?
3. Why is fine-tuning just the last layer faster but worse than fine-tuning all layers?
4. For a production system with 10ms latency requirement, which model would you choose?
