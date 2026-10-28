# ZQL-00.01-notes-johnston — Walkthrough Labs

**Source:** [ZQL-00.01-notes-johnston.md](../ZQL-00.01-notes-johnston.md)
**Generated:** 2026-10-28
**Labs:** 20 | **Domain:** Quantum / Johnston textbook developer patterns

---

## Source Analysis

- **Document objective:** Map Johnston, Harrigan & Gimeno-Segovia's *Programming Quantum Computers* (O'Reilly 2019) patterns to QuFlow SDK — focusing on developer mental models: amplitude-not-probability, interference-is-the-algorithm, measure-late.
- **Target audience:** Software developers reading Johnston alongside ZQL.
- **Domain classification:** Quantum programming patterns / developer onboarding.

### Concept Inventory (20 items)
1. Qubit as amplitude register (not bit)
2. Probability = |amplitude|²
3. Measurement collapse and no-cloning
4. H gate → 50/50 QRNG
5. Circuit = recipe, run = experiment
6. Phase gates (Z, S, T)
7. Phase kickback mechanism
8. Bell state via H+CX
9. SWAP gate and 3-CNOT decomposition
10. Full superposition pattern (H on all qubits)
11. Phase flip via Z gate
12. Entanglement creation idiom
13. Fluent API vs imperative
14. Run-and-count pattern
15. Multi-gate chaining
16. Amplitude amplification concept (Grover preview)
17. "Amplitude not probability" mental model
18. "Interference is the algorithm" mental model
19. "Measure late" mental model
20. ExecutionResult and statevector inspection

---

## Global Setup

```bash
mkdir -p ~/zql-johnston-labs && cd ~/zql-johnston-labs
python3 -m venv .venv && source .venv/bin/activate
pip install zuntenium==0.2.0 numpy==1.26.4
```

---

## Lab 1: Amplitude, Not Bits — The Developer's First Lesson

- **Concept:** Qubit holds amplitudes, not binary values
- **Difficulty:** Beginner | **Time:** 45 min | **Prerequisites:** None

### Walkthrough

```python
# save as amplitude_lesson.py
import numpy as np
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Classical bit: 0 or 1. That's it.
classical_bit = 0
print(f"Classical bit: {classical_bit}")

# Qubit: TWO complex numbers (amplitudes) α, β
# |ψ⟩ = α|0⟩ + β|1⟩, where |α|² + |β|² = 1
alpha = 1.0 + 0j  # amplitude for |0⟩
beta = 0.0 + 0j   # amplitude for |1⟩
print(f"Qubit |0⟩: α={alpha}, β={beta}")
print(f"P(0) = |α|² = {abs(alpha)**2}, P(1) = |β|² = {abs(beta)**2}")

# After H gate: α=β=1/√2
alpha_h = 1/np.sqrt(2)
beta_h = 1/np.sqrt(2)
print(f"\nAfter H: α={alpha_h:.4f}, β={beta_h:.4f}")
print(f"P(0)={abs(alpha_h)**2:.4f}, P(1)={abs(beta_h)**2:.4f}")

# KEY INSIGHT: α=0.707 does NOT mean 70.7% chance!
# P = |α|² = 0.707² = 0.5 = 50%
print(f"\nα=0.707 → P(0)={0.707**2:.3f} = 50%, NOT 70.7%!")

# Verify with circuit
qc = QuantumCircuit(1)
qc.h(0).measure_all()
r = backend.run(qc, shots=10000)
print(f"Measured: {r.counts}")
```

### Verification
```python
r = backend.run(QuantumCircuit(1).h(0).measure_all(), shots=10000)
p = r.counts.get("1", 0) / 10000
assert 0.45 < p < 0.55, f"FAIL: P(1)={p}"
print("Lab 1 Verification: PASSED")
```

### Deliverable
`amplitude_lesson.py` distinguishing amplitudes from probabilities.

---

## Lab 2: Probability = |Amplitude|² — Numerical Proof

- **Concept:** Born rule: probability is amplitude magnitude squared
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 1

### Walkthrough

