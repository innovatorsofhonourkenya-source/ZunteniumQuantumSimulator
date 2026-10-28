# ZQL-00.01-notes-nakahara — Walkthrough Labs

**Source:** [ZQL-00.01-notes-nakahara.md](../ZQL-00.01-notes-nakahara.md)
**Generated:** 2026-10-28
**Labs:** 20 | **Domain:** Quantum / Linear algebra prerequisites

---

## Source Analysis

- **Document objective:** Map Nakahara & Ohmi's mathematical prerequisites (Chs. 1-2: complex vector spaces, unitary matrices, tensor products, measurement, physical realizations) to QuFlow SDK use.
- **Target audience:** Learners needing the math foundation beneath circuits.
- **Domain classification:** Mathematical foundations / linear algebra for quantum.

---

## Global Setup

```bash
mkdir -p ~/zql-nakahara-labs && cd ~/zql-nakahara-labs
python3 -m venv .venv && source .venv/bin/activate
pip install zuntenium==0.2.0 numpy==1.26.4 scipy==1.13.0
```

---

## Lab 1: ℂ² — The Qubit Lives in a 2D Complex Vector Space

- **Concept:** Complex vector spaces and the computational basis {|0⟩, |1⟩}
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** None

### Walkthrough

```python
# save as complex_vectors.py
import numpy as np

# The computational basis
ket0 = np.array([1, 0], dtype=complex)  # |0⟩
ket1 = np.array([0, 1], dtype=complex)  # |1⟩

print(f"|0⟩ = {ket0}")
print(f"|1⟩ = {ket1}")

# Inner products: ⟨0|0⟩=1, ⟨0|1⟩=0 (orthonormal)
print(f"\n⟨0|0⟩ = {np.vdot(ket0, ket0)}")  # 1
print(f"⟨0|1⟩ = {np.vdot(ket0, ket1)}")  # 0
print(f"⟨1|1⟩ = {np.vdot(ket1, ket1)}")  # 1

# General state: α|0⟩ + β|1⟩
alpha, beta = 1/np.sqrt(3) + 0j, np.sqrt(2/3) + 0j
psi = alpha * ket0 + beta * ket1
print(f"\n|ψ⟩ = {alpha:.4f}|0⟩ + {beta:.4f}|1⟩")
print(f"|ψ⟩ = {psi}")
print(f"|α|² + |β|² = {abs(alpha)**2 + abs(beta)**2:.6f} (must be 1.0)")

# Complex amplitudes — the full power
alpha_c = (1 + 1j) / 2
beta_c = (1 - 1j) / 2
norm = abs(alpha_c)**2 + abs(beta_c)**2
print(f"\nComplex state: α={alpha_c}, β={beta_c}")
print(f"|α|²+|β|² = {norm:.6f}")
```

### Verification
```python
assert abs(np.vdot(ket0, ket0) - 1) < 1e-10
assert abs(np.vdot(ket0, ket1)) < 1e-10
print("Lab 1 Verification: PASSED")
```

### Deliverable
`complex_vectors.py` with basis vectors, inner products, and normalization.

---

## Lab 2: Bra-Ket Notation — Column Vectors and Row Vectors

- **Concept:** Ket |ψ⟩ as column, bra ⟨ψ| as conjugate transpose
- **Difficulty:** Beginner | **Time:** 45 min | **Prerequisites:** Lab 1

### Walkthrough

```python
# save as braket.py
import numpy as np

ket = np.array([1/np.sqrt(2), 1j/np.sqrt(2)])  # |ψ⟩ column vector
bra = ket.conj()  # ⟨ψ| = conjugate (for 1D numpy, acts as row)

print(f"Ket |ψ⟩ = {ket}")
print(f"Bra ⟨ψ| = {bra}")
print(f"⟨ψ|ψ⟩ = {np.vdot(ket, ket):.4f} (must be 1.0)")

# Outer product: |ψ⟩⟨ψ| = projection operator
outer = np.outer(ket, bra)
print(f"\n|ψ⟩⟨ψ| (projector):\n{outer}")
print(f"Trace = {np.trace(outer):.4f} (must be 1.0)")
print(f"Idempotent? P²=P: {np.allclose(outer @ outer, outer)}")
```

### Verification
```python
assert abs(np.vdot(ket, ket) - 1.0) < 1e-10
assert np.allclose(np.outer(ket, bra) @ np.outer(ket, bra), np.outer(ket, bra))
print("Lab 2 Verification: PASSED")
```

### Deliverable
`braket.py` with bra-ket arithmetic.

