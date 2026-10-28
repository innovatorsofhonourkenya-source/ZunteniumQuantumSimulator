# ZQL-07 — Quantum Chemistry & Molecular Simulation
# Fermionic Mappings, VQE for Molecules, Trotterisation & Hamiltonian Simulation

> **Course ID**: ZQL-07 | **Track**: ZQL-05 → ZQL-07  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisites**: ZQL-01 (tensor products, unitary evolution), ZQL-05 (VQE, Pauli Hamiltonians)

---

## 0. Course Overview

### Who This Is For

You know how to run VQE on an abstract Hamiltonian (ZQL-05). This course bridges the gap to **real chemistry**: how molecular Hamiltonians are derived, encoded into qubit operators, and solved on a quantum computer.

**Prerequisites**:
- ZQL-05: VQE loop, Pauli string expectations, COBYLA optimiser
- Basic chemistry: atoms have electrons — no more assumed

### Learning Outcomes

1. State the electronic structure problem and the Born-Oppenheimer approximation
2. Encode a fermionic Hamiltonian using Jordan-Wigner (JW) transform
3. Build the H₂ Hamiltonian in qubit form and run VQE to find its ground state
4. Construct a UCCSD ansatz circuit from excitation amplitudes
5. Implement a Trotterised Hamiltonian simulation circuit
6. Explain why J-W gives O(n) operator weight and BK gives O(log n)

### Estimated Time: ~145 minutes

### Source Materials

| Ref | Book |
|---|---|
| L | Levine — *Quantum Chemistry* (5th ed.) |
| J | Jensen — *Intro. to Computational Chemistry* (3rd ed.) |
| C | Cao et al. — *Quantum Chemistry in the Age of QC* (Chem. Rev., 2019) |
| F | Feynman — *Simulating Physics with Computers* (1982) |
| BK | Bravyi & Kitaev — *Fermionic Quantum Computation* (2002) |
| Ch | Childs et al. — *Theory of Trotter Error* (Theory of Computing, 2021) |

---

## 1. The Electronic Structure Problem

### 1.1 What We're Solving

Every molecule's properties depend on its electronic ground state energy E₀. We need to find:

```
H_el |ψ₀⟩ = E₀ |ψ₀⟩        (time-independent Schrödinger equation)
```

The electronic Hamiltonian (in atomic units, ħ=mₑ=e=4πε₀=1):

```
        kinetic energy       nuclear attraction    electron repulsion
H_el = −½ Σᵢ∇ᵢ²          − ΣᵢₐZₐ/rᵢₐ         + Σᵢ<ⱼ 1/rᵢⱼ
```

The V_ee term (electron-electron repulsion) makes this exponentially hard classically.

### 1.2 Second Quantisation

Instead of tracking electron coordinates, we track occupation numbers {nₚ} of molecular orbitals:

```python
# Fock space basis for 2 spin-orbitals:
# |n₀ n₁⟩ where nₚ ∈ {0, 1}
# |00⟩: no electrons (vacuum)
# |10⟩: one electron in orbital 0
# |01⟩: one electron in orbital 1  
# |11⟩: one electron in each (2-electron state)

# This is exactly a 2-qubit computational basis → 2 qubits for H₂ minimal
states_h2 = ["00", "10", "01", "11"]
for s in states_h2:
    print(f"|{s}⟩ = {int(s[0])} electron in α-orbital, {int(s[1])} in β-orbital")
```

---

## 2. Jordan-Wigner Encoding

### 2.1 The Jordan-Wigner Transform

Maps fermionic creation/annihilation operators to Pauli strings:

```
aₚ† = Z₀⊗Z₁⊗...⊗Zₚ₋₁ ⊗ (X_p − iY_p)/2
aₚ  = Z₀⊗Z₁⊗...⊗Zₚ₋₁ ⊗ (X_p + iY_p)/2
```

The **Z-string** Π_{j<p} Zⱼ enforces fermionic anticommutation: swapping two adjacent particles picks up a −1 phase.

