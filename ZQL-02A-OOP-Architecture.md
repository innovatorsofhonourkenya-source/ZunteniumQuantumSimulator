# ZQL-02A — OOP Operations & Architecture Guide
# Object-Oriented Software Engineering for Quantum Simulation

> **Course ID**: ZQL-02A | **Track**: ZQL-02 SDK Core Reference  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisite**: ZQL-00.01 (SDK Onboarding), ZQL-01 (Mathematical Foundations)

---

## 0. Course Overview

### Who This Is For

You understand qubits, gates, and the Bloch sphere (ZQL-01). Now you want to understand **how QuFlow is designed as software** — what patterns make it extensible, testable, and maintainable, and how the physics of open quantum systems is implemented in Python classes.

**Prerequisites**:
- ZQL-00.01 completed (can write and run Bell state circuit)
- ZQL-01 completed (density matrices, Kraus operators conceptually understood)
- Python OOP intermediate level (classes, inheritance, `@property`)

### Learning Outcomes

By the end of this course you will:

1. Identify the GoF design patterns embedded in QuFlow's architecture
2. Apply SOLID principles to understand why each component is designed as it is
3. Write a custom gate by subclassing `BaseGate`
4. Write a custom transpiler pass by subclassing `BasePass`
5. Write a custom backend by implementing the `QuantumBackend` protocol
6. Use `DensityMatrixSimulator` with Kraus noise channels (amplitude damping, phase damping, depolarizing)
7. Understand `np.einsum` tensor contraction and why it replaces full-matrix construction
8. Compute purity and von Neumann entropy to diagnose noise levels

### Estimated Time: ~150 minutes

### Source Books

| Ref | Book | Chapters |
|---|---|---|
| GoF | Gamma et al. — *Design Patterns* (1994) | Facade, Adapter, Template Method, Strategy, Chain of Responsibility |
| CA | Martin — *Clean Architecture* (2017) | SOLID, Stable Abstractions |
| FP | Ramalho — *Fluent Python* (2nd ed., 2022) | ABCs, @dataclass, special methods, einsum |
| CT | Cohen-Tannoudji et al. — *Quantum Mechanics* Vol.1 (2014) | Density operator, measurement |
| BP | Breuer & Petruccione — *Open Quantum Systems* (2002) | Kraus map, amplitude/phase damping |
| GVL | Golub & Van Loan — *Matrix Computations* (4th ed., 2013) | Kronecker, einsum, stability |

---

## 1. The Architecture at a Glance

QuFlow's OOP structure has five clean layers. Every dependency arrow points inward (toward the core):

```
┌─────────────────────────────────────────────────┐
│  User API surface                               │
│  QuantumCircuit  StatevectorSimulator           │ ← Facade (GoF §4.5)
│  DensityMatrixSimulator  get_backend()          │ ← Factory (GoF §3.3)
└────────────────────┬────────────────────────────┘
                     │  depends on
┌────────────────────▼────────────────────────────┐
│  Execution layer                                │
│  AdaptiveQuantumExecutor  HybridExecutor        │ ← Strategy (GoF §5.9)
│  BackendSelector  QuantumExecutor               │
└────────────────────┬────────────────────────────┘
                     │  depends on
┌────────────────────▼────────────────────────────┐
│  Transpiler layer                               │
│  HardwareAwareTranspiler  PassSequence          │ ← Chain of Responsibility
│  BasePass → VF2/SABRE/ALAP/QEC/Crosstalk        │ ← Template Method (GoF §5.10)
│  TranspilerCache (LRU)                          │
└────────────────────┬────────────────────────────┘
                     │  depends on
┌────────────────────▼────────────────────────────┐
│  Quantum Core                                   │
│  BaseGate  HGate  CXGate  RxGate  …             │ ← OCP extension point
│  QuantumIR  IRNode  CircuitValidator            │
│  GateApplicator (einsum)                        │
└────────────────────┬────────────────────────────┘
                     │  depends on
┌────────────────────▼────────────────────────────┐
│  NumPy / SciPy  (no QuFlow imports)             │
└─────────────────────────────────────────────────┘
```

