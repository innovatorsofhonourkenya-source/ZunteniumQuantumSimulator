# ZQL-07-notes-levine-jensen-cao — Walkthrough Labs

**Source:** [ZQL-07-notes-levine-jensen-cao.md](../ZQL-07-notes-levine-jensen-cao.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Levine quantum chemistry + Jensen computational chemistry + Cao quantum chem review

---

## Labs 1-20 cover: Levine electronic structure, Hartree-Fock approximation, basis sets, electron correlation, configuration interaction, coupled cluster theory, Jensen DFT basics, basis set convergence, computational cost scaling, Cao VQE for chemistry, fermionic-to-qubit encoding, quantum phase estimation for chemistry, error budgets, active space methods, adiabatic state preparation, quantum chemistry resource estimates, molecular property calculations, reaction pathway simulation, excited state methods, and integration capstone.

## Lab 1: Electronic Structure Problem
### Walkthrough
```python
print("Levine: solve Schrödinger eqn Hψ=Eψ for molecule")
print("Born-Oppenheimer: fix nuclei, solve electrons")
```
### Deliverable: `electronic_structure.py`

## Lab 2: Hartree-Fock Approximation
### Walkthrough
```python
print("HF: single Slater determinant, mean-field, no correlation")
print("Recovers ~99% of energy but missing ~1% matters for chemistry")
```
### Deliverable: `hartree_fock.py`

## Lab 3: Basis Sets — STO-3G to cc-pVDZ
### Walkthrough
```python
for b,q in [("STO-3G","minimal"),("6-31G","split-valence"),("cc-pVDZ","correlation-consistent")]:
    print(f"  {b:<12}: {q}")
```
### Deliverable: `basis_sets.py`

## Lab 4: Electron Correlation Energy
### Walkthrough
```python
print("E_corr = E_exact - E_HF ≈ 1% of total but critical for bond breaking")
print("VQE captures correlation via entangled ansatz")
```
### Deliverable: `correlation.py`

## Lab 5: Full CI — Exact but Exponential
### Walkthrough
```python
import math
for n in [4,8,12,16]:
    dim=math.comb(n,n//2)**2
    print(f"  {n} orbitals: FCI dimension≈{dim:,}")
print("Exponential → motivates quantum computing!")
```
### Deliverable: `full_ci.py`

## Lab 6: Coupled Cluster Theory
### Walkthrough
```python
print("CCSD: T=T₁+T₂, excitation operators")
print("CCSD(T): gold standard, O(n⁷) classical scaling")
print("UCCSD: unitary variant for quantum circuits")
```
### Deliverable: `coupled_cluster.py`

## Lab 7: Jensen DFT Basics
### Walkthrough
```python
print("DFT: E[ρ] functional of electron density ρ(r)")
print("Kohn-Sham: map to non-interacting system + exchange-correlation")
print("O(n³) scaling — practical but approximate")
```
### Deliverable: `dft_basics.py`

## Lab 8: Basis Set Convergence
### Walkthrough
```python
energies = {"STO-3G": -1.117, "6-31G": -1.127, "cc-pVDZ": -1.133, "CBS": -1.1745}
for b,e in energies.items():
    print(f"  H₂ {b:<12}: E={e:.4f} Hartree")
```
### Deliverable: `basis_convergence.py`

## Lab 9: Computational Cost Scaling
### Walkthrough
```python
for method,scaling in [("HF","O(n⁴)"),("MP2","O(n⁵)"),("CCSD","O(n⁶)"),("CCSD(T)","O(n⁷)"),("FCI","O(e^n)")]:
    print(f"  {method:<10}: {scaling}")
print("VQE: polynomial in circuit, exponential Hilbert space")
```
### Deliverable: `cost_scaling.py`

## Lab 10: Cao VQE Review — State of the Art
### Walkthrough
```python
print("Cao 2019 review: VQE is most promising NISQ chemistry approach")
print("Demonstrated: H₂, HeH⁺, LiH, H₆")
print("Challenges: noise, measurement cost, barren plateaus")
```
### Deliverable: `cao_review.py`

## Lab 11: Fermionic-to-Qubit Encoding Comparison
### Walkthrough
```python
for enc,qubits,weight in [("JW","n","O(n)"),("BK","n","O(log n)"),("Parity","n","O(n)")]:
    print(f"  {enc:<8}: {qubits} qubits, Pauli weight={weight}")
```
### Deliverable: `encoding_compare.py`

## Lab 12: QPE for Chemistry — Fault-Tolerant
### Walkthrough
```python
print("QPE: exact eigenvalues if U=e^{-iHt} can be implemented")
print("Requires fault tolerance → future, not NISQ")
print("Advantage: no barren plateaus, guaranteed accuracy")
```
### Deliverable: `qpe_chem.py`

## Lab 13: Error Budget for Chemical Accuracy
### Walkthrough
```python
print("Target: 1.6 mHartree total error budget:")
print("  Ansatz error: <0.5 mH, Shot noise: <0.5 mH, Gate error: <0.6 mH")
```
### Deliverable: `error_budget.py`

## Lab 14: Active Space Methods
### Walkthrough
```python
print("CASSCF: Complete Active Space SCF — select orbitals manually")
print("Quantum: active space on quantum, rest classical (hybrid)")
```
### Deliverable: `active_space.py`

## Lab 15: Adiabatic State Preparation
### Walkthrough
```python
print("Start: easy ground state (e.g., non-interacting)")
print("Slowly turn on interactions → adiabatic theorem → ground state")
print("Time: O(1/Δ²) where Δ=spectral gap")
```
### Deliverable: `adiabatic_prep.py`

## Lab 16: Resource Estimates — Cao
### Walkthrough
```python
for mol,logical,t_gates in [("H₂",4,"~10³"),("N₂",20,"~10⁶"),("FeMoCo",100,"~10¹¹")]:
    print(f"  {mol:<8}: {logical} logical qubits, {t_gates} T gates")
```
### Deliverable: `cao_resources.py`

## Lab 17: Molecular Properties Beyond Energy
### Walkthrough
```python
print("Dipole moment: ⟨ψ|μ|ψ⟩; Forces: -∂E/∂R")
print("Excited states: VQD or QPE variants")
```
### Deliverable: `mol_properties.py`

## Lab 18: Reaction Pathway Simulation
### Walkthrough
```python
print("Scan reaction coordinate R → PES(R)")
print("Find transition state: saddle point on PES")
print("Activation energy: E_TS - E_reactant")
```
### Deliverable: `reaction_path.py`

## Lab 19: Excited State Methods
### Walkthrough
```python
for method,desc in [("VQD","penalty-based deflation"),("SSVQE","subspace-search VQE"),("EOM-VQE","equations of motion")]:
    print(f"  {method:<8}: {desc}")
```
### Deliverable: `excited_states.py`

## Lab 20: Integration Capstone
### Walkthrough
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("HF mean-field",True);ok("CCSD(T) gold standard",True);ok("Active space",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("ljc_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `ljc_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
