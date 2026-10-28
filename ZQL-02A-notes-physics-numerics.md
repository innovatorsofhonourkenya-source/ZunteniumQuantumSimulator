# ZQL-02A Step 1 Notes — Physics + Numerics Sources

---

# A. Cohen-Tannoudji, Diu & Laloë — *Quantum Mechanics* Vol. 1 (2014)

**Relevant for: density matrix physics, projective measurement, Born rule collapse**

## Ch. III — The Postulates of Quantum Mechanics

### The State Vector
- State = ray in Hilbert space (equivalence class of vectors differing by a phase).
- **[SDK]** `StatevectorSimulator` stores one representative vector |ψ⟩ as a numpy array. Global phase is irrelevant to measurement.

### Observable and Measurement
- Observable = Hermitian operator A with spectral decomposition A = Σₙ aₙ|aₙ⟩⟨aₙ|.
- Measurement of A: probability P(aₙ) = |⟨aₙ|ψ⟩|² (Born rule). Post-measurement state: |aₙ⟩.
- **[SDK]** Computational basis measurement (Z basis) = observing Pauli Z ⊗ Z ⊗ … ⊗ Z. `qc.measure_all()` samples Born rule → `result.counts`.

### The Density Operator
- Pure state ρ = |ψ⟩⟨ψ|: one sharp quantum state.
- Mixed state ρ = Σᵢ pᵢ|ψᵢ⟩⟨ψᵢ|: statistical mixture (classical uncertainty + quantum).
- Evolution: ρ → UρU† (unitary). Expectation: ⟨A⟩ = Tr(ρA).
- **[SDK]** `DensityMatrixSimulator.apply_gate(matrix, qubits)` implements ρ → UρU†.
  `DensityMatrixSimulator.expectation_value(observable)` implements Tr(ρO).

### Projective Measurement on ρ
- Probability of outcome aₙ: P(aₙ) = Tr(Pₙ ρ) where Pₙ = |aₙ⟩⟨aₙ|.
- Post-measurement ρ: ρ' = Pₙ ρ Pₙ / P(aₙ).
- **[SDK]** `DensityMatrixSimulator.get_probabilities()` returns diag(ρ) = P(x) for all basis x. `DensityMatrixSimulator.sample(shots)` draws from this distribution.

## Ch. IV — Simple Applications

### Two-State System (Qubit)
- Basis: {|+⟩,|−⟩} for spin-½. H gate maps from {|0⟩,|1⟩} to {|+⟩,|−⟩}.
- Larmor precession: spin precesses around B-field → exactly Rz(ωt) on Bloch sphere.
- **[ANALOGY]** Gate sequences simulate controlled quantum dynamics: `qc.rz(omega*t, 0)` = Larmor precession.

---

# B. Breuer & Petruccione — *The Theory of Open Quantum Systems* (Oxford, 2002)

**Relevant for: Kraus operators, amplitude damping, phase damping, decoherence**

## Ch. 2 — Density Matrix and Open System Dynamics

### Why Open Systems?
- Closed system: |ψ⟩ evolves unitarily. But real qubits interact with environment (phonons, photons, other qubits) → entanglement with environment → effective mixed state.
- The *reduced* density matrix ρ_S = Tr_E(ρ_SE) describes the qubit alone, and this can be non-unitary.

### Kraus Representation Theorem
- **Any** physically valid quantum channel ε on ρ can be written as:
  ε(ρ) = Σₖ Kₖ ρ Kₖ†  where Σₖ Kₖ†Kₖ = I (completeness)
- The Kₖ are Kraus operators. They need not be square; they can be non-unitary.
- **[SDK]** `DensityMatrixSimulator.apply_noise_channel(kraus_ops, qubit)` directly implements ε(ρ) = Σₖ Kₖρₖ†.

### Common Noise Channels

**Amplitude Damping** (energy loss, T1 decay):
```
K0 = [[1, 0], [0, √(1-γ)]]    K1 = [[0, √γ], [0, 0]]
```
Physical meaning: qubit decays from |1⟩ to |0⟩ with probability γ.

**Phase Damping** (dephasing, T2 decay):
```
K0 = [[1, 0], [0, √(1-λ)]]    K1 = [[0, 0], [0, √λ]]
```
Physical meaning: off-diagonal elements of ρ shrink → coherence lost.