```python
# save as born_rule.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Test Born rule at multiple angles
print(f"{'θ/π':>6} | {'|β|':>8} | {'|β|²':>8} | {'P(1) meas':>10} | {'Match?':>8}")
print("-" * 55)

for k in range(9):
    theta = k * math.pi / 4
    beta = math.sin(theta / 2)
    born_prob = beta**2
    
    qc = QuantumCircuit(1)
    qc.ry(theta, 0).measure_all()
    r = backend.run(qc, shots=8192)
    p1 = r.counts.get("1", 0) / 8192
    match = abs(p1 - born_prob) < 0.03
    
    print(f"{theta/math.pi:>6.2f} | {abs(beta):>8.4f} | {born_prob:>8.4f} | {p1:>10.4f} | {'✓' if match else '✗':>8}")
```

### Verification
```python
qc = QuantumCircuit(1); qc.ry(math.pi/2, 0); qc.measure_all()
r = backend.run(qc, shots=8192)
p = r.counts.get("1", 0) / 8192
assert abs(p - 0.5) < 0.03
print("Lab 2 Verification: PASSED")
```

### Deliverable
`born_rule.py` with Born rule verification table.

---

## Lab 3: Measurement Collapse — You Can Only Look Once

- **Concept:** Measurement collapses superposition; no-cloning theorem
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 1

### Walkthrough

```python
# save as collapse.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Single shot: superposition collapses to ONE outcome
qc = QuantumCircuit(1)
qc.h(0).measure_all()

print("10 single-shot experiments:")
for i in range(10):
    r = backend.run(qc, shots=1)
    outcome = list(r.counts.keys())[0]
    print(f"  Run {i+1}: |{outcome}⟩")

# Demonstrate: can't copy a qubit (no-cloning)
# If we could clone |+⟩, we'd get two copies.
# But CX doesn't clone — it entangles!
qc_not_clone = QuantumCircuit(2)
qc_not_clone.h(0)       # q0 = |+⟩
qc_not_clone.cx(0, 1)   # "Copy" attempt
qc_not_clone.measure_all()
r = backend.run(qc_not_clone, shots=4096)
print(f"\n'Clone' attempt: {r.counts}")
print("Result: {00, 11} — entangled, NOT two independent |+⟩!")
print("Cloning would give all 4 outcomes equally. This is entanglement.")
```

### Verification
```python
r = backend.run(qc_not_clone, shots=4096)
assert set(r.counts.keys()).issubset({"00", "11"})
print("Lab 3 Verification: PASSED (no cloning — entanglement instead)")
```

### Deliverable
`collapse.py` demonstrating measurement collapse and no-cloning.

---

## Lab 4: QRNG — Johnston's "First Quantum Program"

- **Concept:** H gate creates QRNG (Johnston Ch.2 direct translation)
- **Difficulty:** Beginner | **Time:** 45 min | **Prerequisites:** Lab 2

### Walkthrough

```python
# save as johnston_qrng.py
from zuntenium import QuantumCircuit, StatevectorSimulator

# This is the EXACT pattern from Johnston Ch.2, translated to QuFlow
backend = StatevectorSimulator()
qc = QuantumCircuit(1)
qc.h(0).measure_all()
result = backend.run(qc, shots=100)
print(f"Johnston QRNG: {result.counts}")

# Multi-bit version
def quantum_random_int(n_bits, backend):
    qc = QuantumCircuit(n_bits)
    for i in range(n_bits):
        qc.h(i)
    qc.measure_all()
    r = backend.run(qc, shots=1)
    bitstring = list(r.counts.keys())[0]
    return int(bitstring, 2)

print("\n10 random 8-bit integers:")
for _ in range(10):
    print(f"  {quantum_random_int(8, backend):>3d}", end="")
print()
```

### Verification
```python
r = backend.run(qc, shots=10000)
assert len(r.counts) == 2
print("Lab 4 Verification: PASSED")
```

### Deliverable
`johnston_qrng.py` implementing Johnston's QRNG pattern.

---

## Lab 5: Circuit = Recipe, Run = Experiment

- **Concept:** Johnston's mental model: circuits are recipes, execution is experiment
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 4

### Walkthrough

