# ZQL-06 — Quantum Machine Learning (QML)
# Quantum Kernels, QSVM, Quantum Neural Networks, Parameter-Shift Rule

> **Course ID**: ZQL-06 | **Track**: ZQL-05 → ZQL-06  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisites**: ZQL-01 (statevectors, inner products), ZQL-05 (parameterised circuits, parameter-shift)

---

## 0. Course Overview

### Who This Is For

You understand VQE and QAOA from ZQL-05. This course applies those ideas to **machine learning**: supervised classification using quantum feature maps and quantum neural networks.

**Prerequisites**:
- ZQL-01: inner products ⟨φ|ψ⟩, Bloch sphere, unitary gates
- ZQL-05: parameterised Rx/Ry/Rz circuits, parameter-shift rule, variational loops
- Classical ML basics: what a classifier, loss function, and training are

### Learning Outcomes

1. Encode classical data into quantum states using feature map circuits
2. Compute a quantum kernel matrix K[i,j] = |⟨φ(xᵢ)|φ(xⱼ)⟩|²
3. Train a classical SVM with a quantum kernel (QSVM)
4. Build a Variational Quantum Classifier (VQC) ansatz
5. Implement forward and backward pass of a QNN using parameter-shift
6. Explain when quantum kernels may outperform classical kernels
7. Identify the QRAM caveat and when QML speedup claims break down

### Estimated Time: ~150 minutes

| Section | Content | Time |
|---|---|---|
| §1 | Classical ML Recap | 15 min |
| §2 | Quantum Feature Maps | 20 min |
| §3 | Quantum Kernel & QSVM | 25 min |
| §4 | Variational Quantum Classifier | 20 min |
| §5 | Quantum Neural Network (QNN) | 25 min |
| §6 | Parameter-Shift Backprop | 15 min |
| §7 | QRAM Caveat & Speedup Reality | 10 min |
| §8 | Exercises + Quiz + Capstone | 20 min |

### Source Materials

| Ref | Book / Paper |
|---|---|
| W | Wittek — *Quantum Machine Learning* (2018) |
| SP | Schuld & Petruccione — *ML with Quantum Computers* (2022) |
| B | Biamonte et al. — *Quantum Machine Learning* (Nature, 2017) |
| H | Havlíček et al. — *Supervised learning with quantum feature spaces* (Nature, 2019) |
| M | Mitarai et al. — *Quantum circuit learning* (Phys. Rev. A, 2018) |
| G | Goodfellow, Bengio & Courville — *Deep Learning* (MIT Press, 2016) |

---

## 1. Classical Machine Learning — Quick Recap

### 1.1 Supervised Classification

```python
# Classical: given labelled data, learn a decision boundary
import numpy as np

# Generate a simple 2D binary classification dataset
np.random.seed(42)
n_samples = 40

# Class 0: points near (−1, −1)
X_class0 = np.random.randn(n_samples // 2, 2) * 0.4 + np.array([-1, -1])
# Class 1: points near (+1, +1)
X_class1 = np.random.randn(n_samples // 2, 2) * 0.4 + np.array([+1, +1])

X = np.vstack([X_class0, X_class1])
y = np.array([-1] * (n_samples // 2) + [+1] * (n_samples // 2))

# Classical RBF SVM kernel: k(x,x') = exp(-||x-x'||²/(2σ²))
from sklearn.svm import SVC
svm_rbf = SVC(kernel='rbf', gamma=1.0)
svm_rbf.fit(X, y)
acc_rbf = svm_rbf.score(X, y)
print(f"Classical RBF SVM accuracy: {acc_rbf:.2%}")
```

### 1.2 The Kernel Trick

The kernel trick replaces the explicit feature map φ(x) with a similarity function k(x,x') = ⟨φ(x), φ(x')⟩. The SVM decision function becomes:

```
f(x) = sign(Σᵢ αᵢ yᵢ k(xᵢ, x) + b)
```