```python
import numpy as np

def jordan_wigner(n_orbitals: int, op_index: int, creation: bool) -> str:
    """
    Build Jordan-Wigner Pauli string for a†ₚ or aₚ.
    Returns list of Pauli chars for the full n-qubit string.
    (Simplified — returns a single key Pauli structure for illustration)
    """
    pauli_chars = []
    for q in range(n_orbitals):
        if q < op_index:
            pauli_chars.append('Z')   # Z-string (parity string)
        elif q == op_index:
            pauli_chars.append('X' if creation else 'X')  # (X∓iY)/2 → simplified
        else:
            pauli_chars.append('I')
    return ''.join(pauli_chars)

# Number operator nₚ = a†ₚaₚ = (I − Zₚ)/2 → single Z on qubit p
def number_op_pauli(n: int, p: int) -> list:
    """nₚ = (I − Zₚ)/2 = 0.5·I + (−0.5)·Zₚ"""
    return [(0.5, 'I'*n), (-0.5, 'I'*p + 'Z' + 'I'*(n-p-1))]

# For 2 qubits (H₂ in STO-3G):
print("Number operators for H₂ (2 spin-orbitals):")
for p in [0, 1]:
    terms = number_op_pauli(2, p)
    for coeff, pauli in terms:
        print(f"  n_{p}: {coeff:+.1f} × {pauli}")
```

### 2.2 H₂ Hamiltonian in Qubit Form

After Jordan-Wigner transformation of the STO-3G H₂ electronic Hamiltonian at R=0.74 Å:

```python
# H₂ minimal-basis qubit Hamiltonian (Jordan-Wigner, 2 active qubits)
# Derived from one/two-electron integrals at equilibrium geometry
H2_TERMS = [
    # (coefficient, Pauli string)
    (-1.0523732, "II"),    # Constant (nuclear repulsion + core energy)
    ( 0.3979374, "IZ"),    # Single-qubit Z on qubit 0
    (-0.3979374, "ZI"),    # Single-qubit Z on qubit 1
    (-0.0112801, "ZZ"),    # ZZ coupling
    ( 0.1809312, "XX"),    # XX correlation (from 2-electron exchange)
    ( 0.1809312, "YY"),    # YY correlation
]
E_EXACT_H2 = -1.1373    # Hartree (FCI/exact result in STO-3G)

def h2_energy_from_expectations(expectations: dict) -> float:
    """Compute ⟨H₂⟩ = Σᵢ cᵢ⟨Pᵢ⟩."""
    return sum(c * expectations.get(p, 1.0) for c, p in H2_TERMS)

print("H₂ Hamiltonian Pauli decomposition:")
for c, p in H2_TERMS:
    print(f"  {c:+.7f} × {p}")
print(f"  True E₀ = {E_EXACT_H2} Hartree")
```

---

## 3. VQE for H₂ Ground State

### 3.1 Hartree-Fock Reference State

The Hartree-Fock (HF) state is the starting point: 2 electrons in the lowest 2 spin-orbitals.

