# ZQL-08 Step 1 Notes — Egger (Quantum Finance) + Hillier & Lieberman (OR) + Sutton & Barto (RL)

---

# A. Egger et al. — *Quantum Computing for Finance* (IEEE Trans. Quantum Engg., 2020)

## Finance Problems as QUBO

### Portfolio Optimisation (Markowitz)
- **Goal**: select assets to maximise return − λ·risk.
- Classical form: max μᵀx − λxᵀΣx s.t. Σxᵢ = 1, xᵢ ∈ {0,1} (binary portfolio).
- QUBO: add penalty term A(Σxᵢ − B)² for budget constraint B.
- **[SDK]** H_C = −Σᵢ μᵢ Zᵢ + λΣᵢ<ⱼ Σᵢⱼ ZᵢZⱼ + A(Σᵢ Zᵢ − B)².

### Risk Analysis (Monte Carlo)
- Quantum speedup: quantum amplitude estimation (QAE) gives O(1/ε) vs classical O(1/ε²) for Monte Carlo.
- **[Context]** This speedup is real (quadratic), uses QPE internally.

### Credit Risk Scoring
- Map credit portfolio exposure to Ising model.
- Quantum advantage: exponentially many correlation patterns encoded in superposition.

## VQE for Portfolio as Finance Example
```python
# 4-asset portfolio optimisation as VQE/QAOA
ASSETS = ["AAPL", "GOOGL", "MSFT", "TSLA"]
EXPECTED_RETURNS = [0.05, 0.03, 0.04, 0.08]   # μᵢ
COVARIANCE = [                                    # Σᵢⱼ (simplified)
    [0.04, 0.01, 0.01, 0.02],
    [0.01, 0.02, 0.005, 0.01],
    [0.01, 0.005, 0.02, 0.01],
    [0.02, 0.01, 0.01, 0.06],
]
BUDGET = 2  # Select exactly 2 assets
RISK_FACTOR = 2.0   # λ

# QUBO penalty: A·(Σxᵢ − B)² = A·(ΣΣ xᵢxⱼ − 2BΣxᵢ + B²)
# → Q[i,j] += 2A for i≠j; Q[i,i] += A(1 − 2B)
```

---

# B. Hillier & Lieberman — *Introduction to Operations Research* (11th ed.)

## Classical OR → Quantum Bridge

### Linear Programming (Ch. 3)
- LP: minimise cᵀx s.t. Ax=b, x≥0. Solvable in polynomial time (Interior Point, Simplex).
- **[Quantum]** Quantum Interior Point (HHL-based): speedup if A is sparse and well-conditioned. Conditional on QRAM.

### Integer Linear Programming (ILP) (Ch. 12)
- Add integrality constraint x ∈ {0,1}^n → NP-hard.
- **[Quantum]** ILP → QUBO → Ising → QAOA. Key ZQL-08 pipeline.

### Network Flow (Ch. 9)
- Min-cost flow, max-flow, shortest path.
- **[Quantum]** Grover search gives quadratic speedup for unstructured graph search. Not clear for structured flow problems.

### Scheduling (Ch. 15)
- Job-shop, machine scheduling: NP-hard variants.
- **[Quantum]** Scheduling as QUBO: x_{j,t} = 1 if job j starts at time t.
  - n jobs × T time slots → nT binary variables.
  - Penalties for conflicts, precedence constraints.

## OR Problem → QUBO Template
```python
def or_to_qubo_template(n_vars: int) -> dict:
    """
    Template for OR → QUBO conversion.
    Returns Q matrix as {(i,j): coefficient} dict.
    Convention: minimise x^T Q x, xᵢ ∈ {0,1}.
    """
    Q = {}
    # Objective terms → Q[i,i]  (linear in x)
    # Quadratic terms → Q[i,j] with i<j (bilinear xᵢxⱼ)
    # Constraint penalty: A*(Σxᵢ - B)² expands to:
    #   diagonal: A*(1 - 2B) for each i
    #   off-diagonal: 2A for each (i,j) pair
    return Q
```

---

# C. Sutton & Barto — *Reinforcement Learning: An Introduction* (2nd ed., MIT Press)

**Underpins adaptive shot allocation strategies in Zuntenium's QAOA backend.**

## Multi-Armed Bandit (MAB) (Ch. 2)
- n arms (= n parameter settings or backend options).
- At each step: choose arm → receive reward → update estimate.
- Goal: maximise cumulative reward (= minimise circuit evaluation waste).

## ε-greedy Strategy
- With probability ε: explore (try a random arm).
- With probability 1−ε: exploit (try best known arm).
- **[SDK]** `ShotAllocator` in zuntenium uses MAB-style: allocate more shots to Pauli terms with highest variance (exploration) and fewer to near-converged terms (exploitation).

## UCB1 (Upper Confidence Bound) (Ch. 2.7)
```
arm_score[k] = estimate[k] + C · sqrt(log(t) / N[k])
```
where t = total pulls, N[k] = pulls for arm k, C = exploration constant.
- **[SDK]** UCB-style adaptive shot allocation: over-estimate variance of under-sampled Pauli terms to preferentially re-measure them.

## Policy Gradient Methods (Ch. 13)
- Directly optimise the policy π(θ) by gradient ascent on expected reward.
- **[QML analogy]** VQE/QAOA training is exactly policy gradient: θ → E(θ), optimise θ to reduce energy.
- Parameter-shift rule = the quantum analogue of REINFORCE policy gradient.

## Adaptive Shot Allocation for QAOA
```python
def adaptive_shot_allocator(pauli_terms: list, total_shots: int,
                              prev_variances: dict) -> dict:
    """
    Allocate shots to Pauli terms proportionally to their variance.
    Higher-variance terms need more shots to reduce statistical noise.
    """
    if not prev_variances:
        # Uniform initialisation (exploration)
        return {p: total_shots // len(pauli_terms) for _, p in pauli_terms}
    
    total_var = sum(prev_variances.values()) + 1e-10
    shots = {}
    for _, pauli_str in pauli_terms:
        fraction = prev_variances.get(pauli_str, 1.0) / total_var
        shots[pauli_str] = max(64, int(fraction * total_shots))
    return shots
```