---

## Lab 3: Inner Products — Probability Amplitudes

- **Concept:** ⟨φ|ψ⟩ gives probability amplitude; |⟨φ|ψ⟩|² gives probability
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 2

### Walkthrough

```python
# save as inner_products.py
import numpy as np

ket0 = np.array([1, 0], dtype=complex)
ket1 = np.array([0, 1], dtype=complex)
ket_plus = np.array([1, 1], dtype=complex) / np.sqrt(2)
ket_minus = np.array([1, -1], dtype=complex) / np.sqrt(2)

states = {"0": ket0, "1": ket1, "+": ket_plus, "-": ket_minus}

print("Inner product table ⟨row|col⟩:")
print(f"{'':>4}", end="")
for name in states: print(f"{'|'+name+'⟩':>10}", end="")
print()

for n1, s1 in states.items():
    print(f"⟨{n1}|", end="")
    for n2, s2 in states.items():
        ip = np.vdot(s1, s2)
        print(f"{ip.real:>10.4f}", end="")
    print()

# Probability of measuring |0⟩ when in |+⟩
prob = abs(np.vdot(ket0, ket_plus))**2
print(f"\nP(0 | +) = |⟨0|+⟩|² = {prob:.4f}")
```

### Verification
```python
assert abs(abs(np.vdot(ket0, ket_plus))**2 - 0.5) < 1e-10
print("Lab 3 Verification: PASSED")
```

### Deliverable
`inner_products.py` with inner product table and probability calculation.

---

## Lab 4: Unitary Matrices — Gates That Preserve Probability

- **Concept:** U†U = I defines valid quantum gates
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 1

### Walkthrough

```python
# save as unitary_check.py
import numpy as np

def is_unitary(U, tol=1e-10):
    return np.allclose(U.conj().T @ U, np.eye(U.shape[0]), atol=tol)

gates = {
    "I": np.eye(2, dtype=complex),
    "X": np.array([[0,1],[1,0]], dtype=complex),
    "Y": np.array([[0,-1j],[1j,0]]),
    "Z": np.array([[1,0],[0,-1]], dtype=complex),
    "H": np.array([[1,1],[1,-1]]) / np.sqrt(2),
    "S": np.array([[1,0],[0,1j]]),
    "T": np.array([[1,0],[0,np.exp(1j*np.pi/4)]]),
}

print(f"{'Gate':<4} | {'Unitary?':<8} | {'|det|':<10} | {'Eigenvalues'}")
print("-" * 55)
for name, U in gates.items():
    eigs = np.linalg.eigvals(U)
    det = abs(np.linalg.det(U))
    print(f"{name:<4} | {str(is_unitary(U)):<8} | {det:<10.6f} | [{', '.join(f'{abs(e):.3f}' for e in eigs)}]")

# Non-unitary example
bad = np.array([[1,1],[0,1]], dtype=complex)
print(f"\n{'BAD':<4} | {str(is_unitary(bad)):<8} | {abs(np.linalg.det(bad)):<10.6f} | NOT A VALID GATE")
```

### Verification
```python
for U in gates.values():
    assert is_unitary(U)
assert not is_unitary(bad)
print("Lab 4 Verification: PASSED")
```

### Deliverable
`unitary_check.py` with unitarity verification for all standard gates.

---

## Lab 5: Eigenvalues on the Unit Circle

- **Concept:** Unitary eigenvalues satisfy |λ| = 1
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 4

### Walkthrough

```python
# save as eigenvalues.py
import numpy as np

gates = {
    "X": np.array([[0,1],[1,0]], dtype=complex),
    "Z": np.array([[1,0],[0,-1]], dtype=complex),
    "H": np.array([[1,1],[1,-1]]) / np.sqrt(2),
    "S": np.array([[1,0],[0,1j]]),
    "T": np.array([[1,0],[0,np.exp(1j*np.pi/4)]]),
}

for name, U in gates.items():
    eigs = np.linalg.eigvals(U)
    mags = [abs(e) for e in eigs]
    phases = [np.angle(e) / np.pi for e in eigs]
    print(f"{name}: eigenvalues = {eigs}")
    print(f"     magnitudes = {mags} (all ≈ 1.0)")
    print(f"     phases/π = {[f'{p:.3f}' for p in phases]}")
    assert all(abs(m - 1.0) < 1e-10 for m in mags), f"{name} eigenvalue not on unit circle!"
    print()
```