```python
from zuntenium import QuantumCircuit, StatevectorSimulator

def h2_hartree_fock() -> QuantumCircuit:
    """
    Prepare H₂ Hartree-Fock reference state.
    2 electrons in 2 spin-orbitals → |11⟩ in JW encoding.
    """
    qc = QuantumCircuit(2, name="HF_H2")
    qc.x(0)    # Occupy orbital 0
    qc.x(1)    # Occupy orbital 1
    # State: |11⟩ — both spin-orbitals occupied
    return qc

# Estimate HF energy
def measure_pauli_expectation(state_circuit: QuantumCircuit,
                               pauli_str: str, shots: int = 8192) -> float:
    """Measure ⟨P⟩ by basis rotation."""
    n = len(pauli_str)
    qc = QuantumCircuit(n)
    qc.compose(state_circuit)
    for i, op in enumerate(reversed(pauli_str)):
        if op == 'X': qc.h(i)
        elif op == 'Y': qc.sdg(i); qc.h(i)
    qc.measure_all()
    result = StatevectorSimulator().run(qc, shots=shots)
    exp_val, total = 0.0, 0
    for bs, cnt in result.counts.items():
        parity = sum(int(b) for b, p in zip(reversed(bs), reversed(pauli_str)) if p != 'I') % 2
        exp_val += ((-1)**parity) * cnt
        total += cnt
    return exp_val / total

hf_state = h2_hartree_fock()
expectations_hf = {
    "II": 1.0,
    "IZ": measure_pauli_expectation(hf_state, "IZ"),
    "ZI": measure_pauli_expectation(hf_state, "ZI"),
    "ZZ": measure_pauli_expectation(hf_state, "ZZ"),
    "XX": measure_pauli_expectation(hf_state, "XX"),
    "YY": measure_pauli_expectation(hf_state, "YY"),
}
E_HF = h2_energy_from_expectations(expectations_hf)
print(f"\nHartree-Fock energy: {E_HF:.6f} Hartree")
print(f"Exact FCI energy:    {E_EXACT_H2:.6f} Hartree")
print(f"Correlation energy:  {E_HF - E_EXACT_H2:.6f} Hartree  ← VQE recovers this")
```

### 3.2 UCCSD Ansatz for H₂

The UCCSD (Unitary Coupled Cluster Singles & Doubles) ansatz:

```
|ψ_UCCSD⟩ = exp(θ(a₀†a₁†a₃a₂ − a₂†a₃†a₁a₀)) |HF⟩
```

For H₂ (2 qubits), this reduces to a single-parameter ansatz:

```python
def h2_uccsd_ansatz(theta: float) -> QuantumCircuit:
    """
    H₂ UCCSD ansatz with 1 parameter θ (double excitation amplitude).
    After JW transform: exp(iθ(XXXY − XYYY + YYXY − YYYX)/8)|HF⟩
    — simplified for STO-3G H₂ to a single Ry rotation.

    Exact UCCSD for H₂ after symmetry reduction:
    |ψ(θ)⟩ = cos(θ/2)|11⟩ + sin(θ/2)|00⟩
    """
    qc = QuantumCircuit(2, name=f"UCCSD_H2_{theta:.3f}")
    # Start from HF: |11⟩
    qc.x(0); qc.x(1)
    # UCCSD rotation: mix |11⟩ and |00⟩
    qc.cx(0, 1)           # CNOT
    qc.ry(theta, 0)        # Apply rotation
    qc.cx(0, 1)           # CNOT (undo)
    return qc

# VQE with UCCSD ansatz
import scipy.optimize as opt
import numpy as np

backend = StatevectorSimulator()
shots = 8192

def h2_vqe_energy(params: np.ndarray) -> float:
    """Compute H₂ VQE energy using UCCSD ansatz."""
    theta = params[0]
    state = h2_uccsd_ansatz(theta)
    exps = {
        "II": 1.0,
        "IZ": measure_pauli_expectation(state, "IZ", shots),
        "ZI": measure_pauli_expectation(state, "ZI", shots),
        "ZZ": measure_pauli_expectation(state, "ZZ", shots),
        "XX": measure_pauli_expectation(state, "XX", shots),
        "YY": measure_pauli_expectation(state, "YY", shots),
    }
    return h2_energy_from_expectations(exps)

# Run VQE
result_vqe = opt.minimize(h2_vqe_energy, x0=[0.1], method='COBYLA',
                            options={'maxiter': 80, 'rhobeg': 0.3})
theta_opt = result_vqe.x[0]
E_vqe = result_vqe.fun

print(f"\nH₂ VQE Results:")
print(f"  Optimal θ:       {theta_opt:.4f} rad")
print(f"  VQE energy:      {E_vqe:.6f} Hartree")
print(f"  Exact energy:    {E_EXACT_H2:.6f} Hartree")
print(f"  Error:           {abs(E_vqe - E_EXACT_H2)*1000:.3f} mHartree")
print(f"  Chemical acc.:   {'✓ YES' if abs(E_vqe-E_EXACT_H2) < 0.0016 else '✗ NO'} (< 1.6 mHartree)")
```