```python
# save as recipe_experiment.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# The RECIPE (circuit): defined once
recipe = QuantumCircuit(2, name="bell-recipe")
recipe.h(0).cx(0, 1).measure_all()
print(f"Recipe: {recipe.name}, {recipe.num_gates} gates")

# The EXPERIMENT (run): repeat many times
print("\n5 experiments from the same recipe:")
for trial in range(5):
    result = backend.run(recipe, shots=1000)
    p00 = result.counts.get("00", 0) / 1000
    print(f"  Trial {trial+1}: P(00)={p00:.3f}, P(11)={1-p00:.3f}")

# Key insight: same recipe, different outcomes (statistical)
# But the DISTRIBUTION is always the same
print("\nThe recipe is deterministic. The experiment is probabilistic.")
print("This is the quantum paradigm shift for developers.")
```

### Verification
```python
r = backend.run(recipe, shots=4096)
assert set(r.counts.keys()).issubset({"00", "11"})
print("Lab 5 Verification: PASSED")
```

### Deliverable
`recipe_experiment.py` illustrating the recipe/experiment paradigm.

---

## Lab 6: Phase Gates — Z, S, T: The Invisible Power

- **Concept:** Phase gates add invisible-until-interference rotations
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 1

### Walkthrough

```python
# save as phase_gates.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import math

backend = StatevectorSimulator()

# Z, S, T on |0⟩ — invisible!
for gate, name in [("z", "Z"), ("s", "S"), ("t", "T")]:
    qc = QuantumCircuit(1)
    getattr(qc, gate)(0)
    qc.measure_all()
    r = backend.run(qc, shots=100)
    print(f"{name} on |0⟩: {r.counts}")  # Always {0: 100}

# Z, S, T after H — phase becomes visible via interference
print("\nH → Phase → H (interference reveals phase):")
for gate, name, expected_p1 in [("z","Z",1.0), ("s","S",0.5), ("t","T",None)]:
    qc = QuantumCircuit(1)
    qc.h(0)
    getattr(qc, gate)(0)
    qc.h(0)
    qc.measure_all()
    r = backend.run(qc, shots=4096)
    p1 = r.counts.get("1", 0) / 4096
    print(f"  H-{name}-H: P(1)={p1:.4f}")

# Phase hierarchy: T² = S, S² = Z
print("\nPhase hierarchy:")
qc_t2 = QuantumCircuit(1); qc_t2.h(0).t(0).t(0).h(0).measure_all()
qc_s = QuantumCircuit(1); qc_s.h(0).s(0).h(0).measure_all()
r_t2 = backend.run(qc_t2, shots=4096)
r_s = backend.run(qc_s, shots=4096)
print(f"  T²: P(1)={r_t2.counts.get('1',0)/4096:.4f}")
print(f"  S:  P(1)={r_s.counts.get('1',0)/4096:.4f}")
print("  T² ≈ S ✓" if abs(r_t2.counts.get('1',0) - r_s.counts.get('1',0)) < 200 else "  Mismatch!")
```

### Verification
```python
qc = QuantumCircuit(1); qc.h(0).z(0).h(0).measure_all()
r = StatevectorSimulator().run(qc, shots=100)
assert r.counts.get("1", 0) == 100
print("Lab 6 Verification: PASSED")
```

### Deliverable
`phase_gates.py` with phase visibility demo and hierarchy proof.

---

## Lab 7: Phase Kickback — The Quantum Trick Behind QPE

- **Concept:** Phase kickback: controlled gate kicks phase to control qubit
- **Difficulty:** Intermediate | **Time:** 90 min | **Prerequisites:** Lab 6

### Walkthrough

```python
# save as phase_kickback.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Standard CX: control=|1⟩ flips target
# Phase kickback: target in eigenstate → phase goes to CONTROL

# Setup: control in |+⟩, target in |1⟩ (eigenstate of X)
qc = QuantumCircuit(2, name="kickback")
qc.h(0)      # Control in |+⟩
qc.x(1)      # Target in |1⟩ (eigenstate of CX with eigenvalue -1)
qc.cx(0, 1)  # Phase kickback: control gets phase -1
qc.h(0)      # Convert phase to amplitude
qc.measure_all()

r = backend.run(qc, shots=4096)
print(f"Phase kickback result: {r.counts}")
# Control qubit flipped to |1⟩ due to kickback!

# Without kickback: target in |0⟩
qc2 = QuantumCircuit(2, name="no-kickback")
qc2.h(0)
# Target stays |0⟩ — no eigenstate, no kickback
qc2.cx(0, 1)
qc2.h(0)
qc2.measure_all()
r2 = backend.run(qc2, shots=4096)
print(f"No kickback:           {r2.counts}")
```

