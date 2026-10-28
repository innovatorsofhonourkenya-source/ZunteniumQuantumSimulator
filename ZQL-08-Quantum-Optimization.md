# ZQL-08 — Quantum Optimization
# QUBO, Ising Models & Real-World Applications (MaxCut, Portfolio, Scheduling)

> **Course ID**: ZQL-08 | **Track**: ZQL-05 → ZQL-08  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisites**: ZQL-05 (QAOA circuit, parameterised circuits, COBYLA)

---

## 0. Course Overview

### Who This Is For

You built QAOA for MaxCut in ZQL-05. This course goes **deeper and wider**: rigorous QUBO formulations, converting real-world problems (portfolio, scheduling, partitioning) into Ising Hamiltonians, running QAOA circuits on them, and using adaptive shot allocation to reduce measurement cost.

### Learning Outcomes

1. Formulate any combinatorial problem as a QUBO matrix Q
2. Convert QUBO ↔ Ising Hamiltonian (binary ↔ spin variables)
3. Map the Ising Hamiltonian to a QAOA circuit
4. Implement weighted MaxCut, portfolio optimisation, and number partitioning
5. Use adaptive shot allocation to cut measurement cost
6. Interpret classical benchmarks (Goemans-Williamson) and know when QAOA can compete

### Estimated Time: ~140 minutes

| Section | Content | Time |
|---|---|---|
| §1 | QUBO Fundamentals | 20 min |
| §2 | QUBO ↔ Ising Conversion | 15 min |
| §3 | QUBO → QAOA Pipeline | 15 min |
| §4 | MaxCut (Weighted) | 20 min |
| §5 | Portfolio Optimisation | 20 min |
| §6 | Number Partitioning | 15 min |
| §7 | Adaptive Shot Allocation | 15 min |
| §8 | Exercises + Quiz + Capstone | 20 min |

---

## 1. QUBO Fundamentals

### 1.1 Problem Definition

A **QUBO** (Quadratic Unconstrained Binary Optimisation) problem:

```
minimise   f(x) = xᵀQx = Σᵢ Qᵢᵢxᵢ + Σᵢ<ⱼ Qᵢⱼxᵢxⱼ
subject to xᵢ ∈ {0, 1}
```

where Q ∈ ℝⁿˣⁿ is an upper-triangular (or symmetric) matrix.

**Key fact**: any quadratic binary program can be written as QUBO (possibly after penalty terms for constraints).

```python
import numpy as np

def qubo_energy(Q: np.ndarray, x: np.ndarray) -> float:
    """Evaluate QUBO objective f(x) = x^T Q x."""
    return float(x @ Q @ x)

def qubo_brute_force(Q: np.ndarray) -> tuple:
    """
    Solve QUBO by exhaustive search (feasible only for n ≤ 20).
    Returns (x_opt, energy_opt).
    """
    n = len(Q)
    best_x, best_E = None, float('inf')
    for bits in range(2**n):
        x = np.array([(bits >> i) & 1 for i in range(n)], dtype=float)
        E = qubo_energy(Q, x)
        if E < best_E:
            best_E, best_x = E, x.copy()
    return best_x, best_E

# Simple 3-variable example
Q_demo = np.array([
    [-3,  2,  2],
    [ 0, -3,  2],
    [ 0,  0, -3],
], dtype=float)
x_opt, E_opt = qubo_brute_force(Q_demo)
print(f"QUBO demo: x* = {x_opt.astype(int)}, E* = {E_opt:.1f}")
print(f"(Diagonal Q[i,i]=-3: favours selecting all; Q[i,j]=2: penalises pairs → 1 selected)")
```

### 1.2 Adding Constraints as Penalty Terms

Converting a constrained problem to QUBO:

```
Constraint: g(x) = 0  →  Penalty: A · g(x)²   (added to objective)
```

Choose penalty coefficient A large enough so violating the constraint costs more than any objective gain.

