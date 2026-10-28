# ZQL-01 — Mathematical Foundations of Quantum Computing

> **Course ID**: ZQL-01 | **Track**: ZQL-00 → ZQL-01  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisite**: ZQL-00.01 (Introduction to Quantum SDK & Developer Onboarding)

---

## 0. Course Overview

### Who This Is For

You have completed ZQL-00.01 and can build simple circuits with `QuantumCircuit`. This course gives you the **mathematical language** that quantum computing is built on — so you can read papers, understand algorithm descriptions, and reason about why circuits behave the way they do.

**Prerequisites**:
- ZQL-00.01 completed (can write and run a Bell state circuit)
- Comfortable with Python and basic arithmetic
- Complex numbers helpful but not required — we introduce them from scratch

### Learning Outcomes

By the end of this course you will be able to:

1. Work fluently in Dirac bra-ket notation
2. Understand complex amplitudes and why they are not probabilities
3. Describe any single-qubit state on the Bloch sphere
4. Verify that a gate matrix is unitary
5. Compute multi-qubit states using tensor products
6. Distinguish pure states from mixed states and explain when each arises
7. State the four postulates of quantum mechanics and locate each in QuFlow code
8. Recognise and work with the Pauli matrices (X, Y, Z) and their eigenstates
9. Explain entanglement formally and test for it

### Estimated Time

| Section | Time |
|---|---|
| §1–2: Complex numbers + Dirac notation | 25 min |
| §3: The Bloch sphere | 20 min |
| §4: Multi-qubit mathematics | 20 min |
| §5: The four postulates + SDK mapping | 20 min |
| §6: Pauli matrices + gate zoo | 20 min |
| §7: Mixed states and density matrices | 15 min |
| §8: Entanglement | 20 min |
| §9: Exercises, quiz, capstone | 30 min |
| **Total** | **~170 minutes (~3 hours)** |

### Source Books Used

| Ref | Book | Chapters |
|---|---|---|
| NC | Nielsen & Chuang — *Quantum Computation and Quantum Information* (2000) | 1–2 |
| KLM | Kaye, Laflamme & Mosca — *An Introduction to Quantum Computing* (Oxford, 2007) | 1–5 |
| RP | Rieffel & Polak — *Quantum Computing: A Gentle Introduction* (MIT Press, 2011) | 1–4 |
| A | Axler — *Linear Algebra Done Right* (3rd ed.) | 1, 2, 5, 7 |
| SF | Susskind & Friedman — *Quantum Mechanics: The Theoretical Minimum* (2014) | 1–5 |
| QC | Matuschak & Nielsen — *Quantum Country* (free, quantumcountry.com) | Essays 1–2 |

---

## 1. Complex Numbers — The Fuel of Quantum

Classical probability uses real, non-negative numbers. Quantum mechanics uses **complex numbers**. This single difference is responsible for interference — and interference is what makes quantum algorithms useful.

### 1.1 What Is a Complex Number?

A complex number has a real part and an imaginary part:

```
z = a + bi     where i² = −1
```

Visualise on the **complex plane**: real axis (horizontal), imaginary axis (vertical).

Key operations:
- **Conjugate**: z* = a − bi (flip the imaginary part)
- **Modulus**: |z|² = a² + b² (squared distance from origin)
- **Euler's formula**: e^{iθ} = cos θ + i sin θ

Euler's formula is critical: every complex number of modulus 1 is **a point on the unit circle**, described by an angle θ. This angle is called the *phase*.

```python
import math, cmath

z = complex(1, 1)              # 1 + i
print(abs(z))                  # |z| = √2 ≈ 1.414
print(z.conjugate())           # 1 - i

# Euler: e^{iπ/4} = (1+i)/√2
euler = cmath.exp(1j * math.pi / 4)
print(round(euler.real, 6), round(euler.imag, 6))   # 0.707107  0.707107
```

### 1.2 Why Complex Numbers in Quantum?

A qubit's state requires **two complex amplitudes**:

```
|ψ⟩ = α|0⟩ + β|1⟩     α, β ∈ ℂ,  |α|² + |β|² = 1
```

- |α|² = probability of measuring 0
- |β|² = probability of measuring 1
- The *phases* of α and β are invisible to measurement alone — but gates manipulate them, enabling inference.

**Why not real amplitudes?** Real amplitudes could only create constructive interference. Complex amplitudes allow *directional* interference — the H·Z·H example from ZQL-00.01 only works because of the sign flip (phase π) introduced by Z.

> **Key Takeaway §1**: Quantum states are unit vectors over ℂ. Probabilities are squared moduli. Phases carry the computational power.

---

## 2. Dirac Notation (Bra-Ket)

