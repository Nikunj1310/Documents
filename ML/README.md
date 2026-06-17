# ML Study System — Master Index

> **Start here.** This file is your navigation hub for the entire system.

---

## Quick Start

```
New to ML?       → Start at chapters/01_math_foundations.md
Know the math?   → Start at chapters/03_core_ml_concepts.md  
Know classical ML? → Start at chapters/09_neural_networks.md
Just want to code? → Jump to any project in projects/
Interview prep?  → extras/interview_prep.md
Quick reference? → cheatsheets/
```

---

## Full File Map

```
Documents/ML/
│
├── README.md                          ← This file
├── 00_strategy.md                     ← Full study strategy, roadmap, schedule
│
├── chapters/
│   ├── 01_math_foundations.md         Phase 0  — Linear algebra, calculus, probability
│   ├── 02_python_data_science_tools.md Phase 1 — NumPy, Pandas, Matplotlib, sklearn
│   ├── 03_core_ml_concepts.md         Phase 2  — The learning problem, ERM, GD, bias-variance
│   ├── 04_linear_models.md            Phase 3a — Linear/logistic regression, regularization
│   ├── 05_tree_based_methods.md       Phase 3b — Trees, Random Forests, XGBoost, LightGBM
│   ├── 06_classical_ml.md             Phase 3c — SVM, k-NN, Naive Bayes, clustering, PCA
│   ├── 07_model_evaluation.md         Phase 4a — Metrics, CV strategies, hyperparameter tuning
│   ├── 08_feature_engineering.md      Phase 4b — Encoding, scaling, creation, selection
│   ├── 09_neural_networks.md          Phase 5  — MLP from scratch, backprop, PyTorch
│   ├── 10_convolutional_neural_networks.md Phase 6 — CNNs, ResNet, transfer learning
│   ├── 11_sequence_models.md          Phase 7  — RNNs, LSTMs, attention mechanism
│   ├── 12_transformers_nlp.md         Phase 8  — Transformers, BERT, GPT, fine-tuning
│   ├── 13_generative_models.md        Phase 9  — VAEs, GANs, Diffusion Models
│   ├── 14_reinforcement_learning.md   Phase 10 — MDPs, Q-learning, DQN, PPO, RLHF
│   ├── 15_ml_production.md            Phase 11 — MLflow, FastAPI, Docker, monitoring
│   └── 16_advanced_topics.md          Phase 12 — GNNs, LLM agents, multimodal, efficient ML
│
├── projects/
│   ├── project_01_house_price_prediction.md  — End-to-end tabular ML
│   ├── project_02_image_classification.md    — CNN + transfer learning
│   ├── project_03_nlp_text_classification.md — BERT fine-tuning
│   ├── project_04_recommendation_system.md   — Matrix factorization + Neural CF
│   └── project_05_rl_agent.md                — DQN agents
│
├── cheatsheets/
│   ├── cs_numpy_pandas.md             — Array/DataFrame operations quick reference
│   ├── cs_sklearn.md                  — Sklearn API, preprocessing, metrics
│   ├── cs_pytorch.md                  — PyTorch tensors, training loop, modules
│   ├── cs_math.md                     — Key equations across all chapters
│   └── cs_algorithms.md               — When to use which algorithm
│
└── extras/
    ├── interview_prep.md              — Coding + theory interview questions & answers
    └── paper_reading_list.md          — Seminal papers in order of reading
```

---

## Chapter Dependencies

```
Ch01 (Math)
  └── Ch03 (Core ML)
        ├── Ch04 (Linear Models)
        │     └── Ch07 (Evaluation)
        │           └── Ch08 (Feature Eng.)
        ├── Ch05 (Trees)          ─── Ch07, Ch08
        ├── Ch06 (Classical ML)   ─── Ch07
        └── Ch09 (Neural Nets)
              ├── Ch10 (CNNs)
              │     └── Ch13 (Generative)
              ├── Ch11 (Sequences)
              │     └── Ch12 (Transformers)
              │           └── Ch13 (Generative, VAE side)
              ├── Ch14 (RL)
              └── Ch15 (Production)   ← depends on any trained model

Ch02 (Python Tools)  ← prerequisite for all practical work
Ch16 (Advanced)      ← reads after Ch09–Ch15
```

---

## Progress Tracker

Copy this into a note and check off as you go. Include the date you finished each.

```
FOUNDATIONS
[ ] Ch01 — Math Foundations                      Finished: ________
[ ] Ch02 — Python & Data Science Tools           Finished: ________

CLASSICAL ML
[ ] Ch03 — Core ML Concepts                      Finished: ________
[ ] Ch04 — Linear Models                         Finished: ________
[ ] Ch05 — Tree-Based Methods                    Finished: ________
[ ] Ch06 — Classical ML                          Finished: ________
[ ] Ch07 — Model Evaluation                      Finished: ________
[ ] Ch08 — Feature Engineering                   Finished: ________

DEEP LEARNING
[ ] Ch09 — Neural Networks from Scratch          Finished: ________
[ ] Ch10 — Convolutional Neural Networks         Finished: ________
[ ] Ch11 — Sequence Models & RNNs               Finished: ________
[ ] Ch12 — Transformers & Modern NLP             Finished: ________
[ ] Ch13 — Generative Models                     Finished: ________
[ ] Ch14 — Reinforcement Learning                Finished: ________
[ ] Ch15 — ML in Production                      Finished: ________
[ ] Ch16 — Advanced Topics                       Finished: ________

PROJECTS
[ ] Project 01 — House Price Prediction          Finished: ________
[ ] Project 02 — Image Classification            Finished: ________
[ ] Project 03 — NLP Text Classification         Finished: ________
[ ] Project 04 — Recommendation System           Finished: ________
[ ] Project 05 — RL Agent                        Finished: ________

EXTRAS
[ ] Interview Prep — Theory Questions            Finished: ________
[ ] Interview Prep — Coding Questions            Finished: ________
[ ] Paper Reading List — Core Papers             Finished: ________
```

---

## Recommended Weekly Schedule

| Week | Chapters | Project Work |
|------|----------|--------------|
| 1–3 | Ch01 | Math exercises |
| 4 | Ch02 | Python exercises |
| 5–6 | Ch03 | GD experiments |
| 7–8 | Ch04 | Start Project 01 |
| 9–11 | Ch05, Ch06 | Project 01 continued |
| 12–13 | Ch07, Ch08 | Finish Project 01 |
| 14–16 | Ch09 | MLP from scratch |
| 17–18 | Ch10 | Start Project 02 |
| 19–20 | Ch11 | Finish Project 02 |
| 21–23 | Ch12 | Start Project 03 |
| 24–25 | Ch13 | Finish Project 03 |
| 26–28 | Ch14 | Start Project 05 |
| 29–30 | Ch15 | Deploy Project 01 |
| 31–35 | Ch16 | Start Project 04, interview prep |