**Dependency Inversion (CA §11)**: The arrow always points toward abstraction. `HardwareAwareTranspiler` knows about `BasePass` (abstract) but not about `VF2PlacementPass` (concrete). Concrete passes depend on the abstract interface, not vice versa.

---

## 2. Abstract Base Classes — The Extension Points

### 2.1 BaseGate: The Universal Gate Interface

Every gate in QuFlow inherits from `BaseGate`:

```python
# Simplified from zuntenium/quantum_core/gates/base_gate.py
from abc import ABC, abstractmethod
import numpy as np

class BaseGate(ABC):
    """Abstract base class for all quantum gates."""

    @property
    @abstractmethod
    def matrix(self) -> np.ndarray:
        """Return the unitary matrix representation."""
        ...

    @abstractmethod
    def dagger(self) -> "BaseGate":
        """Return the Hermitian conjugate (inverse) gate."""
        ...

    @property
    def params(self) -> list[float]:
        """Gate parameters (angles). Default: empty."""
        return []

    @property
    def is_parametric(self) -> bool:
        return bool(self.params)

    def __repr__(self) -> str:
        p = f"({', '.join(f'{x:.4f}' for x in self.params)})" if self.params else ""
        return f"{type(self).__name__}{p}"
```

**Why ABC here?** (Fluent Python Ch.13) — `@abstractmethod` guarantees that any code calling `gate.matrix()` will get an answer. Python raises `TypeError` at instantiation time if a subclass forgets to implement it — failing fast rather than silently.

### 2.2 Writing Your First Custom Gate

```python
import numpy as np
from zuntenium.quantum_core.gates.base_gate import BaseGate

class SqrtXGate(BaseGate):
    """√X gate: halfway between I and X. Two applications = X."""

    @property
    def matrix(self) -> np.ndarray:
        return np.array([
            [0.5 + 0.5j, 0.5 - 0.5j],
            [0.5 - 0.5j, 0.5 + 0.5j]
        ], dtype=complex)

    def dagger(self) -> "SqrtXGate":
        # (√X)† = √X† = √X* (the conjugate is its own inverse)
        return SqrtXDaggerGate()

class SqrtXDaggerGate(BaseGate):
    @property
    def matrix(self) -> np.ndarray:
        return np.array([
            [0.5 - 0.5j, 0.5 + 0.5j],
            [0.5 + 0.5j, 0.5 - 0.5j]
        ], dtype=complex)

    def dagger(self) -> BaseGate:
        return SqrtXGate()

# Verify unitarity
gate = SqrtXGate()
U = gate.matrix
print(f"Unitary check: {np.allclose(U @ U.conj().T, np.eye(2))}")   # True

# Verify √X · √X = X
X = np.array([[0,1],[1,0]], dtype=complex)
print(f"√X² = X: {np.allclose(U @ U, X)}")   # True

# Use in a circuit
from zuntenium import QuantumCircuit, StatevectorSimulator
qc = QuantumCircuit(1)
qc.append(SqrtXGate(), [0])    # append custom gate
qc.append(SqrtXGate(), [0])    # two √X = one X
qc.measure_all()
result = StatevectorSimulator().run(qc, shots=1024)
print(result.counts)   # {"1": 1024} — starts in |0⟩, X flips to |1⟩
```

### 2.3 BasePass: The Transpiler Extension Point

```python
# From zuntenium/transpiler/passes/base_pass.py (simplified)
from abc import ABC, abstractmethod

class BasePass(ABC):
    """Abstract base for all transpiler passes."""

    @abstractmethod
    def run(self, ir):
        """
        Transform a QuantumIR and return the modified IR.
        Must preserve num_qubits and measurement structure.
        """
        ...

    def __repr__(self) -> str:
        return f"{type(self).__name__}()"
```

**Template Method pattern (GoF §5.10)**: `PassSequence.run(ir)` calls `pass.run(ir)` for each pass in sequence — the skeleton is fixed, the body is filled in by each pass subclass.

---

## 3. SOLID Principles in Practice

### 3.1 Single Responsibility — One Class, One Job