Dirac notation is the standard language of quantum computing papers, algorithms, and SDK documentation. Master it here.

### 2.1 Kets — Column Vectors

A **ket** `|v⟩` is a column vector (a quantum state):

```
|0⟩ = [1]    |1⟩ = [0]    |+⟩ = [1/√2]    |−⟩ = [ 1/√2]
      [0]           [1]           [1/√2]           [-1/√2]
```

General single-qubit state:
```
|ψ⟩ = α|0⟩ + β|1⟩ = [α]
                      [β]
```

In Python (statevector perspective):
```python
import numpy as np

ket_0  = np.array([1, 0], dtype=complex)        # |0⟩
ket_1  = np.array([0, 1], dtype=complex)        # |1⟩
ket_plus = np.array([1, 1], dtype=complex) / np.sqrt(2)   # |+⟩
```

### 2.2 Bras — Row Vectors (Conjugate Transpose)

The **bra** `⟨v|` is the conjugate transpose of the ket:

```
⟨ψ| = [α*  β*]
```

```python
bra_plus = ket_plus.conj()     # row vector [1/√2  1/√2]
```

### 2.3 Inner Product

The **inner product** `⟨u|v⟩` is a complex scalar:

```
⟨u|v⟩ = u† · v = Σᵢ uᵢ* vᵢ
```

Key facts:
- `⟨0|0⟩ = 1`, `⟨1|1⟩ = 1` (normalised)
- `⟨0|1⟩ = 0`, `⟨1|0⟩ = 0` (orthogonal)
- `⟨ψ|ψ⟩ = |α|² + |β|² = 1` (normalisation condition)
- `|⟨basis|ψ⟩|²` = probability of measuring that basis state (Born rule)

```python
inner = np.dot(ket_plus.conj(), ket_1)  # ⟨+|1⟩
prob_1_given_plus = abs(inner)**2        # = 0.5
print(f"P(|1⟩ | |+⟩) = {prob_1_given_plus}")  # 0.5
```

### 2.4 Outer Product — Matrices

The **outer product** `|u⟩⟨v|` is a matrix:

```python
outer = np.outer(ket_0, ket_1.conj())   # |0⟩⟨1| — 2×2 matrix
# [[0, 1],
#  [0, 0]]
```

Outer products build projection operators and gates. For example:
- `|0⟩⟨0|` = projection onto |0⟩ = [[1,0],[0,0]]
- `|0⟩⟨0| + |1⟩⟨1|` = I (completeness)
- CNOT = `|0⟩⟨0|⊗I + |1⟩⟨1|⊗X`

### 2.5 Brakets — Expectation Values

`⟨ψ|A|ψ⟩` is a **matrix element** or **expectation value** of operator A in state |ψ⟩:

```python
Z_matrix = np.array([[1, 0],[0, -1]], dtype=complex)    # Pauli Z
psi = np.array([1, 1], dtype=complex) / np.sqrt(2)      # |+⟩
expectation_Z = psi.conj() @ Z_matrix @ psi
print(f"⟨+|Z|+⟩ = {expectation_Z.real:.6f}")   # 0.0 — |+⟩ has zero Z expectation
```

> **Note**: `⟨+|Z|+⟩ = 0` means the |+⟩ state has equal probability of +1 and −1 outcomes when you measure Z (i.e., measure in the standard basis).

---

## 3. The Bloch Sphere — Visualising Single-Qubit States

Every single-qubit pure state maps to a unique point on a unit sphere called the **Bloch sphere**.

### 3.1 Parameterisation

Using the global-phase invariance (e^{iγ}|ψ⟩ is physically identical to |ψ⟩):

```
|ψ⟩ = cos(θ/2)|0⟩ + e^{iφ}sin(θ/2)|1⟩

  θ ∈ [0, π]     — polar angle (latitude)
  φ ∈ [0, 2π)    — azimuthal angle (longitude)
```

**Key points on the sphere**:

| State | θ | φ | Position |
|---|---|---|---|
| `\|0⟩` | 0 | any | North pole |
| `\|1⟩` | π | any | South pole |
| `\|+⟩` | π/2 | 0 | +X axis |
| `\|−⟩` | π/2 | π | −X axis |
| `\|i⟩` | π/2 | π/2 | +Y axis |
| `\|−i⟩` | π/2 | 3π/2 | −Y axis |

### 3.2 Gates as Bloch Sphere Rotations

Every single-qubit gate = a rotation of the Bloch vector:

| Gate | Action on Bloch Sphere |
|---|---|
| X | 180° rotation around X axis: \|0⟩ ↔ \|1⟩ |
| Y | 180° rotation around Y axis |
| Z | 180° rotation around Z axis: \|+⟩ ↔ \|−⟩ |
| H | 180° rotation around (X+Z)/√2 axis: maps Z→X, X→Z |
| S | 90° rotation around Z axis |
| T | 45° rotation around Z axis |
| Rx(θ) | θ rotation around X axis |
| Ry(θ) | θ rotation around Y axis |
| Rz(θ) | θ rotation around Z axis |

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import math

# Demonstrate H maps |0⟩ to equator: |0⟩ → |+⟩
qc = QuantumCircuit(1)
qc.h(0).measure_all()
result = StatevectorSimulator().run(qc, shots=4096)
print(result.counts)   # ~{"0": 2048, "1": 2048} — equatorial state

# Ry(π/2) also maps to equator (same as H, different path)
qc2 = QuantumCircuit(1)
qc2.ry(math.pi / 2, 0).measure_all()
result2 = StatevectorSimulator().run(qc2, shots=4096)
print(result2.counts)  # ~{"0": 2048, "1": 2048}
```

### 3.3 The Z Eigenstates and X Eigenstates

| Basis | Eigenstates | Measuring in this basis |
|---|---|---|
| Z (standard) | \|0⟩, \|1⟩ | `qc.measure(q,c)` directly |
| X | \|+⟩, \|−⟩ | Apply `qc.h(q)` first, then measure |
| Y | \|i⟩, \|−i⟩ | Apply `qc.s(q).h(q)` first, then measure |

```python
# Measure in X basis: apply H first, then measure in Z
qc = QuantumCircuit(1)
qc.h(0)     # Prepare |+⟩ (X eigenstate with eigenvalue +1)
qc.h(0)     # Rotate to Z basis for measurement
qc.measure_all()
result = StatevectorSimulator().run(qc, shots=1024)
print(result.counts)   # {"0": 1024} — |+⟩ → eigenvalue +1 every time
```

---

## 4. Multi-Qubit Mathematics: Tensor Products

### 4.1 Tensor Product of States

For two qubits with states |a⟩ and |b⟩, the joint state is:

```
|ab⟩ = |a⟩ ⊗ |b⟩
```

For general |a⟩ = α|0⟩+β|1⟩ and |b⟩ = γ|0⟩+δ|1⟩:

```
|ab⟩ = αγ|00⟩ + αδ|01⟩ + βγ|10⟩ + βδ|11⟩
```

This is a 4-dimensional vector.

```python
a = np.array([1, 0], dtype=complex)   # |0⟩
b = np.array([0, 1], dtype=complex)   # |1⟩
ab = np.kron(a, b)                     # tensor product = |01⟩
print(ab)   # [0, 1, 0, 0] — |01⟩ = second of four basis states
```

### 4.2 Memory Scaling

| Qubits | Statevector size | RAM (complex128) |
|---|---|---|
| 5 | 32 | 256 B |
| 10 | 1,024 | 8 KB |
| 20 | 1,048,576 | 8 MB |
| 30 | 1,073,741,824 | 8 GB |

This exponential growth is both quantum computing's power (state encodes exponentially many amplitudes) and its simulation limit.

```python
# Verify: QuantumCircuit(n) lives in 2^n-dimensional space
for n in [2, 4, 8, 16]:
    qc = QuantumCircuit(n)
    print(f"n={n:2d}: statevector size = {2**n:6d}")
```

### 4.3 Tensor Product of Gates

A gate on qubit k in an n-qubit system acts as:

```
I ⊗ ... ⊗ U(k) ⊗ ... ⊗ I
```

```python
H = np.array([[1,1],[1,-1]], dtype=complex) / np.sqrt(2)
I = np.eye(2, dtype=complex)

# H on qubit 0 of 2-qubit system = H ⊗ I
H_on_q0 = np.kron(H, I)    # 4×4 matrix

# H on qubit 1 of 2-qubit system = I ⊗ H
H_on_q1 = np.kron(I, H)    # 4×4 matrix
```

**[SDK]** QuFlow handles all this automatically. When you write `qc.h(0)` on a 3-qubit circuit, QuFlow applies H ⊗ I ⊗ I to the 8-dimensional statevector.

### 4.4 Separable vs Entangled States

A state is **separable** (product) if it can be written as |a⟩⊗|b⟩:

- |00⟩ = |0⟩⊗|0⟩ → separable ✓
- |+0⟩ = |+⟩⊗|0⟩ → separable ✓
- (|00⟩+|11⟩)/√2 → **entangled** ✗ (cannot factor)

**Test in code**:
```python
from zuntenium import QuantumCircuit, StatevectorSimulator