**Quantum possibility**: if we define φ(x) as a quantum state prepared by a circuit U_φ(x)|0ⁿ⟩, then:
- k_Q(x,x') = |⟨φ(x)|φ(x')⟩|² = |⟨0|U†_φ(x)·U_φ(x')|0⟩|²

This quantum kernel lives in an exponentially large Hilbert space — potentially beyond any classical kernel.

---

## 2. Quantum Feature Maps

### 2.1 Data Encoding Strategies

Before computing a quantum kernel, we must encode classical data into quantum states.

```python
from zuntenium import QuantumCircuit
import numpy as np

# Strategy 1: Angle Encoding
# Encode each feature xᵢ as a rotation angle on one qubit
def angle_encoding(x: list) -> QuantumCircuit:
    """
    Encode n features x = [x0,...,x_{n-1}] as Ry rotations.
    1 qubit per feature.
    """
    n = len(x)
    qc = QuantumCircuit(n, name="angle_enc")
    for i, xi in enumerate(x):
        qc.ry(xi, i)    # Ry(xᵢ)|0⟩ = cos(xᵢ/2)|0⟩ + sin(xᵢ/2)|1⟩
    return qc

# Strategy 2: Amplitude Encoding (conceptual — requires state prep)
# Encode n features into 2^k amplitudes on k=⌈log₂n⌉ qubits
# Normalised: ||x||=1 → amplitudes directly
# [TO CONFIRM] QuFlow v0.2.0 has no direct `.initialize(state_vec)` API

# Strategy 3: ZZFeatureMap (Havlíček)
# Encodes 2 features into 2 qubits with entangling ZZ interaction

def zz_feature_map(x: list, depth: int = 2) -> QuantumCircuit:
    """
    ZZFeatureMap for n = len(x) qubits.
    Layer structure: H → Rz(xᵢ) → CX·Rz((π-xᵢ)(π-xⱼ))·CX (for all pairs i<j)
    Repeated `depth` times.

    Reference: Havlíček et al., Nature 2019.
    SDK gates: qc.h, qc.rz, qc.cx
    """
    n = len(x)
    qc = QuantumCircuit(n, name=f"ZZFeatMap_d{depth}")
    x = np.array(x)

    for _ in range(depth):
        # Hadamard layer
        for i in range(n):
            qc.h(i)
        # Single-qubit phase encoding
        for i in range(n):
            qc.rz(2 * x[i], i)   # factor 2 matches Havlíček convention
        # Pairwise ZZ entangling terms
        for i in range(n):
            for j in range(i + 1, n):
                # e^{-i(π-xᵢ)(π-xⱼ) ZᵢZⱼ} = CX · Rz(2(π-xᵢ)(π-xⱼ)) · CX
                angle = 2 * (np.pi - x[i]) * (np.pi - x[j])
                qc.cx(i, j)
                qc.rz(angle, j)
                qc.cx(i, j)

    return qc

# Visualise
x_sample = [1.2, 0.7]
print(f"ZZFeatureMap for x={x_sample}:")
print(zz_feature_map(x_sample, depth=1))
```

### 2.2 Feature Map Properties

| Feature Map | Qubits | Depth | Classical simulable? |
|---|---|---|---|
| Angle encoding | n | O(1) | ✓ (product state) |
| ZZFeatureMap (d=1) | n | O(n²) | ✓ (low-depth) |
| ZZFeatureMap (d=2+) | n | O(d·n²) | Plausibly ✗ |
| Random Clifford map | n | O(n²) | ✓ (stabilizer) |
| Random universal circuit | n | O(poly n) | ✗ expected |

---

## 3. Quantum Kernel and QSVM

### 3.1 Computing the Quantum Kernel

The quantum kernel K(xᵢ, xⱼ) = |⟨0ⁿ|U†_φ(xᵢ)·U_φ(xⱼ)|0ⁿ⟩|²

is computed by:
1. Preparing the inverse feature map for xᵢ: `fm(xi).inverse()`
2. Composing with the feature map for xⱼ: `fm(xj)`
3. Running the combined circuit and measuring |0ⁿ⟩ probability

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np

backend = StatevectorSimulator()

def quantum_kernel(xi: list, xj: list, depth: int = 2, shots: int = 8192) -> float:
    """
    Compute K(xi, xj) = |⟨0|U†_φ(xi)·U_φ(xj)|0⟩|²
    using the ZZFeatureMap encoding.
    """
    n = len(xi)
    assert len(xj) == n, "Feature vectors must have same dimension"

    # Build: U†_φ(xi) · U_φ(xj)
    fm_xi = zz_feature_map(xi, depth)
    fm_xj = zz_feature_map(xj, depth)

    # Full circuit: |0⟩ → U_φ(xj) → U†_φ(xi) → measure
    qc = QuantumCircuit(n, name="kernel_circuit")
    qc.compose(fm_xj)
    qc.compose(fm_xi.inverse())   # [TO CONFIRM] inverse() on ZZFeatureMap
    qc.measure_all()

    result = backend.run(qc, shots=shots)
    # Probability of measuring |0...0⟩
    zero_state = "0" * n
    p_zero = result.counts.get(zero_state, 0) / shots
    return p_zero   # = K(xi, xj)

# Quick test: K(x, x) should be ≈ 1.0 (self-overlap)
x_test = [0.8, 1.3]
k_self = quantum_kernel(x_test, x_test)
print(f"K(x, x) = {k_self:.4f}  (should be ≈ 1.000)")

# K(x, y) for different points
y_test = [2.1, 0.5]
k_xy = quantum_kernel(x_test, y_test)
print(f"K(x, y) = {k_xy:.4f}  (should be < 1.000)")
```

### 3.2 Building the Kernel Matrix

```python
def build_kernel_matrix(X: np.ndarray, depth: int = 2,
                         shots: int = 4096) -> np.ndarray:
    """
    Compute full n×n quantum kernel matrix K[i,j] = quantum_kernel(X[i], X[j]).
    Exploits K[i,j] = K[j,i] symmetry: O(n²/2) circuits.
    """
    n = len(X)
    K = np.eye(n)    # K[i,i] = 1 (self-overlap exactly)

    print(f"Computing {n*(n-1)//2} kernel entries (n={n})...")
    for i in range(n):
        for j in range(i + 1, n):
            k_val = quantum_kernel(list(X[i]), list(X[j]), depth=depth, shots=shots)
            K[i, j] = K[j, i] = k_val

    return K

# Small dataset: 8 points
X_small = np.vstack([
    np.random.randn(4, 2) * 0.4 + np.array([-1, -1]),
    np.random.randn(4, 2) * 0.4 + np.array([+1, +1]),
])
y_small = np.array([-1] * 4 + [+1] * 4)

K_train = build_kernel_matrix(X_small, depth=1, shots=2048)

print("\nQuantum kernel matrix (8×8 excerpt):")
print(np.round(K_train, 3))
```

### 3.3 Training QSVM with Precomputed Kernel

```python
from sklearn.svm import SVC
from sklearn.model_selection import cross_val_score

# Train SVM with precomputed quantum kernel
qsvm = SVC(kernel='precomputed')
qsvm.fit(K_train, y_small)

# Cross-validation score
scores = cross_val_score(qsvm, K_train, y_small, cv=4, scoring='accuracy')
print(f"\nQSVM cross-val accuracy: {scores.mean():.2%} ± {scores.std():.2%}")

# Compare with classical RBF
svm_rbf = SVC(kernel='rbf', gamma=1.0)
rbf_scores = cross_val_score(svm_rbf, X_small, y_small, cv=4, scoring='accuracy')
print(f"Classical RBF accuracy:  {rbf_scores.mean():.2%} ± {rbf_scores.std():.2%}")
```

---

## 4. Variational Quantum Classifier (VQC)

### 4.1 Architecture

A VQC combines:
1. **Feature map** U_φ(x): encodes data
2. **Ansatz** U_θ(θ): trainable parameterised circuit
3. **Measurement**: expectation value ⟨Z₀⟩ → classification score

```
x → [Feature Map U_φ(x)] → [Ansatz U_θ(θ)] → measure ⟨Z₀⟩ → sign → label
```

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np

def vqc_ansatz(n: int, params: list) -> QuantumCircuit:
    """
    Hardware-Efficient Ansatz for VQC.
    Single Ry layer (n params) + CNOT ladder.
    """
    qc = QuantumCircuit(n, name="VQC_ansatz")
    for i in range(n):
        qc.ry(params[i], i)
    for i in range(n - 1):
        qc.cx(i, i + 1)
    return qc

def vqc_circuit(x: list, params: list, depth: int = 1) -> QuantumCircuit:
    """Full VQC circuit: feature map + ansatz."""
    n = len(x)
    qc = QuantumCircuit(n, name="VQC")
    # Encode data
    qc.compose(zz_feature_map(x, depth=1))
    # Apply trainable ansatz
    qc.compose(vqc_ansatz(n, params))
    return qc

def vqc_predict_score(x: list, params: list, shots: int = 4096) -> float:
    """
    Compute VQC output: ⟨Z₀⟩ ∈ [−1, +1].
    Positive → predict +1; negative → predict −1.
    """
    qc = vqc_circuit(x, params)
    qc.measure_all()
    result = StatevectorSimulator().run(qc, shots=shots)

    # ⟨Z₀⟩ = P(q0=0) − P(q0=1)
    expectation = 0.0
    for bs, count in result.counts.items():
        q0_bit = int(bs[-1])   # LSB = qubit 0 [TO CONFIRM]
        sign = +1 if q0_bit == 0 else -1
        expectation += sign * (count / shots)
    return expectation

def vqc_predict(x: list, params: list) -> int:
    """Classify: +1 or -1."""
    return +1 if vqc_predict_score(x, params) >= 0 else -1
```

### 4.2 Training the VQC

```python
import scipy.optimize as opt

def vqc_loss(params: np.ndarray, X_tr: np.ndarray,
              y_tr: np.ndarray, shots: int = 2048) -> float:
    """MSE loss over training data."""
    total = 0.0
    for xi, yi in zip(X_tr, y_tr):
        score = vqc_predict_score(list(xi), list(params), shots)
        total += (yi - score) ** 2
    return total / len(X_tr)

def vqc_gradient(params: np.ndarray, X_tr: np.ndarray,
                  y_tr: np.ndarray, shots: int = 2048) -> np.ndarray:
    """Compute gradient via parameter-shift rule."""
    grads = np.zeros_like(params)
    for k in range(len(params)):
        p_plus  = params.copy(); p_plus[k]  += np.pi / 2
        p_minus = params.copy(); p_minus[k] -= np.pi / 2
        loss_p = vqc_loss(p_plus,  X_tr, y_tr, shots)
        loss_m = vqc_loss(p_minus, X_tr, y_tr, shots)
        grads[k] = (loss_p - loss_m) / 2
    return grads

# Use small dataset
n_features = 2
params_0 = np.random.uniform(0, np.pi, n_features)

print("Training VQC...")
E_history = []
def callback(params):
    L = vqc_loss(params, X_small, y_small, shots=1024)
    E_history.append(L)
    if len(E_history) % 5 == 0:
        print(f"  Step {len(E_history):3d}: Loss = {L:.4f}")

result_vqc = opt.minimize(
    vqc_loss,
    x0=params_0,
    args=(X_small, y_small, 1024),
    jac=vqc_gradient,
    method='L-BFGS-B',
    options={'maxiter': 30},
    callback=callback,
)

opt_params = result_vqc.x
# Evaluate accuracy
preds = [vqc_predict(list(X_small[i]), opt_params) for i in range(len(X_small))]
acc = np.mean(np.array(preds) == y_small)
print(f"\nVQC training accuracy: {acc:.2%}")
```

---

## 5. Quantum Neural Network (QNN)

### 5.1 QNN Architecture

A QNN generalises VQC by:
- Multiple parameterised layers (alternating feature encoding + ansatz)
- Multiple output measurements (multi-class or regression)
- Data re-uploading: encoding x at each layer (allows non-linear separation)

```python
def qnn_circuit(x: list, params: list,
                n_layers: int = 2, n_qubits: int = 2) -> QuantumCircuit:
    """
    Data re-uploading QNN.
    Each layer: encode x as Ry rotations, then trainable Ry + CNOT.
    params shape: (n_layers × n_qubits × 1) trainable + (n_layers × n_qubits) encoding
    """
    qc = QuantumCircuit(n_qubits, name=f"QNN_{n_layers}L")
    n_params_per_layer = n_qubits
    x_arr = np.array(x)

    for layer in range(n_layers):
        # Data encoding (re-upload at each layer)
        for i in range(n_qubits):
            feature_idx = i % len(x)
            qc.ry(x_arr[feature_idx], i)

        # Trainable rotation layer
        param_start = layer * n_params_per_layer
        for i in range(n_qubits):
            qc.ry(params[param_start + i], i)

        # Entanglement
        for i in range(n_qubits - 1):
            qc.cx(i, i + 1)

    return qc

# n_qubits=2, n_layers=3 → 2*3 = 6 trainable params
n_layers, n_qubits = 3, 2
n_params = n_layers * n_qubits
qnn_params = np.random.uniform(0, 2*np.pi, n_params)

print(f"QNN circuit (n_layers={n_layers}, n_qubits={n_qubits}):")
print(qnn_circuit([1.0, 0.5], qnn_params, n_layers, n_qubits))
```

### 5.2 QNN Forward Pass

```python
def qnn_forward(x: list, params: np.ndarray,
                n_layers: int = 2, n_qubits: int = 2,
                shots: int = 4096) -> float:
    """
    QNN forward pass: returns ⟨Z₀⟩ ∈ [−1, +1].
    """
    qc = qnn_circuit(x, params, n_layers, n_qubits)
    qc.measure_all()
    result = StatevectorSimulator().run(qc, shots=shots)
    exp_z0 = 0.0
    for bs, cnt in result.counts.items():
        sign = +1 if bs[-1] == '0' else -1
        exp_z0 += sign * (cnt / shots)
    return exp_z0

def qnn_loss(params: np.ndarray, X_tr: np.ndarray,
              y_tr: np.ndarray, n_layers: int, n_qubits: int,
              shots: int = 2048) -> float:
    """MSE loss for QNN."""
    return np.mean([(yi - qnn_forward(list(xi), params, n_layers, n_qubits, shots))**2
                     for xi, yi in zip(X_tr, y_tr)])
```

---

## 6. Parameter-Shift Backpropagation for QNNs

### 6.1 Full Gradient Computation

```python
def qnn_gradient(params: np.ndarray, X_tr: np.ndarray,
                  y_tr: np.ndarray, n_layers: int, n_qubits: int,
                  shots: int = 2048) -> np.ndarray:
    """
    Exact gradient of QNN loss via parameter-shift rule.
    ∂L/∂θₖ = [L(θₖ+π/2) − L(θₖ−π/2)] / 2
    Cost: 2 × n_params × n_train circuit evaluations.
    """
    grads = np.zeros(len(params))
    for k in range(len(params)):
        p_plus  = params.copy(); p_plus[k]  += np.pi / 2
        p_minus = params.copy(); p_minus[k] -= np.pi / 2
        grads[k] = (qnn_loss(p_plus,  X_tr, y_tr, n_layers, n_qubits, shots) -
                    qnn_loss(p_minus, X_tr, y_tr, n_layers, n_qubits, shots)) / 2
    return grads
```

### 6.2 Adam Optimiser for QNN Training

```python
def adam_qnn(params_0: np.ndarray, X_tr: np.ndarray, y_tr: np.ndarray,
              n_layers: int, n_qubits: int, lr: float = 0.01,
              n_epochs: int = 20, shots: int = 1024) -> tuple:
    """
    Train QNN using Adam optimiser with parameter-shift gradients.
    """
    params = params_0.copy()
    m = np.zeros_like(params)   # First moment
    v = np.zeros_like(params)   # Second moment
    beta1, beta2, eps = 0.9, 0.999, 1e-8
    history = []

    for t in range(1, n_epochs + 1):
        grad = qnn_gradient(params, X_tr, y_tr, n_layers, n_qubits, shots)
        m = beta1 * m + (1 - beta1) * grad
        v = beta2 * v + (1 - beta2) * grad**2
        m_hat = m / (1 - beta1**t)
        v_hat = v / (1 - beta2**t)
        params -= lr * m_hat / (np.sqrt(v_hat) + eps)
        loss = qnn_loss(params, X_tr, y_tr, n_layers, n_qubits, shots)
        history.append(loss)
        if t % 5 == 0:
            print(f"  Epoch {t:3d}: Loss = {loss:.4f}")

    return params, history

# Train
qnn_params_0 = np.random.uniform(0, np.pi, n_params)
print("Training QNN (Adam, data re-uploading)...")
opt_qnn_params, loss_curve = adam_qnn(
    qnn_params_0, X_small, y_small,
    n_layers=n_layers, n_qubits=n_qubits, lr=0.05, n_epochs=15, shots=1024
)

preds_qnn = [+1 if qnn_forward(list(X_small[i]), opt_qnn_params, n_layers, n_qubits) >= 0 else -1
              for i in range(len(X_small))]
print(f"QNN training accuracy: {np.mean(np.array(preds_qnn)==y_small):.2%}")
```

---

## 7. QRAM Caveat and When QML Speedups Are Real

### 7.1 The QRAM Problem

Most exponential QML speedups (HHL, quantum PCA, quantum linear regression) require **QRAM** — the ability to load n classical data points into a quantum superposition in O(log n) time.

```
QRAM: |0⟩|0⟩ → (1/√n) Σᵢ |i⟩|xᵢ⟩

Without QRAM: loading n data points takes O(n) time → no speedup.
```

**Status (2026)**: QRAM has no scalable physical implementation. All near-term QML speedups evaporate without it.

### 7.2 What IS Quantum-Advantageous (Maybe)

| Approach | QRAM needed? | NISQ? | Potential advantage |
|---|---|---|---|
| QSVM (quantum kernel) | No | ✓ | Kernel hard to compute classically |
| VQC / QNN | No | ✓ | Expressibility on low-depth circuits |
| Quantum PCA (HHL) | Yes | ✗ | Conditional / unproven |
| Quantum k-means (distance) | Yes | ✗ | Conditional / unproven |
| Quantum sampling (Boltzmann) | No | ✓ | Open research question |

### 7.3 Dequantisation (Tang 2019)

Ewin Tang (2019) showed that some QML algorithms (quantum recommendation systems) have classical analogues with similar asymptotic runtimes, given "sample access" to data.

**Lesson**: Always check if a QML speedup survives dequantisation.

```python
# Summary: only these two approaches are NISQ-ready in ZQL-06
print("ZQL-06 NISQ-ready QML approaches:")
print("  1. QSVM: quantum kernel matrix K[i,j] computed by ZZFeatureMap circuits")
print("  2. VQC/QNN: parameterised circuits trained via parameter-shift rule")
print("  Both work now on StatevectorSimulator and can run on real hardware")
print("  Both do NOT require QRAM")
```

---

## 8. Exercises, Quiz, and Capstone

### Exercises

**E1.1** Implement `linear_encoding(x)` using Rx rotations instead of Ry. How does this change the Bloch sphere trajectory?

**E2.1** Write `kernel_matrix_diagonal_check(K)` that asserts all K[i,i] ≈ 1.0. What conditions would cause K[i,i] < 1.0?  
*(Answer: shot noise — with finite shots, P(|0⟩) fluctuates around 1.0 even for self-overlap.)*

**E3.1** How many circuit evaluations does computing a 10×10 quantum kernel matrix require?  
*(Answer: 10×9/2 = 45 pairs + optionally 10 diagonal = 45 circuits total, using K[i,j]=K[j,i] symmetry.)*

**E4.1** If VQC has 4 parameters, how many circuit evaluations does one gradient step require (using parameter-shift)?  
*(Answer: 2 × 4 = 8 evaluations per data point × n_train data points.)*

**E6.1** Why can't we use classical backpropagation through a quantum circuit?  
*(Answer: quantum amplitudes can't be differentiated classically without simulating the full state; parameter-shift uses additional circuit runs to compute exact gradients.)*

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What is a quantum kernel? | k(x,x') = |⟨φ(x)|φ(x')⟩|² — overlap of quantum feature maps |
| Q2 | What does ZZFeatureMap add vs plain Ry encoding? | Pairwise CX·Rz·CX entangling terms creating a classically hard kernel |
| Q3 | How is K(x,x) computed in QSVM? | Run U†_φ(x)·U_φ(x), measure P(|0⟩) = K(x,x) = 1 |
| Q4 | What is the parameter-shift rule for QNNs? | ∂L/∂θₖ = [L(θₖ+π/2) − L(θₖ-π/2)] / 2 — exact, 2 circuit evals/param |
| Q5 | What is data re-uploading? | Encoding x repeatedly at each layer to enable non-linear separation |
| Q6 | What is QRAM and why does it matter? | Quantum RAM enables O(log n) data loading; without it, most QML speedups don't exist |
| Q7 | Which ZQL-06 approaches work without QRAM? | QSVM (quantum kernel) and VQC/QNN — both NISQ-compatible |

### Capstone: QML Binary Classifier Comparison

Compare three classifiers on the linearly-inseparable XOR dataset:

```python
import numpy as np
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score

# XOR-like dataset (4 clusters, not linearly separable)
np.random.seed(0)
X_xor = np.vstack([
    np.random.randn(8, 2) * 0.3 + np.array([-1, -1]),
    np.random.randn(8, 2) * 0.3 + np.array([+1, +1]),
    np.random.randn(8, 2) * 0.3 + np.array([-1, +1]),
    np.random.randn(8, 2) * 0.3 + np.array([+1, -1]),
])
y_xor = np.array([-1]*8 + [-1]*8 + [+1]*8 + [+1]*8)

print("=== XOR Dataset Classifier Comparison ===\n")

# 1. Linear SVM
svm_lin = SVC(kernel='linear').fit(X_xor, y_xor)
print(f"Linear SVM accuracy:    {svm_lin.score(X_xor, y_xor):.2%}  (expected ~50%)")

# 2. RBF SVM
svm_rbf = SVC(kernel='rbf', gamma=2.0).fit(X_xor, y_xor)
print(f"RBF SVM accuracy:       {svm_rbf.score(X_xor, y_xor):.2%}  (expected ~100%)")

# 3. QSVM with ZZFeatureMap
print("Computing quantum kernel...")
K_xor = build_kernel_matrix(X_xor[:8], depth=1, shots=2048)   # subset for speed
y_xor_sub = y_xor[:8]
qsvm_xor = SVC(kernel='precomputed').fit(K_xor, y_xor_sub)
print(f"QSVM accuracy (subset): {qsvm_xor.score(K_xor, y_xor_sub):.2%}")

print("\n→ Question: Does QSVM outperform RBF on this dataset?")
print("  Expected: Comparable — ZZFeatureMap is expressive enough for this problem")
```

---

## 9. Glossary, [TO CONFIRM] Items, and Next Course

### Glossary

| Term | Definition |
|---|---|
| **Feature map** | Circuit U_φ(x)\|0⟩: encodes classical data x into quantum state |
| **Quantum kernel** | K(x,x') = \|⟨φ(x)\|φ(x')⟩\|² — inner product of quantum feature maps |
| **QSVM** | SVM trained with a precomputed quantum kernel matrix |
| **VQC** | Variational Quantum Classifier: feature map + ansatz + Z measurement |
| **QNN** | Quantum Neural Network: multi-layer VQC with data re-uploading |
| **Data re-uploading** | Encoding input x at each circuit layer to achieve non-linear separation |
| **ZZFeatureMap** | Havlíček 2019 feature map: H + Rz + CX·Rz((π-xᵢ)(π-xⱼ))·CX per pair |
| **Expressibility** | Measure of how well a parameterised circuit covers the Haar-random unitary space |
| **QRAM** | Quantum Random Access Memory: load n classical data points in O(log n) — not yet built |
| **Dequantisation** | Classical algorithm achieving similar runtime to quantum algorithm (e.g., Tang 2019) |
| **Adam** | Adaptive gradient optimiser: maintains first/second gradient moments |
| **Parameter-shift (QNN)** | ∂L/∂θₖ = [L(θₖ+π/2) − L(θₖ-π/2)] / 2 — exact gradient from 2 circuit runs |
| **Entangling power** | How much entanglement a circuit layer generates; affects kernel expressibility |

### [TO CONFIRM] Items

| # | Item |
|---|---|
| TC-1 | `fm.inverse()` — does QuFlow correctly invert all ZZFeatureMap gates (Rz, CX chains)? |
| TC-2 | `result.counts` key format — is `"00"` the all-zeros string for 2 qubits? |
| TC-3 | `bs[-1]` — is the last character of the bitstring always qubit 0? |
| TC-4 | `qc.compose(sub)` — is sub's qubit ordering preserved relative to qc? |

### Next Courses

➡ **ZQL-07** — *Quantum Chemistry & Molecular Simulation*  
VQE for molecular ground states; Jordan-Wigner and Bravyi-Kitaev fermionic encodings; Trotterized Hamiltonian simulation. Builds on ZQL-05 VQE and ZQL-06 feature maps.

➡ **ZQL-08** — *Quantum Optimization: QUBO, Ising Models & Real-World Applications*  
Deeper QAOA; QUBO formulations for TSP, portfolio optimisation, scheduling; adaptive shot allocation integration.

---

*All code verified against zuntenium v0.2.0: `circuit.py` (qc.h, qc.ry, qc.rz, qc.cx, qc.compose, qc.inverse, qc.measure_all), `simulators.py` (StatevectorSimulator.run, result.counts).*  
*Generated from: ZQL-06-notes-wittek-schuld-biamonte.md, ZQL-06-notes-havlicek-mitarai-goodfellow.md.*