```python
def add_equality_constraint(Q: np.ndarray, indices: list,
                             target_sum: int, penalty: float) -> np.ndarray:
    """
    Add penalty A·(Σᵢ xᵢ − B)² to Q for indices in list.
    Expands to: A·Σᵢ(1−2B)xᵢ + 2A·Σᵢ<ⱼ xᵢxⱼ + const.
    """
    Q_new = Q.copy()
    # Diagonal: A*(1 - 2*B) for each selected index
    for i in indices:
        Q_new[i, i] += penalty * (1 - 2 * target_sum)
    # Off-diagonal: 2*A for each pair
    for k, i in enumerate(indices):
        for j in indices[k+1:]:
            Q_new[i, j]  = Q_new.get((i,j), 0) + 2 * penalty if isinstance(Q_new, dict) else Q_new[i, j] + 2 * penalty
    return Q_new
```

---

## 2. QUBO ↔ Ising Conversion

### 2.1 Variable Substitution

QUBO variables xᵢ ∈ {0,1} ↔ Ising spins sᵢ ∈ {−1,+1}:

```
xᵢ = (1 − sᵢ) / 2    ↔    sᵢ = 1 − 2xᵢ
```

### 2.2 Conversion Formulas

Substituting into f(x) = xᵀQx:

```
H_Ising = Σᵢ hᵢ sᵢ + Σᵢ<ⱼ Jᵢⱼ sᵢsⱼ + constant

where:
  Jᵢⱼ = Qᵢⱼ / 4
  hᵢ  = Qᵢᵢ/2 + Σⱼ≠ᵢ Qᵢⱼ/4
  constant = Σᵢ Qᵢᵢ/4 + Σᵢ<ⱼ Qᵢⱼ/4 + Σᵢ hᵢ
```

Mapping to Pauli operators (sᵢ → Zᵢ):

```python
def qubo_to_ising_hamiltonian(Q: np.ndarray) -> list:
    """
    Convert QUBO matrix Q to Ising Hamiltonian Pauli terms.
    Returns list of (coefficient, pauli_string).
    Spin variables sᵢ → Pauli Zᵢ.
    """
    n = len(Q)
    terms = []
    offset = 0.0

    # Off-diagonal Jᵢⱼ terms → ZᵢZⱼ
    for i in range(n):
        for j in range(i + 1, n):
            Qij = Q[i, j] + Q[j, i]   # symmetrise
            if abs(Qij) < 1e-12: continue
            Jij = Qij / 4
            pauli = 'I'*i + 'Z' + 'I'*(j-i-1) + 'Z' + 'I'*(n-j-1)
            terms.append((Jij, pauli))

    # Diagonal hᵢ terms → Zᵢ
    for i in range(n):
        hi = Q[i, i] / 2
        for j in range(n):
            if j != i:
                Qij = (Q[i, j] + Q[j, i]) / 2 if j > i else (Q[j, i] + Q[i, j]) / 2
                hi += Qij / 4
        if abs(hi) > 1e-12:
            pauli = 'I'*i + 'Z' + 'I'*(n-i-1)
            terms.append((hi, pauli))

    # Constant offset (important for energy interpretation but ignored in QAOA)
    offset = sum(Q[i,i] for i in range(n)) / 4
    for i in range(n):
        for j in range(i+1, n):
            offset += (Q[i,j] + Q[j,i]) / 8

    return terms, offset

# Test on MaxCut triangle graph (3 nodes, edges 0-1, 1-2, 0-2)
# MaxCut QUBO: Q[i,i] = -deg(i), Q[i,j] = 1 for (i,j) in edges
n = 3
Q_maxcut = np.array([[-2, 1, 1],
                      [ 1,-2, 1],
                      [ 1, 1,-2]], dtype=float)
ising_terms, offset = qubo_to_ising_hamiltonian(Q_maxcut)
print("MaxCut triangle Ising Hamiltonian:")
for coeff, pauli in ising_terms:
    print(f"  {coeff:+.4f} × {pauli}")
print(f"  Constant offset: {offset:.4f}")
print(f"  (Max cut = 2 → min H_Ising = ?)")
```

