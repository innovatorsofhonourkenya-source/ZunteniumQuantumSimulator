# ZQL-11 Step 1 Notes — Nielsen & Chuang Ch.10 + Roffe 2019 Review + Gottesman 1997

---

# A. Nielsen & Chuang — Ch. 10: Quantum Error Correction

## 10.1 Classical vs Quantum Error Correction

### Classical ECC
- 3-bit repetition code: 0→000, 1→111. Correct 1-bit flip by majority vote.
- Hamming distance d=3 → corrects 1 error.

### Quantum Challenges
1. **Bit-flip errors**: X gate noise → |ψ⟩ = α|0⟩+β|1⟩ → α|1⟩+β|0⟩.
2. **Phase-flip errors**: Z gate noise → α|0⟩+β|1⟩ → α|0⟩−β|1⟩.
3. **No-cloning**: cannot copy |ψ⟩ for redundancy.
4. **Continuous errors**: arbitrary rotation, not just discrete flips.

### Three-Qubit Bit-Flip Code
- Encode: α|0⟩+β|1⟩ → α|000⟩+β|111⟩.
- Syndrome measurement: measure Z₀Z₁ and Z₁Z₂ (2 ancilla qubits).
- No collapse of logical qubit — parity checks only!

```
Encode:    |ψ⟩──●──●──[data]
                │  │
           |0⟩──⊕──│──[ancilla 1]
           |0⟩─────⊕──[ancilla 2]

Syndrome:  Z₀Z₁ = +1 → no error
           Z₀Z₁ = −1 → X on qubit 0 or 1 (syndrome tells which)
```

## 10.2 Shor's 9-Qubit Code

Concatenates bit-flip + phase-flip protection:
- **Phase-flip code**: encode |ψ⟩ → (|000⟩+|111⟩)/√2 for |0⟩, (|000⟩−|111⟩)/√2 for |1⟩.
- **Bit-flip code**: apply inside each 3-qubit block.
- Result: 9 physical qubits encode 1 logical qubit, corrects any 1-qubit error.

### Parameters: [[9, 1, 3]] Code
- n=9 physical, k=1 logical, d=3 (distance).
- 8 stabilizer generators (8 ancilla measurements per cycle).

## 10.3 Stabilizer Formalism

A stabilizer code is defined by its **stabilizer group** S: set of Pauli operators {sᵢ} such that:
```
sᵢ|ψ_L⟩ = +|ψ_L⟩ for all i
```

The codespace = +1 eigenspace of all stabilizers.

### Error Detection
- Error E happens: Esᵢ = ±sᵢE. If E anticommutes with sᵢ: measurement of sᵢ gives −1 → syndrome detects error.
- Syndrome = pattern of {−1, +1} from stabilizer measurements → lookup table → correction.

### Logical Operators
- Logical X̄: Pauli operator that commutes with all stabilizers but flips |0̄⟩↔|1̄⟩.
- Logical Z̄: commutes with stabilizers but applies phase flip to |1̄⟩.
- Distance d = minimum weight of a logical operator.

---

# B. Joschka Roffe — *Quantum Error Correction: An Introductory Guide* (Contemp. Phys., 2019)

## Surface Code

### Structure
- 2D grid of data qubits + ancilla qubits.
- Distance-d surface code: d² + (d−1)² = 2d²−2d+1 data qubits.
- d=3: 9 data qubits + 8 ancilla = 17 total.
- Stabilizers: X-type (plaquettes) + Z-type (vertices).

### [[d², 1, d]] Surface Code
- n = d² physical qubits, k = 1 logical qubit, distance d.
- Threshold: ~1% physical error rate (per gate).
- Below threshold: more qubits → exponentially lower logical error.

### Threshold Theorem
If physical error rate p < p_th (threshold):
```
p_L ≈ A × (p/p_th)^{⌈d/2⌉}
```
where p_L = logical error rate, A = constant.

**[SDK]** `qec_encoder.py` implements surface code structure at d=3,5.

### Logical Error Rate vs Distance
| d | Physical qubits | p=0.5% | p=0.1% |
|---|---|---|---|
| 3 | 9 | 1.0e-2 | 5.0e-4 |
| 5 | 25 | 3.3e-4 | 1.7e-7 |
| 7 | 49 | 1.1e-5 | 5.8e-11 |
| 9 | 81 | 3.7e-7 | 2.0e-14 |

### Magic State Distillation
- Clifford gates are transversal in surface code (fault tolerant).
- Non-Clifford (T gate): requires magic state distillation.
- 15:1 protocol: 15 noisy T states → 1 high-fidelity T gate.
- Resource overhead: large — major reason quantum advantage is far-term.

---

# C. Daniel Gottesman — *Stabilizer Codes and Quantum Error Correction* (Caltech PhD thesis, 1997)

## CSS Codes (Calladeri-Shor-Steane)

CSS codes use two classical codes C₁⊇C₂:
```
Stabilizers:
  Z-type: Hᵤ for each u ∈ C₁⊥
  X-type: Hᵥ for each v ∈ C₂
```

### Steane [[7,1,3]] Code
- Derived from [7,4,3] Hamming code.
- 7 physical qubits, 1 logical qubit, distance 3.
- 6 stabilizer generators (3 X-type + 3 Z-type).

### Stabilizer Generators for Steane Code
```
X-type:  IIIXXXX, IXXIIXX, XIXIXIX
Z-type:  IIIZZZZ, IZZIIZZ, ZIZIZIZ

Logical X̄: XXXXXXX
Logical Z̄: ZZZZZZZ
```

## Transversal Gates
- **Transversal**: gate applied independently to each qubit — errors don't propagate.
- Steane code transversal gates: {X, Y, Z, H, CX, S} — all Clifford gates.
- T gate is NOT transversal — requires magic state distillation.
- **[SDK]**: `qec_encoder.py` encodes surface code; transversal Clifford gate application is O(d²) circuit.
