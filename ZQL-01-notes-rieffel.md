# ZQL-01 Step 1 Notes — Rieffel & Polak, *Quantum Computing: A Gentle Introduction* (MIT Press, 2011) — Chapters 1–4

---

## Ch. 1 — From Classical Bits to Quantum Bits

### Classical Foundations
- Classical state: n bits → 2^n possible configurations (only one active at a time).
- Classical gate: deterministic Boolean function f: {0,1}^n → {0,1}^m.
- **Key contrast**: quantum gates are *linear* and *reversible*; classical NAND/AND are not.

### The Qubit
- Rieffel & Polak frame: "a qubit is a two-state quantum system with a unit vector as state."
- Notation: |0⟩=(1,0)ᵀ, |1⟩=(0,1)ᵀ. State |ψ⟩ = α|0⟩ + β|1⟩.
- Global phase invariance: e^{iγ}|ψ⟩ is physically identical to |ψ⟩. Only *relative* phases matter.

### Dirac Notation (Rieffel approach)
- Bra-ket: ket |v⟩ is the state; bra ⟨v| is its dual. Inner product ⟨u|v⟩.
- Outer product |u⟩⟨v| = matrix.
- Braket: ⟨u|A|v⟩ = matrix element (expectation value or transition amplitude).

---

## Ch. 2 — Two Qubits and Entanglement

### Two-Qubit States
- Basis: {|00⟩, |01⟩, |10⟩, |11⟩} — 4 amplitudes.
- Product (separable): |ψ⟩ = |a⟩⊗|b⟩ = (α₀|0⟩+α₁|1⟩)⊗(β₀|0⟩+β₁|1⟩).
- Entangled: cannot be written as product. Example: (|00⟩+|11⟩)/√2.

### CNOT and Entanglement Creation
- CNOT unentangles in one basis and entangles in another.
- H⊗I followed by CNOT creates Bell state from |00⟩.
- Rieffel's test for entanglement: reduced density matrix ρ_A = Tr_B(ρ_AB) is mixed iff entangled.
- **[SDK]** Partial trace not in direct API (v0.2.0 `[TO CONFIRM]`), but EntanglementDetector could be built from `DensityMatrixSimulator`.

---

## Ch. 3 — Quantum Gates

### Gate Zoo (Rieffel)
- Phase gates: generalisation Rz(θ) = diag(1, e^{iθ}). S = Rz(π/2); T = Rz(π/4).
- Controlled-U: applies U to target if control = |1⟩. General two-qubit gate family.
- U3(θ,φ,λ): composed of Rz·Ry·Rz. Spans all SU(2).
- **[SDK]**: All these are implemented: `qc.rz/ry/rx`, `qc.u(theta,phi,lam,q)`, `qc.cx/cz/cy/crx/cry/crz`.

### Gate Composition Rules
- Gates compose as matrix products: (U₁ applied first, U₂ applied second) → U₂·U₁ on the state vector.
- Circuit left-to-right = matrix right-to-left.
- **[SDK]** `QuantumCircuit._instructions` list order = circuit left-to-right = application order.

---

## Ch. 4 — Quantum Algorithms (Introduction)

### Quantum Interference Patterns
- Rieffel's framing: "the art of quantum algorithm design is to set up an interference pattern that amplifies the amplitude of the correct answer."
- Constructive interference: amplitudes add → larger probability.
- Destructive interference: amplitudes cancel → smaller probability.

### Quantum Speedup Sources
1. **Superposition**: evaluate f on exponentially many inputs simultaneously.
2. **Entanglement**: create non-local correlations that encode global structure.
3. **Interference**: steer amplitudes so correct answer dominates.

### Implications for SDK Users
- You're not "running many computations in parallel" — you're choreographing an *amplitude ballet*.
- Measurement collapses everything. Design algorithms to interfere *before* measuring.
- **[SDK]** `StatevectorSimulator` shows you the full amplitude vector for debugging; real hardware only gives you measurement outcomes.

---

## Key Teaching Points from Rieffel & Polak

| Book Concept | ZQL-01 Teaching Point |
|---|---|
| Global phase invariance | Why e^{iγ}H = H in effect; phases only matter relatively |
| Reduced density matrix | Formal definition of entanglement |
| Gate left-to-right vs. matrix right-to-left | Critical for reading circuit diagrams correctly |
| "Amplitude ballet" framing | Mental model for algorithm design (ZQL-04/05) |
| Controlled-U family | Foundation for VQE parameterised gates (CRx, CRy, CRz) |