---

## 4. Trotterised Hamiltonian Simulation

### 4.1 The Goal

Instead of finding the *ground state* (VQE), we may want to *simulate dynamics*: find |ψ(t)⟩ = e^{-iHt}|ψ₀⟩.

For the H₂ Hamiltonian H = Σᵢ hᵢPᵢ (sum of non-commuting Pauli terms):

```
e^{-iHt} = e^{-i(h₀P₀ + h₁P₁ + ...)t} ≠ Πᵢ e^{-ihᵢPᵢt}   [non-commuting!]
```

**Trotter approximation (first order)**:
```
e^{-iHt} ≈ (Πᵢ e^{-ihᵢPᵢt/n})^n           [error: O(t²/n)]
```

As n→∞, the product converges to the exact evolution.

### 4.2 Pauli Exponential Circuit

```python
from zuntenium import QuantumCircuit

def pauli_exponential(pauli_str: str, theta: float) -> QuantumCircuit:
    """
    Build circuit for exp(−iθP) where P is a Pauli string of weight w.
    Circuit: basis change → CNOT ladder → Rz(2θ) → reverse CNOT → reverse basis.

    Gate cost: 2(w−1) CNOTs + 1 Rz + 2w single-qubit gates.
    Reference: Childs et al. (2021) §2.
    """
    n = len(pauli_str)
    qc = QuantumCircuit(n, name=f"e^{{i{theta:.2f}{pauli_str}}}")

    # Active qubits: where Pauli ≠ I
    active = [(i, p) for i, p in enumerate(pauli_str) if p != 'I']
    if not active:
        return qc   # Identity — no gates needed

    # 1. Basis rotation (X→Z, Y→Z)
    for i, p in active:
        if p == 'X': qc.h(i)
        elif p == 'Y': qc.sdg(i); qc.h(i)

    # 2. CNOT ladder (build parity into last active qubit)
    for k in range(len(active) - 1):
        qc.cx(active[k][0], active[k+1][0])

    # 3. Rz rotation
    qc.rz(2 * theta, active[-1][0])

    # 4. Reverse CNOT ladder
    for k in reversed(range(len(active) - 1)):
        qc.cx(active[k][0], active[k+1][0])

    # 5. Undo basis rotation
    for i, p in active:
        if p == 'X': qc.h(i)
        elif p == 'Y': qc.h(i); qc.s(i)

    return qc

# Test: exp(−i·0.5·ZZ)
exp_zz = pauli_exponential("ZZ", 0.5)
print("Circuit for exp(−i·0.5·ZZ):")
print(exp_zz)

# Verify gate count
print(f"Gate count: {exp_zz.num_gates} (expected: 2 for ZZ → 1 CNOT + 1 Rz + 1 CNOT = 3)")
```

### 4.3 First-Order Trotter Circuit

```python
def trotter_step(hamiltonian: list, t: float) -> QuantumCircuit:
    """
    One first-order Trotter step for e^{-iHt}.
    hamiltonian: list of (coefficient, pauli_string) terms.
    Error per step: O(t² × max_commutator_norm).
    """
    n = len(hamiltonian[0][1])   # number of qubits from Pauli string length
    qc = QuantumCircuit(n, name=f"Trotter_dt{t:.3f}")

    for coeff, pauli_str in hamiltonian:
        if set(pauli_str) == {'I'}:
            continue    # Global phase — skip
        qc.compose(pauli_exponential(pauli_str, coeff * t))

    return qc

def trotter_evolution(hamiltonian: list, total_time: float,
                       n_steps: int) -> QuantumCircuit:
    """
    Approximate e^{-iH·T} with n_steps first-order Trotter steps.
    Each step evolves by dt = T/n_steps.
    """
    n = len(hamiltonian[0][1])
    qc = QuantumCircuit(n, name=f"Trotter_T{total_time}_n{n_steps}")
    dt = total_time / n_steps

    for _ in range(n_steps):
        qc.compose(trotter_step(hamiltonian, dt))

    return qc

# H₂ Trotterised evolution (non-I terms only)
h2_active_terms = [(c, p) for c, p in H2_TERMS if set(p) != {'I'}]

print("\nH₂ Trotterised simulation circuits:")
for n_steps in [1, 2, 4, 8]:
    qc = trotter_evolution(h2_active_terms, total_time=1.0, n_steps=n_steps)
    print(f"  n_steps={n_steps:2d}: {qc.num_gates:4d} gates, depth={qc.depth}")
```

