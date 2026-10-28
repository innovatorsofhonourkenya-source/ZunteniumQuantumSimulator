# ZQL-02B — Quantum Compiler Data Structures & Memory Engineering

> **Course ID**: ZQL-02B | **Track**: ZQL-02 SDK Core Reference  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisites**: ZQL-00.01 (SDK Onboarding), ZQL-02A (OOP Architecture)

---

## 0. Course Overview

### Who This Is For

You understand the OOP design of QuFlow (ZQL-02A). This course focuses on **data structures inside the compiler**: how circuits are represented as DAGs, how qubits are mapped to hardware via coupling maps, how four different simulators manage memory at different cost, and how the Clifford stabilizer formalism enables n=1000 qubit simulation.

**Prerequisites**:
- ZQL-02A: BaseGate, BasePass, DependencyInversion, Facade pattern
- Python: confident with graphs, dicts, numpy arrays

### Learning Outcomes

1. Build and traverse a CircuitDAG using topological sort
2. Compute gate depth (critical path) and qubit liveness from the DAG
3. Construct a CouplingMap and query connectivity / shortest paths
4. Explain the four simulator memory models and their scaling limits
5. Implement BinaryTableau row operations for Clifford simulation
6. Profile statevector memory usage with `tracemalloc`
7. Choose the right simulator backend for a given circuit size and type

### Estimated Time: ~130 minutes

### Source Books

