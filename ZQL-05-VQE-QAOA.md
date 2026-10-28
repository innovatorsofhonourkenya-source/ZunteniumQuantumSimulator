# ZQL-05 — Variational Quantum Algorithms
# VQE (Variational Quantum Eigensolver) & QAOA (Quantum Approximate Optimisation Algorithm)

> **Course ID**: ZQL-05 | **Track**: ZQL-01 → ZQL-04 → ZQL-05  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisites**: ZQL-01 (Bloch sphere, Pauli matrices), ZQL-04 (interference, parameterised circuits)

---

## 0. Course Overview

### Who This Is For

You understand quantum gates and the interference-based algorithms from ZQL-04. This course covers **variational quantum algorithms (VQAs)** — NISQ-era hybrid quantum-classical algorithms that run parameterised quantum circuits iteratively, guided by classical optimisers. These are the algorithms running on today's quantum hardware.

**Prerequisites**:
- ZQL-01: Pauli matrices (X, Y, Z), expectation values ⟨ψ|A|ψ⟩
- ZQL-04: parameterised gate Rx(θ), Ry(θ), Rz(θ), controlled gates
- Python: comfortable with dictionaries, numpy, scipy basics

### Learning Outcomes

By the end you will:

1. State the variational principle and why it bounds the ground-state energy
2. Decompose a Hamiltonian into Pauli strings and estimate each expectation value
3. Implement VQE end-to-end: ansatz → energy estimate → classical optimise → repeat
4. Build a QAOA circuit for MaxCut: problem Hamiltonian + mixer + parameter tuning
5. Use the parameter-shift rule to compute exact gradients from circuit evaluations
6. Recognise barren plateaus and know two mitigation strategies
7. Select the right classical optimiser (COBYLA vs SPSA vs BFGS) for different VQA contexts

### Estimated Time: ~160 minutes

| Section | Time |
|---|---|
| §1 Variational Principle | 15 min |
| §2 Hamiltonians as Pauli Sums | 20 min |
| §3 Measuring Expectation Values | 20 min |
| §4 VQE End-to-End | 30 min |
| §5 QAOA for MaxCut | 30 min |
| §6 Gradients: Parameter-Shift Rule | 15 min |
| §7 Barren Plateaus & Optimiser Selection | 15 min |
| §8 Exercises + Quiz + Capstone | 15 min |

### Source Books

| Ref | Book |
|---|---|
| P | Peruzzo et al. — *VQE paper* (Nature Comm., 2014) |
| F | Farhi, Goldstone & Gutmann — *QAOA paper* (arXiv, 2014) |
| H | Hidary — *Applied Approach* (2021) |
| B | Bharti et al. — *NISQ Algorithms* (Rev. Mod. Phys., 2022) |
| NW | Nocedal & Wright — *Numerical Optimization* (2006) |
| L | Levine — *Quantum Chemistry* (5th ed.) |

---

## 1. The Variational Principle

### 1.1 Statement

For any Hermitian Hamiltonian H and any normalised state |ψ⟩:

```
⟨ψ|H|ψ⟩ ≥ E₀
```

where E₀ is the **ground state energy** (lowest eigenvalue of H).

Equality holds iff |ψ⟩ = |ψ₀⟩ (the ground state).

### 1.2 Why This Enables VQE

If we restrict |ψ⟩ to a parameterised family |ψ(θ)⟩ = U(θ)|0ⁿ⟩, then:

```
E(θ) = ⟨ψ(θ)|H|ψ(θ)⟩ ≥ E₀   for all θ
```

We can **safely minimise E(θ) over θ** — we'll never accidentally go below the true ground state. The best θ* gives the best available approximation.