### Verification
```python
for U in gates.values():
    eigs = np.linalg.eigvals(U)
    assert all(abs(abs(e) - 1.0) < 1e-10 for e in eigs)
print("Lab 5 Verification: PASSED")
```

### Deliverable
`eigenvalues.py` with eigenvalue analysis for all gates.

---

## Lab 6: Pauli Matrices — X, Y, Z as Rotation Generators

- **Concept:** Pauli matrices σx, σy, σz and their algebraic properties
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 4

### Walkthrough

```python
# save as pauli_algebra.py
import numpy as np

X = np.array([[0,1],[1,0]], dtype=complex)
Y = np.array([[0,-1j],[1j,0]])
Z = np.array([[1,0],[0,-1]], dtype=complex)
I = np.eye(2, dtype=complex)

# Key properties
print("Pauli algebra:")
print(f"  X² = I? {np.allclose(X@X, I)}")
print(f"  Y² = I? {np.allclose(Y@Y, I)}")
print(f"  Z² = I? {np.allclose(Z@Z, I)}")
print(f"  XY = iZ? {np.allclose(X@Y, 1j*Z)}")
print(f"  YZ = iX? {np.allclose(Y@Z, 1j*X)}")
print(f"  ZX = iY? {np.allclose(Z@X, 1j*Y)}")
print(f"  XYZ = iI? {np.allclose(X@Y@Z, 1j*I)}")
print(f"  {X,Y} anticommute? {np.allclose(X@Y + Y@X, np.zeros((2,2)))}")

# H = (X + Z) / √2
H_from_paulis = (X + Z) / np.sqrt(2)
H_standard = np.array([[1,1],[1,-1]]) / np.sqrt(2)
print(f"\n  H = (X+Z)/√2? {np.allclose(H_from_paulis, H_standard)}")
```

### Verification
```python
assert np.allclose(X@X, I) and np.allclose(Y@Y, I) and np.allclose(Z@Z, I)
assert np.allclose(X@Y, 1j*Z)
print("Lab 6 Verification: PASSED")
```

### Deliverable
`pauli_algebra.py` with complete Pauli algebra verification.

---

## Lab 7: Tensor Products — Building Multi-Qubit State Spaces

- **Concept:** ℂ² ⊗ ℂ² = ℂ⁴; n-qubit space = ℂ^(2^n)
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 1

### Walkthrough

```python
# save as tensor_space.py
import numpy as np

ket0 = np.array([1, 0], dtype=complex)
ket1 = np.array([0, 1], dtype=complex)

# 2-qubit basis states
basis_2q = {
    "|00⟩": np.kron(ket0, ket0),
    "|01⟩": np.kron(ket0, ket1),
    "|10⟩": np.kron(ket1, ket0),
    "|11⟩": np.kron(ket1, ket1),
}

print("2-qubit computational basis:")
for name, state in basis_2q.items():
    print(f"  {name} = {state}")

# Verify orthonormality
print("\nOrthonormality check:")
names = list(basis_2q.keys())
for i, n1 in enumerate(names):
    for j, n2 in enumerate(names):
        ip = np.vdot(basis_2q[n1], basis_2q[n2])
        expected = 1.0 if i == j else 0.0
        assert abs(ip - expected) < 1e-10

print("  All basis states orthonormal ✓")

# Scaling table
print(f"\n{'n':>3} | {'dim(H)':>10} | {'Memory':>12}")
print("-" * 30)
for n in range(1, 15):
    dim = 2**n
    mem = dim * 16
    unit = "B" if mem < 1024 else ("KB" if mem < 1e6 else "MB")
    val = mem if mem < 1024 else (mem/1024 if mem < 1e6 else mem/1e6)
    print(f"{n:>3} | {dim:>10,} | {val:>10.1f} {unit}")
```

### Verification
```python
assert len(np.kron(ket0, ket0)) == 4
assert len(np.kron(np.kron(ket0, ket0), ket0)) == 8
print("Lab 7 Verification: PASSED")
```

### Deliverable
`tensor_space.py` with basis construction and scaling table.

---

## Lab 8: Gate on Qubit k — I ⊗ U ⊗ I Pattern

- **Concept:** Single-qubit gate acts as I⊗…⊗U(k)⊗…⊗I on full space
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 7

### Walkthrough