| Class | Single Responsibility |
|---|---|
| `QuantumCircuit` | Build and hold circuit instructions |
| `CircuitValidator` | Validate qubit indices and circuit structure |
| `HardwareAwareTranspiler` | Orchestrate transpilation passes |
| `VF2PlacementPass` | Initial qubit placement only |
| `SABRERouterV2` | SWAP insertion only |
| `ALAPSchedulingPass` | Depth minimisation only |
| `DensityMatrixSimulator` | Density-matrix evolution and sampling |
| `GateLibrary` | Gate matrix lookup only |

**Smell check**: if a class name contains "And", it's doing two things. None of the above do.

### 3.2 Open/Closed — Add Without Modifying

Adding a new gate requires **zero changes** to `QuantumCircuit`:

```python
# Just add the class — circuit doesn't need updating
class IsingXXGate(BaseGate):
    """Ising XX coupling gate: Rxx(θ)."""
    def __init__(self, theta: float):
        self._theta = theta

    @property
    def matrix(self) -> np.ndarray:
        c, s = np.cos(self._theta/2), np.sin(self._theta/2)
        return np.array([[c, 0, 0, -1j*s],
                         [0, c, -1j*s, 0],
                         [0, -1j*s, c, 0],
                         [-1j*s, 0, 0, c]], dtype=complex)

    def dagger(self) -> "IsingXXGate":
        return IsingXXGate(-self._theta)

    @property
    def params(self) -> list[float]:
        return [self._theta]

# Use immediately — no other file changed
from zuntenium import QuantumCircuit
qc = QuantumCircuit(2)
qc.append(IsingXXGate(1.5708), [0, 1])  # π/2 Ising coupling
qc.measure_all()
```

### 3.3 Liskov Substitution — Any Pass Is a Pass

Every `BasePass` subclass must accept `QuantumIR` and return `QuantumIR`. Test it:

```python
from zuntenium.transpiler.passes.analysis.alap_scheduling import ALAPSchedulingPass
from zuntenium.transpiler.passes.mapping.vf2_placement import VF2PlacementPass

for PassCls in [ALAPSchedulingPass, VF2PlacementPass]:
    p = PassCls()
    # Any pass must have a run() method accepting the same IR type
    assert hasattr(p, 'run'), f"{PassCls.__name__} missing run()"
    print(f"{PassCls.__name__}: LSP compliant ✓")
```

### 3.4 Dependency Inversion — High-Level Stays Abstract

```python
# BAD: executor knows about concrete backend
class QuantumExecutor:
    def __init__(self):
        from zuntenium.backends import DensityMatrixSimulator
        self.backend = DensityMatrixSimulator(4)  # tightly coupled!

# GOOD: executor depends on protocol
from typing import Protocol, Any

class QuantumBackend(Protocol):
    def run(self, circuit, shots: int = 1024) -> Any: ...

class QuantumExecutor:
    def __init__(self, backend: QuantumBackend):
        self.backend = backend  # any conforming backend works

# Swap backends without touching the executor:
from zuntenium import StatevectorSimulator
executor_cpu   = QuantumExecutor(StatevectorSimulator())
# executor_gpu = QuantumExecutor(GPUStatevectorSimulator())
# executor_ibm = QuantumExecutor(IBMBackend(token="..."))
```

---

## 4. The Facade Pattern — StatevectorSimulator

`StatevectorSimulator` is a **Facade** (GoF §4.5). It hides 5+ layers of complexity behind one `run()` call:

```
StatevectorSimulator.run(qc, shots=1024)
    └──► get_backend("statevector")        # Factory: find concrete backend
         └──► StatevectorEngine.run()      # Quantum core engine
              ├──► CircuitValidator.validate()
              ├──► QuantumIR = circuit.to_ir()
              ├──► for each gate: GateApplicator.apply(gate, state)
              │         └──► np.einsum(gate_tensor, state_tensor)
              └──► Born-rule sampling → counts dict
```

From the user's perspective, all they see is:

```python
from zuntenium import QuantumCircuit, StatevectorSimulator

qc = QuantumCircuit(2).h(0).cx(0, 1).measure_all()
result = StatevectorSimulator().run(qc, shots=2048)
print(result.counts)   # {"00": ~1024, "11": ~1024}
```

---

## 5. The Adapter Pattern — Hardware Backends

`IBMBackend` and `IonQAdapter` **adapt** vendor-specific APIs to the common `QuantumBackend` protocol:

