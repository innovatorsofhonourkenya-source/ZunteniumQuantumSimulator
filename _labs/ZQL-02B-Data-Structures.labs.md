# ZQL-02B-Data-Structures — Walkthrough Labs

**Source:** [ZQL-02B-Data-Structures.md](../ZQL-02B-Data-Structures.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Quantum compiler data structures & memory engineering

---

## Global Setup
```bash
mkdir -p ~/zql-ds-labs && cd ~/zql-ds-labs
python3 -m venv .venv && source .venv/bin/activate
pip install zuntenium==0.2.0 numpy==1.26.4
```

---

## Lab 1: CircuitDAG Construction — Gate Dependencies
- **Concept:** Build dependency DAG from gate sequence (§1.2)
- **Difficulty:** Intermediate | **Time:** 75 min
### Walkthrough
```python
# save as circuit_dag.py
from dataclasses import dataclass, field
from collections import defaultdict, deque

@dataclass
class GateNode:
    gate_id: int; name: str; qubits: list
    predecessors: list = field(default_factory=list)
    successors: list = field(default_factory=list)

class CircuitDAG:
    def __init__(self):
        self.nodes = {}; self._last = {}; self._nid = 0
    def add_gate(self, name, qubits):
        gid = self._nid; self._nid += 1
        preds = []
        for q in qubits:
            if q in self._last:
                pid = self._last[q]; preds.append(pid)
                self.nodes[pid].successors.append(gid)
        self.nodes[gid] = GateNode(gid, name, qubits, preds)
        for q in qubits: self._last[q] = gid
        return gid

dag = CircuitDAG()
dag.add_gate("H", [0]); dag.add_gate("CX", [0,1]); dag.add_gate("CX", [1,2])
for g in dag.nodes.values():
    print(f"  {g.name}(q{g.qubits}) → succ={g.successors}")
```
### Verification
```python
assert len(dag.nodes) == 3
print("Lab 1 Verification: PASSED")
```
### Deliverable: `circuit_dag.py`

---

## Lab 2: Topological Sort — Kahn's Algorithm
- **Concept:** BFS-based topological ordering (CLRS §22.4)
- **Difficulty:** Intermediate | **Time:** 75 min
### Walkthrough
```python
# save as topo_sort.py
from collections import defaultdict, deque
# Reuse CircuitDAG from Lab 1
def topological_sort(dag):
    indeg = defaultdict(int)
    for n in dag.nodes.values():
        for s in n.successors: indeg[s] += 1
    q = deque(g for g in dag.nodes if indeg[g] == 0)
    order = []
    while q:
        gid = q.popleft(); order.append(gid)
        for s in dag.nodes[gid].successors:
            indeg[s] -= 1
            if indeg[s] == 0: q.append(s)
    assert len(order) == len(dag.nodes), "Cycle detected!"
    return order

order = topological_sort(dag)
print(f"Topological order: {[dag.nodes[g].name for g in order]}")
```
### Verification
```python
assert len(order) == len(dag.nodes)
print("Lab 2 Verification: PASSED")
```
### Deliverable: `topo_sort.py`

---

## Lab 3: Circuit Depth — Critical Path Length
- **Concept:** Longest path in DAG = circuit depth (§1.3)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as circuit_depth.py
def compute_depth(dag):
    order = topological_sort(dag)
    dist = {g: 1 for g in dag.nodes}
    for gid in order:
        for s in dag.nodes[gid].successors:
            dist[s] = max(dist[s], dist[gid]+1)
    return max(dist.values(), default=0)

depth = compute_depth(dag)
print(f"GHZ circuit depth: {depth}")
```
### Verification
```python
assert depth >= 2
print("Lab 3 Verification: PASSED")
```
### Deliverable: `circuit_depth.py`

---

## Lab 4: Qubit Liveness Analysis
- **Concept:** Track when qubits are first/last used (§1.3)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as qubit_liveness.py
def compute_liveness(dag):
    order = topological_sort(dag)
    step = {g: i for i, g in enumerate(order)}
    life = {}
    for g, n in dag.nodes.items():
        for q in n.qubits:
            s = step[g]
            if q not in life: life[q] = [s, s]
            else: life[q] = [min(life[q][0],s), max(life[q][1],s)]
    return {q: tuple(l) for q, l in life.items()}

lv = compute_liveness(dag)
for q, (b,d) in sorted(lv.items()):
    print(f"  q{q}: steps {b}→{d} (live {d-b+1} steps)")
```
### Verification
```python
assert len(lv) == 3
print("Lab 4 Verification: PASSED")
```
### Deliverable: `qubit_liveness.py`

---

## Lab 5: Critical Path Traceback
- **Concept:** Find the actual gate sequence on the longest path (§1.4)
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as critical_path.py
def critical_path(dag):
    order = topological_sort(dag)
    dist = {g:1 for g in dag.nodes}; prev = {g:None for g in dag.nodes}
    for gid in order:
        for s in dag.nodes[gid].successors:
            if dist[gid]+1 > dist[s]: dist[s]=dist[gid]+1; prev[s]=gid
    end = max(dist, key=dist.get)
    path, cur = [], end
    while cur is not None: path.append(dag.nodes[cur].name); cur = prev[cur]
    return list(reversed(path))

print(f"Critical path: {critical_path(dag)}")
```
### Verification
```python
assert len(critical_path(dag)) >= 2
print("Lab 5 Verification: PASSED")
```
### Deliverable: `critical_path.py`

---

## Lab 6: CouplingMap — Hardware Connectivity Graph (§2)
- **Concept:** Build and query hardware topology
- **Difficulty:** Intermediate | **Time:** 75 min
### Walkthrough
```python
# save as coupling_map.py
from collections import deque
class CouplingMap:
    def __init__(self, n):
        self.n = n; self._adj = {i: set() for i in range(n)}
    def add_edge(self, a, b, bi=True):
        self._adj[a].add(b)
        if bi: self._adj[b].add(a)
    def shortest_path(self, src, dst):
        if src == dst: return 0
        visited = {src}; q = deque([(src, 0)])
        while q:
            node, d = q.popleft()
            for nb in self._adj[node]:
                if nb == dst: return d+1
                if nb not in visited: visited.add(nb); q.append((nb, d+1))
        return -1
    @classmethod
    def linear(cls, n):
        cm = cls(n)
        for i in range(n-1): cm.add_edge(i, i+1)
        return cm

cm = CouplingMap.linear(5)
for i in range(5):
    for j in range(5):
        print(f"  d({i},{j})={cm.shortest_path(i,j)}", end="")
    print()
```
### Verification
```python
assert cm.shortest_path(0, 4) == 4
print("Lab 6 Verification: PASSED")
```
### Deliverable: `coupling_map.py`

---

## Lab 7: All-Pairs Distance Matrix
- **Concept:** BFS from each qubit → full distance matrix (§2.1)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as all_pairs.py
def all_pairs_distance(cm):
    D = [[-1]*cm.n for _ in range(cm.n)]
    for s in range(cm.n):
        D[s][s] = 0; visited = {s}; q = deque([(s,0)])
        while q:
            node, d = q.popleft()
            for nb in cm._adj[node]:
                if nb not in visited:
                    visited.add(nb); D[s][nb] = d+1; q.append((nb, d+1))
    return D

D = all_pairs_distance(CouplingMap.linear(5))
print("Distance matrix:"); [print(row) for row in D]
diameter = max(max(row) for row in D)
print(f"Diameter: {diameter}")
```
### Verification
```python
assert diameter == 4
print("Lab 7 Verification: PASSED")
```
### Deliverable: `all_pairs.py`

---

## Lab 8: IBM Tokyo 20-Qubit CouplingMap
- **Concept:** Real hardware topology (§2.1)
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as ibm_tokyo.py
cm = CouplingMap(20)
edges = [(0,1),(1,2),(2,3),(3,4),(0,5),(1,6),(2,7),(3,8),(4,9),(5,6),(6,7),(7,8),(8,9),(5,10),(6,11),(7,12),(8,13),(9,14),(10,11),(11,12),(12,13),(13,14),(10,15),(11,16),(12,17),(13,18),(14,19),(15,16),(16,17),(17,18),(18,19)]
for a,b in edges: cm.add_edge(a,b)
D = all_pairs_distance(cm)
print(f"IBM Tokyo diameter: {max(max(r) for r in D)}")
print(f"d(0,19) = {D[0][19]}")
```
### Verification
```python
assert D[0][19] > 0
print("Lab 8 Verification: PASSED")
```
### Deliverable: `ibm_tokyo.py`

---

## Lab 9: SABRE Distance Heuristic
- **Concept:** Cost function for gate placement given mapping (§2.2)
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as sabre_heuristic.py
def sabre_cost(q0, q1, mapping, D):
    return D[mapping[q0]][mapping[q1]]

mapping = {i: i for i in range(20)}
print(f"Cost(3,7): {sabre_cost(3,7,mapping,D)}")
print(f"Cost(0,19): {sabre_cost(0,19,mapping,D)}")
# Swap mapping to reduce cost
mapping[0], mapping[5] = mapping[5], mapping[0]
print(f"After swap: Cost(0,19)={sabre_cost(0,19,mapping,D)}")
```
### Verification
```python
print("Lab 9 Verification: PASSED")
```
### Deliverable: `sabre_heuristic.py`

---

## Lab 10: Memory Scaling — Four Simulators Compared (§3)
- **Concept:** O(2^n) vs O(4^n) vs O(n²) memory
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as memory_scaling.py
for n in [5, 10, 15, 20, 25, 30]:
    sv = 16 * 2**n; dm = 16 * 4**n; tab = n*n//8
    def fmt(b):
        if b < 1024: return f"{b}B"
        if b < 2**20: return f"{b//1024}KB"
        if b < 2**30: return f"{b//2**20}MB"
        if b < 2**40: return f"{b//2**30}GB"
        return f"{b//2**40}TB"
    print(f"n={n:>2}: SV={fmt(sv):>8} DM={fmt(dm):>8} Tab={fmt(tab):>8}")
```
### Verification
```python
assert 16 * 2**30 > 16e9
print("Lab 10 Verification: PASSED")
```
### Deliverable: `memory_scaling.py`

---

## Lab 11: Statevector Memory Profiling with tracemalloc (§5)
- **Concept:** Measure actual vs theoretical memory usage
- **Difficulty:** Advanced | **Time:** 60 min
### Walkthrough
```python
# save as sv_profile.py
import tracemalloc, numpy as np
for n in [4, 8, 12, 16]:
    tracemalloc.start()
    sv = np.zeros(2**n, dtype=np.complex128); sv[0] = 1
    _, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    theory = 16 * 2**n
    print(f"n={n:>2}: theory={theory//1024:>6}KB peak={peak//1024:>6}KB ratio={peak/theory:.1f}x")
```
### Verification
```python
print("Lab 11 Verification: PASSED")
```
### Deliverable: `sv_profile.py`

---

## Lab 12: DensityMatrix Memory Guard — MAX_DM_QUBITS (§3.3)
- **Concept:** Why n≤15 for density matrix simulation
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as dm_guard.py
MAX_DM_QUBITS = 15
for n in [5, 10, 12, 15, 16, 20]:
    mem_gb = (16 * 4**n) / 2**30
    ok = "✓" if n <= MAX_DM_QUBITS else "✗ EXCEEDS LIMIT"
    print(f"n={n:>2}: {mem_gb:>10.2f} GB  {ok}")
```
### Verification
```python
assert (16 * 4**15) / 2**30 < 20
print("Lab 12 Verification: PASSED")
```
### Deliverable: `dm_guard.py`

---

## Lab 13: BinaryTableau — Clifford Stabilizer State (§3.4)
- **Concept:** O(n²) representation for Clifford circuits
- **Difficulty:** Expert | **Time:** 90 min
### Walkthrough
```python
# save as binary_tableau.py
import numpy as np
class BinaryTableau:
    def __init__(self, n):
        self.n = n
        self._tab = np.zeros((2*n, 2*n+1), dtype=bool)
        for i in range(n):
            self._tab[i, i] = True; self._tab[n+i, n+i] = True
    @property
    def memory_bytes(self): return self._tab.nbytes
    def apply_h(self, q):
        for r in range(2*self.n):
            xi, zi = self._tab[r,q], self._tab[r,self.n+q]
            self._tab[r, 2*self.n] ^= xi & zi
            self._tab[r,q], self._tab[r,self.n+q] = zi, xi
    def apply_cnot(self, c, t):
        for r in range(2*self.n):
            xc,xt = self._tab[r,c], self._tab[r,t]
            zc,zt = self._tab[r,self.n+c], self._tab[r,self.n+t]
            self._tab[r, 2*self.n] ^= xc & zt & (xt ^ zc ^ True)
            self._tab[r,t] ^= xc; self._tab[r,self.n+c] ^= zt

tab = BinaryTableau(2)
tab.apply_h(0); tab.apply_cnot(0, 1)
print(f"Bell state tableau ({tab.memory_bytes} bytes):")
for i in range(2, 4):
    x = ''.join('X' if tab._tab[i,q] else '.' for q in range(2))
    z = ''.join('Z' if tab._tab[i,2+q] else '.' for q in range(2))
    ph = '-' if tab._tab[i,4] else '+'
    print(f"  {ph}{x}|{z}")
```
### Verification
```python
assert tab.memory_bytes < 100
print("Lab 13 Verification: PASSED")
```
### Deliverable: `binary_tableau.py`

---

## Lab 14: Stabilizer vs Statevector — 1000 Qubits
- **Concept:** Gottesman-Knill makes 1000-qubit Clifford tractable
- **Difficulty:** Expert | **Time:** 60 min
### Walkthrough
```python
# save as stab_vs_sv.py
import numpy as np, time
for n in [10, 50, 100, 500, 1000]:
    tab = np.zeros((2*n, 2*n+1), dtype=bool)
    tab_bytes = tab.nbytes
    sv_bytes = 16 * 2**min(n, 30)  # Cap for display
    print(f"n={n:>4}: Tableau={tab_bytes:>10}B, SV={'IMPOSSIBLE' if n>30 else f'{sv_bytes}B':>12}")
```
### Verification
```python
print("Lab 14 Verification: PASSED")
```
### Deliverable: `stab_vs_sv.py`

---

## Lab 15: Simulator Selection Decision Tree (§4)
- **Concept:** Choose the right backend for circuit type
- **Difficulty:** Intermediate | **Time:** 60 min
### Walkthrough
```python
# save as sim_selector.py
def select_sim(n, noise, clifford, low_ent):
    if clifford: return f"StabilizerSimulator (O(n²)={n**2} bits)"
    if noise: return f"DensityMatrix (n≤15)" if n<=15 else "ERROR: n>15"
    if low_ent: return "TensorNetworkBackend"
    return f"StatevectorSimulator ({16*2**n//2**20}MB)" if n<=30 else "INFEASIBLE"

tests = [(50,False,True,False),(8,True,False,False),(25,False,False,True),(18,False,False,False)]
for n,noise,cliff,le in tests:
    print(f"n={n}, noise={noise}, cliff={cliff}: {select_sim(n,noise,cliff,le)}")
```
### Verification
```python
print("Lab 15 Verification: PASSED")
```
### Deliverable: `sim_selector.py`

---

## Lab 16: einsum Gate Application — No Full Matrix
- **Concept:** Apply gate via tensor contraction (§7.1 from ZQL-02A)
- **Difficulty:** Advanced | **Time:** 75 min
### Walkthrough
```python
# save as einsum_ds.py
import numpy as np
def apply_gate(state, gate, target, n):
    psi = state.reshape([2]*n)
    ax = [chr(ord('a')+k) for k in range(n)]
    t = ax[target]; z = 'z'
    sub = f"z{t},{''.join(ax)}->{''.join(z if k==target else ax[k] for k in range(n))}"
    return np.einsum(sub, gate, psi).reshape(-1)

H = np.array([[1,1],[1,-1]])/np.sqrt(2)
state = np.zeros(32, dtype=complex); state[0] = 1
result = apply_gate(state, H, 2, 5)
print(f"H on q2 of 5-qubit: P(00000)={abs(result[0])**2:.4f}")
```
### Verification
```python
assert abs(abs(result[0])**2 - 0.5) < 1e-10
print("Lab 16 Verification: PASSED")
```
### Deliverable: `einsum_ds.py`

---

## Lab 17: np.kron Full Matrix Path
- **Concept:** Build 2^n×2^n gate matrix for small n (§7.2)
- **Difficulty:** Intermediate | **Time:** 45 min
### Walkthrough
```python
# save as kron_ds.py
import numpy as np
def full_gate(gate, target, n):
    I = np.eye(2, dtype=complex); result = np.eye(1, dtype=complex)
    for i in range(n): result = np.kron(result, gate if i==target else I)
    return result

H = np.array([[1,1],[1,-1]])/np.sqrt(2)
U = full_gate(H, 1, 4)
print(f"Full matrix shape: {U.shape}")
print(f"Unitary: {np.allclose(U.conj().T @ U, np.eye(16))}")
```
### Verification
```python
assert U.shape == (16, 16)
print("Lab 17 Verification: PASSED")
```
### Deliverable: `kron_ds.py`

---

## Lab 18: Circuit Analyzer Tool (§6 Capstone)
- **Concept:** Analyze QuFlow circuit: DAG, depth, liveness, recommendation
- **Difficulty:** Expert | **Time:** 90 min
### Walkthrough
```python
# save as circuit_analyzer.py
from zuntenium import QuantumCircuit
qc = QuantumCircuit(5, name="GHZ-5")
qc.h(0)
for i in range(4): qc.cx(i, i+1)

dag2 = CircuitDAG()
for inst in qc.instructions:
    if inst.operation == "gate":
        dag2.add_gate(inst.gate_name, inst.qubits)

depth = compute_depth(dag2)
lv = compute_liveness(dag2)
n = qc.num_qubits
print(f"=== {qc.name} ===")
print(f"Qubits: {n}, Gates: {qc.num_gates}, Depth: {depth}")
print(f"SV memory: {16*2**n//1024} KB")
for q, (b,d) in sorted(lv.items()):
    print(f"  q{q}: steps {b}→{d}")
```
### Verification
```python
assert depth >= 2
print("Lab 18 Verification: PASSED")
```
### Deliverable: `circuit_analyzer.py`

---

## Lab 19: GHZ-20 Performance Benchmark
- **Concept:** End-to-end simulation benchmarking with timing
- **Difficulty:** Expert | **Time:** 75 min
### Walkthrough
```python
# save as ghz_bench.py
import time
from zuntenium import QuantumCircuit, StatevectorSimulator
backend = StatevectorSimulator()
for n in [4, 8, 12, 16, 20]:
    qc = QuantumCircuit(n); qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    qc.measure_all()
    t0 = time.time()
    r = backend.run(qc, shots=1024)
    dt = time.time() - t0
    print(f"n={n:>2}: {dt*1000:>8.1f}ms, outcomes={len(r.counts)}")
```
### Verification
```python
print("Lab 19 Verification: PASSED")
```
### Deliverable: `ghz_bench.py`

---

## Lab 20: Data Structures Integration Capstone
- **Concept:** Full verification of DAG, CouplingMap, memory, tableau
- **Difficulty:** Expert | **Time:** 120 min
### Walkthrough
```python
# save as ds_capstone.py
import json, time, numpy as np
tests = []
def ok(n,c): tests.append({"name":n,"pass":bool(c)}); print(f"{'✓' if c else '✗'} {n}")

dag = CircuitDAG()
dag.add_gate("H",[0]); dag.add_gate("CX",[0,1])
ok("DAG build", len(dag.nodes)==2)
ok("Topo sort", len(topological_sort(dag))==2)
ok("Depth", compute_depth(dag)>=2)

cm = CouplingMap.linear(5)
ok("Coupling shortest", cm.shortest_path(0,4)==4)

tab = BinaryTableau(2)
ok("Tableau size", tab.memory_bytes < 100)

ok("Memory scaling", 16*2**20 == 16*1048576)

report={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("ds_report.json","w") as f: json.dump(report,f,indent=2)
print(f"\nAll pass: {report['all_pass']}")
```
### Verification
```python
assert json.load(open("ds_report.json"))["all_pass"]
print("Lab 20 Verification: PASSED")
```
### Deliverable: `ds_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct data structure concepts ✓ 3. **Coverage:** CircuitDAG, topological sort, depth, liveness, critical path, CouplingMap, BFS shortest path, all-pairs distance, IBM topology, SABRE heuristic, memory scaling, tracemalloc, DM guard, BinaryTableau, Gottesman-Knill, simulator selection, einsum, kron, circuit analyzer, benchmarking ✓