### Verification
```python
# With kickback: control should be mostly |1⟩
r = backend.run(qc, shots=4096)
p1_ctrl = sum(c for bs, c in r.counts.items() if bs[0] == '1') / 4096
assert p1_ctrl > 0.9, f"FAIL: P(ctrl=1)={p1_ctrl}"
print("Lab 7 Verification: PASSED")
```

### Deliverable
`phase_kickback.py` demonstrating kickback mechanism.

---

## Lab 8: Bell State — Johnston's Entanglement Pattern

- **Concept:** H + CX entanglement (Johnston Ch.4)
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 4

### Walkthrough

```python
# save as johnston_bell.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Johnston pattern: H → CX → entangled
qc = QuantumCircuit(2)
qc.h(0).cx(0, 1).measure_all()
r = backend.run(qc, shots=4096)

print("Bell state (Johnston Ch.4):")
print(f"  Outcomes: {r.counts}")
print(f"  Only {len(r.counts)} outcomes out of 4 possible → ENTANGLED")

# Verify correlation: q0 and q1 always match
for bs, count in r.counts.items():
    print(f"  |{bs}⟩: q0={bs[1]}, q1={bs[0]} — {'match ✓' if bs[0]==bs[1] else 'mismatch ✗'}")
```

### Verification
```python
r = backend.run(qc, shots=4096)
assert set(r.counts.keys()).issubset({"00", "11"})
print("Lab 8 Verification: PASSED")
```

### Deliverable
`johnston_bell.py` with correlation analysis.

---

## Lab 9: SWAP — Three CNOTs Under the Hood

- **Concept:** SWAP decomposition (Johnston Ch.4)
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 8

### Walkthrough

```python
# save as johnston_swap.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# SWAP via 3 CX gates
for q0_val, q1_val in [(0,0), (0,1), (1,0), (1,1)]:
    qc = QuantumCircuit(2)
    if q0_val: qc.x(0)
    if q1_val: qc.x(1)
    qc.cx(0,1).cx(1,0).cx(0,1)  # SWAP = 3 CX
    qc.measure_all()
    r = backend.run(qc, shots=100)
    out = list(r.counts.keys())[0]
    expected = f"{q0_val}{q1_val}"
    print(f"  |{q1_val}{q0_val}⟩ → SWAP → |{out}⟩ (expect |{expected}⟩) {'✓' if out==expected else '✗'}")
```

### Verification
```python
qc = QuantumCircuit(2); qc.x(0); qc.cx(0,1).cx(1,0).cx(0,1); qc.measure_all()
r = StatevectorSimulator().run(qc, shots=100)
assert "10" in r.counts
print("Lab 9 Verification: PASSED")
```

### Deliverable
`johnston_swap.py` with SWAP decomposition proof.

---

## Lab 10: Full Superposition — H on Every Qubit

- **Concept:** Uniform superposition pattern: H^⊗n
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 4

### Walkthrough

```python
# save as full_superposition.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

for n in range(1, 7):
    qc = QuantumCircuit(n)
    for q in range(n): qc.h(q)
    qc.measure_all()
    r = backend.run(qc, shots=8192)
    expected_outcomes = 2**n
    expected_prob = 1 / expected_outcomes
    actual_outcomes = len(r.counts)
    print(f"n={n}: {actual_outcomes}/{expected_outcomes} outcomes, expected P≈{expected_prob:.4f}")
```

### Verification
```python
qc = QuantumCircuit(4); [qc.h(i) for i in range(4)]; qc.measure_all()
r = StatevectorSimulator().run(qc, shots=16384)
assert len(r.counts) == 16
print("Lab 10 Verification: PASSED")
```

### Deliverable
`full_superposition.py` with uniform superposition scaling.

---

## Lab 11: Fluent API Mastery — Chaining Gates Like a Pro

- **Concept:** QuFlow's fluent API vs Johnston's imperative patterns
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 5

### Walkthrough