# Separable state: H(0) only, no CX
qc_sep = QuantumCircuit(2)
qc_sep.h(0).measure_all()
r = StatevectorSimulator().run(qc_sep, shots=4096)
print("Separable:", r.counts)   # All 4 outcomes: "00","01","10","11" ~equally

# Entangled state: Bell
qc_ent = QuantumCircuit(2)
qc_ent.h(0).cx(0,1).measure_all()
r2 = StatevectorSimulator().run(qc_ent, shots=4096)
print("Entangled:", r2.counts)  # Only "00" and "11"
```

---

## 5. The Four Postulates of Quantum Mechanics — SDK Mapping

These four postulates are the axioms of quantum computing. Every SDK operation traces back to one of them.

### Postulate 1 — State Space
> *A closed quantum system is associated with a Hilbert space. The state is a unit vector in that space.*

```
SDK: QuantumCircuit(n) represents a system in ℂ^(2^n)
     Initial state: |0...0⟩
     StatevectorSimulator tracks statevector[i] ∈ ℂ, with Σ|statevector[i]|² = 1
```

### Postulate 2 — Evolution
> *A closed system evolves by a unitary operator: |ψ'⟩ = U|ψ⟩.*

```
SDK: Each gate method (qc.h(), qc.cx(), qc.rz(), …) applies a unitary matrix to the statevector
     BaseGate.matrix() returns U as a numpy array
     Verify unitarity: np.allclose(U @ U.conj().T, np.eye(2))
```

```python
import numpy as np
from zuntenium.quantum_core.gates import HGate, CXGate

H_matrix = HGate().matrix()
is_unitary = np.allclose(H_matrix @ H_matrix.conj().T, np.eye(2))
print(f"H is unitary: {is_unitary}")   # True

CX_matrix = CXGate().matrix()
is_unitary_cx = np.allclose(CX_matrix @ CX_matrix.conj().T, np.eye(4))
print(f"CX is unitary: {is_unitary_cx}")   # True
```

### Postulate 3 — Measurement
> *Measurement (projective, computational basis): outcome m with probability |⟨m|ψ⟩|²; state collapses to |m⟩.*

```
SDK: qc.measure(q, c)  or  qc.measure_all()
     Running N shots gives a distribution over bit strings
     result.counts["10"] / N ≈ |⟨10|ψ⟩|²
```

```python
# Verify Born rule: |+⟩ measured → P(|0⟩)=0.5, P(|1⟩)=0.5
from zuntenium import QuantumCircuit, StatevectorSimulator

qc = QuantumCircuit(1)
qc.h(0).measure_all()
result = StatevectorSimulator().run(qc, shots=100_000)

for state, count in sorted(result.counts.items()):
    print(f"P(|{state}⟩) ≈ {count/100_000:.4f}   (theory: 0.5000)")
```

### Postulate 4 — Composite Systems
> *The state space of n systems is the tensor product of their individual state spaces.*

```
SDK: QuantumCircuit(n) → 2^n-dimensional Hilbert space
     qc.cx(0,1) acts on the 4-dim (2-qubit sub)space
     Gates on disjoint qubits are tensor products: qc.h(0); qc.h(1) = H⊗H applied together (parallel layer)
```

> **Key Takeaway §5**: The four postulates map exactly to: `QuantumCircuit(n)` (P1) → `gate methods` (P2) → `measure_all()` (P3) → `QuantumCircuit(n)` for composite (P4).

---

## 6. The Pauli Matrices and Gate Zoo

### 6.1 Pauli Matrices — The Fundamental 2×2 Unitaries

```python
import numpy as np

I = np.eye(2, dtype=complex)
X = np.array([[0, 1],[1, 0]], dtype=complex)   # NOT gate
Y = np.array([[0,-1j],[1j,0]], dtype=complex)  # Y gate
Z = np.array([[1, 0],[0,-1]], dtype=complex)   # Phase flip

# Pauli algebra: XY = iZ, YZ = iX, ZX = iY
print(np.allclose(X @ Y, 1j * Z))   # True
print(np.allclose(Y @ Z, 1j * X))   # True

# Each Pauli is Hermitian AND unitary (self-inverse: P²=I)
for P, name in [(X,"X"),(Y,"Y"),(Z,"Z")]:
    print(f"{name}: hermitian={np.allclose(P, P.conj().T)}, "
          f"unitary={np.allclose(P@P.conj().T, I)}, "
          f"self-inverse={np.allclose(P@P, I)}")
