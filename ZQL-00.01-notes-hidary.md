# ZQL-00.01 Step 1 Notes — Hidary, *Quantum Computing: An Applied Approach* (2nd ed., 2021)

**Mapped to QuFlow SDK components. Marks: [SDK] = direct SDK concept | [ANALOGY] = teaching only**

---

## Chapter 1 — Why Quantum Now?
- NISQ era: 50–1000 noisy qubits. Not fault-tolerant but useful for near-term algorithms.
- *Three pillars*: superposition → parallel amplitude manipulation; entanglement → correlated multi-qubit states; interference → amplify right answers, cancel wrong ones.
- SDK relevance: **[SDK]** `StatevectorSimulator` directly simulates these three phenomena classically up to ~25 qubits.

## Chapter 2 — Qubits and Gates
- Qubit state: **|ψ⟩ = α|0⟩ + β|1⟩**, |α|² + |β|² = 1.
- Bloch sphere: north pole = |0⟩, south pole = |1⟩, equator = maximum superposition.
- Gates = unitary matrices U (U†U = I). Quantum gates are always reversible.
- Key single-qubit gates:
  - H (Hadamard): |0⟩ → (|0⟩+|1⟩)/√2 — creates equal superposition
  - X (Pauli-X): classical NOT, |0⟩↔|1⟩
  - Z (Pauli-Z): phase flip, |1⟩ → −|1⟩
  - Rx/Ry/Rz: rotations on Bloch sphere by angle θ
- **[SDK]** QuFlow: `qc.h(0)`, `qc.x(0)`, `qc.z(0)`, `qc.rx(theta, 0)` — each maps to a gate object (HGate, XGate, …) stored as `Instruction`.

## Chapter 3 — Multi-Qubit Systems
- Tensor product: 2-qubit state space = ℂ² ⊗ ℂ² = ℂ⁴.
- State vector for n qubits = 2ⁿ complex amplitudes — this is the exponential memory scaling.
- CNOT (CX): control + target. If control=|1⟩ → flip target. Entangles qubits.
- Bell state: H(0), CNOT(0,1) → (|00⟩+|11⟩)/√2. Cannot be written as product of single-qubit states.
- **[SDK]** `qc.cx(0, 1)` — `CXGate()`, stored as 2-qubit `Instruction(qubits=[0,1])`.

## Chapter 4 — Circuit Model & SDK Patterns
- Hidary uses Cirq/Qiskit; identical *conceptual* patterns to QuFlow:
  - Create circuit → add gates → measure → run on backend → get result counts
- **[SDK]** Direct mapping:
  - `cirq.Circuit()` → `QuantumCircuit(n)`
  - `circuit.append(gate)` → `qc.h(0).cx(0,1)` (fluent)
  - `simulator.simulate(circuit)` → `backend.run(qc, shots=1024)`
- Parameters (VQE/QAOA): Hidary shows parametrized gates. **[SDK]** `qc.ry(theta, 0)` + `qc.bind_parameters({"theta": 1.5})`.

## Chapter 5 — Running on Simulators
- Statevector sim: exact amplitude tracking, zero noise. Good for correctness testing.
- Measurement sampling: collapse → probabilistic counts. Run multiple shots for statistics.
- **[SDK]** `StatevectorSimulator` (alias `StatevectorBackendRunner`) wraps `get_backend("statevector")`.
  - `backend.run(qc, shots=1024)` → result object with `.counts` dict.

## Chapter 12 — Hardware Backends (IBM, IonQ, Braket, Rigetti)
- Hardware error sources: gate errors, readout errors, decoherence (T1/T2).
- Backend routing: qubits on chip are not all-to-all connected; need SWAP insertion.
- Basis translation: hardware only supports a native gate set (e.g., CX + U3).
- **[SDK]** `HardwareAwareTranspiler` handles routing + decomposition. `IBMBackend` / MHOS adapters map to hardware APIs.

---

## Concept→SDK Mapping Table (Hidary)

| Hidary Concept | QuFlow Class/Method | Teaching or Real? |
|---|---|---|
| Qubit state vector | `StatevectorSimulator` internal state | [SDK] |
| Gate = unitary matrix | `BaseGate.matrix()` | [SDK] |
| Circuit = sequence of gates | `QuantumCircuit._instructions` | [SDK] |
| Measurement = collapse | `qc.measure(q, c)` + `qc.measure_all()` | [SDK] |
| Simulator run | `backend.run(qc, shots=N)` | [SDK] |
| Parameter binding | `qc.bind_parameters(params)` | [SDK] |
| Hardware backend | `IBMBackend`, MHOS adapters | [SDK] |
| Transpiler (routing + decomp) | `HardwareAwareTranspiler` | [SDK] |
