# Machine Learning — Complete Study Strategy

> **Goal:** Develop deep, working knowledge of ML — from mathematical foundations through modern deep learning and production deployment — with the ability to implement, debug, and reason about models from first principles.

---

## Philosophy

Most ML courses teach you to *use* libraries. This curriculum teaches you to *understand* what the libraries are doing. You will derive equations, implement algorithms from scratch, then confirm them against production libraries. By the end, you will not need to memorize formulas — you will understand why they exist.

---

## The Full Roadmap

```
Phase 0  Mathematical Foundations         ~3 weeks
Phase 1  Python & Data Science Tools      ~1 week
Phase 2  Core ML Concepts                 ~2 weeks
Phase 3  Classical Machine Learning       ~4 weeks
Phase 4  Model Evaluation & Engineering   ~2 weeks
Phase 5  Neural Networks from Scratch     ~3 weeks
Phase 6  Convolutional Neural Networks    ~2 weeks
Phase 7  Sequence Models & Attention      ~2 weeks
Phase 8  Transformers & Modern NLP        ~3 weeks
Phase 9  Generative Models                ~2 weeks
Phase 10 Reinforcement Learning           ~3 weeks
Phase 11 ML in Production (MLOps)         ~2 weeks
Phase 12 Implementation Projects          ~6 weeks
─────────────────────────────────────────────────
Total                                    ~35 weeks (~8–9 months)
```

This is a deep track. Compress or skip topics you already know. If you have prior statistics/calculus knowledge, Phase 0 becomes 1 week. If you know Python well, skip Phase 1.

---

## Daily Study Template

| Block | Duration | Activity |
|-------|----------|----------|
| Theory | 45–60 min | Read the chapter, work through derivations by hand |
| Code | 45–60 min | Implement the day's concept from scratch, then in a library |
| Review | 15 min | Write a 5-sentence summary without looking at notes |
| Problem | 30 min | Solve 1–2 exercises or Kaggle challenge problems |

**Total: ~2.5–3 hours/day.** Consistent daily work beats marathon weekends.

---

## Phase-by-Phase Breakdown

### Phase 0 — Mathematical Foundations (3 weeks)
**Why first:** Every ML algorithm is expressed in math. If you skip this, you will hit a wall in Phase 5.

| Week | Topics |
|------|--------|
| 1 | Linear algebra: vectors, matrices, dot products, matrix multiply, transpose, inverse, rank |
| 2 | Calculus: derivatives, partial derivatives, gradients, the chain rule, Taylor expansion |
| 3 | Probability & statistics: distributions, expectation, variance, Bayes theorem, MLE |

**Milestone:** Derive the normal equation for linear regression by hand.

---

### Phase 1 — Python & Data Science Tools (1 week)
**Goal:** Fluency with NumPy, Pandas, Matplotlib, and basic Scikit-learn.

**Milestone:** Load a CSV, clean it, visualize distributions, fit a model, evaluate it — all without following a tutorial.

---

### Phase 2 — Core ML Concepts (2 weeks)
**Goal:** Understand the *learning problem* deeply — what it means to generalize, why models fail, and how optimization works.

**Key ideas:** Loss functions, empirical risk minimization, bias-variance tradeoff, gradient descent, train/val/test split.

**Milestone:** Implement gradient descent from scratch on a toy dataset. Observe learning rate effects.

---

### Phase 3 — Classical Machine Learning (4 weeks)

| Week | Topics |
|------|--------|
| 1 | Linear & logistic regression, regularization |
| 2 | Decision trees, random forests |
| 3 | Gradient boosting, XGBoost, LightGBM |
| 4 | SVMs, k-NN, Naive Bayes, clustering, PCA |

**Milestone:** Win a beginner Kaggle tabular competition using only classical ML.

---

### Phase 4 — Model Evaluation & Feature Engineering (2 weeks)
**Why a separate phase:** Practitioners spend 60–70% of time here. Weak evaluation = wrong conclusions.

**Key skills:** Cross-validation, confusion matrices, ROC/AUC, feature encoding, imputation, scaling, selection.

**Milestone:** Improve a baseline model significantly through feature engineering alone (no algorithm changes).

---

### Phase 5 — Neural Networks from Scratch (3 weeks)
**Goal:** Understand backpropagation well enough to implement it manually.

| Week | Topics |
|------|--------|
| 1 | Perceptrons, activations, loss surfaces |
| 2 | Backpropagation derivation, implement MLP in NumPy |
| 3 | Optimization (Adam, SGD+momentum), regularization (dropout, batch norm), PyTorch basics |

**Milestone:** Implement a 2-layer MLP in pure NumPy that achieves >97% on MNIST.

---

### Phase 6 — Convolutional Neural Networks (2 weeks)
**Topics:** Convolution math, pooling, classic architectures (LeNet → ResNet), transfer learning, data augmentation.