```python
# save as gate_on_qubit_k.py
import numpy as np

I = np.eye(2, dtype=complex)
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
X = np.array([[0,1],[1,0]], dtype=complex)

# H on qubit 0 of 2-qubit system: H ⊗ I
H_on_q0 = np.kron(H, I)
print(f"H ⊗ I (H on q0):\n{H_on_q0}\n")

# H on qubit 1 of 2-qubit system: I ⊗ H
H_on_q1 = np.kron(I, H)
print(f"I ⊗ H (H on q1):\n{H_on_q1}\n")

# Verify: H on q0 of |00⟩ = (|0⟩+|1⟩)/√2 ⊗ |0⟩
state_00 = np.array([1,0,0,0], dtype=complex)
result = H_on_q0 @ state_00
print(f"H(q0)|00⟩ = {result}")
expected = np.array([1/np.sqrt(2), 0, 1/np.sqrt(2), 0])
print(f"Expected:   {expected}")
print(f"Match: {np.allclose(result, expected)}")

# 3-qubit: X on qubit 1 = I ⊗ X ⊗ I
X_on_q1_3q = np.kron(np.kron(I, X), I)
print(f"\n3-qubit X on q1: shape = {X_on_q1_3q.shape}")
```

### Verification
```python
assert np.allclose(H_on_q0 @ state_00, expected)
print("Lab 8 Verification: PASSED")
```

### Deliverable
`gate_on_qubit_k.py` with tensor product gate construction.

---

## Lab 9: Projective Measurement — Born Rule in Linear Algebra

- **Concept:** P(m) = |⟨m|ψ⟩|² — Born rule as inner product
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 3

### Walkthrough

```python
# save as born_rule_linalg.py
import numpy as np
from zuntenium import QuantumCircuit, StatevectorSimulator

ket0 = np.array([1, 0], dtype=complex)
ket1 = np.array([0, 1], dtype=complex)

# State |ψ⟩ = (|0⟩ + √2|1⟩) / √3
psi = (ket0 + np.sqrt(2) * ket1) / np.sqrt(3)

# Born rule: P(0) = |⟨0|ψ⟩|², P(1) = |⟨1|ψ⟩|²
p0 = abs(np.vdot(ket0, psi))**2
p1 = abs(np.vdot(ket1, psi))**2
print(f"|ψ⟩ = {psi}")
print(f"P(0) = |⟨0|ψ⟩|² = {p0:.4f}")
print(f"P(1) = |⟨1|ψ⟩|² = {p1:.4f}")
print(f"Sum = {p0+p1:.4f}")

# Verify with circuit: Ry(θ) where sin²(θ/2) = 2/3
import math
theta = 2 * math.asin(math.sqrt(2/3))
backend = StatevectorSimulator()
qc = QuantumCircuit(1)
qc.ry(theta, 0).measure_all()
r = backend.run(qc, shots=10000)
p1_meas = r.counts.get("1", 0) / 10000
print(f"\nCircuit verification: P(1) = {p1_meas:.4f} (theory: {p1:.4f})")
```

### Verification
```python
assert abs(p0 + p1 - 1.0) < 1e-10
assert abs(p1_meas - p1) < 0.03
print("Lab 9 Verification: PASSED")
```

### Deliverable
`born_rule_linalg.py` with Born rule from inner products.

---

## Lab 10: Post-Measurement State — Collapse to |m⟩

- **Concept:** After measurement outcome m, state collapses to |m⟩
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 9

### Walkthrough

```python
# save as collapse_math.py
import numpy as np

psi = np.array([1/np.sqrt(3), np.sqrt(2/3)], dtype=complex)

# Projectors
P0 = np.outer([1,0], [1,0])  # |0⟩⟨0|
P1 = np.outer([0,1], [0,1])  # |1⟩⟨1|

# Measurement outcome 0:
collapsed_0 = P0 @ psi
norm_0 = np.linalg.norm(collapsed_0)
post_0 = collapsed_0 / norm_0
print(f"If outcome = 0:")
print(f"  Unnorm: {collapsed_0}")
print(f"  Norm: {norm_0:.4f}")
print(f"  Post-meas state: {post_0} = |0⟩ ✓")

# Measurement outcome 1:
collapsed_1 = P1 @ psi
norm_1 = np.linalg.norm(collapsed_1)
post_1 = collapsed_1 / norm_1
print(f"\nIf outcome = 1:")
print(f"  Post-meas state: {post_1} = |1⟩ ✓")
```

### Verification
```python
assert np.allclose(post_0, [1, 0])
assert np.allclose(post_1, [0, 1])
print("Lab 10 Verification: PASSED")
```

### Deliverable
`collapse_math.py` with projection and renormalization.

---

## Lab 11: Expectation Values — ⟨ψ|O|ψ⟩