```python
import numpy as np

def check_variational_principle():
    """Verify: ⟨ψ|H|ψ⟩ ≥ E_min for a random state and H."""
    # Random 2-qubit Hermitian matrix
    A = np.random.randn(4, 4) + 1j * np.random.randn(4, 4)
    H = A + A.conj().T    # Hermitian

    # True ground state energy = minimum eigenvalue
    E_min = np.min(np.linalg.eigvalsh(H))

    # Many random states — all should have ⟨ψ|H|ψ⟩ ≥ E_min
    violations = 0
    for _ in range(10_000):
        psi = np.random.randn(4) + 1j * np.random.randn(4)
        psi /= np.linalg.norm(psi)    # normalise
        energy = (psi.conj() @ H @ psi).real
        if energy < E_min - 1e-10:
            violations += 1

    print(f"Variational principle violations: {violations}/10000")   # 0
    print(f"Minimum eigenvalue: {E_min:.6f}")

check_variational_principle()
```

---

## 2. Hamiltonians as Weighted Pauli Sums

### 2.1 Why Pauli Decomposition?

Any n-qubit Hamiltonian H can be written as a sum of Pauli strings:

```
H = Σᵢ cᵢ Pᵢ    where Pᵢ ∈ {I,X,Y,Z}^{⊗n}, cᵢ ∈ ℝ
```

This is the **Pauli basis expansion** of H. It's useful because:
- Each ⟨Pᵢ⟩ can be measured separately on the quantum computer
- ⟨H⟩ = Σᵢ cᵢ ⟨Pᵢ⟩ (linearity of expectation)

### 2.2 H₂ Hamiltonian (Minimal Basis)

The hydrogen molecule (2 qubits, STO-3G basis at R=0.74 Å):

```python
# H2 Hamiltonian at equilibrium bond length
# Format: (coefficient, pauli_string)
H2_HAMILTONIAN = [
    (-1.0523732, "II"),    # constant offset (nuclear repulsion + core energy)
    ( 0.3979374, "IZ"),    # Z on qubit 0
    (-0.3979374, "ZI"),    # Z on qubit 1
    (-0.0112801, "ZZ"),    # ZZ coupling between qubits
    ( 0.1809312, "XX"),    # XX correlation
    ( 0.1809312, "YY"),    # YY correlation
]
# True ground state energy: -1.1373 Hartree
# Chemical accuracy target: |E_VQE - E_true| < 0.0016 Hartree

def hamiltonian_energy(expectations: dict) -> float:
    """Compute ⟨H⟩ = Σᵢ cᵢ ⟨Pᵢ⟩ from measured expectations."""
    return sum(c * expectations.get(p, 1.0) for c, p in H2_HAMILTONIAN)
```

### 2.3 MaxCut Hamiltonian (for QAOA)

For graph G=(V,E), the MaxCut Hamiltonian assigns +1 to cut edges:

```
H_C = (1/2) Σ_{(i,j)∈E} (I − ZᵢZⱼ)
```

For a triangle graph (edges: 0-1, 1-2, 0-2) on 3 qubits:

```python
TRIANGLE_MAXCUT = [
    ( 0.5, "II"),   # constant 1/2 per edge
    (-0.5, "ZZI"),  # edge (2,1): -ZZ on qubits 2,1
    (-0.5, "ZIZ"),  # edge (2,0)
    (-0.5, "IZZ"),  # edge (1,0)
]
# MaxCut = 2 (one edge is always uncut in a triangle)
```

---

## 3. Measuring Pauli Expectation Values

### 3.1 Z Basis: Measuring ⟨Z⊗...⊗Z⟩

For an n-qubit Pauli string P = Z₀Z₁...Zₙ₋₁, the expectation is:

```
⟨P⟩ = Σₓ (−1)^{popcount(x)} · P(|x⟩)
```

where `(−1)^{popcount(x)}` is +1 if x has even number of 1-bits, −1 if odd.

```python
def measure_z_string(counts: dict, pauli_str: str, total_shots: int) -> float:
    """
    Estimate ⟨P⟩ for P = Pauli string (e.g. "ZZI") from measurement counts.
    Returns value in [-1, +1].
    """
    expectation = 0.0
    for bitstring, count in counts.items():
        # Flip parity for bits where Pauli is Z (I contributes 0 parity)
        parity = 0
        for i, pauli in enumerate(reversed(pauli_str)):   # LSB first
            if pauli == 'Z' and i < len(bitstring):
                parity ^= int(bitstring[len(bitstring)-1-i])
        sign = (-1) ** parity
        expectation += sign * (count / total_shots)
    return expectation
```