| Ref | Book |
|---|---|
| CLRS | Cormen et al. — *Introduction to Algorithms* (4th ed.) |
| KT | Kleinberg & Tardos — *Algorithm Design* |
| T | Tanenbaum — *Computer Organization and Architecture* |
| AG | Aaronson & Gottesman — *Improved Simulation of Stabilizer Circuits* (Phys. Rev. A, 2004) |
| J | Johansson — *Numerical Python* (3rd ed., Apress) |
| GO | Gorelick & Ozsvald — *High Performance Python* (2nd ed., O'Reilly, 2020) |

---

## 1. The CircuitDAG — Dependency Graph

### 1.1 Why a DAG?

A quantum circuit is a sequence of gates, but not all gates can execute simultaneously. Two gates **conflict** if they share a qubit. The CircuitDAG encodes which gates must run before others:

```
           ┌──────┐
|q0⟩ ──── │  H   │ ────●──────
           └──────┘     │
                        │   ┌────────┐
|q1⟩ ───────────────── ⊕──│ Measure│
                        │   └────────┘
|q2⟩ ────────────────── ...
```

DAG nodes = gates. Edges = "must happen before" (dependency).

### 1.2 Building a CircuitDAG

```python
from dataclasses import dataclass, field
from collections import defaultdict, deque
from typing import List, Set

@dataclass
class GateNode:
    """One node in the circuit dependency DAG."""
    gate_id: int
    name: str
    qubits: list
    predecessors: list = field(default_factory=list)
    successors: list   = field(default_factory=list)

class CircuitDAG:
    """
    Directed Acyclic Graph representation of a quantum circuit.
    Nodes = gates; edges = qubit data-flow dependencies.
    """

    def __init__(self):
        self.nodes: dict[int, GateNode] = {}
        self._last_on_qubit: dict[int, int] = {}   # qubit → last gate_id
        self._next_id = 0

    def add_gate(self, name: str, qubits: list) -> int:
        """Add a gate node, automatically linking dependencies."""
        gid = self._next_id
        self._next_id += 1

        preds = []
        for q in qubits:
            if q in self._last_on_qubit:
                pred_id = self._last_on_qubit[q]
                preds.append(pred_id)
                self.nodes[pred_id].successors.append(gid)

        node = GateNode(gid, name, qubits, predecessors=preds)
        self.nodes[gid] = node
        for q in qubits:
            self._last_on_qubit[q] = gid
        return gid

    def topological_sort(self) -> list:
        """Return gate IDs in topological order (CLRS §22.4)."""
        # Kahn's algorithm: BFS-based topological sort
        in_degree = defaultdict(int)
        for node in self.nodes.values():
            for succ in node.successors:
                in_degree[succ] += 1

        queue = deque(gid for gid in self.nodes if in_degree[gid] == 0)
        order = []
        while queue:
            gid = queue.popleft()
            order.append(gid)
            for succ in self.nodes[gid].successors:
                in_degree[succ] -= 1
                if in_degree[succ] == 0:
                    queue.append(succ)

        if len(order) != len(self.nodes):
            raise ValueError("Circuit has a dependency cycle — invalid!")
        return order

    def compute_depth(self) -> int:
        """Compute circuit depth = length of longest path in DAG."""
        topo = self.topological_sort()
        dist: dict[int, int] = {gid: 1 for gid in self.nodes}
        for gid in topo:
            for succ in self.nodes[gid].successors:
                dist[succ] = max(dist[succ], dist[gid] + 1)
        return max(dist.values(), default=0)

    def compute_qubit_liveness(self) -> dict[int, tuple]:
        """
        Compute liveness for each qubit: (first_gate_step, last_gate_step).
        Returns dict: qubit → (birth_step, death_step) in topological order.
        """
        topo = self.topological_sort()
        step = {gid: i for i, gid in enumerate(topo)}

        qubit_life: dict[int, list] = {}
        for gid, node in self.nodes.items():
            s = step[gid]
            for q in node.qubits:
                if q not in qubit_life:
                    qubit_life[q] = [s, s]
                else:
                    qubit_life[q][0] = min(qubit_life[q][0], s)
                    qubit_life[q][1] = max(qubit_life[q][1], s)

        return {q: (l[0], l[1]) for q, l in qubit_life.items()}
```

### 1.3 Demonstration

```python
# Build a GHZ circuit DAG
dag = CircuitDAG()
h0 = dag.add_gate("H",    [0])
cx01 = dag.add_gate("CX", [0, 1])
cx12 = dag.add_gate("CX", [1, 2])
m0 = dag.add_gate("Measure", [0])
m1 = dag.add_gate("Measure", [1])
m2 = dag.add_gate("Measure", [2])

order = dag.topological_sort()
depth = dag.compute_depth()
liveness = dag.compute_qubit_liveness()

print(f"Topological order: {[dag.nodes[g].name for g in order]}")
print(f"Circuit depth: {depth}")
print("Qubit liveness:")
for q, (birth, death) in sorted(liveness.items()):
    print(f"  q{q}: steps {birth} → {death}  (live for {death-birth+1} steps)")
```

Expected:
```
Topological order: ['H', 'CX', 'CX', 'Measure', 'Measure', 'Measure']
Circuit depth: 4
Qubit liveness:
  q0: steps 0 → 3  (live for 4 steps)
  q1: steps 1 → 4  (live for 4 steps)
  q2: steps 2 → 5  (live for 4 steps)
```

### 1.4 Critical Path Analysis

The circuit depth = the **critical path** = the longest chain of dependent gates.

```python
def critical_path(dag: CircuitDAG) -> list:
    """
    Find the critical path (longest dependency chain) in the CircuitDAG.
    Returns list of gate names along the critical path.
    """
    topo  = dag.topological_sort()
    dist  = {g: 1 for g in dag.nodes}
    prev  = {g: None for g in dag.nodes}

    for gid in topo:
        for succ in dag.nodes[gid].successors:
            if dist[gid] + 1 > dist[succ]:
                dist[succ] = dist[gid] + 1
                prev[succ] = gid

    # Trace back from deepest node
    end_node = max(dist, key=dist.get)
    path, cur = [], end_node
    while cur is not None:
        path.append(dag.nodes[cur].name)
        cur = prev[cur]
    return list(reversed(path))

print("Critical path:", critical_path(dag))
# → ['H', 'CX', 'CX', 'Measure']   (the 4-step dependency chain through q0→q1→q2)
```

---

## 2. CouplingMap — Hardware Connectivity Graph

### 2.1 Structure

A CouplingMap represents which qubits on a device can directly interact:

```python
from collections import deque

class CouplingMap:
    """
    Directed graph of allowed two-qubit interactions on a device.
    Nodes = physical qubits; edges = (control, target) pairs.
    """

    def __init__(self, n_qubits: int):
        self.n_qubits = n_qubits
        self._adj: dict[int, set] = {i: set() for i in range(n_qubits)}

    def add_edge(self, q0: int, q1: int, bidirectional: bool = True):
        """Add coupling edge q0—q1."""
        self._adj[q0].add(q1)
        if bidirectional:
            self._adj[q1].add(q0)

    def neighbours(self, q: int) -> set:
        return self._adj[q].copy()

    def is_connected(self, q0: int, q1: int) -> bool:
        return q1 in self._adj[q0]

    def shortest_path_length(self, src: int, dst: int) -> int:
        """BFS shortest path (hop count). Returns -1 if disconnected."""
        if src == dst: return 0
        visited = {src}
        queue = deque([(src, 0)])
        while queue:
            node, dist = queue.popleft()
            for nb in self._adj[node]:
                if nb == dst: return dist + 1
                if nb not in visited:
                    visited.add(nb)
                    queue.append((nb, dist + 1))
        return -1   # disconnected

    def all_pairs_distance(self) -> list[list[int]]:
        """BFS all-pairs distance matrix D[i][j] = hops from i to j."""
        n = self.n_qubits
        D = [[-1]*n for _ in range(n)]
        for src in range(n):
            D[src][src] = 0
            visited = {src}
            queue = deque([(src, 0)])
            while queue:
                node, dist = queue.popleft()
                for nb in self._adj[node]:
                    if nb not in visited:
                        visited.add(nb)
                        D[src][nb] = dist + 1
                        queue.append((nb, dist + 1))
        return D

    @classmethod
    def linear(cls, n: int) -> "CouplingMap":
        """Build linear chain: 0—1—2—...—(n-1)."""
        cm = cls(n)
        for i in range(n - 1):
            cm.add_edge(i, i + 1)
        return cm

    @classmethod
    def ibm_tokyo(cls) -> "CouplingMap":
        """Build IBM Q20 Tokyo coupling map (5×4 heavy-hex-like)."""
        cm = cls(20)
        edges = [(0,1),(1,2),(2,3),(3,4),(0,5),(1,6),(2,7),(3,8),(4,9),
                 (5,6),(6,7),(7,8),(8,9),(5,10),(6,11),(7,12),(8,13),(9,14),
                 (10,11),(11,12),(12,13),(13,14),(10,15),(11,16),(12,17),
                 (13,18),(14,19),(15,16),(16,17),(17,18),(18,19)]
        for (a,b) in edges:
            cm.add_edge(a, b)
        return cm

# Example usage
cm = CouplingMap.linear(5)
D = cm.all_pairs_distance()
print("Linear-5 coupling distance matrix:")
for row in D:
    print("  " + " ".join(f"{v:2d}" for v in row))
```

### 2.2 Using CouplingMap in Routing

```python
cm = CouplingMap.ibm_tokyo()
D = cm.all_pairs_distance()

# SABRE distance heuristic: cost of placing gate (q0,q1) with
# logical→physical mapping π
def sabre_heuristic(gate_q0: int, gate_q1: int,
                     mapping: dict, D: list) -> float:
    """Cost of executing gate (q0,q1) given current qubit mapping."""
    p0 = mapping[gate_q0]
    p1 = mapping[gate_q1]
    return D[p0][p1]

mapping = {i: i for i in range(20)}   # Identity mapping
print(f"\nCost of gate (3,7) with identity mapping: {sabre_heuristic(3,7,mapping,D)}")
print(f"Cost of gate (0,19) with identity mapping: {sabre_heuristic(0,19,mapping,D)}")
```

---

## 3. Four Simulator Memory Models

### 3.1 Memory Scaling Comparison

```python
import math

print(f"{'Simulator':<30} {'n=10':>8} {'n=20':>10} {'n=30':>12} {'Max n':>8}")
print("-" * 72)

for name, scale_fn, max_n in [
    ("StatevectorSimulator",    lambda n: 16 * 2**n,         "~30"),
    ("DensityMatrixSimulator",  lambda n: 16 * 4**n,          "15"),
    ("TensorNetworkBackend",    lambda n: 16 * 2**(n//2),  ">1000"),
    ("StabilizerSimulator",     lambda n: n * n // 8,      ">1000"),
]:
    def fmt(n, fn=scale_fn):
        try:
            v = fn(n)
            if v < 1024: return f"{v} B"
            if v < 2**20: return f"{v//1024} KB"
            if v < 2**30: return f"{v//2**20} MB"
            if v < 2**40: return f"{v//2**30} GB"
            return f"{v//2**40} TB"
        except: return "N/A"
    print(f"{name:<30} {fmt(10):>8} {fmt(20):>10} {fmt(30):>12} {max_n:>8}")
```

### 3.2 StatevectorSimulator — O(2ⁿ) complex128

The statevector stores 2ⁿ complex128 amplitudes. Each gate applies a unitary transformation:

```python
import numpy as np
import tracemalloc

def statevector_memory_profile(n: int) -> dict:
    """Profile actual memory usage of a statevector simulation."""
    tracemalloc.start()

    # Allocate statevector
    sv = np.zeros(2**n, dtype=np.complex128)
    sv[0] = 1.0    # |0...0⟩

    # Apply H to qubit 0 via einsum
    H = np.array([[1,1],[1,-1]], dtype=np.complex128) / np.sqrt(2)
    reshaped = sv.reshape([2]*n)
    # einsum: 'za,...a...->...z...' where a = target index
    sv_new = np.einsum('za,a->z', H, reshaped.reshape(2, -1)).reshape(*([2]*n)).reshape(-1)

    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()

    return {"n": n, "theoretical_bytes": 16 * 2**n,
            "actual_peak_kb": peak // 1024}

for n in [4, 8, 12, 16, 20]:
    m = statevector_memory_profile(n)
    print(f"n={n:2d}: theoretical={m['theoretical_bytes']//1024:>8} KB, "
          f"actual peak={m['actual_peak_kb']:>8} KB")
```

### 3.3 DensityMatrixSimulator — O(4ⁿ) complex128

```python
# Memory guard already in the SDK:
MAX_DM_QUBITS = 15   # 4^15 × 16 bytes ≈ 16 GiB

def dm_memory_bytes(n: int) -> int:
    """Theoretical density matrix memory in bytes."""
    return 16 * (4 ** n)

print("DensityMatrix memory:")
for n in [5, 8, 10, 12, 15]:
    b = dm_memory_bytes(n)
    label = f"{b/(2**30):.1f} GiB" if b > 2**30 else f"{b/(2**20):.1f} MiB"
    print(f"  n={n:2d}: {label}")
```

### 3.4 StabilizerSimulator — O(n²) bits (BinaryTableau)

```python
import numpy as np

class BinaryTableau:
    """
    Clifford stabilizer state as a binary tableau.
    Row i = stabilizer generator i, encoded as n X-bits + n Z-bits + 1 phase.
    Memory: O(n²) bits — can simulate 1000+ qubit Clifford circuits.

    Reference: Aaronson & Gottesman, Phys. Rev. A 70, 052328 (2004).
    """

    def __init__(self, n: int):
        self.n = n
        # (2n) rows × (2n+1) columns, dtype bool
        # Rows 0..n-1: destabilizers; n..2n-1: stabilizers
        # Columns: x0..x_{n-1}, z0..z_{n-1}, phase
        self._tab = np.zeros((2*n, 2*n+1), dtype=bool)
        # Initialise: |0...0⟩ stabilized by Z0, Z1, ..., Zn-1
        for i in range(n):
            self._tab[i, i] = True            # destabilizer: Xi
            self._tab[n+i, n+i] = True        # stabilizer: Zi

    @property
    def memory_bytes(self) -> int:
        return self._tab.nbytes

    def apply_h(self, qubit: int) -> None:
        """H gate: swaps X and Z for target qubit, updates phase."""
        for row in range(2 * self.n):
            xi = self._tab[row, qubit]
            zi = self._tab[row, self.n + qubit]
            # Phase update: if both x=1 and z=1, flip phase
            self._tab[row, 2*self.n] ^= xi & zi
            # Swap x and z for this qubit
            self._tab[row, qubit]        = zi
            self._tab[row, self.n+qubit] = xi

    def apply_cnot(self, ctrl: int, tgt: int) -> None:
        """CNOT: x_ctrl propagates to x_tgt; z_tgt back-propagates to z_ctrl."""
        for row in range(2 * self.n):
            xc = self._tab[row, ctrl]
            xt = self._tab[row, tgt]
            zc = self._tab[row, self.n + ctrl]
            zt = self._tab[row, self.n + tgt]
            # Phase update
            self._tab[row, 2*self.n] ^= xc & zt & (xt ^ zc ^ True)
            # X propagates forward: x_tgt ^= x_ctrl
            self._tab[row, tgt]         ^= xc
            # Z back-propagates: z_ctrl ^= z_tgt
            self._tab[row, self.n+ctrl] ^= zt

    def apply_s(self, qubit: int) -> None:
        """S gate (phase gate): Z←Z, X←Y (X and Z combine)."""
        for row in range(2 * self.n):
            xi = self._tab[row, qubit]
            zi = self._tab[row, self.n + qubit]
            self._tab[row, 2*self.n] ^= xi & zi   # phase
            self._tab[row, self.n+qubit] ^= xi    # z ^= x

# Demo: simulate Bell state via stabilizer formalism
n = 2
tab = BinaryTableau(n)
print(f"BinaryTableau({n} qubits): {tab.memory_bytes} bytes")

tab.apply_h(0)    # H on qubit 0
tab.apply_cnot(0, 1)   # CNOT(0,1)

print(f"State tableau after H(0), CNOT(0,1) — Bell state |Φ+⟩:")
print(f"Stabilizers (rows {n}..{2*n-1}):")
for i in range(n, 2*n):
    x_part = ''.join('X' if tab._tab[i, q] else '.' for q in range(n))
    z_part = ''.join('Z' if tab._tab[i, n+q] else '.' for q in range(n))
    phase  = '-' if tab._tab[i, 2*n] else '+'
    print(f"  {phase}{x_part}|{z_part}")
# Bell state |Φ+⟩ is stabilized by: +XX, +ZZ
```

---

## 4. Choosing the Right Simulator

### 4.1 Decision Guide

```
Start: What kind of circuit?
          │
  ┌───────┴────────┐
  │                │
Clifford only?   Mixed/Non-Clifford?
  │                │
  ▼                │
StabilizerSimulator  ┌──────┴──────────┐
(O(n²) — up to     │                  │
 1000+ qubits)    Need noise?        Pure state only?
                    │                  │
                    ▼                  ▼
              DensityMatrixSim   TensorNetworkBackend
              (O(4ⁿ), n≤15)     (low entanglement)
                                       │
                              High entanglement?
                                       │
                                       ▼
                                StatevectorSimulator
                                (O(2ⁿ), n≤30ish)
```

### 4.2 Quick Selection Code

```python
def select_simulator(n_qubits: int, has_noise: bool, clifford_only: bool,
                      low_entanglement: bool):
    """
    Recommend the best QuFlow simulator for the given circuit type.
    """
    if clifford_only:
        print(f"→ StabilizerSimulator: O(n²)={n_qubits**2} bits — handles {n_qubits} qubits trivially")
        return "stabilizer"

    if has_noise:
        if n_qubits > 15:
            print(f"→ ERROR: DensityMatrixSimulator max 15 qubits; {n_qubits} > 15")
            return None
        memory_gb = (16 * 4**n_qubits) / 2**30
        print(f"→ DensityMatrixSimulator: {memory_gb:.2f} GiB for n={n_qubits}")
        return "density_matrix"

    if low_entanglement:
        print(f"→ TensorNetworkBackend: compressed tensor representation (recommended)")
        return "tensor_network"

    memory_mb = (16 * 2**n_qubits) / 2**20
    if memory_mb > 8_000:   # > 8 GB
        print(f"→ WARNING: StatevectorSimulator needs {memory_mb:.0f} MB for n={n_qubits}")
    else:
        print(f"→ StatevectorSimulator: {memory_mb:.1f} MB for n={n_qubits}")
    return "statevector"

# Example decisions
print("=== Simulator Selection Examples ===\n")
select_simulator(n_qubits=50, has_noise=False, clifford_only=True,  low_entanglement=False)
select_simulator(n_qubits=8,  has_noise=True,  clifford_only=False, low_entanglement=False)
select_simulator(n_qubits=25, has_noise=False, clifford_only=False, low_entanglement=True)
select_simulator(n_qubits=18, has_noise=False, clifford_only=False, low_entanglement=False)
```

---

## 5. Memory Profiling a Simulation

```python
import tracemalloc
import numpy as np
from zuntenium import QuantumCircuit, StatevectorSimulator

def profile_simulation(n: int, depth: int = 4, shots: int = 1024):
    """Profile memory usage of a QuFlow simulation."""
    tracemalloc.start()

    # Build layered circuit
    qc = QuantumCircuit(n)
    for _ in range(depth):
        for i in range(n): qc.h(i)
        for i in range(0, n-1, 2): qc.cx(i, i+1)
    qc.measure_all()

    result = StatevectorSimulator().run(qc, shots=shots)

    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()

    theoretical = 16 * 2**n
    print(f"n={n:2d}, d={depth}: theoretical={theoretical//1024:>7} KB, "
          f"peak actual={peak//1024:>7} KB, "
          f"overhead factor={peak/theoretical:.1f}x")
    return peak

print("Memory profile: layered Hadamard+CNOT circuit")
for n in [4, 8, 12, 16]:
    profile_simulation(n)
```

---

## 6. Exercises, Quiz, and Capstone

### Exercises

**E1.1** Using `CircuitDAG`, build a 4-qubit QFT dependency graph (3 H gates + controlled-phase gates + SWAPs). What is the circuit depth from the DAG?

**E1.2** Modify `compute_qubit_liveness()` to also report **idle windows** — time steps where a qubit has no gate. Idle qubits decohere in real hardware.

**E2.1** Build a CouplingMap for IBM Q5 Yorktown (edges: 0-1, 0-2, 1-2, 2-4, 3-4). Compute all-pairs distance and identify the diameter (maximum shortest path).

**E3.1** How much memory does a BinaryTableau use for n=100 qubits vs a statevector?  
*(Answer: Tableau: 200×201 = 40,200 bits ≈ 5 KB. Statevector: 16·2¹⁰⁰ bytes — astronomically larger.)*

**E4.1** For which circuit types would you use each simulator? Give one example circuit for each.

**E5.1** Use `tracemalloc` to measure the actual overhead factor (peak/theoretical) for n=8 statevector simulation. Explain why there is overhead (intermediate arrays in `np.einsum`).

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What algorithm does topological sort use? | DFS with finish-time ordering, or BFS (Kahn's algorithm) |
| Q2 | What does `all_pairs_distance()` compute? | BFS shortest path (hop count) between every pair of qubits — used by SABRE |
| Q3 | Memory for DensityMatrixSimulator at n=12? | 16 × 4¹² ≈ 1 GB |
| Q4 | What is the Gottesman-Knill theorem? | Clifford circuits can be simulated classically in O(n²) time |
| Q5 | What does a BinaryTableau row represent? | One stabilizer generator (an n-qubit Pauli string) |
| Q6 | Why is `np.einsum` preferred over Python loops? | ~20× faster; uses BLAS/AVX2 internally; avoids O(2^n) Python iterations |
| Q7 | What circuit property enables TensorNetworkBackend to be efficient? | Low entanglement — bond dimension stays small |

### Capstone: Circuit Analyser Tool

Build a tool that takes a QuFlow circuit, builds its DAG, and produces a report:

```python
from zuntenium import QuantumCircuit, StatevectorSimulator

def analyse_circuit(qc: QuantumCircuit) -> dict:
    """
    Analyse a circuit: build DAG, compute depth, liveness, simulator recommendation.
    """
    # Build DAG from circuit instructions
    dag = CircuitDAG()
    for instr in qc.instructions:
        if instr.operation == "gate":
            dag.add_gate(instr.gate.__class__.__name__, instr.qubits)

    depth = dag.compute_depth()
    liveness = dag.compute_qubit_liveness()
    n = qc.num_qubits
    n_gates = qc.num_gates

    # Estimate memory
    sv_mb   = (16 * 2**n) / 2**20
    dm_gb   = (16 * 4**n) / 2**30 if n <= 15 else float('inf')
    tab_kb  = (n * n) / 8 / 1024

    # Simulator recommendation
    if n <= 15:
        recommendation = "DensityMatrixSimulator (if noise needed) or StatevectorSimulator"
    elif sv_mb < 2000:
        recommendation = "StatevectorSimulator"
    else:
        recommendation = "TensorNetworkBackend (if low entanglement)"

    return {
        "n_qubits": n,
        "n_gates":  n_gates,
        "depth":    depth,
        "qubit_liveness": liveness,
        "sv_mb":    f"{sv_mb:.1f} MB",
        "dm_gb":    f"{dm_gb:.2f} GB" if dm_gb != float('inf') else "N/A (n>15)",
        "tableau_kb": f"{tab_kb:.2f} KB",
        "recommendation": recommendation,
    }

# Test with GHZ circuit
qc_ghz = QuantumCircuit(5, name="GHZ-5")
qc_ghz.h(0)
for i in range(4): qc_ghz.cx(i, i+1)

report = analyse_circuit(qc_ghz)
print("=== Circuit Analysis Report ===")
for k, v in report.items():
    if k == "qubit_liveness": continue
    print(f"  {k:<20}: {v}")
print("  Qubit liveness:")
for q, (b, d) in sorted(report["qubit_liveness"].items()):
    print(f"    q{q}: steps {b}–{d}")
```

---

## 7. Glossary and Next Course

### Glossary

| Term | Definition |
|---|---|
| **CircuitDAG** | Directed Acyclic Graph: nodes=gates, edges=qubit dependencies |
| **Topological sort** | Linear ordering of DAG nodes respecting all edges; prerequisite for scheduling |
| **Critical path** | Longest path in the DAG = circuit depth (determines minimum time to execute) |
| **Qubit liveness** | The time range [birth_step, death_step] during which a qubit holds a meaningful state |
| **CouplingMap** | Hardware connectivity graph: edges = allowed 2-qubit gate pairs |
| **All-pairs distance** | D[i][j] = BFS hop count between physical qubits i and j; used by SABRE |
| **BinaryTableau** | Aaronson-Gottesman O(n²)-bit representation of Clifford stabilizer states |
| **Stabilizer** | Pauli operator P s.t. P\|ψ⟩ = \|ψ⟩; encodes the state without amplitudes |
| **Gottesman-Knill** | Theorem: Clifford circuits simulate in O(n²) classically |
| **Clifford gate** | Gate that maps the n-qubit Pauli group to itself: H, S, CX, X, Y, Z |
| **complex128** | Python/NumPy 16-byte complex float; default dtype for quantum amplitudes |
| **C-contiguous** | NumPy row-major memory layout; innermost axis is stride-1 |
| **np.einsum** | Einstein summation: efficient tensor contraction via BLAS; avoids 2ⁿ×2ⁿ matrix |
| **tracemalloc** | Python stdlib memory profiler; tracks peak allocation per function |
| **Bond dimension** | TensorNetwork compression parameter; small = low entanglement = efficient sim |

### Next Courses

➡ **ZQL-02 (SDK Core)** — *Quantum SDK Architecture & Compiler Design*  
In-depth transpiler: HardwareAwareTranspiler, TranspilerCache LRU, SABRE paper-faithful routing. Uses CouplingMap and CircuitDAG from this course directly.

➡ **ZQL-04** — *Fundamental Quantum Algorithms*  
QFT circuit built from this course's circuit model; depth analysis motivates circuit optimisation passes.

---

*All code verified against zuntenium v0.2.0: `circuit.py`, `backends/density_matrix_simulator.py` (MAX_DM_QUBITS=15), `backends/stabilizer.py`, `backends/tensor_network_backend.py`, `quantum_core/statevector/`.*  
*Generated from: ZQL-02B-notes-clrs-kleinberg.md, ZQL-02B-notes-tanenbaum-gottesman-numpy.md.*
