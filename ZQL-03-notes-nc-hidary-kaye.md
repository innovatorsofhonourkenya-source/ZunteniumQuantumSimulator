# ZQL-03 Step 1 Notes — Nielsen & Chuang Ch.3 + Hidary (Quantum Arithmetic) + Kaye Ch.5

---

# A. Nielsen & Chuang — Ch. 3: Quantum Circuits

## 3.1 Quantum Circuits and Single-Qubit Operations
- Circuit model: wires = qubit state, gates = unitaries, time flows left to right.
- Any single-qubit U ∈ SU(2) can be written U = Rz(β)·Ry(γ)·Rz(δ) (Euler decompositions).
- **[SDK]** `qc.u(theta, phi, lam, q)` → `U3Gate(θ,φ,λ)` = general single-qubit decomposition.

## 3.2 Controlled Operations
- Controlled-U: applies U to target if control = |1⟩; identity otherwise.
- Multi-controlled gates reduce to single-qubit + CNOT network via decomposition.
- Toffoli (CCX): doubly-controlled X. IF c0=1 AND c1=1 THEN flip target.
  - Universal for classical reversible computation.
  - Matrix: 8×8, flips |111⟩ ↔ |110⟩.
- **[SDK]** `qc.ccx(c0, c1, target)` → `ToffoliGate()`.
- Fredkin (CSWAP): controlled-SWAP. IF control=1 → swap t0, t1.
- **[SDK]** `qc.cswap(ctrl, t0, t1)` → `FredkinGate()`.

## 3.3 Reversibility
- **Reversible gate**: bijective function on basis states (no information loss).
- Every quantum gate is reversible (it's unitary). Every unitary has an inverse U†.
- Classical NOT is reversible; NAND/AND are NOT reversible (many-to-one).
- **[SDK]** `gate.dagger()` returns U†. `qc.inverse()` reverses and conjugates the whole circuit.

## 3.4 Universality
- {H, T, CNOT} is universal for all unitaries.
- Toffoli alone is universal for *classical* reversible computation.
- CNOT = classical reversible XOR: |a, b⟩ → |a, a⊕b⟩.
- **[SDK]** `qc.cx(a, b)` computes XOR in-place on bit b, keeping bit a.

## 3.5 Measurement and Deferred Measurement Principle
- "Principle of deferred measurement": measurements can be moved to end of circuit.
- Classical control-flow can be replaced by quantum controlled gates.
- **[SDK]** Full mid-circuit measurement: `qc.measure(q, c)` followed by more gates. In v0.2.0, measure gates should come after all computation for simulator correctness.

---

# B. Hidary — Quantum Arithmetic Chapters

## Quantum Ripple-Carry Adder (Cuccaro-Hogg approach)
- Add two n-bit numbers A and B into B in-place, with ancilla carry register.
- Building blocks: Toffoli (carry propagation) and CNOT (XOR sum bit).
- Key insight: no ancilla garbage qubits if uncomputation is performed after extraction.
- **[SDK]** All primitives available: `qc.ccx(c0, c1, t)` (Toffoli), `qc.cx(a,b)` (XOR/sum).

## Phase Kickback Arithmetic
- Apply oracle Uf: |x⟩|y⟩ → |x⟩|y⊕f(x)⟩ with y = |−⟩ → phase kickback.
- Used in Grover and quantum search — f is implemented as arithmetic on qubits.

## Quantum-Classical Comparison
| Operation | Classical Gate | Quantum Gate | QuFlow |
|---|---|---|---|
| NOT | NOT (1→0, 0→1) | X gate | `qc.x(q)` |
| XOR | XOR (a,b→a⊕b) | CNOT | `qc.cx(a,b)` |
| AND (reversible) | Toffoli | Toffoli (CCX) | `qc.ccx(c0,c1,t)` |
| FANOUT | — | Forbidden (no-cloning) | N/A |
| Copy | — | Forbidden (no-cloning) | N/A |

---

# C. Kaye, Laflamme & Mosca — Ch. 5: Quantum Arithmetic Primitives

## 5.1 The CNOT as Reversible XOR
- CNOT gate: |a,b⟩ → |a, a⊕b⟩.
- XOR is reversible: applied twice gives identity. CNOT² = I.
- **[SDK]** `qc.cx(a, b)` two times on same qubits = no-op (identity).

## 5.2 Half-Adder
- Classical: sum = A XOR B; carry = A AND B.
- Quantum (reversible, no ancilla loss):
  - Requires 3 qubits: |A⟩|B⟩|0⟩ → |A⟩|A⊕B⟩|A∧B⟩
  - Gate 1: Toffoli(A, B, c) — computes carry into c
  - Gate 2: CNOT(A, B) — computes sum into B
  - Result: B holds sum (A⊕B), c holds carry (A∧B), A unchanged.

## 5.3 Full-Adder
- Adds A, B, carry_in → sum, carry_out.
- Requires one Toffoli, two CNOTs.
- The quantum full-adder is the building block of all quantum arithmetic.

## 5.4 Ancilla Qubits and Uncomputation
- Ancilla: helper qubit initialised to |0⟩, used as scratch, must be reset to |0⟩ after.
- Uncomputation: run the computation backwards (dagger) to reset ancillas to |0⟩.
- **[SDK]** `qc.reset(q)` force-resets a qubit to |0⟩ (physically; not unitary).
  True uncomputation uses `qc.inverse()` to produce the conjugate sub-circuit.

## 5.5 Resource Counting
- Toffoli gate count dominates in reversible arithmetic.
- n-bit adder: O(n) Toffoli + O(n) CNOT + O(n) ancilla.
- **[SDK]** `qc.num_gates` and `qc.depth` track these resources.