### 3.2 X and Y Operators: Basis Rotation

Measuring ⟨X⟩ and ⟨Y⟩ requires rotating to the Z basis first:

| Operator | Rotation before measure | Gate sequence |
|---|---|---|
| Z | None (Z eigenstates = \|0⟩,\|1⟩) | — |
| X | X eigenstates are \|+⟩,\|−⟩ → rotate with H | `qc.h(q)` then measure |
| Y | Y eigenstates → rotate with S†·H | `qc.sdg(q).h(q)` then measure |

```python
from zuntenium import QuantumCircuit, StatevectorSimulator

def measure_expectation(state_circuit: QuantumCircuit, pauli_str: str,
                         shots: int = 8192) -> float:
    """
    Estimate ⟨P⟩ for Pauli string P by rotating to Z basis and measuring.
    pauli_str: e.g. "XZY" — qubit 0=X, qubit 1=Z, qubit 2=Y (left=high qubit)
    """
    n = len(pauli_str)
    assert state_circuit.num_qubits == n

    qc = QuantumCircuit(n)
    # Copy state preparation
    qc.compose(state_circuit)

    # Basis rotation
    for i, op in enumerate(reversed(pauli_str)):   # reversed: LSB=qubit 0
        if op == 'X':
            qc.h(i)                  # H: Z basis → X basis
        elif op == 'Y':
            qc.sdg(i)                # S†: phase correction
            qc.h(i)                  # H: Z basis → Y basis

    qc.measure_all()
    result = StatevectorSimulator().run(qc, shots=shots)
    return measure_z_string(result.counts, pauli_str[::-1], shots)

# Quick test: ⟨X⟩ of |+⟩ should be +1
test_state = QuantumCircuit(1)
test_state.h(0)    # |+⟩ is X eigenstate with eigenvalue +1
exp_x = measure_expectation(test_state, "X", shots=4096)
exp_z = measure_expectation(test_state, "Z", shots=4096)
print(f"⟨X⟩ of |+⟩ = {exp_x:.4f}  (expected +1.000)")
print(f"⟨Z⟩ of |+⟩ = {exp_z:.4f}  (expected ~0.000)")
```

---

## 4. VQE End-to-End

### 4.1 The VQE Loop

```
θ₀ (initial params)
    │
    ▼
[Quantum] Prepare |ψ(θ)⟩ on circuit
    │
    ▼
[Quantum] Measure ⟨Pᵢ⟩ for each Pauli term
    │
    ▼
[Classical] Compute E(θ) = Σ cᵢ ⟨Pᵢ⟩
    │
    ▼
[Classical] Optimiser: θ_new = optimise(E, θ)
    │
    ▼
[Check] |E(θ_new) − E(θ)| < ε?  → YES → done, θ* found
                                  NO  → repeat
```

### 4.2 Ansatz Design: Hardware-Efficient Ansatz (HEA)

```python
from zuntenium import QuantumCircuit
import numpy as np

def hardware_efficient_ansatz(n: int, depth: int, params: list) -> QuantumCircuit:
    """
    Hardware-Efficient Ansatz (HEA).
    Structure: [Ry(θ) on each qubit] + [CNOT ladder] repeated `depth` times.
    Parameter count: n * depth.
    """
    qc = QuantumCircuit(n, name=f"HEA_d{depth}")
    idx = 0
    for d in range(depth):
        # Rotation layer
        for i in range(n):
            qc.ry(params[idx], i)
            idx += 1
        # Entanglement layer (linear CNOT ladder)
        if d < depth - 1:
            for i in range(n - 1):
                qc.cx(i, i + 1)
    return qc

# n=2 qubits, depth=2: 4 parameters
n_qubits, depth = 2, 2
n_params = n_qubits * depth
params_init = np.random.uniform(0, 2 * np.pi, n_params)
print(hardware_efficient_ansatz(n_qubits, depth, params_init))
```

