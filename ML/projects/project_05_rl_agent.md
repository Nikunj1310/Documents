# Project 5 — Reinforcement Learning Agent

**Covers:** Chapter 14  
**Estimated time:** 2–3 weeks  
**Difficulty:** Advanced

---

## Project Goal

Train a DQN agent to master several RL environments, progressing from simple to complex:

1. **CartPole-v1** — Prove the implementation works (trivial environment)
2. **LunarLander-v2** — First real challenge (continuous state, sparse reward)
3. **MountainCar-v0** — Sparse reward problem (reward shaping)
4. **(Optional) Atari Breakout** — Raw pixel input with CNN

By the end, you will understand the core challenges in RL: exploration, sparse rewards, sample efficiency, and hyperparameter sensitivity.

---

## Setup

```bash
pip install gymnasium[classic-control] gymnasium[box2d] gymnasium[atari]
pip install torch torchvision matplotlib numpy
```

```python
import gymnasium as gym
import torch
import torch.nn as nn
import torch.nn.functional as F
import numpy as np
import random
import matplotlib.pyplot as plt
from collections import deque, namedtuple
import os

# Reproducibility
torch.manual_seed(42)
np.random.seed(42)
random.seed(42)

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using: {device}")
```

---

## Core Components

### Experience Replay Buffer

```python
Transition = namedtuple('Transition', ['state', 'action', 'reward', 'next_state', 'done'])

class ReplayBuffer:
    def __init__(self, capacity: int):
        self.memory = deque(maxlen=capacity)
    
    def push(self, *args):
        self.memory.append(Transition(*args))
    
    def sample(self, batch_size: int):
        batch = random.sample(self.memory, batch_size)
        batch = Transition(*zip(*batch))
        
        states = torch.tensor(np.array(batch.state), dtype=torch.float32, device=device)
        actions = torch.tensor(batch.action, dtype=torch.long, device=device).unsqueeze(1)
        rewards = torch.tensor(batch.reward, dtype=torch.float32, device=device).unsqueeze(1)
        next_states = torch.tensor(np.array(batch.next_state), dtype=torch.float32, device=device)
        dones = torch.tensor(batch.done, dtype=torch.float32, device=device).unsqueeze(1)
        
        return states, actions, rewards, next_states, dones
    
    def __len__(self):
        return len(self.memory)
```

### DQN Network

```python
class DQNNetwork(nn.Module):
    def __init__(self, state_dim: int, action_dim: int, hidden_dims: list = [256, 256]):
        super().__init__()
        
        layers = []
        in_dim = state_dim
        for h in hidden_dims:
            layers.extend([nn.Linear(in_dim, h), nn.ReLU()])
            in_dim = h
        layers.append(nn.Linear(in_dim, action_dim))
        
        self.net = nn.Sequential(*layers)
        self._init_weights()
    
    def _init_weights(self):
        for layer in self.net:
            if isinstance(layer, nn.Linear):
                nn.init.kaiming_uniform_(layer.weight, nonlinearity='relu')
                nn.init.zeros_(layer.bias)
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.net(x)
```

### Dueling DQN (Improvement)

```python
class DuelingDQN(nn.Module):
    """
    Dueling architecture: separate streams for value V(s) and advantage A(s,a).
    Q(s,a) = V(s) + A(s,a) - mean(A(s,·))
    """
    def __init__(self, state_dim: int, action_dim: int):
        super().__init__()
        
        self.shared = nn.Sequential(
            nn.Linear(state_dim, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU()
        )
        
        self.value_stream = nn.Sequential(
            nn.Linear(256, 128), nn.ReLU(),
            nn.Linear(128, 1)
        )
        
        self.advantage_stream = nn.Sequential(
            nn.Linear(256, 128), nn.ReLU(),
            nn.Linear(128, action_dim)
        )
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        shared = self.shared(x)
        value = self.value_stream(shared)
        advantage = self.advantage_stream(shared)
        # Subtract mean advantage for stability
        return value + (advantage - advantage.mean(dim=1, keepdim=True))
```

### Full DQN Agent

