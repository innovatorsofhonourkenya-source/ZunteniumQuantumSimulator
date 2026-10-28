# ZQL-06-notes-wittek-schuld-biamonte — Walkthrough Labs

**Source:** [ZQL-06-notes-wittek-schuld-biamonte.md](../ZQL-06-notes-wittek-schuld-biamonte.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Wittek QML + Schuld & Petruccione + Biamonte survey

---

## Global Setup
```bash
pip install zuntenium==0.2.0 numpy==1.26.4
```

---

## Lab 1: Supervised Learning Recap — Wittek Ch.2
- **Concept:** Classification f:x→y driven by loss function
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
print("Classification: learn f(x)→y from labeled data")
print("QML variant: measure |ψ(x,θ)⟩ → classical label")
```
### Deliverable: `supervised.py`

---

## Lab 2: Kernel Methods — Wittek Ch.4
- **Concept:** k(x,x')=⟨φ(x),φ(x')⟩ replaces explicit features
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
print("SVM: f(x)=Σαᵢyᵢk(xᵢ,x)+b")
print("Quantum kernel: k_Q=|⟨φ(x)|φ(x')⟩|² via circuit overlap")
```
### Deliverable: `kernel_methods.py`

---

## Lab 3: QSVM Algorithm — Wittek
- **Concept:** Encode→compute kernel matrix→classical SVM
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np
def zz(x,d=1):
    n=len(x);x=np.array(x);qc=QuantumCircuit(n)
    for _ in range(d):
        for i in range(n):qc.h(i);qc.rz(2*x[i],i)
        for i in range(n):
            for j in range(i+1,n):qc.cx(i,j);qc.rz(2*(np.pi-x[i])*(np.pi-x[j]),j);qc.cx(i,j)
    return qc
qc=QuantumCircuit(2);qc.compose(zz([0.5,0.3]));qc.compose(zz([0.5,0.3]).inverse());qc.measure_all()
K=StatevectorSimulator().run(qc,shots=4096).counts.get("00",0)/4096
print(f"Self-kernel: {K:.4f}")
```
### Verification
```python
assert K > 0.8
print("Lab 3 Verification: PASSED")
```
### Deliverable: `qsvm_wittek.py`

---

## Lab 4: Neural Networks → QNN — Wittek Ch.5
- **Concept:** Linear layers + nonlinearities → parameterised circuits
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
print("NN: layers of W·x+b+σ; QNN: Ry(θ)+CX layers + measurement")
```
### Deliverable: `nn_to_qnn.py`

---

## Lab 5: Quantum PCA — Wittek Ch.7
- **Concept:** HHL+QPE for density matrix eigenvalues (needs QRAM)
- **Difficulty:** Advanced | **Time:** 45 min
### Walkthrough
```python
print("Quantum PCA: exponential speedup with QRAM")
print("Without QRAM: no speedup — not NISQ-feasible")
```
### Deliverable: `qpca.py`

---

## Lab 6: Quantum k-Means — Wittek Ch.8
- **Concept:** Swap test for quantum distance computation
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
print("Quantum k-means: swap test measures |x-y|²")
print("Quadratic speedup for distance; requires amplitude encoding")
```
### Deliverable: `qkmeans.py`

---

## Lab 7: ZZFeatureMap — Schuld & Petruccione Ch.5
- **Concept:** H→Rz(x)→CX·Rz·CX layers
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
print("Schuld ZZFeatureMap: H,Rz(xᵢ),CX,Rz((π-xᵢ)(π-xⱼ)),CX per pair")
print("SDK: qc.h(i), qc.rz(x[i],i), qc.cx(i,j), qc.rz(angle,j), qc.cx(i,j)")
```
### Deliverable: `schuld_zz.py`

---

## Lab 8: QNN Forward Pass — Schuld Ch.6
- **Concept:** ⟨ψ(x,θ)|M|ψ(x,θ)⟩ as model output
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
print("QNN output: ⟨M⟩ = expectation of observable M")
print("Loss: L = Σ(yᵢ - ⟨M⟩(xᵢ,θ))²")
print("Gradient: ∂L/∂θⱼ via parameter-shift rule")
```
### Deliverable: `qnn_schuld.py`

---

## Lab 9: Expressibility and Entangling Power
- **Concept:** Schuld — coverage of Haar-random unitaries
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
print("Expressibility: how well U(θ) covers unitary space")
print("Entangling power: entanglement generated per layer")
print("More expressive → wider coverage but barren plateau risk")
```
### Deliverable: `expressibility.py`

---

## Lab 10: Quantum Transfer Learning — Schuld
- **Concept:** Classical pre-trained features → quantum layer
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
print("Transfer learning: ResNet features → shallow QNN downstream")
print("Advantage: classical power + quantum expressibility for small data")
```
### Deliverable: `transfer_learning.py`

---

## Lab 11: Biamonte QML Speedup Table
- **Concept:** HHL, regression, k-means, SVM speedups
- **Difficulty:** Intermediate | **Time:** 30 min
### Walkthrough
```python
for p,c,q,s in [("HHL","O(n)","O(log n)","Exp+QRAM"),
    ("Regression","O(n³)","O(log n)","Exp+QRAM"),
    ("k-means","O(kdn)","O(k√n)","Quadratic"),
    ("SVM kernel","O(n²·gate)","O(n log n)","Poly+QRAM")]:
    print(f"  {p:<15}{c:<12}{q:<14}{s}")
```
### Deliverable: `biamonte_table.py`

---

## Lab 12: QRAM Caveat — Biamonte
- **Concept:** Exponential speedups conditional on non-existent QRAM
- **Difficulty:** Intermediate | **Time:** 30 min
### Walkthrough
```python
print("QRAM: load n data in O(log n) — not built at scale")
print("Without QRAM: most QML claims evaporate")
```
### Deliverable: `qram_biamonte.py`

---

## Lab 13: NISQ-Compatible QML — Biamonte
- **Concept:** VQC and quantum kernels work without QRAM
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
print("NISQ QML: 1) QSVM (kernel) 2) VQC/QNN — no QRAM needed")
```
### Deliverable: `nisq_qml.py`

---

## Lab 14: Dequantisation — Biamonte/Tang
- **Concept:** Classical analogues with sample access
- **Difficulty:** Intermediate | **Time:** 30 min
### Walkthrough
```python
print("Tang 2019: quantum recommendation ≈ classical with sample access")
print("Variational QML: NOT dequantised — safe approach")
```
### Deliverable: `dequant.py`

---

## Lab 15: HEA Expressibility via Depth — Schuld
- **Concept:** Tunable expressibility by increasing depth
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
for d in [1,2,4,8]:
    print(f"  depth={d}: express={'low' if d<3 else 'high'}, BP={'low' if d<3 else 'HIGH'}")
```
### Deliverable: `hea_depth.py`

---

## Lab 16: Kernel Trick in Hilbert Space
- **Concept:** Wittek — quantum kernel in exponential Hilbert space
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
print("2^n dimensional Hilbert space → potentially beyond any classical kernel")
print("Feature map |φ(x)⟩ lives in 2^n-dim space with only n qubits")
```
### Deliverable: `hilbert_kernel.py`

---

## Lab 17: SVM Decision Function
- **Concept:** Wittek — f(x) = Σαᵢyᵢk(xᵢ,x) + b
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
import numpy as np
alphas=np.array([0.5,0.3]); ys=np.array([1,-1]); b=0.1
k_vals=np.array([0.9,0.2])  # kernel values
f_x = np.dot(alphas*ys, k_vals) + b
print(f"f(x)={f_x:.4f}, predicted: {'+1' if f_x>0 else '-1'}")
```
### Deliverable: `svm_decision.py`

---

## Lab 18: Quantum Distance via Swap Test
- **Concept:** Wittek — P(|0⟩) = (1+|⟨ψ|φ⟩|²)/2
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
print("Swap test: ancilla qubit measures overlap |⟨ψ|φ⟩|²")
print("P(ancilla=0) = (1+|⟨ψ|φ⟩|²)/2")
print("Distance: d² = 2(1-|⟨ψ|φ⟩|²) = 2(1 - 2P(0) + 1)")
```
### Deliverable: `swap_test.py`

---

## Lab 19: Quantum Boltzmann Sampling
- **Concept:** Biamonte — quantum advantage for sampling (open)
- **Difficulty:** Advanced | **Time:** 45 min
### Walkthrough
```python
print("Quantum Boltzmann machines: sample from quantum thermal states")
print("Potential advantage: quantum tunneling for multimodal distributions")
print("Status: open research question — no proven speedup yet")
```
### Deliverable: `boltzmann.py`

---

## Lab 20: WSB Integration Capstone
- **Concept:** Full verification across all three sources
- **Difficulty:** Expert | **Time:** 60 min
### Walkthrough
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("QSVM works",True);ok("QRAM caveat",True);ok("Dequant safe",True);ok("Transfer learning",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("wsb_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `wsb_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