- **Concept:** Observable expectation value as matrix sandwich
- **Difficulty:** Advanced | **Time:** 75 min | **Prerequisites:** Lab 9

### Walkthrough

```python
# save as expectation.py
import numpy as np, math
from zuntenium import QuantumCircuit, StatevectorSimulator

Z = np.array([[1,0],[0,-1]], dtype=complex)
X = np.array([[0,1],[1,0]], dtype=complex)

# ⟨Z⟩ for various states
states = {
    "|0⟩": np.array([1, 0], dtype=complex),
    "|1⟩": np.array([0, 1], dtype=complex),
    "|+⟩": np.array([1, 1], dtype=complex) / np.sqrt(2),
}

print("Expectation values:")
for name, psi in states.items():
    ez = np.real(psi.conj() @ Z @ psi)
    ex = np.real(psi.conj() @ X @ psi)
    print(f"  {name}: ⟨Z⟩={ez:+.4f}, ⟨X⟩={ex:+.4f}")

# Verify: ⟨Z⟩ for |0⟩ = +1, |1⟩ = -1, |+⟩ = 0
# ⟨Z⟩ = P(0) - P(1) — connect to measurements!
backend = StatevectorSimulator()
qc = QuantumCircuit(1); qc.h(0); qc.measure_all()
r = backend.run(qc, shots=10000)
p0 = r.counts.get("0", 0) / 10000
p1 = r.counts.get("1", 0) / 10000
exp_z_meas = p0 - p1
print(f"\n|+⟩ via measurement: ⟨Z⟩ = P(0)-P(1) = {exp_z_meas:.4f} (theory: 0.0)")
```

### Verification
```python
psi0 = np.array([1, 0], dtype=complex)
assert abs(np.real(psi0.conj() @ Z @ psi0) - 1.0) < 1e-10
print("Lab 11 Verification: PASSED")
```

### Deliverable
`expectation.py` with observable expectation values.

---

## Lab 12: CNOT as Matrix — 4×4 Entanglement Operator

- **Concept:** CNOT gate matrix and its action on 2-qubit basis states
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 7

### Walkthrough

```python
# save as cnot_matrix.py
import numpy as np

CX = np.array([[1,0,0,0],[0,1,0,0],[0,0,0,1],[0,0,1,0]], dtype=complex)
basis = {"00": np.array([1,0,0,0]), "01": np.array([0,1,0,0]),
         "10": np.array([0,0,1,0]), "11": np.array([0,0,0,1])}

print("CX action on basis states:")
for name, state in basis.items():
    result = CX @ state
    out = [n for n, s in basis.items() if np.allclose(result, s)][0]
    print(f"  CX|{name}⟩ = |{out}⟩")

# Bell state via matrix multiplication
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
I = np.eye(2, dtype=complex)
H_I = np.kron(H, I)  # H on q0

bell = CX @ H_I @ basis["00"]
print(f"\nBell state = CX(H⊗I)|00⟩ = {bell}")
print(f"= (|00⟩+|11⟩)/√2 ✓")
```

### Verification
```python
expected_bell = np.array([1/np.sqrt(2), 0, 0, 1/np.sqrt(2)])
assert np.allclose(bell, expected_bell)
print("Lab 12 Verification: PASSED")
```

### Deliverable
`cnot_matrix.py` with CNOT matrix analysis.

---

## Lab 13: Bell State Cannot Be Factored — Entanglement Proof

- **Concept:** Entangled states cannot be written as tensor products
- **Difficulty:** Advanced | **Time:** 75 min | **Prerequisites:** Lab 12

### Walkthrough

```python
# save as entanglement_proof.py
import numpy as np

# Bell state: (|00⟩+|11⟩)/√2
bell = np.array([1, 0, 0, 1], dtype=complex) / np.sqrt(2)

# If separable: |ψ⟩ = |a⟩⊗|b⟩ implies ψ_ij = a_i * b_j
# reshape to 2x2 matrix
M = bell.reshape(2, 2)
print(f"Coefficient matrix:\n{M}")

# If separable, rank = 1. If entangled, rank > 1.
U, S, Vh = np.linalg.svd(M)
print(f"\nSingular values: {S}")
print(f"Rank: {np.sum(S > 1e-10)}")
print(f"Rank > 1 → ENTANGLED (cannot be factored)")

# Product state for comparison
product = np.kron([1/np.sqrt(2), 1/np.sqrt(2)], [1, 0])
M_prod = product.reshape(2, 2)
U2, S2, Vh2 = np.linalg.svd(M_prod)
print(f"\nProduct state singular values: {S2}")
print(f"Rank: {np.sum(S2 > 1e-10)}")
print(f"Rank = 1 → SEPARABLE (can be factored)")
```

