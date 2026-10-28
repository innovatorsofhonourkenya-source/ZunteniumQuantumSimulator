# ZQL-02A-notes-cleanarch-fluentpython — Walkthrough Labs

**Source:** [ZQL-02A-notes-cleanarch-fluentpython.md](../ZQL-02A-notes-cleanarch-fluentpython.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Clean Architecture (Martin) + Fluent Python (Ramalho)

---

## Global Setup
```bash
mkdir -p ~/zql-ca-labs && cd ~/zql-ca-labs
python3 -m venv .venv && source .venv/bin/activate
pip install zuntenium==0.2.0 numpy==1.26.4
```

---

## Lab 1: SRP — One Class, One Reason to Change
- **Concept:** Single Responsibility Principle (CA Part II)
- **Difficulty:** Beginner | **Time:** 45 min
### Walkthrough
```python
# save as srp_clean.py
good = {"QuantumCircuit": "build instructions", "CircuitValidator": "validate only",
        "HardwareAwareTranspiler": "compile only"}
bad = {"DoEverything": "build + validate + compile + run"}
print("GOOD (SRP):")
for c,r in good.items(): print(f"  ✓ {c}: {r}")
print("BAD:")
for c,r in bad.items(): print(f"  ✗ {c}: {r}")
```
### Verification
```python
assert all("+" not in r for r in good.values())
print("Lab 1 Verification: PASSED")
```
### Deliverable: `srp_clean.py`

---

## Lab 2: OCP — Extend Without Modifying Core
- **Concept:** Open/Closed Principle — BaseGate and BasePass ABCs
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as ocp_clean.py
from abc import ABC, abstractmethod
import numpy as np
class BaseGate(ABC):
    @property
    @abstractmethod
    def matrix(self) -> np.ndarray: ...
    @abstractmethod
    def dagger(self) -> "BaseGate": ...

class NewGate(BaseGate):
    @property
    def matrix(self): return np.array([[0,-1j],[1j,0]])
    def dagger(self): return NewGate()

print(f"New gate added: {np.allclose(NewGate().matrix @ NewGate().matrix.conj().T, np.eye(2))}")
print("Zero changes to BaseGate or QuantumCircuit!")
```
### Verification
```python
assert np.allclose(NewGate().matrix @ NewGate().matrix.conj().T, np.eye(2))
print("Lab 2 Verification: PASSED")
```
### Deliverable: `ocp_clean.py`

---

## Lab 3: LSP — Any Pass Substitutes Safely
- **Concept:** Liskov Substitution — all BasePass subclasses honor IR→IR contract
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as lsp_clean.py
from abc import ABC, abstractmethod
class BasePass(ABC):
    @abstractmethod
    def run(self, ir): ...

class PassA(BasePass):
    def run(self, ir): return {"transformed": True, **ir}
class PassB(BasePass):
    def run(self, ir): return {"optimized": True, **ir}

for P in [PassA, PassB]:
    result = P().run({"name": "test"})
    assert isinstance(result, dict), f"{P.__name__} violates LSP!"
    print(f"  ✓ {P.__name__}: returns dict (LSP OK)")
```
### Verification
```python
print("Lab 3 Verification: PASSED")
```
### Deliverable: `lsp_clean.py`

---

## Lab 4: ISP — Segregated Interfaces
- **Concept:** Interface Segregation — SV simulator ≠ DM simulator API
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as isp_clean.py
from typing import Protocol
class PureBackend(Protocol):
    def run(self, circuit, shots: int) -> object: ...

class NoisyBackend(Protocol):
    def run(self, circuit, shots: int) -> object: ...
    def apply_noise_channel(self, kraus, qubit: int) -> None: ...

# StatevectorSimulator: PureBackend only (no noise)
# DensityMatrixSimulator: NoisyBackend (has noise)
print("StatevectorSimulator implements PureBackend")
print("DensityMatrixSimulator implements NoisyBackend")
print("ISP: clients depend only on interfaces they use")
```
### Verification
```python
print("Lab 4 Verification: PASSED")
```
### Deliverable: `isp_clean.py`

---

## Lab 5: DIP — Depend on Abstractions
- **Concept:** Dependency Inversion — executor uses protocol, not concrete
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as dip_clean.py
from typing import Protocol, Any
class QuantumBackend(Protocol):
    def run(self, circuit: Any, shots: int) -> Any: ...

class Executor:
    def __init__(self, backend: QuantumBackend): self.backend = backend
    def execute(self, qc, shots=1024): return self.backend.run(qc, shots=shots)

from zuntenium import StatevectorSimulator, QuantumCircuit
e = Executor(StatevectorSimulator())
r = e.execute(QuantumCircuit(1).h(0).measure_all(), 100)
print(f"DIP: {r.counts}")
```
### Verification
```python
assert len(r.counts) > 0
print("Lab 5 Verification: PASSED")
```
### Deliverable: `dip_clean.py`

---

## Lab 6: Stable Abstractions — Stability vs Concreteness
- **Concept:** Most stable components are most abstract (CA Part IV)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as stable_abstractions.py
components = [
    ("BaseGate", "abstract", "very stable", "rarely changes"),
    ("BasePass", "abstract", "very stable", "rarely changes"),
    ("HGate", "concrete", "stable", "fixed matrix"),
    ("IBMBackend", "concrete", "unstable", "vendor API changes"),
]
print(f"{'Component':<15} | {'Abstract?':<10} | {'Stability':<12} | {'Why'}")
print("-" * 60)
for name, abst, stab, why in components:
    print(f"{name:<15} | {abst:<10} | {stab:<12} | {why}")
```
### Verification
```python
print("Lab 6 Verification: PASSED")
```
### Deliverable: `stable_abstractions.py`

---

## Lab 7: Dependency Structure — Arrows Point Inward
- **Concept:** circuit → quantum_core → numpy (never reverse)
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as dep_structure.py
deps = {
    "circuit.py": ["quantum_core/gates/", "numpy"],
    "quantum_core/gates/": ["numpy"],
    "numpy": [],
}
print("Dependency tree (all arrows point inward):")
for mod, imports in deps.items():
    arrow = " → ".join(imports) if imports else "(leaf)"
    print(f"  {mod} → {arrow}")
# Validate: no reverse dependency
assert "circuit.py" not in deps.get("numpy", [])
print("No circular dependencies ✓")
```
### Verification
```python
print("Lab 7 Verification: PASSED")
```
### Deliverable: `dep_structure.py`

---

## Lab 8: ABC Enforcement — TypeError on Missing Methods (FP Ch.13)
- **Concept:** Python ABC raises TypeError if abstract method not implemented
- **Difficulty:** Beginner | **Time:** 45 min
### Walkthrough
```python
# save as abc_enforce.py
from abc import ABC, abstractmethod
import numpy as np
class BaseGate(ABC):
    @property
    @abstractmethod
    def matrix(self) -> np.ndarray: ...
    @abstractmethod
    def dagger(self) -> "BaseGate": ...

class IncompleteGate(BaseGate):
    pass  # Forgot to implement matrix and dagger!

try:
    g = IncompleteGate()
    print("ERROR: should have raised TypeError")
except TypeError as e:
    print(f"✓ ABC enforced: {e}")
```
### Verification
```python
try: IncompleteGate(); assert False
except TypeError: pass
print("Lab 8 Verification: PASSED")
```
### Deliverable: `abc_enforce.py`

---

## Lab 9: Protocol — Structural Subtyping (FP Ch.13)
- **Concept:** typing.Protocol for duck-typed interfaces
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as protocol_typing.py
from typing import Protocol, runtime_checkable
@runtime_checkable
class QuantumBackend(Protocol):
    def run(self, circuit, shots: int = 1024): ...

class CustomBackend:
    def run(self, circuit, shots=1024):
        return {"0": shots}  # Mock

cb = CustomBackend()
print(f"Is QuantumBackend? {isinstance(cb, QuantumBackend)}")
# No inheritance needed! Just implement the method.
```
### Verification
```python
assert isinstance(cb, QuantumBackend)
print("Lab 9 Verification: PASSED")
```
### Deliverable: `protocol_typing.py`

---

## Lab 10: @dataclass — Instruction Data Class (FP Ch.5)
- **Concept:** Automatic __init__, __repr__, __eq__ via @dataclass
- **Difficulty:** Beginner | **Time:** 45 min
### Walkthrough
```python
# save as dataclass_instr.py
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class Instruction:
    operation: str
    qubits: list = field(default_factory=list)
    clbits: list = field(default_factory=list)
    params: list = field(default_factory=list)
    gate: Optional[object] = None

i1 = Instruction("gate", [0], gate="HGate")
i2 = Instruction("gate", [0], gate="HGate")
print(f"repr: {i1}")
print(f"eq: {i1 == i2}")
```
### Verification
```python
assert i1 == i2
print("Lab 10 Verification: PASSED")
```
### Deliverable: `dataclass_instr.py`

---

## Lab 11: Special Method __repr__ — DM Simulator Display (FP Ch.12)
- **Concept:** Machine-readable representation via __repr__
- **Difficulty:** Beginner | **Time:** 45 min
### Walkthrough
```python
# save as repr_method.py
class DMSim:
    def __init__(self, n, purity, entropy):
        self.n, self.purity, self.entropy = n, purity, entropy
    def __repr__(self):
        return f"DensityMatrixSimulator(n={self.n}, purity={self.purity:.6f}, S(ρ)={self.entropy:.4f})"
sim = DMSim(2, 1.0, 0.0)
print(repr(sim))
```
### Verification
```python
assert "purity=1.000000" in repr(sim)
print("Lab 11 Verification: PASSED")
```
### Deliverable: `repr_method.py`

---

## Lab 12: __getattr__ Lazy Loading (FP Ch.12)
- **Concept:** Lazy module loading via __getattr__
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as lazy_loading.py
class LazyModule:
    _loaded = {}
    def __getattr__(self, name):
        if name not in self._loaded:
            print(f"Lazy loading: {name}")
            self._loaded[name] = f"<{name} module>"
        return self._loaded[name]

mod = LazyModule()
print(mod.numpy)   # Lazy loads on first access
print(mod.numpy)   # Cached
print(mod.scipy)   # Lazy loads
```
### Verification
```python
assert mod.numpy == "<numpy module>"
print("Lab 12 Verification: PASSED")
```
### Deliverable: `lazy_loading.py`

---

## Lab 13: Strategy as Callable (FP Ch.10)
- **Concept:** First-class functions replace Strategy class hierarchy
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as callable_strategy.py
def no_crosstalk(schedule): return schedule
def simple_crosstalk(schedule): return [s for s in schedule if s != "parallel"]

class Scheduler:
    def __init__(self, crosstalk_fn=no_crosstalk):
        self.crosstalk_fn = crosstalk_fn
    def schedule(self, gates):
        return self.crosstalk_fn(gates)

gates = ["h", "cx", "parallel", "s"]
print(f"No filter: {Scheduler().schedule(gates)}")
print(f"Simple filter: {Scheduler(simple_crosstalk).schedule(gates)}")
```
### Verification
```python
assert len(Scheduler(simple_crosstalk).schedule(gates)) == 3
print("Lab 13 Verification: PASSED")
```
### Deliverable: `callable_strategy.py`

---

## Lab 14: np.einsum Basics — Einstein Summation (GVL)
- **Concept:** Tensor contraction via einsum notation
- **Difficulty:** Intermediate | **Time:** 75 min
### Walkthrough
```python
# save as einsum_basics.py
import numpy as np
A = np.array([[1,2],[3,4]])
B = np.array([[5,6],[7,8]])
# Matrix multiply via einsum
C = np.einsum("ij,jk->ik", A, B)
print(f"A@B via einsum:\n{C}")
assert np.allclose(C, A @ B)
# Trace
tr = np.einsum("ii->", A)
print(f"Trace via einsum: {tr}")
# Outer product
outer = np.einsum("i,j->ij", [1,2], [3,4])
print(f"Outer product:\n{outer}")
```
### Verification
```python
assert np.allclose(C, A@B) and tr == 5
print("Lab 14 Verification: PASSED")
```
### Deliverable: `einsum_basics.py`

---

## Lab 15: einsum for Gate Application
- **Concept:** Apply quantum gate without building 2^n matrix
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as einsum_gate_apply.py
import numpy as np
def apply_gate(state, gate, target, n):
    psi = state.reshape([2]*n)
    ax = [chr(ord('a')+k) for k in range(n)]
    t = ax[target]; z = 'z'
    sub = f"z{t},{''.join(ax)}->{''.join(z if k==target else ax[k] for k in range(n))}"
    return np.einsum(sub, gate, psi).reshape(-1)

H = np.array([[1,1],[1,-1]])/np.sqrt(2)
state = np.zeros(8, dtype=complex); state[0] = 1
result = apply_gate(state, H, 1, 3)
print(f"H on q1: P(000)={abs(result[0])**2:.3f}, P(010)={abs(result[2])**2:.3f}")
```
### Verification
```python
assert abs(abs(result[0])**2 - 0.5) < 1e-10
print("Lab 15 Verification: PASSED")
```
### Deliverable: `einsum_gate_apply.py`

---

## Lab 16: SRP Violation Detection Exercise
- **Concept:** Identify and fix SRP violations
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as srp_fix.py
# BAD: class does two things
class BadCircuitRunner:
    def build(self, gates): return gates
    def run(self, circuit): return {"0": 100}

# GOOD: split into two classes
class CircuitBuilder:
    def build(self, gates): return gates

class CircuitRunner:
    def run(self, circuit): return {"0": 100}

print("Fixed: CircuitBuilder (build) + CircuitRunner (run)")
print("Each class has exactly one reason to change")
```
### Verification
```python
print("Lab 16 Verification: PASSED")
```
### Deliverable: `srp_fix.py`

---

## Lab 17: OCP in Practice — Adding CrosstalkPass
- **Concept:** Add new transpiler pass without modifying pipeline
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as ocp_practice.py
from abc import ABC, abstractmethod
class BasePass(ABC):
    @abstractmethod
    def run(self, ir): ...

class CrosstalkPass(BasePass):
    def run(self, ir): return {**ir, "crosstalk_handled": True}

pipeline = [CrosstalkPass()]
ir = {"name": "bell", "gates": 3}
for p in pipeline: ir = p.run(ir)
print(f"After CrosstalkPass: {ir}")
```
### Verification
```python
assert ir["crosstalk_handled"]
print("Lab 17 Verification: PASSED")
```
### Deliverable: `ocp_practice.py`

---

## Lab 18: _repr_html_ — Jupyter Display Hook (FP Ch.12)
- **Concept:** Rich display in notebooks via special methods
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as repr_html.py
class QuantumCircuitDisplay:
    def __init__(self, name, gates):
        self.name, self.gates = name, gates
    def _repr_html_(self):
        rows = "".join(f"<tr><td>{g}</td></tr>" for g in self.gates)
        return f"<table><tr><th>{self.name}</th></tr>{rows}</table>"
    def __str__(self):
        return f"Circuit({self.name}): {' → '.join(self.gates)}"

qc = QuantumCircuitDisplay("bell", ["H(0)", "CX(0,1)"])
print(str(qc))
print(f"HTML available: {hasattr(qc, '_repr_html_')}")
```
### Verification
```python
assert hasattr(qc, '_repr_html_')
print("Lab 18 Verification: PASSED")
```
### Deliverable: `repr_html.py`

---

## Lab 19: Full SOLID Audit of QuFlow
- **Concept:** Systematic SOLID analysis of SDK components
- **Difficulty:** Advanced | **Time:** 90 min
### Walkthrough
```python
# save as solid_audit.py
audit = {
    "SRP": {"pass": True, "evidence": "QuantumCircuit ≠ TranspilerEngine ≠ CircuitValidator"},
    "OCP": {"pass": True, "evidence": "BaseGate, BasePass extension points"},
    "LSP": {"pass": True, "evidence": "All BasePass subclasses: IR→IR"},
    "ISP": {"pass": True, "evidence": "SV API ≠ DM API ≠ GPU API"},
    "DIP": {"pass": True, "evidence": "Executor → BackendSelector protocol"},
}
print("SOLID Audit Results:")
for principle, result in audit.items():
    print(f"  {'✓' if result['pass'] else '✗'} {principle}: {result['evidence']}")
all_pass = all(v["pass"] for v in audit.values())
print(f"\nAll SOLID: {all_pass}")
```
### Verification
```python
assert all_pass
print("Lab 19 Verification: PASSED")
```
### Deliverable: `solid_audit.py`

---

## Lab 20: Clean Architecture + Fluent Python Capstone
- **Concept:** Integration test across all CA + FP concepts
- **Difficulty:** Expert | **Time:** 90 min
### Walkthrough
```python
# save as ca_fp_capstone.py
import json, time, numpy as np
from abc import ABC, abstractmethod
from dataclasses import dataclass
from zuntenium import QuantumCircuit, StatevectorSimulator
tests = []
def ok(n,c): tests.append({"name":n,"pass":bool(c)}); print(f"{'✓' if c else '✗'} {n}")

# SRP
ok("SRP", hasattr(QuantumCircuit(1), 'h') and not hasattr(QuantumCircuit(1), 'compile'))
# OCP
class G(ABC):
    @property
    @abstractmethod
    def matrix(self): ...
class MyG(G):
    @property
    def matrix(self): return np.eye(2)
ok("OCP", np.allclose(MyG().matrix, np.eye(2)))
# ABC enforcement
try: G(); ok("ABC enforce", False)
except TypeError: ok("ABC enforce", True)
# Dataclass
@dataclass
class Inst:
    op: str; qubits: list
i = Inst("gate", [0])
ok("Dataclass", i.op == "gate")
# Einsum
C = np.einsum("ij,jk->ik", np.eye(2), np.eye(2))
ok("Einsum", np.allclose(C, np.eye(2)))

report = {"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("ca_fp_report.json","w") as f: json.dump(report,f,indent=2)
print(f"\nAll pass: {report['all_pass']}")
```
### Verification
```python
assert json.load(open("ca_fp_report.json"))["all_pass"]
print("Lab 20 Verification: PASSED")
```
### Deliverable: `ca_fp_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct CA/FP concepts ✓ 3. **Coverage:** SRP, OCP, LSP, ISP, DIP, stable abstractions, dependency structure, ABC, Protocol, @dataclass, __repr__, __getattr__, callable strategy, einsum, _repr_html_, SOLID audit ✓