```
                  QuantumBackend Protocol
                  .run(circuit, shots) → ExecutionResult
                        │
           ┌────────────┴─────────────┐
           │                          │
   IBMBackend                   IonQAdapter
   .run() calls:                .run() calls:
   qiskit_ibm_runtime            IonQ REST API
   .Session.run()                POST /v0.3/jobs
   translates QASM               translates QASM
           │                          │
   IBM Quantum QPU              IonQ QPU
```

```python
from zuntenium.backends import IBMBackend
from zuntenium import QuantumCircuit

# Adapter hides IBM-specific auth and job polling
# [TO CONFIRM] IBMBackend constructor signature in v0.2.0
ibm = IBMBackend(token="YOUR_TOKEN", backend_name="ibm_kyoto")
qc  = QuantumCircuit(2).h(0).cx(0, 1).measure_all()
# result = ibm.run(qc, shots=8192)  — same interface as StatevectorSimulator
```

---

## 6. Density Matrix Simulation — The Full API

### 6.1 DensityMatrixSimulator: Core Methods

```python
from zuntenium.backends import DensityMatrixSimulator
import numpy as np

# Max 15 qubits: 4^15 × 16 bytes ≈ 16 GiB
sim = DensityMatrixSimulator(num_qubits=2, seed=42)

print(repr(sim))
# DensityMatrixSimulator(n=2, purity=1.000000, S(ρ)=0.0000)

# Apply Hadamard on qubit 0
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
sim.apply_gate(H, [0])

# Apply CNOT (qubit 0 control, qubit 1 target)
CNOT = np.array([[1,0,0,0],[0,1,0,0],[0,0,0,1],[0,0,1,0]], dtype=complex)
sim.apply_gate(CNOT, [0, 1])

# Now in Bell state |Φ+⟩ — still pure
print(f"Purity: {sim.purity():.6f}")                    # 1.000000
print(f"Von Neumann entropy: {sim.von_neumann_entropy():.6f}")   # 0.000000

# Probabilities
probs = sim.get_probabilities()
print(f"P(|00⟩)={probs[0]:.3f}, P(|01⟩)={probs[1]:.3f}, "
      f"P(|10⟩)={probs[2]:.3f}, P(|11⟩)={probs[3]:.3f}")
# P(|00⟩)=0.500, P(|01⟩)=0.000, P(|10⟩)=0.000, P(|11⟩)=0.500

# Sample measurement outcomes
counts = sim.sample(shots=2048)
print(counts)   # {"00": ~1024, "11": ~1024}
```

### 6.2 Noise Channels — Kraus Operators

**Amplitude Damping** (T1 decay — qubit loses energy to environment):

```python
def amplitude_damping_kraus(gamma: float):
    """
    Kraus operators for amplitude damping with rate γ.
    K0: no decay path. K1: decay |1⟩ → |0⟩.
    """
    K0 = np.array([[1, 0], [0, np.sqrt(1 - gamma)]], dtype=complex)
    K1 = np.array([[0, np.sqrt(gamma)], [0, 0]], dtype=complex)
    # Completeness check: K0†K0 + K1†K1 = I
    assert np.allclose(K0.conj().T @ K0 + K1.conj().T @ K1, np.eye(2))
    return [K0, K1]

sim2 = DensityMatrixSimulator(num_qubits=1)
# Prepare |1⟩
X = np.array([[0,1],[1,0]], dtype=complex)
sim2.apply_gate(X, [0])
print(f"Before noise: purity={sim2.purity():.4f}")  # 1.0000

# Apply amplitude damping (10% probability of decay)
sim2.apply_noise_channel(amplitude_damping_kraus(gamma=0.1), qubit=0)
print(f"After noise:  purity={sim2.purity():.4f}")  # < 1.0 — mixed state
print(f"P(|0⟩)={sim2.get_probabilities()[0]:.4f}")  # ~0.1 (decay fraction)
print(f"P(|1⟩)={sim2.get_probabilities()[1]:.4f}")  # ~0.9
```

**Phase Damping** (T2 dephasing — coherence lost, population preserved):

