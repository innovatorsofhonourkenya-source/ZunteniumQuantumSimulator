# ZQL-01 Step 1 Notes — Kaye, Laflamme & Mosca, *An Introduction to Quantum Computing* (Oxford, 2007)

**Undergraduate-level rigour. Chapters 1–5 most relevant for ZQL-01.**

---

## Ch. 1 — Mathematical Prerequisites

### Complex Numbers
- z = a + bi. Modulus |z|² = a²+b². Conjugate z* = a−bi.
- In quantum: α and β are complex; |α|² = probability.
- Euler: e^{iθ} = cos θ + i sin θ → unit circle in ℂ. Phases are just angles.

### Vectors and Inner Products
- Vector space over ℂ: closed under addition and scalar mult.
- Inner product ⟨u,v⟩ = Σᵢ uᵢ*vᵢ (conjugate-linear in first argument).
- Norm: ||v|| = √⟨v,v⟩. Normalised = unit norm.
- **Orthonormal basis**: ⟨eᵢ,eⱼ⟩ = δᵢⱼ. Any v = Σᵢ ⟨eᵢ,v⟩·eᵢ.

### Matrices and Unitarity
- Conjugate transpose (Hermitian adjoint): (A†)ᵢⱼ = (Aⱼᵢ)*.
- Hermitian: A†=A eigenvalues real.
- Unitary: UU†=I. Columns and rows form orthonormal sets. Preserves norms.
- **Key**: all quantum gates are unitary. The statevector's norm is always preserved.

### Tensor Products
- |ψ₁⟩⊗|ψ₂⟩ = |ψ₁ψ₂⟩. Dimension: dim(H₁)·dim(H₂).
- (A⊗B)(|u⟩⊗|v⟩) = (A|u⟩)⊗(B|v⟩).
- CNOT in tensor product form: |0⟩⟨0|⊗I + |1⟩⟨1|⊗X.
- **[SDK]** `QuantumCircuit(n)` works in ℂ^(2^n). Gate on qubit k = I^{⊗k} ⊗ U ⊗ I^{⊗(n-k-1)}.

---

## Ch. 2 — Quantum States, Operators, and Measurement

### Bloch Sphere (Kaye treatment)
- Parameterise: |ψ⟩ = cos(θ/2)|0⟩ + e^{iφ}sin(θ/2)|1⟩.
- θ=0 → |0⟩; θ=π → |1⟩; θ=π/2, φ=0 → |+⟩; θ=π/2, φ=π → |−⟩; θ=π/2, φ=π/2 → |i⟩.
- Any single-qubit unitary U = e^{iα}Rn̂(θ) for some axis n̂ and angle θ.
- **[SDK intuition]** `qc.rx(theta, 0)` rotates the Bloch vector around the X axis by θ. `qc.h(0)` rotates from Z-axis to X-axis direction (|+⟩).

### Projective Measurement
- Measuring in computational basis: project onto |0⟩ or |1⟩.
- General observable M = Σₘ m|m⟩⟨m| (spectral decomp). ⟨M⟩ = ⟨ψ|M|ψ⟩.
- Uncertainty principle: ΔAΔB ≥ |⟨[A,B]⟩|/2. For X and Z: ΔXΔZ ≥ 1/2.
- **[SDK]** `qc.measure(q,c)` always measures in computational {|0⟩,|1⟩} basis.

---

## Ch. 3 — Quantum Circuits

### Circuit Model
- Circuits have: wires (qubits), gates (unitaries), measurements.
- Time flows left to right. Wires are qubits (not bits).
- Gate depth vs. gate count: depth = critical path (what matters for noise).
- **[SDK]** `qc.depth` = critical path. `qc.num_gates` = total gate count.

### Universality (Kaye §3.5)
- Any unitary U ∈ SU(2ⁿ) can be approximated to ε by a sequence of gates from {H,T,CNOT}.
- Solovay-Kitaev theorem: length O(log^c(1/ε)) gates suffice.
- **[SDK]** `KAKDecompositionPass` implements exact 2-qubit synthesis; single-qubit approximation is hardware's job.

---

## Ch. 5 — Quantum Algorithms Basics

### Quantum Parallelism
- H⊗n applied to |0...0⟩ creates uniform superposition of all 2^n basis states.
- Applying Uf (oracle gate) evaluates f on all inputs simultaneously.
- But: measuring collapses to one result. You need interference to extract useful output.

### Deutsch-Jozsa Algorithm (Kaye §5.3)
- Problem: is f:{0,1}^n →{0,1} constant or balanced? Classical: Ω(2^n) queries. Quantum: 1 query.
- Steps: H^⊗n → Uf → H^⊗n → measure. Constant f → measure all 0s; balanced → at least one 1.
- **[ANALOGY]** ZQL-04 content; here the teaching point is: quantum algorithms compute via global interference, not sequential enumeration.

---

## Key Teaching Points from Kaye

| Concept | Why it Matters for ZQL-01 |
|---|---|
| Tensor product dimension 2^n | Explains why >30 qubits are hard to simulate |
| Bloch sphere parameterisation | Visual mental model for rotation gates (Rx/Ry/Rz) |
| Unitary = norm-preserving | Explains why gates cannot be "lossy" |
| Measurement = projection | Explains post-measurement circuits |
| Solovay-Kitaev | Why gate decomposition is always possible |