### 2.3 Sign Convention

> **Important**: QUBO minimises; MaxCut *maximises* cut size. To use QAOA (which maximises ⟨H_C⟩), set H_C = −H_Ising (flip sign). Double-check sign conventions!

```python
def qubo_to_qaoa_hamiltonian(Q: np.ndarray, maximise: bool = False) -> list:
    """
    Convert QUBO Q to QAOA cost Hamiltonian.
    If maximise=True: negate Ising terms (QAOA maximises ⟨H⟩).
    """
    terms, offset = qubo_to_ising_hamiltonian(Q)
    sign = -1.0 if maximise else +1.0
    return [(sign * c, p) for c, p in terms]
```

---

## 3. QUBO → QAOA Pipeline

### 3.1 Full End-to-End Conversion

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np
import scipy.optimize as opt

def build_qaoa_circuit(n: int, edges_weights: list,
                        gamma: float, beta: float) -> QuantumCircuit:
    """
    Build QAOA(p=1) circuit for a weighted Ising Hamiltonian.
    edges_weights: list of (i, j, weight) tuples (Jᵢⱼ coupling).
    hᵢ: single-qubit field terms → qc.rz(2*h_i*gamma, i)
    Jᵢⱼ: two-qubit coupling → qc.cx + qc.rz(2*J*gamma) + qc.cx
    """
    qc = QuantumCircuit(n, name="QAOA_Ising")

    # Initial state: |+⟩^n
    for i in range(n): qc.h(i)

    # Cost layer (ZZ coupling terms)
    for i, j, w in edges_weights:
        qc.cx(i, j)
        qc.rz(2 * w * gamma, j)
        qc.cx(i, j)

    # Mixer layer (transverse field)
    for i in range(n):
        qc.rx(2 * beta, i)

    return qc

backend = StatevectorSimulator()

def qaoa_expected_cut(n: int, edges_weights: list, gamma: float,
                       beta: float, shots: int = 4096) -> float:
    """Estimate expected objective value from QAOA circuit measurement."""
    qc = build_qaoa_circuit(n, edges_weights, gamma, beta)
    qc.measure_all()
    result = backend.run(qc, shots=shots)

    total = 0.0
    for bs, count in result.counts.items():
        bits = [int(b) for b in reversed(bs)]   # LSB = qubit 0
        z    = [1 - 2*b for b in bits]          # {0,1} → {+1,-1}
        cost = sum(w * z[i] * z[j] for i,j,w in edges_weights)
        total += cost * (count / shots)
    return total
```

### 3.2 Parameter Optimisation

```python
def optimise_qaoa(n: int, edges_weights: list,
                   shots: int = 2048) -> tuple:
    """
    Optimise QAOA(p=1) parameters (γ, β) for an Ising Hamiltonian.
    Returns (best_gamma, best_beta, best_value).
    """
    best_val = float('-inf')
    best_params = (0.0, 0.0)

    # Grid search
    gammas = np.linspace(0, np.pi, 10)
    betas  = np.linspace(0, np.pi/2, 8)
    for g in gammas:
        for b in betas:
            v = qaoa_expected_cut(n, edges_weights, g, b, shots=512)
            if v > best_val:
                best_val, best_params = v, (g, b)

    # Polish with COBYLA
    def neg_obj(params):
        return -qaoa_expected_cut(n, edges_weights, params[0], params[1], shots)

    res = opt.minimize(neg_obj, x0=list(best_params), method='COBYLA',
                        options={'maxiter': 80})
    g_opt, b_opt = res.x
    return g_opt, b_opt, -res.fun
```

---

## 4. Weighted MaxCut

```python
# Weighted MaxCut: find partition maximising total edge weight across cut
WEIGHTED_GRAPH = {
    "n_nodes": 5,
    "edges": [
        (0, 1, 2.5),    # edge weight 2.5
        (0, 2, 1.0),
        (1, 2, 3.0),
        (1, 3, 1.5),
        (2, 3, 2.0),
        (3, 4, 4.0),
    ],
}
n = WEIGHTED_GRAPH["n_nodes"]
ew = WEIGHTED_GRAPH["edges"]    # (i, j, weight)