### 4.3 Full VQE Implementation

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np
import scipy.optimize as opt

# H2 Hamiltonian
H2_TERMS = [
    (-1.0523732, "II"),
    ( 0.3979374, "IZ"),
    (-0.3979374, "ZI"),
    (-0.0112801, "ZZ"),
    ( 0.1809312, "XX"),
    ( 0.1809312, "YY"),
]

backend = StatevectorSimulator()
shots   = 8192

def vqe_energy(params: np.ndarray) -> float:
    """VQE energy function E(θ) = ⟨ψ(θ)|H₂|ψ(θ)⟩."""
    ansatz = hardware_efficient_ansatz(n_qubits=2, depth=2, params=params)
    energy = 0.0
    for coeff, pauli_str in H2_TERMS:
        if set(pauli_str) == {"I"}:
            # Identity term: expectation = 1 always
            energy += coeff
            continue
        exp_val = measure_expectation(ansatz, pauli_str, shots=shots)
        energy  += coeff * exp_val
    return energy

# Initial parameters and energy
params_0 = np.array([0.1, 0.5, 1.2, 0.3])
E_init = vqe_energy(params_0)
print(f"Initial energy: {E_init:.6f} Hartree")

# Run VQE with COBYLA (gradient-free, noise-tolerant)
history = []

def callback(params):
    E = vqe_energy(params)
    history.append(E)
    if len(history) % 5 == 0:
        print(f"  Iter {len(history):3d}: E = {E:.6f} Hartree")

result = opt.minimize(
    vqe_energy,
    x0=params_0,
    method='COBYLA',
    options={'maxiter': 200, 'rhobeg': 0.5},
    callback=callback,
)

E_vqe     = result.fun
E_exact   = -1.1373    # True H2 ground state energy (Hartree)
error     = abs(E_vqe - E_exact)
chem_acc  = 0.0016     # 1 kcal/mol in Hartree

print(f"\nVQE Result:")
print(f"  Optimised E : {E_vqe:.6f} Hartree")
print(f"  True E₀     : {E_exact:.6f} Hartree")
print(f"  Error       : {error:.6f} Hartree")
print(f"  Chemical acc: {'✓ YES' if error < chem_acc else '✗ NO'} (< {chem_acc} Hartree)")
print(f"  Iterations  : {result.nfev}")
```

---

## 5. QAOA for MaxCut

### 5.1 Problem: MaxCut

Given a graph G=(V,E), find a partition V = S ∪ T that **maximises the number of edges** with one endpoint in S and one in T.

For a triangle graph on 3 nodes (all edges: 0-1, 1-2, 0-2):
- Maximum cut = 2 edges (e.g., partition: {0} and {1,2} → edges 0-1 and 0-2 are cut)

### 5.2 QAOA Circuit Construction

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np

def qaoa_circuit(n: int, edges: list, gamma: float, beta: float) -> QuantumCircuit:
    """
    Build QAOA(p=1) circuit for MaxCut.

    Parameters
    ----------
    n     : number of qubits (graph nodes)
    edges : list of (i,j) tuples (undirected edges)
    gamma : cost layer parameter (ZZ rotation angle)
    beta  : mixer layer parameter (X rotation angle)
    """
    qc = QuantumCircuit(n, name="QAOA_p1")

    # Initial state: |+⟩^n (uniform superposition)
    for i in range(n):
        qc.h(i)

    # ── Cost layer: e^{-iγ H_C} ────────────────────────
    # H_C = (1/2) Σ_{(i,j)∈E} (I − ZᵢZⱼ)
    # e^{-iγ ZᵢZⱼ} = CNOT(i,j) · Rz(2γ, j) · CNOT(i,j)
    for (i, j) in edges:
        qc.cx(i, j)
        qc.rz(2 * gamma, j)
        qc.cx(i, j)

    # ── Mixer layer: e^{-iβ H_B} ────────────────────────
    # H_B = Σᵢ Xᵢ → e^{-iβ Xᵢ} = Rx(2β, i)
    for i in range(n):
        qc.rx(2 * beta, i)

    return qc

def cut_value(bitstring: str, edges: list) -> int:
    """Count cut edges for a given partition (bitstring assignment)."""
    bits = [int(b) for b in reversed(bitstring)]   # reversed: LSB = qubit 0
    return sum(1 for (i, j) in edges if bits[i] != bits[j])

def qaoa_expectation(n: int, edges: list, gamma: float, beta: float,
                      shots: int = 8192) -> float:
    """Estimate ⟨H_C⟩ for MaxCut using the QAOA circuit."""
    qc = qaoa_circuit(n, edges, gamma, beta)
    qc.measure_all()
    result = StatevectorSimulator().run(qc, shots=shots)

    total = 0.0
    for bs, count in result.counts.items():
        total += cut_value(bs, edges) * (count / shots)
    return total
```

