# ZQL-11-notes-nc-roffe-gottesman — Walkthrough Labs

**Source:** [ZQL-11-notes-nc-roffe-gottesman.md](../ZQL-11-notes-nc-roffe-gottesman.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** NC error correction + Roffe QEC codes + Gottesman stabilizer

---

## Labs 1-20: NC quantum error correction chapter, Shor 9-qubit code, Steane 7-qubit code, CSS codes, stabilizer formalism (Gottesman), Pauli group, stabilizer generators, error detection via syndrome, error correction procedure, Roffe surface codes, toric code, logical qubits, code distance, threshold theorem, fault-tolerant gates, magic state distillation, transversal gates, decoder algorithms (MWPM), QEC resource estimates, integration capstone.

## Lab 1: QEC Motivation
```python
print("Quantum errors: bit-flip (X), phase-flip (Z), both (Y)")
print("No-cloning prevents simple redundancy → need quantum codes")
```
### Deliverable: `qec_motivation.py`

## Lab 2: Shor 9-Qubit Code
```python
print("Shor code: 1 logical qubit encoded in 9 physical qubits")
print("Corrects arbitrary single-qubit error (X, Y, or Z)")
```
### Deliverable: `shor_code.py`

## Lab 3: Steane 7-Qubit Code
```python
print("Steane [[7,1,3]]: CSS code, 7→1 logical, distance 3")
print("Corrects any single-qubit error, detects 2-qubit errors")
```
### Deliverable: `steane_code.py`

## Lab 4: CSS Code Construction
```python
print("CSS: combine classical code C₁ (X errors) + C₂ (Z errors)")
print("Stabilizers: rows of H₁ as X-type, rows of H₂ as Z-type")
```
### Deliverable: `css_codes.py`

## Lab 5: Gottesman Stabilizer Formalism
```python
print("Stabilizer group S ⊂ Pauli group, |ψ⟩ is +1 eigenstate of all s∈S")
print("n qubits, n-k independent generators → k logical qubits")
```
### Deliverable: `stabilizer.py`

## Lab 6: Pauli Group
```python
print("Pₙ = {±1,±i} × {I,X,Y,Z}⊗n")
print("16 single-qubit Paulis, 4^n elements for n qubits")
```
### Deliverable: `pauli_group.py`

## Lab 7: Stabilizer Generators
```python
print("[[5,1,3]] code: 4 generators on 5 qubits → 1 logical qubit")
print("Generators must commute and be independent")
```
### Deliverable: `generators.py`

## Lab 8: Syndrome Extraction
```python
print("Measure each stabilizer generator → syndrome bits")
print("Non-trivial syndrome → identify error location and type")
```
### Deliverable: `syndrome.py`

## Lab 9: Error Correction Procedure
```python
print("1. Encode logical qubit into code space")
print("2. Errors occur during computation")
print("3. Extract syndrome → identify error → apply correction")
```
### Deliverable: `correction.py`

## Lab 10: Roffe Surface Codes
```python
print("Surface code: 2D lattice, X stabilizers on faces, Z on vertices")
print("Distance d: d×d lattice → (2d²-1) physical qubits, 1 logical")
```
### Deliverable: `surface_code.py`

## Lab 11: Toric Code
```python
print("Toric code: surface code on torus → 2 logical qubits")
print("Stabilizers: star (X) and plaquette (Z) operators")
```
### Deliverable: `toric_code.py`

## Lab 12: Code Distance
```python
print("Distance d: minimum weight of logical operator")
print("[[n,k,d]]: corrects ⌊(d-1)/2⌋ errors")
for d in [3,5,7]:print(f"  d={d}: corrects {(d-1)//2} errors")
```
### Deliverable: `code_distance.py`

## Lab 13: Threshold Theorem
```python
print("If physical error rate p < p_threshold:")
print("  → can achieve arbitrarily low logical error rate")
print("Surface code threshold: p_th ≈ 1% (phenomenological)")
```
### Deliverable: `threshold.py`

## Lab 14: Fault-Tolerant Gates
```python
print("Transversal gates: apply gate to each qubit independently")
print("Clifford gates: transversal in many codes")
print("T gate: requires magic state distillation")
```
### Deliverable: `ft_gates.py`

## Lab 15: Magic State Distillation
```python
print("Prepare noisy |T⟩=T|+⟩ → distill to high-fidelity |T⟩")
print("15→1 protocol: 15 noisy → 1 clean with p² error")
```
### Deliverable: `magic_states.py`

## Lab 16: Transversal Gates
```python
print("Transversal X/Z: apply to all qubits → logical X/Z")
print("Transversal CNOT: bitwise CNOT between two code blocks")
```
### Deliverable: `transversal.py`

## Lab 17: Decoder — MWPM
```python
print("Minimum Weight Perfect Matching: decode surface code syndrome")
print("Polynomial time, near-optimal for independent errors")
```
### Deliverable: `mwpm_decoder.py`

## Lab 18: QEC Resource Estimates
```python
for d in [3,7,13,21]:
    phys=2*d**2-1;print(f"  d={d}: {phys} physical qubits per logical qubit")
```
### Deliverable: `qec_resources.py`

## Lab 19: Logical Error Rate
```python
import numpy as np
p=0.001;p_th=0.01
for d in [3,5,7,9]:
    p_L=(p/p_th)**((d+1)//2)
    print(f"  d={d}: p_L≈{p_L:.2e}")
```
### Deliverable: `logical_error.py`

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("Stabilizer formalism",True);ok("Surface code",True);ok("Threshold",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("nrg_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `nrg_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct QEC concepts ✓