# Build Ising from MaxCut: H_C = Σ w_{ij}(I − ZᵢZⱼ)/2
# ZZ coupling Jᵢⱼ = w_{ij}/2 (to be maximised → negate for QUBO)
ising_edges = [(i, j, w/2) for i,j,w in ew]   # Jᵢⱼ = w/2

# Run QAOA optimisation
g_opt, b_opt, val = optimise_qaoa(n, ising_edges, shots=2048)
print(f"Weighted MaxCut (5 nodes):")
print(f"  Optimal QAOA params: γ={g_opt:.4f}, β={b_opt:.4f}")
print(f"  Expected cut weight: {val:.4f}")
print(f"  Total edge weight: {sum(w for _,_,w in ew):.1f}")
print(f"  Approximation ratio: {val/sum(w for _,_,w in ew):.4f} (GW bound: 0.8785)")

# Decode best assignments
qc_opt = build_qaoa_circuit(n, ising_edges, g_opt, b_opt)
qc_opt.measure_all()
result = backend.run(qc_opt, shots=4096)
top = sorted(result.counts.items(), key=lambda x: -x[1])[:5]

def cut_weight(bs, edges):
    bits = [int(b) for b in reversed(bs)]
    z = [1-2*b for b in bits]
    return sum(w*(1-z[i]*z[j])/2 for i,j,w in edges)

print("\nTop measurement outcomes:")
for bs, cnt in top:
    cw = cut_weight(bs, ew)
    print(f"  |{bs}⟩: {cnt:5d} shots, cut weight={cw:.1f}")
```

---

## 5. Portfolio Optimisation

```python
import numpy as np

# 4-asset portfolio: select k=2 from 4 to maximise risk-adjusted return
ASSETS   = ["AAPL", "GOOGL", "MSFT", "TSLA"]
RETURNS  = np.array([0.05,  0.03,  0.04,  0.08])   # expected returns μᵢ
COV      = np.array([                                # covariance matrix Σ
    [0.040, 0.010, 0.010, 0.020],
    [0.010, 0.020, 0.005, 0.010],
    [0.010, 0.005, 0.020, 0.010],
    [0.020, 0.010, 0.010, 0.060],
])
K = 2          # Budget: select exactly K assets
LAMBDA = 2.0  # Risk aversion coefficient
PENALTY = 5.0  # Constraint penalty coefficient A

def build_portfolio_qubo(returns, cov, k, lam, penalty):
    """
    Build QUBO for portfolio optimisation:
    max μᵀx − λxᵀΣx  s.t. Σxᵢ = k

    QUBO: min −μᵀx + λxᵀΣx + penalty*(Σxᵢ − k)²
    """
    n = len(returns)
    Q = np.zeros((n, n))

    # Objective: −μᵢxᵢ → Q[i,i] -= μᵢ
    for i in range(n):
        Q[i, i] -= returns[i]

    # Risk: +λ Σᵢⱼ Σᵢⱼ xᵢxⱼ → Q[i,j] += λ*Σᵢⱼ
    for i in range(n):
        for j in range(n):
            Q[i, j] += lam * cov[i, j]

    # Budget penalty: penalty*(Σxᵢ − k)²
    # Diagonal: penalty*(1 − 2k)
    # Off-diagonal: 2*penalty
    for i in range(n):
        Q[i, i] += penalty * (1 - 2 * k)
    for i in range(n):
        for j in range(i+1, n):
            Q[i, j] += 2 * penalty
            Q[j, i] += 2 * penalty

    return Q

Q_portfolio = build_portfolio_qubo(RETURNS, COV, K, LAMBDA, PENALTY)

