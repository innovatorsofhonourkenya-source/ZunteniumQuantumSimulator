# ZQL-06 Step 1 Notes — Wittek (QML textbook) + Schuld & Petruccione + Biamonte Survey

---

# A. Peter Wittek — *Quantum Machine Learning* (Elsevier, 2018)

**The seminal QML textbook. Freely available online.**

## Part I: Classical ML Foundations

### Supervised Learning (Ch. 2)
- **Classification**: given labelled data {(xᵢ, yᵢ)}, learn f: x → y.
- **Regression**: same but y ∈ ℝ.
- Loss function L(f(x), y) drives training; minimise via gradient descent.
- **[SDK]** Classical ML output labels. QML variant: quantum state |ψ(x,θ)⟩ is measured → classical label.

### Kernel Methods (Ch. 4)
- **Kernel trick**: replace inner product ⟨φ(x), φ(x')⟩ with k(x,x') — efficient when φ is high-dimensional.
- **SVM** decision function: f(x) = Σᵢ αᵢ yᵢ k(xᵢ, x) + b.
- **Quantum kernel**: k_Q(x,x') = |⟨φ(x)|φ(x')⟩|² where |φ(x)⟩ = U_φ(x)|0ⁿ⟩.
- Advantage: quantum feature maps may be classically hard to compute.

### Neural Networks (Ch. 5)
- Feed-forward NN: layers of linear transformations + non-linearities.
- Backpropagation: chain rule to compute ∂L/∂w for all weights w.
- **[SDK]** QNN variant: replace linear layers with parameterised quantum circuits; backprop via parameter-shift rule.

## Part II: Quantum ML Algorithms

### Quantum PCA (Ch. 7)
- HHL algorithm + quantum phase estimation → eigenvalue/eigenvector of density matrix.
- Exponential speedup only for very specific data access models (QRAM).

### Quantum k-Means (Ch. 8)
- Swap test + distance measure in quantum amplitude encoding.
- Quadratic speedup for distance computation via quantum parallelism.

### Quantum SVM (QSVM) — Key Algorithm
- Encode data into quantum states using feature map circuit U_φ(x).
- Compute kernel matrix K[i,j] = |⟨0ⁿ|U†_φ(xᵢ)·U_φ(xⱼ)|0ⁿ⟩|².
- Train classical SVM with quantum kernel matrix.
- **[SDK]** Use `StatevectorSimulator` to compute K[i,j] from circuit overlap:
  ```python
  qc = U_phi_dagger(xi).compose(U_phi(xj))
  p_zero = result.counts.get("00...0", 0) / shots
  kernel_val = p_zero   # |⟨0|U†U|0⟩|² = overlap²
  ```

---

# B. Schuld & Petruccione — *Machine Learning with Quantum Computers* (Springer, 2022)

**Advanced, rigorous — covers kernel methods, variational classifiers, QNNs.**

## Quantum Feature Maps (Ch. 5)
- A feature map Φ: ℝᵐ → Hilbert space. Implemented as quantum circuit U_Φ(x).
- **ZZFeatureMap** (Havlíček et al.): two layers of:
  1. H on each qubit (Hadamard layer)  
  2. Rz(xᵢ) on each qubit (encode each feature as phase)
  3. CX(i,j) + Rz((π−xᵢ)(π−xⱼ), j) + CX(i,j) for each pair (i,j) — the "ZZ" term
- **[SDK]** Decomposed as: `qc.h(i)`, `qc.rz(x[i], i)`, `qc.cx(i,j)`, `qc.rz(..., j)`, `qc.cx(i,j)`.

## Quantum Neural Networks (Ch. 6)
- Parameterised quantum circuit U(θ) as a trainable model.
- Forward pass: compute ⟨ψ(x,θ)|M|ψ(x,θ)⟩ (expectation value of observable M).
- Loss function: L = Σᵢ (yᵢ − ⟨M⟩(xᵢ,θ))².
- Gradient: ∂L/∂θⱼ via parameter-shift rule.

## Expressibility and Entangling Power
- **Expressibility**: how well U(θ) covers the Haar-random unitary space.
- **Entangling power**: how much entanglement U(θ) generates.
- More expressive ansatz → wider coverage but more susceptible to barren plateaus.
- **[SDK]** HEA (hardware-efficient ansatz) has tunable expressibility via depth.

## Quantum Transfer Learning
- Pre-train classical neural network (ResNet, etc.) → extract features.
- Feed features into a shallow QNN (quantum layer) for downstream task.
- Advantage: classical power + quantum expressibility for small datasets.

---

# C. Biamonte et al. — *Quantum Machine Learning* (Nature, 2017)

**The most cited QML survey. Sets the landscape.**

## Key QML Problems & Speedups
| Problem | Classical | Quantum | Speedup |
|---|---|---|---|
| Linear systems (HHL) | O(n) | O(log n) | Exponential (with QRAM) |
| Linear regression | O(n³) | O(log n) | Exponential (with QRAM) |
| k-means clustering | O(kdn) | O(k√n) | Quadratic |
| SVM kernel matrix | O(n²·gate) | O(n log n) via QRAM | Polynomial |

## QRAM Caveat
- Most exponential QML speedups require QRAM (quantum random access memory).
- QRAM: load arbitrary classical data into a quantum superposition.
- **Problem**: QRAM is not yet experimentally realised at scale. Claims of exponential speedup for actual ML datasets are conditional.

## Near-Term QML (NISQ-compatible)
- Variational quantum classifiers (VQC): NISQ-compatible, no QRAM needed.
- Quantum kernel methods: NISQ-compatible (compute |⟨φ(x)|φ(x')⟩|² via Hadamard test).
- These are the practical ZQL-06 focus.

## Dequantisation (Le Gall/Tang results)
- Some QML algorithms have classical counterparts that achieve similar (polylogarithmic) runtimes given classical sample access.
- Lesson: quantum speedup in ML claims must be carefully verified. Variational QML has no such classical simulation.