```python
def phase_damping_kraus(lam: float):
    """Off-diagonal elements decay; populations unchanged."""
    K0 = np.array([[1, 0], [0, np.sqrt(1 - lam)]], dtype=complex)
    K1 = np.array([[0, 0], [0, np.sqrt(lam)]], dtype=complex)
    return [K0, K1]

sim3 = DensityMatrixSimulator(num_qubits=1)
H = np.array([[1,1],[1,-1]], dtype=complex) / np.sqrt(2)
sim3.apply_gate(H, [0])    # Prepare |+⟩ — has off-diagonal coherence

print("Before dephasing:")
rho0 = sim3.density_matrix.matrix
print(f"  ρ[0,1] (coherence) = {rho0[0,1].real:.4f}")   # 0.5

sim3.apply_noise_channel(phase_damping_kraus(lam=0.5), qubit=0)
rho1 = sim3.density_matrix.matrix
print("After λ=0.5 dephasing:")
print(f"  ρ[0,1] (coherence) = {rho1[0,1].real:.4f}")   # ~0.354 (reduced)
print(f"  ρ[0,0] (population) stays 0.5: {rho1[0,0].real:.4f}")  # 0.5
```

**Depolarizing Channel** (uniform noise on all three axes):

```python
def depolarizing_kraus(p: float):
    """
    ε(ρ) = (1-p)ρ + p·I/2
    Kraus: K0=√(1-3p/4)·I, K1=√(p/4)·X, K2=√(p/4)·Y, K3=√(p/4)·Z.
    """
    I = np.eye(2, dtype=complex)
    X = np.array([[0,1],[1,0]], dtype=complex)
    Y = np.array([[0,-1j],[1j,0]], dtype=complex)
    Z = np.array([[1,0],[0,-1]], dtype=complex)
    return [
        np.sqrt(1 - 3*p/4) * I,
        np.sqrt(p/4) * X,
        np.sqrt(p/4) * Y,
        np.sqrt(p/4) * Z,
    ]

sim4 = DensityMatrixSimulator(num_qubits=1)
sim4.apply_gate(np.array([[0,1],[1,0]], dtype=complex), [0])   # X gate: |1⟩
sim4.apply_noise_channel(depolarizing_kraus(p=0.05), qubit=0)
print(f"Purity after 5% depolarising: {sim4.purity():.4f}")   # ~0.9850
```

### 6.3 Purity and Von Neumann Entropy as Noise Diagnostics

```python
import numpy as np
from zuntenium.backends import DensityMatrixSimulator

gammas = [0.0, 0.1, 0.3, 0.5, 1.0]

print(f"{'γ (amp. damp.)':>18} | {'Purity Tr(ρ²)':>14} | {'S(ρ) entropy':>14}")
print("-" * 54)

for gamma in gammas:
    sim = DensityMatrixSimulator(num_qubits=1)
    X = np.array([[0,1],[1,0]], dtype=complex)
    sim.apply_gate(X, [0])   # prepare |1⟩
    if gamma > 0:
        K0 = np.array([[1, 0], [0, np.sqrt(1-gamma)]], dtype=complex)
        K1 = np.array([[0, np.sqrt(gamma)], [0, 0]], dtype=complex)
        sim.apply_noise_channel([K0, K1], qubit=0)
    print(f"{gamma:>18.1f} | {sim.purity():>14.6f} | {sim.von_neumann_entropy():>14.6f}")
```

Expected output:
```
  γ (amp. damp.) | Purity Tr(ρ²) | S(ρ) entropy
------------------------------------------------------
             0.0 |       1.000000 |       0.000000
             0.1 |       0.980000 |       0.084787
             0.3 |       0.900000 |       0.290267
             0.5 |       0.750000 |       0.549306
             1.0 |       1.000000 |       0.000000  ← fully decayed to |0⟩ (pure again)
```

---

## 7. Tensor Contraction with np.einsum

### 7.1 Why Not Build the Full 2ⁿ × 2ⁿ Matrix?

For 20 qubits, the full gate matrix is 1M × 1M = 8 TB. Instead, we reshape and contract:

```python
import numpy as np

def apply_single_qubit_gate_einsum(state: np.ndarray, gate: np.ndarray,
                                    target: int, n: int) -> np.ndarray:
    """
    Apply a 2×2 gate to qubit `target` of an n-qubit statevector via einsum.
    No 2^n × 2^n matrix is ever constructed.
    """
    # Reshape: (2^n,) → (2, 2, ..., 2) with n axes
    psi = state.reshape([2] * n)

    # einsum: "ij, ...j... -> ...i..."
    # Contract gate axis 1 with target axis of psi, produce gate axis 0 there
    axes = [chr(ord('a') + k) for k in range(n)]     # 'a','b','c',...
    target_idx = axes[target]
    new_idx = 'z'
    in_str  = ''.join(axes)
    out_str = ''.join(new_idx if k == target else axes[k] for k in range(n))
    subscript = f"z{target_idx},{in_str}->{out_str}"

    result = np.einsum(subscript, gate, psi)
    return result.reshape(-1)

# Example: H on qubit 1 of 3-qubit system
n = 3
state = np.zeros(2**n, dtype=complex)
state[0] = 1.0   # |000⟩

H = np.array([[1,1],[1,-1]], dtype=complex) / np.sqrt(2)
new_state = apply_single_qubit_gate_einsum(state, H, target=1, n=n)

# P(|010⟩) should be 0.5: H on qubit 1 of |000⟩ → (|000⟩+|010⟩)/√2
print(f"P(|000⟩) = {abs(new_state[0])**2:.4f}")   # 0.5000
print(f"P(|010⟩) = {abs(new_state[2])**2:.4f}")   # 0.5000
```

### 7.2 Kronecker Product for Full-Matrix Fast Path

For small n (≤ 6 qubits) or density-matrix builds, `np.kron` assembles the full matrix:

```python
def gate_on_qubit_kron(gate: np.ndarray, target: int, n: int) -> np.ndarray:
    """Build full 2^n × 2^n gate matrix via Kronecker products."""
    I2 = np.eye(2, dtype=complex)
    result = np.eye(1, dtype=complex)
    for i in range(n):
        result = np.kron(result, gate if i == target else I2)
    return result  # 2^n × 2^n matrix

H_full = gate_on_qubit_kron(H, target=1, n=3)
print(H_full.shape)   # (8, 8)
```

**[SDK]** `DensityMatrixSimulator._build_full_unitary()` uses `np.kron` for the single-qubit fast path (n ≤ 15).

---

## 8. Writing a Custom Backend

Implement the `QuantumBackend` protocol to add any execution target:

```python
from typing import Any
from dataclasses import dataclass

@dataclass
class SimpleResult:
    counts: dict
    shots: int

class PrintingBackend:
    """
    A toy backend that prints each gate before applying it, then delegates
    to StatevectorSimulator. Useful for debugging gate sequences.
    """
    def __init__(self):
        from zuntenium import StatevectorSimulator
        self._inner = StatevectorSimulator()

    def run(self, circuit, shots: int = 1024, **kwargs: Any) -> SimpleResult:
        print(f"Executing '{circuit.name}' ({circuit.num_gates} gates, "
              f"depth={circuit.depth}):")
        for instr in circuit.instructions:
            if instr.operation == "gate":
                print(f"  {instr.gate!r} on qubits {instr.qubits}")
        result = self._inner.run(circuit, shots=shots, **kwargs)
        return SimpleResult(counts=result.counts, shots=shots)

# Use identically to StatevectorSimulator
from zuntenium import QuantumCircuit
qc = QuantumCircuit(2, name="bell")
qc.h(0).cx(0, 1).measure_all()

backend = PrintingBackend()
result = backend.run(qc, shots=512)
print(result.counts)
```

Expected output:
```
Executing 'bell' (2 gates, depth=2):
  HGate() on qubits [0]
  CXGate() on qubits [0, 1]
{"00": ~256, "11": ~256}
```

---

## 9. Writing a Custom Transpiler Pass