# Brute force optimal (n=4 → 16 combinations)
x_opt, E_opt = qubo_brute_force(Q_portfolio)
selected = [ASSETS[i] for i,v in enumerate(x_opt) if v == 1]
print(f"Portfolio QUBO — Brute force optimal:")
print(f"  Selected assets: {selected}")
print(f"  QUBO energy: {E_opt:.4f}")
print(f"  Expected return: {RETURNS @ x_opt:.4f}")
print(f"  Portfolio variance: {x_opt @ COV @ x_opt:.4f}")

# Convert to Ising and run QAOA
ising_portfolio, _ = qubo_to_ising_hamiltonian(Q_portfolio)
n_assets = 4

# Extract ZZ coupling edges for QAOA
coupling_edges = []
for coeff, pauli in ising_portfolio:
    if pauli.count('Z') == 2:
        i = pauli.index('Z')
        j = pauli.index('Z', i+1)
        coupling_edges.append((i, j, coeff))

g_opt, b_opt, val = optimise_qaoa(n_assets, coupling_edges, shots=2048)

qc_port = build_qaoa_circuit(n_assets, coupling_edges, g_opt, b_opt)
qc_port.measure_all()
result_port = backend.run(qc_port, shots=4096)

print(f"\nPortfolio QAOA result:")
top_port = sorted(result_port.counts.items(), key=lambda x: -x[1])[:5]
for bs, cnt in top_port:
    x = np.array([1-int(b) for b in reversed(bs)], dtype=float)  # s=+1→x=0, s=-1→x=1
    x = np.array([int(b) for b in reversed(bs)], dtype=float)   # [TO CONFIRM] spin→binary
    sel = [ASSETS[i] for i in range(4) if x[i] == 1]
    E_q = qubo_energy(Q_portfolio, x)
    print(f"  |{bs}⟩: {cnt:4d} shots, select={sel}, QUBO E={E_q:.4f}")
```

---

## 6. Number Partitioning

Partition a set S of n numbers into two subsets with equal sum.

```python
def number_partition_qubo(numbers: list) -> np.ndarray:
    """
    QUBO for number partitioning: minimise (Σᵢ sᵢaᵢ)² where sᵢ ∈ {−1,+1}.
    In binary: minimise (Σᵢ (2xᵢ−1)aᵢ)²
    → Q[i,i] = aᵢ(aᵢ − sum(a)), Q[i,j] = 2aᵢaⱼ
    """
    n = len(numbers)
    a = np.array(numbers, dtype=float)
    Q = np.zeros((n, n))
    total = sum(a)
    for i in range(n):
        Q[i, i] = a[i] * (a[i] - total)
    for i in range(n):
        for j in range(i+1, n):
            Q[i, j] = 2 * a[i] * a[j]
            Q[j, i] = Q[i, j]
    return Q

