# ZQL-02B Step 1 Notes — CLRS (Algorithms) + Kleinberg & Tardos (Algorithm Design)

---

# A. Cormen, Leiserson, Rivest & Stein — *Introduction to Algorithms* (4th ed., CLRS)

**Relevant chapters: 6 (heaps), 22–26 (graph algorithms for CircuitDAG)**

## Ch. 22 — Elementary Graph Algorithms

### Representations
- **Adjacency list**: G = (V, E); space O(V+E). Best for sparse graphs (circuits are sparse).
- **Adjacency matrix**: V×V matrix; space O(V²). Good for dense graphs or frequent edge queries.
- **[SDK]** `CircuitDAG` uses adjacency-list representation: each node stores its successor list. Gate DAGs are always sparse (O(gates) edges, not O(gates²)).

### Depth-First Search (DFS)
- DFS on DAG produces **finish times** → reverse finish order = topological sort.
- DFS complexity: O(V+E).
- **[SDK]** `DepthAnalysisPass` uses DFS to compute circuit depth. Topological sort of DAG gives valid gate scheduling order.

### Topological Sort
- Prerequisite for ASAP/ALAP scheduling (ZQL-02B core).
- Algorithm: DFS, push to stack on finish → pop order = topological order. O(V+E).
- **[SDK]** `ALAPSchedulingPass` builds dependency DAG then topologically sorts to assign time steps.

## Ch. 23 — Minimum Spanning Trees
- Prim's/Kruskal's: find lowest-weight spanning tree.
- **[SDK analogy]** Coupling map spanning tree used as initial qubit layout seed for VF2PlacementPass.

## Ch. 24 — Single-Source Shortest Paths
- Dijkstra: O((V+E)·log V) for non-negative weights.
- Bellman-Ford: O(VE) for negative weights.
- **[SDK]** SABRE router uses BFS (unweighted shortest paths = hop distance) to build `D[i,j]` qubit distance matrix. For noise-aware routing: weighted Dijkstra on edge fidelity costs.

## Ch. 25 — All-Pairs Shortest Paths
- Floyd-Warshall: O(V³). BFS all-pairs for unit-weight: O(V·(V+E)).
- **[SDK]** `SABRERouterV2._build_distance_matrix()` runs BFS from each qubit to build the full distance matrix D — exactly the CLRS all-pairs BFS approach.
  ```python
  # BFS from each source qubit
  for source in range(n_qubits):
      dist[source] = bfs(coupling_graph, source)  # O(V+E)
  # Total: O(V·(V+E))
  ```

## Ch. 26 — Maximum Flow (conceptual)
- Max-flow / min-cut: partition graph into two sets maximising flow.
- **[ANALOGY]** MaxCut quantum problem is the combinatorial dual: maximise |cut| using QAOA (ZQL-05). Not used directly in SDK routing.

---

# B. Kleinberg & Tardos — *Algorithm Design*

**Relevant: graph algorithms, dynamic programming, greedy — applicable to DAG scheduling**

## Ch. 3 — Graphs
- DAG (Directed Acyclic Graph): no cycles. All quantum circuits must have acyclic gate dependencies.
- Strongly connected components (Tarjan's/Kosaraju's): detect cycles in dependency graph. If circuit has cycles → invalid (non-physical).
- **[SDK]** `CircuitValidator` should detect cycles in gate dependency DAG. Any cycle would mean a gate depends on its own output — physically impossible.

## Ch. 4 — Greedy Algorithms
- **Interval scheduling**: schedule maximum non-overlapping intervals. Greedy: earliest-finish-time first.
- **[SDK]** ASAP scheduling allocates gates to earliest available time step (greedy). ALAP reverses this.
- **Activity selection**: select compatible activities (a solution to scheduling conflicts).
- **[SDK]** `CrosstalkSchedulingPass` uses a greedy conflict-avoidance approach: if two gates on adjacent qubits would crosstalk, delay the later one.

## Ch. 6 — Dynamic Programming

### Weighted Interval Scheduling
- Optimal sub-structure: best schedule for intervals 1..j = max(include j, exclude j).
- **[SDK analogy]** Optimal qubit routing is DP-hard in general. SABRE approximates it greedily.

### Sequence Alignment (Edit Distance)
- DP table; O(mn) time.
- **[SDK]** Compiler pass optimisation: finding the minimal set of SWAP insertions between two qubit mappings is conceptually similar to edit-distance DP.

## Ch. 13 — Randomised Algorithms
- Monte Carlo vs Las Vegas algorithms.
- **[SDK]** SABRE router: the 5 random trials (seeds) + 3-traversal loop is a Las Vegas-style search. It terminates with a valid solution but may find different (non-optimal) solutions each run.