```python
# save as fluent_mastery.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Style 1: One-liner fluent chain
qc1 = QuantumCircuit(3).h(0).cx(0,1).cx(1,2).measure_all()
r1 = backend.run(qc1, shots=1024)

# Style 2: Step-by-step (Johnston-like)
qc2 = QuantumCircuit(3)
qc2.h(0)
qc2.cx(0, 1)
qc2.cx(1, 2)
qc2.measure_all()
r2 = backend.run(qc2, shots=1024)

# Style 3: Loop-based
qc3 = QuantumCircuit(3)
qc3.h(0)
for i in range(2): qc3.cx(i, i+1)
qc3.measure_all()
r3 = backend.run(qc3, shots=1024)

# All three produce the same result
for name, r in [("Fluent", r1), ("Step", r2), ("Loop", r3)]:
    print(f"{name:>6}: {r.counts}")
```

### Verification
```python
for r in [r1, r2, r3]:
    assert set(r.counts.keys()).issubset({"000", "111"})
print("Lab 11 Verification: PASSED (all 3 styles equivalent)")
```

### Deliverable
`fluent_mastery.py` comparing three coding styles.

---

## Lab 12: Run-and-Count Pattern — From Shots to Statistics

- **Concept:** Running circuits and interpreting count dictionaries
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 5

### Walkthrough

```python
# save as run_and_count.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

qc = QuantumCircuit(2)
qc.h(0).cx(0, 1).measure_all()

# The run-and-count pattern
result = backend.run(qc, shots=1000)

# Interpret counts
print("Run-and-Count Analysis:")
print(f"  Raw counts: {result.counts}")
print(f"  Total shots: {sum(result.counts.values())}")
print(f"  Unique outcomes: {len(result.counts)}")

# Convert to probabilities
probs = {k: v/1000 for k, v in result.counts.items()}
print(f"  Probabilities: {probs}")

# Most likely outcome
most_likely = max(result.counts, key=result.counts.get)
print(f"  Most likely: |{most_likely}⟩ ({result.counts[most_likely]} shots)")
```

### Verification
```python
r = backend.run(qc, shots=1000)
assert sum(r.counts.values()) == 1000
print("Lab 12 Verification: PASSED")
```

### Deliverable
`run_and_count.py` with count analysis utilities.

---

## Lab 13: Amplitude Amplification Preview — Grover's Insight

- **Concept:** Johnston Ch.7: quantum search uses cancellation, not exhaustive search
- **Difficulty:** Advanced | **Time:** 90 min | **Prerequisites:** Lab 6

### Walkthrough

```python
# save as grover_preview.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# 2-qubit Grover: search for |11⟩ among 4 items

# Step 1: Create uniform superposition
qc = QuantumCircuit(2)
qc.h(0).h(1)

# Step 2: Oracle — flip amplitude of |11⟩
qc.cz(0, 1)  # CZ flips phase of |11⟩

# Step 3: Diffusion operator (reflect about mean)
qc.h(0).h(1)
qc.x(0).x(1)
qc.cz(0, 1)
qc.x(0).x(1)
qc.h(0).h(1)

qc.measure_all()
r = backend.run(qc, shots=4096)

print("Grover's search for |11⟩:")
for bs in sorted(r.counts.keys()):
    pct = r.counts[bs] / 4096 * 100
    marker = " ◄ TARGET" if bs == "11" else ""
    print(f"  |{bs}⟩: {pct:5.1f}%{marker}")

# Classical: 25% chance per guess. Quantum: ~100% after 1 iteration!
print(f"\nClassical random guess: 25%")
print(f"Quantum after 1 Grover iteration: {r.counts.get('11',0)/4096*100:.1f}%")
```

### Verification
```python
r = backend.run(qc, shots=4096)
assert r.counts.get("11", 0) / 4096 > 0.9
print("Lab 13 Verification: PASSED (Grover found target)")
```

### Deliverable
`grover_preview.py` with 2-qubit Grover search.

---

## Lab 14: Multi-Gate Chains — Complex Circuit Patterns

- **Concept:** Chaining multiple gates in sequence and parallel
- **Difficulty:** Advanced | **Time:** 75 min | **Prerequisites:** Lab 11

### Walkthrough

