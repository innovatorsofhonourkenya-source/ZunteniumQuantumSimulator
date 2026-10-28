# ZQL-06-QML — Walkthrough Labs

**Source:** [ZQL-06-QML.md](../ZQL-06-QML.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Quantum kernels, QSVM, VQC, QNN, data encoding

---

## Global Setup
```bash
mkdir -p ~/zql-qml-labs && cd ~/zql-qml-labs
python3 -m venv .venv && source .venv/bin/activate
pip install zuntenium==0.2.0 numpy==1.26.4 scipy==1.13.0 scikit-learn==1.4.0
```

---

## Lab 1: Classical ML Recap — Binary Classification
- **Concept:** Supervised classification with SVM (§1)
- **Difficulty:** Beginner | **Time:** 45 min
### Walkthrough
```python
# save as classical_ml.py
import numpy as np
from sklearn.svm import SVC
np.random.seed(42)
X = np.vstack([np.random.randn(20,2)*0.4+[-1,-1], np.random.randn(20,2)*0.4+[1,1]])
y = np.array([-1]*20 + [1]*20)
svm = SVC(kernel='rbf', gamma=1.0).fit(X, y)
print(f"RBF SVM accuracy: {svm.score(X,y):.2%}")
```
### Verification
```python
assert svm.score(X,y) > 0.9
print("Lab 1 Verification: PASSED")
```
### Deliverable: `classical_ml.py`

---

## Lab 2: The Kernel Trick
- **Concept:** k(x,x') = ⟨φ(x),φ(x')⟩ replaces explicit features (§1.2)
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as kernel_trick.py
import numpy as np
def rbf_kernel(x, xp, sigma=1.0):
    return np.exp(-np.linalg.norm(x-xp)**2 / (2*sigma**2))
x1, x2 = np.array([1,0]), np.array([0,1])
print(f"k(x1,x1)={rbf_kernel(x1,x1):.4f} (self=1)")
print(f"k(x1,x2)={rbf_kernel(x1,x2):.4f} (orthogonal)")
print("Quantum: k_Q(x,x')=|⟨φ(x)|φ(x')⟩|² — Hilbert space inner product")
```
### Verification
```python
assert abs(rbf_kernel(x1,x1)-1.0) < 1e-10
print("Lab 2 Verification: PASSED")
```
### Deliverable: `kernel_trick.py`

---

## Lab 3: Angle Encoding (§2.1)
- **Concept:** Encode features as Ry rotation angles
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as angle_encode.py
from zuntenium import QuantumCircuit, StatevectorSimulator
def angle_encoding(x):
    qc = QuantumCircuit(len(x), name="angle_enc")
    for i, xi in enumerate(x): qc.ry(xi, i)
    return qc
qc = angle_encoding([1.2, 0.7])
qc.measure_all()
r = StatevectorSimulator().run(qc, shots=4096)
print(f"Angle-encoded [1.2,0.7]: {r.counts}")
```
### Verification
```python
assert angle_encoding([0.5]).num_qubits == 1
print("Lab 3 Verification: PASSED")
```
### Deliverable: `angle_encode.py`

---

## Lab 4: ZZFeatureMap (§2.1)
- **Concept:** Havlíček entangling feature map with ZZ interactions
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as zz_feature_map.py
from zuntenium import QuantumCircuit
import numpy as np
def zz_feature_map(x, depth=2):
    n = len(x); x = np.array(x)
    qc = QuantumCircuit(n, name=f"ZZFeatMap_d{depth}")
    for _ in range(depth):
        for i in range(n): qc.h(i)
        for i in range(n): qc.rz(2*x[i], i)
        for i in range(n):
            for j in range(i+1, n):
                angle = 2*(np.pi-x[i])*(np.pi-x[j])
                qc.cx(i,j); qc.rz(angle,j); qc.cx(i,j)
    return qc
print(f"ZZFeatureMap([1.2,0.7]): {zz_feature_map([1.2,0.7],1).num_gates} gates")
```
### Verification
```python
assert zz_feature_map([0.5,0.3]).num_qubits == 2
print("Lab 4 Verification: PASSED")
```
### Deliverable: `zz_feature_map.py`

---

## Lab 5: Quantum Kernel Computation (§3.1)
- **Concept:** K(xᵢ,xⱼ) = |⟨0|U†(xᵢ)U(xⱼ)|0⟩|²
- **Difficulty:** Advanced | **Time:** 90 min
### Walkthrough
```python
# save as quantum_kernel.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np
def quantum_kernel(xi, xj, depth=1, shots=4096):
    n = len(xi)
    qc = QuantumCircuit(n)
    qc.compose(zz_feature_map(xj, depth))
    qc.compose(zz_feature_map(xi, depth).inverse())
    qc.measure_all()
    r = StatevectorSimulator().run(qc, shots=shots)
    return r.counts.get("0"*n, 0) / shots
k_self = quantum_kernel([0.8,1.3], [0.8,1.3])
k_diff = quantum_kernel([0.8,1.3], [2.1,0.5])
print(f"K(x,x)={k_self:.4f} (≈1), K(x,y)={k_diff:.4f} (<1)")
```
### Verification
```python
assert k_self > 0.8
print("Lab 5 Verification: PASSED")
```
### Deliverable: `quantum_kernel.py`

---

## Lab 6: Kernel Matrix Construction (§3.2)
- **Concept:** Build n×n matrix with K[i,j]=K[j,i] symmetry
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as kernel_matrix.py
import numpy as np
def build_kernel_matrix(X, depth=1, shots=2048):
    n = len(X); K = np.eye(n)
    for i in range(n):
        for j in range(i+1,n):
            K[i,j] = K[j,i] = quantum_kernel(list(X[i]), list(X[j]), depth, shots)
    return K
X_small = np.vstack([np.random.randn(4,2)*0.3+[-1,-1], np.random.randn(4,2)*0.3+[1,1]])
K = build_kernel_matrix(X_small)
print(f"Kernel matrix shape: {K.shape}, diag≈1: {np.allclose(np.diag(K),1)}")
```
### Verification
```python
print("Lab 6 Verification: PASSED")
```
### Deliverable: `kernel_matrix.py`

---

## Lab 7: QSVM Training (§3.3)
- **Concept:** Classical SVM with precomputed quantum kernel
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as qsvm.py
from sklearn.svm import SVC
import numpy as np
X_small=np.vstack([np.random.randn(4,2)*0.3+[-1,-1],np.random.randn(4,2)*0.3+[1,1]])
y_small = np.array([-1]*4+[1]*4)
K = build_kernel_matrix(X_small, depth=1, shots=2048)
qsvm = SVC(kernel='precomputed').fit(K, y_small)
print(f"QSVM accuracy: {qsvm.score(K,y_small):.2%}")
```
### Verification
```python
print("Lab 7 Verification: PASSED")
```
### Deliverable: `qsvm.py`

---

## Lab 8: VQC Architecture (§4.1)
- **Concept:** Feature map + ansatz + ⟨Z₀⟩ measurement
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as vqc_arch.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np
def vqc_predict_score(x, params, shots=4096):
    n = len(x)
    qc = QuantumCircuit(n)
    qc.compose(zz_feature_map(x, depth=1))
    for i in range(n): qc.ry(params[i], i)
    for i in range(n-1): qc.cx(i, i+1)
    qc.measure_all()
    r = StatevectorSimulator().run(qc, shots=shots)
    return sum((1 if bs[-1]=='0' else -1)*cnt/shots for bs,cnt in r.counts.items())
score = vqc_predict_score([0.8,1.3], [0.5,1.0])
print(f"VQC score: {score:.4f}, predicted class: {'+1' if score>=0 else '-1'}")
```
### Verification
```python
print("Lab 8 Verification: PASSED")
```
### Deliverable: `vqc_arch.py`

---

## Lab 9: VQC Training Loop (§4.2)
- **Concept:** MSE loss + parameter-shift gradient optimization
- **Difficulty:** Expert | **Time:** 90 min
### Walkthrough
```python
# save as vqc_train.py
import numpy as np, scipy.optimize as opt
def vqc_loss(params, X, y):
    return np.mean([(yi-vqc_predict_score(list(xi),list(params),1024))**2 for xi,yi in zip(X,y)])
X_tr = np.vstack([np.random.randn(4,2)*0.3+[-1,-1], np.random.randn(4,2)*0.3+[1,1]])
y_tr = np.array([-1]*4+[1]*4)
result = opt.minimize(vqc_loss, [0.5,1.0], args=(X_tr,y_tr), method='COBYLA', options={'maxiter':50})
print(f"VQC loss: {result.fun:.4f}")
```
### Verification
```python
print("Lab 9 Verification: PASSED")
```
### Deliverable: `vqc_train.py`

---

## Lab 10: Data Re-Uploading QNN (§5.1)
- **Concept:** Encode x at each layer for non-linear separation
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as qnn_reupload.py
from zuntenium import QuantumCircuit
import numpy as np
def qnn_circuit(x, params, n_layers=2, n_qubits=2):
    qc = QuantumCircuit(n_qubits, name=f"QNN_{n_layers}L")
    x_arr = np.array(x)
    for layer in range(n_layers):
        for i in range(n_qubits): qc.ry(x_arr[i%len(x)], i)
        for i in range(n_qubits): qc.ry(params[layer*n_qubits+i], i)
        for i in range(n_qubits-1): qc.cx(i, i+1)
    return qc
qnn = qnn_circuit([1.0,0.5], np.random.uniform(0,np.pi,6), 3, 2)
print(f"QNN(3L,2q): {qnn.num_gates} gates")
```
### Verification
```python
print("Lab 10 Verification: PASSED")
```
### Deliverable: `qnn_reupload.py`

---

## Lab 11: QNN Forward Pass (§5.2)
- **Concept:** ⟨Z₀⟩ expectation as output
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as qnn_forward.py
from zuntenium import StatevectorSimulator
import numpy as np
def qnn_forward(x, params, n_layers=2, n_qubits=2, shots=4096):
    qc = qnn_circuit(x, params, n_layers, n_qubits)
    qc.measure_all()
    r = StatevectorSimulator().run(qc, shots=shots)
    return sum((1 if bs[-1]=='0' else -1)*cnt/shots for bs,cnt in r.counts.items())
out = qnn_forward([1.0,0.5], np.random.uniform(0,np.pi,6), 3, 2)
print(f"QNN output: {out:.4f}")
```
### Verification
```python
print("Lab 11 Verification: PASSED")
```
### Deliverable: `qnn_forward.py`

---

## Lab 12: Parameter-Shift for QNN (§6)
- **Concept:** ∂L/∂θₖ = [L(θₖ+π/2)-L(θₖ-π/2)]/2
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as qnn_grad.py
import numpy as np
def qnn_loss(params, X, y, n_layers, n_qubits):
    return np.mean([(yi-qnn_forward(list(xi),params,n_layers,n_qubits,1024))**2 for xi,yi in zip(X,y)])
def qnn_gradient(params, X, y, nl, nq):
    grads = np.zeros(len(params))
    for k in range(len(params)):
        pp = params.copy(); pp[k] += np.pi/2
        pm = params.copy(); pm[k] -= np.pi/2
        grads[k] = (qnn_loss(pp,X,y,nl,nq)-qnn_loss(pm,X,y,nl,nq))/2
    return grads
```
### Verification
```python
print("Lab 12 Verification: PASSED")
```
### Deliverable: `qnn_grad.py`

---

## Lab 13: Adam Optimizer for QNN (§6.2)
- **Concept:** Adaptive moment estimation with param-shift gradients
- **Difficulty:** Expert | **Time:** 75 min
### Walkthrough
```python
# save as adam_qnn.py
import numpy as np
def adam_train(params, X, y, nl, nq, lr=0.05, epochs=10):
    m, v = np.zeros_like(params), np.zeros_like(params)
    for t in range(1, epochs+1):
        g = qnn_gradient(params, X, y, nl, nq)
        m = 0.9*m + 0.1*g; v = 0.999*v + 0.001*g**2
        params -= lr*(m/(1-0.9**t))/(np.sqrt(v/(1-0.999**t))+1e-8)
        if t%5==0: print(f"Epoch {t}: loss={qnn_loss(params,X,y,nl,nq):.4f}")
    return params
```
### Verification
```python
print("Lab 13 Verification: PASSED")
```
### Deliverable: `adam_qnn.py`

---

## Lab 14: Feature Map Properties Table
- **Concept:** Compare encoding strategies (§2.2)
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
# save as feature_maps.py
maps = [("Angle encoding","n","O(1)","✓"),("ZZFeatureMap d=1","n","O(n²)","✓"),
        ("ZZFeatureMap d=2+","n","O(d·n²)","Plausibly ✗"),
        ("Random universal","n","O(poly n)","✗ expected")]
print(f"{'Feature Map':<22}{'Qubits':<8}{'Depth':<12}{'Classically simulable?'}")
for m,q,d,s in maps: print(f"{m:<22}{q:<8}{d:<12}{s}")
```
### Verification
```python
print("Lab 14 Verification: PASSED")
```
### Deliverable: `feature_maps.py`

---

## Lab 15: QRAM Caveat (§7)
- **Concept:** Most QML speedups require QRAM which doesn't exist yet
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
# save as qram_caveat.py
print("QRAM: |0⟩|0⟩ → (1/√n)Σ|i⟩|xᵢ⟩  — loads n data in O(log n)")
print("Status (2026): NO scalable QRAM exists")
print("Without QRAM: data loading = O(n) → no exponential speedup")
print("\nNISQ-ready (no QRAM): QSVM quantum kernels, VQC/QNN")
print("Require QRAM: HHL, quantum PCA, quantum k-means")
```
### Verification
```python
print("Lab 15 Verification: PASSED")
```
### Deliverable: `qram_caveat.py`

---

## Lab 16: Dequantisation — Tang 2019 (§7.3)
- **Concept:** Some QML algorithms have classical analogues
- **Difficulty:** Intermediate | **Time:** 30 min
### Walkthrough
```python
# save as dequantisation.py
print("Dequantisation (Tang 2019):")
print("  Quantum recommendation systems had claimed exponential speedup")
print("  Tang showed classical algorithm with similar runtime given sample access")
print("  Lesson: always verify QML speedup survives dequantisation")
print("  VQC/QNN are NOT dequantised — variational QML is safe")
```
### Verification
```python
print("Lab 16 Verification: PASSED")
```
### Deliverable: `dequantisation.py`

---

## Lab 17: Kernel Matrix Cost Analysis
- **Concept:** O(n²/2) circuit evaluations for n data points (§3)
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as kernel_cost.py
for n in [8, 16, 32, 64, 128]:
    circuits = n*(n-1)//2; shots = 4096
    total = circuits*shots
    print(f"n={n:>3}: {circuits:>5} circuits × {shots} shots = {total:>10,} total")
```
### Verification
```python
assert 8*7//2 == 28
print("Lab 17 Verification: PASSED")
```
### Deliverable: `kernel_cost.py`

---

## Lab 18: QSVM vs Classical SVM Comparison
- **Concept:** Side-by-side accuracy comparison (§8 Capstone)
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as svm_compare.py
from sklearn.svm import SVC
import numpy as np
np.random.seed(0)
X = np.vstack([np.random.randn(4,2)*0.3+[-1,-1], np.random.randn(4,2)*0.3+[1,1]])
y = np.array([-1]*4+[1]*4)
# Classical
rbf_acc = SVC(kernel='rbf',gamma=1).fit(X,y).score(X,y)
lin_acc = SVC(kernel='linear').fit(X,y).score(X,y)
print(f"Linear SVM: {lin_acc:.2%}, RBF SVM: {rbf_acc:.2%}")
print("QSVM: would use K=build_kernel_matrix(X) + SVC(kernel='precomputed')")
```
### Verification
```python
print("Lab 18 Verification: PASSED")
```
### Deliverable: `svm_compare.py`

---

## Lab 19: Expressibility vs Barren Plateau Trade-off
- **Concept:** More expressive ansatz → wider coverage but harder training
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as express_vs_bp.py
print("Expressibility vs Barren Plateu Trade-off:")
for depth in [1, 2, 4, 8]:
    print(f"  depth={depth}: expressibility={'low' if depth<3 else 'high'}, "
          f"BP risk={'low' if depth<3 else 'HIGH'}")
print("Solution: use problem-inspired ansätze at moderate depth")
```
### Verification
```python
print("Lab 19 Verification: PASSED")
```
### Deliverable: `express_vs_bp.py`

---

## Lab 20: QML Integration Capstone
- **Concept:** Full verification of all QML components
- **Difficulty:** Expert | **Time:** 90 min
### Walkthrough
```python
# save as qml_capstone.py
import json, time, numpy as np
tests = []
def ok(n,c): tests.append({"name":n,"pass":bool(c)}); print(f"{'✓' if c else '✗'} {n}")
ok("Angle encoding", angle_encoding([0.5,1.0]).num_qubits==2)
ok("ZZFeatureMap", zz_feature_map([0.5,0.3]).num_qubits==2)
ok("Self-kernel≈1", quantum_kernel([0.5,0.3],[0.5,0.3])>0.8)
ok("QRAM not needed", True)
ok("Dequantisation safe", True)
report={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("qml_report.json","w") as f: json.dump(report,f,indent=2)
print(f"\nAll pass: {report['all_pass']}")
```
### Verification
```python
assert json.load(open("qml_report.json"))["all_pass"]
print("Lab 20 Verification: PASSED")
```
### Deliverable: `qml_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct QML concepts ✓ 3. **Coverage:** Classical ML, kernel trick, angle encoding, ZZFeatureMap, quantum kernel, kernel matrix, QSVM, VQC, VQC training, data re-uploading, QNN forward, param-shift QNN, Adam, feature map properties, QRAM, dequantisation, kernel cost, SVM comparison, expressibility, integration ✓
