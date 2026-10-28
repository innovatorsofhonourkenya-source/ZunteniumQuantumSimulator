# ZQL-02A Step 1 Notes — Clean Architecture (Martin, 2017) + Fluent Python (Ramalho, 2022)

---

# A. Clean Architecture — Robert C. Martin ("Uncle Bob")

## Part II — SOLID Principles

### Single Responsibility (SRP)
- A module should have one, and only one, reason to change.
- **[SDK]** `QuantumCircuit` = circuit builder only. `CircuitValidator` = validation only. `HardwareAwareTranspiler` = compilation only. No class does both.
- Violation example: old code had `QuantumCircuit.compile()` calling hardware routing — now split into separate `TranspilerEngine`.

### Open/Closed (OCP)
- Open for extension, closed for modification.
- **[SDK]** `BaseGate` ABC: adding a new gate (e.g., `SqrtXGate`) requires zero changes to `QuantumCircuit`. Add the gate class, register it—nothing else breaks.
- **[SDK]** `BasePass` ABC: new transpiler pass (e.g., `CrosstalkSchedulingPass`) adds without modifying the pipeline runner.

### Liskov Substitution (LSP)
- Subtypes must be substitutable for their base types without altering correctness.
- **[SDK]** Any `BasePass` subclass must: accept `QuantumIR` → return `QuantumIR`. `VF2PlacementPass`, `ALAPSchedulingPass`, `CrosstalkSchedulingPass` all satisfy this contract.

### Interface Segregation (ISP)
- Clients should not depend on interfaces they don't use.
- **[SDK]** `StatevectorSimulator` (CPU) vs `GPUStatevectorSimulator`: each implements only the methods relevant to its capability. GPU backend does not expose noise channel API (noise is density-matrix territory).

### Dependency Inversion (DIP)
- High-level modules should not depend on low-level modules; both depend on abstractions.
- **[SDK]** `AdaptiveQuantumExecutor` depends on `BackendSelector` protocol, not on `IBMBackend` concretely. Swap IBM for IonQ → executor unchanged.

## Part IV — Component Principles

### Stable Abstractions Principle
- A component should be as abstract as it is stable.
- **[SDK]** `BaseGate`, `BasePass` are the most stable (rarely change) and the most abstract (no concrete implementation). `IBMBackend` is concrete and less stable (changes with vendor API updates).

### Dependency Structure
- Dependency arrows must always point toward stability.
- **[SDK]** `circuit.py` → `quantum_core/gates/` → `numpy`. The quantum_core never imports from circuit. The circuit layer never imports transpiler internals.

---

# B. Fluent Python — Luciano Ramalho (2nd ed., 2022)

## Ch. 13 — Interfaces, Protocols, ABCs

### Abstract Base Classes
- `from abc import ABC, abstractmethod`
- Python ABCs enforce interface: unimplemented abstract methods → TypeError at instantiation.
- **[SDK]** `BaseGate(ABC)` with `@abstractmethod matrix()` and `@abstractmethod dagger()`. Every gate subclass is forced to provide these.

```python
from abc import ABC, abstractmethod
import numpy as np

class BaseGate(ABC):
    @property
    @abstractmethod
    def matrix(self) -> np.ndarray: ...

    @abstractmethod
    def dagger(self) -> "BaseGate": ...

    @property
    def params(self) -> list[float]:
        return []
```

### Protocols (Structural Subtyping)
- `typing.Protocol`: duck-typed interface. No inheritance required.
- **[SDK]** `QuantumBackend` is a protocol: any object with `.run(circuit, shots) -> ExecutionResult` qualifies. Enables custom backends without inheriting from a base class.

## Ch. 5 — Data Class Builders

### @dataclass
- Generates `__init__`, `__repr__`, `__eq__` automatically.
- **[SDK]** `Instruction` is a dataclass:
```python
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class Instruction:
    operation: str
    qubits: list[int] = field(default_factory=list)
    clbits: list[int] = field(default_factory=list)
    params: list[float] = field(default_factory=list)
    gate: Optional[object] = None
```

## Ch. 12 — Special Methods

- `__repr__`: machine-readable repr. `DensityMatrixSimulator.__repr__` shows purity and entropy.
- `__str__`: human-readable. `QuantumCircuit.__str__` → calls `draw()` → ASCII diagram.
- `_repr_html_`: Jupyter-display hook. `QuantumCircuit._repr_html_()` renders dark-mode HTML diagram.
- `__getattr__`: lazy module loading. `zuntenium.__getattr__` defers heavy numpy imports until first use.

## Ch. 10 — Design Patterns with First-Class Functions

### Strategy as Callable
- Instead of Strategy class hierarchy, use plain functions (first-class objects).
- **[SDK]** The `crosstalk_fn` parameter in `CrosstalkSchedulingPass` is an injectable callable strategy — no need for a `CrosstalkStrategy` ABC.

## np.einsum — Tensor Contraction (Golub & Van Loan complement)
- `np.einsum("ij,jk->ik", A, B)` = matrix multiply A@B.
- For gate application: `np.einsum("ab,bc->ac", gate_reshaped, state_reshaped)` contracts gate indices onto qubit axes of statevector.
- **[SDK]** `GateApplicator` in `quantum_core/statevector/` uses einsum for multi-qubit gate application without building the full 2^n × 2^n matrix.

---

## SOLID → SDK Mapping Summary

| Principle | QuFlow implementation |
|---|---|
| SRP | `QuantumCircuit` ≠ `TranspilerEngine` ≠ `CircuitValidator` |
| OCP | `BaseGate`, `BasePass` extension points |
| LSP | All `BasePass` subclasses honour `QuantumIR → QuantumIR` contract |
| ISP | Statevector API ≠ density-matrix API ≠ GPU API |
| DIP | `AdaptiveQuantumExecutor` → `BackendSelector` protocol |