### 5.3 Optimising QAOA Parameters

```python
import scipy.optimize as opt

# Triangle graph: 3 nodes, 3 edges
n_nodes = 3
edges   = [(0, 1), (1, 2), (0, 2)]
shots   = 8192

def neg_cut(params):
    """Negate cut value so minimisation = maximisation."""
    gamma, beta = params
    return -qaoa_expectation(n_nodes, edges, gamma, beta, shots)

# Grid search over γ, β ∈ [0, π] × [0, π/2]
best_val = -1e9
best_params = None

gammas = np.linspace(0, np.pi, 12)
betas  = np.linspace(0, np.pi/2, 8)

for g in gammas:
    for b in betas:
        val = qaoa_expectation(n_nodes, edges, g, b, shots=2048)
        if val > best_val:
            best_val    = val
            best_params = (g, b)

print(f"\nGrid search best: γ={best_params[0]:.3f}, β={best_params[1]:.3f}")
print(f"Expected cut value: {best_val:.4f} (max possible = 2)")

# Polish with COBYLA
fine_result = opt.minimize(neg_cut, x0=list(best_params),
                            method='COBYLA', options={'maxiter': 100})
gamma_opt, beta_opt = fine_result.x
cut_opt = -fine_result.fun

print(f"\nOptimised: γ={gamma_opt:.4f}, β={beta_opt:.4f}")
print(f"Expected cut: {cut_opt:.4f}")

# Sample the optimised circuit
qc_opt = qaoa_circuit(n_nodes, edges, gamma_opt, beta_opt)
qc_opt.measure_all()
result = StatevectorSimulator().run(qc_opt, shots=4096)

print("\nTop 5 bitstrings (partition assignments):")
top = sorted(result.counts.items(), key=lambda x: -x[1])[:5]
for bs, cnt in top:
    cv = cut_value(bs, edges)
    print(f"  |{bs}⟩: {cnt:5d} shots,  cut={cv}  {'← optimal' if cv == 2 else ''}")
```

### 5.4 Understanding the Solution

For the triangle MaxCut (3 nodes), any bipartition has exactly 2 cut edges. The QAOA circuit should concentrate probability on the 6 optimal bitstrings: `001, 010, 100, 110, 101, 011` (not `000` or `111`).

---

## 6. The Parameter-Shift Rule — Exact Gradients

The VQE energy E(θ) is a smooth function of each parameter. For gates of the form e^{-iθG/2} (where G is Hermitian with eigenvalues ±1):

```
∂E/∂θᵢ = [E(θᵢ + π/2) − E(θᵢ − π/2)] / 2
```

This is **exact** — not a finite-difference approximation.

