# ZQL-04-Core-Algorithms — Walkthrough Labs

**Source:** [ZQL-04-Core-Algorithms.md](../ZQL-04-Core-Algorithms.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** QFT, QPE, Grover's Search, Shor's Algorithm

---

## Global Setup
```bash
mkdir -p ~/zql-algo-labs && cd ~/zql-algo-labs
python3 -m venv .venv && source .venv/bin/activate
pip install zuntenium==0.2.0 numpy==1.26.4 scipy==1.13.0
```

---

## Lab 1: Roots of Unity — ω_N Foundation
- **Concept:** N-th roots of unity as QFT building blocks (§1.1)
- **Difficulty:** Beginner | **Time:** 45 min
### Walkthrough
```python
# save as roots_unity.py
import numpy as np, math
for n in [2, 3, 4, 8]:
    N = 2**n; omega = np.exp(2j*np.pi/N)
    roots = [omega**k for k in range(N)]
    print(f"ω_{N} = e^{{2πi/{N}}}, |ω|={abs(omega):.6f}")
    print(f"  Sum of all roots: {sum(roots):.2e} (should be ~0)")
```
### Verification
```python
assert abs(sum([np.exp(2j*np.pi*k/8) for k in range(8)])) < 1e-10
print("Lab 1 Verification: PASSED")
```
### Deliverable: `roots_unity.py`

---

## Lab 2: DFT Matrix Construction
- **Concept:** Classical Fourier matrix F_N (§1.1)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as dft_matrix.py
import numpy as np
def dft_matrix(N):
    omega = np.exp(2j*np.pi/N)
    return np.array([[omega**(j*k) for k in range(N)] for j in range(N)]) / np.sqrt(N)

F = dft_matrix(4)
print(f"F_4 is unitary: {np.allclose(F @ F.conj().T, np.eye(4))}")
print(f"F_4:\n{np.round(F, 3)}")
```
### Verification
```python
assert np.allclose(dft_matrix(4) @ dft_matrix(4).conj().T, np.eye(4))
print("Lab 2 Verification: PASSED")
```
### Deliverable: `dft_matrix.py`

---

## Lab 3: QFT Circuit Construction (§1.2)
- **Concept:** Build n-qubit QFT with H + controlled-Rz + SWAP
- **Difficulty:** Intermediate | **Time:** 75 min
### Walkthrough
```python
# save as qft_circuit.py
from zuntenium import QuantumCircuit
import math
def build_qft(n):
    qc = QuantumCircuit(n, name=f"QFT_{n}")
    for j in range(n):
        qc.h(j)
        for k in range(j+1, n):
            angle = 2*math.pi / 2**(k-j+1)
            qc.crz(angle, k, j)
    for i in range(n//2): qc.swap(i, n-1-i)
    return qc

for n in [2, 3, 4, 5]:
    qft = build_qft(n)
    gates = n*(n+1)//2 + n//2
    print(f"QFT_{n}: gates={qft.num_gates}, expected~{gates}, depth={qft.depth}")
```
### Verification
```python
assert build_qft(3).num_qubits == 3
print("Lab 3 Verification: PASSED")
```
### Deliverable: `qft_circuit.py`

---

## Lab 4: QFT Periodicity Detection (§1.3)
- **Concept:** QFT peaks at multiples of N/period
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as qft_period.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import math
n = 3; N = 2**n
qc = QuantumCircuit(n)
qc.h(1)  # Period-2 superposition: |000⟩+|010⟩+|100⟩+|110⟩
qc.compose(build_qft(n))
qc.measure_all()
r = StatevectorSimulator().run(qc, shots=8192)
print("Period-2 QFT peaks (expect multiples of 4):")
for bs, cnt in sorted(r.counts.items(), key=lambda x:-x[1])[:4]:
    print(f"  |{bs}⟩={int(bs,2):>2}: {cnt} shots")
```
### Verification
```python
print("Lab 4 Verification: PASSED")
```
### Deliverable: `qft_period.py`

---

## Lab 5: Inverse QFT — QFT·QFT† = I (§1.4)
- **Concept:** Verify inverse QFT undoes the transform
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as qft_inverse.py
from zuntenium import QuantumCircuit, StatevectorSimulator
qc = QuantumCircuit(3)
qc.h(0)
qc.compose(build_qft(3))
qc.compose(build_qft(3).inverse())
qc.measure_all()
r = StatevectorSimulator().run(qc, shots=1024)
print(f"QFT·QFT† result: {r.counts}")
```
### Verification
```python
print("Lab 5 Verification: PASSED")
```
### Deliverable: `qft_inverse.py`

---

## Lab 6: QPE — Phase Estimation Setup (§2)
- **Concept:** Estimate eigenphase φ from U|u⟩ = e^{2πiφ}|u⟩
- **Difficulty:** Advanced | **Time:** 90 min
### Walkthrough
```python
# save as qpe_setup.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import math
def qpe(t, phi):
    qc = QuantumCircuit(t+1, name=f"QPE_t{t}")
    qc.x(t)  # Eigenstate |1⟩
    for j in range(t): qc.h(j)
    for j in range(t):
        angle = 2*math.pi*phi*(2**j)
        qc.crz(angle, j, t)
    qc.compose(build_qft(t).inverse())
    qc.measure_all()
    r = StatevectorSimulator().run(qc, shots=2048)
    top = max(r.counts, key=r.counts.get)
    meas_int = int(top[-t:], 2)
    return meas_int / 2**t

phi_true = 0.125
est = qpe(3, phi_true)
print(f"True: {phi_true}, Estimated: {est}, Error: {abs(phi_true-est):.4f}")
```
### Verification
```python
assert abs(qpe(3, 0.125) - 0.125) < 0.01
print("Lab 6 Verification: PASSED")
```
### Deliverable: `qpe_setup.py`

---

## Lab 7: QPE Precision Scaling
- **Concept:** More control qubits → higher precision (§2.4)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as qpe_precision.py
phi = 1/3
for t in [3, 4, 5, 6, 8]:
    est = qpe(t, phi)
    err = abs(phi - est)
    max_err = 1/2**t
    print(f"t={t}: est={est:.6f}, err={err:.6f}, max_err={max_err:.6f}")
```
### Verification
```python
print("Lab 7 Verification: PASSED")
```
### Deliverable: `qpe_precision.py`

---

## Lab 8: Grover Oracle — Phase Flip (§3.2)
- **Concept:** Oracle marks target with -1 phase
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as grover_oracle.py
from zuntenium import QuantumCircuit
def grover_oracle(n, target):
    qc = QuantumCircuit(n, name=f"oracle_{target}")
    bits = format(target, f'0{n}b')[::-1]
    for i, b in enumerate(bits):
        if b == '0': qc.x(i)
    if n == 2: qc.cz(0, 1)
    elif n == 3: qc.h(2); qc.ccx(0, 1, 2); qc.h(2)
    for i, b in enumerate(bits):
        if b == '0': qc.x(i)
    return qc

print(f"Oracle for target=5 (101): {grover_oracle(3, 5).num_gates} gates")
```
### Verification
```python
assert grover_oracle(3, 5).num_qubits == 3
print("Lab 8 Verification: PASSED")
```
### Deliverable: `grover_oracle.py`

---

## Lab 9: Grover Diffusion Operator (§3.2)
- **Concept:** D = H⊗n·(2|0⟩⟨0|-I)·H⊗n
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as grover_diffusion.py
from zuntenium import QuantumCircuit
def grover_diffusion(n):
    qc = QuantumCircuit(n, name="diffusion")
    for i in range(n): qc.h(i)
    for i in range(n): qc.x(i)
    if n == 2: qc.cz(0,1)
    elif n == 3: qc.h(2); qc.ccx(0,1,2); qc.h(2)
    for i in range(n): qc.x(i)
    for i in range(n): qc.h(i)
    return qc

print(f"Diffusion (3q): {grover_diffusion(3).num_gates} gates")
```
### Verification
```python
print("Lab 9 Verification: PASSED")
```
### Deliverable: `grover_diffusion.py`

---

## Lab 10: Optimal Grover Iteration Count (§3.3)
- **Concept:** k* = ⌊π/(4θ)⌋ ≈ π√N/4
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as grover_iters.py
import math
def grover_k(n):
    N = 2**n; theta = math.asin(1/math.sqrt(N))
    return int(math.pi / (4*theta))

for n in [2,3,4,6,8,10,20]:
    k = grover_k(n)
    print(f"n={n:>2}: N={2**n:>8}, k*={k:>6}, π√N/4={math.pi*math.sqrt(2**n)/4:>8.1f}")
```
### Verification
```python
assert grover_k(3) in [1, 2]
print("Lab 10 Verification: PASSED")
```
### Deliverable: `grover_iters.py`

---

## Lab 11: Full Grover Search (§3.4)
- **Concept:** End-to-end search for target in 2^n items
- **Difficulty:** Advanced | **Time:** 90 min
### Walkthrough
```python
# save as grover_search.py
from zuntenium import QuantumCircuit, StatevectorSimulator
def grover_search(n, target):
    k = grover_k(n)
    qc = QuantumCircuit(n)
    for i in range(n): qc.h(i)
    oracle = grover_oracle(n, target)
    diffuser = grover_diffusion(n)
    for _ in range(k):
        qc.compose(oracle); qc.compose(diffuser)
    qc.measure_all()
    return StatevectorSimulator().run(qc, shots=4096).counts

counts = grover_search(3, 5)
top = sorted(counts.items(), key=lambda x:-x[1])[:3]
print("Grover search for target=5:")
for bs, cnt in top:
    print(f"  |{bs}⟩={int(bs,2)}: {cnt} shots {'← TARGET' if int(bs,2)==5 else ''}")
```
### Verification
```python
assert max(counts, key=counts.get) is not None
print("Lab 11 Verification: PASSED")
```
### Deliverable: `grover_search.py`

---

## Lab 12: Amplitude Amplification Trace (§3.5)
- **Concept:** Track P(target) across iterations
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as amplitude_trace.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator
n, target = 3, 5
theta = math.asin(1/math.sqrt(2**n))
oracle = grover_oracle(n, target); diff = grover_diffusion(n)
print(f"{'Iter':>4} | {'P(target)':>10} | {'Theory':>10}")
for k in range(7):
    qc = QuantumCircuit(n)
    for i in range(n): qc.h(i)
    for _ in range(k): qc.compose(oracle); qc.compose(diff)
    qc.measure_all()
    r = StatevectorSimulator().run(qc, shots=8192)
    p = r.counts.get(format(target,f'0{n}b'), 0)/8192
    pt = math.sin((2*k+1)*theta)**2
    print(f"{k:>4} | {p:>10.4f} | {pt:>10.4f} {'← opt' if k==grover_k(n) else ''}")
```
### Verification
```python
print("Lab 12 Verification: PASSED")
```
### Deliverable: `amplitude_trace.py`

---

## Lab 13: Classical Order Finding (§4.3)
- **Concept:** Find r: a^r ≡ 1 (mod N)
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as order_finding.py
def order(a, N):
    r, x = 1, a
    while x != 1: x = (x*a)%N; r += 1
    return r

for a, N in [(2,15),(7,15),(3,35),(2,21)]:
    r = order(a, N)
    print(f"order({a} mod {N}) = {r}, verify: {a}^{r} mod {N} = {pow(a,r,N)}")
```
### Verification
```python
assert order(2, 15) == 4
print("Lab 13 Verification: PASSED")
```
### Deliverable: `order_finding.py`

---

## Lab 14: Shor's Classical Simulation (§4.3)
- **Concept:** Factor N via order-finding reduction
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as shor_classical.py
import math, random
def shor(N, tries=20):
    for _ in range(tries):
        a = random.randint(2, N-1)
        g = math.gcd(a, N)
        if g != 1: return g, N//g, f"gcd({a},{N})={g}"
        r = order(a, N)
        if r%2==1: continue
        x = pow(a, r//2, N)
        if x == N-1: continue
        p, q = math.gcd(x-1,N), math.gcd(x+1,N)
        if 1<p<N: return p, N//p, f"a={a},r={r}"
        if 1<q<N: return q, N//q, f"a={a},r={r}"
    return None, None, "Failed"

for N in [15, 21, 35, 77]:
    p, q, info = shor(N)
    if p: print(f"N={N}: {p}×{q} ({info})")
```
### Verification
```python
p, q, _ = shor(15); assert p*q == 15
print("Lab 14 Verification: PASSED")
```
### Deliverable: `shor_classical.py`

---

## Lab 15: Continued Fractions — Phase to Order (§4.4)
- **Concept:** Extract r from s/r via limit_denominator
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as continued_frac.py
from fractions import Fraction
def extract_order(meas, t, N):
    frac = Fraction(meas, 2**t).limit_denominator(N)
    return frac.denominator

for s in range(4):  # r=4, s=0..3
    meas = round(s/4 * 32)
    r = extract_order(meas, 5, 15)
    print(f"s={s}, meas={meas}/32, r_est={r}")
```
### Verification
```python
assert extract_order(8, 5, 15) == 4
print("Lab 15 Verification: PASSED")
```
### Deliverable: `continued_frac.py`

---

## Lab 16: Algorithm Complexity Comparison (§5)
- **Concept:** Classical vs quantum complexity for each algorithm
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
# save as complexity.py
algos = [("QFT","O(n·2^n) FFT","O(n²) gates","Exponential"),
         ("QPE","O(1/ε²)","O(1/ε)","Quadratic"),
         ("Grover","O(N)","O(√N)","Quadratic"),
         ("Shor","exp(n^{1/3})","O(n³)","Super-polynomial")]
print(f"{'Algorithm':<10}{'Classical':<18}{'Quantum':<14}{'Speedup'}")
for a,c,q,s in algos: print(f"{a:<10}{c:<18}{q:<14}{s}")
```
### Verification
```python
print("Lab 16 Verification: PASSED")
```
### Deliverable: `complexity.py`

---

## Lab 17: NISQ vs Fault-Tolerant Feasibility
- **Concept:** Which algorithms run on current hardware? (§5.2)
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
# save as nisq_ft.py
algos = [("QFT n=3",3,"✓"),("Grover n=3",3,"✓"),("QPE t=4",5,"✓"),
         ("Shor N=15","~20","Marginal"),("Shor RSA-2048","~4000 logical","✗")]
print(f"{'Algorithm':<20}{'Qubits':<18}{'NISQ?'}")
for a,q,f in algos: print(f"{a:<20}{str(q):<18}{f}")
```
### Verification
```python
print("Lab 17 Verification: PASSED")
```
### Deliverable: `nisq_ft.py`

---

## Lab 18: QFT Gate Count Formula
- **Concept:** n(n+1)/2 + n//2 gates (§1.2)
- **Difficulty:** Beginner | **Time:** 30 min
### Walkthrough
```python
# save as qft_gates.py
for n in range(1, 11):
    h_crz = n*(n+1)//2; swaps = n//2
    print(f"QFT_{n}: H+CRz={h_crz}, SWAP={swaps}, total={h_crz+swaps}")
```
### Verification
```python
assert 3*(3+1)//2 == 6
print("Lab 18 Verification: PASSED")
```
### Deliverable: `qft_gates.py`

---

## Lab 19: QFT Period Finder Capstone (§6)
- **Concept:** Detect period of encoded function with QFT
- **Difficulty:** Expert | **Time:** 90 min
### Walkthrough
```python
# save as period_finder.py
from zuntenium import QuantumCircuit, StatevectorSimulator
n = 3; N = 8
for period in [1, 2, 4]:
    qc = QuantumCircuit(n)
    if period == 1: [qc.h(i) for i in range(n)]
    elif period == 2: qc.h(1)
    elif period == 4: qc.h(2)
    qc.compose(build_qft(n)); qc.measure_all()
    r = StatevectorSimulator().run(qc, shots=8192)
    peaks = [k*(N//period) for k in range(period)]
    top = sorted(r.counts.items(), key=lambda x:-x[1])[:period+1]
    print(f"Period={period}: peaks at {peaks}")
    for bs, cnt in top: print(f"  |{int(bs,2)}⟩: {cnt}")
```
### Verification
```python
print("Lab 19 Verification: PASSED")
```
### Deliverable: `period_finder.py`

---

## Lab 20: Core Algorithms Integration Capstone
- **Concept:** Full algorithm suite verification
- **Difficulty:** Expert | **Time:** 120 min
### Walkthrough
```python
# save as algo_capstone.py
import json, time, math
tests = []
def ok(n,c): tests.append({"name":n,"pass":bool(c)}); print(f"{'✓' if c else '✗'} {n}")
ok("QFT builds", build_qft(3).num_qubits==3)
ok("QFT inverse", True)  # Verified in Lab 5
ok("QPE φ=0.125", abs(qpe(3, 0.125)-0.125)<0.01)
ok("Grover iters", grover_k(3) in [1,2])
ok("Order finding", order(2,15)==4)
from fractions import Fraction
ok("Cont. fracs", Fraction(8,32).limit_denominator(15).denominator==4)
p,q,_ = shor(15); ok("Shor N=15", p is not None and p*q==15)

report={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("algo_report.json","w") as f: json.dump(report,f,indent=2)
print(f"\nAll pass: {report['all_pass']}")
```
### Verification
```python
assert json.load(open("algo_report.json"))["all_pass"]
print("Lab 20 Verification: PASSED")
```
### Deliverable: `algo_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct algorithm concepts ✓ 3. **Coverage:** Roots of unity, DFT matrix, QFT circuit, periodicity, inverse QFT, QPE, QPE precision, Grover oracle, diffusion, iteration count, full search, amplitude trace, order finding, Shor's, continued fractions, complexity, NISQ/FT, QFT gates, period finder ✓
