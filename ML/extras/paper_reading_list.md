# Essential Paper Reading List

> Seminal papers organized in the order to read them. For each, the goal is not to memorize — it is to understand the idea deeply enough to explain it and implement it.

---

## How to Read a Paper

Use the three-pass method:
1. **First pass (10 min):** Title, abstract, introduction, conclusion, figures. Answer: what problem, what approach, what result?
2. **Second pass (1 hr):** Read fully but skip proofs. Note every equation and concept you don't understand.
3. **Third pass (4 hr):** Virtually re-implement. For every claim, ask: how would I prove this? Could I implement this?

---

## Phase 1: Classical ML Foundations

**Read after Chapter 4–6**

| Paper | Why Read It |
|-------|-------------|
| Tibshirani (1996) "Regression Shrinkage and Selection via the Lasso" | The original Lasso paper. Elegant — introduces L1 regularization and sparsity |
| Breiman (2001) "Random Forests" | Original RF paper. Readable, clear intuitions about bagging and feature subsampling |
| Freund & Schapire (1997) "A Decision-Theoretic Generalization of On-Line Learning and an Application to Boosting" | Original AdaBoost. Surprising theoretical guarantees |
| Cortes & Vapnik (1995) "Support-Vector Networks" | Original SVM paper. Math is beautiful, motivation for kernels is clear |

---

## Phase 2: Deep Learning Revolution

**Read after Chapter 9–10**

| Paper | Why Read It |
|-------|-------------|
| LeCun et al. (1998) "Gradient-Based Learning Applied to Document Recognition" | LeNet — CNNs work, end-to-end learning works. Historical foundation |
| Krizhevsky, Sutskever & Hinton (2012) "ImageNet Classification with Deep CNNs (AlexNet)" | The paper that started the deep learning boom. ReLU, dropout, GPU training |
| Simonyan & Zisserman (2014) "Very Deep Convolutional Networks (VGGNet)" | Simple idea: depth + small kernels works |
| He et al. (2016) "Deep Residual Learning for Image Recognition (ResNet)" | Skip connections. Enabled training 100+ layer networks. One of the most cited ML papers ever |
| Ioffe & Szegedy (2015) "Batch Normalization" | BN paper. Read to understand the original motivation, even if the explanation is debated |
| Srivastava et al. (2014) "Dropout: A Simple Way to Prevent Neural Networks from Overfitting" | Original dropout paper. Surprisingly readable, connects to ensemble methods |
| Kingma & Ba (2015) "Adam: A Method for Stochastic Optimization" | Original Adam paper. Understand the update rule from first principles |

---

## Phase 3: Sequence Models

**Read after Chapter 11**

| Paper | Why Read It |
|-------|-------------|
| Hochreiter & Schmidhuber (1997) "Long Short-Term Memory" | Original LSTM. Dense, but foundational. Focus on the cell state mechanism |
| Cho et al. (2014) "Learning Phrase Representations using RNN Encoder-Decoder (GRU)" | GRUs introduced. Simpler than LSTM |
| Bahdanau, Cho & Bengio (2015) "Neural Machine Translation by Jointly Learning to Align and Translate" | Original attention mechanism. The step before Transformers. Read carefully |
| Sutskever, Vinyals & Le (2014) "Sequence to Sequence Learning with Neural Networks" | Seq2seq encoder-decoder. Foundation of MT and later work |

---

## Phase 4: Transformers & Modern NLP

**Read after Chapter 12. These are non-negotiable.**

| Paper | Why Read It |
|-------|-------------|
| Vaswani et al. (2017) "Attention Is All You Need" | The Transformer paper. Read 3 times. Implement every equation |
| Devlin et al. (2018) "BERT: Pre-training of Deep Bidirectional Transformers" | BERT. Changed NLP. Understand masked LM and NSP |
| Radford et al. (2018) "Improving Language Understanding by Generative Pre-Training (GPT)" | GPT-1. Language model pre-training + fine-tuning paradigm |
| Radford et al. (2019) "Language Models are Unsupervised Multitask Learners (GPT-2)" | Zero-shot emergence, scaling begins |
| Brown et al. (2020) "Language Models are Few-Shot Learners (GPT-3)" | In-context learning, emergence at scale. Few-shot prompting introduced |
| Liu et al. (2019) "RoBERTa: A Robustly Optimized BERT" | What actually matters in BERT training? Good ablation study |
| Raffel et al. (2020) "Exploring the Limits of Transfer Learning with T5" | Text-to-text unified framework. The value is the scale of ablations |
| Wei et al. (2022) "Emergent Abilities of Large Language Models" | When do capabilities appear? Understanding scaling |

---

## Phase 5: Generative Models

**Read after Chapter 13**