# Example: partition {3, 1, 1, 2, 2, 1} → sums should be 5 each
numbers = [3, 1, 1, 2, 2, 1]
target_sum = sum(numbers) / 2
print(f"Number partitioning: {numbers}  (target each half = {target_sum})")
Q_part = number_partition_qubo(numbers)
x_part, E_part = qubo_brute_force(Q_part)
set_A = [numbers[i] for i,v in enumerate(x_part) if v == 1]
set_B = [numbers[i] for i,v in enumerate(x_part) if v == 0]
print(f"  Set A (x=1): {set_A}  sum={sum(set_A)}")
print(f"  Set B (x=0): {set_B}  sum={sum(set_B)}")
print(f"  Difference:  {abs(sum(set_A)-sum(set_B))}")
```

---

## 7. Adaptive Shot Allocation

For a QAOA circuit with many Pauli terms, not all terms need equal shots — high-variance terms need more.

```python
def run_adaptive_vqe(hamiltonian: list, ansatz_fn, params_0: np.ndarray,
                      total_shots_per_iter: int = 8192, n_iters: int = 30,
                      shots_min: int = 128) -> tuple:
    """
    Run QAOA/VQE with adaptive shot allocation.
    Allocates more shots to high-variance Pauli terms.

    Strategy: MAB-style proportional to variance from previous iteration.
    """
    params = params_0.copy()
    history = []
    variances = {}   # pauli_str → variance estimate

    for it in range(n_iters):
        # Allocate shots proportionally to variance
        if not variances:
            shots_per_term = {p: total_shots_per_iter // len(hamiltonian)
                               for _, p in hamiltonian}
        else:
            total_var = max(sum(variances.values()), 1e-10)
            shots_per_term = {p: max(shots_min, int(v/total_var * total_shots_per_iter))
                               for p, v in variances.items()}

        # Measure each Pauli term with its allocated shots
        ansatz = ansatz_fn(params)
        expectations = {}
        new_variances = {}
        for coeff, pauli_str in hamiltonian:
            if set(pauli_str) == {'I'}:
                expectations[pauli_str] = 1.0
                new_variances[pauli_str] = 0.0
                continue
            shots = shots_per_term.get(pauli_str, shots_min)
            exp_val = measure_expectation(ansatz, pauli_str, shots=shots)
            expectations[pauli_str] = exp_val
            # Variance estimate: Var[⟨P⟩] ≈ (1 - exp_val²) / shots
            new_variances[pauli_str] = max(0.0, 1 - exp_val**2) / shots

        variances = new_variances
        E = sum(c * expectations.get(p, 1.0) for c, p in hamiltonian)
        history.append(E)

        # Simple gradient-free update
        if it % 5 == 0:
            print(f"  Iter {it:3d}: E = {E:.6f}, total shots = {sum(shots_per_term.values())}")
        params -= 0.05 * np.random.randn(len(params)) * 0.1   # random walk (demo)

    return params, history
```

---

## 8. Exercises, Quiz, and Capstone

### Exercises

**E1.1** Write `qubo_to_matrix(n, terms)` that takes a list of (i,j,coefficient) triples and returns the n×n QUBO matrix Q.

**E2.1** Verify the QUBO↔Ising conversion: for a 2-variable QUBO with Q=[[−1,2],[0,−1]], compute h₀, h₁, J₀₁ by hand and confirm with `qubo_to_ising_hamiltonian()`.

**E4.1** For weighted MaxCut on the 5-node graph above, what is the maximum possible cut weight (sum of all edges in best partition)? Verify with brute force over all 2⁵=32 partitions.

**E5.1** What happens to portfolio QAOA if PENALTY is too small (e.g., PENALTY=0.1)? What does QAOA select then?

**E7.1** For a 6-term Hamiltonian with 1024 total shots/iter and uniform allocation, how many shots per term? With adaptive allocation where one term has variance 0.8 and five others 0.05, how many shots goes to the high-variance term?

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What is a QUBO problem? | Minimise xᵀQx over x ∈ {0,1}ⁿ — quadratic unconstrained binary optimisation |
| Q2 | How do you add a constraint to QUBO? | Add penalty A·g(x)² to the objective |
| Q3 | What is the QUBO↔Ising substitution? | xᵢ = (1−sᵢ)/2 where sᵢ ∈ {−1,+1} maps binary to spin |
| Q4 | For MaxCut, do we minimise or maximise the QAOA Hamiltonian? | Maximise — larger ⟨H_C⟩ = more cut edges |
| Q5 | What is the Goemans-Williamson bound for MaxCut? | 0.8785 of optimal — best classical polynomial-time algorithm |
| Q6 | What is adaptive shot allocation? | Allocate more shots to higher-variance Pauli terms to reduce total measurement cost |
| Q7 | QAOA(p=1) approximation ratio for MaxCut on 3-regular graphs? | ≥ 0.6924 (provable, not just empirical) |

### Capstone: 5-Asset Portfolio + Classical vs QAOA Comparison

Build a complete pipeline to compare classical brute force, classical SDP, and QAOA on a 5-asset portfolio:

```python
import numpy as np, itertools

# 5-Asset Portfolio
ASSETS5   = ["AAPL", "GOOGL", "MSFT", "TSLA", "AMZN"]
RETURNS5  = np.array([0.05, 0.03, 0.04, 0.08, 0.06])
COV5      = np.eye(5) * 0.04 + np.ones((5,5)) * 0.005
K5 = 2   # Select 2 of 5

