# ZQL-03 Step 1 Notes — Mano & Kime (Classical Logic) + Maslov & Dueck (Reversible Synthesis)

---

# A. Mano & Kime — *Logic and Computer Design Fundamentals*

**Baseline classical logic — foundation for understanding quantum equivalents.**

## Boolean Algebra (Ch. 2)
- Two values: 0 and 1.
- Operations: AND (·), OR (+), NOT (¬).
- XOR: A⊕B = A·B̄ + Ā·B = "either but not both".
- De Morgan: ¬(A·B) = Ā+B̄; ¬(A+B) = Ā·B̄.
- NAND/NOR are universal for classical computing (but NOT reversible).

## Half-Adder (Ch. 4)
- Two inputs (A, B), two outputs (Sum, Carry):
  - Sum = A ⊕ B
  - Carry = A · B
- Truth table:
  ```
  A B | Sum Carry
  0 0 |  0    0
  0 1 |  1    0
  1 0 |  1    0
  1 1 |  0    1
  ```
- **[SDK contrast]** Classical: 2 inputs → 2 outputs (not reversible — can't recover A,B from Sum,Carry alone). Quantum half-adder: 3 qubits → 3 qubits, keeps A unchanged.

## Full-Adder (Ch. 4)
- Three inputs (A, B, Cᵢₙ), two outputs (Sum, Cₒᵤₜ):
  - Sum = A ⊕ B ⊕ Cᵢₙ
  - Cₒᵤₜ = A·B + Cᵢₙ·(A⊕B)
- Ripple-carry adder: chain n full-adders for n-bit addition. O(n) depth.

## Multiplexer and Decoder (Ch. 5)
- Multiplexer (MUX): selects one of 2ⁿ inputs with n select bits.
- **[ANALOGY]** Quantum multiplexing uses superposition of the select register — evaluates all branches simultaneously.

## Flip-Flops and Sequential Logic (Ch. 6–7)
- Classical memory requires bistable elements (flip-flops). State is persistent.
- **[Contrast]** Quantum "memory" = qubit state — collapses on measurement. Quantum RAM (QRAM) is an open research problem.

---

## Classical vs Quantum Logic: Summary Table

| Logic Element | Classical | Quantum | Reversible? |
|---|---|---|---|
| NOT | NOT gate | X gate | Both ✓ |
| AND | AND gate | No direct equivalent | Classical: ✗ |
| AND (reversible) | — | Toffoli (CCX) | ✓ |
| XOR | XOR circuit | CNOT | ✓ |
| NAND | NAND gate | Toffoli + X | ✓ (quantum) |
| FANOUT | Wire split | Forbidden (no-cloning) | N/A |
| Half-adder | 2→2 (irreversible) | 3→3 (reversible) | Quantum ✓ |
| Full-adder | 3→2 (irreversible) | 4→4 (reversible) | Quantum ✓ |

---

# B. Maslov & Dueck — *Reversible Logic Synthesis* (2003 paper)

**Academic foundation for constructing reversible Boolean circuits.**

## Core Problem
- Given a Boolean function f: {0,1}ⁿ → {0,1}ⁿ (bijective), synthesise a reversible circuit using only:
  - NOT (1-qubit)
  - CNOT (2-qubit)
  - Toffoli CCX (3-qubit)
- This gate set {NOT, CNOT, Toffoli} is called the NCT library.

## Synthesis Methods
- **Truth table approach**: enumerate all 2ⁿ rows, greedily match each output row to a gate sequence.
- **Permutation approach**: represent the function as a permutation of {0,1}ⁿ; decompose the permutation into transpositions (each = a controlled-NOT variant).
- **Cost metric**: "quantum cost" = number of CNOT and single-qubit gates in the realisation.

## Ancilla Lines
- Extra |0⟩ initialised lines (ancillas) reduce the circuit cost but increase qubit count.
- Trade-off: fewer ancillas → deeper circuit; more ancillas → shallower but wider circuit.
- **[SDK]** Every ancilla qubit costs one more `QuantumCircuit(n_logical + n_ancilla)` qubit. Ancillas must be reset (uncomputed) to |0⟩ before reuse.

## Key Results
- Any reversible function on n bits can be synthesised with at most O(n·2ⁿ) NCT gates.
- Optimal synthesis is NP-hard for large n → practical compilers use heuristics.
- **[SDK implication]** The `KAKDecompositionPass` + `ConsolidateBlocksPass` in QuFlow perform exactly this: decompose multi-qubit unitaries into CNOT + single-qubit gates efficiently.

## Garbage Bits Problem
- Naïve synthesis produces extra output lines ("garbage") that are entangled with the result.
- Solution: **Bennett's trick** — compute f(x) into output register, copy result, uncompute f(x) to reset ancillas. Cost: ~2× gates.
- **[SDK]** `qc.inverse()` implements the uncomputation: run the sub-circuit backwards to clean ancillas.

## Relevance to QuFlow
| Maslov/Dueck Concept | QuFlow Element |
|---|---|
| NCT gate library | `qc.x()`, `qc.cx()`, `qc.ccx()` |
| Ancilla line | Extra qubits in `QuantumCircuit(n + n_ancilla)` |
| Garbage bit | Entangled ancilla qubit not uncomputed |
| Bennett uncomputation | `qc.inverse()` on sub-circuit |
| Quantum cost | `qc.num_gates` and `qc.depth` |
| Permutation decomposition | `KAKDecompositionPass` (2-qubit blocks) |