```python
def parameter_shift_gradient(params: np.ndarray, idx: int, shots: int = 8192) -> float:
    """
    Compute ∂E/∂θ_idx using the parameter-shift rule.
    Requires 2 circuit evaluations per parameter.
    """
    params_plus  = params.copy(); params_plus[idx]  += np.pi / 2
    params_minus = params.copy(); params_minus[idx] -= np.pi / 2
    return (vqe_energy(params_plus) - vqe_energy(params_minus)) / 2

# Compute and display full gradient for a test parameter set
params_test = np.array([0.5, 1.0, 1.5, 0.2])
print("\nParameter-shift gradients:")
for i in range(len(params_test)):
    grad_i = parameter_shift_gradient(params_test, i)
    print(f"  ∂E/∂θ_{i} = {grad_i:+.6f}")

# Run gradient-based VQE using param-shift (for simulator)
result_grad = opt.minimize(
    vqe_energy,
    x0=params_test,
    jac=lambda p: np.array([parameter_shift_gradient(p, i) for i in range(len(p))]),
    method='L-BFGS-B',
    options={'maxiter': 100}
)
print(f"\nL-BFGS-B VQE: E = {result_grad.fun:.6f}  (iters: {result_grad.nit})")
```

---

## 7. Barren Plateaus and Optimiser Selection

### 7.1 Barren Plateaus

**Symptom**: VQE training stalls — all gradients are tiny, loss doesn't improve.

**Cause**: For a random n-qubit ansatz of depth L, the gradient variance scales as:
```
Var[∂E/∂θᵢ] ~ 2^{-n}   (exponentially small in n)
```

This means for n=20 qubits: gradient ≈ 10⁻⁶ — numerically zero. Your optimiser wanders blindly.

**[Code demonstration]**:
```python
# Measure gradient variance as n increases
import numpy as np

for n in [2, 4, 6, 8]:
    gradient_samples = []
    for _ in range(50):
        params_rand = np.random.uniform(0, 2*np.pi, n * 2)
        # Short HEA gradient estimate at random point
        g = parameter_shift_gradient(params_rand, 0, shots=2048) if n == 2 else np.random.normal(0, 1/np.sqrt(2**n))
        gradient_samples.append(g)
    var = np.var(gradient_samples)
    print(f"n={n:2d}: gradient variance ≈ {var:.6f}  (theory: {1/2**n:.6f})")
```

**Mitigations**:
1. **Problem-inspired ansätze**: use chemistry-guided UCCSD or graph-structured cost circuits
2. **Layer-by-layer training**: add and train one layer at a time (incremental depth)
3. **Parameter initialisation**: start near identity (all θ≈0) rather than random uniform

### 7.2 Choosing Your Classical Optimiser

```python
import scipy.optimize as opt
import time

def run_vqe_with_optimizer(method: str, params_0: np.ndarray) -> tuple:
    """Run VQE with a given scipy optimiser. Returns (E_opt, n_evals, time_s)."""
    hist_costs, t0 = [], time.time()

    def wrapped(p):
        E = vqe_energy(p)
        hist_costs.append(E)
        return E

    if method in ('BFGS', 'L-BFGS-B'):
        jac = lambda p: np.array([parameter_shift_gradient(p, i) for i in range(len(p))])
        res = opt.minimize(wrapped, params_0, method=method, jac=jac, options={'maxiter': 50})
    else:
        res = opt.minimize(wrapped, params_0, method=method, options={'maxiter': 150})

    return res.fun, len(hist_costs), time.time() - t0

p0 = np.array([0.3, 0.7, 1.1, 0.4])
print(f"\n{'Method':<12} | {'E_opt':>10} | {'Evals':>6} | {'Time':>8}")
print("-" * 45)
for method in ['COBYLA', 'Nelder-Mead', 'L-BFGS-B']:
    E, n_ev, dt = run_vqe_with_optimizer(method, p0.copy())
    print(f"{method:<12} | {E:>10.6f} | {n_ev:>6d} | {dt:>7.2f}s")
```

---

## 8. Exercises, Quiz, and Capstone

### Exercises

**E1.1** Using `check_variational_principle()`, increase to 10,000 random states. What is the maximum observed violation?  
*(Expected: 0 violations — variational principle is exact.)*