| Paper | Why Read It |
|-------|-------------|
| Kingma & Welling (2013) "Auto-Encoding Variational Bayes (VAE)" | Original VAE. Derive the ELBO yourself before reading |
| Goodfellow et al. (2014) "Generative Adversarial Networks" | Original GAN. The minimax game formulation. Short and readable |
| Arjovsky, Chintala & Bottou (2017) "Wasserstein GAN" | Why training instability happens, and one fix. Math-heavy but important |
| Karras et al. (2019) "A Style-Based Generator Architecture for GANs (StyleGAN)" | Separating style and content. State-of-the-art image generation |
| Ho, Jain & Abbeel (2020) "Denoising Diffusion Probabilistic Models (DDPM)" | Original diffusion paper. Derive every equation |
| Rombach et al. (2022) "High-Resolution Image Synthesis with Latent Diffusion Models (Stable Diffusion)" | Diffusion in latent space for efficiency |

---

## Phase 6: Reinforcement Learning

**Read after Chapter 14**

| Paper | Why Read It |
|-------|-------------|
| Watkins & Dayan (1992) "Q-Learning" | Convergence proof for tabular Q-learning. Short |
| Mnih et al. (2015) "Human-level Control Through Deep RL (DQN)" | Original DQN. Playing Atari from pixels. Experience replay + target network |
| van Hasselt, Guez & Silver (2016) "Deep RL with Double Q-Learning" | Fixes overestimation bias in DQN |
| Wang et al. (2016) "Dueling Network Architectures for Deep RL" | Value + advantage decomposition |
| Schulman et al. (2017) "Proximal Policy Optimization Algorithms (PPO)" | The algorithm used in ChatGPT's RLHF |
| Ouyang et al. (2022) "Training Language Models to Follow Instructions with Human Feedback (InstructGPT)" | RLHF applied to LLMs. How ChatGPT was trained |

---

## Phase 7: Efficient & Practical ML

**Read with Chapter 15–16**

| Paper | Why Read It |
|-------|-------------|
| Chen & Guestrin (2016) "XGBoost: A Scalable Tree Boosting System" | XGBoost paper. Regularized objectives, second-order optimization, systems tricks |
| Hu et al. (2022) "LoRA: Low-Rank Adaptation of Large Language Models" | Parameter-efficient fine-tuning. 1/100th the trainable parameters, same performance |
| Dao et al. (2022) "FlashAttention: Fast and Memory-Efficient Exact Attention" | Tiling + recomputation. Major practical advancement for long contexts |
| Hinton, Vinyals & Dean (2015) "Distilling the Knowledge in a Neural Network" | Knowledge distillation. Soft targets, temperature scaling |

---

## Phase 8: Graph Neural Networks

| Paper | Why Read It |
|-------|-------------|
| Kipf & Welling (2017) "Semi-Supervised Classification with Graph Convolutional Networks (GCN)" | Original GCN. Spectral convolution simplified to local propagation |
| Veličković et al. (2018) "Graph Attention Networks (GAT)" | Attention applied to graphs. Fixes GCN's equal-weighting issue |
| Hamilton, Ying & Leskovec (2017) "Inductive Representation Learning on Large Graphs (GraphSAGE)" | Scales to large graphs via sampling. Practical |
| Xu et al. (2019) "How Powerful are Graph Neural Networks? (GIN)" | Theoretical analysis of GNN expressive power |

---

## The "Meta" Papers (Read These Whenever)

| Paper | Why Read It |
|-------|-------------|
| Lecun, Bengio & Hinton (2015) "Deep Learning" (Nature) | Concise review of the field by its founders |
| Karpathy (2015) "The Unreasonable Effectiveness of Recurrent Neural Networks" | Famous blog post, not a paper — but essential reading for intuition |
| Sculley et al. (2015) "Hidden Technical Debt in Machine Learning Systems" | Why ML systems are hard to maintain. Required reading before building production ML |
| Goodfellow et al. "Deep Learning" textbook | Not a paper but the canonical textbook — treat it as a long paper |

---

## How to Find Papers

- **Arxiv (arxiv.org):** cs.LG, stat.ML, cs.CV, cs.CL sections
- **Papers With Code (paperswithcode.com):** Papers + implementations + benchmarks
- **Semantic Scholar (semanticscholar.org):** Citation graph, related papers
- **Connected Papers (connectedpapers.com):** Visual citation map for any paper
- **Google Scholar:** Citation counts, find follow-up work

## Tracking Your Reading

Maintain a simple log:

```
Date | Paper | Status | Key Takeaway
-----|-------|--------|-------------
2024-03-15 | Attention Is All You Need | Read | Self-attention = O(n²) but parallelizable; learned positions; each layer is attn + FFN + residual
2024-03-22 | BERT | In Progress | Bidirectionality via masking; pretraining then fine-tuning changes NLP
```