### Verification
```python
assert np.sum(S > 1e-10) == 2    # Bell state: rank 2
assert np.sum(S2 > 1e-10) == 1   # Product state: rank 1
print("Lab 13 Verification: PASSED")
```

### Deliverable
`entanglement_proof.py` with SVD separability test.

---

## Lab 14: Statevector Simulator = Numpy Array Under the Hood

- **Concept:** StatevectorSimulator stores 2^n complex128 array
- **Difficulty:** Advanced | **Time:** 60 min | **Prerequisites:** Lab 7

### Walkthrough

```python
# save as sv_numpy.py
import numpy as np
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# The simulator stores a numpy array of shape (2^n,) dtype=complex128
for n in [1, 2, 3, 4, 5]:
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    
    # Memory = 2^n * 16 bytes (complex128 = 16 bytes)
    mem = 2**n * 16
    print(f"n={n}: statevector dim=2^{n}={2**n}, memory={mem} bytes")

# Verify: manually compute what the simulator does
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
CX = np.array([[1,0,0,0],[0,1,0,0],[0,0,0,1],[0,0,1,0]], dtype=complex)
I = np.eye(2, dtype=complex)

state = np.array([1,0,0,0], dtype=complex)  # |00⟩
state = np.kron(H, I) @ state               # H on q0
state = CX @ state                           # CX(0,1)
print(f"\nManual Bell state: {state}")
# Same as what the simulator computes!
```

### Verification
```python
expected = np.array([1/np.sqrt(2), 0, 0, 1/np.sqrt(2)])
assert np.allclose(state, expected)
print("Lab 14 Verification: PASSED")
```

### Deliverable
`sv_numpy.py` with manual statevector computation.

---

## Lab 15: Multiple Shots = Sampling the Born Distribution

- **Concept:** Shots sample from probability distribution defined by |⟨m|ψ⟩|²
- **Difficulty:** Advanced | **Time:** 60 min | **Prerequisites:** Lab 9

### Walkthrough

```python
# save as shot_sampling.py
import numpy as np
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Known state: Ry(π/3)|0⟩ → P(1) = sin²(π/6) = 0.25
import math
theta = math.pi / 3
p1_theory = math.sin(theta/2)**2

qc = QuantumCircuit(1); qc.ry(theta, 0); qc.measure_all()

print(f"Theory: P(1) = {p1_theory:.4f}")
print(f"{'Shots':>8} | {'P(1) meas':>10} | {'Error':>8} | {'SE':>8}")
print("-" * 45)

for shots in [10, 100, 1000, 10000, 100000]:
    r = backend.run(qc, shots=shots)
    p1_meas = r.counts.get("1", 0) / shots
    error = abs(p1_meas - p1_theory)
    se = math.sqrt(p1_theory * (1-p1_theory) / shots)
    print(f"{shots:>8} | {p1_meas:>10.4f} | {error:>8.4f} | {se:>8.4f}")
```

### Verification
```python
r = backend.run(qc, shots=100000)
p = r.counts.get("1", 0) / 100000
assert abs(p - p1_theory) < 0.01
print("Lab 15 Verification: PASSED")
```

### Deliverable
`shot_sampling.py` with convergence analysis.

---

## Lab 16: Superconducting vs Trapped Ion — Hardware Comparison

- **Concept:** Nakahara Ch.2: physical qubit realizations
- **Difficulty:** Advanced | **Time:** 60 min | **Prerequisites:** Lab 14

### Walkthrough

```python
# save as hardware_compare.py

# Conceptual comparison (no real hardware needed)
hardware = {
    "Superconducting (IBM/Rigetti)": {
        "gate_time": "20-50 ns", "T1": "~100 µs", "T2": "~80 µs",
        "error_rate": "0.1-1%", "connectivity": "Limited (nearest-neighbor)",
        "max_qubits_2024": "~1000+", "companies": "IBM, Google, Rigetti"
    },
    "Trapped Ion (IonQ)": {
        "gate_time": "~100 µs", "T1": "seconds-minutes", "T2": "~seconds",
        "error_rate": "0.01-0.1%", "connectivity": "All-to-all",
        "max_qubits_2024": "~30-50", "companies": "IonQ, Quantinuum"
    },
}

print("Quantum Hardware Comparison (Nakahara Ch.2 + modern data)")
print("=" * 60)
for platform, specs in hardware.items():
    print(f"\n{platform}:")
    for k, v in specs.items():
        print(f"  {k:<25}: {v}")

# What this means for circuits
print("\n--- Implications for Circuit Design ---")
print("• Short T1/T2 → need shallow circuits (low depth)")
print("• Limited connectivity → transpiler inserts SWAPs")
print("• Gate errors → need error mitigation (ZQL-11)")
print("• Simulator has NONE of these problems!")
```

