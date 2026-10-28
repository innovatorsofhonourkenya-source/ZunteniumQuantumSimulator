# ZQL-07 Step 1 Notes — Levine (Quantum Chemistry) + Jensen (Computational Chemistry) + Cao 2019 Review

---

# A. Ira N. Levine — *Quantum Chemistry* (5th–7th ed.)

## Electronic Structure Problem

The central problem: find the electronic ground state of a molecule at fixed nuclear positions.

### Born-Oppenheimer Approximation
- Electrons move much faster than nuclei (mₑ/Mₙ ≈ 1/1836).
- Decouple electronic and nuclear motion: solve electronic Schrödinger equation at fixed nuclear geometry.
- Result: potential energy surface E(R) as function of nuclear coordinates R.

### Electronic Hamiltonian (in atomic units)
```
H_el = -½Σᵢ∇ᵢ² - ΣᵢΣₐ Zₐ/rᵢₐ + Σᵢ<ⱼ 1/rᵢⱼ
       ═══════════   ═══════════════   ═══════════
       T_e (kinetic)   V_ne (nuclear)  V_ee (repulsion)
```
- i,j: electrons; a: nuclei; Zₐ: nuclear charge.
- The V_ee term (electron-electron repulsion) makes the problem hard — no closed form.

### Slater Determinants (Antisymmetry)
- Electrons are fermions — the wavefunction must be antisymmetric: ψ(...,i,...,j,...) = −ψ(...,j,...,i,...).
- **Slater determinant**: |SD⟩ = (n!)^{-1/2} det[φ₁(x₁) ... φₙ(xₙ)].
- One-electron functions φᵢ(x): spin-orbitals (spatial × spin).

## Hartree-Fock (HF) Methods
- Approximate ground state with a *single* Slater determinant.
- Self-consistent field (SCF): iteratively optimise orbital coefficients.
- Error: "correlation energy" = E_exact − E_HF (HF misses electron correlation).
- **[VQE context]** VQE can recover correlation energy that HF misses.

---

# B. Frank Jensen — *Introduction to Computational Chemistry* (3rd ed., Wiley)

## Post-HF Methods (capturing correlation)
- **Configuration Interaction (CI)**: expand wavefunction in many Slater determinants.
  - Full CI (FCI): exact within a given basis, but exponentially expensive O(2^n basis functions).
  - **[Context]** FCI is what VQE is competing with; VQE can in principle recover FCI on a quantum computer polynomially.
- **Coupled Cluster (CCSD, CCSD(T))**: "gold standard" of quantum chemistry. O(N⁶), O(N⁷).
- **Density Functional Theory (DFT)**: trade wavefunction for electron density; O(N³). Approximations limit accuracy.

## Basis Sets
- Basis functions {χᵢ}: set of functions expressing molecular orbitals.
- STO-3G (minimal): crude but fast. 6-31G, cc-pVDZ, cc-pVTZ: increasingly accurate/expensive.
- **[SDK]** H₂ in STO-3G gives a 4-dimensional Fock space → 2-qubit problem via Jordan-Wigner.

## Second Quantisation
- Create/annihilate operators: aₚ† (create electron in orbital p), aₚ (annihilate).
- N-electron states: aₚ†aᵧ†....|vacuum⟩.
- Electronic Hamiltonian in second quantisation:
  ```
  H = Σₚᵧ hₚᵧ aₚ†aᵧ + ½Σₚᵧᵣₛ (pq|rs) aₚ†aᵧ†aₛaᵣ
  ```
  where hₚᵧ = one-electron integrals, (pq|rs) = two-electron integrals.

---

# C. Cao et al. — *Quantum Chemistry in the Age of Quantum Computing* (Chem. Rev., 2019)

## The Quantum Chemistry Advantage
| Method | Classical cost | Quantum cost | When better |
|---|---|---|---|
| FCI | O(N! or exp) | O(poly N) via VQE | Strongly correlated |
| CCSD(T) | O(N⁷) | O(N²-³) via Trotter | Weakly correlated |
| DFT | O(N³) | Comparable | Not clear |

## Qubit Encodings for Fermionic Systems

### Jordan-Wigner Transform (JW)
Maps fermionic operators to Pauli strings:
```
aₚ† = (Π_{j<p} Zⱼ) ⊗ (X_p - iY_p)/2
aₚ  = (Π_{j<p} Zⱼ) ⊗ (X_p + iY_p)/2
```
- JW maps n spin-orbitals → n qubits.
- A single occupation number is encoded locally.
- ZZ string grows with qubit index → "non-local" encoding.
- **[SDK]** Pauli terms in JW-encoded H₂: II, IZ, ZI, ZZ, XX, YY (6 terms for minimal basis).

### Bravyi-Kitaev Transform (BK)
- Encodes both occupation and parity information more compactly.
- Most Pauli strings have O(log n) weight (vs O(n) for JW).
- For large molecules: BK encoding reduces circuit depth.

## VQE Hardware Requirements
| Molecule | Qubits (JW) | Pauli terms | Gate depth (est.) |
|---|---|---|---|
| H₂ (STO-3G) | 4 (2 with symmetry) | 6 | ~10 |
| LiH (STO-3G) | 12 | 631 | ~100 |
| BeH₂ (STO-3G) | 14 | 666 | ~150 |
| FeMoco (active space) | 54–100 | ~100k | Fault-tolerant |

## Trotterization (Hamiltonian Simulation)
For time-evolving a Hamiltonian H = ΣᵢHᵢ (non-commuting terms):
```
e^{-iHt} ≈ (Πᵢ e^{-iHᵢt/n})^n   [Lie-Trotter, n→∞]
```
Higher-order Suzuki-Trotter: e^{-iH₁t}e^{-iH₂t}e^{-iH₂t}e^{-iH₁t} (2nd order symmetric).
**[SDK]** Each e^{-iHᵢt}: if Hᵢ is a Pauli string, use `qc.rx/ry/rz` or `qc.rzz`.

## UCCSD Ansatz
Unitary Coupled Cluster Singles and Doubles:
```
|ψ_UCCSD⟩ = exp(T₁ + T₂ - T₁† - T₂†)|HF⟩
```
where T₁: single excitations (1e- from occupied to virtual), T₂: double excitations.
- **[SDK]** UCCSD → Pauli decomposition → parameterised Rz/CNOT circuit.
- n parameters = number of (single + double) excitation amplitudes.