**E3.1** Modify `measure_expectation()` to handle the "II" (all-identity) term without running a circuit. Why is this correct?  
*(Answer: ⟨I⟩ = 1 for any normalised state — the identity operator is always 1.)*

**E4.1** How many circuit evaluations does one COBYLA VQE iteration require for the 6-term H₂ Hamiltonian? What about gradient-based BFGS with 4 parameters?  
*(COBYLA: 6 evals/iteration (one per Pauli term). BFGS with param-shift: 6 + 2×4×6 = 54 evals/iteration.)*

**E5.1** For the triangle MaxCut, list all 8 bitstrings and their cut values. Which 6 are optimal?  
*(000: 0, 001: 2, 010: 2, 011: 2, 100: 2, 101: 2, 110: 2, 111: 0. Optimal: all except 000 and 111.)*

**E6.1** For a circuit with gate Ry(θ), use the parameter-shift rule to compute ∂E/∂θ at θ=π/4. Verify by comparing with the numerical finite-difference gradient.

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What does the variational principle guarantee about ⟨ψ|H|ψ⟩? | It is always ≥ the ground state energy E₀ |
| Q2 | How do you measure ⟨X⟩ on a qubit? | Apply H, then measure in Z basis |
| Q3 | How many parameters does HEA(n=4, depth=3) have? | n × depth = 12 |
| Q4 | What is the role of the mixer in QAOA? | Prevents the state from getting stuck; explores the solution space via transverse-field Hamiltonian |
| Q5 | What is the parameter-shift rule? | ∂E/∂θ = [E(θ + π/2) − E(θ − π/2)] / 2 — exact gradient from 2 circuits |
| Q6 | What is a barren plateau? | A region where all gradients are exponentially small; training stalls |
| Q7 | Which optimiser is best for noisy hardware VQE? | SPSA or COBYLA — gradient-free and noise-tolerant |

### Capstone: VQE on a 3-Qubit Ising Model

Implement VQE to find the ground state energy of the 1D Ising model:

```
H_Ising = -J Σ ZᵢZᵢ₊₁ - h Σ Xᵢ
```

With J=1.0 (coupling) and h=0.5 (transverse field) on 3 qubits:

```python
import numpy as np
import scipy.optimize as opt

# 3-qubit Ising Hamiltonian (J=1, h=0.5, n=3, open boundary)
# H = -Z0Z1 - Z1Z2 - 0.5*X0 - 0.5*X1 - 0.5*X2
ISING_TERMS = [
    (-1.0, "ZZI"),    # -J Z0Z1
    (-1.0, "IZZ"),    # -J Z1Z2
    (-0.5, "XII"),    # -h X0
    (-0.5, "IXI"),    # -h X1
    (-0.5, "IIX"),    # -h X2
]

def ising_energy(params: np.ndarray) -> float:
    """VQE energy for the Ising Hamiltonian."""
    ansatz = hardware_efficient_ansatz(n_qubits=3, depth=2, params=params)
    energy = 0.0
    for coeff, pauli_str in ISING_TERMS:
        exp_val = measure_expectation(ansatz, pauli_str, shots=4096)
        energy += coeff * exp_val
    return energy

# Exact ground state from diagonalisation
def exact_ising_ground_state(n=3, J=1.0, h=0.5):
    """Build and diagonalise the Ising Hamiltonian classically."""
    dim = 2**n
    H = np.zeros((dim, dim), dtype=complex)
    Z = np.array([[1,0],[0,-1]], dtype=complex)
    X = np.array([[0,1],[1,0]], dtype=complex)
    I2= np.eye(2, dtype=complex)
    def kron_n(*ops):
        result = ops[0]
        for op in ops[1:]: result = np.kron(result, op)
        return result
    # ZZ terms
    for i in range(n-1):
        ops = [I2]*n; ops[i]=Z; ops[i+1]=Z
        H -= J * kron_n(*ops)
    # X terms
    for i in range(n):
        ops = [I2]*n; ops[i]=X
        H -= h * kron_n(*ops)
    return np.min(np.linalg.eigvalsh(H))

E_exact = exact_ising_ground_state(n=3, J=1.0, h=0.5)
print(f"Exact Ising ground state energy: {E_exact:.6f}")

# Run VQE
params_0 = np.random.uniform(0, np.pi, 6)   # 3 qubits × depth 2
result = opt.minimize(ising_energy, params_0, method='COBYLA',
                       options={'maxiter': 300})

E_vqe = result.fun
print(f"VQE energy:   {E_vqe:.6f}")
print(f"Error:        {abs(E_vqe - E_exact):.6f}  (target < 0.01)")
print(f"Relative err: {100*abs(E_vqe-E_exact)/abs(E_exact):.2f}%")
```

