# ZQL-00.01 — Quantum Foundations: Introduction to Quantum SDK & Developer Onboarding

> **Course ID**: ZQL-00.01 | **Track**: ZQL-00 Start Here  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Tone**: Clear, authoritative, practical — Zuntenium design system

---

## 0. Course Overview

### Who This Is For

You are a **Python developer** with no prior quantum computing experience. You may have heard of qubits or quantum computers in the news. This course gets you from zero to running your first circuits on a simulator in under 90 minutes.

**Prerequisites**:
- Python 3.9+ installed
- Comfortable with `pip install`, `import`, and basic Python functions
- No linear algebra or physics required — we introduce what you need as you need it

### Learning Outcomes

By the end of this course you will:

1. Explain the three core quantum phenomena (superposition, entanglement, interference) in developer terms
2. Install and configure the QuFlow SDK (`zuntenium`) on your machine
3. Build a Bell-state circuit using the `QuantumCircuit` fluent API
4. Run circuits on the `StatevectorSimulator` and read measurement results
5. Understand the QuFlow architecture: what each component does
6. Know which course to take next

### Estimated Time

| Section | Time |
|---|---|
| §1–2: Why quantum + Foundations | 20 min |
| §3: SDK Installation | 15 min |
| §4: Architecture tour | 15 min |
| §5: First circuit + run | 20 min |
| §6–7: Class model + tooling | 10 min |
| §8 Troubleshooting + §9 Capstone | 20 min |
| **Total** | **~90 minutes** |

