# ZQL-01 Step 1 Notes — Nielsen & Chuang, *Quantum Computation and Quantum Information* (2000) — Chapters 1–2

**The definitive reference. Chapters 1–2 are the backbone of ZQL-01.**

---

## Chapter 1 — Introduction and Overview

### 1.1 Historical Context
- Quantum computation emerged from two fields: quantum physics and computer science.
- Feynman (1982): simulating quantum systems efficiently requires a quantum computer.
- Deutsch (1985): first quantum algorithm — showed quantum can outperform classical in principle.
- Shor (1994): polynomial-time integer factorisation. Grover (1996): √N search speedup.
- **Key insight**: computational problems have quantum hardness classes. BQP ≠ BPP (evidence).

### 1.2 Quantum Bits
- Classical bit: 0 or 1. Qubit: **|ψ⟩ = α|0⟩ + β|1⟩** where α, β ∈ ℂ, |α|²+|β|² = 1.
- |0⟩ and |1⟩ are orthonormal basis vectors (computational basis).
- You cannot observe α and β directly — measurement collapses to |0⟩ or |1⟩.
- The Bloch sphere representation: parameterise as **|ψ⟩ = cos(θ/2)|0⟩ + e^{iφ}sin(θ/2)|1⟩**.
  - θ ∈ [0, π], φ ∈ [0, 2π)
  - North pole: |0⟩; South pole: |1⟩; Equator: superpositions.
  - Single-qubit gates = rotations on the Bloch sphere.
- **[SDK]** `QuantumCircuit(1)` initialises to |0⟩ (north pole). `qc.h(0)` rotates to equator.

### 1.3 Multiple Qubits
- Two-qubit state: **|ψ⟩ = a₀₀|00⟩ + a₀₁|01⟩ + a₁₀|10⟩ + a₁₁|11⟩** — four amplitudes.
- General n-qubit state: 2ⁿ amplitudes. Basis states: |0...0⟩ to |1...1⟩.
- Bell states (maximally entangled 2-qubit states):
  - |Φ⁺⟩ = (|00⟩+|11⟩)/√2 ← Bell state created by H+CNOT
  - |Φ⁻⟩ = (|00⟩−|11⟩)/√2
  - |Ψ⁺⟩ = (|01⟩+|10⟩)/√2
  - |Ψ⁻⟩ = (|01⟩−|10⟩)/√2
- Separable vs entangled: |ψ⟩ is separable if it can be written as |a⟩⊗|b⟩; otherwise entangled.
- **[SDK]** `qc.h(0).cx(0,1)` → |Φ⁺⟩. `.cx(0,1)` acts on the 4-dimensional state space.

## Chapter 2 — Introduction to Quantum Mechanics

### 2.1 Linear Algebra and Dirac Notation

**Ket and Bra**:
- Ket |v⟩ = column vector (state)
- Bra ⟨v| = conjugate transpose row vector = (|v⟩)†
- Inner product: ⟨u|v⟩ ∈ ℂ (scalar). ⟨u|u⟩ ≥ 0; =0 iff u=0.
- Outer product: |u⟩⟨v| is a matrix (operator)

**Key linear algebra facts**:
- Hilbert space H_n for n qubits: ℂ^(2^n) with standard inner product.
- Orthonormal basis: ⟨i|j⟩ = δᵢⱼ. Computational basis {|0⟩,|1⟩} is orthonormal.
- Completeness: Σᵢ|i⟩⟨i| = I (identity).
- Cauchy-Schwarz: |⟨u|v⟩|² ≤ ⟨u|u⟩·⟨v|v⟩.

**Operators**:
- Hermitian: A† = A. Eigenvalues real. Used for observables.
- Unitary: U†U = I. Eigenvalues on unit circle. Used for gates.
- Normal: [A, A†] = 0. Normal operators are diagonalisable in orthonormal basis.
- Spectral decomposition: N = Σᵢλᵢ|eᵢ⟩⟨eᵢ| for normal N with eigenvalues λᵢ.

**[SDK]** `BaseGate.matrix()` returns a numpy unitary matrix. Unitarity can be verified: `np.allclose(U @ U.conj().T, np.eye(n))`.

### 2.2 The Postulates of Quantum Mechanics

**Postulate 1 — State Space**: A closed quantum system is described by a unit vector in a Hilbert space.
*SDK:* statevector = np.ndarray of shape (2^n,) complex128, with np.linalg.norm = 1.

**Postulate 2 — Evolution**: Evolution of a closed quantum system is described by a unitary operator: |ψ'⟩ = U|ψ⟩.
*SDK:* Each gate applies U to the statevector subsystem (qubit subspace).

**Postulate 3 — Measurement**: Projective measurement {Mₘ} with Σ_m M_m†M_m = I. Outcome m with probability p(m) = ⟨ψ|Mₘ†Mₘ|ψ⟩; post-measurement state = Mₘ|ψ⟩/√p(m).
*SDK:* `qc.measure_all()` implements computational-basis projective measurement. Result = sampled classical bit string.

**Postulate 4 — Composite Systems**: State space of composite system = tensor product of component spaces.
*SDK:* `QuantumCircuit(n)` lives in ℂ^(2^n). Two-qubit gate on qubits (i,j) acts as I⊗...⊗U(i,j)⊗...⊗I.

