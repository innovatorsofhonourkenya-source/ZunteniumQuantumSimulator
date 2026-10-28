# ZQL-02A Step 1 Notes — Gang of Four, *Design Patterns* (Gamma, Helm, Johnson & Vlissides, 1994)

**Patterns directly implemented in QuFlow SDK.**

---

## Creational Patterns

### Factory Method (GoF §3.3)
- Intent: define interface for creating an object; subclasses decide which class to instantiate.
- **[SDK]** `get_backend(name: str)` in `zuntenium.api.quantum_api` — caller requests "statevector", "density_matrix", etc.; factory returns the correct concrete backend.
```python
from zuntenium.api.quantum_api import get_backend
backend = get_backend("statevector")   # returns StatevectorBackendRunner
dm_backend = get_backend("density_matrix")  # returns DensityMatrixSimulator wrapper
```

### Singleton / Registry (GoF §3.5)
- Intent: ensure one instance; provide global access point.
- **[SDK]** `BackendRegistry` — `register(name, backend_cls)` and `get(name)`. One registry instance manages all available backends.

---

## Structural Patterns

### Facade (GoF §4.5)
- Intent: provide simplified interface to a complex subsystem.
- **[SDK]** `StatevectorSimulator` is the facade: inside, it calls `get_backend("statevector")`, which calls `quantum_core.statevector.StatevectorEngine`, which calls `GateApplicator`, which calls `np.einsum`. Caller just writes `backend.run(qc, shots=1024)`.

### Adapter (GoF §4.1)
- Intent: convert the interface of a class into another interface clients expect.
- **[SDK]** `IBMBackend`, `IonQAdapter` — each wraps the vendor-specific REST API in the standard `QuantumBackend.run(circuit, shots)` interface. Adapter pattern enables MHOS to treat all hardware uniformly.

---

## Behavioural Patterns

### Template Method (GoF §5.10)
- Intent: define skeleton of algorithm in base class; subclasses fill in specific steps.
- **[SDK]** `BasePass` (transpiler) — defines `run(circuit)` interface. Each pass (VF2PlacementPass, SABRERouterV2, ALAPSchedulingPass) overrides the body while preserving the pipeline contract (pre/post conditions).

### Strategy (GoF §5.9)
- Intent: define a family of algorithms; encapsulate each; make them interchangeable.
- **[SDK]** `AdaptiveQuantumExecutor` uses `BackendSelector` strategy to pick the simulator/hardware target at runtime. Swappable without changing the executor.

### Observer / Event (GoF §5.7)
- Intent: define one-to-many dependency so state change notifies all dependants.
- **[SDK analogy]** OTel instrumentation (OpenTelemetry spans) emits events when pass manager runs each pass — downstream metrics consumers observe without the pass knowing about them.

### Chain of Responsibility (GoF §5.1)
- Intent: pass request along chain of handlers; each handler decides to handle or pass on.
- **[SDK]** `PassSequence` / `PassManager` — transpiler pipeline. Each pass runs, transforms the circuit, and passes the result to the next. If a pass is a no-op (pass predicate false), it forwards unchanged.

---

## Mapping Summary

| GoF Pattern | QuFlow Component | File |
|---|---|---|
| Factory Method | `get_backend()` | `api/quantum_api.py` |
| Registry / Singleton | `BackendRegistry` | `transpiler/backend_registry.py` |
| Facade | `StatevectorSimulator` | `simulators.py` |
| Adapter | `IBMBackend`, `IonQAdapter` | `backends/ibm_backend.py`, `transpiler/backends/ionq_adapter.py` |
| Template Method | `BasePass.run()` | `transpiler/passes/base_pass.py` |
| Strategy | `BackendSelector` | `execution/` |
| Chain of Responsibility | `PassSequence` | `transpiler/passes/pass_manager.py` |
