# ZQL-05 Step 1 Notes — Peruzzo 2014 (VQE) + Farhi 2014 (QAOA) + Bharti 2022 (Review)

---

# A. Peruzzo et al. — *A variational eigenvalue solver on a photonic quantum processor* (Nature Comm., 2014)

**The founding VQE paper. DOI: 10.1038/ncomms5213**

## Core Idea
- **Problem**: Find the ground-state energy of a quantum Hamiltonian H (e.g., a molecular Hamiltonian).
- Classical approach: full configuration interaction (FCI) — exact but exponentially expensive.
- VQE approach: use a parameterised quantum state |ψ(θ)⟩ and minimise ⟨ψ(θ)|H|ψ(θ)⟩ classically.

## The Variational Principle (physics basis)
- For any normalised |ψ⟩: ⟨ψ|H|ψ⟩ ≥ E_ground (the ground state energy).
- Equality holds iff |ψ⟩ = |ψ_ground⟩.
- So: minimising ⟨H⟩ over all reachable |ψ(θ)⟩ finds the best available approximation to the ground state.

## Algorithm
1. Choose ansatz: |ψ(θ)⟩ = U(θ)|0ⁿ⟩ (parameterised quantum circuit)
2. Prepare |ψ(θ)⟩ on quantum hardware
3. Measure ⟨H⟩ = Σᵢ cᵢ ⟨Pᵢ⟩ (H decomposed into Pauli strings with coefficients)
4. Classical optimiser updates θ to reduce ⟨H⟩
5. Repeat until convergence

## VQE Components

| Component | Description | SDK |
|---|---|---|
| Ansatz U(θ) | Parameterised circuit | `qc.rx/ry/rz(theta, q)`, `qc.u(θ,φ,λ,q)` |
| Hamiltonian H | Sum of weighted Pauli strings | `PauliHamiltonian` class |
| Expectation ⟨Pᵢ⟩ | Basis rotation + measurement | X→H·Z·H, Y→S†·H·Z·H·S |
| Classical loop | Minimise ⟨H⟩(θ) | `scipy.optimize.minimize` or built-in |

## Key Result
- Demonstrated for HeH⁺ on photonic hardware. Ground state energy within chemical accuracy (< 1.6 mHartree error).

---

# B. Farhi, Goldstone & Gutmann — *A Quantum Approximate Optimization Algorithm* (2014, arXiv:1411.4028)

## Core Idea
- **Problem**: Combinatorial optimisation — e.g., MaxCut: partition graph into two sets to maximise cut edges.
- QAOA: a parameterised quantum circuit that approximates the solution quality, controlled by depth p.
- As p→∞, QAOA converges to the exact solution (adiabatic limit).

## The Circuit Structure
- **Problem unitary** C: implements the cost Hamiltonian H_C in the exponent.
  - e^{-iγH_C}: applies e^{-iγZᵢZⱼ} for each edge (i,j).
  - Implemented with `qc.rzz(2*gamma, i, j)` or `qc.cx(i,j).rz(2*gamma, j).cx(i,j)`.
- **Mixer unitary** B: applies e^{-iβH_B} where H_B = Σᵢ Xᵢ (transverse field).
  - Implemented with `qc.rx(2*beta, i)` for each qubit i.

## QAOA(p) Circuit
```
|ψ⟩ = e^{-iβ_p H_B} · e^{-iγ_p H_C} · ... · e^{-iβ₁ H_B} · e^{-iγ₁ H_C} · |+⟩^n
```
Parameters: θ = (γ₁,...,γ_p, β₁,...,β_p) — 2p real numbers to optimise.

## Approximation Ratio
- QAOA(p=1) achieves guarantee ≥ 0.6924 for MaxCut on 3-regular graphs.
- Higher p → better approximation; QAOA(∞) = optimal.

## SDK Mapping
```python
# One QAOA layer for MaxCut on edge (i,j):
def qaoa_cost_layer(n, edges, gamma):
    qc = QuantumCircuit(n)
    for (i, j) in edges:
        qc.cx(i, j)              # Decomposed RZZ
        qc.rz(2 * gamma, j)
        qc.cx(i, j)
    return qc

def qaoa_mixer_layer(n, beta):
    qc = QuantumCircuit(n)
    for i in range(n):
        qc.rx(2 * beta, i)       # Transverse field mixer
    return qc
```

---

# C. Bharti et al. — *Noisy Intermediate-Scale Quantum (NISQ) Algorithms* (Rev. Mod. Phys., 2022)

**Comprehensive NISQ survey: 100+ pages covering VQA landscape exhaustively.**

## Key VQA Taxonomy
1. **VQE variants**: Hardware-Efficient Ansatz (HEA), UCCSD ansatz, ADAPT-VQE
2. **QAOA and extensions**: Warm-start QAOA, recursive QAOA (RQAOA), multi-angle QAOA
3. **Quantum Machine Learning**: QNN, quantum kernel methods, parameterised circuits for classification
4. **Quantum Approximate Simulation**: variational quantum simulation of dynamics

## Barren Plateaus (critical NISQ problem)
- Problem: gradient ∂⟨H⟩/∂θᵢ becomes exponentially small for random ansätze on n qubits.
- Symptoms: training loss doesn't improve; all parameter gradients ≈ 0.
- Mitigation: problem-inspired ansätze, layer-by-layer training, gradient-aware optimisers.
- **[SDK]** Barren plateaus affect any deep `qc.rx/ry/rz` layer circuit with random initialisation.

## Classical Optimisers in VQA
| Optimiser | Type | Use Case |
|---|---|---|
| COBYLA | Gradient-free | Small θ, noisy hardware |
| Nelder-Mead | Gradient-free | General purpose, noise-tolerant |
| BFGS/L-BFGS-B | Gradient-based | Simulator environments |
| SPSA | Stochastic gradient | Noisy hardware |
| Adam | Stochastic gradient | QML tasks |

## Shot Noise and Measurement Cost
- P(outcome) estimated from finite shots → statistical error ∝ 1/√shots.
- For m Pauli terms in H each requiring s shots: total = m·s shots per VQE iteration.
- **[SDK]** `StatevectorSimulator.run(qc, shots=N)` controls shot count per evaluation.

## Hardware-Efficient Ansatz (HEA)
- Layer structure: single-qubit Ry/Rz rotations + CNOT entangling layers.
- Advantage: native to hardware gate set, minimal SWAP overhead.
- Disadvantage: expressibility doesn't match chemistry Hamiltonians → may have barren plateaus.
```python
def hardware_efficient_ansatz(n, depth, params):
    """Build HEA: alternating Ry(θ) rotation layers + CNOT entanglement."""
    qc = QuantumCircuit(n)
    idx = 0
    for d in range(depth):
        for i in range(n):
            qc.ry(params[idx], i)
            idx += 1
        for i in range(n - 1):
            qc.cx(i, i + 1)    # Linear entanglement
    return qc
```