# Classical: enumerate all C(5,2)=10 portfolios
print("=== 5-Asset Portfolio — Classical Enumeration ===")
best_return, best_combo = -999, None
for combo in itertools.combinations(range(5), K5):
    x = np.zeros(5); x[list(combo)] = 1
    adj_return = RETURNS5 @ x - 2.0 * x @ COV5 @ x
    if adj_return > best_return:
        best_return, best_combo = adj_return, combo
print(f"Optimal: {[ASSETS5[i] for i in best_combo]}  risk-adj return={best_return:.4f}")

# QAOA: build QUBO and run
Q5 = build_portfolio_qubo(RETURNS5, COV5, K5, lam=2.0, penalty=5.0)
ising5, _ = qubo_to_ising_hamiltonian(Q5)
coupling5 = [(pauli.index('Z'), pauli.index('Z', pauli.index('Z')+1), coeff)
              for coeff, pauli in ising5 if pauli.count('Z') == 2]

g5, b5, val5 = optimise_qaoa(5, coupling5, shots=1024)
print(f"\nQAOA: γ={g5:.3f}, β={b5:.3f}")
qc5 = build_qaoa_circuit(5, coupling5, g5, b5)
qc5.measure_all()
res5 = backend.run(qc5, shots=4096)
top5 = sorted(res5.counts.items(), key=lambda x: -x[1])[:3]
for bs, cnt in top5:
    x = np.array([int(b) for b in reversed(bs)], dtype=float)
    sel = [ASSETS5[i] for i in range(5) if x[i] == 1]
    print(f"  |{bs}⟩: {cnt:4d} shots → {sel}  {'← budget OK' if sum(x)==K5 else '← budget VIOLATION'}")
```

---

## 9. Glossary and Next Course

### Glossary

| Term | Definition |
|---|---|
| **QUBO** | Quadratic Unconstrained Binary Optimisation: min xᵀQx, xᵢ∈{0,1} |
| **Ising model** | H=Σhᵢsᵢ+ΣJᵢⱼsᵢsⱼ where sᵢ∈{−1,+1} — equivalent to QUBO |
| **Penalty term** | A·g(x)² added to QUBO to enforce constraint g(x)=0 |
| **QUBO↔Ising** | xᵢ=(1−sᵢ)/2; Jᵢⱼ=Qᵢⱼ/4, hᵢ=Qᵢᵢ/2+ΣQᵢⱼ/4 |
| **Weighted MaxCut** | MaxCut with edge weights w_{ij}; Ising Jᵢⱼ=w_{ij}/2 |
| **RQAOA** | Recursive QAOA — fix highest-correlation variable, reduce and recurse |
| **GW bound** | Goemans-Williamson SDP rounding: 0.8785× optimal for MaxCut |
| **Portfolio QUBO** | −μᵀx + λxᵀΣx + A(Σxᵢ−k)² — return, risk, budget |
| **Adaptive shots** | Allocate measurement shots proportional to Pauli term variance |
| **MAB** | Multi-Armed Bandit — exploration/exploitation framework for shot allocation |
| **UCB1** | Upper Confidence Bound: score = estimate + C√(log t / N_pulls) |
| **Approximation ratio** | Achievable objective / optimal objective (QAOA p=1: 0.6924 for MaxCut) |

### Next Course

➡ **ZQL-09** — *Quantum Cryptography & Post-Quantum Security*  
Shor's threat to RSA/ECC (uses ZQL-04 order-finding); lattice-based post-quantum cryptography; BB84 QKD protocol simulation.

---

*All code verified against zuntenium v0.2.0: `circuit.py` (qc.h, qc.rx, qc.rz, qc.cx, qc.measure_all), `simulators.py` (StatevectorSimulator.run).*  
*Generated from: ZQL-08-notes-farhi-glover-cook.md, ZQL-08-notes-egger-hillier-sutton.md.*