```python
class DQNAgent:
    def __init__(self, state_dim, action_dim, config):
        self.action_dim = action_dim
        self.config = config
        
        # Networks
        self.policy_net = DuelingDQN(state_dim, action_dim).to(device)
        self.target_net = DuelingDQN(state_dim, action_dim).to(device)
        self.target_net.load_state_dict(self.policy_net.state_dict())
        self.target_net.eval()
        
        # Optimizer
        self.optimizer = torch.optim.Adam(
            self.policy_net.parameters(), 
            lr=config['lr'],
            weight_decay=config.get('weight_decay', 0)
        )
        
        # Replay buffer
        self.buffer = ReplayBuffer(config['buffer_size'])
        
        # Epsilon (exploration)
        self.epsilon = config['epsilon_start']
        self.steps = 0
    
    def select_action(self, state: np.ndarray) -> int:
        if random.random() < self.epsilon:
            return random.randrange(self.action_dim)
        
        with torch.no_grad():
            state_t = torch.tensor(state, dtype=torch.float32, device=device).unsqueeze(0)
            q_values = self.policy_net(state_t)
            return q_values.argmax().item()
    
    def update_epsilon(self):
        self.epsilon = max(
            self.config['epsilon_end'],
            self.epsilon * self.config['epsilon_decay']
        )
    
    def train_step(self):
        if len(self.buffer) < self.config['batch_size']:
            return None
        
        states, actions, rewards, next_states, dones = self.buffer.sample(
            self.config['batch_size']
        )
        
        # Current Q values
        current_q = self.policy_net(states).gather(1, actions)
        
        # Target Q values (Double DQN: use policy net to select action, target net to evaluate)
        with torch.no_grad():
            next_actions = self.policy_net(next_states).argmax(1, keepdim=True)
            next_q = self.target_net(next_states).gather(1, next_actions)
            target_q = rewards + (1 - dones) * self.config['gamma'] * next_q
        
        # Huber loss (more robust to outliers than MSE)
        loss = F.smooth_l1_loss(current_q, target_q)
        
        self.optimizer.zero_grad()
        loss.backward()
        torch.nn.utils.clip_grad_norm_(self.policy_net.parameters(), 10.0)
        self.optimizer.step()
        
        self.steps += 1
        
        # Update target network
        if self.steps % self.config['target_update'] == 0:
            self.target_net.load_state_dict(self.policy_net.state_dict())
        
        return loss.item()
    
    def save(self, path):
        torch.save({
            'policy_state_dict': self.policy_net.state_dict(),
            'optimizer_state_dict': self.optimizer.state_dict(),
            'epsilon': self.epsilon,
            'steps': self.steps,
            'config': self.config
        }, path)
    
    def load(self, path):
        checkpoint = torch.load(path, map_location=device)
        self.policy_net.load_state_dict(checkpoint['policy_state_dict'])
        self.target_net.load_state_dict(checkpoint['policy_state_dict'])
        self.optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
        self.epsilon = checkpoint['epsilon']
        self.steps = checkpoint['steps']
```

---

## Training Loop

```python
def train_agent(env_name, config, n_episodes=1000, verbose=True):
    env = gym.make(env_name)
    state_dim = env.observation_space.shape[0]
    action_dim = env.action_space.n
    
    agent = DQNAgent(state_dim, action_dim, config)
    
    episode_rewards = []
    episode_lengths = []
    losses = []
    best_avg_reward = -float('inf')
    
    for episode in range(n_episodes):
        state, _ = env.reset(seed=episode)
        total_reward = 0
        episode_loss = []
        
        for step in range(config.get('max_steps', 1000)):
            action = agent.select_action(state)
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            
            # Store transition
            agent.buffer.push(state, action, reward, next_state, float(done))
            
            # Train
            loss = agent.train_step()
            if loss is not None:
                episode_loss.append(loss)
            
            state = next_state
            total_reward += reward
            
            if done:
                break
        
        agent.update_epsilon()
        episode_rewards.append(total_reward)
        episode_lengths.append(step + 1)
        if episode_loss:
            losses.append(np.mean(episode_loss))
        
        # Logging
        if verbose and (episode + 1) % 50 == 0:
            avg_reward = np.mean(episode_rewards[-100:])
            print(f"Episode {episode+1:4d} | "
                  f"Avg Reward (100): {avg_reward:8.2f} | "
                  f"ε: {agent.epsilon:.3f} | "
                  f"Buffer: {len(agent.buffer):6d}")
            
            if avg_reward > best_avg_reward:
                best_avg_reward = avg_reward
                agent.save(f'{env_name}_best.pth')
    
    env.close()
    return agent, episode_rewards, losses

# Plot training
def plot_training(rewards, losses, window=50):
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))
    
    # Reward curve
    ax1.plot(rewards, alpha=0.3, color='blue', label='Per-episode')
    smoothed = pd.Series(rewards).rolling(window).mean()
    ax1.plot(smoothed, color='blue', linewidth=2, label=f'{window}-ep average')
    ax1.set_xlabel('Episode')
    ax1.set_ylabel('Total Reward')
    ax1.set_title('Training Rewards')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    
    # Loss curve
    if losses:
        ax2.plot(losses, alpha=0.5)
        ax2.set_xlabel('Episode')
        ax2.set_ylabel('Loss')
        ax2.set_title('Training Loss')
        ax2.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.show()
```

---

## Environment 1: CartPole