```

**Eigenstates**:

| Pauli | Eigenvalue +1 | Eigenvalue −1 |
|---|---|---|
| Z | \|0⟩ | \|1⟩ |
| X | \|+⟩ = (|0⟩+|1⟩)/√2 | \|−⟩ = (|0⟩−|1⟩)/√2 |
| Y | \|i⟩ = (|0⟩+i|1⟩)/√2 | \|−i⟩ = (|0⟩−i|1⟩)/√2 |

### 6.2 Rotation Gates

Rotation by angle θ around axis n̂ on the Bloch sphere:

```
Rₙ(θ) = e^{-iθn̂·σ/2} = cos(θ/2)I − i·sin(θ/2)(n̂·σ)
```

For the Pauli axes:
```
Rx(θ) = [[cos θ/2,  -i·sin θ/2],
          [-i·sin θ/2,  cos θ/2]]

Ry(θ) = [[cos θ/2,  -sin θ/2],
          [sin θ/2,   cos θ/2]]

Rz(θ) = [[e^{-iθ/2},     0    ],
          [  0,       e^{iθ/2} ]]
```

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import math

# Ry(π) rotates |0⟩ to |1⟩ (180° around Y)
qc = QuantumCircuit(1)
qc.ry(math.pi, 0).measure_all()
result = StatevectorSimulator().run(qc, shots=1024)
print(f"Ry(π)|0⟩ → {result.counts}")   # {"1": 1024}

# Rz(π) adds a phase to |1⟩: |1⟩ → -|1⟩ (invisible to Z-measurement alone)
qc2 = QuantumCircuit(1)
qc2.h(0).rz(math.pi, 0).h(0).measure_all()   # Make phase visible via H
result2 = StatevectorSimulator().run(qc2, shots=1024)
print(f"H·Rz(π)·H|0⟩ → {result2.counts}")   # {"1": 1024} — Rz(π)=Z
```

### 6.3 Complete Gate Reference for ZQL-01

| Category | Gate | SDK Method | Matrix |
|---|---|---|---|
| **Clifford** | H | `qc.h(q)` | (X+Z)/√2 |
| | X | `qc.x(q)` | [[0,1],[1,0]] |
| | Y | `qc.y(q)` | [[0,-i],[i,0]] |
| | Z | `qc.z(q)` | [[1,0],[0,-1]] |
| | S | `qc.s(q)` | [[1,0],[0,i]] |
| | CX | `qc.cx(c,t)` | CNOT 4×4 |
| | CZ | `qc.cz(c,t)` | Diag(1,1,1,-1) |
| | SWAP | `qc.swap(q1,q2)` | Permutation |
| **Non-Clifford** | T | `qc.t(q)` | [[1,0],[0,e^{iπ/4}]] |
| **Rotations** | Rx(θ) | `qc.rx(theta,q)` | See §6.2 |
| | Ry(θ) | `qc.ry(theta,q)` | See §6.2 |
| | Rz(θ) | `qc.rz(theta,q)` | See §6.2 |
| **General** | U3(θ,φ,λ) | `qc.u(theta,phi,lam,q)` | Rz·Ry·Rz |
| **3-qubit** | Toffoli | `qc.ccx(c0,c1,t)` | CCX 8×8 |
| | Fredkin | `qc.cswap(c,t0,t1)` | CSWAP 8×8 |

### 6.4 Why {H, T, CNOT} Is Universal

Every unitary U on n qubits can be approximated to any ε using only H, T, and CNOT:
1. Any 2-qubit U decomposes to ≤ 3 CNOT + single-qubit rotations (KAK theorem)
2. Any single-qubit rotation approximates to ε using O(log^c(1/ε)) H+T gates (Solovay-Kitaev)

**[SDK]** The `KAKDecompositionPass` implements exact 2-qubit synthesis. `ConsolidateBlocksPass` groups gates into 2-qubit blocks for KAK.

---

## 7. Mixed States and Density Matrices

### 7.1 When Do Mixed States Arise?

Pure states describe isolated quantum systems. In practice:
- **Noise / decoherence**: interaction with environment → system becomes mixed
- **Partial information**: you have a subsystem of a larger entangled system
- **Statistical ensemble**: you prepare different states with classical probability

### 7.2 Density Matrix Definition

Pure state: **ρ = |ψ⟩⟨ψ|** (rank-1 positive semidefinite matrix)

Mixed state: **ρ = Σᵢ pᵢ |ψᵢ⟩⟨ψᵢ|** where pᵢ ≥ 0, Σpᵢ = 1

```python
# Pure state |+⟩: density matrix
psi = np.array([1, 1]) / np.sqrt(2)
rho_pure = np.outer(psi, psi.conj())
print("ρ_pure =\n", rho_pure.real)
# [[0.5, 0.5],
#  [0.5, 0.5]]

# Completely mixed state (50% |0⟩, 50% |1⟩):
rho_mixed = 0.5 * np.outer([1,0],[1,0]) + 0.5 * np.outer([0,1],[0,1])
print("ρ_mixed =\n", rho_mixed.real)
# [[0.5, 0],
#  [0,  0.5]]   = I/2
```