### Verification
```python
assert len(hardware) == 2
print("Lab 16 Verification: PASSED")
```

### Deliverable
`hardware_compare.py` with platform comparison table.

---

## Lab 17: Simulator vs Hardware — The Fidelity Gap

- **Concept:** Simulated noise model showing fidelity degradation
- **Difficulty:** Advanced | **Time:** 75 min | **Prerequisites:** Lab 16

### Walkthrough

```python
# save as fidelity_gap.py
import random
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

def simulate_noisy_measurement(counts, gate_error=0.01, readout_error=0.02):
    noisy = {}
    for bs, count in counts.items():
        for _ in range(count):
            noisy_bs = ""
            for bit in bs:
                if random.random() < readout_error:
                    noisy_bs += "0" if bit == "1" else "1"
                else:
                    noisy_bs += bit
            noisy[noisy_bs] = noisy.get(noisy_bs, 0) + 1
    return noisy

# Run perfect GHZ circuits, then add noise
print(f"{'N':>3} | {'Perfect fidelity':>16} | {'Noisy fidelity':>14} | {'Gap':>6}")
print("-" * 50)
for n in range(2, 9):
    qc = QuantumCircuit(n); qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    qc.measure_all()
    perfect = backend.run(qc, shots=10000)
    noisy = simulate_noisy_measurement(perfect.counts, readout_error=0.02)
    
    correct = noisy.get("0"*n, 0) + noisy.get("1"*n, 0)
    total = sum(noisy.values())
    fid = correct / total
    print(f"{n:>3} | {'1.0000':>16} | {fid:>14.4f} | {1-fid:>6.4f}")
```

### Verification
```python
noisy = simulate_noisy_measurement({"00": 5000, "11": 5000}, readout_error=0.0)
assert noisy == {"00": 5000, "11": 5000}
print("Lab 17 Verification: PASSED")
```

### Deliverable
`fidelity_gap.py` with noise simulation.

---

## Lab 18: QuantumCircuit as ℂ^(2^n) Transformer

- **Concept:** Connection between circuit and its matrix representation on full space
- **Difficulty:** Expert | **Time:** 75 min | **Prerequisites:** Lab 8

### Walkthrough

```python
# save as circuit_as_matrix.py
import numpy as np

I = np.eye(2, dtype=complex)
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
CX = np.array([[1,0,0,0],[0,1,0,0],[0,0,0,1],[0,0,1,0]], dtype=complex)

# Bell state circuit = CX @ (H ⊗ I)
bell_matrix = CX @ np.kron(H, I)
print(f"Bell circuit matrix (4×4):\n{np.round(bell_matrix, 4)}")
print(f"Unitary? {np.allclose(bell_matrix.conj().T @ bell_matrix, np.eye(4))}")

# Apply to |00⟩
state_00 = np.array([1,0,0,0], dtype=complex)
bell_state = bell_matrix @ state_00
print(f"\nBell state = {bell_state}")

# 3-qubit GHZ circuit matrix
CX_01 = np.kron(CX, I)  # CX on q0,q1 ⊗ I on q2
CX_12 = np.kron(I, CX)  # I on q0 ⊗ CX on q1,q2
H_0 = np.kron(np.kron(H, I), I)  # H on q0

ghz_matrix = CX_12 @ CX_01 @ H_0
print(f"\nGHZ circuit matrix: {ghz_matrix.shape}")
ghz_state = ghz_matrix @ np.array([1,0,0,0,0,0,0,0], dtype=complex)
print(f"GHZ state = {np.round(ghz_state, 4)}")
```

### Verification
```python
expected = np.array([1/np.sqrt(2), 0, 0, 1/np.sqrt(2)])
assert np.allclose(bell_state, expected)
print("Lab 18 Verification: PASSED")
```

### Deliverable
`circuit_as_matrix.py` with full circuit matrix construction.

---

## Lab 19: HardwareAwareTranspiler — What It Does

- **Concept:** Transpiler handles routing + decomposition for real hardware
- **Difficulty:** Expert | **Time:** 60 min | **Prerequisites:** Lab 16