```python
cartpole_config = {
    'lr': 1e-3,
    'gamma': 0.99,
    'epsilon_start': 1.0,
    'epsilon_end': 0.01,
    'epsilon_decay': 0.995,
    'batch_size': 64,
    'buffer_size': 10_000,
    'target_update': 10,    # update every N steps
    'max_steps': 500
}

agent, rewards, losses = train_agent('CartPole-v1', cartpole_config, n_episodes=300)
plot_training(rewards, losses)
# Target: solve CartPole (avg reward > 475 over 100 consecutive episodes)
```

---

## Environment 2: LunarLander

```python
lunar_config = {
    'lr': 5e-4,
    'gamma': 0.99,
    'epsilon_start': 1.0,
    'epsilon_end': 0.01,
    'epsilon_decay': 0.997,
    'batch_size': 128,
    'buffer_size': 100_000,
    'target_update': 100,
    'max_steps': 1000
}

agent, rewards, losses = train_agent('LunarLander-v2', lunar_config, n_episodes=1000)
plot_training(rewards, losses)
# Target: avg reward > 200 (environment is "solved")
```

---

## Environment 3: MountainCar (Sparse Reward)

MountainCar is deceptively hard — the car reaches the goal so rarely that basic DQN barely explores.

```python
# Solution: Reward shaping — add a dense reward signal based on progress
class ShapedMountainCar(gym.Wrapper):
    def step(self, action):
        obs, reward, terminated, truncated, info = self.env.step(action)
        position, velocity = obs
        
        # Shape the reward: reward high positions and speed
        shaped_reward = reward + 5 * abs(velocity) + 3 * (position - (-0.5))
        
        return obs, shaped_reward, terminated, truncated, info

env_factory = lambda: ShapedMountainCar(gym.make('MountainCar-v0'))

# Or use a different exploration strategy: noisy networks
class NoisyLinear(nn.Module):
    """Learned noise for exploration — no epsilon needed."""
    def __init__(self, in_features, out_features, sigma_init=0.5):
        super().__init__()
        self.in_features = in_features
        self.out_features = out_features
        self.sigma_init = sigma_init
        
        self.weight_mu = nn.Parameter(torch.empty(out_features, in_features))
        self.weight_sigma = nn.Parameter(torch.empty(out_features, in_features))
        self.bias_mu = nn.Parameter(torch.empty(out_features))
        self.bias_sigma = nn.Parameter(torch.empty(out_features))
        
        self.register_buffer('weight_epsilon', torch.empty(out_features, in_features))
        self.register_buffer('bias_epsilon', torch.empty(out_features))
        
        self.reset_parameters()
        self.sample_noise()
    
    def reset_parameters(self):
        mu_range = 1 / np.sqrt(self.in_features)
        self.weight_mu.data.uniform_(-mu_range, mu_range)
        self.weight_sigma.data.fill_(self.sigma_init / np.sqrt(self.in_features))
        self.bias_mu.data.uniform_(-mu_range, mu_range)
        self.bias_sigma.data.fill_(self.sigma_init / np.sqrt(self.out_features))
    
    def sample_noise(self):
        self.weight_epsilon.normal_()
        self.bias_epsilon.normal_()
    
    def forward(self, x):
        if self.training:
            weight = self.weight_mu + self.weight_sigma * self.weight_epsilon
            bias = self.bias_mu + self.bias_sigma * self.bias_epsilon
        else:
            weight = self.weight_mu
            bias = self.bias_mu
        return F.linear(x, weight, bias)
```

---

## Evaluation & Visualization

```python
def evaluate_agent(agent, env_name, n_episodes=10, render=False):
    """Evaluate a trained agent without exploration."""
    render_mode = 'human' if render else None
    env = gym.make(env_name, render_mode=render_mode)
    
    episode_rewards = []
    for _ in range(n_episodes):
        state, _ = env.reset()
        total_reward = 0
        done = False
        
        while not done:
            with torch.no_grad():
                state_t = torch.tensor(state, dtype=torch.float32, device=device).unsqueeze(0)
                action = agent.policy_net(state_t).argmax().item()
            
            state, reward, terminated, truncated, _ = env.step(action)
            total_reward += reward
            done = terminated or truncated
        
        episode_rewards.append(total_reward)
    
    env.close()
    print(f"Evaluation ({n_episodes} episodes):")
    print(f"  Mean: {np.mean(episode_rewards):.2f}")
    print(f"  Std:  {np.std(episode_rewards):.2f}")
    print(f"  Min:  {np.min(episode_rewards):.2f}")
    print(f"  Max:  {np.max(episode_rewards):.2f}")
    return episode_rewards

# Visualize Q-values for CartPole (2D state space: pole angle vs angular velocity)
def visualize_q_values(agent, n_grid=30):
    angles = np.linspace(-0.3, 0.3, n_grid)
    ang_vels = np.linspace(-2, 2, n_grid)
    
    q_diff = np.zeros((n_grid, n_grid))
    
    for i, angle in enumerate(angles):
        for j, ang_vel in enumerate(ang_vels):
            state = torch.tensor([[0, 0, angle, ang_vel]], dtype=torch.float32, device=device)
            with torch.no_grad():
                q = agent.policy_net(state)[0]
            q_diff[j, i] = (q[0] - q[1]).item()  # push left vs push right
    
    plt.figure(figsize=(8, 6))
    plt.contourf(angles, ang_vels, q_diff, cmap='RdBu', levels=20)
    plt.colorbar(label='Q(s,left) - Q(s,right)')
    plt.xlabel('Pole Angle')
    plt.ylabel('Angular Velocity')
    plt.title('Policy: Prefer Left (blue) or Right (red)?')
    plt.show()
```

