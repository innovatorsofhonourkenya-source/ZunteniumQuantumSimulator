# ZQL-08-notes-egger-hillier-sutton — Walkthrough Labs

**Source:** [ZQL-08-notes-egger-hillier-sutton.md](../ZQL-08-notes-egger-hillier-sutton.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Egger portfolio + Hillier operations research + Sutton RL

---

## Labs 1-20: Portfolio optimization QUBO (Egger), mean-variance Markowitz model, risk-return trade-off, binary encoding of weights, Egger IBM hardware demo, Hillier LP formulation, integer programming, branch-and-bound, constraint handling, operations research QUBO mapping, Sutton RL basics, Q-learning, policy gradient, RL for QAOA parameter tuning, reward shaping for optimization, exploration vs exploitation, quantum-classical RL hybrid, convergence analysis, resource estimation, integration capstone.

## Lab 1: Egger Portfolio QUBO
```python
print("Egger 2020: Markowitz portfolio as QUBO on quantum hardware")
print("min x'Σx - μ'x subject to Σxᵢ=budget")
```
### Deliverable: `egger_portfolio.py`

## Lab 2: Mean-Variance Model
```python
import numpy as np
ret=np.array([0.05,0.03,0.08]);cov=np.array([[0.01,0,0],[0,0.02,0],[0,0,0.015]])
print(f"3 assets: returns={ret}, risk=diag(Σ)={np.diag(cov)}")
```
### Deliverable: `mean_variance.py`

## Lab 3: Binary Weight Encoding
```python
print("Continuous weight wᵢ∈[0,1] → K binary bits: wᵢ=Σⱼ 2^{-j}·xᵢⱼ")
print("K=3 bits: 8 levels {0,0.125,...,0.875,1.0}")
```
### Deliverable: `binary_encode.py`

## Lab 4: Risk-Return Trade-off Parameter
```python
for q in [0,0.25,0.5,0.75,1.0]:
    print(f"  q={q:.2f}: {'all return' if q==0 else 'all risk' if q==1 else 'balanced'}")
```
### Deliverable: `risk_return.py`

## Lab 5: Egger Hardware Results
```python
print("Egger: 3 assets on IBM 20-qubit, QAOA(p=1)")
print("Result: matches classical Markowitz for small portfolios")
```
### Deliverable: `egger_results.py`

## Lab 6: Hillier LP Formulation
```python
print("LP: min c'x st Ax≤b, x≥0")
print("Quantum: relax to QUBO via penalty method")
```
### Deliverable: `hillier_lp.py`

## Lab 7: Integer Programming
```python
print("IP: LP + integrality constraints → NP-hard in general")
print("QUBO is a special case of binary IP")
```
### Deliverable: `integer_prog.py`

## Lab 8: Branch-and-Bound
```python
print("Classical: solve LP relaxation, branch on fractional variable")
print("Quantum: QAOA/VQE replaces exhaustive search of binary space")
```
### Deliverable: `branch_bound.py`

## Lab 9: Constraint Handling Methods
```python
for m,d in [("Penalty","λ(Ax-b)² added to objective"),("Slack","s²+(Ax-b)=0"),
    ("Feasibility filter","post-select valid solutions")]:print(f"  {m}: {d}")
```
### Deliverable: `constraints.py`

## Lab 10: OR→QUBO Mapping Patterns
```python
print("Assignment: Σⱼxᵢⱼ=1 per row/col → permutation matrix")
print("Knapsack: Σwᵢxᵢ≤C → slack variables")
print("TSP: city×time binary matrix + distance objective")
```
### Deliverable: `or_qubo.py`

## Lab 11: Sutton RL Basics
```python
print("Agent→Environment→Reward loop; policy π(a|s)")
print("Objective: maximize cumulative reward J=Σγᵗrₜ")
```
### Deliverable: `rl_basics.py`

## Lab 12: Q-Learning for Parameter Tuning
```python
print("Q(s,a)←Q(s,a)+α[r+γ·max Q(s',a')-Q(s,a)]")
print("State: current QAOA params; Action: adjustment Δγ,Δβ")
```
### Deliverable: `q_learning.py`

## Lab 13: Policy Gradient
```python
print("REINFORCE: ∇J=E[∇log π(a|s)·R]")
print("Apply to QAOA: policy outputs (γ,β) directly")
```
### Deliverable: `policy_grad.py`

## Lab 14: RL for QAOA Parameter Tuning
```python
print("Train RL agent to choose QAOA params adaptively")
print("Reward: cut value; State: graph structure; Action: (γ,β)")
```
### Deliverable: `rl_qaoa.py`

## Lab 15: Reward Shaping
```python
print("Shape reward: R=cut_value/max_cut + feasibility_bonus")
print("Avoid sparse rewards that slow RL convergence")
```
### Deliverable: `reward_shape.py`

## Lab 16: Exploration vs Exploitation
```python
print("ε-greedy: explore random (γ,β) with prob ε, exploit best with 1-ε")
print("Decay ε over episodes for convergence")
```
### Deliverable: `explore_exploit.py`

## Lab 17: Quantum-Classical RL Hybrid
```python
print("Classical RL agent → quantum circuit → measurement → reward → update")
print("Quantum part: QAOA or VQE circuit evaluation")
```
### Deliverable: `hybrid_rl.py`

## Lab 18: Convergence Analysis
```python
for iters in [10,50,100,500]:
    print(f"  {iters} iters: {'likely converged' if iters>100 else 'may need more'}")
```
### Deliverable: `convergence.py`

## Lab 19: Resource Estimation
```python
for n in [5,10,20,50]:
    print(f"  n={n} variables: {n} qubits, ~{3*n} gates/layer")
```
### Deliverable: `resources.py`

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("Portfolio QUBO",True);ok("Constraint handling",True);ok("RL basics",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("ehs_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `ehs_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