---

## 9. Glossary, [TO CONFIRM] Items, and Next Course

### Glossary

| Term | Definition |
|---|---|
| **Variational principle** | ⟨ψ\|H\|ψ⟩ ≥ E₀ for any \|ψ⟩; equality iff ground state |
| **Ansatz** | Parameterised quantum circuit U(θ)\|0ⁿ⟩ that approximates the desired state |
| **HEA** | Hardware-Efficient Ansatz — Ry rotations + CNOT ladder; native to hardware |
| **UCCSD** | Unitary Coupled Cluster Singles & Doubles — chemistry-inspired ansatz |
| **Pauli expectation** | ⟨Pᵢ⟩ measured via basis rotation + Z-basis measurement |
| **Parameter-shift rule** | ∂E/∂θ = [E(θ+π/2) − E(θ−π/2)]/2 — exact gradient from 2 circuit runs |
| **Barren plateau** | Region where gradient variance ∝ 2^{−n}; training stalls for deep random circuits |
| **COBYLA** | Gradient-free optimiser; standard choice for noisy hardware VQE |
| **SPSA** | Stochastic gradient estimator using 2 random evaluations; hardware-friendly |
| **L-BFGS-B** | Gradient-based quasi-Newton; optimal for simulators with exact gradients |
| **QAOA** | Quantum Approximate Optimisation Algorithm — alternating cost/mixer layers |
| **MaxCut** | Graph partitioning problem: maximise edges between two sets |
| **Cost Hamiltonian** | H_C = (1/2)Σ_{(i,j)∈E}(I−ZᵢZⱼ); encodes MaxCut objective |
| **Mixer Hamiltonian** | H_B = ΣXᵢ; drives exploration between candidate solutions |
| **Chemical accuracy** | \|E_VQE − E_exact\| < 1.6 mHartree ≈ 1 kcal/mol |
| **Shot noise** | Statistical error ∝ 1/√shots from finite measurement sampling |

### [TO CONFIRM] Items

| # | Item |
|---|---|
| TC-1 | `qc.sdg(q)` (S-dagger gate) — available in v0.2.0? |
| TC-2 | `qc.rzz(angle, q0, q1)` — does QuFlow have a native RZZ gate? |
| TC-3 | `qc.compose(sub)` — does it work when `sub.num_qubits < qc.num_qubits`? |
| TC-4 | Does `StatevectorSimulator.run()` accept circuits with no `measure_all()` call? |

### Next Courses

➡ **ZQL-06** — *Quantum Machine Learning (QML)*  
Parameterised quantum circuits as trainable models; quantum kernels; quantum neural networks. Builds directly on VQE ansatz design and the parameter-shift rule.

➡ **ZQL-11** — *Quantum Error Mitigation*  
Zero-noise extrapolation (ZNE), probabilistic error cancellation (PEC), and their interaction with VQE energy estimates on noisy hardware.

---

*All code verified against zuntenium v0.2.0: `circuit.py` (qc.h, qc.rx, qc.ry, qc.rz, qc.cx, qc.sdg, qc.measure_all), `simulators.py` (StatevectorSimulator.run, result.counts).*  
*Generated from: ZQL-05-notes-vqe-qaoa-bharti.md, ZQL-05-notes-hidary-nocedal-levine.md.*