**Depolarizing Channel**:
```
ε(ρ) = (1-p) ρ + p·I/2
```
Equivalent Kraus form: K0=√(1-3p/4)·I, K1=√(p/4)·X, K2=√(p/4)·Y, K3=√(p/4)·Z.

**[SDK usage]**:
```python
import numpy as np
from zuntenium.backends import DensityMatrixSimulator

gamma = 0.1  # 10% amplitude damping
K0 = np.array([[1, 0], [0, np.sqrt(1 - gamma)]], dtype=complex)
K1 = np.array([[0, np.sqrt(gamma)], [0, 0]], dtype=complex)

sim = DensityMatrixSimulator(num_qubits=1)
# Apply gate then noise
sim.apply_gate(np.array([[1,1],[1,-1]])/np.sqrt(2), [0])  # H gate
sim.apply_noise_channel([K0, K1], qubit=0)
print(f"Purity after noise: {sim.purity():.4f}")   # < 1.0
print(f"Von Neumann entropy: {sim.von_neumann_entropy():.4f}")  # > 0
```

### Lindblad Master Equation (conceptual awareness)
- Continuous-time limit: dρ/dt = -i[H,ρ] + Σₖ(LₖρLₖ† - ½{Lₖ†Lₖ,ρ})
- Lₖ = jump operators (collapse operators). Lₖ=√γ·σ₋ for amplitude damping.
- **[ANALOGY]** QuFlow's Kraus map = one discrete time step of Lindblad evolution.

---

# C. Golub & Van Loan — *Matrix Computations* (4th ed., 2013)

**Relevant for: tensor contraction, np.einsum, numerical stability**

## Ch. 1 — Matrix Operations

### Kronecker Products
- (A⊗B)ᵢⱼ,ₖₗ = Aᵢₖ · Bⱼₗ. Shape: (mn)×(pq) for A:m×p, B:n×q.
- **[SDK]** `DensityMatrixSimulator._build_full_unitary()` uses `np.kron` for single-qubit fast path.

### Tensor Contraction via Index Notation
- Einstein summation convention: repeated index = sum over that index.
- `np.einsum("ij,jk->ik", A, B)` = A @ B.
- For gate application: reshape statevector to (2,2,...,2) axis tensor, contract gate indices.
- **[SDK]** `GateApplicator` uses `np.einsum` for efficient in-place gate application without forming 2^n × 2^n matrices.

## Ch. 2 — Matrix Analysis

### Numerical Stability
- Gate matrices should maintain unitarity: U†U − I should be ~1e-15 (machine epsilon).
- **[SDK]** `DensityMatrixSimulator` uses defensive renormalization: `probs = probs / probs.sum()` before sampling — guards against small numerical drift making probabilities sum to 1 + ε.

### Eigenvalue Problems
- Density matrix ρ is positive semidefinite: all eigenvalues ≥ 0.
- Von Neumann entropy: S(ρ) = −Tr(ρ log ρ) = −Σᵢ λᵢ log λᵢ (sum over eigenvalues of ρ).
- **[SDK]** `DensityMatrixSimulator.von_neumann_entropy()` computes this via `np.linalg.eigvalsh(ρ)`.

## Ch. 12 — Special Linear Systems

### Sparse Gate Representation
- For large n, the 2^n × 2^n gate matrix is mostly identity. Sparse or tensor representations are critical.
- **[SDK]** `DensityMatrixSimulator._build_full_unitary()` builds the full dense matrix only when needed (for small n ≤ 15). For large n, the einsum-based approach avoids materialising the full matrix.

---

## Cross-Book Synthesis

| Topic | Cohen-Tannoudji | Breuer & Petruccione | Golub & Van Loan | SDK |
|---|---|---|---|---|
| Pure state evolution | ρ → UρU† | — | Matrix mult | `apply_gate()` |
| Noise channel | — | Kraus ε(ρ) | — | `apply_noise_channel()` |
| Measurement | Born rule / diagonal | Tr(Pₙρ) | — | `get_probabilities()` |
| Tensor product gate | — | — | Kronecker product | `np.kron` fast path |
| Gate on tensor | — | — | Einsum contraction | `GateApplicator.einsum` |
| Entropy | — | — | Eigenvalues of ρ | `von_neumann_entropy()` |
| State purity | Tr(ρ²) | Tr(ρ²) | — | `purity()` |
