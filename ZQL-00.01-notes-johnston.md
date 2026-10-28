# ZQL-00.01 Step 1 Notes — Johnston, Harrigan & Gimeno-Segovia, *Programming Quantum Computers* (O'Reilly, 2019)

**Mapped to QuFlow SDK components.**

---

## Part I — Quantum Concepts for Developers

### Ch. 1 — The Qubit
- Author frame: "a qubit is a register that can hold *amplitudes*, not bits."
- Amplitude: a complex number. Probability = |amplitude|². The ±1 sign is the secret weapon (interference).
- Reading out: you can read it, but you collapse it. You cannot clone it.
- **[SDK]** Measurement side-effect: `qc.measure_all()` — after measurement, qubit is in a definite classical state. The `statevector` backend can report the full amplitude vector *before* measurement via `ExecutionResult`.

### Ch. 2 — Your First Quantum Program: Random Number Generator
- H gate creates 50/50 superposition. Measuring gives random 0 or 1.
- Key mental model: the circuit is a *recipe*; running it is the *experiment*.
- **[SDK]** Exact QuFlow translation:
  ```python
  from zuntenium import QuantumCircuit, StatevectorSimulator
  qc = QuantumCircuit(1)
  qc.h(0).measure_all()
  backend = StatevectorSimulator()
  result = backend.run(qc, shots=100)
  # result.counts = {"0": ~50, "1": ~50}
  ```

### Ch. 3 — Phase Kickback
- Phase: Z, S, T gates add a phase rotation to |1⟩. Globally invisible; relatively detectable (interference).
- Phase kickback: controlled gate with eigenstate target → kicks phase back to control.
- **[SDK]** `qc.s(0)` (S gate = √Z), `qc.t(0)` (T gate = ⁴√Z), `qc.cz(0,1)` (symmetric phase).
- **[ANALOGY]** Phase kickback is exploited in QPE and Grover (ZQL-04); here it's a mental model.

### Ch. 4 — Circuits with Entanglement
- Bell state creation: H(0) → CX(0,1). Now the two qubits are correlated regardless of distance.
- SWAP via 3 CNOTs: SWAP(a,b) = CX(a,b); CX(b,a); CX(a,b)
- **[SDK]** `qc.swap(0,1)` — maps to `SWAPGate()`. Internally the transpiler decomposes to 3 CX on devices that lack native SWAP.

### Ch. 7 — Amplitude Amplification (Grover)
- The essence: flip the amplitude of the target state negative (oracle), then reflect about the average (diffusion). Repeat π/(4 sin⁻¹(1/√N)) ≈ π√N/4 times.
- **[ANALOGY]** Full Grover is ZQL-04 content. For ZQL-00 the teaching point is: quantum circuits compute by *cancellation*, not exhaustive search.

---

## Hands-On Patterns (Johnston)

| Pattern | Book Example | QuFlow Idiom |
|---|---|---|
| Full superposition | H on every qubit | `for q in range(n): qc.h(q)` |
| Phase flip | Z gate | `qc.z(q)` |
| Create entanglement | H + CX | `qc.h(0).cx(0, 1)` |
| Run and count | `qpu.run(prog, n_samples=1000)` | `backend.run(qc, shots=1000)` |
| Chain multiple gates | Fluent API | `qc.h(0).x(1).cx(0,1).measure_all()` |

---

## Key Developer Mental Models

1. **"Amplitude not probability"** — until you measure, the qubit holds complex amplitudes. You manipulate *amplitudes* with gates.
2. **"Interference is the algorithm"** — correct answers interfere constructively; wrong answers cancel.
3. **"Measure late"** — measure only when you need to read out. Premature measurement collapses the superposition.
4. **[TO CONFIRM]** QuFlow's `ExecutionResult` — does it expose `.statevector` for pre-measurement amplitude inspection?