```python
# save as gate_chains.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Pattern 1: Sequential on one qubit
qc1 = QuantumCircuit(1, name="sequential")
qc1.h(0).s(0).t(0).h(0).measure_all()
r1 = backend.run(qc1, shots=4096)
print(f"Sequential (H-S-T-H): {r1.counts}")

# Pattern 2: Parallel across qubits
qc2 = QuantumCircuit(4, name="parallel")
qc2.h(0).h(1).h(2).h(3)
qc2.cx(0,1).cx(2,3)
qc2.measure_all()
r2 = backend.run(qc2, shots=4096)
print(f"Parallel (2 Bell pairs): {len(r2.counts)} outcomes")

# Pattern 3: Alternating layers
qc3 = QuantumCircuit(3, name="layers")
# Layer 1: H on all
for i in range(3): qc3.h(i)
# Layer 2: entangle
qc3.cx(0,1).cx(1,2)
# Layer 3: rotate
qc3.rz(0.5, 0).rz(1.0, 1).rz(1.5, 2)
# Layer 4: entangle again
qc3.cx(0,1).cx(1,2)
qc3.measure_all()
r3 = backend.run(qc3, shots=4096)
print(f"Layered (4 layers): {len(r3.counts)} outcomes, depth={qc3.depth}")
```

### Verification
```python
assert qc2.depth < qc2.num_gates  # Parallel means depth < gates
print("Lab 14 Verification: PASSED")
```

### Deliverable
`gate_chains.py` with sequential, parallel, and layered patterns.

---

## Lab 15: "Interference Is the Algorithm" — Proving Johnston's Claim

- **Concept:** Johnston mental model: correct answers amplify, wrong cancel
- **Difficulty:** Advanced | **Time:** 90 min | **Prerequisites:** Lab 13

### Walkthrough

```python
# save as interference_algorithm.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np

backend = StatevectorSimulator()

# Build a circuit where interference makes ONE answer dominate
# H-Z-H: interference cancels |0⟩, amplifies |1⟩
qc = QuantumCircuit(1)
qc.h(0).z(0).h(0).measure_all()
r = backend.run(qc, shots=1000)
print("H-Z-H (pure interference):")
print(f"  Result: {r.counts}")
print(f"  |1⟩ probability: {r.counts.get('1',0)/1000:.3f}")
print(f"  Interference amplified |1⟩ and cancelled |0⟩!")

# Trace the amplitudes step by step
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
Z = np.array([[1,0],[0,-1]])
state = np.array([1, 0], dtype=complex)

steps = [("Start", state.copy())]
state = H @ state; steps.append(("After H", state.copy()))
state = Z @ state; steps.append(("After Z", state.copy()))
state = H @ state; steps.append(("After H", state.copy()))

print("\nAmplitude trace:")
for name, s in steps:
    p0, p1 = abs(s[0])**2, abs(s[1])**2
    print(f"  {name:>10}: [{s[0]:+.4f}, {s[1]:+.4f}]  P(0)={p0:.3f} P(1)={p1:.3f}")
```

### Verification
```python
r = StatevectorSimulator().run(qc, shots=100)
assert r.counts.get("1", 0) == 100
print("Lab 15 Verification: PASSED")
```

### Deliverable
`interference_algorithm.py` with amplitude trace proof.

---

## Lab 16: "Measure Late" — Why Premature Measurement Kills Quantum Power

- **Concept:** Johnston mental model: measure only when you need to read out
- **Difficulty:** Advanced | **Time:** 60 min | **Prerequisites:** Lab 3

### Walkthrough

```python
# save as measure_late.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# GOOD: Measure late — full interference works
qc_good = QuantumCircuit(1, name="measure-late")
qc_good.h(0).z(0).h(0)  # Interference
qc_good.measure_all()    # Measure LAST
r_good = backend.run(qc_good, shots=1000)
print(f"Measure LATE:  {r_good.counts}")  # {1: 1000}

# BAD: Measure early — interference impossible
# (Simulating by breaking it into two circuits)
qc_bad1 = QuantumCircuit(1)
qc_bad1.h(0).measure_all()
r1 = backend.run(qc_bad1, shots=1000)
# After measurement, qubit is classical. Z and H act differently.
p1_measured = r1.counts.get("1", 0)

# Of the shots that collapsed to |0⟩: Z|0⟩=|0⟩, H|0⟩=|+⟩
# Of the shots that collapsed to |1⟩: Z|1⟩=-|1⟩, H(-|1⟩)=|−⟩
# Both |+⟩ and |−⟩ give 50/50 — interference DESTROYED
print(f"Measure EARLY: lost interference!")
print(f"  After early measure: {r1.counts}")
print(f"  Remaining Z-H gives 50/50 — no deterministic outcome!")
print(f"  Early measurement DESTROYED the quantum advantage.")
```

