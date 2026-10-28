# ZQL-01 Step 1 Notes — Combined: Axler, Susskind & Friedman, Quantum Country

---

# A. Sheldon Axler — *Linear Algebra Done Right* (3rd ed.)

**Relevant chapters: 1, 2, 5, 7 (complex inner product spaces and operators)**

## Ch. 1 — Vector Spaces
- Abstract definition: set V with addition and scalar multiplication over field F (ℝ or ℂ).
- Subspaces, spans, direct sums.
- **[SDK relevance]** The n-qubit Hilbert space ℂ^(2^n) is a vector space over ℂ. All the abstract rules apply to statevectors.

## Ch. 2 — Finite-Dimensional Vector Spaces
- Basis: linearly independent spanning set. Dimension = number of basis vectors.
- Every finite-dimensional inner product space has an orthonormal basis.
- Gram-Schmidt: algorithm to orthonormalise any basis.
- **[SDK]** The computational basis {|00...0⟩,...,|11...1⟩} is already orthonormal.

## Ch. 5 — Eigenvalues and Eigenvectors
- Av = λv. Eigenvalues, eigenvectors, eigenspaces.
- Diagonalisation: A = PDP⁻¹ with D = diag(λ₁,...,λₙ).
- Cayley-Hamilton: A satisfies its own characteristic polynomial.
- **[SDK]** Each Pauli gate has eigenvalues ±1. Z-basis states |0⟩,|1⟩ are eigenstates of Z. Measuring in a basis = computing eigenvalues.

## Ch. 7 — Operators on Inner Product Spaces
- **Self-adjoint (Hermitian) operators**: A = A†. Eigenvalues real; eigenvectors orthogonal. Used for observables.
- **Unitary operators**: U†U = I. Eigenvalues on unit circle. Used for gates.
- Normal operators: commute with their adjoint. Diagonalisable.
- Spectral theorem: every self-adjoint operator on finite-dim complex inner product space is diagonalisable with real eigenvalues.
- **[SDK]** The Pauli matrices σₓ, σᵧ, σ_z are Hermitian; H, X, Z are both Hermitian *and* unitary (self-inverse). CX is unitary but not Hermitian.

**Key Axler insight for ZQL-01**: Axler never uses determinants — basis everything on linear maps. This is the right perspective for quantum: think about *operators acting on state space*, not just matrices.

---

# B. Susskind & Friedman — *Quantum Mechanics: The Theoretical Minimum* (2014)

**Relevant lectures: 1–5 (quantum states, spin, observables, entanglement)**

## Lecture 1–2 — States and Spin
- Susskind's framing: a quantum system is a device that prepares and measures spin states.
- Spin-½ system is exactly a qubit: |u⟩ (spin-up) = |0⟩, |d⟩ (spin-down) = |1⟩.
- Measurement along X axis: eigenstates of σₓ are |+⟩ = (|0⟩+|1⟩)/√2 and |−⟩ = (|0⟩−|1⟩)/√2.
- **[SDK]** Measuring in X basis = apply H gate then measure in Z basis. `qc.h(q).measure(q,c)`.

## Lecture 3 — Observables and Expectation Values
- Observable = Hermitian operator A. ⟨A⟩ = ⟨ψ|A|ψ⟩.
- Pauli expectation values: ⟨Z⟩ = |α|² − |β|² (difference of probabilities for |0⟩ and |1⟩).
- Variance: (ΔA)² = ⟨A²⟩ − ⟨A⟩².
- **[SDK relevance]** VQE (ZQL-05) measures ⟨H⟩ = Σᵢ cᵢ⟨Pᵢ⟩ where Pᵢ are Pauli strings. `PauliHamiltonian.estimate_expectation()`.

## Lecture 4 — Time Evolution
- Time evolution: |ψ(t)⟩ = e^{-iHt/ℏ}|ψ(0)⟩. The generator H is the Hamiltonian.
- Schrödinger equation: iℏ d|ψ⟩/dt = H|ψ⟩.
- **[ANALOGY]** Gate-based quantum computing discretises this: each gate U = e^{-iHt} for some effective Hamiltonian H and time t. Rx(θ) = e^{-iθX/2}. ZQL-07 content.

## Lecture 5 — Entanglement
- Susskind's entanglement criterion: if Alice and Bob share a 2-qubit state, it's entangled iff measuring Alice's qubit instantly determines Bob's outcome (in any basis).
- EPR paradox, Bell inequalities (conceptual awareness).
- **[SDK]** Bell state in QuFlow: `qc.h(0).cx(0,1)`. Verify: outcomes are always "00" or "11".

---

# C. Quantum Country — Matuschak & Nielsen (free, quantumcountry.com)

**Format**: mnemonic essays with embedded flashcard system (spaced repetition).

## "Quantum Computing for Computer Scientists" Essay
- Nielsen's teaching framing: quantum computing = quantum gates + measurement.
- Key mnemonic: "H gate = coin flip; CNOT = classical copy machine — but quantum-entangled".
- Superposition quiz insight: "the qubit doesn't choose — it holds *both* until you look."

## Strong Mnemonics for ZQL-01

| Flashcard Concept | ZQL-01 Application |
|---|---|
| |0⟩ and |1⟩ are orthogonal | ⟨0|1⟩ = 0 — they are as different as possible |
| "Amplitudes not probabilities" | Square to get probability; gates manipulate amplitudes |
| Born rule: p = \|amplitude\|² | `result.counts` is Born rule sampling |
| H² = I | Two Hadamard gates cancel: `qc.h(0).h(0)` = identity |
| CNOT creates entanglement from |+⟩ | `qc.h(0).cx(0,1)` → Bell state |
| Measurement is irreversible | Never `qc.measure()` mid-circuit if you need the qubit later |

## Quantum Country's Superposition Exercise (adapted for SDK)
```python
# Demonstrate H² = I
from zuntenium import QuantumCircuit, StatevectorSimulator
qc = QuantumCircuit(1)
qc.h(0).h(0).measure_all()           # H applied twice
result = StatevectorSimulator().run(qc, shots=1024)
print(result.counts)   # Always {"0": 1024} — H² = Identity
```

## Teaching Notes on Spaced Repetition
- Quantum Country suggests revisiting concepts 3×: immediately, day 3, day 10.
- For ZQL-01 the recommended review schedule: read §2–§4 once, run exercises same day, re-read Bloch sphere §3 on day 3, re-run capstone on day 7.