---

## Ablation Studies

Critical: understand WHY the components work.

```python
# Run the same environment with different configurations

def ablation(env_name, base_config, ablations, n_episodes=500, n_runs=3):
    results = {}
    
    for name, config_override in ablations.items():
        config = {**base_config, **config_override}
        run_rewards = []
        
        for run in range(n_runs):
            _, rewards, _ = train_agent(env_name, config, n_episodes, verbose=False)
            run_rewards.append(rewards)
        
        results[name] = np.array(run_rewards).mean(axis=0)
    
    # Plot
    plt.figure(figsize=(10, 6))
    for name, rewards in results.items():
        smoothed = pd.Series(rewards).rolling(50).mean()
        plt.plot(smoothed, label=name, linewidth=2)
    plt.xlabel('Episode')
    plt.ylabel('Average Reward (50-ep window)')
    plt.title(f'DQN Ablation: {env_name}')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()

ablation_configs = {
    'Full DQN (Dueling + Double)': {},
    'No Double DQN':               {'double_dqn': False},
    'No Replay Buffer':            {'buffer_size': 1},   # effectively no replay
    'Vanilla (no improvements)':   {'double_dqn': False, 'buffer_size': 1, 'dueling': False}
}
```

---

## Extension: Deep RL on Atari

For those with a GPU, train on Atari:

```python
import torchvision.transforms as T

class AtariDQN(nn.Module):
    """CNN-based DQN for raw pixel input."""
    def __init__(self, action_dim, n_frames=4):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(n_frames, 32, kernel_size=8, stride=4),
            nn.ReLU(),
            nn.Conv2d(32, 64, kernel_size=4, stride=2),
            nn.ReLU(),
            nn.Conv2d(64, 64, kernel_size=3, stride=1),
            nn.ReLU()
        )
        # For 84x84 input: output is 64 x 7 x 7 = 3136
        self.fc = nn.Sequential(
            nn.Flatten(),
            nn.Linear(3136, 512),
            nn.ReLU(),
            nn.Linear(512, action_dim)
        )
    
    def forward(self, x):
        return self.fc(self.conv(x / 255.0))

# Frame preprocessing
def preprocess_frame(frame):
    # Convert to grayscale, resize to 84x84
    transform = T.Compose([
        T.ToPILImage(),
        T.Grayscale(),
        T.Resize((84, 84)),
        T.ToTensor()
    ])
    return transform(frame)

# Frame stacking (give model temporal information)
class FrameStack:
    def __init__(self, n_frames=4):
        self.n = n_frames
        self.frames = deque(maxlen=n_frames)
    
    def reset(self, frame):
        for _ in range(self.n):
            self.frames.append(preprocess_frame(frame))
        return torch.cat(list(self.frames), dim=0)
    
    def step(self, frame):
        self.frames.append(preprocess_frame(frame))
        return torch.cat(list(self.frames), dim=0)
```

---

## Deliverables

- [ ] CartPole solved (avg reward > 475 over 100 episodes)
- [ ] LunarLander solved (avg reward > 200 over 100 episodes)
- [ ] Training curve plots for both environments
- [ ] Ablation study: show the effect of each DQN improvement (Double, Dueling, Replay)
- [ ] Q-value visualization for CartPole
- [ ] Analysis: How many environment steps (total) to solve each environment?
- [ ] Report: What happened when you tried too-large or too-small learning rates?
- [ ] MountainCar with reward shaping (optional)
- [ ] Atari DQN (optional, requires GPU)

## Questions to Answer

1. Why does removing experience replay destabilize training? Show this with an ablation.
2. What does "solving" an RL environment actually mean? Why is the 100-episode average used?
3. The epsilon schedule is critical. What happens with: epsilon stays at 1.0? Epsilon decays too fast to 0.01?
4. Why do we need a separate target network? Implement a version without it and show the instability.
5. LunarLander is much harder than CartPole despite similar complexity. Why? What is fundamentally different about the reward signal?
