# ZQL-00.01 Step 1 Notes — Nakahara & Ohmi, *Quantum Computing: From Linear Algebra to Physical Realizations* (2008) (Chs. 1–2)

**Focus: mathematical prerequisites for SDK use. Chapters 1–2 only.**

---

## Chapter 1 — Linear Algebra Prerequisites

### 1.1 Complex Vector Spaces
- A qubit state lives in ℂ² (2-dimensional complex Hilbert space).
- Basis: {|0⟩, |1⟩} = {(1,0)ᵀ, (0,1)ᵀ} — the *computational basis*.
- General state: **|ψ⟩ = α|0⟩ + β|1⟩** where α,β ∈ ℂ, |α|² + |β|² = 1.
- Bra-ket notation: ket |ψ⟩ is a column vector; bra ⟨ψ| is its conjugate transpose.
- Inner product: ⟨φ|ψ⟩ ∈ ℂ; gives probability amplitude. ⟨0|0⟩=1, ⟨0|1⟩=0.
- **[ANALOGY]** The `StatevectorSimulator` stores the exact state as a `numpy.ndarray` of shape `(2^n,)` complex128 — this is the ket vector.

### 1.2 Unitary Matrices
- A gate U is valid if and only if U†U = UU† = I (preserves norms = probability).
- Eigenvalues of U lie on the unit circle: |λ| = 1.
- **[SDK]** Every gate in QuFlow has a `.matrix()` method returning a numpy unitary matrix. `BaseGate.matrix()`.
- Pauli matrices (σₓ, σᵧ, σ_z): X,Y,Z gates. H = (X+Z)/√2.

### 1.3 Tensor Products
- n-qubit system: H_n = ℂ² ⊗ ℂ² ⊗ … ⊗ ℂ² (n times) = ℂ^(2^n).
- |00⟩ = |0⟩⊗|0⟩ = (1,0,0,0)ᵀ.
- Gate on qubit k acts as I⊗…⊗U(k)⊗…⊗I on the full space.
- **[SDK]** `QuantumCircuit(n)` creates an n-qubit circuit; the statevector simulator stores 2^n complex amplitudes (numpy array of shape `(2^n,)` of dtype complex128).

### 1.4 Measurement
- Projective measurement: Born rule P(m) = |⟨m|ψ⟩|².
- Post-measurement state: collapses to |m⟩.
- Expectation value: ⟨O⟩ = ⟨ψ|O|ψ⟩.
- **[SDK]** `qc.measure(q, c)` / `qc.measure_all()`. Multiple shots → sample the Born rule distribution.

## Chapter 2 — Physical Realizations (conceptual awareness for ZQL-00)

- Superconducting qubits (IBM, Rigetti): two lowest energy levels of a Josephson junction. Gate time ≈ 20–50 ns. T₁ ≈ 100 µs.
- Trapped ions (IonQ): energy levels of laser-cooled ions. Gate time ≈ 100 µs. T₂ ≈ seconds.
- Both: have limited connectivity, finite T₁/T₂, and gate error rates ≈ 0.1–1%.
- **[SDK]** `IBMBackend` / MHOS connects to real hardware. `HardwareAwareTranspiler` accounts for connectivity and gate errors.
- **[ANALOGY]** For ZQL-00 (onboarding), we always use `StatevectorSimulator` — no hardware errors.

---

## Key Math Takeaways for SDK Beginners

| Math Concept | What it means for code |
|---|---|
| State ∈ ℂ² | `QuantumCircuit(1)` starts in |0⟩ = `[1+0j, 0+0j]` |
| Unitary gate U | `qc.h(0)` → U acts on amplitude vector |
| Measurement | `qc.measure_all()` → samples Born rule → `result.counts` |
| n-qubit tensor product | `QuantumCircuit(n)` → 2^n statevector |
| |α|²+|β|²=1 | Simulator always maintains normalization |