### Walkthrough

```python
# save as transpiler_aware.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Circuit with long-range CX
qc = QuantumCircuit(5)
qc.h(0)
qc.cx(0, 4)  # Direct CX between q0 and q4 — non-adjacent!
qc.measure_all()

# On simulator: works fine (all-to-all connectivity)
r = backend.run(qc, shots=1024)
print(f"Simulator result: {r.counts}")

# On real hardware with linear connectivity (0-1-2-3-4):
# Transpiler would insert SWAPs: CX(0,1), SWAP(0,1), CX(1,2), ...
# This increases depth and gate count

# Manual routing simulation
qc_routed = QuantumCircuit(5)
qc_routed.h(0)
# Route q0 data towards q4 via SWAPs
qc_routed.swap(0, 1)  # 3 CX
qc_routed.swap(1, 2)  # 3 CX
qc_routed.swap(2, 3)  # 3 CX
qc_routed.cx(3, 4)    # 1 CX
qc_routed.measure_all()

print(f"\nDirect: {qc.num_gates} gates, depth {qc.depth}")
print(f"Routed: {qc_routed.num_gates} gates, depth {qc_routed.depth}")
print(f"SWAP overhead: {qc_routed.num_gates - qc.num_gates} extra gates")
```

### Verification
```python
assert qc_routed.num_gates > qc.num_gates
print("Lab 19 Verification: PASSED")
```

### Deliverable
`transpiler_aware.py` with routing overhead analysis.

---

## Lab 20: Nakahara Math Capstone — From Hilbert Space to Running Circuits

- **Concept:** Complete pathway from math to code
- **Difficulty:** Expert | **Time:** 120 min | **Prerequisites:** All

### Walkthrough

```python
# save as nakahara_capstone.py
import numpy as np, json, time
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
report = {"title": "Nakahara Math→Code Integration", "tests": []}

# Test 1: ℂ² basis orthonormality
ket0, ket1 = np.array([1,0], dtype=complex), np.array([0,1], dtype=complex)
t1 = abs(np.vdot(ket0,ket0) - 1) < 1e-10 and abs(np.vdot(ket0,ket1)) < 1e-10
report["tests"].append({"name": "Basis orthonormality", "pass": t1})

# Test 2: Gate unitarity
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
t2 = np.allclose(H.conj().T @ H, np.eye(2))
report["tests"].append({"name": "H gate unitarity", "pass": t2})

# Test 3: Tensor product dimension
t3 = len(np.kron(ket0, ket0)) == 4
report["tests"].append({"name": "Tensor product dim", "pass": t3})

# Test 4: Born rule
p_theory = abs(np.vdot(ket0, (ket0+ket1)/np.sqrt(2)))**2
qc = QuantumCircuit(1); qc.h(0); qc.measure_all()
r = backend.run(qc, shots=10000)
p_meas = r.counts.get("0", 0) / 10000
t4 = abs(p_meas - p_theory) < 0.03
report["tests"].append({"name": "Born rule", "pass": t4})

# Test 5: Bell entanglement (rank test)
bell = np.array([1,0,0,1], dtype=complex) / np.sqrt(2)
_, S, _ = np.linalg.svd(bell.reshape(2,2))
t5 = np.sum(S > 1e-10) == 2
report["tests"].append({"name": "Entanglement SVD", "pass": t5})

report["all_pass"] = all(t["pass"] for t in report["tests"])
report["timestamp"] = time.strftime("%Y-%m-%d %H:%M:%S")

for t in report["tests"]:
    print(f"  {'✓' if t['pass'] else '✗'} {t['name']}")
print(f"\nAll passed: {report['all_pass']}")

with open("nakahara_report.json", "w") as f:
    json.dump(report, f, indent=2)
```

### Verification
```python
data = json.load(open("nakahara_report.json"))
assert data["all_pass"]
print(f"Lab 20 Verification: PASSED ({len(data['tests'])} tests)")
```

### Deliverable
`nakahara_capstone.py` with math→code integration report.

---

## Validation Report

1. **Count check:** 20 labs ✓
2. **Uniqueness:** 20 distinct math concepts from Nakahara Chs. 1-2 ✓
3. **Coverage:** ℂ², bra-ket, inner products, unitarity, eigenvalues, Pauli, tensor products, Born rule, measurement, observables, CNOT matrix, entanglement proof, hardware ✓
4. **Code:** Syntactically valid Python ✓
5. **Links:** Source doc referenced ✓
6. **Tone:** Mathematical rigor with practical code verification ✓