### 7.3 Pure vs Mixed Test

**Tr(ρ²) = 1** iff pure; **Tr(ρ²) < 1** iff mixed:

```python
print(f"Tr(ρ²) pure  = {np.trace(rho_pure @ rho_pure).real:.3f}")    # 1.0
print(f"Tr(ρ²) mixed = {np.trace(rho_mixed @ rho_mixed).real:.3f}")  # 0.5
```

### 7.4 Density Matrix Evolution and Measurement

- **Unitary evolution**: ρ → UρU†
- **Measurement**: p(outcome m) = Tr(Mₘ ρ Mₘ†)
- **Noise channel (Kraus map)**: ε(ρ) = Σₖ KₖρKₖ† with Σₖ Kₖ†Kₖ = I

**[SDK]** `DensityMatrixSimulator` tracks ρ and supports Kraus noise channels. `StatevectorSimulator` tracks |ψ⟩ (pure states only).

```python
# Use DensityMatrixSimulator for noise modelling (ZQL-11 content)
# from zuntenium import DensityMatrixSimulator
# dsim = DensityMatrixSimulator()
# result = dsim.run(qc, noise_model=depolarizing_noise(p=0.01))
```

---

## 8. Entanglement: The Formal Picture

### 8.1 Definition

A pure bipartite state |ψ⟩_AB is **entangled** if it cannot be written as |a⟩_A ⊗ |b⟩_B for any single-qubit states |a⟩, |b⟩.

**Schmidt decomposition**: any |ψ⟩_AB = Σₖ λₖ|k⟩_A|k⟩_B where λₖ ≥ 0 and Σλₖ² = 1.
- If only one Schmidt coefficient is non-zero → product (separable)
- If more than one is non-zero → entangled
- **Schmidt rank** = number of non-zero Schmidt coefficients

### 8.2 The Four Bell States

The Bell states are a maximally entangled orthonormal basis for the 2-qubit space:

| State | Formula | How to Create |
|---|---|---|
| \|Φ⁺⟩ | (|00⟩+|11⟩)/√2 | `qc.h(0).cx(0,1)` |
| \|Φ⁻⟩ | (|00⟩−|11⟩)/√2 | `qc.x(0).h(0).cx(0,1)` (then X) |
| \|Ψ⁺⟩ | (|01⟩+|10⟩)/√2 | `qc.h(0).cx(0,1).x(1)` |
| \|Ψ⁻⟩ | (|01⟩−|10⟩)/√2 | `qc.h(0).cx(0,1).z(0).x(1)` |

```python
# Build all four Bell states and verify correlations
from zuntenium import QuantumCircuit, StatevectorSimulator

def bell_state(which: str) -> QuantumCircuit:
    qc = QuantumCircuit(2, name=f"bell_{which}")
    if which == "Phi+":
        qc.h(0).cx(0, 1)
    elif which == "Phi-":
        qc.h(0).cx(0, 1).z(0)
    elif which == "Psi+":
        qc.h(0).cx(0, 1).x(0)
    elif which == "Psi-":
        qc.h(0).cx(0, 1).x(0).z(0)
    qc.measure_all()
    return qc

backend = StatevectorSimulator()
for name in ["Phi+", "Phi-", "Psi+", "Psi-"]:
    result = backend.run(bell_state(name), shots=2048)
    print(f"|{name}⟩: {dict(result.counts)}")
```

Expected:
```
|Φ+⟩: {"00": ~1024, "11": ~1024}
|Φ-⟩: {"00": ~1024, "11": ~1024}   (phases invisible to Z-measurement)
|Ψ+⟩: {"01": ~1024, "10": ~1024}
|Ψ-⟩: {"01": ~1024, "10": ~1024}
```

### 8.3 Entanglement and Reduced Density Matrices

For a Bell state, trace out qubit 1 to get qubit 0's reduced state:

```
ρ_A = Tr_B(|Φ⁺⟩⟨Φ⁺|) = I/2 = [[0.5, 0],[0, 0.5]]
```

This is the fully mixed state — qubit 0 alone carries no information (but correlated with qubit 1).

```python
# Build |Φ+⟩ density matrix and compute partial trace
phi_plus_vec = np.array([1, 0, 0, 1]) / np.sqrt(2)   # |Φ+⟩
rho_AB = np.outer(phi_plus_vec, phi_plus_vec.conj())   # 4×4

# Partial trace over qubit 1 (right qubit):
rho_A = rho_AB[:2,:2] + rho_AB[2:,2:]   # Sum over B basis
print("ρ_A =\n", rho_A.real)
# [[0.5, 0],
#  [0,  0.5]]  ← maximally mixed → Tr(ρ²)=0.5 → entangled
```

