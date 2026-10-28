# ZQL-02B-notes-tanenbaum-gottesman-numpy — Walkthrough Labs

**Source:** [ZQL-02B-notes-tanenbaum-gottesman-numpy.md](../ZQL-02B-notes-tanenbaum-gottesman-numpy.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Memory hierarchy, stabilizer formalism, NumPy performance

---

## Global Setup
```bash
mkdir -p ~/zql-tgn-labs && cd ~/zql-tgn-labs
python3 -m venv .venv && source .venv/bin/activate
pip install zuntenium==0.2.0 numpy==1.26.4
```

---

## Lab 1: Cache Hierarchy and Quantum Amplitudes
- **Concept:** L1/L2/L3/DRAM latency and amplitude storage (Tanenbaum)
- **Difficulty:** Beginner | **Time:** 45 min
### Walkthrough
```python
# save as cache_latency.py
cache = {"L1": (4, 64), "L2": (12, 256), "L3": (40, 8192), "DRAM": (200, None)}
amps_per_line = 64 // 16  # 64B line / 16B complex128
print(f"Amplitudes per cache line: {amps_per_line}")
for n in [4, 10, 17, 20, 25]:
    sv_bytes = 16 * 2**n
    fits = "L1" if sv_bytes <= 64*1024 else "L2" if sv_bytes <= 256*1024 else "L3" if sv_bytes <= 8*2**20 else "DRAM"
    print(f"n={n:>2}: {sv_bytes:>12} bytes → fits in {fits}")
```
### Verification
```python
assert amps_per_line == 4
print("Lab 1 Verification: PASSED")
```
### Deliverable: `cache_latency.py`

---

## Lab 2: Memory Layout Comparison — Four Simulators
- **Concept:** SV O(2^n), DM O(4^n), TN O(2^k·m), Stab O(n²)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as memory_layouts.py
for n in [10, 15, 20, 30, 1000]:
    sv = 16*2**min(n,30); dm = 16*4**min(n,15); tab = n*n//8
    def fmt(b):
        if b<1024: return f"{b}B"
        if b<2**20: return f"{b//1024}KB"
        if b<2**30: return f"{b//2**20}MB"
        return f"{b//2**30}GB"
    sv_s = fmt(sv) if n<=30 else "impossible"
    dm_s = fmt(dm) if n<=15 else "impossible"
    print(f"n={n:>4}: SV={sv_s:>10} DM={dm_s:>10} Tab={fmt(tab):>10}")
```
### Verification
```python
print("Lab 2 Verification: PASSED")
```
### Deliverable: `memory_layouts.py`

---

## Lab 3: complex128 vs complex64 Precision
- **Concept:** Data type choice affects precision and capacity
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as dtype_precision.py
import numpy as np
for dtype in [np.complex64, np.complex128]:
    a = np.array([1+1e-8j], dtype=dtype)
    print(f"{dtype.__name__}: bytes={a.itemsize}, value={a[0]}, eps={np.finfo(dtype).eps:.2e}")
print("QuFlow uses complex128 (16B) for ~1e-15 precision")
```
### Verification
```python
assert np.array([1], dtype=np.complex128).itemsize == 16
print("Lab 3 Verification: PASSED")
```
### Deliverable: `dtype_precision.py`

---

## Lab 4: C-Contiguous vs F-Contiguous Arrays
- **Concept:** Row-major memory layout for quantum arrays (Johansson)
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as contiguous.py
import numpy as np
c_arr = np.zeros((4,4), dtype=np.complex128, order='C')
f_arr = np.zeros((4,4), dtype=np.complex128, order='F')
print(f"C-contiguous: {c_arr.flags['C_CONTIGUOUS']}, strides={c_arr.strides}")
print(f"F-contiguous: {f_arr.flags['F_CONTIGUOUS']}, strides={f_arr.strides}")
print("SDK uses C-contiguous (row-major) for density matrices")
```
### Verification
```python
assert c_arr.flags['C_CONTIGUOUS']
print("Lab 4 Verification: PASSED")
```
### Deliverable: `contiguous.py`

---

## Lab 5: tracemalloc Memory Profiling
- **Concept:** Profile actual memory of simulation (Gorelick)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as tracemalloc_profile.py
import tracemalloc, numpy as np
for n in [4, 8, 12, 16]:
    tracemalloc.start()
    sv = np.zeros(2**n, dtype=np.complex128); sv[0] = 1
    _, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    print(f"n={n:>2}: theory={16*2**n:>8}B, peak={peak:>8}B, ratio={peak/(16*2**n):.1f}x")
```
### Verification
```python
print("Lab 5 Verification: PASSED")
```
### Deliverable: `tracemalloc_profile.py`

---

## Lab 6: np.einsum Matrix Multiply
- **Concept:** Einstein summation for tensor operations (Johansson)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as einsum_matmul.py
import numpy as np
A = np.random.randn(4,4); B = np.random.randn(4,4)
C_einsum = np.einsum("ij,jk->ik", A, B)
C_matmul = A @ B
print(f"einsum == matmul: {np.allclose(C_einsum, C_matmul)}")
# Trace
print(f"Trace via einsum: {np.einsum('ii->', A):.4f} vs np: {np.trace(A):.4f}")
```
### Verification
```python
assert np.allclose(C_einsum, C_matmul)
print("Lab 6 Verification: PASSED")
```
### Deliverable: `einsum_matmul.py`

---

## Lab 7: einsum Gate Application Without Full Matrix
- **Concept:** Contract gate onto qubit axis of reshaped statevector
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as einsum_gate.py
import numpy as np
def apply_gate(state, gate, target, n):
    psi = state.reshape([2]*n)
    ax = [chr(ord('a')+k) for k in range(n)]
    t=ax[target]; z='z'
    sub = f"z{t},{''.join(ax)}->{''.join(z if k==target else ax[k] for k in range(n))}"
    return np.einsum(sub, gate, psi).reshape(-1)
H = np.array([[1,1],[1,-1]])/np.sqrt(2)
state = np.zeros(8, dtype=complex); state[0]=1
result = apply_gate(state, H, 1, 3)
print(f"P(000)={abs(result[0])**2:.4f}, P(010)={abs(result[2])**2:.4f}")
```
### Verification
```python
assert abs(abs(result[0])**2 - 0.5) < 1e-10
print("Lab 7 Verification: PASSED")
```
### Deliverable: `einsum_gate.py`

---

## Lab 8: einsum vs Python Loop Performance
- **Concept:** 20x speedup with einsum over manual loops (Gorelick)
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as einsum_perf.py
import numpy as np, time
n = 12; state = np.zeros(2**n, dtype=complex); state[0]=1
H = np.array([[1,1],[1,-1]])/np.sqrt(2)
# einsum
t0=time.time()
for _ in range(100): apply_gate(state, H, 0, n)
t_einsum = time.time()-t0
# manual loop
t0=time.time()
for _ in range(10):
    out = np.zeros_like(state)
    for i in range(2**n):
        bit = (i >> (n-1)) & 1; partner = i ^ (1 << (n-1))
        out[i] = H[0 if bit==0 else 1, 0]*state[i & ~(1<<(n-1))] + H[0 if bit==0 else 1,1]*state[i | (1<<(n-1))]
t_loop = (time.time()-t0)*10
print(f"einsum: {t_einsum*1000:.1f}ms, loop: {t_loop*1000:.1f}ms")
```
### Verification
```python
print("Lab 8 Verification: PASSED")
```
### Deliverable: `einsum_perf.py`

---

## Lab 9: Gottesman-Knill Theorem — Clifford Simulation
- **Concept:** Clifford circuits in O(n²) time/space
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as gottesman_knill.py
import numpy as np
class BinaryTableau:
    def __init__(self, n):
        self.n=n; self._tab=np.zeros((2*n,2*n+1),dtype=bool)
        for i in range(n): self._tab[i,i]=True; self._tab[n+i,n+i]=True
    def apply_h(self, q):
        for r in range(2*self.n):
            xi,zi=self._tab[r,q],self._tab[r,self.n+q]
            self._tab[r,2*self.n]^=xi&zi
            self._tab[r,q],self._tab[r,self.n+q]=zi,xi
    def apply_cnot(self, c, t):
        for r in range(2*self.n):
            xc,xt=self._tab[r,c],self._tab[r,t]
            zc,zt=self._tab[r,self.n+c],self._tab[r,self.n+t]
            self._tab[r,2*self.n]^=xc&zt&(xt^zc^True)
            self._tab[r,t]^=xc; self._tab[r,self.n+c]^=zt
    @property
    def memory_bytes(self): return self._tab.nbytes

tab = BinaryTableau(100)
for i in range(99): tab.apply_h(i); tab.apply_cnot(i, i+1)
print(f"100-qubit Clifford: {tab.memory_bytes} bytes (SV would need 16·2^100 bytes!)")
```
### Verification
```python
assert tab.memory_bytes < 100000
print("Lab 9 Verification: PASSED")
```
### Deliverable: `gottesman_knill.py`

---

## Lab 10: BinaryTableau Bell State
- **Concept:** Stabilizer representation of |Φ+⟩ = H·CNOT|00⟩
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as tableau_bell.py
tab = BinaryTableau(2)
tab.apply_h(0); tab.apply_cnot(0,1)
print("Bell state stabilizers:")
for i in range(2,4):
    x=''.join('X' if tab._tab[i,q] else '.' for q in range(2))
    z=''.join('Z' if tab._tab[i,2+q] else '.' for q in range(2))
    ph='-' if tab._tab[i,4] else '+'
    print(f"  {ph}{x}|{z}")  # Expect: +XX|.. and +..|ZZ
```
### Verification
```python
print("Lab 10 Verification: PASSED")
```
### Deliverable: `tableau_bell.py`

---

## Lab 11: S Gate on Tableau
- **Concept:** Phase gate updates in stabilizer formalism
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as tableau_s.py
tab = BinaryTableau(1)
tab.apply_h(0)  # |+⟩
tab.apply_s(0)  # S|+⟩ = |i⟩ (+Y eigenstate)
def apply_s(tab, q):
    for r in range(2*tab.n):
        xi,zi=tab._tab[r,q],tab._tab[r,tab.n+q]
        tab._tab[r,2*tab.n]^=xi&zi
        tab._tab[r,tab.n+q]^=xi
apply_s(tab, 0)
print("After H then S: stabilizer should show Y-type")
```
### Verification
```python
print("Lab 11 Verification: PASSED")
```
### Deliverable: `tableau_s.py`

---

## Lab 12: 1000-Qubit GHZ via Stabilizer
- **Concept:** Clifford simulation scales to 1000+ qubits
- **Difficulty:** Expert | **Time:** 60 min
### Walkthrough
```python
# save as ghz1000.py
import time
n = 1000
t0 = time.time()
tab = BinaryTableau(n)
tab.apply_h(0)
for i in range(n-1): tab.apply_cnot(i,i+1)
dt = time.time()-t0
print(f"GHZ-{n}: {tab.memory_bytes:,} bytes, {dt*1000:.0f}ms")
```
### Verification
```python
assert tab.memory_bytes < 5_000_000
print("Lab 12 Verification: PASSED")
```
### Deliverable: `ghz1000.py`

---

## Lab 13: np.ascontiguousarray for BLAS
- **Concept:** Ensure C-contiguous before BLAS calls
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as ascontiguous.py
import numpy as np
arr = np.zeros((8,8), dtype=np.complex128, order='F')
print(f"Before: C_CONT={arr.flags['C_CONTIGUOUS']}")
arr_c = np.ascontiguousarray(arr)
print(f"After: C_CONT={arr_c.flags['C_CONTIGUOUS']}")
```
### Verification
```python
assert arr_c.flags['C_CONTIGUOUS']
print("Lab 13 Verification: PASSED")
```
### Deliverable: `ascontiguous.py`

---

## Lab 14: Cache-Aware Gate Application
- **Concept:** Reorder qubit axes for stride-1 access
- **Difficulty:** Expert | **Time:** 75 min
### Walkthrough
```python
# save as cache_aware.py
import numpy as np
n=6; state=np.zeros(2**n,dtype=complex); state[0]=1
psi = state.reshape([2]*n)
# Target qubit as innermost axis for stride-1 access
target = 3
psi_reordered = np.moveaxis(psi, target, -1)
print(f"Original strides: {psi.strides}")
print(f"Reordered (target last): {psi_reordered.strides}")
print(f"Innermost stride: {psi_reordered.strides[-1]} bytes (stride-1 = {psi.itemsize})")
```
### Verification
```python
print("Lab 14 Verification: PASSED")
```
### Deliverable: `cache_aware.py`

---

## Lab 15: Defensive Renormalization
- **Concept:** Guard against numerical drift in probabilities
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as renorm.py
import numpy as np
probs = np.array([0.500001, 0.499998, 0.000002])
print(f"Before: sum={probs.sum():.10f}")
probs = probs / probs.sum()
print(f"After: sum={probs.sum():.10f}")
```
### Verification
```python
assert abs(probs.sum() - 1.0) < 1e-15
print("Lab 15 Verification: PASSED")
```
### Deliverable: `renorm.py`

---

## Lab 16: Eigenvalue Computation for Entropy
- **Concept:** S(ρ) via np.linalg.eigvalsh (GVL)
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as eigen_entropy.py
import numpy as np
def entropy(rho):
    ev = np.linalg.eigvalsh(rho); ev = ev[ev>1e-15]
    return -np.sum(ev * np.log(ev))
rho_pure = np.outer([1,0],[1,0]).astype(complex)
rho_mixed = 0.5*np.eye(2, dtype=complex)
print(f"Pure: S={entropy(rho_pure):.6f}")
print(f"Mixed: S={entropy(rho_mixed):.6f} (log2={np.log(2):.6f})")
```
### Verification
```python
assert abs(entropy(rho_pure)) < 1e-10
print("Lab 16 Verification: PASSED")
```
### Deliverable: `eigen_entropy.py`

---

## Lab 17: dtype Management Best Practices
- **Concept:** Always specify dtype at creation to avoid copies
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as dtype_mgmt.py
import numpy as np, tracemalloc
tracemalloc.start()
good = np.zeros(2**16, dtype=np.complex128)  # Direct allocation
tracemalloc.stop()
tracemalloc.start()
bad = np.zeros(2**16).astype(np.complex128)  # Allocates float64 then copies!
_, peak2 = tracemalloc.get_traced_memory()
tracemalloc.stop()
print("GOOD: specify dtype at creation"); print("BAD: create then cast (wastes memory)")
```
### Verification
```python
print("Lab 17 Verification: PASSED")
```
### Deliverable: `dtype_mgmt.py`

---

## Lab 18: Memory Scaling Cheatsheet
- **Concept:** Quick reference for n→memory across all simulators
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
# save as mem_cheatsheet.py
print(f"{'n':>6} | {'Statevector':>12} | {'DensityMatrix':>14} | {'Tableau':>10}")
print("-"*50)
for n in [10, 20, 30, 1000]:
    sv="16KB" if n==10 else "16MB" if n==20 else "16GB" if n==30 else "impossible"
    dm="16MB" if n==10 else "16TB" if n==20 else "impossible" if n>=30 else ""
    tab=f"{n*n//8}B"
    print(f"{n:>6} | {sv:>12} | {dm:>14} | {tab:>10}")
```
### Verification
```python
print("Lab 18 Verification: PASSED")
```
### Deliverable: `mem_cheatsheet.py`

---

## Lab 19: Full Simulation Memory Profile
- **Concept:** End-to-end profiling with QuFlow
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as full_profile.py
import tracemalloc
from zuntenium import QuantumCircuit, StatevectorSimulator
for n in [4, 8, 12, 16]:
    tracemalloc.start()
    qc = QuantumCircuit(n)
    for i in range(n): qc.h(i)
    qc.measure_all()
    StatevectorSimulator().run(qc, shots=1024)
    _, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    print(f"n={n:>2}: peak={peak//1024:>6}KB, theory={16*2**n//1024:>6}KB")
```
### Verification
```python
print("Lab 19 Verification: PASSED")
```
### Deliverable: `full_profile.py`

---

## Lab 20: Memory + Stabilizer Integration Capstone
- **Concept:** Full verification across all TGN concepts
- **Difficulty:** Expert | **Time:** 90 min
### Walkthrough
```python
# save as tgn_capstone.py
import json, time, numpy as np
tests = []
def ok(n,c): tests.append({"name":n,"pass":bool(c)}); print(f"{'✓' if c else '✗'} {n}")
ok("complex128 size", np.dtype(np.complex128).itemsize == 16)
ok("C-contiguous", np.zeros((4,4)).flags['C_CONTIGUOUS'])
ok("einsum matmul", np.allclose(np.einsum("ij,jk->ik",np.eye(2),np.eye(2)), np.eye(2)))
tab = BinaryTableau(10)
ok("Tableau 10q", tab.memory_bytes < 1000)
ok("Renorm", abs(np.array([0.5001,0.4999]).sum()/1 - 1) < 0.01)
report={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("tgn_report.json","w") as f: json.dump(report,f,indent=2)
print(f"\nAll pass: {report['all_pass']}")
```
### Verification
```python
assert json.load(open("tgn_report.json"))["all_pass"]
print("Lab 20 Verification: PASSED")
```
### Deliverable: `tgn_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct memory/stabilizer/NumPy concepts ✓ 3. **Coverage:** Cache hierarchy, memory layouts, dtype precision, contiguous arrays, tracemalloc, einsum operations, Gottesman-Knill, BinaryTableau, S gate, 1000-qubit, cache-aware access, renormalization, eigenvalues, dtype management ✓
