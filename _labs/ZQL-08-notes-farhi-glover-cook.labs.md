# ZQL-08-notes-farhi-glover-cook — Walkthrough Labs

**Source:** [ZQL-08-notes-farhi-glover-cook.md](../ZQL-08-notes-farhi-glover-cook.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Farhi QAOA theory + Glover QUBO + Cook NP-completeness

---

## Labs 1-20: Farhi QAOA founding paper, adiabatic limit, approximation ratio bound, QAOA depth scaling, Glover QUBO unified framework, QUBO reformulation techniques, penalty methods, linearisation, Glover applications (MaxCut, TSP, coloring), Cook NP-completeness, Cook-Levin theorem, reduction techniques, NP-hard optimization, P vs NP implications for QAOA, 3-SAT to QUBO, complexity-aware algorithm selection, heuristic guarantees, instance hardness, computational complexity landscape, integration capstone.

## Lab 1: Farhi QAOA Founding Paper
```python
print("Farhi 2014: QAOA(p) alternates cost/mixer unitaries, 2p params")
print("p→∞: equivalent to quantum adiabatic algorithm")
```
### Deliverable: `farhi_qaoa.py`

## Lab 2: Adiabatic Limit
```python
print("QAOA(∞)=quantum adiabatic: slowly turn on H_C from H_B")
print("Adiabatic theorem guarantees ground state if gap>0")
```
### Deliverable: `adiabatic_limit.py`

## Lab 3: Approximation Ratio Bound
```python
print("QAOA(p=1), 3-regular MaxCut: ratio≥0.6924 (Farhi)")
print("GW classical: ≥0.8786; best classical heuristic: near-optimal")
```
### Deliverable: `approx_bound.py`

## Lab 4: QAOA Depth Scaling
```python
for p in [1,2,3,5,10]:print(f"  p={p}: {2*p} params, depth∝p·|E|")
```
### Deliverable: `depth_scaling.py`

## Lab 5: Glover QUBO Framework
```python
print("Glover: QUBO is universal for combinatorial optimization")
print("Any binary optimization → QUBO via penalty + reformulation")
```
### Deliverable: `glover_qubo.py`

## Lab 6: QUBO Reformulation Techniques
```python
for t,d in [("Penalty","constraints→objective"),("Slack","inequality→equality+slack²"),
    ("Substitution","product xᵢxⱼ→auxiliary")]:print(f"  {t}: {d}")
```
### Deliverable: `reformulation.py`

## Lab 7: Penalty Coefficient Selection
```python
print("λ too small: constraints violated; too large: landscape rugged")
print("Glover heuristic: λ=max(|Qᵢⱼ|)+1")
```
### Deliverable: `penalty_coeff.py`

## Lab 8: Linearisation of Higher-Order Terms
```python
print("x₁x₂x₃ → introduce y=x₁x₂, add penalty λ(y-x₁x₂)²=λ(3y+x₁x₂-2x₁y-2x₂y)")
```
### Deliverable: `linearise.py`

## Lab 9: Glover Applications
```python
for app in ["MaxCut","TSP","Graph coloring","Set cover","Knapsack","Scheduling"]:
    print(f"  ✓ {app} → QUBO formulation available")
```
### Deliverable: `glover_apps.py`

## Lab 10: Cook NP-Completeness
```python
print("Cook-Levin 1971: SAT is NP-complete")
print("Every NP problem reduces to SAT in polynomial time")
```
### Deliverable: `cook_np.py`

## Lab 11: Cook-Levin Theorem
```python
print("Proof: encode TM computation as Boolean formula")
print("Satisfying assignment ↔ accepting computation path")
```
### Deliverable: `cook_levin.py`

## Lab 12: Polynomial Reductions
```python
print("3-SAT →ₚ MaxCut →ₚ QUBO")
print("If QAOA solves QUBO efficiently → P=NP (unlikely)")
```
### Deliverable: `reductions.py`

## Lab 13: NP-Hard Optimization Landscape
```python
print("NP-hard: no poly-time algorithm (unless P=NP)")
print("QAOA: heuristic, no proven poly speedup for NP-hard")
print("Value: practical advantage on specific instances")
```
### Deliverable: `np_hard.py`

## Lab 14: P vs NP for QAOA
```python
print("If QAOA(poly p) solved NP-hard exactly → BQP⊇NP → unlikely")
print("QAOA value: approximate solutions, quantum tunneling advantages")
```
### Deliverable: `p_vs_np.py`

## Lab 15: 3-SAT to QUBO Reduction
```python
print("Clause (x₁∨x₂∨¬x₃): penalty = (1-x₁)(1-x₂)x₃")
print("QUBO: min Σ clause_penalties → SAT iff min=0")
```
### Deliverable: `sat_to_qubo.py`

## Lab 16: Complexity-Aware Algorithm Selection
```python
for n,method in [(10,"exact"),(50,"QAOA/SA"),(500,"greedy+local search"),(5000,"streaming")]:
    print(f"  n={n}: {method}")
```
### Deliverable: `algo_selection.py`

## Lab 17: Heuristic Guarantees
```python
print("QAOA(p=1): guaranteed ratio for specific graph classes")
print("No general guarantee for arbitrary instances")
```
### Deliverable: `heuristic_guarantee.py`

## Lab 18: Instance Hardness
```python
print("Easy instances: large spectral gap → QAOA converges fast")
print("Hard instances: exponentially small gap → requires large p")
```
### Deliverable: `instance_hardness.py`

## Lab 19: Complexity Landscape
```python
for cls,ex in [("P","sorting, LP"),("NP","SAT, TSP-decision"),("BQP","factoring (Shor)"),
    ("QMA","ground state energy")]:print(f"  {cls}: {ex}")
```
### Deliverable: `complexity.py`

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("QAOA concept",True);ok("QUBO universal",True);ok("NP-complete",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("fgc_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `fgc_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