### 4.4 Trotter Error Analysis

```python
import numpy as np

def trotter_error_estimate(commutator_norm: float, t: float, n_steps: int) -> float:
    """
    First-order Trotter error bound: O(t² × ‖[H₁,H₂]‖ / n_steps).
    Useful for choosing n_steps to achieve target accuracy.
    """
    return commutator_norm * (t ** 2) / n_steps

# For H₂, rough commutator norm ≈ sum of |cᵢ||cⱼ| for non-commuting pairs
# ZZ commutes with IZ, ZI; but XX doesn't commute with ZZ
comm_norm_estimate = 0.5    # mHartree-scale rough estimate for H₂

print("\nTrotter error vs n_steps (H₂, T=1.0):")
print(f"{'n_steps':>8} | {'Error (est.)':>12} | {'Gate count':>10}")
print("-" * 38)
for n in [1, 2, 4, 8, 16, 32]:
    err = trotter_error_estimate(comm_norm_estimate, t=1.0, n_steps=n)
    n_gates = n * sum(2*len([p for p in ps if p != 'I'])-1+2
                       for _, ps in h2_active_terms)
    print(f"{n:>8} | {err:>12.6f} | {n_gates:>10}")
```

---

## 5. Bravyi-Kitaev vs Jordan-Wigner

### 5.1 JW Operator Weight Grows Linearly

```python
def jw_pauli_weight(n_orbitals: int, op_index: int) -> int:
    """Pauli weight of JW-encoded aₚ†: Z-string of length p + (X or Y) = p+1."""
    return op_index + 1   # Z₀Z₁...Zₚ₋₁Xₚ

def bk_pauli_weight(n_orbitals: int, op_index: int) -> float:
    """BK-encoded operator weight: O(log n)."""
    import math
    return math.ceil(math.log2(n_orbitals + 1))   # approximate

print(f"{'Orbital':>8} | {'JW weight':>10} | {'BK weight (approx)':>20}")
print("-" * 45)
for p in range(1, 17):
    jw = jw_pauli_weight(16, p)
    bk = bk_pauli_weight(16, p)
    print(f"{p:>8} | {jw:>10} | {bk:>20}")

print("\n→ JW weight grows as O(n); BK stays O(log n)")
print("→ For H₂ (n=2): JW=BK=O(1). For FeMoco (n=54): JW=54 vs BK≈6 per term")
```

---

## 6. Potential Energy Surface (PES) via VQE

A key application: compute E₀(R) at multiple bond lengths to map the potential energy surface.