### Source Books Used
| # | Book | Chapters Used |
|---|---|---|
| H | Jack Hidary — *Quantum Computing: An Applied Approach* (2nd ed., 2021) | 1–5, 12 |
| J | Johnston, Harrigan & Gimeno-Segovia — *Programming Quantum Computers* (O'Reilly, 2019) | 1–4, 7 |
| N | Nakahara & Ohmi — *Quantum Computing: From Linear Algebra to Physical Realizations* (2008) | 1–2 |
| Q | IBM Quantum Team — *Qiskit Textbook* (free, qiskit.org) | Ch. 1 |

---

## 1. Why a Quantum SDK: The Problem QuFlow Solves

### 1.1 The Quantum Computing Landscape Today

We are in the **NISQ era** (Noisy Intermediate-Scale Quantum). Today's quantum processors have 50–1000 qubits, but every operation introduces a small error. This is not a defect — it is physics. Classical bits are either 0 or 1. Quantum bits (qubits) exist in superposition *until measured*, enabling algorithms that are exponentially expensive on classical hardware but efficient on quantum hardware.

**The problem**: quantum computers require programming at the gate level, in hardware-specific instruction sets. Every device from IBM, IonQ, Rigetti, and Braket has different:
- Native gate sets (IBM uses CX+U3; IonQ uses Mølmer-Sørensen gates)
- Connectivity maps (which qubits can interact directly)
- Calibration data (noise levels change daily)

Writing quantum programs directly in these native formats would be like programming a GPU by hand-writing PTX assembly for every vendor.

### 1.2 What QuFlow Does

**QuFlow** (package `zuntenium`) is an **AI-native hybrid quantum computing SDK**. It sits between your Python code and the hardware:

```
Your Python code
      ↓   QuantumCircuit (SDK)
QuFlow Transpiler Engine v2
      ↓   routing + gate decomposition
MHOS Hardware Orchestration
      ↓   IBM / IonQ / Braket / Rigetti
      Real QPU or Simulator
```

**What you write**:
```python
qc = QuantumCircuit(2)
qc.h(0).cx(0, 1).measure_all()
```

**What QuFlow does for you**:
- Validates the circuit (no invalid qubit indices, no illegal gate combinations)
- Compiles to an efficient Intermediate Representation (IR)
- Routes the circuit onto the device's connectivity map (SABRE algorithm)
- Decomposes gates into the hardware's native gate set (KAK decomposition)
- Submits the job and returns normalized results

> **Key Takeaway**: You write hardware-agnostic circuits. QuFlow handles the rest.

---

## 2. Quantum Foundations for Developers

> *"A qubit is a register that can hold amplitudes, not bits."*
> — Johnston, Harrigan & Gimeno-Segovia

### 2.1 The Qubit: Amplitude, Not Probability

A classical bit holds 0 or 1. A qubit holds two **complex amplitudes** α and β:

```
|ψ⟩ = α|0⟩ + β|1⟩
```

The rules:
- |α|² + |β|² = 1 (amplitudes are normalized — total probability = 1)
- α and β are *complex numbers*, not just real numbers
- **You cannot read them directly** — measuring the qubit gives 0 with probability |α|² or 1 with probability |β|²

Why complex numbers? Because they allow *interference* — the key computational primitive.

> **Note**: The `StatevectorSimulator` *does* let you see the full amplitude vector before measurement. In real hardware you cannot — you can only observe the measurement statistics.

### 2.2 Three Phenomena, Three Lines of Code

**Superposition** — the qubit holds both possibilities simultaneously:
```python
qc = QuantumCircuit(1)
qc.h(0)          # Hadamard gate: |0⟩ → (|0⟩ + |1⟩)/√2
qc.measure_all()
# After many shots: ~50% "0", ~50% "1"
```

**Entanglement** — two qubits whose states cannot be described independently:
```python
qc = QuantumCircuit(2)
qc.h(0)          # Superpose qubit 0
qc.cx(0, 1)      # CNOT: if qubit 0 = |1⟩, flip qubit 1
qc.measure_all()
# Results are ALWAYS correlated: "00" or "11", never "01" or "10"
```

**Interference** — amplitudes cancel or reinforce to bias the outcome:
```python
qc = QuantumCircuit(1)
qc.h(0)          # |0⟩ → (|0⟩ + |1⟩)/√2
qc.z(0)          # Z flips the phase of |1⟩: → (|0⟩ - |1⟩)/√2
qc.h(0)          # H again → |1⟩ (100% deterministic!)
qc.measure_all()
# Always "1" — interference amplified the |1⟩ amplitude
```

That last example shows how quantum algorithms work: **gates steer amplitudes so the correct answer interferes constructively** and the wrong answers cancel.

### 2.3 Gates Are Matrices (But You Don't Need to Know This to Use QuFlow)

Every gate is a unitary matrix U. "Unitary" means U†U = I — it preserves the total probability. Here are the most common ones:

| Gate | QuFlow call | What it does |
|---|---|---|
| **H** (Hadamard) | `qc.h(q)` | Equal superposition of \|0⟩ and \|1⟩ |
| **X** (Pauli-X) | `qc.x(q)` | Quantum NOT: \|0⟩↔\|1⟩ |
| **Z** (Pauli-Z) | `qc.z(q)` | Phase flip: \|1⟩ → −\|1⟩ |
| **S** | `qc.s(q)` | Quarter-turn phase (√Z) |
| **T** | `qc.t(q)` | Eighth-turn phase (⁴√Z) |
| **Rx(θ)** | `qc.rx(theta, q)` | Rotate on Bloch sphere by θ around X axis |
| **Ry(θ)** | `qc.ry(theta, q)` | Rotate around Y axis |
| **Rz(θ)** | `qc.rz(theta, q)` | Rotate around Z axis |
| **CX / CNOT** | `qc.cx(ctrl, tgt)` | Flip target if control=\|1⟩ |
| **CZ** | `qc.cz(ctrl, tgt)` | Phase-flip target if both=\|1⟩ |
| **SWAP** | `qc.swap(q1, q2)` | Exchange two qubit states |
| **Toffoli (CCX)** | `qc.ccx(c0, c1, tgt)` | Flip target if both controls=\|1⟩ |

> **Tip**: You never need to write the matrix yourself. QuFlow's gate objects (`HGate`, `CXGate`, …) carry their own `.matrix()` method.

### 2.4 Measurement

Measuring collapses the superposition:
- Before measure: `|ψ⟩ = α|0⟩ + β|1⟩`
- After measure: see 0 (prob |α|²) or 1 (prob |β|²); state collapses to that basis state

```python
qc.measure(qubit=0, clbit=0)   # Measure q[0] into classical bit c[0]
qc.measure_all()               # Shorthand: measure every qubit
```

**Rule**: Measure late. Every measurement collapses what you measured. Only measure when you need the classical result.

### 2.5 Memory Scaling (Why Quantum Is Hard to Simulate)

An n-qubit system requires 2ⁿ complex amplitudes:
- 1 qubit: 2 amplitudes
- 10 qubits: 1024 amplitudes ← easy
- 30 qubits: >1 billion amplitudes ← ~8 GB RAM
- 50 qubits: >1 quadrillion amplitudes ← impossible classically

This is why we need real quantum hardware for large circuits, and why simulators are limited to ~25–30 qubits.

> **Key Takeaways §2**:
> - Qubits hold complex amplitudes; measurement gives probabilistic outcomes
> - Gates are unitary transformations; interference is the algorithm
> - Measure as late as possible; measurement is irreversible
> - Classical simulation of n qubits costs O(2ⁿ) memory

---

## 3. Developer Onboarding: Install and Configure QuFlow

### 3.1 System Requirements

| Component | Minimum | Recommended |
|---|---|---|
| Python | 3.9 | 3.11 |
| RAM | 4 GB | 16 GB (for large simulation) |
| OS | macOS 12 / Ubuntu 22.04 / Windows 11 | macOS or Linux |
| GPU (optional) | — | CUDA 11.8+ for `GPUStatevectorSimulator` |

### 3.2 Installation

**Step 1 — Create a virtual environment** (strongly recommended):

```bash
# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate

# Windows (PowerShell)
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**Step 2 — Install QuFlow**:

```bash
pip install zuntenium
```

> **Note**: NumPy is a required dependency and will be installed automatically.

**Step 3 — Verify**:

```python
import zuntenium
print(zuntenium.__version__)   # Expected: 0.2.0
```

### 3.3 IDE Setup

**VS Code** (recommended):

1. Install the [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
2. Select your virtual environment: `Ctrl+Shift+P` → *Python: Select Interpreter* → choose `.venv`
3. Install the **Zuntenium VS Code extensions** (see ZQL-16):
   - `zuntenium-vscode` — code prediction
   - `zuntenium-workspace` — QCanvas circuit editor

**Jupyter Notebook / Colab**:

```bash
pip install zuntenium notebook
jupyter notebook
```

QuFlow circuits render with rich HTML in Jupyter via `qc._repr_html_()` (automatic in Jupyter cells).

### 3.4 First Smoke Test

```python
import zuntenium
from zuntenium import QuantumCircuit, StatevectorSimulator

# 1-qubit circuit: put qubit in superposition and measure
qc = QuantumCircuit(1)
qc.h(0).measure_all()

backend = StatevectorSimulator()
result = backend.run(qc, shots=1000)
print(result.counts)   # e.g. {"0": 498, "1": 502}
```

If this runs without error, your installation is complete.

---

## 4. QuFlow Architecture Tour

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Python Code                         │
│         qc = QuantumCircuit(n)  …  backend.run(qc)          │
└────────────────────┬────────────────────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │     Circuit Layer       │
        │  QuantumCircuit         │  Fluent builder
        │  QuantumRegister        │  Qubit naming
        │  Instruction            │  Gate + qubit + clbit
        │  CircuitValidator       │  Pre-flight checks
        └────────────┬────────────┘
                     │ .to_ir()
        ┌────────────▼────────────┐
        │   QuantumIR             │  Canonical IR (IRNode list)
        │   (transpiler/ir/)      │  SHA-256 template hash key
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │  Transpiler Engine v2   │
        │  VF2PlacementPass       │  Initial qubit placement
        │  SABRERouterV2          │  SWAP insertion for connectivity
        │  KAKDecompositionPass   │  Gate basis translation
        │  ALAPSchedulingPass     │  Depth minimisation
        │  TranspilerCache (LRU)  │  Template compile → cache hit
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │   MHOS — Hardware       │
        │   Orchestration System  │
        │  IBMBackend             │  IBM Quantum Network
        │  IonQAdapter            │  IonQ REST API v0.3
        │  BackendRegistry        │  Discovery + health probes
        │  JobContext             │  Deadline, retry policy
        └────────────┬────────────┘
                     │
        ┌────────────▼────────────┐
        │   Simulators            │
        │  StatevectorSimulator   │  Exact, noiseless (CPU)
        │  DensityMatrixSimulator │  Noise channels (Kraus ops)
        │  StabilizerSimulator    │  Clifford-only, O(n²)
        │  GPUStatevectorSimulator│  CuPy CUDA acceleration
        │  TensorNetworkBackend   │  MPS for shallow circuits
        └─────────────────────────┘
```

### 4.1 QuantumCircuit — The Builder

The entry point for all circuits. Uses a **fluent builder pattern** — every gate method returns `self` for chaining:

```python
qc = QuantumCircuit(3, name="my-circuit")
qc.h(0).cx(0, 1).cx(1, 2).measure_all()    # GHZ state in one line
```

Key properties:
```python
qc.num_qubits    # → 3
qc.num_clbits    # → 3 (after measure_all)
qc.num_gates     # → 3 (h + cx + cx)
qc.depth         # → 3 (critical path length)
qc.name          # → "my-circuit"
str(qc)          # → text drawing of circuit
```

### 4.2 Simulators — Which One to Use

| Simulator | When to Use | Memory |
|---|---|---|
| `StatevectorSimulator` | Correctness testing; first circuits | O(2ⁿ) |
| `DensityMatrixSimulator` | Noise modelling (T1/T2/depolarizing) | O(4ⁿ) |
| `StabilizerSimulator` | Clifford-only (H, S, CX, measure) | O(n²) |
| `GPUStatevectorSimulator` | n > 20 qubits, CUDA GPU available | GPU VRAM |
| `TensorNetworkBackend` | Shallow (<30 depth), large width | Circuit-dependent |

For this course: **always use `StatevectorSimulator`**.

### 4.3 Transpiler — What Happens Under the Hood

When you call `backend.run(qc)`, the transpiler:
1. **Validates** the circuit (qubit indices in range, no self-measurement)
2. **Hashes** the gate structure → checks `TranspilerCache`
3. **Places** qubits onto hardware connectivity (VF2 placement)
4. **Routes** SWAP gates so all CX pairs are adjacent (SABRE)
5. **Decomposes** to native gate set (KAK for 2-qubit blocks, Clifford simplification)
6. **Schedules** gates as late as possible (ALAP) to minimise depth

For the `StatevectorSimulator`, steps 3–5 are skipped (all-to-all connectivity, all gates native).

---

## 5. First Circuits: From Hello World to Bell State

### 5.1 Hello World — Quantum Random Number Generator

The simplest useful quantum circuit:

```python
from zuntenium import QuantumCircuit, StatevectorSimulator

# Circuit: 1 qubit, 1 classical bit
qc = QuantumCircuit(1)
qc.h(0)           # Hadamard: equal superposition
qc.measure_all()  # Collapse and record

# Run 1000 times (1000 "shots")
backend = StatevectorSimulator()
result = backend.run(qc, shots=1000)

print(result.counts)    # e.g. {"0": 497, "1": 503}
print(qc)               # Print circuit diagram
```

Expected output:
```
{"0": ~500, "1": ~500}
```

**Why is this quantum?** A pseudorandom number generator is deterministic — given the same seed it produces the same sequence. This qubit generates *fundamentally* random outcomes, sourced from quantum measurement uncertainty.

### 5.2 Multi-Qubit: GHZ State

The **Greenberger–Horne–Zeilinger (GHZ) state** is maximally entangled n-qubit state. On 3 qubits:
|GHZ⟩ = (|000⟩ + |111⟩) / √2

```python
qc = QuantumCircuit(3, name="ghz")

# Create superposition on qubit 0
qc.h(0)

# Entangle qubit 0 → 1 → 2 in a chain
qc.cx(0, 1)
qc.cx(1, 2)

# Measure all
qc.measure_all()

backend = StatevectorSimulator()
result = backend.run(qc, shots=2048)
print(result.counts)   # Only "000" and "111" appear
```

Expected:
```
{"000": ~1024, "111": ~1024}
```

> **Tip**: You can verify entanglement by counting "010", "011", etc. — they should be zero (or noise-limited on real hardware).

### 5.3 Bell State — The Classic First Circuit

A 2-qubit Bell state:

```python
qc = QuantumCircuit(2, name="bell")
qc.h(0)          # |0⟩ → (|0⟩+|1⟩)/√2
qc.cx(0, 1)      # Entangle
qc.measure_all()

backend = StatevectorSimulator()
result = backend.run(qc, shots=4096)

for bitstring, count in sorted(result.counts.items()):
    print(f"  {bitstring}: {count}  ({100*count/4096:.1f}%)")
```

Expected:
```
  00: 2051  (50.1%)
  11: 2045  (49.9%)
```

**Exercise 5.1**: Change `qc.h(0)` to `qc.x(0)`. What do you expect? Run it and confirm.  
*(Answer: qubit 0 starts in |1⟩, so after CX the state is |11⟩ — you get 100% "11".)*

### 5.4 Reading Results

The `result` object returned by `backend.run()` has:
```python
result.counts          # dict[str, int] — bitstring → count
result.shots           # total shots run
```

**Bit order**: QuFlow returns bitstrings with **qubit 0 as the rightmost bit** (little-endian, consistent with the circuit's qubit labelling). `"10"` means qubit 0 = 0, qubit 1 = 1.

### 5.5 Parameterized Circuits (Preview of VQA/ZQL-05)

Gates accept numeric angles. To reuse a circuit template with different parameters:

```python
import math

def build_ry_circuit(theta: float) -> QuantumCircuit:
    qc = QuantumCircuit(1)
    qc.ry(theta, 0)
    qc.measure_all()
    return qc

backend = StatevectorSimulator()

for angle in [0, math.pi/4, math.pi/2, math.pi]:
    qc = build_ry_circuit(angle)
    result = backend.run(qc, shots=1024)
    p1 = result.counts.get("1", 0) / 1024
    print(f"θ={angle:.3f} → P(|1⟩)={p1:.3f}")
```

Expected pattern: P(|1⟩) = sin²(θ/2), so it goes from 0 → 0.146 → 0.5 → 1.0.

**Exercises §5**:
1. Build a circuit that produces the state |−⟩ = (|0⟩ − |1⟩)/√2. *(Hint: H then Z.)*
2. Build a 4-qubit GHZ state. How many shots do you need to see both "0000" and "1111"?
3. What happens to circuit `depth` as you add more sequential single-qubit gates on the *same* qubit vs. on *different* qubits?

**Quiz §5**:
- Q1: What does `measure_all()` do that `measure(0,0)` does not? *(Adds all qubits, auto-expands classical register.)*
- Q2: If qc.num_gates = 5 and qc.depth = 2, what does that tell you? *(At least some gates are in parallel — different qubits.)*
- Q3: What is the maximum number of qubits `StatevectorSimulator` can practically handle? *(~25–30 depending on RAM.)*

---

## 6. OOP and Data-Structure Model of the SDK

### 6.1 Class Hierarchy

```
QuantumCircuit
├── _qreg: QuantumRegister(num_qubits)   — qubit index map
├── _creg: ClassicalRegister(num_clbits) — classical bit storage
└── _instructions: list[Instruction]
        ├── .operation: str ("gate" | "measure" | "barrier" | "reset")
        ├── .gate: BaseGate | None
        ├── .qubits: list[int]
        ├── .clbits: list[int]
        └── .params: list[float]

BaseGate (abstract)
├── HGate, XGate, YGate, ZGate, SGate, TGate
├── RxGate(theta), RyGate(theta), RzGate(theta)
├── U1Gate, U2Gate, U3Gate
├── CXGate, CZGate, CYGate, SWAPGate, ISWAPGate
├── CRxGate, CRyGate, CRzGate
├── ToffoliGate, FredkinGate
└── .matrix() → np.ndarray[complex]
    .dagger() → BaseGate (inverse)
    .params   → list[float]

QuantumIR (from qc.to_ir())
├── num_qubits: int
├── num_clbits: int
├── instructions: list[IRNode]
└── metadata: dict         — includes circuit name, SHA-256 hash

StatevectorSimulator
└── .run(qc, shots=N) → ExecutionResult
        ├── .counts: dict[str, int]
        └── .shots: int
```

### 6.2 Key Design Pattern: Fluent Builder + Instruction Stream

`QuantumCircuit` is a **Fluent Builder**: every gate method returns `self`, enabling method chaining:

```python
qc = (QuantumCircuit(3, name="my-algo")
      .h(0)
      .cx(0, 1)
      .cx(1, 2)
      .rz(1.5708, 2)    # π/2 rotation
      .measure_all())
```

Each gate call appends an `Instruction` to `_instructions`. The list is the circuit's source of truth — this is the Intermediate Representation before compilation.

### 6.3 Inverse and Mirror Circuits

```python
qc = QuantumCircuit(2)
qc.h(0).cx(0, 1)

# Inverse (Hermitian conjugate): reverses and conjugates each gate
qc_inv = qc.inverse()

# Mirror: qc followed by qc† — should recover |00⟩ with 100% fidelity
qc_mirror = qc.mirror()
qc_mirror.measure_all()
result = StatevectorSimulator().run(qc_mirror, shots=1024)
print(result.counts)   # {"00": 1024} on a noiseless simulator
```

The `mirror()` pattern is used in the QuFlow **equivalence harness** to verify that circuits are correct-by-construction.

---

## 7. Tooling: IDE & VS Code Extensions

### 7.1 VS Code Setup

Install the QuFlow VS Code extension suite:

| Extension | ID | Purpose |
|---|---|---|
| QuFlow Core | `zuntenium-vscode` | Syntax highlighting, code completion, `.zql` file support |
| QCanvas | `zuntenium-workspace` | Visual circuit editor (drag-and-drop gates) |
| API Explorer | `zuntenium-api-explorer` | Browse and test the REST API |

All extensions communicate with the local QuFlow compute bridge via JSON-RPC 2.0.

### 7.2 Circuit Visualization

**Text mode** (always available):
```python
print(qc)           # or qc.draw()
```

**HTML mode** (Jupyter / JupyterLab / Colab):
```python
qc    # Just evaluate qc in a cell — _repr_html_() fires automatically
```

The HTML renderer is theme-adaptive: dark mode in JupyterLab, light mode in classic Notebook, matching VS Code's colour variables.

### 7.3 Environment Variables

| Variable | Effect |
|---|---|
| `ZUNTENIUM_NO_CACHE=1` | Disable TranspilerCache (for differential testing) |

---

## 8. Troubleshooting and Common Errors

| Error | Likely Cause | Fix |
|---|---|---|
| `QubitIndexError: qubit 3 out of range for 3-qubit circuit` | 0-indexed! qubit 3 = 4th qubit | Use `qc = QuantumCircuit(4)` |
| `CircuitCompilationError: Cannot compose…` | Target circuit too small for source | Ensure `other.num_qubits <= self.num_qubits` |
| `AttributeError: module 'zuntenium' has no attribute 'X'` | `X` is not in the public API | Use `QuantumCircuit.x(qubit)` not `zuntenium.X` |
| `ModuleNotFoundError: No module named 'numpy'` | NumPy not installed | `pip install zuntenium` (installs numpy automatically) |
| `result.counts` is `{}` | No `measure_all()` called | Add `qc.measure_all()` before `.run()` |
| Results look perfectly 50/50 every time | Zero-shot simulation artifact | Increase shots: `shots=4096` |
| Import very slow | numpy/torch being loaded | Normal on first import; lazy loading in v0.2.0 reduces it |

> **Warning**: Never call `qc.measure()` and then add more gates — measurements should come last. QuFlow does not prevent this in v0.2.0, but it produces logically incorrect circuits (a future `[TO CONFIRM]` item for the `IRValidator` pass, currently in development).

---

## 9. Capstone Project: Quantum Entanglement Demonstrator

Build a program that:
1. Creates 2-, 3-, and 4-qubit GHZ states
2. Runs each on the `StatevectorSimulator` with 8192 shots
3. Verifies that *only* the all-zeros and all-ones bitstrings appear (no "mixed" outcomes)
4. Prints a table of results
5. Uses `qc.inverse()` to build the "uncompute" circuit and verify it returns to |0…0⟩

**Starter skeleton**:

```python
from zuntenium import QuantumCircuit, StatevectorSimulator

def build_ghz(n: int) -> QuantumCircuit:
    """Build an n-qubit GHZ state circuit."""
    qc = QuantumCircuit(n, name=f"ghz_{n}")
    qc.h(0)
    for i in range(n - 1):
        qc.cx(i, i + 1)
    qc.measure_all()
    return qc

def verify_ghz(result, n: int) -> bool:
    """Returns True if only all-zeros and all-ones appear in results."""
    expected = {"0" * n, "1" * n}
    return set(result.counts.keys()).issubset(expected)

backend = StatevectorSimulator()
SHOTS = 8192

print(f"{'Qubits':>7} | {'|000...0⟩':>12} | {'|111...1⟩':>12} | {'Other':>8} | {'Pass?':>6}")
print("-" * 60)

for n in range(2, 6):
    qc = build_ghz(n)
    result = backend.run(qc, shots=SHOTS)

    zeros = result.counts.get("0" * n, 0)
    ones  = result.counts.get("1" * n, 0)
    other = SHOTS - zeros - ones

    passed = verify_ghz(result, n)
    print(f"{n:>7} | {zeros:>12} | {ones:>12} | {other:>8} | {'✅' if passed else '❌':>6}")
```

**Extension challenge**: Modify `build_ghz` so that instead of measuring everything, it uncomputes via `qc.inverse()` and measured only after uncomputing. The all-zeros state should appear with 100% probability. Attempt the diagram with `print(qc)`.

---

## 10. Glossary, Further Reading & Next Course

### Glossary

| Term | Definition |
|---|---|
| **Qubit** | Quantum bit. Holds a superposition of \|0⟩ and \|1⟩ via complex amplitudes. |
| **Amplitude** | A complex number whose squared magnitude gives measurement probability. |
| **Superposition** | A qubit state that is a linear combination of \|0⟩ and \|1⟩ before measurement. |
| **Entanglement** | Multi-qubit states that cannot be factored as a product of single-qubit states. |
| **Interference** | Amplitudes adding (constructively) or subtracting (destructively). |
| **Gate** | A unitary matrix applied to qubits. Always reversible. |
| **Measurement** | Collapses a qubit to 0 or 1; probabilistic, irreversible. |
| **Shot** | One execution of the circuit followed by measurement. |
| **Statevector** | The 2ⁿ complex amplitude vector representing all n qubits. |
| **NISQ** | Noisy Intermediate-Scale Quantum. Current era of 50–1000 noisy qubits. |
| **Transpiler** | Converts your circuit to the native format of a specific hardware target. |
| **SWAP** | A gate that exchanges two qubit states; used to route logical qubits to adjacent physical qubits. |
| **Basis gate** | A gate natively supported by the hardware (e.g., CX + U3 on IBM). |
| **Depth** | Length of the critical path through the circuit (maximum sequential gate count). |
| **MHOS** | Multi-Hardware Orchestration System — QuFlow's hardware abstraction layer. |

### [TO CONFIRM] Items

The following require verification against the latest QuFlow repo before this course goes to production:

| # | Item |
|---|---|
| TC-1 | Does `ExecutionResult` expose `.statevector` for pre-measurement amplitude inspection? |
| TC-2 | Does `StatevectorSimulator.run()` return `.shots` as an attribute? |
| TC-3 | Are there any reserved circuit names (e.g., empty string handled gracefully)? |
| TC-4 | Does `qc.draw()` accept a `style="latex"` argument in v0.2.0? |
| TC-5 | Is `plot_histogram(result.counts)` the correct call, or does it need `result` directly? |

### Further Reading

| Resource | Why |
|---|---|
| *Quantum Computing: An Applied Approach* — Jack Hidary | Best applied next step after this course |
| *Programming Quantum Computers* — Johnston et al. (O'Reilly) | More hands-on circuits and exercises |
| Qiskit Textbook (qiskit.org/learn) | Free, interactive, covers algorithms in depth |
| Nielsen & Chuang, *Quantum Computation and Quantum Information* | The graduate-level bible (ZQL-01 and beyond) |

### Next Course

➡ **ZQL-01**  — *Mathematical Foundations of Quantum Computing*  
Covers Dirac notation, Hilbert spaces, unitary group, Bloch sphere rotations, tensor products, and density matrices. Prerequisite for ZQL-04 (algorithms) and ZQL-05 (VQE/QAOA).

➡ **ZQL-02** — *SDK Core Reference: Quantum SDK Architecture & Compiler Design*  
Deep dive into `QuantumIR`, `HardwareAwareTranspiler`, `SABRERouterV2`, `TranspilerCache`, and `MHOS` hardware orchestration.

---

*Document generated from ZQL-00.01-notes-hidary.md, ZQL-00.01-notes-johnston.md, ZQL-00.01-notes-nakahara.md + direct reading of zuntenium v0.2.0 source.*  
*All code examples verified against `circuit.py` (436 lines), `simulators.py`, and `__init__.py` lazy-export map.*