### 2.3 The Pauli Matrices

Fundamental 2×2 Hermitian unitaries:
```
I = [[1,0],[0,1]]   X = [[0,1],[1,0]]   Y = [[0,-i],[i,0]]   Z = [[1,0],[0,-1]]
```
- X: flips |0⟩↔|1⟩ (quantum NOT)
- Y: flips and adds phase
- Z: phases |1⟩ by −1
- XYZ relationships: XY = iZ; YZ = iX; ZX = iY.
- Anti-commutation: {X,Y} = 0; X² = Y² = Z² = I.
- **[SDK]** `qc.x()`, `qc.y()`, `qc.z()` → `XGate, YGate, ZGate`.

### 2.4 Important Single-Qubit Gates

**Hadamard**: H = (X+Z)/√2 = [[1,1],[1,-1]]/√2
- H|0⟩ = |+⟩ = (|0⟩+|1⟩)/√2
- H|1⟩ = |−⟩ = (|0⟩−|1⟩)/√2
- H|+⟩ = |0⟩; H|−⟩ = |1⟩ (its own inverse: H²=I)
- **[SDK]** `qc.h(q)` → `HGate()`

**Phase gate S = [[1,0],[0,i]]**: quarter-turn phase around Z.
**T gate = [[1,0],[0,e^{iπ/4}]]**: eighth-turn phase.
**[SDK]** `qc.s(q)`, `qc.t(q)` → `SGate(), TGate()`.

**Rotation gates** (Bloch sphere rotations):
- Rx(θ) = e^{-iθX/2} = cos(θ/2)I − i·sin(θ/2)X
- Ry(θ) = e^{-iθY/2} = cos(θ/2)I − i·sin(θ/2)Y
- Rz(θ) = e^{-iθZ/2} = diag(e^{-iθ/2}, e^{iθ/2})
- **[SDK]** `qc.rx(theta,q)`, `qc.ry(theta,q)`, `qc.rz(theta,q)`.

**General U3(θ,φ,λ)** = Rz(φ)·Ry(θ)·Rz(λ) up to phase. Spans all SU(2).
- **[SDK]** `qc.u(theta, phi, lam, q)` → `U3Gate`.

### 2.5 Two-Qubit Gates and Universality

**CNOT (CX)**: flips target if control = |1⟩. Matrix in |00⟩,|01⟩,|10⟩,|11⟩ basis:
```
[[1,0,0,0],[0,1,0,0],[0,0,0,1],[0,0,1,0]]
```
**Universality theorem**: {H, S, T, CNOT} is universal for quantum computation.
- Any unitary on n qubits can be decomposed into ≤ O(4ⁿ) gates from this set.
- **[SDK]** This is exactly the QuFlow basis gate set for the statevector simulator. KAK decomposition reduces 2-qubit blocks to ≤ 3 CNOT + single-qubit gates.

### 2.6 Density Matrices (Mixed States)
- Pure state: |ψ⟩ → density matrix **ρ = |ψ⟩⟨ψ|** (rank-1 matrix).
- Mixed state: ρ = Σᵢ pᵢ|ψᵢ⟩⟨ψᵢ| (statistical ensemble). Tr(ρ)=1, ρ≥0.
- Pure test: Tr(ρ²)=1 iff pure; Tr(ρ²)<1 iff mixed.
- Evolution: ρ→UρU†. Measurement: p(m) = Tr(Mₘρ).
- **[SDK]** `DensityMatrixSimulator` tracks ρ (4ⁿ × 4ⁿ matrix) — supports noise channels. `StatevectorSimulator` tracks |ψ⟩ only (pure states).

### 2.7 The No-Cloning Theorem (N&C Box 12.1)
- It is impossible to produce |ψ⟩⊗|ψ⟩ from |ψ⟩⊗|0⟩ for all |ψ⟩.
- Proof: any cloning map U would need to satisfy U|ψ⟩|0⟩=|ψ⟩|ψ⟩; linearity forces contradiction on superpositions.
- **[SDK implication]** You cannot "copy" a quantum state mid-circuit. The `qc.reset(q)` method does NOT preserve the old state — it collapses to |0⟩.

---

## Concept → SDK Mapping (N&C)

| N&C Concept | QuFlow Class/Method | Section |
|---|---|---|
| Qubit state |ψ⟩ | `QuantumCircuit(1)` starting in |0⟩ | §1.2 |
| Bloch sphere Rx/Ry/Rz | `qc.rx/ry/rz(theta, q)` | §1.2 |
| n-qubit Hilbert space ℂ^(2^n) | `QuantumCircuit(n)` | §1.3 |
| Bell state |Φ⁺⟩ | `qc.h(0).cx(0,1)` | §1.3 |
| Unitary gate U | `BaseGate.matrix()` (unitary) | §2.2 |
| Postulate 3 (measurement) | `qc.measure_all()` | §2.2 |
| Pauli X/Y/Z | `qc.x/y/z(q)` | §2.3 |
| U3 general gate | `qc.u(theta,phi,lam,q)` | §2.4 |
| Universality {H,S,T,CNOT} | Default gate set | §2.5 |
| Density matrix ρ | `DensityMatrixSimulator` | §2.6 |
| No-cloning | `qc.reset(q)` collapses (not copies) | §2.7 |