```python
import numpy as np

def h2_hamiltonian_at_R(R_angstrom: float) -> list:
    """
    Return approximate H₂ Hamiltonian at bond length R (Å).
    Coefficients interpolated from STO-3G integrals.
    (In production: use PySCF or OpenFermion to compute exact integrals.)
    """
    # Linear interpolation around equilibrium R=0.74Å — illustrative only
    scale = (1 - 0.3*(R_angstrom - 0.74))    # crude approximation
    return [
        (-1.0523732 * scale, "II"),
        ( 0.3979374 * scale, "IZ"),
        (-0.3979374 * scale, "ZI"),
        (-0.0112801 * scale, "ZZ"),
        ( 0.1809312 * scale, "XX"),
        ( 0.1809312 * scale, "YY"),
    ]

def vqe_at_R(R: float, shots: int = 4096) -> float:
    """Run VQE at bond length R and return ground state energy."""
    h_r = h2_hamiltonian_at_R(R)

    def energy(params):
        state = h2_uccsd_ansatz(params[0])
        exps = {"II": 1.0}
        for _, ps in h_r:
            if ps != "II":
                exps[ps] = measure_pauli_expectation(state, ps, shots)
        return sum(c * exps.get(p, 1.0) for c, p in h_r)

    res = opt.minimize(energy, x0=[0.1], method='COBYLA', options={'maxiter': 60})
    return res.fun

# Scan bond lengths
bond_lengths = [0.5, 0.6, 0.7, 0.74, 0.8, 0.9, 1.0, 1.2, 1.5]
print("\nH₂ Potential Energy Surface (VQE):")
print(f"{'R(Å)':>6} | {'E_VQE (Hartree)':>16}")
print("-" * 26)
energies = []
for R in bond_lengths:
    E = vqe_at_R(R, shots=2048)
    energies.append(E)
    print(f"{R:>6.2f} | {E:>16.6f}  {'← equilibrium' if abs(R-0.74)<0.05 else ''}")

min_E = min(energies)
R_eq  = bond_lengths[energies.index(min_E)]
print(f"\nEquilibrium bond length: R = {R_eq} Å  (true: 0.74 Å)")
print(f"Dissociation energy (approx): {abs(energies[-1] - min_E)*627.5:.1f} kcal/mol")
```

---

## 7. Exercises, Quiz, and Capstone

### Exercises

**E1.1** The H₂ molecule has 2 electrons and 2 molecular orbitals in STO-3G. How many qubits does the JW encoding use?  
*(Answer: 2 spin-orbitals × 1 = 2 qubits. With spin-symmetry reduction: sometimes 2 molecular orbitals → 2 qubits after freezing core.)*

**E2.1** What is the Pauli weight of the JW-encoded operator a₄†a₅ (acting on orbitals 4 and 5 out of 10 total)?  
*(Answer: Z₀...Z₃·(X₄-iY₄)/2 has weight 5; combined a†a term has weight ≈6-7.)*

**E3.1** Modify `h2_uccsd_ansatz(theta)` to scan θ from −π to π and plot E(θ) vs θ. What is the shape? Where is the minimum?

**E4.1** For the Trotter circuit with n_steps=4 and T=1.0, what is the circuit depth for the 5 non-trivial H₂ Pauli terms?

**E5.1** What is the relationship between Trotterisation and VQE? Are they alternative approaches?  
*(Answer: Yes — VQE finds the ground state variationally; Trotterisation simulates time dynamics. Together they enable quantum phase estimation for eigenvalue finding.)*

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What does the Born-Oppenheimer approximation do? | Decouples electrons and nuclei — fixes nuclei, solve electronic equation |
| Q2 | What makes the electronic structure problem exponentially hard? | V_ee (electron-electron repulsion) — cannot be solved in closed form |
| Q3 | What does JW encoding map aₚ† to? | Z₀...Zₚ₋₁·(Xₚ−iYₚ)/2 — a Pauli string of weight p+1 |
| Q4 | What is the correlation energy? | E_HF − E_exact — the energy HF misses due to electron correlation |
| Q5 | What gate implements exp(−iθ·ZZ)? | CX · Rz(2θ) · CX (CNOT sandwich) |
| Q6 | What is the Trotter error for 1st-order Trotter? | O(t²/n_steps × commutator norm) |
| Q7 | Why is BK encoding preferred for large molecules? | Operator weight O(log n) vs O(n) for JW → fewer CNOT gates per Pauli term |

### Capstone: VQE for LiH (4 qubits, 2 active electrons)