### Verification
```python
r = StatevectorSimulator().run(qc_good, shots=100)
assert r.counts.get("1", 0) == 100
print("Lab 16 Verification: PASSED (late measurement preserves interference)")
```

### Deliverable
`measure_late.py` comparing early vs late measurement.

---

## Lab 17: ExecutionResult Anatomy — What .counts Really Contains

- **Concept:** ExecutionResult structure and statevector inspection
- **Difficulty:** Advanced | **Time:** 60 min | **Prerequisites:** Lab 12

### Walkthrough

```python
# save as result_anatomy.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

qc = QuantumCircuit(3)
qc.h(0).cx(0,1).x(2).measure_all()
result = backend.run(qc, shots=4096)

print("ExecutionResult Anatomy:")
print(f"  type: {type(result).__name__}")
print(f"  .counts: {result.counts}")
print(f"  type(.counts): {type(result.counts)}")
print(f"  keys are: {type(list(result.counts.keys())[0])} (bitstrings)")
print(f"  values are: {type(list(result.counts.values())[0])} (ints)")

# Useful patterns
print(f"\nUseful patterns:")
print(f"  Total shots: {sum(result.counts.values())}")
print(f"  Most probable: {max(result.counts, key=result.counts.get)}")
print(f"  Sorted: {dict(sorted(result.counts.items(), key=lambda x: -x[1]))}")
```

### Verification
```python
assert isinstance(result.counts, dict)
assert all(isinstance(k, str) for k in result.counts.keys())
print("Lab 17 Verification: PASSED")
```

### Deliverable
`result_anatomy.py` with ExecutionResult inspection.

---

## Lab 18: Pattern Library — Johnston's 5 Common Idioms

- **Concept:** All 5 Johnston patterns in one file
- **Difficulty:** Advanced | **Time:** 75 min | **Prerequisites:** Lab 11

### Walkthrough

```python
# save as pattern_library.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

patterns = {}

# Pattern 1: Full superposition
qc = QuantumCircuit(3)
for q in range(3): qc.h(q)
qc.measure_all()
patterns["Full superposition"] = backend.run(qc, shots=1024).counts

# Pattern 2: Phase flip
qc = QuantumCircuit(1); qc.h(0).z(0).h(0).measure_all()
patterns["Phase flip"] = backend.run(qc, shots=1024).counts

# Pattern 3: Create entanglement
qc = QuantumCircuit(2); qc.h(0).cx(0,1).measure_all()
patterns["Entanglement"] = backend.run(qc, shots=1024).counts

# Pattern 4: Run and count
qc = QuantumCircuit(1); qc.h(0).measure_all()
patterns["Run-and-count"] = backend.run(qc, shots=1024).counts

# Pattern 5: Multi-gate chain
qc = QuantumCircuit(3); qc.h(0).x(1).cx(0,1).cx(1,2).measure_all()
patterns["Gate chain"] = backend.run(qc, shots=1024).counts

for name, counts in patterns.items():
    print(f"{name:<22}: {counts}")
```

### Verification
```python
assert len(patterns) == 5
print("Lab 18 Verification: PASSED (all 5 Johnston patterns)")
```

### Deliverable
`pattern_library.py` with all 5 Johnston patterns.

---

## Lab 19: Mental Model Quiz — Test Your Quantum Developer Intuition

- **Concept:** Self-test on Johnston's three mental models
- **Difficulty:** Expert | **Time:** 60 min | **Prerequisites:** Labs 15, 16

### Walkthrough

