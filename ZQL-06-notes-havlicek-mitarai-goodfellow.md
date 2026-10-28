# ZQL-06 Step 1 Notes — Havlíček 2019 (ZZFeatureMap) + Mitarai 2018 (Param-Shift) + Goodfellow (Deep Learning)

---

# A. Havlíček et al. — *Supervised learning with quantum-enhanced feature spaces* (Nature, 2019)

**The canonical quantum kernel paper. Demonstrated on real IBM hardware.**

## Core Contribution
- Demonstrates QSVM and VQC classifiers on n=2 qubits for a classically-hard separation problem.
- Introduces the **ZZFeatureMap**: a feature map whose inner product is hard to compute classically.

## ZZFeatureMap Definition
The 2-qubit ZZFeatureMap with depth r=2:

```
Layer 1:  H⊗H                    (Hadamard)
          Rz(x₀)⊗Rz(x₁)          (single-qubit phases)  
          CX(0,1)·Rz((π-x₀)(π-x₁), 1)·CX(0,1)  (ZZ entangling phase)
Layer 2:  (same as layer 1, applied again)
```

The ZZ phase (π−x₀)(π−x₁) creates a classically hard-to-simulate quantum kernel.

## Why It's Classically Hard
- The feature-map inner product ⟨φ(x)|φ(x')⟩ involves computing the overlap of a quantum state prepared by a deep circuit.
- For the specific data distribution Havlíček et al. constructed, no efficient classical algorithm can compute this kernel.
- **Key insight**: quantum advantage in ML comes from the hardness of *computing the kernel*, not the *classification*.

## Quantum Advantage Conditions (Havlíček's Theorem)
The quantum classifier outperforms classical SVM only if:
1. The ZZFeatureMap creates an exponentially large feature space that classical kernels cannot span
2. The data lies in a subspace that the quantum kernel separates but classical kernels don't

## Implementation Protocol
1. Prepare |φ(xᵢ)⟩ on circuit (U_φ(xᵢ)|0ⁿ⟩)
2. Prepare U†_φ(xⱼ) (inverse circuit for xⱼ)
3. Apply: run U†_φ(xⱼ)·U_φ(xᵢ)|0ⁿ⟩
4. Count probability of |0ⁿ⟩ = K(xᵢ, xⱼ) = |⟨φ(xⱼ)|φ(xᵢ)⟩|²

**[SDK]**: step 3 uses `feature_map(xi).compose(feature_map(xj).inverse())` then measure.

---

# B. Mitarai et al. — *Quantum circuit learning* (Phys. Rev. A, 2018)

**Primary reference for the parameter-shift rule used in QNN training.**

## The Parameter-Shift Rule (Theorem 1, §III)

For a parameterised gate G(θ) = exp(−iθH/2) where H is Hermitian with eigenvalues ±1:

```
∂⟨M⟩/∂θ = [⟨M⟩(θ + π/2) − ⟨M⟩(θ − π/2)] / 2
```

This is **exact** — not an approximation.

## Proof Sketch
- ⟨M⟩(θ) = ⟨0|U†(θ)MU(θ)|0⟩. With G(θ) = e^{-iθP/2} for Pauli P:
- d/dθ G(θ) = (-iP/2)G(θ). Using the product rule and spectral decomposition:
- ⟨M⟩(θ+π/2) − ⟨M⟩(θ-π/2) = 2·∂⟨M⟩/∂θ.

## Generalised Parameter-Shift
For gates with eigenvalues ±r (not ±1), the shift is π/(4r). For Rz(θ) = e^{-iθZ/2}:
- Eigenvalues of Z: ±1 → shift = π/2. Same as above.
- For decomposed 2-qubit gates: shift can be ±π/4.

## Finite Differences vs Parameter-Shift
| Method | Accuracy | Evals/param | Noise-robust? |
|---|---|---|---|
| Finite diff (Δ→0) | Approximate (O(Δ)) | 2 | No (amplifies noise) |
| Parameter-shift | Exact | 2 | Yes (no limit process) |
| SPSA | Stochastic estimate | 2 (all params) | Most robust |

## QNN Training Algorithm (Mitarai's Protocol)
1. Forward: compute ⟨M⟩(x, θ) for each training point xᵢ
2. Backward: compute ∂⟨M⟩/∂θⱼ via parameter-shift for each parameter j
3. Update: θ ← θ − α·∂L/∂θⱼ (gradient descent)
4. Repeat until convergence

---

# C. Goodfellow, Bengio & Courville — *Deep Learning* (MIT Press, 2016)

**Classical deep learning foundation — required before understanding quantum analogues.**

## Ch. 6 — Feedforward Networks
- Universal approximation theorem: 1 hidden layer + sigmoid can approximate any continuous function.
- **[QML analogy]** Unverified conjecture: QNNs may approximate functions exponentially compactly.

## Ch. 6 — Backpropagation
- Chain rule: ∂L/∂w = ∂L/∂h · ∂h/∂w. Propagates gradients backwards.
- Computational graph: each operation is a node; gradient = product along reverse path.
- **[Quantum]** QNN has no classical computational graph — parameter-shift replaces backprop.

## Ch. 8 — Optimisation for Training
- SGD, Momentum, Adam. Learning rate schedule.
- **[QML]** Adam with parameter-shift = most practical QNN training in ZQL-06.
- Minibatches: compute gradient on random subset of training data → reduces variance per step.

## Ch. 9 — Convolutional Networks (CNN)
- Spatial weight sharing: same filter applied across all positions.
- **[Quantum analogy]** Quantum Convolutional Neural Network (QCNN): same layer applied to all qubit pairs → efficient Clifford-based simulation, fault-tolerant structure.

## Key Classical Concepts Needed for QML
| Classical | Quantum equivalent in ZQL-06 |
|---|---|
| Weight matrix W | Parameterised gate set θ |
| Activation function σ | Measurement + classical post-processing |
| Loss function L | ⟨M⟩ = quantum expectation value |
| Backpropagation | Parameter-shift rule |
| Training set size N | N circuit evaluations |
| Batch gradient | Average ⟨M⟩ over mini-batch |
| Overfitting / regularisation | Expressibility control (reduce depth) |
