# ZQL-08 Step 1 Notes — Farhi QAOA + Glover QUBO + Cook et al. Combinatorial Optimization

---

# A. Farhi, Goldstone & Gutmann — *QAOA paper* (2014) — Deep Dive for ZQL-08

*(ZQL-05 covered QAOA intro; ZQL-08 uses it for real combinatorial problems.)*

## p=1 QAOA Approximation Ratio
- MaxCut on 3-regular graphs: QAOA(p=1) guarantees ≥ **0.6924** of optimal.
- **Key insight**: this is a provable lower bound, not just empirical.
- As p→∞, QAOA approaches the adiabatic algorithm → exact solution.

## QAOA for Weighted MaxCut
- Cost: C(z) = Σ_{(i,j)∈E} w_{ij}·(1 − zᵢzⱼ)/2 where zᵢ ∈ {−1,+1}.
- Hamiltonian: H_C = Σ_{(i,j)} w_{ij}·(I − ZᵢZⱼ)/2.
- Weighted ZZ rotation: `qc.cx(i,j); qc.rz(w_ij * gamma, j); qc.cx(i,j)`.

## QAOA Parameter Landscape
- For small p, landscape is smooth → grid search + COBYLA is effective.
- For large p, landscape becomes more complex → warm-starting helps.
- **Warm start**: initialise QAOA parameters θ to match the continuous relaxation (e.g., SDP relaxation of MaxCut).

## Recursive QAOA (RQAOA)
- Evaluate edge correlations ⟨ZᵢZⱼ⟩ from QAOA state.
- Fix variable with highest |⟨ZᵢZⱼ⟩| to its most likely value.
- Reduce problem size by 1, recurse.
- Fewer qubits needed; improves practical performance.

---

# B. Glover, Kochenberger & Du — *QUBO Tutorial* (4OR, 2019)

**The canonical quadratic unconstrained binary optimisation reference.**

## QUBO Formulation
QUBO problem: minimise x^T Q x where x ∈ {0,1}^n and Q ∈ ℝ^{n×n}.

```
minimise     Σᵢ Qᵢᵢ xᵢ + Σᵢ<ⱼ Qᵢⱼ xᵢxⱼ
subject to   xᵢ ∈ {0, 1}
```

## QUBO ↔ Ising Conversion
Substitution: xᵢ = (1 − sᵢ)/2 where sᵢ ∈ {−1, +1}.

```
QUBO → Ising:  Hᵢₛᵢₙ_g = Σᵢ hᵢ sᵢ + Σᵢ<ⱼ Jᵢⱼ sᵢsⱼ

where:
  Jᵢⱼ = Qᵢⱼ / 4
  hᵢ  = Qᵢᵢ/2 + Σⱼ≠ᵢ Qᵢⱼ/4
  +constant offset
```

## Common Problem → QUBO Mappings

### MaxCut
```
QUBO: minimise −Σ_{(i,j)∈E} (xᵢ + xⱼ − 2xᵢxⱼ)
Q[i,j] = {  +1 if (i,j)∈E (diagonal: -Σneighbours)
           { -2 if (i,j)∈E (off-diagonal pair)
```
**[SDK]** MaxCut → QUBO → Ising → `PauliHamiltonian` → QAOA.

### Max-2-SAT
Any 2-SAT clause (xᵢ ∨ xⱼ) can be converted to a QUBO penalty:
```
Penalty: (1 − xᵢ)(1 − xⱼ) →  1 - xᵢ - xⱼ + xᵢxⱼ
```

### Number Partitioning
Partition S = {a₁,...,aₙ} into two subsets with equal sum.
```
QUBO: Q[i,i] = aᵢ(aᵢ − Σⱼaⱼ),  Q[i,j] = 2aᵢaⱼ  (i≠j)
```

### Travelling Salesman Problem (TSP) — light treatment
- Variables: x_{i,p} = 1 if city i is in position p.
- Constraints: each city visited exactly once, each position filled exactly once.
- QUBO penalty for violations + objective (total distance).
- n cities → n² binary variables → n² qubits.

---

# C. Cook, Cunningham, Pulleyblank & Schrijver — *Combinatorial Optimization* (5th ed., Wiley)

## Graph Theory Fundamentals
- **Cut**: partition V into (S, T); cut edges = {(i,j): i∈S, j∈T}.
- **MaxCut**: NP-hard in general; polynomial-time for planar graphs.
- **Minimum Vertex Cover**: find minimum S ⊆ V with every edge covered. Dual of MaxMatching.
- **Independent Set**: max set of vertices with no shared edge. NP-hard.

## iLPC and SDP Relaxations
- MaxCut SDP relaxation: assign unit vectors vᵢ to each vertex; maximise Σ_{(i,j)} w_{ij}(1 − vᵢ·vⱼ)/2.
- Goemans-Williamson rounding: achieves 0.8785× optimal (best classical approximation).
- **QAOA comparison**: QAOA(p=1) ≥ 0.6924 < GW 0.8785 for MaxCut. Classical SDP beats QAOA at p=1.
- **[Teaching]** Classical approximation algorithms provide the benchmark QAOA must beat.

## NP-Hard Problem Complexity
| Problem | QUBO variables | Classical best | QAOA advantage? |
|---|---|---|---|
| MaxCut (3-reg.) | n | GW 0.8785 | Unclear (p=1: 0.6924) |
| Max-2-SAT | n | 0.940 approx. | Unlikely at small p |
| Number partition | n | Pseudo-polynomial | Open question |
| TSP | n² | Christofides 1.5× | Open question |
| Portfolio opt. | n | Classical QP solvers | Potential edge |
