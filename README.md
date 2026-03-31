# Search & Reinforcement Learning Algorithms in Sokoban

An experimental study comparing classical search algorithms and reinforcement learning algorithms applied to the Sokoban puzzle. This project spans two papers: the first implements five classical search strategies (BFS, DFS, UCS, Greedy Best-First Search, and A*); the second reformulates Sokoban as a Markov Decision Process and investigates three RL algorithms (Q-Learning, SARSA, and DQN).

---

## Table of Contents
- [About Sokoban](#about-sokoban)
- [Project Structure](#project-structure)
- [Part 1: Classical Search Algorithms](#part-1-classical-search-algorithms)
  - [Algorithms Implemented](#algorithms-implemented)
  - [How It Works](#how-it-works)
  - [Algorithm Flow Diagrams](#algorithm-flow-diagrams)
  - [Experimental Results (Search)](#experimental-results-search)
- [Part 2: Reinforcement Learning Algorithms](#part-2-reinforcement-learning-algorithms)
  - [Problem Formulation as MDP](#problem-formulation-as-mdp)
  - [RL Algorithms Implemented](#rl-algorithms-implemented)
  - [Experimental Setup](#experimental-setup)
  - [Experimental Results (RL)](#experimental-results-rl)
- [Visualizations](#visualizations)
- [How to Run](#how-to-run)
- [References](#references)

---

## About Sokoban

Sokoban (Japanese for "warehouse keeper") is a classic combinatorial puzzle. A player moves on a grid and must push boxes onto designated goal locations. The player can move in four cardinal directions (Up, Down, Left, Right) and can push — but never pull — one box at a time. The puzzle is solved when every box rests on a goal square.

Sokoban is known to be PSPACE-complete, making it an excellent benchmark for evaluating search and learning algorithms.

| Symbol | Meaning |
|--------|---------|
| `#` | Wall |
| ` ` | Floor |
| `@` | Player |
| `$` | Box |
| `.` | Goal |
| `+` | Player on Goal |
| `*` | Box on Goal |

---

## Project Structure

```
Search-Algorithms-in-Sokoban/
|-- a-survey-on-classical-search-algorithms.ipynb   # Part 1: classical search
|-- comparative-study-of-search-and-rl-in-sokoban/  # Part 2: RL algorithms
|-- README.md
|-- docs/
|   |-- paper1.pdf                                  # Classical search paper
|   |-- springer_nature.pdf                         # RL paper (Springer Nature)
|-- images/
|   |-- agentworkingA.png                           # A* agent trace visualization
|   |-- mainfig.png                                 # Main architecture figure
|   |-- nodesExpanded.png                           # Nodes expanded bar chart
|   |-- qual_vs_search.png                          # Solution quality vs search efficiency
|   |-- radarchart.png                              # Multi-dimensional radar chart (search)
|   |-- rl_radar.png                                # Multi-metric radar chart (RL)
|   |-- rl_framework.png                            # RL framework overview
```

---

## Part 1: Classical Search Algorithms

### Test Level Used (7×7, 2 boxes, 2 goals)

```
#######
#  .  #
#     #
# $@$ #
#     #
#  .  #
#######
```

### Algorithms Implemented

| Algorithm | Strategy | Optimal | Complete | Data Structure |
|-----------|----------|---------|----------|----------------|
| BFS | Uninformed | Yes | Yes | Queue (FIFO) |
| DFS (depth-limited) | Uninformed | No | No* | Stack (LIFO) |
| UCS | Uninformed | Yes | Yes | Priority Queue |
| Greedy Best-First | Informed | No | No | Priority Queue |
| A* | Informed | Yes | Yes | Priority Queue |

\* DFS completeness depends on the depth limit.

**Heuristic (for informed searches):** Sum of minimum Manhattan distances from each box to its nearest goal position. This heuristic is admissible (never overestimates the true cost) for Sokoban.

### How It Works

The Sokoban puzzle is modeled as a formal search problem:

- **State representation:** Each state is a tuple `(player_position, frozenset_of_box_positions)`. This is hashable, allowing efficient duplicate detection via sets.
- **Transition model:** For each of the four directions, the player attempts to move. If the target cell contains a box, and the cell beyond the box is free, the box is pushed. Moves into walls or into boxes that cannot be pushed are pruned.
- **Goal test:** All boxes rest on goal positions.

### Algorithm Flow Diagrams

#### BFS (Breadth-First Search)
```
flowchart TD
    S[Start: initial state] --> Q[Initialize FIFO queue with start state]
    Q --> D{Queue empty?}
    D -- Yes --> NF[No solution found]
    D -- No --> POP[Dequeue front state]
    POP --> EXP[Expand: generate all successors]
    EXP --> CHK{For each successor: visited?}
    CHK -- Yes --> SKIP[Skip]
    CHK -- No --> GT{Goal test}
    GT -- Yes --> SOL[Return solution path]
    GT -- No --> ADD[Mark visited, enqueue successor]
    ADD --> D
    SKIP --> D
```

#### DFS (Depth-First Search with Depth Limit)
```
flowchart TD
    S[Start: initial state] --> STK[Initialize stack with start state]
    STK --> D{Stack empty?}
    D -- Yes --> NF[No solution found]
    D -- No --> POP[Pop top state]
    POP --> DL{Depth >= limit?}
    DL -- Yes --> D
    DL -- No --> EXP[Expand: generate all successors]
    EXP --> CHK{For each successor: visited?}
    CHK -- Yes --> SKIP[Skip]
    CHK -- No --> GT{Goal test}
    GT -- Yes --> SOL[Return solution path]
    GT -- No --> ADD[Mark visited, push to stack]
    ADD --> D
    SKIP --> D
```

#### UCS (Uniform Cost Search)
```
flowchart TD
    S[Start: initial state, cost = 0] --> PQ[Initialize priority queue ordered by path cost g]
    PQ --> D{Priority queue empty?}
    D -- Yes --> NF[No solution found]
    D -- No --> POP[Pop state with lowest g cost]
    POP --> VIS{Already visited?}
    VIS -- Yes --> D
    VIS -- No --> MARK[Mark visited]
    MARK --> GT{Goal test}
    GT -- Yes --> SOL[Return solution path]
    GT -- No --> EXP[Expand: generate all successors]
    EXP --> PUSH[Push each unvisited successor with cost g + step_cost]
    PUSH --> D
```

#### Greedy Best-First Search
```
flowchart TD
    S[Start: initial state] --> PQ[Initialize priority queue ordered by h(state)]
    PQ --> D{Priority queue empty?}
    D -- Yes --> NF[No solution found]
    D -- No --> POP[Pop state with lowest heuristic h]
    POP --> VIS{Already visited?}
    VIS -- Yes --> D
    VIS -- No --> MARK[Mark visited]
    MARK --> GT{Goal test}
    GT -- Yes --> SOL[Return solution path]
    GT -- No --> EXP[Expand: generate all successors]
    EXP --> H[Compute h(successor) = Manhattan distance]
    H --> PUSH[Push each unvisited successor]
    PUSH --> D
```

#### A* Search
```
flowchart TD
    S[Start: initial state] --> PQ[Initialize priority queue ordered by f = g + h]
    PQ --> D{Priority queue empty?}
    D -- Yes --> NF[No solution found]
    D -- No --> POP[Pop state with lowest f = g + h]
    POP --> VIS{Already visited?}
    VIS -- Yes --> D
    VIS -- No --> MARK[Mark visited]
    MARK --> GT{Goal test}
    GT -- Yes --> SOL[Return solution path and cost]
    GT -- No --> EXP[Expand: generate all successors]
    EXP --> CALC[For each successor: g' = g + cost, h' = Manhattan, f' = g' + h']
    CALC --> PUSH[Push each unvisited successor with f']
    PUSH --> D
```

#### Overall System Pipeline
```
flowchart LR
    A[Input: Sokoban Level Grid] --> B[State Extraction]
    B --> C[Search Algorithm Selection]
    C --> D1[BFS]
    C --> D2[DFS]
    C --> D3[UCS]
    C --> D4[Greedy BFS]
    C --> D5[A*]
    D1 --> E[Solution Path + Metrics]
    D2 --> E
    D3 --> E
    D4 --> E
    D5 --> E
    E --> F[Performance Comparison and Visualization]
```

### Experimental Results (Search)

All algorithms were run on the same 7×7 level with 2 boxes and 2 goals.

| Algorithm | Path Length | Nodes Expanded | Time (s) | Optimal | Complete |
|-----------|-------------|----------------|----------|---------|----------|
| BFS | 15 | 4309 | 0.094651 | Yes | Yes |
| DFS | 199 | 4927 | 0.012571 | No | No* |
| UCS | 15 | 5223 | 0.020785 | Yes | Yes |
| Greedy BFS | 15 | 36 | 0.000198 | No | No |
| A* | 15 | 2465 | 0.013979 | Yes | Yes |

\* DFS completeness depends on depth limit (set to 200 in this experiment).

**Key Observations:**
- BFS, UCS, and A* all found the optimal solution of 15 moves.
- DFS found a valid but highly suboptimal solution of 199 moves, as expected from a depth-first strategy.
- Greedy BFS expanded only 36 nodes (fastest), but does not guarantee optimality.
- A* strikes the best balance: optimal path while expanding significantly fewer nodes (2465) than BFS (4309) or UCS (5223).
- UCS expanded the most nodes because uniform step costs make it behave similarly to BFS but with added priority queue overhead.

---

## Part 2: Reinforcement Learning Algorithms

This paper reformulates Sokoban as a Markov Decision Process (MDP) and investigates RL as a scalable alternative to explicit state-space search. The motivation is that A*'s dependence on explicit frontier expansion and memory-intensive data structures limits its applicability to larger instances.

> Code repository: https://github.com/SaurabMishra12/comparative-study-of-search-and-rl-in-sokoban

### Problem Formulation as MDP

**Environment:** 8×8 grid with 3 movable boxes, 3 goal locations, and 28 static walls — leaving 36 valid navigable cells. This significantly increases combinatorial complexity compared to the 7×7 search benchmark.

**State Space:** Each state is encoded as a tuple of integer coordinates:

```
s = (player_r, player_c, box1_r, box1_c, box2_r, box2_c, box3_r, box3_c)
```

For tabular methods, this tuple is used directly as a discrete state identifier. For DQN, goal coordinates are also included, yielding a 14-dimensional input vector normalized to [0, 1] to stabilize training.

**Action Space:** `A = {UP, DOWN, LEFT, RIGHT}`. Each action deterministically updates the state subject to Sokoban constraints: movements into walls are disallowed, and box pushes are valid only if the destination cell is unoccupied.

**Reward Structure:**

| Event | Reward |
|-------|--------|
| All boxes on goals (terminal state) | +100 |
| Each timestep (step penalty) | -1 |
| Invalid action (wall collision or infeasible box push) | -5 |
| Action reduces aggregate Manhattan distance to goals | +2 |

The shaping reward (+2) is intentionally kept small to guide exploration without biasing agents toward locally optimal but globally suboptimal behavior.

### RL Algorithms Implemented

#### Q-Learning (Off-Policy Tabular TD Control)

Q-Learning directly approximates the optimal action-value function, independent of the behavior policy. Update rule:

```
Q(s, a) ← Q(s, a) + α [ r + γ · max_a' Q(s', a') − Q(s, a) ]
```

By bootstrapping from the maximum estimated future value, Q-Learning achieves rapid convergence but may exhibit high variance during early training due to overestimation bias. Uses ε-greedy exploration with fast decay (ε: 1.0 → 0.01), prioritizing convergence speed.

#### SARSA (On-Policy Tabular TD Control)

SARSA evaluates and improves the policy actually being executed. Update rule:

```
Q(s, a) ← Q(s, a) + α [ r + γ · Q(s', a') − Q(s, a) ]
```

where `a'` is the next action sampled from the current policy. Because SARSA incorporates exploratory actions into its updates, it produces more conservative value estimates — particularly beneficial in Sokoban where exploratory moves can cause irreversible deadlocks. Uses a slower ε-decay (ε: 1.0 → 0.05) for smoother, safer convergence.

#### Deep Q-Network (DQN)

To address scalability limitations of tabular methods, DQN approximates the action-value function with a neural network, enabling generalization across similar states without explicitly storing all state-action pairs.

**Network architecture:**
```
Input(14) → FC(64) → ReLU → FC(64) → ReLU → FC(32) → ReLU → FC(4)
```

**TD loss minimized during training:**
```
L(θ) = [ Q_θ(s, a) − ( r + γ · max_a' Q_θ̄(s', a') ) ]²
```

Two stabilization mechanisms are used:
- **Experience Replay:** Replay buffer of capacity 10,000; transitions sampled uniformly to break temporal correlations and improve sample efficiency.
- **Target Network:** Parameters (θ̄) synchronized every 300 steps to provide stable learning targets.

### Experimental Setup

All algorithms are evaluated within a controlled framework. Each method is trained over multiple episodes with a maximum horizon of 300 steps per episode. Episodes terminate upon reaching the goal state (all boxes placed correctly) or when the step limit is exceeded.

Three primary evaluation criteria are used:
1. **Convergence speed** — number of episodes required to achieve stable performance
2. **Solution quality** — cumulative reward and trajectory efficiency
3. **Memory efficiency** — storage requirements of tabular vs. function approximation methods

| Hyperparameter | Q-Learning | SARSA | DQN |
|----------------|------------|-------|-----|
| Learning Rate | 0.2 | 0.1 | 5×10⁻⁴ |
| Discount Factor (γ) | 0.99 | 0.99 | 0.99 |
| ε Decay Rate | 0.9995 | 0.9998 | 0.998 |
| Final ε | 0.01 | 0.05 | 0.02 |
| Training Episodes | 10,000 | 10,000 | 3,000 |

DQN requires fewer training episodes due to its ability to generalize across states via function approximation.

### Experimental Results (RL)

#### Convergence Speed & Solution Quality

- **Q-Learning** converges fastest by aggressively propagating reward signals via greedy bootstrapping. Achieves tight reward distribution and shorter solution trajectories, but exhibits high variance during early training.
- **SARSA** converges more slowly due to its on-policy nature but produces more stable, conservative policies that avoid irreversible deadlocks — a key advantage in Sokoban.
- **DQN** achieves moderate convergence speed through state generalization, reducing the need for exhaustive state visitation. However, it exhibits a bimodal performance profile: either converging to a near-optimal solution or failing to solve the task entirely, reflecting inherent instability from function approximation in constraint-heavy environments.

#### Memory Efficiency

- **Tabular methods (Q-Learning & SARSA):** Q-tables scale linearly with visited state-action pairs. Both expand to over 50,000 entries in the 8×8 environment (SARSA slightly more due to prolonged exploration). This growth becomes prohibitive as the state space increases.
- **DQN:** Maintains a fixed memory footprint (~5,000 network parameters), independent of the number of visited states. This provides a significant scalability advantage, at the cost of increased sample complexity and sensitivity to hyperparameters.

#### Impact of Reward Shaping

The +2 shaping reward significantly accelerates learning by providing intermediate feedback aligned with the task objective. Increasing the shaping magnitude to +5 causes a collapse in behavioral diversity — all algorithms converge to similar policies, masking the intrinsic algorithmic differences. The calibrated +2 value preserves meaningful distinctions in learning dynamics, highlighting Q-Learning's aggressive optimization, SARSA's risk-aware behavior, and DQN's generalization capability.

---

## Visualizations

| Image | Description |
|-------|-------------|
| `nodesExpanded.png` | Nodes expanded per search algorithm |
| `qual_vs_search.png` | Solution quality vs. search efficiency |
| `radarchart.png` | Multi-dimensional radar chart (search algorithms) |
| `agentworkingA.png` | A* agent working trace |
| `rl_framework.png` | RL framework overview (MDP → agent → evaluation) |
| `rl_radar.png` | Multi-metric radar: success rate, solution quality, reward, training speed, sample efficiency (RL) |

---

## How to Run

**Requirements:** Python 3.7+, Jupyter Notebook or JupyterLab, `numpy`, `pandas`, `matplotlib`, `torch` (for DQN)

```bash
# Clone the repository
git clone https://github.com/SaurabMishra12/Search-Algorithms-in-Sokoban.git
cd Search-Algorithms-in-Sokoban

# Install dependencies
pip install numpy pandas matplotlib torch
```

**Part 1 — Classical Search:**
```bash
jupyter notebook a-survey-on-classical-search-algorithms.ipynb
```
Execute all cells sequentially. The notebook will parse the Sokoban level, run all five search algorithms, print the performance comparison table, and generate visualization charts.

**Part 2 — Reinforcement Learning:**
```bash
# See: https://github.com/SaurabMishra12/comparative-study-of-search-and-rl-in-sokoban
```

---

## References

1. Culberson, J. (1997). Sokoban is PSPACE-complete. *Technical Report TR 97-02*, Department of Computing Science, University of Alberta.
2. Russell, S., Norvig, P. *Artificial Intelligence: A Modern Approach*, 3rd ed. Prentice Hall, 2010.
3. Sutton, R.S., Barto, A.G. *Reinforcement Learning: An Introduction*, 2nd ed. MIT Press, 2018.
4. Mnih, V., et al. Human-level control through deep reinforcement learning. *Nature* 518(7540), 529–533 (2015). https://doi.org/10.1038/nature14236
5. Local paper: `docs/paper1.pdf`
6. Local paper: `docs/springer_nature.pdf`
