# 10. Modern Deep RL Algorithm

Num of pages: 1
Status: Done
Type: theory

| **Algorithm** | **Category** | **Core Idea & Key Innovations** | **Strengths** | **Weaknesses** | **Applications** |
| --- | --- | --- | --- | --- | --- |
| **DQN** (Deep Q-Network) | Value-Based | Combines Q-learning with Deep Neural Networks. Uses **experience replay** to break correlations and **target networks** to stabilize updates. | Simple and effective in discrete action spaces. | Struggles with continuous actions; sample inefficient. | Atari, simple games. |
| **A2C**(Advantage Actor-Critic) | On-Policy Actor-Critic | Uses both an actor (policy) and critic (value), with advantage estimates to reduce variance. Uses synchronous updates across parallel environments. | Lower variance than vanilla policy gradients; supports parallelism. | Sample inefficient (requires fresh samples); sensitive to hyperparameters. | Atari, basic robotics. |
| **TRPO** (Trust Region Policy Optimization) | On-Policy Actor-Critic | Optimizes policy with guaranteed monotonic improvement using a **trust region constraint** (using KL-divergence constraint). |  | Computationally expensive (second-order optimization); complex implementation. | Simulated robotics. |
| **PPO**(Proximal Policy Optimization) | On-Policy Actor-Critic | Trust-region inspired but uses a **clipped surrogate objective** to limit updates, avoiding complex second-order optimization. | Easy to implement and tune; stable training. | Still sample inefficient due to on-policy nature; sensitive to batch size. | Robotics, games. |
| **DDPG** (Deep Deterministic Policy Gradient) | Off-Policy Actor-Critic | Extends DQN to continuous spaces using **deterministic policies**. Uses target networks and experience replay. | Handles continuous control; sample efficient. | Prone to instability and overestimation bias; sensitive to hyperparameters. | Robotic manipulation, MuJoCo. |
| **TD3** (Twin Delayed DDPG) | Off-Policy Actor-Critic | Improves DDPG by addressing overestimation. Uses **Twin Q-networks** (min of two), **delayed policy updates** (updates the actor less frequently than the critics), and **target policy smoothing** (by adding small noise). | Significantly more stable than DDPG; good sample efficiency. | Still relatively complex; lower exploration capabilities than SAC. | Continuous control, robotic locomotion. |
| **SAC** (Soft Actor-Critic) | Off-Policy Actor-Critic | Maximizes **reward** and **entropy** to encourage exploration. Uses a stochastic policy and automatic temperature tuning. | Highly sample efficient, stable, and explores well. | Complex implementation; sensitive to entropy settings. | Robotics, locomotion. |
| **DreamerV3** | Model-Based RL | Learns a **world model** and plans by imagining future trajectories in a latent space. | Highly sample efficient; strong performance in continuous control. | Complex architecture; sensitive to the quality of the learned model. | Visual control tasks, low-data robotics. |
| **MuZero** | Model-Based RL | Learns a latent model and performs **MCTS** without knowing the environment dynamics. | SOTA in complex planning; no predefined model needed. | Extremely complex and computationally expensive. | Chess, Go, Atari, strategic planning. |
| **Decision Transformer** | Offline / Sequence-Based RL | Treats RL as a sequence prediction problem. A Transformer predicts actions conditioned on the desired future return. | Works well with large offline datasets. | No explicit policy optimization; performance limited by dataset quality; no online learning. | Offline RL in healthcare, recommender systems. |

---