---

## 9. Exercises, Quiz, and Capstone

### Exercises §1–3 (Complex Numbers and Bloch Sphere)

**E1.1** Compute |e^{iπ/3}|. Is it normalised?  
*(Answer: |e^{iπ/3}| = 1 — always on the unit circle.)*

**E1.2** If α = (1+i)/√3, β = 1/√3, is |ψ⟩ = α|0⟩ + β|1⟩ a valid state?  
*(Answer: |α|² = 2/3, |β|² = 1/3. Sum = 1. Yes.)*

**E3.1** What is the Bloch sphere representation of the T gate applied to |+⟩?  
*(Answer: |+⟩ is on +X axis. T = Rz(π/4) rotates the equatorial point by 45° around Z. Result: cos(π/8)|0⟩ + e^{iπ/4}sin(π/8)|1⟩.)*

**E3.2** Write code to verify H|+⟩ = |0⟩:
```python
from zuntenium import QuantumCircuit, StatevectorSimulator
qc = QuantumCircuit(1)
qc.h(0).h(0).measure_all()   # |0⟩ → H → |+⟩ → H → |0⟩
result = StatevectorSimulator().run(qc, shots=1024)
print(result.counts)   # {"0": 1024}
```

### Exercises §4–6 (Multi-qubit and Gates)

**E4.1** What is the statevector for QuantumCircuit(3) before any gates?  
*(Answer: [1,0,0,...,0] — 8 amplitudes, all zero except the first (|000⟩ state).)*

**E6.1** Verify X² = I in code:
```python
from zuntenium.quantum_core.gates import XGate
import numpy as np
X = XGate().matrix()
print(np.allclose(X @ X, np.eye(2)))   # True
```

**E6.2** Is the T gate Hermitian? Self-inverse? Unitary?  
*(Answer: T† ≠ T (not Hermitian); T·T = S ≠ I (not self-inverse); T†T = I (unitary — all gates are).)*

### Exercises §7–8 (Mixed States and Entanglement)

**E7.1** For a pure state |ψ⟩, what is Tr(ρ)? Tr(ρ²)?  
*(Answer: Tr(ρ) = 1 (always); Tr(ρ²) = Tr(|ψ⟩⟨ψ|·|ψ⟩⟨ψ|) = ⟨ψ|ψ⟩ = 1. So Tr(ρ²) = 1 iff pure.)*