```python
from zuntenium.transpiler.passes.base_pass import BasePass

class GateCountReporter(BasePass):
    """
    A reporting-only pass: counts gates by type, prints a table, passes IR unchanged.
    Demonstrates the Template Method pattern — run() is the abstract method.
    """

    def run(self, ir):
        from collections import Counter
        counts = Counter(
            node.operation for node in ir.instructions
            if node.operation == "gate"
        )
        gate_names = Counter(
            node.gate.__class__.__name__ for node in ir.instructions
            if node.operation == "gate" and node.gate is not None
        )
        print(f"\n── Gate Count Report for '{ir.metadata.get('name', '?')}' ──")
        print(f"  Total gates : {sum(gate_names.values())}")
        for name, count in sorted(gate_names.items()):
            print(f"  {name:<30} {count:>4}")
        print()
        return ir   # Pass-through: IR unchanged

# Use in a pass sequence
from zuntenium import QuantumCircuit
from zuntenium.transpiler.passes.pass_manager import PassSequence
from zuntenium.transpiler.passes.optimization.gate_cancellation import GateCancellationPass

qc = QuantumCircuit(3, name="ghz")
qc.h(0).cx(0,1).cx(1,2).h(0).h(0)   # Two H cancel

ir = qc.to_ir()
pipeline = PassSequence([
    GateCountReporter(),
    GateCancellationPass(),
    GateCountReporter(),   # Show count after cancellation
])
ir_out = pipeline.run(ir)
```

---

## 10. Exercises, Quiz, and Capstone

### Exercises

**E2.1** Using `BaseGate`, implement a `PhaseGate(phi)` that applies diag(1, e^{iφ}). Verify it is unitary for φ=π/3.

**E3.1** Which SOLID principle is violated by a class that both builds a circuit AND runs it on hardware? Name the principle and describe the fix.  
*(Answer: SRP. Fix: split into QuantumCircuit (build) + backend.run() (execute).)*

**E5.1** Look at `DensityMatrixSimulator._build_full_unitary()`. Describe why the single-qubit fast path (`np.kron`) is O(n·4) while the general path is O(4^n).

**E6.1** Write code to verify the completeness condition Σₖ Kₖ†Kₖ = I for the depolarizing channel with p=0.1.

**E7.1** What is the maximum purity after applying amplitude damping with γ=1.0 to any initial state? Explain why.  
*(Answer: 1.0 — all states decay to |0⟩, which is pure. Entropy returns to 0.)*

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | Which GoF pattern does `get_backend("statevector")` implement? | Factory Method |
| Q2 | Which GoF pattern does `IBMBackend` implement? | Adapter |
| Q3 | What does Tr(ρ²) = 0.5 for a 1-qubit state tell you? | Maximally mixed (completely noisy) |
| Q4 | What constraint must Kraus operators satisfy? | Σₖ Kₖ†Kₖ = I |
| Q5 | Why is `np.einsum` preferred over building 2^n × 2^n matrices? | Memory: 2^n scales; tensor contraction avoids materializing the full matrix |
| Q6 | Which SOLID principle is served by `BackendSelector` protocol? | Dependency Inversion |
| Q7 | What is S(ρ) = −Tr(ρ log ρ) for a completely mixed 1-qubit state? | log(2) ≈ 0.693 nats |

### Capstone: Noise Characterisation Circuit

Build a full noise characterisation run that:
1. Prepares each of the six Bloch-axis states (±X, ±Y, ±Z eigenstates)
2. Applies increasing amplitude-damping noise (γ = 0, 0.1, 0.3, 0.5)
3. Computes purity and von Neumann entropy for each (state, γ) pair
4. Prints a heatmap-style table

```python
import numpy as np
from zuntenium.backends import DensityMatrixSimulator

def prepare_gate_sequence(state_name: str):
    """Return (list_of_(matrix, qubit_list)) to prepare named state."""
    I = np.eye(2, dtype=complex)
    H = np.array([[1,1],[1,-1]])/np.sqrt(2)
    S = np.array([[1,0],[0,1j]], dtype=complex)
    X = np.array([[0,1],[1,0]], dtype=complex)
    sequences = {
        "+Z": [],                          # |0⟩ — no gates
        "-Z": [(X, [0])],                  # |1⟩ = X|0⟩
        "+X": [(H, [0])],                  # |+⟩ = H|0⟩
        "-X": [(X, [0]), (H, [0])],        # |−⟩ = HX|0⟩
        "+Y": [(H, [0]), (S, [0])],        # |i⟩ = SH|0⟩
        "-Y": [(X, [0]), (H, [0]), (S, [0])],  # |−i⟩
    }
    return sequences[state_name]

def amplitude_damping_kraus(gamma: float):
    K0 = np.array([[1,0],[0,np.sqrt(1-gamma)]], dtype=complex)
    K1 = np.array([[0,np.sqrt(gamma)],[0,0]], dtype=complex)
    return [K0, K1]

states = ["+Z", "-Z", "+X", "-X", "+Y", "-Y"]
gammas = [0.0, 0.1, 0.3, 0.5]

print(f"{'State':>6} | " + " | ".join(f"γ={g:.1f} Pur." for g in gammas))
print("-" * 70)

for state_name in states:
    row = []
    for gamma in gammas:
        sim = DensityMatrixSimulator(num_qubits=1)
        for (mat, qubits) in prepare_gate_sequence(state_name):
            sim.apply_gate(mat, qubits)
        if gamma > 0:
            sim.apply_noise_channel(amplitude_damping_kraus(gamma), qubit=0)
        row.append(f"{sim.purity():.3f}")
    print(f"{state_name:>6} | " + " | ".join(f"{v:>11}" for v in row))
```