**Milestone:** Fine-tune a ResNet on a custom image dataset using PyTorch.

---

### Phase 7 — Sequence Models & Attention (2 weeks)
**Topics:** RNNs, vanishing gradients, LSTMs, GRUs, sequence-to-sequence, attention mechanism derivation.

**Milestone:** Train an LSTM language model on a small text corpus.

---

### Phase 8 — Transformers & Modern NLP (3 weeks)
**Topics:** Self-attention derivation, multi-head attention, positional encoding, BERT, GPT architecture, fine-tuning with HuggingFace, prompt engineering.

**Milestone:** Fine-tune a BERT model for text classification on a real dataset.

---

### Phase 9 — Generative Models (2 weeks)
**Topics:** VAEs, GANs (theory + pitfalls), diffusion models (DDPM conceptually).

**Milestone:** Train a simple GAN on a toy dataset. Observe mode collapse.

---

### Phase 10 — Reinforcement Learning (3 weeks)

| Week | Topics |
|------|--------|
| 1 | MDPs, Bellman equations, dynamic programming |
| 2 | Q-learning, DQN |
| 3 | Policy gradients, REINFORCE, Actor-Critic |

**Milestone:** Train a DQN agent to play CartPole.

---

### Phase 11 — ML in Production (2 weeks)
**Topics:** Experiment tracking (MLflow), model packaging, REST API (FastAPI), Docker, monitoring, data/model drift.

**Milestone:** Deploy a trained model as a REST API inside a Docker container.

---

### Phase 12 — Implementation Projects (6 weeks)
See the `/projects/` directory. Five end-to-end projects that touch every part of the stack.

---

## Resources

### Books (in order of reading)
1. **Mathematics for Machine Learning** — Deisenroth, Faisal, Ong (free PDF online) — Phase 0
2. **Hands-On Machine Learning** — Aurélien Géron — Phase 2–4
3. **The Elements of Statistical Learning** — Hastie et al. (free PDF) — Phase 3 depth
4. **Deep Learning** — Goodfellow, Bengio, Courville (free online) — Phase 5–9
5. **Reinforcement Learning: An Introduction** — Sutton & Barto (free PDF) — Phase 10

### Courses (supplementary, not primary)
- Fast.ai Practical Deep Learning — great intuition building (Phase 6+)
- CS231n Stanford (CNNs) — Phase 6
- CS224n Stanford (NLP) — Phase 8
- David Silver RL Course — Phase 10

### Practice
- **Kaggle** — competitions + notebooks for all phases
- **Papers With Code** — track state-of-the-art, reproduce papers
- **LeetCode ML section** — interview prep (do alongside projects)

---

## How to Use the Chapter Notes

Each chapter in `/chapters/` follows this structure:
1. **Motivation** — why this exists and what problem it solves
2. **Core Theory** — derivations, math, key equations
3. **Intuition** — geometric/visual explanations
4. **Implementation** — code from scratch, then library version
5. **Common Mistakes** — what trips people up
6. **Exercises** — problems to solve before moving on
7. **Further Reading** — papers and resources to go deeper

Do **not** skip the exercises. They are where understanding becomes knowledge.

---

## Progress Tracking

Copy this checklist and check off chapters as you complete them:

```
[ ] Ch 01 — Mathematical Foundations
[ ] Ch 02 — Python & Data Science Tools
[ ] Ch 03 — Core ML Concepts
[ ] Ch 04 — Linear Models
[ ] Ch 05 — Tree-Based Methods
[ ] Ch 06 — Classical ML (SVM, KNN, Clustering, PCA)
[ ] Ch 07 — Model Evaluation & Metrics
[ ] Ch 08 — Feature Engineering
[ ] Ch 09 — Neural Networks from Scratch
[ ] Ch 10 — Convolutional Neural Networks
[ ] Ch 11 — Sequence Models & RNNs
[ ] Ch 12 — Transformers & Modern NLP
[ ] Ch 13 — Generative Models
[ ] Ch 14 — Reinforcement Learning
[ ] Ch 15 — ML in Production
[ ] Project 01 — House Price Prediction
[ ] Project 02 — Image Classification
[ ] Project 03 — NLP Text Classification
[ ] Project 04 — Recommendation System
[ ] Project 05 — RL Agent
```

---

## Mindset Notes

- **Confusion is normal.** If something doesn't click after 30 minutes, sleep on it. The brain consolidates during rest.
- **Implement before you feel ready.** The act of writing code reveals gaps that reading hides.
- **Don't tutorial-hop.** One resource per topic, go deep, move on.
- **Teach it.** If you can't explain a concept to a rubber duck, you don't understand it yet.
- **Build in public.** Push your implementations to GitHub. It creates accountability and a portfolio.