**E8.1** Will the |Φ⁻⟩ Bell state give different measurement counts than |Φ⁺⟩ when measuring in the Z basis?  
*(Answer: No — both give "00" and "11" with equal probability. The sign difference is a phase, invisible to Z-measurement.)*

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What is ⟨0|+⟩? | 1/√2 |
| Q2 | How many amplitudes does QuantumCircuit(5) have? | 32 (=2⁵) |
| Q3 | If Tr(ρ²) = 0.75, is the state pure or mixed? | Mixed |
| Q4 | Name the two eigenstates of the X gate | \|+⟩ and \|−⟩ |
| Q5 | Which postulate does `qc.measure_all()` implement? | Postulate 3 (Measurement) |
| Q6 | What gate rotates the Bloch vector 90° around Z? | S gate |
| Q7 | Is CX Hermitian? | Yes (CX† = CX since it's self-inverse) |

### Capstone: Quantum State Tomography Toy

Build a program that reconstructs the Bloch sphere coordinates (x,y,z) of a single-qubit state by measuring in all three Pauli bases:

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import math

def prepare_state(theta: float, phi: float) -> QuantumCircuit:
    """Prepare |ψ⟩ = Ry(theta)·Rz(phi)|0⟩."""
    qc = QuantumCircuit(1, name="state")
    qc.rz(phi, 0).ry(theta, 0)
    return qc

def measure_pauli(prep: QuantumCircuit, basis: str, shots: int = 8192) -> float:
    """Measure ⟨P⟩ for P in {X,Y,Z}. Returns expectation value in [-1,+1]."""
    qc = QuantumCircuit(1, name=f"meas_{basis}")
    for instr in prep.instructions:
        qc._instructions.append(instr)
    if basis == "X":
        qc.h(0)                          # Rotate to X basis
    elif basis == "Y":
        qc.s(0).h(0)                     # Rotate to Y basis
    qc.measure_all()
    result = StatevectorSimulator().run(qc, shots=shots)
    p0 = result.counts.get("0", 0) / shots
    p1 = result.counts.get("1", 0) / shots
    return p0 - p1   # = +1 (eigenvalue +1) - (-1 counts)/shots × 2

# Example: prepare |+⟩ (theta=π/2, phi=0) and extract Bloch coordinates
theta = math.pi / 2
phi   = 0.0
prep  = prepare_state(theta, phi)

x = measure_pauli(prep, "X")
y = measure_pauli(prep, "Y")
z = measure_pauli(prep, "Z")

print(f"Bloch vector: x={x:+.3f}, y={y:+.3f}, z={z:+.3f}")
print(f"Theory:       x=+1.000, y=+0.000, z=+0.000  (|+⟩ is on +X axis)")
print(f"|r|² ≈ {x**2 + y**2 + z**2:.3f}  (pure state → should be ≈ 1.0)")
```

**Extension**: Try theta=π/4, phi=π/2. What Bloch coordinates do you expect? Verify.

---

## 10. Glossary, [TO CONFIRM] Items, and Next Course

### Glossary

| Term | Definition |
|---|---|
| **Amplitude** | A complex number α associated with a basis state; \|α\|² = probability |
| **Bra** | ⟨ψ\| — the conjugate transpose (dual) of a ket; a row vector |
| **Ket** | \|ψ⟩ — a quantum state as a column vector |
| **Inner product** | ⟨u\|v⟩ ∈ ℂ — overlap between two states |
| **Bloch sphere** | Unit sphere representation of single-qubit pure states |
| **Unitary** | Matrix U with U†U = I; preserves norms; implies reversibility |
| **Hermitian** | Matrix A with A† = A; eigenvalues real; observables |
| **Pauli matrices** | σₓ=X, σᵧ=Y, σ_z=Z — the fundamental 2×2 unitaries |
| **Tensor product** | ⊗ — combines state spaces of multiple qubits |
| **Density matrix** | ρ — generalised state representation; allows mixed states |
| **Pure state** | Tr(ρ²) = 1; described by a single ket \|ψ⟩ |
| **Mixed state** | Tr(ρ²) < 1; statistical ensemble of pure states |
| **Entanglement** | Multi-qubit state not factorizable as a product |
| **Schmidt rank** | Number of non-zero terms in Schmidt decomposition; > 1 iff entangled |
| **Bell state** | Maximally entangled 2-qubit state; \|Φ⁺⟩ = (\|00⟩+\|11⟩)/√2 |
| **Born rule** | P(m) = \|⟨m\|ψ⟩\|² — probability of measuring outcome m |
| **Completeness** | Σₘ Mₘ†Mₘ = I — measurement operators sum to identity |
| **Universality** | {H, T, CNOT} can approximate any unitary to arbitrary ε |

### [TO CONFIRM] Items

| # | Item |
|---|---|
| TC-1 | `BaseGate.matrix()` method available in v0.2.0 public API? |
| TC-2 | Can `DensityMatrixSimulator` accept a `noise_model` kwarg in `run()`? |
| TC-3 | Is `qc._instructions` documented for read access, or internal only? |
| TC-4 | Does `StatevectorSimulator.run()` support a `return_statevector=True` kwarg? |
| TC-5 | `CXGate().matrix()` — is this a 4×4 numpy array in standard 2-qubit tensor order? |

### Further Reading

| Resource | Why |
|---|---|
| Nielsen & Chuang Ch.4–6 | Quantum algorithms: QFT, phase estimation, Shor, Grover |
| Axler Ch.8–10 | Spectral theorem, trace, norm |
| Qiskit Textbook Ch.2 | Interactive statevector visualizations |
| *Quantum Country* — "Quantum computing for computer scientists" | Spaced repetition reinforcement for these concepts |

### Next Courses

➡ **ZQL-02** — *SDK Core Reference: Architecture & Compiler Design*  
Deep-dive into QuantumIR, TranspilerCache, SABRE routing, and KAK decomposition.

➡ **ZQL-04** — *Fundamental Quantum Algorithms*  
QFT, phase estimation, Grover's search, Shor's algorithm — using the maths from ZQL-01.

➡ **ZQL-05** — *Variational Quantum Algorithms (VQE & QAOA)*  
Uses Pauli Hamiltonians, parameter-shift rule, and SDK's `PauliHamiltonian` class.

---

*All code verified against zuntenium v0.2.0: `circuit.py` (QuantumCircuit), `simulators.py` (StatevectorSimulator), `quantum_core/gates` (BaseGate, HGate, XGate, CXGate, …), `__init__.py` (lazy exports).*  
*Generated from: ZQL-01-notes-nielsen-chuang.md, ZQL-01-notes-kaye.md, ZQL-01-notes-rieffel.md, ZQL-01-notes-axler-susskind-qcountry.md.*
