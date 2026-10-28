# ZQL-11 Step 1 Notes — Li & Benjamin (ZNE) + van den Berg (PEC) + Cai et al. 2023 (QEM Review)

---

# A. Li & Benjamin — *Efficient Variational Quantum Simulator Incorporating Active Error Minimisation* (Phys. Rev. X, 2017)

**Original ZNE paper.**

## Zero Noise Extrapolation (ZNE)

### Core Idea
If we can run circuits at multiple noise levels λ₁ < λ₂ < λ₃, we observe:
```
⟨O⟩(λ) = ⟨O⟩_ideal + c₁λ + c₂λ² + ...
```
Extrapolate to λ→0 to estimate ⟨O⟩_ideal.

### Noise Amplification Methods
1. **Gate folding**: replace gate G → G·G†·G (3× folding, noise ≈ 3×).
   - r-fold: replace G → (G·G†)^{r-1}·G (noise ≈ (2r-1)×).
   - **[SDK]** Fold each gate in circuit by inserting G†G pairs.
2. **Circuit stretching**: insert identity layers (noisy I = X·X on hardware).
3. **Pulse stretching**: stretch pulses in time (hardware-level).

### Extrapolation Methods
- **Linear**: fit ⟨O⟩ = a + b·λ, extrapolate to λ=0. Uses 2 noise levels.
- **Richardson extrapolation**: exact polynomial of degree n uses n+1 noise levels.
  ```
  ⟨O⟩_ZNE = Σᵢ cᵢ⟨O⟩(λᵢ)   with Σcᵢ=1, Σcᵢλᵢ^k=0 for k=1..n
  ```
- **Exponential**: fit ⟨O⟩(λ) = A·e^{-Bλ} + C. Useful for depolarising noise.

### ZNE Costs
- Circuit evaluations: (number of noise levels) × (number of shots).
- Gate overhead: circuit with folding ratio r has ~r× more gates.

---

# B. van den Berg et al. — *Probabilistic Error Cancellation with Sparse Pauli-Lindblad Models* (Nature Physics, 2023)

**State-of-the-art PEC implementation.**

## Probabilistic Error Cancellation (PEC)

### Core Idea
Express ideal gate G as a quasi-probability distribution over noisy gates:
```
G_ideal = Σᵢ γᵢ Gᵢ_noisy    (quasi-probability decomposition)
Σ|γᵢ| = γ   (sampling overhead, γ ≥ 1)
```

### γ and Sampling Overhead
- γ = Σ|γᵢ| is the 1-norm of quasi-probabilities.
- Each circuit run samples one noisy implementation {Gᵢ} with sign s_i = sign(γᵢ).
- After many samples: ⟨O⟩_ideal ≈ γ × avg(sᵢ × ⟨O_i⟩).
- Variance: increases by γ² → need γ²× more shots.
- For n-gate circuit with per-gate overhead γ₀: total overhead γ = γ₀^n (exponential in circuit depth).

### Sparse Pauli-Lindblad Model (van den Berg)
- Noise model: Λ(ρ) = Σₖ rₖ Pₖ ρ Pₖ, where Pₖ are k-sparse Pauli strings.
- Learn noise rates rₖ from randomised benchmarking data.
- Result: compact noise description → efficient PEC quasi-probabilities.

### When to Use ZNE vs PEC
| Property | ZNE | PEC |
|---|---|---|
| Bias | Residual (model error) | Unbiased (exact) |
| Variance overhead | Low | High (γ²×) |
| Requires noise model | No | Yes |
| NISQ practical | ✓ Most practical | ✓ With learned model |
| Circuit depth limit | Low-medium | Medium |

---

# C. Cai, Babbush et al. — *Quantum Error Mitigation* (Rev. Mod. Phys., 2023)

**Comprehensive review of all error mitigation methods.**

## QEM Taxonomy

| Method | Idea | Overhead | Bias |
|---|---|---|---|
| ZNE | Noise amplification + extrapolation | Low-medium | Residual |
| PEC | Quasi-prob decomposition of ideal gate | Exponential in depth | None |
| Clifford Data Regression (CDR) | Learn noise from Clifford circuits → apply to target | Medium | Residual |
| SEM (Symmetry Expansion) | Project density matrix onto symmetry eigenspace | Low | Residual |
| Virtual Distillation | Combine purified copies of noisy ρⁿ | Medium | Residual |
| Readout Correction | Invert measurement confusion matrix A | Low | None if A known |
| DDSM | Mitigation via dual-state ρ⊗|0⟩ probe | Low | Residual |

## Readout Error Correction

The simplest form of error mitigation: correct measurement confusion.

```
Measure calibration circuits |0⟩ⁿ and |1⟩ⁿ → confusion matrix A.
A[i,j] = P(measure i | prepared j)
p_noisy = A × p_ideal → p_ideal = A⁻¹ × p_noisy
```

**[SDK]** Accessible via post-processing of `result.counts` with a calibration matrix.

## Clifford Data Regression (CDR)
1. Pick near-Clifford circuits (replace few T gates with Clifford approximations).
2. Both noisy and noiseless (via stabilizer simulator) evaluations are cheap.
3. Learn linear regression: ⟨O⟩_ideal ≈ a × ⟨O⟩_noisy + b.
4. Apply to target circuit.

**[SDK]** `StabilizerSimulator` enables noiseless Clifford simulation for CDR training data.
