# ZQL-05 Step 1 Notes — Hidary (VQE/QAOA) + Nocedal & Wright (Optimization) + Levine (Chemistry)

---

# A. Hidary — VQE and QAOA Chapters

## VQE Chapter
- **Hidary's framing**: "VQE is quantum-classical ping-pong — the quantum computer measures energy, the classical optimiser serves back better parameters."
- Key steps with SDK mapping:

| VQE Step | Hidary Description | SDK |
|---|---|---|
| 1. Ansatz | Choose parameterised circuit | `QuantumCircuit` + `qc.rx/ry/rz` |
| 2. Prepare | Run ansatz circuit | `StatevectorSimulator.run()` |
| 3. Measure | Estimate Pauli expectations | Basis rotation + measure |
| 4. Sum | ⟨H⟩ = Σcᵢ⟨Pᵢ⟩ | Classical weighted sum |
| 5. Optimise | Update θ to minimise ⟨H⟩ | `scipy.optimize.minimize(COBYLA)` |

## QAOA Chapter
- **Hidary's framing**: "QAOA converts a classical combinatorial problem graph into phases and mixers."
- Key points:
  - The cost Hamiltonian H_C encodes the problem: MaxCut edge (i,j) → ZᵢZⱼ term.
  - Maximising cut = minimising −H_C (flip sign).
  - The mixer Hamiltonian H_B = ΣXᵢ explores the solution space.
  - Increasing p improves quality but exponentially increases classical optimisation cost.

## Parameter-Shift Rule (Hidary §4.5 equivalent)
- For a circuit containing a rotation gate Rₖ(θ): ∂⟨H⟩/∂θ = [⟨H⟩(θ+π/2) − ⟨H⟩(θ−π/2)] / 2.
- Allows exact gradient computation using only circuit evaluations — no finite differences.
- **[SDK]** Requires two additional circuit runs per parameter per iteration.

```python
def parameter_shift_gradient(qc_fn, params, idx, H_fn, shots=4096):
    """
    Compute partial derivative ∂⟨H⟩/∂θ_idx using parameter-shift rule.
    qc_fn(params) → QuantumCircuit
    H_fn(counts) → float (energy estimate from measurement counts)
    """
    import math
    params_plus  = params.copy(); params_plus[idx]  += math.pi / 2
    params_minus = params.copy(); params_minus[idx] -= math.pi / 2
    E_plus  = H_fn(qc_fn(params_plus),  shots)
    E_minus = H_fn(qc_fn(params_minus), shots)
    return (E_plus - E_minus) / 2
```

---

# B. Nocedal & Wright — *Numerical Optimization* (2nd ed., Springer, 2006)

**Classical numerical optimisation — the foundation of the VQE outer loop.**

## Ch. 2 — Unconstrained Optimisation: Fundamentals
- Problem: minimise f(θ) over θ ∈ ℝᵈ (here: d = number of VQE parameters).
- Convergence conditions: ∇f(θ*) = 0 (stationary point), ∇²f(θ*) > 0 (local minimum).
- Gradient descent: θ ← θ − α·∇f(θ). Step size α must be chosen carefully.

## Ch. 6 — Gradient-Free (Derivative-Free) Methods
- **COBYLA** (Constrained Optimisation By Linear Approximations):
  - Builds linear models of f around current point.
  - No gradients needed — key for noisy quantum hardware where exact gradients are unavailable.
  - Convergence: slow for high-dimensional θ (> 100 parameters).
- **Nelder-Mead** (simplex method): robust gradient-free; often first choice for noisy problems.
- **[SDK]** `scipy.optimize.minimize(method='COBYLA')` is the standard VQE optimizer choice.

## Ch. 7 — Quasi-Newton Methods (L-BFGS-B)
- Approximate second-order information using gradient history.
- L-BFGS: limited-memory BFGS. Memory: O(m·d) where m=10 stored vectors, d=number of params.
- **[SDK]** Works well with `StatevectorSimulator` (exact gradients via parameter-shift). Poor on noisy hardware (gradient noise confuses BFGS curvature estimates).

## Ch. 9 — Stochastic Approximation (SPSA)
- **SPSA** (Simultaneous Perturbation Stochastic Approximation):
  - Estimates gradient with only 2 circuit evaluations regardless of d.
  - Perturbs ALL parameters simultaneously by a random ±1 vector.
  - Key formula: gₖ = [f(θ+cₖΔₖ) − f(θ−cₖΔₖ)] / (2cₖΔₖ)
  - **[SDK]** Most hardware-friendly VQE optimizer for high-d parameterised circuits.

## Optimiser Choice Guide for VQE

| Situation | Recommended Optimiser | Rationale |
|---|---|---|
| Simulator (exact) | L-BFGS-B or BFGS | Exact gradients via param-shift; fast convergence |
| Hardware (noisy) | SPSA or COBYLA | Noise-tolerant; few circuit evaluations per step |
| QML (many params) | Adam with param-shift | Stochastic, good for large d |
| First exploration | Nelder-Mead | Simple, gradient-free, good starting point |

---

# C. Levine — *Quantum Chemistry* (5th ed.)

**Background on molecular Hamiltonians — what VQE is actually minimising.**

## What Is a Molecular Hamiltonian?
- **Electronic Hamiltonian** H_el: energy of electrons in a molecule.
- H_el = T_e (kinetic) + V_ee (electron-electron repulsion) + V_ne (nuclear attraction).
- Ground state energy E₀ = min⟨ψ|H_el|ψ⟩ — what VQE finds.

## Second Quantisation (qubit encoding)
- Jordan-Wigner transformation maps fermionic operators to Pauli strings:
  - Fermionic creation operator aₚ† → (Z^{⊗p}) ⊗ (X−iY)/2 ⊗ I^{rest}
- Result: H_el = Σᵢⱼ hᵢⱼ PauliString_ij + ... — a weighted sum of Pauli strings.
- **[SDK]** `PauliHamiltonian` stores exactly this: list of (coefficient, PauliString) pairs.

## Chemical Accuracy
- Target: |E_VQE − E_exact| < 1 kcal/mol ≈ 1.6 mHartree.
- Required for meaningful chemistry predictions.
- VQE on current NISQ hardware meets this for small molecules (H₂, HeH⁺, LiH).

## Simple Example: H₂ Hamiltonian (2 qubits)
The minimal-basis H₂ Hamiltonian (STO-3G) in qubit form is:
```
H = g₀·I + g₁·Z₀ + g₂·Z₁ + g₃·Z₀Z₁ + g₄·X₀X₁ + g₅·Y₀Y₁
```
Coefficients gᵢ depend on internuclear distance R.

```python
# H2 Hamiltonian at R=0.74 Å (equilibrium)
h2_hamiltonian = [
    (-1.0523732,  "II"),   # identity term
    ( 0.3979374,  "IZ"),   # Z on qubit 0
    (-0.3979374,  "ZI"),   # Z on qubit 1
    (-0.0112801,  "ZZ"),   # ZZ coupling
    ( 0.1809312,  "XX"),   # XX correlation
    ( 0.1809312,  "YY"),   # YY correlation
]
# True ground state energy: -1.1373 Hartree
```