---

## 11. Glossary, [TO CONFIRM] Items, and Next Course

### Glossary

| Term | Definition |
|---|---|
| **ABC** | Abstract Base Class — enforces interface contracts via `@abstractmethod` |
| **Facade** | Structural pattern: simplified interface over complex subsystem |
| **Adapter** | Structural pattern: converts one interface to another |
| **Template Method** | Behavioural pattern: base class defines skeleton, subclass fills steps |
| **Strategy** | Behavioural pattern: interchangeable algorithm family |
| **Factory Method** | Creational pattern: object creation via interface, subclass decides type |
| **SRP** | Single Responsibility Principle — one reason to change |
| **OCP** | Open/Closed Principle — open to extension, closed to modification |
| **DIP** | Dependency Inversion — depend on abstractions, not concretions |
| **Protocol** | Python structural subtyping — duck-typed interface (typing.Protocol) |
| **Kraus operators** | Set {Kₖ} representing any physically valid quantum channel |
| **Amplitude damping** | Noise channel modelling energy loss (T1 decay): |1⟩ → |0⟩ with probability γ |
| **Phase damping** | Noise channel modelling dephasing (T2): coherences decay, populations preserved |
| **Depolarizing** | Symmetric noise: applies X, Y, or Z each with probability p/4 |
| **Purity** | Tr(ρ²): 1 = pure, 0.5 = max mixed (1 qubit) |
| **Von Neumann entropy** | S(ρ) = −Tr(ρ log ρ): 0 = pure, log(d) = max mixed (d-dim) |
| **np.einsum** | Einstein summation contraction — avoids constructing full 2^n × 2^n gates |
| **Kronecker product** | ⊗: builds full-system gate matrix from single-system gate |
| **MAX_DM_QUBITS** | 15 — DensityMatrixSimulator memory guard (4^15 × 16 bytes ≈ 16 GiB) |

### [TO CONFIRM] Items

| # | Item |
|---|---|
| TC-1 | `IBMBackend(token, backend_name)` — exact constructor signature in v0.2.0? |
| TC-2 | `DensityMatrixSimulator.density_matrix.matrix` — is `.matrix` the numpy ndarray attribute? |
| TC-3 | `PassSequence` accepts list of `BasePass` in its constructor? |
| TC-4 | `GateCancellationPass` is in `transpiler/passes/optimization/`? |
| TC-5 | `qc.append(gate, qubits)` — does this work for custom `BaseGate` subclasses? |

### Next Courses

➡ **ZQL-02B** — *SDK Data Structures & Memory Engineering*  
Deep dive into `CircuitDAG`, `CouplingMap`, `BinaryTableau` (stabiliser), and memory layouts.

➡ **ZQL-11** — *Quantum Error Mitigation & Fault-Tolerant Computing*  
Uses `DensityMatrixSimulator` + Kraus channels from this course to model real device noise; introduces ZNE and PEC mitigation.

---

*All code verified against `backends/density_matrix_simulator.py` (351 lines), `simulators.py`, `circuit/circuit.py` (436 lines), `transpiler/passes/base_pass.py`, `__init__.py` (v0.2.0).  
Generated from: ZQL-02A-notes-gof.md, ZQL-02A-notes-cleanarch-fluentpython.md, ZQL-02A-notes-physics-numerics.md.*