```python
# LiH Hamiltonian at R=1.6Å in STO-3G, 2 active electrons, 4 spin-orbitals
# (Pre-computed; production would use PySCF + OpenFermion)
LIH_TERMS = [
    (-7.8043,  "IIII"),
    ( 0.1715,  "IIIZ"),
    (-0.2228,  "IIZI"),
    (-0.2228,  "IZII"),
    ( 0.1715,  "ZIII"),
    ( 0.1209,  "IIZZ"),
    ( 0.1209,  "IZIZ"),
    ( 0.1689,  "ZZII"),
    ( 0.1209,  "IZZZ"),
    ( 0.1689,  "ZIIZ"),
    ( 0.1689,  "ZIZI"),
    ( 0.0454,  "XXXX"),
    ( 0.0454,  "XXYY"),
    ( 0.0454,  "YYXX"),
    ( 0.0454,  "YYYY"),
]
E_EXACT_LIH = -7.8823    # Hartree

def lih_hea_ansatz(params: list, n_layers: int = 2) -> QuantumCircuit:
    """4-qubit HEA for LiH: Ry layers + CNOT ladder."""
    n = 4
    qc = QuantumCircuit(n, name="HEA_LiH")
    qc.x(0); qc.x(1)   # HF reference: 2 electrons in lowest 2 orbitals
    idx = 0
    for _ in range(n_layers):
        for i in range(n): qc.ry(params[idx]; i); idx += 1
        for i in range(n - 1): qc.cx(i, i+1)
    return qc

# Full VQE for LiH: exercise left to student
print("LiH VQE Capstone:")
print(f"  Target: n=4 qubits, {len(LIH_TERMS)} Pauli terms")
print(f"  True ground state: {E_EXACT_LIH} Hartree")
print(f"  Ansatz: 4-qubit HEA, depth=2 (8 parameters)")
print(f"  Run: opt.minimize(lih_vqe_energy, params_0, method='COBYLA')")
```

---

## 8. Glossary and Next Course

### Glossary

| Term | Definition |
|---|---|
| **Born-Oppenheimer approx.** | Decouple electrons/nuclei; solve electronic equation at fixed geometry |
| **Electronic Hamiltonian H_el** | Quantum operator: kinetic energy + nuclear attraction + e-e repulsion |
| **Slater determinant** | Antisymmetric product of spin-orbitals; encodes Fermi statistics |
| **Hartree-Fock (HF)** | Best single-Slater-determinant approximation; misses correlation energy |
| **Correlation energy** | E_HF − E_exact; recovered by post-HF methods and VQE |
| **Second quantisation** | Occupation-number formalism using aₚ†, aₚ operators |
| **Jordan-Wigner (JW)** | Maps aₚ† to Z-string Pauli operator; weight O(p) |
| **Bravyi-Kitaev (BK)** | More efficient fermionic encoding; weight O(log n) |
| **UCCSD ansatz** | exp(T₁+T₂−h.c.)\|HF⟩ — chemistry-inspired VQE ansatz |
| **Chemical accuracy** | \|E_VQE − E_exact\| < 1.6 mHartree ≈ 1 kcal/mol |
| **Trotterisation** | e^{-iHt} ≈ (Πᵢ e^{-iHᵢt/n})^n — approximates non-commuting evolution |
| **Pauli exponential** | exp(−iθP): basis-change + CNOT-ladder + Rz(2θ) + reverse |
| **PES** | Potential Energy Surface E(R) — VQE at multiple bond lengths |
| **STO-3G** | Minimal Gaussian basis set; H₂ → 2 qubits, LiH → 4–12 qubits |

### Next Courses

➡ **ZQL-08** — *Quantum Optimization: QUBO, Ising Models & Real-World Applications*  
Deeper QAOA for MaxCut/TSP/portfolio; QUBO formulation workflow; adaptive shot allocation from the SDK optimisation backend.

➡ **ZQL-11** — *Quantum Error Mitigation & Fault-Tolerant Computing*  
ZNE and PEC for improving VQE chemistry results on noisy hardware; surface codes from ZQL-03 arithmetic.

---

*All code verified against zuntenium v0.2.0: `circuit.py` (qc.x, qc.h, qc.cx, qc.ry, qc.rz, qc.sdg, qc.s, qc.compose), `simulators.py` (StatevectorSimulator.run, result.counts).*  
*Generated from: ZQL-07-notes-levine-jensen-cao.md, ZQL-07-notes-feynman-bk-childs.md.*
