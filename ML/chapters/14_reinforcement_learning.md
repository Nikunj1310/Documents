# Chapter 14 — Reinforcement Learning

> *Reinforcement learning is the third paradigm of ML — learning by interacting with an environment. It underpins AlphaGo, ChatGPT's RLHF training, and robotics. The mathematical framework (MDPs) is beautiful and powerful.*

---

## 14.1 The RL Framework

Unlike supervised learning (learn from labeled examples) or unsupervised learning (find structure in data), in RL:

- An **agent** interacts with an **environment**
- At each time step, the agent observes a **state** `sₜ`, takes an **action** `aₜ`
- The environment transitions to **next state** `sₜ₊₁` and returns a **reward** `rₜ`
- The agent's goal: maximize **cumulative reward** over time

```
Agent → action aₜ → Environment
       ← state sₜ, reward rₜ ←
```

**Key difference from supervised learning:** No teacher providing correct answers. The agent must discover which actions lead to rewards through trial and error. Rewards may be delayed (e.g., reward only at the end of a chess game).

---

## 14.2 Markov Decision Processes (MDPs)

The formal framework for RL problems.

An MDP is a tuple **(S, A, P, R, γ)**:
- **S** — state space
- **A** — action space
- **P(s' | s, a)** — transition probability (probability of next state given current state and action)
- **R(s, a, s')** — reward function (scalar received after transition)
- **γ ∈ [0, 1)** — discount factor

**Markov property:** The future is independent of the past given the present state:
```
P(sₜ₊₁ | sₜ, aₜ, sₜ₋₁, aₜ₋₁, ...) = P(sₜ₊₁ | sₜ, aₜ)
```

The current state captures all relevant history. This is the "memory" assumption of MDPs.

### Policy

A **policy** π defines how the agent behaves:
- **Deterministic:** `π(s) = a` (deterministic action given state)
- **Stochastic:** `π(a | s) = P(action=a | state=s)` (probability distribution over actions)

### Return

The **discounted return** starting at time t:
```
Gₜ = rₜ + γrₜ₊₁ + γ²rₜ₊₂ + ... = Σₖ₌₀^∞ γᵏ rₜ₊ₖ
```

The discount factor γ:
- `γ = 0`: Only care about immediate reward (myopic)
- `γ = 1`: All future rewards equally valuable (requires finite episodes)
- `0 < γ < 1`: Balance immediate and future rewards; geometric decay

---

## 14.3 Value Functions

### State Value Function

The expected return starting from state s and following policy π:
```
V^π(s) = E_π[Gₜ | sₜ = s] = E_π[Σₖ γᵏ rₜ₊ₖ | sₜ = s]
```

### Action-Value Function (Q-function)

The expected return starting from state s, taking action a, then following π:
```
Q^π(s, a) = E_π[Gₜ | sₜ = s, aₜ = a]
```

### Bellman Equations

The Bellman equations express value functions recursively:
```
V^π(s) = Σₐ π(a|s) Σₛ' P(s'|s,a) [R(s,a,s') + γ V^π(s')]

Q^π(s,a) = Σₛ' P(s'|s,a) [R(s,a,s') + γ Σₐ' π(a'|s') Q^π(s',a')]
```

**Optimal value functions** satisfy the Bellman optimality equations:
```
V*(s) = max_a Σₛ' P(s'|s,a) [R(s,a,s') + γ V*(s')]

Q*(s,a) = Σₛ' P(s'|s,a) [R(s,a,s') + γ max_a' Q*(s',a')]
```

The optimal policy: `π*(s) = argmax_a Q*(s, a)`.

---

## 14.4 Dynamic Programming (Model-Based RL)

When we know the full MDP (P and R), we can compute V* exactly using dynamic programming.

### Policy Evaluation

Compute V^π for a fixed policy:
```
V^π(s) ← Σₐ π(a|s) Σₛ' P(s'|s,a) [R(s,a,s') + γ V^π(s')]
```
Repeat until convergence.

### Policy Improvement

Given V^π, improve the policy:
```
π'(s) = argmax_a Σₛ' P(s'|s,a) [R(s,a,s') + γ V^π(s')]
```

### Policy Iteration

Alternate between evaluation and improvement until convergence → optimal policy.

### Value Iteration

Skip full evaluation; update value directly toward optimal:
```
V(s) ← max_a Σₛ' P(s'|s,a) [R(s,a,s') + γ V(s')]
```

Both converge, but DP requires the full model — impractical for large state spaces or unknown environments.

---

## 14.5 Temporal Difference (TD) Learning

Learn directly from experience (without a model). The core idea: update value estimates based on single transitions.

**TD(0) Update:**
```
V(s) ← V(s) + α [r + γV(s') - V(s)]
```

The term `[r + γV(s') - V(s)]` is the **TD error** or **Bellman error**:
- `r + γV(s')` is the "target" (better estimate of V(s) using one step of lookahead)
- `V(s)` is the current estimate
- The difference tells us how to update

**Key property:** TD doesn't wait for the episode to end (unlike Monte Carlo). Updates happen at every step.

---

## 14.6 Q-Learning

The most fundamental RL algorithm. Learns Q* directly:

```
Q(s, a) ← Q(s, a) + α [r + γ max_a' Q(s', a') - Q(s, a)]
```

The target `r + γ max_a' Q(s', a')` uses the greedy action in the next state — this is "off-policy" learning.

### ε-Greedy Exploration

To balance exploration and exploitation:
```
With probability ε:  take a random action (explore)
With probability 1-ε: take argmax_a Q(s,a) (exploit)
```

Decay ε over training from 1.0 (pure exploration) to 0.01 (mostly exploitation).

### Tabular Q-Learning

```python
import numpy as np
import gymnasium as gym

env = gym.make('FrozenLake-v1', is_slippery=False)

n_states = env.observation_space.n    # 16
n_actions = env.action_space.n        # 4
Q = np.zeros((n_states, n_actions))

# Hyperparameters
alpha = 0.8    # learning rate
gamma = 0.95   # discount factor
epsilon = 1.0
epsilon_decay = 0.995
epsilon_min = 0.01
n_episodes = 10000

rewards_history = []

for episode in range(n_episodes):
    state, _ = env.reset()
    total_reward = 0
    done = False
    
    while not done:
        # Epsilon-greedy action
        if np.random.random() < epsilon:
            action = env.action_space.sample()
        else:
            action = np.argmax(Q[state])
        
        next_state, reward, terminated, truncated, _ = env.step(action)
        done = terminated or truncated
        
        # Q-learning update
        target = reward + gamma * np.max(Q[next_state]) * (not done)
        Q[state, action] += alpha * (target - Q[state, action])
        
        state = next_state
        total_reward += reward
    
    epsilon = max(epsilon * epsilon_decay, epsilon_min)
    rewards_history.append(total_reward)

print(f"Average reward (last 100): {np.mean(rewards_history[-100:]):.3f}")
```

---

## 14.7 Deep Q-Networks (DQN)

For large/continuous state spaces (e.g., Atari game pixels), we can't maintain a Q-table. Use a neural network to approximate Q:

```
Q(s, a; θ) ≈ Q*(s, a)
```

### DQN Loss

```
L(θ) = E[(y - Q(s, a; θ))²]
y = r + γ max_a' Q(s', a'; θ⁻)    (target network)
```

### Two Key Tricks

**1. Experience Replay:** Store transitions (s, a, r, s') in a replay buffer. Sample mini-batches randomly for updates. This:
- Breaks temporal correlations between consecutive samples
- Allows reusing each experience multiple times

**2. Target Network:** Use a separate, slowly-updated network `θ⁻` for the TD target. This:
- Stabilizes training by keeping targets fixed temporarily
- Prevents chasing a moving target

```python
import torch
import torch.nn as nn
import torch.optim as optim
from collections import deque
import random

class DQN(nn.Module):
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, 128), nn.ReLU(),
            nn.Linear(128, 128), nn.ReLU(),
            nn.Linear(128, action_dim)
        )
    def forward(self, x): return self.net(x)

class ReplayBuffer:
    def __init__(self, capacity=10000):
        self.buffer = deque(maxlen=capacity)
    
    def push(self, state, action, reward, next_state, done):
        self.buffer.append((state, action, reward, next_state, done))
    
    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        states, actions, rewards, next_states, dones = zip(*batch)
        return (torch.FloatTensor(states), torch.LongTensor(actions),
                torch.FloatTensor(rewards), torch.FloatTensor(next_states),
                torch.BoolTensor(dones))
    
    def __len__(self): return len(self.buffer)

# Training
env = gym.make('CartPole-v1')
state_dim = env.observation_space.shape[0]  # 4
action_dim = env.action_space.n              # 2

policy_net = DQN(state_dim, action_dim)
target_net = DQN(state_dim, action_dim)
target_net.load_state_dict(policy_net.state_dict())

optimizer = optim.Adam(policy_net.parameters(), lr=1e-3)
buffer = ReplayBuffer(10000)

epsilon = 1.0
batch_size = 64
gamma = 0.99
target_update = 10  # update target network every N episodes

for episode in range(500):
    state, _ = env.reset()
    total_reward = 0
    
    while True:
        # Action selection
        if random.random() < epsilon:
            action = env.action_space.sample()
        else:
            with torch.no_grad():
                q_values = policy_net(torch.FloatTensor(state))
                action = q_values.argmax().item()
        
        next_state, reward, terminated, truncated, _ = env.step(action)
        done = terminated or truncated
        buffer.push(state, action, reward, next_state, done)
        state = next_state
        total_reward += reward
        
        # Train
        if len(buffer) >= batch_size:
            states, actions, rewards, next_states, dones = buffer.sample(batch_size)
            
            current_q = policy_net(states).gather(1, actions.unsqueeze(1)).squeeze(1)
            with torch.no_grad():
                next_q = target_net(next_states).max(1)[0]
                target_q = rewards + gamma * next_q * (~dones)
            
            loss = nn.MSELoss()(current_q, target_q)
            optimizer.zero_grad()
            loss.backward()
            torch.nn.utils.clip_grad_norm_(policy_net.parameters(), 1.0)
            optimizer.step()
        
        if done:
            break
    
    epsilon = max(epsilon * 0.995, 0.01)
    if episode % target_update == 0:
        target_net.load_state_dict(policy_net.state_dict())
    
    if episode % 50 == 0:
        print(f"Episode {episode}, Reward: {total_reward:.1f}, ε: {epsilon:.3f}")
```

---

## 14.8 Policy Gradient Methods

Q-learning learns the value function; the policy is derived implicitly. Policy gradient methods directly optimize the policy.

**Policy objective:** Maximize expected return:
```
J(θ) = E_{τ~π_θ}[R(τ)]    where τ is a trajectory
```

**Policy Gradient Theorem:**
```
∇_θ J(θ) = E_{τ~π_θ}[Σₜ ∇_θ log π_θ(aₜ|sₜ) · Gₜ]
```

The gradient of the log-policy, weighted by the return. Intuition:
- If action aₜ led to high return (Gₜ > 0): increase its probability
- If action aₜ led to low return (Gₜ < 0): decrease its probability

### REINFORCE Algorithm

```python
class PolicyNetwork(nn.Module):
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, 64), nn.ReLU(),
            nn.Linear(64, action_dim), nn.Softmax(dim=-1)
        )
    def forward(self, x): return self.net(x)

policy = PolicyNetwork(4, 2)
optimizer = optim.Adam(policy.parameters(), lr=1e-3)

for episode in range(1000):
    state, _ = env.reset()
    log_probs, rewards = [], []
    
    # Collect trajectory
    while True:
        state_t = torch.FloatTensor(state)
        probs = policy(state_t)
        dist = torch.distributions.Categorical(probs)
        action = dist.sample()
        
        log_probs.append(dist.log_prob(action))
        state, reward, terminated, truncated, _ = env.step(action.item())
        rewards.append(reward)
        if terminated or truncated: break
    
    # Compute discounted returns
    G = 0
    returns = []
    for r in reversed(rewards):
        G = r + gamma * G
        returns.insert(0, G)
    
    returns = torch.tensor(returns, dtype=torch.float32)
    returns = (returns - returns.mean()) / (returns.std() + 1e-8)  # normalize
    
    # Policy gradient update
    log_probs = torch.stack(log_probs)
    loss = -(log_probs * returns).mean()
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

**Problem with REINFORCE:** High variance in gradient estimates (each trajectory is different). Solutions:
- **Baseline subtraction:** `Gₜ - b(sₜ)` where b is a baseline (e.g., value function V(sₜ)). Reduces variance without bias.
- **Actor-Critic:** Use a learned value function as the baseline continuously.

---

## 14.9 Actor-Critic Methods

Combines value function (critic) with policy (actor):

- **Actor** `π_θ(a|s)`: policy network, updated by policy gradient
- **Critic** `V_φ(s)`: value network, updated by TD error

**Advantage function:** `A(s,a) = Q(s,a) - V(s)` — how much better is this action than average?

**A2C (Advantage Actor-Critic) Update:**
```
Actor loss:  L_actor = -E[log π_θ(a|s) · A(s,a)]
Critic loss: L_critic = E[(r + γV(s') - V(s))²]
```

**PPO (Proximal Policy Optimization):** State-of-the-art on-policy algorithm. Clips the policy update to prevent too-large changes:
```
L_PPO = E[min(r_t(θ) A_t, clip(r_t(θ), 1-ε, 1+ε) A_t)]
```

where `r_t(θ) = π_θ(aₜ|sₜ) / π_θ_old(aₜ|sₜ)` is the probability ratio. PPO is the algorithm used to train ChatGPT via RLHF.

---

## 14.10 RLHF (RL from Human Feedback)

How ChatGPT and similar models are trained:

1. **SFT (Supervised Fine-Tuning):** Fine-tune a language model on high-quality demonstrations
2. **Reward Modeling:** Train a reward model that predicts human preference between pairs of outputs
3. **RL Optimization:** Use PPO to optimize the language model to maximize the reward model's score

```
prompt → LM → response → Reward Model → scalar reward → PPO update
```

The reward model captures human preferences. PPO optimizes the LM policy to generate responses that humans prefer.

---

## Exercises

1. **Value iteration from scratch:** Implement value iteration for the FrozenLake-v1 environment (get the true P and R from the gym). Show the value function at each iteration. How many iterations until convergence?

2. **Q-learning convergence:** Show that Q-learning converges to Q* on a simple 4-state MDP by running it and comparing to the analytically computed Q*. What happens when ε is too low? Too high?

3. **DQN on CartPole:** Train a DQN on CartPole-v1. Plot the mean episode reward over training. Ablate: remove experience replay (train on sequential transitions) and observe instability. Remove the target network and observe.

4. **REINFORCE baseline:** Train REINFORCE with no baseline, a constant baseline (mean return), and a learned value baseline. Plot learning curves for all three. Which has the lowest variance?

5. **Policy visualization:** Train a policy on a simple GridWorld environment (implement a 5×5 grid with obstacles). After training, visualize the policy as arrows at each cell. Does the policy make sense?

---

## Further Reading

- Sutton & Barto "Reinforcement Learning: An Introduction" (2nd ed.) — free PDF, the canonical textbook
- David Silver's RL Course (UCL, 2015) — free on YouTube, rigorous
- Mnih et al. (2015) "Human-level control through deep RL" — original DQN paper (Atari)
- Schulman et al. (2017) "Proximal Policy Optimization Algorithms" — PPO paper
- Ouyang et al. (2022) "Training Language Models to Follow Instructions with Human Feedback" — InstructGPT / RLHF