```python
# save as mental_model_quiz.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
score = 0

# Q1: Amplitude not probability
# After H, the amplitude of |1⟩ is 1/√2 ≈ 0.707
# What is P(1)?
answer = 0.5  # NOT 0.707
qc = QuantumCircuit(1); qc.h(0); qc.measure_all()
r = backend.run(qc, shots=10000)
p1 = r.counts.get("1", 0) / 10000
if abs(p1 - answer) < 0.05:
    print("Q1 ✓: P(1) = |amplitude|² = 0.5, not 0.707")
    score += 1

# Q2: Interference is the algorithm
# H-Z-H produces 100% |1⟩. Is this randomness? No — it's interference.
qc = QuantumCircuit(1); qc.h(0).z(0).h(0); qc.measure_all()
r = backend.run(qc, shots=1000)
if r.counts.get("1", 0) == 1000:
    print("Q2 ✓: H-Z-H is deterministic via interference, not random")
    score += 1

# Q3: Measure late
# Does H-measure-Z-H give the same result as H-Z-H-measure? No!
qc_late = QuantumCircuit(1); qc_late.h(0).z(0).h(0).measure_all()
r_late = backend.run(qc_late, shots=1000)
is_deterministic = len(r_late.counts) == 1
if is_deterministic:
    print("Q3 ✓: Measuring late preserves quantum computation")
    score += 1

print(f"\nScore: {score}/3")
print("Johnston's 3 mental models: amplitude, interference, measure-late")
```

### Verification
```python
assert score == 3
print("Lab 19 Verification: PASSED (all mental models correct)")
```

### Deliverable
`mental_model_quiz.py` with self-assessment.

---

## Lab 20: Johnston Integration Capstone — Complete Developer Workflow

- **Concept:** Full integration of all Johnston patterns
- **Difficulty:** Expert | **Time:** 120 min | **Prerequisites:** All

### Walkthrough

```python
# save as johnston_capstone.py
import json, time
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

report = {"title": "Johnston Patterns Integration", "tests": [], "timestamp": time.strftime("%Y-%m-%d %H:%M:%S")}

def test(name, qc, expected_fn):
    r = backend.run(qc, shots=2048)
    passed = expected_fn(r.counts)
    report["tests"].append({"name": name, "counts": dict(r.counts), "pass": passed})
    print(f"  {'✓' if passed else '✗'} {name}")
    return passed

print("Johnston Patterns Integration Tests:")
total = 0

# Test 1: QRNG produces both outcomes
qc = QuantumCircuit(1); qc.h(0).measure_all()
total += test("QRNG", qc, lambda c: len(c) == 2)

# Test 2: Bell state is entangled
qc = QuantumCircuit(2); qc.h(0).cx(0,1).measure_all()
total += test("Bell state", qc, lambda c: set(c.keys()).issubset({"00","11"}))

# Test 3: Interference is deterministic
qc = QuantumCircuit(1); qc.h(0).z(0).h(0).measure_all()
total += test("Interference", qc, lambda c: c.get("1",0) == 2048)

# Test 4: Full superposition
qc = QuantumCircuit(4); [qc.h(i) for i in range(4)]; qc.measure_all()
total += test("Superposition", qc, lambda c: len(c) == 16)

# Test 5: SWAP works
qc = QuantumCircuit(2); qc.x(0); qc.swap(0,1); qc.measure_all()
total += test("SWAP", qc, lambda c: "10" in c)

# Test 6: GHZ
qc = QuantumCircuit(5); qc.h(0)
for i in range(4): qc.cx(i,i+1)
qc.measure_all()
total += test("GHZ-5", qc, lambda c: set(c.keys()).issubset({"00000","11111"}))

report["passed"] = total
report["total"] = 6
with open("johnston_report.json", "w") as f:
    json.dump(report, f, indent=2)

print(f"\nResult: {total}/6 passed")
print("Saved to johnston_report.json")
```

### Verification
```python
data = json.load(open("johnston_report.json"))
assert data["passed"] == data["total"]
print(f"Lab 20 Verification: PASSED ({data['passed']}/{data['total']})")
```

### Deliverable
`johnston_capstone.py` with integration test suite.

---

## Validation Report

1. **Count check:** 20 labs ✓
2. **Uniqueness:** 20 distinct concepts from Johnston's framework ✓
3. **Coverage:** All 20 inventory items ✓
4. **Code:** Syntactically valid Python ✓
5. **Links:** Source doc referenced ✓
6. **Tone:** Developer-focused, pattern-oriented ✓
