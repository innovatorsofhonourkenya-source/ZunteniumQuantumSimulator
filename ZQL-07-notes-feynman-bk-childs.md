# ZQL-07 Step 1 Notes — Feynman 1982 + Bravyi-Kitaev 2002 + Childs Trotter Review 2021

---

# A. Feynman 1982 — *Simulating Physics with Computers* (Int. J. Theor. Phys.)

**The foundational paper motivating quantum simulation.**

## Core Argument
- "Nature isn't classical, dammit." — Feynman's observation: simulating quantum systems classically requires exponential resources.
- For n interacting quantum particles: 2ⁿ amplitudes → classical simulation costs ~2ⁿ operations.
- Feynman's conjecture: a quantum computer (a "universal quantum simulator") can simulate quantum physics in polynomial time.

## Implications for Quantum Chemistry
- Molecules are quantum systems → FCI is exponentially hard classically.
- A quantum computer can represent the molecular wavefunction in n qubits (one per spin-orbital).
- Gate operations on qubits = operations in Fock space.
- **[SDK]** Running VQE on QuFlow to find H₂ ground state is a realisation of Feynman's vision.

## "Computational Primitive" Insight
- Feynman identified that any local interaction Hamiltonian H can be simulated by a quantum circuit.
- This is the foundation of Trotterised Hamiltonian simulation in ZQL-07.

---

# B. Bravyi & Kitaev 2002 — *Fermionic Quantum Computation* (arXiv:quant-ph/0003137)

**Primary reference for the Bravyi-Kitaev (BK) encoding.**

## The Jordan-Wigner Problem  
- JW: encode occupation number nₚ directly in qubit p.
- Z-string aₚ† = Z₀Z₁...Zₚ₋₁(Xₚ-iYₚ)/2 → Pauli weight O(p) (grows with qubit index).
- For large molecules: JW operators have O(n) weight → O(n) CNOT gates per term.

## Bravyi-Kitaev Encoding
- Splits occupation and parity information into two registers.
- Occupation bits: stored directly (like JW).
- Parity bits: partial sums stored for efficient parity computation.
- Result: operators have weight O(log n), not O(n).

## Scaling Comparison
| Encoding | Operator weight | CNOT gates per Pauli term | Best for |
|---|---|---|---|
| Jordan-Wigner | O(n) | O(n) | Few qubits, simple implementation |
| Bravyi-Kitaev | O(log n) | O(log n) | Large molecules, deep circuits |
| BKSF (BK superfast) | O(1) | O(1) | Lattice Hamiltonians only |

## **[SDK]** Encoding Choice
- For H₂ (2 qubits after symmetry reduction): JW and BK are equivalent.
- For LiH (10+ qubits): BK reduces operator weight from O(10) to O(log 10 ≈ 3).
- QuFlow's `PauliHamiltonian` stores pre-encoded Pauli strings — the encoding choice happens in the chemistry pre-processing layer.

---

# C. Childs et al. — *Theory of Trotter Error with Commutator Scaling* (Theory of Computing, 2021)

**Comprehensive treatment of Trotterised Hamiltonian simulation.**

## First-Order Trotter
```
e^{-it(H₁+H₂)} ≈ e^{-itH₁} e^{-itH₂}   [error = O(t²‖[H₁,H₂]‖)]
```

## Second-Order Symmetric Trotter (Suzuki)
```
e^{-it(H₁+H₂)} ≈ e^{-itH₁/2} e^{-itH₂} e^{-itH₁/2}   [error = O(t³‖[H₁,[H₁,H₂]]‖)]
```

Higher-order Suzuki-Trotter: 2k-th order has error O(t^{2k+1}).

## Commutator Scaling Insight (Childs' contribution)
- Error depends on **commutator norms** ‖[Hᵢ, Hⱼ]‖, not just ‖H‖.
- For molecular Hamiltonians: commutators are small → fewer Trotter steps needed than naive analysis suggests.
- Practical result: LiH Hamiltonian simulation needs O(10²) Trotter steps for chemical accuracy, not O(10⁶).

## Gate Cost of Trotter Steps
Each e^{-iθPᵢ} for Pauli string Pᵢ of weight w: O(w) CNOT gates.
```
e^{-iθ X₀Y₁Z₂} = [H₀][S†₁H₁][H₂] · CX(0,1) · CX(1,2) · Rz(2θ, 2) · CX(1,2) · CX(0,1) · [H₀][H₁S₁][H₂]
```

**[SDK]** Gate decomposition for arbitrary Pauli exponential:
```python
def pauli_exponential(pauli_str: str, theta: float) -> QuantumCircuit:
    """
    Build circuit for exp(-iθ P) where P is a Pauli string.
    e^{-iθ P} requires:
      - Basis change: H for X, S†·H for Y, nothing for Z
      - CNOT ladder (control = qubit n-1, target = qubit n)
      - Rz(2θ) on last qubit  
      - Reverse CNOT + reverse basis change
    """
    n = len(pauli_str)
    qc = QuantumCircuit(n, name=f"exp_i{theta:.2f}_{pauli_str}")

    # Basis rotation: X → Z basis; Y → Z basis
    for i, pauli in enumerate(pauli_str):
        if pauli == 'X': qc.h(i)
        elif pauli == 'Y': qc.sdg(i); qc.h(i)

    # CNOT ladder (build parity on last active qubit)
    active = [i for i, p in enumerate(pauli_str) if p != 'I']
    for k in range(len(active) - 1):
        qc.cx(active[k], active[k+1])

    # Rotation
    if active:
        qc.rz(2 * theta, active[-1])

    # Undo CNOT ladder
    for k in reversed(range(len(active) - 1)):
        qc.cx(active[k], active[k+1])

    # Undo basis rotation
    for i, pauli in enumerate(pauli_str):
        if pauli == 'X': qc.h(i)
        elif pauli == 'Y': qc.h(i); qc.s(i)

    return qc
```
