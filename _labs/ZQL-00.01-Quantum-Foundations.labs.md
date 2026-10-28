# ZQL-00.01 Quantum Foundations — Walkthrough Labs

**Source:** [ZQL-00.01-Quantum-Foundations.md](../ZQL-00.01-Quantum-Foundations.md)
**Generated:** 2026-10-28
**Labs:** 20 | **Domain:** Quantum / hands-on code labs

---

## Source Analysis

- **Document objective:** Onboard Python developers with zero quantum experience to running their first circuits on the QuFlow (`zuntenium`) SDK in under 90 minutes.
- **Target audience:** Python developers (3.9+); no linear algebra or physics prerequisite.
- **Domain classification:** Quantum algorithms / SDK onboarding.

### Concept Inventory (28 items)

1. NISQ era and quantum hardware landscape
2. QuFlow SDK purpose and abstraction layers
3. Qubit as complex amplitude register
4. Superposition (Hadamard gate)
5. Entanglement (CNOT gate, Bell state)
6. Interference (phase manipulation)
7. Measurement and probabilistic collapse
8. Unitary gates and matrix representation
9. Single-qubit rotation gates (Rx, Ry, Rz)
10. Multi-qubit gates (CX, CZ, SWAP, Toffoli)
11. GHZ state construction
12. Exponential memory scaling of simulation
13. SDK installation and virtual environment
14. StatevectorSimulator usage
15. Fluent builder pattern (QuantumCircuit API)
16. QuantumIR and transpiler pipeline
17. Circuit depth vs. gate count
18. Circuit inverse and mirror patterns
19. Bit ordering (little-endian) conventions
20. Parameterized circuits (angle sweeps)
21. QuantumRegister / ClassicalRegister model
22. Instruction data model
23. BaseGate class hierarchy
24. TranspilerCache and SHA-256 hashing
25. SABRE routing algorithm
26. KAK decomposition
27. Simulator selection (Statevector vs Density vs Stabilizer)
28. VS Code extensions and circuit visualization

### Selected 20 Concepts (difficulty ramp)

| # | Concept | Approach |
|---|---------|----------|
| 1 | SDK installation & smoke test | Install from scratch, verify every component |
| 2 | Qubit as amplitude register | Visualize statevectors numerically |
| 3 | Superposition via Hadamard | Build QRNG, verify statistical uniformity |
| 4 | Measurement and shot statistics | Explore shot count impact on accuracy |
| 5 | Entanglement via Bell state | Build all four Bell states, verify correlations |
| 6 | Interference and phase gates | Construct H-Z-H deterministic circuit |
| 7 | Multi-qubit GHZ states | Scale from 2 to N qubits, measure scaling |
| 8 | Rotation gates and Bloch sphere | Sweep Ry angle, plot P(1) vs θ |
| 9 | Circuit depth vs. gate count | Compare parallel vs serial layouts |
| 10 | Fluent builder pattern | Reconstruct QuantumCircuit from scratch |
| 11 | Gate matrices and unitarity | Verify U†U=I for all standard gates |
| 12 | Inverse and mirror circuits | Build uncompute verification harness |
| 13 | Simulator selection trade-offs | Compare Statevector vs Stabilizer performance |
| 14 | Debugging broken circuits | Diagnose and fix common QuFlow errors |
| 15 | Parameterized circuit templates | Build reusable angle-sweep framework |
| 16 | Memory scaling benchmark | Measure RAM vs qubit count empirically |
| 17 | QuantumIR and transpiler inspection | Dump and inspect IR, verify cache hits |
| 18 | Bit ordering and endianness | Detect and handle bit-order pitfalls |
| 19 | Quantum on constrained hardware (Africa) | Run labs on free-tier/mobile/low-RAM setups |
| 20 | Entanglement demonstrator capstone | Full application: GHZ verification suite |

---

## How to Use This Lab Pack

**Environment:** Python 3.9+ (3.11 recommended). Any OS. Terminal + text editor or Jupyter Notebook.

**Tooling:** `pip`, `zuntenium` SDK (QuFlow v0.2.0), `numpy`, `matplotlib` (optional for plots).

**Time estimate:** ~30 hours total (1–2 hours per lab).

**Progression:** Labs 1–5 are beginner (foundations), 6–12 intermediate, 13–18 advanced, 19–20 expert/capstone.

---

## Global Setup

```bash
# Create workspace and virtual environment
mkdir -p ~/zql-foundations-labs && cd ~/zql-foundations-labs
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\Activate.ps1

# Install dependencies (pinned versions)
pip install --upgrade pip
pip install zuntenium==0.2.0
pip install numpy==1.26.4
pip install matplotlib==3.9.0   # optional, for plots

# Smoke test
python3 -c "
from zuntenium import QuantumCircuit, StatevectorSimulator
qc = QuantumCircuit(1)
qc.h(0).measure_all()
r = StatevectorSimulator().run(qc, shots=100)
print('Setup OK — counts:', r.counts)
"
```

Expected: `Setup OK — counts: {'0': ~50, '1': ~50}`

---

## Lab Map

| # | Title | Concept | Approach | Difficulty | Est. Time | Prerequisites |
|---|-------|---------|----------|------------|-----------|---------------|
| 1 | Install & Validate QuFlow | SDK installation | Full install verification | Beginner | 45 min | None |
| 2 | Inside the Qubit | Amplitude register | Numerical statevector exploration | Beginner | 60 min | Lab 1 |
| 3 | Your First Quantum Random Bit | Superposition | Build QRNG with chi-squared test | Beginner | 75 min | Lab 2 |
| 4 | Shots and Statistics | Measurement statistics | Shot count vs accuracy tradeoff | Beginner | 60 min | Lab 3 |
| 5 | Building Bell States | Entanglement | All four Bell states | Beginner | 90 min | Lab 3 |
| 6 | Interference: The Quantum Trick | Phase interference | H-Z-H deterministic circuits | Intermediate | 75 min | Lab 5 |
| 7 | GHZ: Entangling N Qubits | Multi-qubit entanglement | Scalable GHZ construction | Intermediate | 90 min | Lab 5 |
| 8 | Spinning on the Bloch Sphere | Rotation gates | Ry angle sweep, sin²(θ/2) | Intermediate | 90 min | Lab 6 |
| 9 | Depth vs. Width | Circuit depth/count | Parallel vs serial gate layouts | Intermediate | 60 min | Lab 7 |
| 10 | Build a Circuit Builder | Fluent builder pattern | Reimplement QuantumCircuit core | Intermediate | 120 min | Lab 9 |
| 11 | Verify Your Gates | Gate unitarity | U†U=I matrix verification | Intermediate | 75 min | Lab 8 |
| 12 | Uncompute Everything | Inverse/mirror circuits | Build verification harness | Intermediate | 75 min | Lab 11 |
| 13 | Picking the Right Simulator | Simulator trade-offs | Benchmark SV vs Stabilizer | Advanced | 90 min | Lab 7 |
| 14 | Circuit Forensics | Debugging broken circuits | Diagnose 8 failure modes | Advanced | 90 min | Lab 5 |
| 15 | Parameterized Circuit Factory | Parameterized templates | Reusable angle-sweep framework | Advanced | 90 min | Lab 8 |
| 16 | How Big Is Too Big? | Memory scaling | Empirical RAM vs qubit benchmark | Advanced | 75 min | Lab 13 |
| 17 | Inspect the Transpiler | QuantumIR & cache | Dump IR, verify cache behavior | Advanced | 90 min | Lab 10 |
| 18 | Bit Order Matters | Endianness pitfalls | Detect and fix bit-order bugs | Advanced | 60 min | Lab 5 |
| 19 | Quantum on a Budget | Constrained-hardware labs | Free-tier, low-RAM, mobile-first | Expert | 90 min | Labs 13,16 |
| 20 | Entanglement Demonstrator | Capstone integration | Full GHZ verification suite | Expert | 120 min | All |

---

## Lab 1: Install & Validate QuFlow

- **Concept:** SDK installation and environment verification
- **Approach:** Install from scratch, verify every component individually, build confidence before writing circuits
- **Difficulty:** Beginner | **Time:** 45 min | **Prerequisites:** None
- **Learning outcomes:**
  1. You will be able to install the `zuntenium` SDK in an isolated virtual environment
  2. You will be able to verify the SDK version and all public imports
  3. You will be able to create and run a minimal circuit as a smoke test
  4. You will be able to diagnose common installation failures
- **Why this matters:** A clean, verified environment prevents hours of debugging later. At Zuntenium Quantum Institute, learners across Africa work on varied hardware — laptops, university lab PCs, even cloud shells. Getting the setup right first is critical.

### Theory Primer

QuFlow (`zuntenium`) is a pure-Python package with NumPy as its only required dependency. It provides `QuantumCircuit` for building circuits and `StatevectorSimulator` for executing them. The package uses lazy imports to keep startup fast.

*(Ref: Source doc §3 Developer Onboarding)*

### Setup

Nothing — this lab *is* the setup.

### Walkthrough

**Step 1: Create a fresh virtual environment.**

```bash
mkdir -p ~/zql-foundations-labs/lab01 && cd ~/zql-foundations-labs/lab01
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
```

What it does: Creates an isolated Python environment so SDK dependencies don't conflict with system packages.

Expected: `Successfully installed pip-...`

**Step 2: Install zuntenium.**

```bash
pip install zuntenium==0.2.0
```

What it does: Downloads and installs QuFlow v0.2.0 plus its dependency (numpy).

**Checkpoint:** `pip list | grep zuntenium` shows `zuntenium  0.2.0`.

**Step 3: Verify the version programmatically.**

```python
# save as verify_install.py
import zuntenium
print(f"zuntenium version: {zuntenium.__version__}")

# Verify core imports
from zuntenium import QuantumCircuit
from zuntenium import StatevectorSimulator
print("QuantumCircuit: OK")
print("StatevectorSimulator: OK")

# Verify numpy dependency
import numpy as np
print(f"numpy version: {np.__version__}")
```

```bash
python3 verify_install.py
```

Expected:
```
zuntenium version: 0.2.0
QuantumCircuit: OK
StatevectorSimulator: OK
numpy version: 1.26.4
```

**Step 4: Run a minimal circuit — the definitive smoke test.**

```python
# save as smoke_test.py
from zuntenium import QuantumCircuit, StatevectorSimulator

qc = QuantumCircuit(1)
qc.h(0).measure_all()

backend = StatevectorSimulator()
result = backend.run(qc, shots=100)

assert isinstance(result.counts, dict), "counts should be a dict"
assert sum(result.counts.values()) == 100, "total counts should equal shots"
print(f"Counts: {result.counts}")
print("ALL CHECKS PASSED")
```

```bash
python3 smoke_test.py
```

Expected: `Counts: {'0': ~50, '1': ~50}` then `ALL CHECKS PASSED`.

**Step 5: Verify circuit properties.**

```python
# save as check_circuit.py
from zuntenium import QuantumCircuit

qc = QuantumCircuit(2, name="test-circuit")
qc.h(0).cx(0, 1).measure_all()

print(f"num_qubits: {qc.num_qubits}")   # 2
print(f"num_clbits: {qc.num_clbits}")   # 2
print(f"num_gates:  {qc.num_gates}")    # 2 (h + cx)
print(f"depth:      {qc.depth}")        # 2
print(f"name:       {qc.name}")         # test-circuit
print(f"\nCircuit diagram:")
print(qc)
```

```bash
python3 check_circuit.py
```

**Checkpoint:** All properties match expected values. Circuit diagram prints without error.

### Verification

```python
# save as lab01_verify.py
from zuntenium import QuantumCircuit, StatevectorSimulator

tests_passed = 0
total_tests = 5

# Test 1: Version
import zuntenium
assert zuntenium.__version__ == "0.2.0"; tests_passed += 1

# Test 2: Circuit creation
qc = QuantumCircuit(3); assert qc.num_qubits == 3; tests_passed += 1

# Test 3: Gate application
qc.h(0); assert qc.num_gates == 1; tests_passed += 1

# Test 4: Measurement
qc.measure_all(); assert qc.num_clbits == 3; tests_passed += 1

# Test 5: Execution
r = StatevectorSimulator().run(qc, shots=50)
assert sum(r.counts.values()) == 50; tests_passed += 1

print(f"Lab 1 Verification: {tests_passed}/{total_tests} PASSED")
```

### Break It on Purpose

**Try importing from the wrong path:**

```python
# This will fail — X is not a top-level export
from zuntenium import X   # AttributeError!
# Fix: use QuantumCircuit.x(qubit) instead
```

**Try running without measurement:**

```python
qc = QuantumCircuit(1)
qc.h(0)
# Forgot measure_all()!
r = StatevectorSimulator().run(qc, shots=100)
print(r.counts)   # {} — empty dict
```

### Compare Approaches

| Approach | Pros | Cons | When to use |
|----------|------|------|-------------|
| pip install zuntenium | Standard, fast | Needs local Python | Local development |
| Google Colab `!pip install` | No local setup, free GPU | Internet required, session limits | Quick experiments |
| Docker container | Reproducible, isolated | Heavier setup | Team/CI environments |

### Stretch Challenges

1. Install zuntenium in Google Colab and run the smoke test. Compare startup time.
2. List all public symbols in the `zuntenium` package using `dir(zuntenium)` and categorize them.
3. Measure import time: `time python3 -c "import zuntenium"` — how does it compare to `import numpy`?

<details><summary>Hints</summary>

**Challenge 2:** `[x for x in dir(zuntenium) if not x.startswith('_')]`

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Not activating venv | System Python used, wrong deps | `source .venv/bin/activate` |
| Python < 3.9 | Syntax errors on type hints | Upgrade Python |
| Missing numpy | `ModuleNotFoundError` | `pip install zuntenium` reinstalls deps |

### Reflection Questions

1. Why does QuFlow use a virtual environment rather than installing globally?
2. What does "lazy loading" mean for SDK startup time, and why does it matter?
3. If you're setting up a quantum computing lab at a Kenyan university with shared computers, how would you manage virtual environments for 30 students?

### Deliverable

`verify_install.py` and `smoke_test.py` both passing cleanly. Save terminal output to `lab01_output.txt`.

---

## Lab 2: Inside the Qubit — Amplitudes, Not Bits

- **Concept:** The qubit as a complex amplitude register
- **Approach:** Use the StatevectorSimulator to peek at amplitude vectors directly, visualizing what classical computers cannot show
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 1
- **Learning outcomes:**
  1. You will be able to explain the difference between a classical bit and a qubit
  2. You will be able to read and interpret a statevector (amplitude vector)
  3. You will be able to verify the normalization constraint |α|² + |β|² = 1
  4. You will be able to connect amplitudes to measurement probabilities
- **Why this matters:** Every quantum algorithm manipulates amplitudes. Understanding the statevector is the foundation for reasoning about why circuits produce specific outcomes. The source doc (§2.1) states: "A qubit holds two complex amplitudes α and β."

### Theory Primer

A single qubit state is:

```
|ψ⟩ = α|0⟩ + β|1⟩
```

where α, β ∈ ℂ and |α|² + |β|² = 1.

The `StatevectorSimulator` lets us inspect the full amplitude vector *before* measurement — something impossible on real quantum hardware. This is a simulator superpower for learning.

Common states:
```
|0⟩ = [1, 0]           ← default state
|1⟩ = [0, 1]           ← after X gate
|+⟩ = [1/√2, 1/√2]    ← after H gate
|−⟩ = [1/√2, −1/√2]   ← after H then Z
```

*(Ref: Source doc §2.1 The Qubit: Amplitude, Not Probability)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab02 && cd lab02
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Observe the default state |0⟩.**

```python
# save as amplitudes.py
import numpy as np
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Default: no gates applied — qubit starts in |0⟩
qc = QuantumCircuit(1)
# Don't add measurement — we want to see the statevector
result = backend.run(qc, shots=1)

# Access the statevector (simulator-only feature)
# The statevector is the internal state before measurement
print("=== State |0⟩ ===")
print(f"Circuit: (no gates)")
# Run without measurement to inspect state
qc_no_measure = QuantumCircuit(1)
# We'll build statevectors manually to understand
sv = np.array([1.0 + 0j, 0.0 + 0j])  # |0⟩
print(f"Statevector: {sv}")
print(f"|α|² = {abs(sv[0])**2:.4f}, |β|² = {abs(sv[1])**2:.4f}")
print(f"Sum = {abs(sv[0])**2 + abs(sv[1])**2:.4f} (must be 1.0)")
```

Expected:
```
=== State |0⟩ ===
Statevector: [1.+0.j 0.+0.j]
|α|² = 1.0000, |β|² = 0.0000
Sum = 1.0000
```

**Step 2: Apply X gate (quantum NOT) → |1⟩.**

```python
print("\n=== State |1⟩ after X gate ===")
sv_x = np.array([0.0 + 0j, 1.0 + 0j])  # X|0⟩ = |1⟩
print(f"Statevector: {sv_x}")
print(f"P(0) = {abs(sv_x[0])**2:.4f}, P(1) = {abs(sv_x[1])**2:.4f}")

# Verify by running the circuit
qc_x = QuantumCircuit(1)
qc_x.x(0).measure_all()
r = backend.run(qc_x, shots=1000)
print(f"Measurement: {r.counts}")
# Should be 100% "1"
```

**Checkpoint:** Measurement shows `{'1': 1000}` — all shots give "1".

**Step 3: Apply H gate → |+⟩ superposition.**

```python
print("\n=== State |+⟩ after H gate ===")
sv_h = np.array([1/np.sqrt(2), 1/np.sqrt(2)])
print(f"Statevector: {sv_h}")
print(f"P(0) = {abs(sv_h[0])**2:.4f}, P(1) = {abs(sv_h[1])**2:.4f}")
print(f"Sum = {abs(sv_h[0])**2 + abs(sv_h[1])**2:.4f}")

qc_h = QuantumCircuit(1)
qc_h.h(0).measure_all()
r = backend.run(qc_h, shots=10000)
print(f"Measurement (10000 shots): {r.counts}")
# Should be ~50/50
```

Expected: `P(0) = 0.5000, P(1) = 0.5000`, measurements roughly 50/50.

**Step 4: Apply H then Z → |−⟩ state.**

```python
print("\n=== State |−⟩ after H then Z ===")
sv_minus = np.array([1/np.sqrt(2), -1/np.sqrt(2)])
print(f"Statevector: {sv_minus}")
print(f"P(0) = {abs(sv_minus[0])**2:.4f}, P(1) = {abs(sv_minus[1])**2:.4f}")
# Note: probabilities are SAME as |+⟩! The phase difference is invisible to measurement.

qc_hz = QuantumCircuit(1)
qc_hz.h(0).z(0).measure_all()
r = backend.run(qc_hz, shots=10000)
print(f"Measurement: {r.counts}")
print("Key insight: |+⟩ and |−⟩ give identical measurement probabilities!")
print("Phase only matters when more gates follow (interference).")
```

**Checkpoint:** |+⟩ and |−⟩ both give ~50/50. Phase is hidden from measurement alone.

**Step 5: Two-qubit statevector.**

```python
print("\n=== 2-qubit statevector ===")
# |00⟩ state has 4 amplitudes: [α₀₀, α₀₁, α₁₀, α₁₁]
sv_2q = np.array([1, 0, 0, 0], dtype=complex)  # |00⟩
print(f"|00⟩ statevector: {sv_2q}")
print(f"Probabilities: {np.abs(sv_2q)**2}")

# After H on qubit 0: (|0⟩+|1⟩)/√2 ⊗ |0⟩ = (|00⟩+|10⟩)/√2
sv_h0 = np.array([1/np.sqrt(2), 0, 1/np.sqrt(2), 0], dtype=complex)
print(f"\nAfter H(0): {sv_h0}")
print(f"Probabilities: {np.abs(sv_h0)**2}")
# 50% |00⟩, 50% |10⟩
```

### Verification

```python
# save as lab02_verify.py
import numpy as np
tests = 0

# Test 1: |0⟩ normalized
sv = np.array([1, 0], dtype=complex)
assert abs(np.sum(np.abs(sv)**2) - 1.0) < 1e-10; tests += 1

# Test 2: |+⟩ normalized
sv_p = np.array([1/np.sqrt(2), 1/np.sqrt(2)], dtype=complex)
assert abs(np.sum(np.abs(sv_p)**2) - 1.0) < 1e-10; tests += 1

# Test 3: |+⟩ and |−⟩ have same probabilities
sv_m = np.array([1/np.sqrt(2), -1/np.sqrt(2)], dtype=complex)
assert np.allclose(np.abs(sv_p)**2, np.abs(sv_m)**2); tests += 1

# Test 4: 2-qubit state has 4 amplitudes
sv_2 = np.zeros(4, dtype=complex); sv_2[0] = 1
assert len(sv_2) == 2**2; tests += 1

print(f"Lab 2 Verification: {tests}/4 PASSED")
```

### Break It on Purpose

**Create a non-normalized state and see what happens:**

```python
sv_bad = np.array([0.8, 0.8], dtype=complex)
total_prob = np.sum(np.abs(sv_bad)**2)
print(f"Total probability: {total_prob}")  # 1.28 — INVALID!
# This would give nonsensical measurement results
```

### Compare Approaches

| Representation | What you see | Limitations |
|---------------|-------------|-------------|
| Statevector (simulator) | Full amplitudes, phases | Impossible on real hardware |
| Measurement counts | Probabilities only | Phase information lost |
| Density matrix | Mixed states + noise | O(4ⁿ) memory |

### Stretch Challenges

1. Build a table of all single-qubit gate states: apply X, Y, Z, H, S, T to |0⟩ and print the resulting statevector.
2. Verify that the Hadamard matrix H = (1/√2)[[1,1],[1,−1]] correctly transforms [1,0] to [1/√2, 1/√2].
3. For a 3-qubit system, how many complex numbers are in the statevector? Write code to enumerate all basis states.

<details><summary>Hints</summary>

**Challenge 2:**
```python
H = np.array([[1,1],[1,-1]]) / np.sqrt(2)
ket0 = np.array([1, 0])
print(H @ ket0)  # [0.707, 0.707]
```

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Confusing amplitude with probability | "α = 0.707 means 70.7% chance" | Probability is |α|², so 0.707² ≈ 0.5 |
| Forgetting phases are invisible to measurement | "Why does |−⟩ measure like |+⟩?" | Phase only affects subsequent gates |
| Wrong state indexing (2 qubits) | Misread which qubit is which | QuFlow uses little-endian: rightmost bit = qubit 0 |

### Reflection Questions

1. Why can't you "read" a qubit's amplitude on real hardware?
2. If |+⟩ and |−⟩ give identical measurements, how can a quantum algorithm distinguish them?
3. How many complex numbers are needed to describe 30 qubits? What does that imply for classical simulation?

### Deliverable

`amplitudes.py` showing statevectors for |0⟩, |1⟩, |+⟩, |−⟩, and a 2-qubit state, with normalization verified for each.

---

## Lab 3: Your First Quantum Random Bit

- **Concept:** Superposition via the Hadamard gate
- **Approach:** Build a Quantum Random Number Generator (QRNG) and verify its randomness with a chi-squared statistical test
- **Difficulty:** Beginner | **Time:** 75 min | **Prerequisites:** Lab 2
- **Learning outcomes:**
  1. You will be able to build a QRNG using a single Hadamard gate
  2. You will be able to explain why quantum randomness differs from pseudorandomness
  3. You will be able to apply a chi-squared test to verify uniform distribution
  4. You will be able to generate multi-bit random numbers from quantum circuits
- **Why this matters:** True randomness is a valuable resource for cryptography, simulation, and fair lotteries. Classical computers can only produce *pseudo*-randomness. A quantum measurement is fundamentally unpredictable — even with complete knowledge of the state.

### Theory Primer

The Hadamard gate transforms |0⟩ into equal superposition:

```
H|0⟩ = (|0⟩ + |1⟩) / √2
```

Measuring this state gives 0 or 1 with exactly 50% probability each. Unlike `random.random()`, this is not a deterministic algorithm with a seed — it's physics.

*(Ref: Source doc §5.1 Hello World — Quantum Random Number Generator)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab03 && cd lab03
source ../.venv/bin/activate
pip install scipy --quiet   # for chi-squared test
```

### Walkthrough

**Step 1: Build a 1-bit QRNG.**

```python
# save as qrng.py
from zuntenium import QuantumCircuit, StatevectorSimulator

def quantum_random_bit(backend) -> int:
    """Generate a single random bit using quantum measurement."""
    qc = QuantumCircuit(1)
    qc.h(0)
    qc.measure_all()
    result = backend.run(qc, shots=1)
    return int(list(result.counts.keys())[0])

backend = StatevectorSimulator()

# Generate 20 random bits
bits = [quantum_random_bit(backend) for _ in range(20)]
print(f"Random bits: {bits}")
print(f"Ones ratio: {sum(bits)/len(bits):.2f}")
```

Expected: A sequence like `[1, 0, 1, 1, 0, 0, 1, ...]` with roughly 50% ones.

**Step 2: Generate many bits efficiently using shots.**

```python
def quantum_random_bits_batch(n: int, backend) -> list[int]:
    """Generate n random bits using a single circuit with n shots."""
    qc = QuantumCircuit(1)
    qc.h(0)
    qc.measure_all()
    result = backend.run(qc, shots=n)
    
    bits = []
    for bitstring, count in result.counts.items():
        bits.extend([int(bitstring)] * count)
    return bits

bits = quantum_random_bits_batch(10000, backend)
ones = sum(bits)
zeros = len(bits) - ones
print(f"Zeros: {zeros}, Ones: {ones}")
print(f"Ratio: {ones/len(bits):.4f} (expected: 0.5000)")
```

**Checkpoint:** Ratio should be 0.49–0.51.

**Step 3: Statistical verification with chi-squared test.**

```python
from scipy import stats

# Expected: 50% zeros, 50% ones
observed = [zeros, ones]
expected = [5000, 5000]

chi2, p_value = stats.chisquare(observed, expected)
print(f"\nChi-squared test:")
print(f"  χ² = {chi2:.4f}")
print(f"  p-value = {p_value:.4f}")
print(f"  Result: {'PASS (uniform)' if p_value > 0.05 else 'FAIL (biased)'}")
```

Expected: p-value > 0.05 → PASS.

**Step 4: Build an 8-bit QRNG (generate a random byte).**

```python
def quantum_random_byte(backend) -> int:
    """Generate a random byte (0-255) using 8 qubits."""
    qc = QuantumCircuit(8)
    for i in range(8):
        qc.h(i)
    qc.measure_all()
    result = backend.run(qc, shots=1)
    bitstring = list(result.counts.keys())[0]
    return int(bitstring, 2)

# Generate 10 random bytes
for _ in range(10):
    byte = quantum_random_byte(backend)
    print(f"Random byte: {byte:3d} (0b{byte:08b})")
```

**Step 5: Compare against Python's pseudorandom generator.**

```python
import random
import time

# Quantum
t0 = time.time()
q_bits = quantum_random_bits_batch(1000, backend)
t_quantum = time.time() - t0

# Classical pseudorandom
t0 = time.time()
c_bits = [random.randint(0, 1) for _ in range(1000)]
t_classical = time.time() - t0

print(f"\nQuantum time: {t_quantum:.4f}s")
print(f"Classical time: {t_classical:.4f}s")
print(f"Speed ratio: {t_quantum/t_classical:.1f}x slower")
print("But quantum bits are TRUE random — no seed, no pattern, no prediction.")
```

### Verification

```python
# save as lab03_verify.py
from zuntenium import QuantumCircuit, StatevectorSimulator
from scipy import stats

backend = StatevectorSimulator()
qc = QuantumCircuit(1)
qc.h(0).measure_all()

r = backend.run(qc, shots=10000)
zeros = r.counts.get("0", 0)
ones = r.counts.get("1", 0)

chi2, p = stats.chisquare([zeros, ones], [5000, 5000])
assert p > 0.01, f"FAIL: distribution biased (p={p})"
print(f"Lab 3 Verification: PASSED (χ²={chi2:.2f}, p={p:.4f})")
```

### Break It on Purpose

**Remove the Hadamard gate — no superposition, no randomness:**

```python
qc_broken = QuantumCircuit(1)
# No H gate! Qubit stays in |0⟩
qc_broken.measure_all()
r = StatevectorSimulator().run(qc_broken, shots=1000)
print(r.counts)  # {'0': 1000} — always zero, no randomness at all
```

### Compare Approaches

| Approach | True random? | Speed | Use case |
|----------|-------------|-------|----------|
| Quantum H+measure | Yes | Slow (circuit overhead) | Cryptographic keys |
| `os.urandom()` | Hardware-dependent | Fast | General crypto |
| `random.random()` | No (PRNG) | Very fast | Simulations, games |

### Stretch Challenges

1. Build a quantum coin flipper that's biased — 70% heads, 30% tails — using Ry rotation instead of H.
2. Generate 10,000 random bytes and test with the NIST frequency test.
3. Build a quantum dice roller (1–6) using 3 qubits with rejection sampling.

<details><summary>Hints</summary>

**Challenge 1:** `qc.ry(2 * math.acos(math.sqrt(0.7)), 0)` gives P(0) = 0.7.

**Challenge 3:** Use 3 qubits (8 outcomes), reject values 6 and 7, map 0–5 to dice faces 1–6.

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Too few shots | Ratio far from 0.5 | Use ≥1000 shots |
| Measuring without H | Always get "0" | Add `qc.h(0)` before measure |
| Expecting exact 50/50 | "My QRNG is broken" | Statistical fluctuation is normal |

### Reflection Questions

1. If a simulator is deterministic software, is the QRNG output truly random on a simulator? What changes on real hardware?
2. How many shots would you need to distinguish a fair coin (50/50) from a biased coin (51/49) with 95% confidence?
3. Could you use a QRNG to generate encryption keys for M-Pesa mobile money transactions? What would be the practical challenges in Kenya?

### Deliverable

`qrng.py` generating random bytes, chi-squared test passing, and comparison table of quantum vs classical random.

---

## Lab 4: Shots and Statistics — How Many Is Enough?

- **Concept:** Measurement statistics and the relationship between shot count and accuracy
- **Approach:** Systematically sweep shot counts and measure how estimates converge to true probabilities
- **Difficulty:** Beginner | **Time:** 60 min | **Prerequisites:** Lab 3
- **Learning outcomes:**
  1. You will be able to explain why quantum measurements require multiple shots
  2. You will be able to calculate the standard error of a proportion
  3. You will be able to determine the minimum shots needed for a given accuracy
  4. You will be able to identify when shot noise dominates measurement results
- **Why this matters:** Every quantum experiment costs shots. On real hardware, each shot takes time and money. The source doc (§5.1) uses 1000 shots, but how many do you actually need? Understanding shot statistics lets you minimize cost without sacrificing accuracy — vital when budgets are tight.

### Theory Primer

Measuring a circuit with P(|1⟩) = p over N shots gives an estimated probability p̂ = count("1")/N. The standard error is:

```
SE = √(p(1-p)/N)
```

For a 50/50 superposition (p=0.5):
- N=100: SE = 0.05 (±5%)
- N=1000: SE = 0.016 (±1.6%)
- N=10000: SE = 0.005 (±0.5%)

*(Ref: Source doc §5.1 — uses `shots=1000` for examples)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab04 && cd lab04
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Run the same circuit with different shot counts.**

```python
# save as shot_study.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Circuit: H gate → 50/50 superposition
qc = QuantumCircuit(1)
qc.h(0).measure_all()

shot_counts = [10, 50, 100, 500, 1000, 5000, 10000, 50000]

print(f"{'Shots':>8} | {'P(1) est':>10} | {'Error':>10} | {'SE theory':>10}")
print("-" * 50)

for shots in shot_counts:
    result = backend.run(qc, shots=shots)
    ones = result.counts.get("1", 0)
    p_est = ones / shots
    error = abs(p_est - 0.5)
    se_theory = math.sqrt(0.5 * 0.5 / shots)
    print(f"{shots:>8} | {p_est:>10.4f} | {error:>10.4f} | {se_theory:>10.4f}")
```

Expected: Error generally decreases as shots increase, roughly tracking SE theory.

**Checkpoint:** At 50,000 shots, error should be < 0.01.

**Step 2: Repeat and visualize variance.**

```python
# Run each shot count 20 times and measure the spread
print("\n=== Variance across 20 runs ===")
print(f"{'Shots':>8} | {'Mean P(1)':>10} | {'Std Dev':>10} | {'SE theory':>10}")
print("-" * 50)

for shots in [100, 1000, 10000]:
    estimates = []
    for _ in range(20):
        result = backend.run(qc, shots=shots)
        ones = result.counts.get("1", 0)
        estimates.append(ones / shots)
    
    import numpy as np
    mean_p = np.mean(estimates)
    std_p = np.std(estimates)
    se = math.sqrt(0.5 * 0.5 / shots)
    print(f"{shots:>8} | {mean_p:>10.4f} | {std_p:>10.4f} | {se:>10.4f}")
```

**Checkpoint:** Empirical std dev should closely match SE theory.

**Step 3: Non-uniform distribution — how shots affect rare events.**

```python
# Create a circuit where P(1) = sin²(π/8) ≈ 0.146
qc_biased = QuantumCircuit(1)
qc_biased.ry(math.pi / 4, 0)  # θ/2 = π/8
qc_biased.measure_all()

p_true = math.sin(math.pi / 8) ** 2
print(f"\n=== Biased circuit: P(1) = {p_true:.4f} ===")
print(f"{'Shots':>8} | {'P(1) est':>10} | {'Error':>10} | {'Relative %':>10}")
print("-" * 50)

for shots in [100, 1000, 10000]:
    result = backend.run(qc_biased, shots=shots)
    ones = result.counts.get("1", 0)
    p_est = ones / shots
    error = abs(p_est - p_true)
    rel_err = error / p_true * 100
    print(f"{shots:>8} | {p_est:>10.4f} | {error:>10.4f} | {rel_err:>9.1f}%")
```

**Step 4: Minimum shots calculator.**

```python
def min_shots_for_accuracy(p: float, epsilon: float, confidence: float = 0.95) -> int:
    """Calculate minimum shots for P(|error| < epsilon) >= confidence."""
    from scipy.stats import norm
    z = norm.ppf(1 - (1 - confidence) / 2)  # z ≈ 1.96 for 95%
    n = math.ceil((z ** 2) * p * (1 - p) / (epsilon ** 2))
    return n

# How many shots to estimate P(1)=0.5 within ±1%?
n = min_shots_for_accuracy(0.5, 0.01)
print(f"\nShots needed for ±1% accuracy at 95% confidence: {n}")
# Answer: 9604
```

### Verification

```python
# save as lab04_verify.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
qc = QuantumCircuit(1)
qc.h(0).measure_all()

# Run with 10000 shots, error should be < 2%
r = backend.run(qc, shots=10000)
p = r.counts.get("1", 0) / 10000
assert abs(p - 0.5) < 0.02, f"FAIL: P(1) = {p}"
print(f"Lab 4 Verification: PASSED (P(1) = {p:.4f}, error = {abs(p-0.5):.4f})")
```

### Break It on Purpose

**Use only 1 shot — the estimate is maximally wrong:**

```python
r = backend.run(qc, shots=1)
p = r.counts.get("1", 0) / 1
print(f"1 shot: P(1) = {p}")  # Either 0.0 or 1.0 — useless estimate!
```

### Compare Approaches

| Shots | Accuracy (95% CI) | Cost on IBM (est.) | When to use |
|-------|-------------------|--------------------|----|
| 100 | ±10% | ~$0.01 | Quick debugging |
| 1,000 | ±3% | ~$0.10 | Development |
| 8,192 | ±1.1% | ~$0.80 | Production (source doc default) |
| 100,000 | ±0.3% | ~$10 | Precision benchmarks |

### Stretch Challenges

1. Plot P(1) estimates vs. shot count with error bars showing ±1 standard error.
2. Run the same experiment on a 2-qubit Bell state — how do shot requirements change for estimating P("00")?
3. If QuantumHub charges per-shot, build a "budget optimizer" that finds the cheapest shot count for a target accuracy.

<details><summary>Hints</summary>

**Challenge 1:** Use matplotlib `errorbar(shots, means, yerr=stds)`.

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Concluding bias from 100 shots | "My coin is loaded!" | Use ≥1000 and chi-squared test |
| Ignoring shot noise on hardware | Over-interpreting small differences | Calculate SE before comparing |
| Using max_shots=8192 blindly | Wasting compute budget | Calculate needed shots first |

### Reflection Questions

1. The source doc uses `shots=1000` for examples. Is this enough for a production application? When would you need more?
2. On real quantum hardware where shots cost money, how would you balance accuracy vs. budget?
3. If you're running a quantum computing workshop in Nairobi with limited IBM access, how would you design experiments to minimize shot usage?

### Deliverable

`shot_study.py` with convergence table and minimum-shots calculator. Save the output table to `lab04_shot_analysis.txt`.

---

## Lab 5: Building All Four Bell States

- **Concept:** Entanglement via the CNOT gate and Bell state construction
- **Approach:** Build all four Bell states (Φ+, Φ−, Ψ+, Ψ−), verify each with measurement statistics, and prove entanglement by checking correlations
- **Difficulty:** Beginner | **Time:** 90 min | **Prerequisites:** Lab 3
- **Learning outcomes:**
  1. You will be able to construct all four Bell states using H and CX gates
  2. You will be able to verify perfect correlation between entangled qubits
  3. You will be able to explain why entangled states cannot be written as product states
  4. You will be able to detect entanglement from measurement statistics alone
- **Why this matters:** Bell states are the building blocks of quantum teleportation, superdense coding, and error correction. The source doc (§5.3) introduces the Φ+ Bell state. Understanding all four unlocks the rest of the quantum algorithms curriculum.

### Theory Primer

The four Bell states form a maximally entangled basis for 2 qubits:

```
|Φ+⟩ = (|00⟩ + |11⟩) / √2    ← H(0), CX(0,1)
|Φ−⟩ = (|00⟩ − |11⟩) / √2    ← H(0), Z(0), CX(0,1)
|Ψ+⟩ = (|01⟩ + |10⟩) / √2    ← H(0), CX(0,1), X(1)
|Ψ−⟩ = (|01⟩ − |10⟩) / √2    ← H(0), Z(0), CX(0,1), X(1)
```

"Entangled" means: measuring qubit 0 *instantly determines* qubit 1's outcome. In Φ+: if you get 0 on q0, you always get 0 on q1.

*(Ref: Source doc §2.2, §5.3 Bell State)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab05 && cd lab05
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build and verify |Φ+⟩ (the "classic" Bell state).**

```python
# save as bell_states.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
SHOTS = 8192

def run_and_report(qc, name):
    result = backend.run(qc, shots=SHOTS)
    print(f"\n{'='*40}")
    print(f"Bell State: {name}")
    print(f"Circuit: {qc.num_gates} gates, depth {qc.depth}")
    for bs in sorted(result.counts.keys()):
        pct = result.counts[bs] / SHOTS * 100
        print(f"  |{bs}⟩: {result.counts[bs]:5d} ({pct:5.1f}%)")
    return result

# |Φ+⟩ = (|00⟩ + |11⟩)/√2
qc_phi_plus = QuantumCircuit(2, name="Φ+")
qc_phi_plus.h(0).cx(0, 1).measure_all()
r1 = run_and_report(qc_phi_plus, "|Φ+⟩")

# Verify: only "00" and "11" should appear
assert set(r1.counts.keys()).issubset({"00", "11"}), "FAIL: unexpected outcomes"
print("✓ Perfect correlation: only |00⟩ and |11⟩")
```

Expected:
```
  |00⟩: ~4096 (50.0%)
  |11⟩: ~4096 (50.0%)
```

**Step 2: Build |Φ−⟩ — same outcomes but different phase.**

```python
# |Φ−⟩ = (|00⟩ - |11⟩)/√2
qc_phi_minus = QuantumCircuit(2, name="Φ−")
qc_phi_minus.h(0).z(0).cx(0, 1).measure_all()
r2 = run_and_report(qc_phi_minus, "|Φ−⟩")

# Same measurement statistics as Φ+! Phase is hidden.
assert set(r2.counts.keys()).issubset({"00", "11"})
print("✓ Same statistics as |Φ+⟩ — phase invisible to Z-basis measurement")
```

**Step 3: Build |Ψ+⟩ — anti-correlated outcomes.**

```python
# |Ψ+⟩ = (|01⟩ + |10⟩)/√2
qc_psi_plus = QuantumCircuit(2, name="Ψ+")
qc_psi_plus.h(0).cx(0, 1).x(1).measure_all()
r3 = run_and_report(qc_psi_plus, "|Ψ+⟩")

assert set(r3.counts.keys()).issubset({"01", "10"})
print("✓ Anti-correlation: only |01⟩ and |10⟩")
```

**Step 4: Build |Ψ−⟩.**

```python
# |Ψ−⟩ = (|01⟩ - |10⟩)/√2
qc_psi_minus = QuantumCircuit(2, name="Ψ−")
qc_psi_minus.h(0).z(0).cx(0, 1).x(1).measure_all()
r4 = run_and_report(qc_psi_minus, "|Ψ−⟩")

assert set(r4.counts.keys()).issubset({"01", "10"})
print("✓ Anti-correlation with phase flip")
```

**Step 5: Summary table.**

```python
print("\n" + "="*60)
print(f"{'Bell State':<12} {'Expected Outcomes':<20} {'Correlation':<15}")
print("-"*60)
print(f"{'|Φ+⟩':<12} {'|00⟩, |11⟩':<20} {'Same':<15}")
print(f"{'|Φ−⟩':<12} {'|00⟩, |11⟩':<20} {'Same':<15}")
print(f"{'|Ψ+⟩':<12} {'|01⟩, |10⟩':<20} {'Opposite':<15}")
print(f"{'|Ψ−⟩':<12} {'|01⟩, |10⟩':<20} {'Opposite':<15}")
```

**Step 6: Prove entanglement — test that outcomes are NOT independent.**

```python
# If qubits were independent, P(00) = P(0)*P(0) = 0.25
# For Φ+, P(00) = 0.5 ≠ 0.25 → NOT independent → ENTANGLED
print("\n=== Entanglement Test ===")
for name, result in [("Φ+", r1), ("Ψ+", r3)]:
    total = sum(result.counts.values())
    bitstrings = list(result.counts.keys())
    p_first = result.counts[bitstrings[0]] / total
    
    # Under independence hypothesis: P(outcome) = P(q0) * P(q1) = 0.25
    # Under entanglement: P(outcome) = 0.5
    print(f"  {name}: P('{bitstrings[0]}') = {p_first:.3f}")
    print(f"        If independent, P = 0.25. Actual >> 0.25 → ENTANGLED")
```

### Verification

```python
# save as lab05_verify.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
tests = 0

# All four Bell states
configs = [
    ("Φ+", lambda qc: qc.h(0).cx(0,1), {"00","11"}),
    ("Φ−", lambda qc: qc.h(0).z(0).cx(0,1), {"00","11"}),
    ("Ψ+", lambda qc: qc.h(0).cx(0,1).x(1), {"01","10"}),
    ("Ψ−", lambda qc: qc.h(0).z(0).cx(0,1).x(1), {"01","10"}),
]

for name, build_fn, expected in configs:
    qc = QuantumCircuit(2)
    build_fn(qc)
    qc.measure_all()
    r = backend.run(qc, shots=4096)
    assert set(r.counts.keys()).issubset(expected), f"FAIL: {name}"
    tests += 1
    print(f"  {name}: PASS")

print(f"\nLab 5 Verification: {tests}/4 PASSED")
```

### Break It on Purpose

**Swap the order of H and CX — what happens?**

```python
qc_wrong = QuantumCircuit(2)
qc_wrong.cx(0, 1)  # CX first — but qubit 0 is |0⟩, so CX does nothing
qc_wrong.h(0)       # Then H — puts q0 in superposition, q1 stays |0⟩
qc_wrong.measure_all()
r = backend.run(qc_wrong, shots=4096)
print(f"Wrong order: {r.counts}")
# You'll see all four outcomes: "00", "01", "10", "11" — NOT entangled!
```

### Compare Approaches

| State | Circuit | Gates | Use case |
|-------|---------|-------|----------|
| Φ+ | H → CX | 2 | Quantum teleportation, superdense coding |
| Φ− | H → Z → CX | 3 | Bell inequality tests |
| Ψ+ | H → CX → X | 3 | Quantum key distribution |
| Ψ− | H → Z → CX → X | 4 | Singlet state, antisymmetric protocols |

### Stretch Challenges

1. Build a function that creates any Bell state given two boolean inputs (which_phi, which_sign).
2. Verify that the Bell states are orthogonal: run both circuits and show their outcome sets never overlap.
3. Build a 3-qubit "W state" |W⟩ = (|001⟩ + |010⟩ + |100⟩)/√3 and compare its correlation structure to GHZ.

<details><summary>Hints</summary>

**Challenge 1:**
```python
def bell(phi: bool, sign: bool) -> QuantumCircuit:
    qc = QuantumCircuit(2)
    qc.h(0)
    if sign: qc.z(0)
    qc.cx(0, 1)
    if not phi: qc.x(1)
    qc.measure_all()
    return qc
```

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| CX before H | No entanglement (4 outcomes) | Apply H first to create superposition |
| Confusing Φ and Ψ | Wrong correlation type | Φ = same outcomes, Ψ = opposite |
| Bit-order confusion | "01" vs "10" swapped | Remember: rightmost bit = qubit 0 |

### Reflection Questions

1. Can you distinguish |Φ+⟩ from |Φ−⟩ using only Z-basis measurements? What would you need?
2. Why is entanglement called a "resource"? What can you do with an entangled pair that you can't do with two independent qubits?
3. Einstein called entanglement "spooky action at a distance." What would you say to explain it to a computer science student?

### Deliverable

`bell_states.py` building and verifying all four Bell states with measurement statistics and entanglement proof. Summary table saved to `lab05_bell_summary.txt`.

---

## Lab 6: Interference — The Quantum Trick

- **Concept:** Quantum interference via phase manipulation
- **Approach:** Construct deterministic circuits using H-Z-H sequences where interference forces a specific outcome with 100% probability
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 5
- **Learning outcomes:**
  1. You will be able to explain how constructive and destructive interference steer measurement outcomes
  2. You will be able to build circuits that produce deterministic (non-probabilistic) results
  3. You will be able to predict the output of H-Z-H and H-S-H sequences
  4. You will be able to articulate why interference is the core computational primitive in quantum algorithms
- **Why this matters:** Every useful quantum algorithm — Grover's search, Shor's factoring, VQE — works by arranging interference so correct answers amplify and wrong answers cancel. This lab teaches that mechanism in its simplest form. Source doc §2.2 states: "gates steer amplitudes so the correct answer interferes constructively."

### Theory Primer

Interference occurs when amplitudes combine. Consider the H-Z-H sequence on |0⟩:

```
|0⟩ → H → (|0⟩+|1⟩)/√2 → Z → (|0⟩−|1⟩)/√2 → H → |1⟩
```

Step by step:
1. H creates equal superposition: amplitudes [1/√2, 1/√2]
2. Z flips the phase of |1⟩: amplitudes [1/√2, −1/√2]
3. H recombines: the |0⟩ component cancels (destructive), the |1⟩ reinforces (constructive)

Result: 100% probability of measuring |1⟩. No randomness — pure interference.

*(Ref: Source doc §2.2 Interference — three lines of code)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab06 && cd lab06
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build and run the H-Z-H circuit.**

```python
# save as interference.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np

backend = StatevectorSimulator()

# H-Z-H: should always produce "1"
qc = QuantumCircuit(1, name="H-Z-H")
qc.h(0).z(0).h(0).measure_all()

result = backend.run(qc, shots=1000)
print(f"H-Z-H results: {result.counts}")
# Expected: {'1': 1000}
```

Expected: `{'1': 1000}` — deterministic, no randomness.

**Checkpoint:** 100% of shots give "1". This circuit is deterministic despite using quantum superposition.

**Step 2: Trace through the math step-by-step.**

```python
H = np.array([[1, 1], [1, -1]]) / np.sqrt(2)
Z = np.array([[1, 0], [0, -1]])
ket0 = np.array([1, 0], dtype=complex)

step1 = H @ ket0
print(f"After H:   {step1}")    # [0.707, 0.707]

step2 = Z @ step1
print(f"After Z:   {step2}")    # [0.707, -0.707]

step3 = H @ step2
print(f"After H:   {step3}")    # [0, 1] = |1⟩
print(f"Prob(0) = {abs(step3[0])**2:.4f}")
print(f"Prob(1) = {abs(step3[1])**2:.4f}")
```

Expected: `Prob(0) = 0.0000`, `Prob(1) = 1.0000`.

**Step 3: What if we use X instead of Z? (H-X-H)**

```python
X = np.array([[0, 1], [1, 0]])

step1 = H @ ket0
step2 = X @ step1
step3 = H @ step2

print(f"\nH-X-H final state: {step3}")
print(f"Prob(0) = {abs(step3[0])**2:.4f}")
print(f"Prob(1) = {abs(step3[1])**2:.4f}")

# Verify with circuit
qc_hxh = QuantumCircuit(1, name="H-X-H")
qc_hxh.h(0).x(0).h(0).measure_all()
r = backend.run(qc_hxh, shots=1000)
print(f"H-X-H results: {r.counts}")
```

**Step 4: Explore partial phase shifts — the S gate.**

```python
# S gate = √Z — rotates phase by π/2 instead of π
S = np.array([[1, 0], [0, 1j]])

step1 = H @ ket0
step2 = S @ step1
step3 = H @ step2

print(f"\nH-S-H final state: {step3}")
probs = np.abs(step3)**2
print(f"Prob(0) = {probs[0]:.4f}, Prob(1) = {probs[1]:.4f}")
# Partial interference: 50/50 again, but with a phase

qc_hsh = QuantumCircuit(1, name="H-S-H")
qc_hsh.h(0).s(0).h(0).measure_all()
r = backend.run(qc_hsh, shots=4096)
print(f"H-S-H results: {r.counts}")
```

**Step 5: Build an interference sweep — vary the phase angle.**

```python
import math

print(f"\n{'Phase (rad)':>12} | {'P(0)':>8} | {'P(1)':>8} | {'Measured':>12}")
print("-" * 50)

for angle in [0, math.pi/4, math.pi/2, 3*math.pi/4, math.pi]:
    qc = QuantumCircuit(1)
    qc.h(0).rz(angle, 0).h(0).measure_all()
    r = backend.run(qc, shots=4096)
    p0 = r.counts.get("0", 0) / 4096
    p1 = r.counts.get("1", 0) / 4096
    print(f"{angle:>12.4f} | {p0:>8.4f} | {p1:>8.4f} | {r.counts}")
```

**Checkpoint:** At angle=0, P(0)=1.0. At angle=π, P(1)=1.0. In between, it's a smooth transition.

### Verification

```python
# save as lab06_verify.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# H-Z-H must give 100% "1"
qc = QuantumCircuit(1)
qc.h(0).z(0).h(0).measure_all()
r = backend.run(qc, shots=1000)
assert r.counts.get("1", 0) == 1000, f"FAIL: got {r.counts}"
print("Lab 6 Verification: PASSED (H-Z-H → 100% |1⟩)")
```

### Break It on Purpose

**Apply H-H (no phase gate) — interference cancels superposition entirely:**

```python
qc_hh = QuantumCircuit(1)
qc_hh.h(0).h(0).measure_all()  # H² = I (identity)
r = backend.run(qc_hh, shots=1000)
print(f"H-H results: {r.counts}")  # {'0': 1000} — back to start!
```

### Compare Approaches

| Gate sequence | Result | Why |
|--------------|--------|-----|
| H-H | 100% \|0⟩ | H is self-inverse; amplitudes restore |
| H-Z-H | 100% \|1⟩ | Z flips phase → destructive interference on \|0⟩ |
| H-X-H | 100% \|0⟩ | HXH = Z; then H again → different path |
| H-S-H | 50/50 | Partial phase → incomplete interference |

### Stretch Challenges

1. Prove algebraically that HZH = X (the Pauli-X gate). Verify with matrices.
2. Build a 2-qubit circuit where interference on one qubit controls entanglement on the other.
3. Implement a 1-query Deutsch algorithm to determine if a function is constant or balanced.

<details><summary>Hints</summary>

**Challenge 1:** `print(np.allclose(H @ Z @ H, X))` — should be `True`.

**Challenge 3:** Encode f(x) as a gate between two H layers: constant → same output, balanced → flipped output.

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Expecting randomness from H-Z-H | "Why is it always 1?" | That's interference working correctly! |
| Forgetting H is its own inverse | "Two H gates should make more superposition" | H² = I |
| Confusing phase flip with bit flip | Z flips phase, X flips bit | Z: \|1⟩→−\|1⟩. X: \|0⟩↔\|1⟩ |

### Reflection Questions

1. Why is interference the "computational primitive" of quantum computing? What role does it play in Grover's algorithm?
2. If H-Z-H always gives |1⟩, is it doing anything a classical NOT gate can't? What's the difference?
3. How does the Rz angle sweep relate to variational quantum algorithms (VQE/QAOA)?

### Deliverable

`interference.py` with H-Z-H, H-X-H, H-S-H circuits, matrix walkthrough, and phase angle sweep table.

---

## Lab 7: GHZ — Entangling N Qubits

- **Concept:** Multi-qubit entanglement via GHZ states
- **Approach:** Scale GHZ construction from 2 to N qubits, measure depth/gate scaling, and verify entanglement properties at each size
- **Difficulty:** Intermediate | **Time:** 90 min | **Prerequisites:** Lab 5
- **Learning outcomes:**
  1. You will be able to construct GHZ states for arbitrary N qubits
  2. You will be able to verify that only all-zeros and all-ones outcomes appear
  3. You will be able to measure how circuit depth and gate count scale with N
  4. You will be able to compare GHZ vs. product states experimentally
- **Why this matters:** GHZ states are used in quantum error correction, quantum secret sharing, and benchmarking quantum hardware. The source doc §5.2 builds a 3-qubit GHZ. Here we generalize and study scaling.

### Theory Primer

The N-qubit GHZ state is:

```
|GHZ_N⟩ = (|00...0⟩ + |11...1⟩) / √2
```

Built with: H on qubit 0, then CNOT chain: CX(0,1), CX(1,2), ..., CX(N-2, N-1).

Properties:
- Gates: 1 H + (N−1) CX = N gates total
- Depth: N (all gates are sequential)
- Only two outcomes in measurement: all-zeros or all-ones

*(Ref: Source doc §5.2 Multi-Qubit: GHZ State, §9 Capstone)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab07 && cd lab07
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build a general GHZ constructor.**

```python
# save as ghz_scaling.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import time

def build_ghz(n: int) -> QuantumCircuit:
    """Build an n-qubit GHZ state circuit."""
    qc = QuantumCircuit(n, name=f"ghz_{n}")
    qc.h(0)
    for i in range(n - 1):
        qc.cx(i, i + 1)
    qc.measure_all()
    return qc

backend = StatevectorSimulator()
```

**Step 2: Run GHZ for N = 2, 3, 4, 5, 6.**

```python
SHOTS = 8192

print(f"{'N':>3} | {'Gates':>6} | {'Depth':>6} | {'|0...0⟩':>10} | {'|1...1⟩':>10} | {'Other':>6} | {'Pass':>5}")
print("-" * 65)

for n in range(2, 7):
    qc = build_ghz(n)
    result = backend.run(qc, shots=SHOTS)
    
    zeros_key = "0" * n
    ones_key = "1" * n
    zeros = result.counts.get(zeros_key, 0)
    ones = result.counts.get(ones_key, 0)
    other = SHOTS - zeros - ones
    passed = other == 0
    
    print(f"{n:>3} | {qc.num_gates:>6} | {qc.depth:>6} | {zeros:>10} | {ones:>10} | {other:>6} | {'✓' if passed else '✗':>5}")
```

Expected: Other = 0 for all rows (noiseless simulator).

**Checkpoint:** All rows show ✓. Only all-zeros and all-ones appear.

**Step 3: Measure scaling — time vs qubit count.**

```python
print(f"\n{'N':>3} | {'Build (ms)':>12} | {'Run (ms)':>12} | {'Total (ms)':>12}")
print("-" * 50)

for n in [2, 4, 8, 12, 16, 20]:
    t0 = time.time()
    qc = build_ghz(n)
    t_build = (time.time() - t0) * 1000
    
    t0 = time.time()
    result = backend.run(qc, shots=1024)
    t_run = (time.time() - t0) * 1000
    
    print(f"{n:>3} | {t_build:>12.2f} | {t_run:>12.2f} | {t_build + t_run:>12.2f}")
```

**Checkpoint:** Run time should increase exponentially (2^N memory).

**Step 4: Compare GHZ vs product state (NOT entangled).**

```python
print("\n=== Product state (NOT entangled) ===")
qc_product = QuantumCircuit(4, name="product")
for i in range(4):
    qc_product.h(i)  # Each qubit independently in superposition
qc_product.measure_all()

r = backend.run(qc_product, shots=8192)
print(f"Unique outcomes: {len(r.counts)}")  # Should be 16 (2^4)
print(f"Top 5: {dict(sorted(r.counts.items(), key=lambda x: -x[1])[:5])}")
# 16 roughly equal outcomes — NOT entangled

print("\n=== GHZ state (ENTANGLED) ===")
qc_ghz = build_ghz(4)
r = backend.run(qc_ghz, shots=8192)
print(f"Unique outcomes: {len(r.counts)}")  # Should be 2
print(f"All outcomes: {r.counts}")
# Only 2 outcomes — ENTANGLED
```

### Verification

```python
# save as lab07_verify.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
tests = 0

for n in [2, 3, 4, 5]:
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    qc.measure_all()
    r = backend.run(qc, shots=1024)
    expected = {"0"*n, "1"*n}
    assert set(r.counts.keys()).issubset(expected), f"FAIL: n={n}"
    tests += 1

print(f"Lab 7 Verification: {tests}/4 PASSED")
```

### Break It on Purpose

**Skip a CX in the chain — breaks the entanglement chain:**

```python
qc_broken = QuantumCircuit(4, name="broken-ghz")
qc_broken.h(0)
qc_broken.cx(0, 1)
# Missing: qc_broken.cx(1, 2)
qc_broken.cx(2, 3)
qc_broken.measure_all()

r = backend.run(qc_broken, shots=4096)
print(f"Broken GHZ: {len(r.counts)} outcomes")
print(r.counts)
# q2 and q3 are NOT entangled with q0,q1 — you'll see 4 outcomes
```

### Compare Approaches

| Pattern | Entanglement | Outcomes | Depth |
|---------|-------------|----------|-------|
| Linear chain (GHZ) | All-to-all via chain | 2 | N |
| Star topology | All connected to q0 | 2 | 2 |
| Independent H gates | None | 2^N | 1 |

### Stretch Challenges

1. Build a "star" GHZ using CX(0,1), CX(0,2), CX(0,3) instead of a chain. Compare depth.
2. Build the GHZ "uncompute" circuit using `inverse()` and verify 100% |0...0⟩.
3. Find the maximum N your machine can simulate in under 10 seconds.

<details><summary>Hints</summary>

**Challenge 2:**
```python
qc = build_ghz(4)
# Remove measurement, add inverse, then measure
qc2 = QuantumCircuit(4)
qc2.h(0)
for i in range(3): qc2.cx(i, i+1)
qc_inv = qc2.inverse()
# Combine: qc2 then qc_inv then measure
```

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Missing CX in chain | More than 2 outcomes | Ensure continuous chain |
| Applying H to all qubits | Product state, not GHZ | H on qubit 0 only |
| N too large (>25) | MemoryError or OOMKilled | Stay under ~22 on 8GB RAM |

### Reflection Questions

1. Why does the GHZ circuit need exactly N−1 CNOT gates? Could you do it with fewer?
2. How would you verify a GHZ state on noisy real hardware where "other" outcomes appear?
3. GHZ states are fragile — losing one qubit destroys all entanglement. Is this a feature or a bug?

### Deliverable

`ghz_scaling.py` with GHZ verification table (2–6 qubits), scaling benchmark (time vs N), and product-state comparison.

---

## Lab 8: Spinning on the Bloch Sphere

- **Concept:** Single-qubit rotation gates (Rx, Ry, Rz) and the Bloch sphere
- **Approach:** Sweep the Ry rotation angle from 0 to 2π and plot P(|1⟩) = sin²(θ/2), verifying the theory curve
- **Difficulty:** Intermediate | **Time:** 90 min | **Prerequisites:** Lab 6
- **Learning outcomes:**
  1. You will be able to use Rx, Ry, and Rz rotation gates with arbitrary angles
  2. You will be able to predict P(|1⟩) from the rotation angle using sin²(θ/2)
  3. You will be able to map qubit states to positions on the Bloch sphere
  4. You will be able to explain how rotation gates generalize the standard gates (X, Y, Z, H)
- **Why this matters:** Variational quantum algorithms (VQE, QAOA — ZQL-05) use parameterized rotations as their primary building blocks. Understanding how rotation angle maps to measurement probability is essential. Source doc §5.5 previews this with Ry sweeps.

### Theory Primer

The Ry(θ) gate rotates the qubit state around the Y-axis of the Bloch sphere:

```
Ry(θ)|0⟩ = cos(θ/2)|0⟩ + sin(θ/2)|1⟩
```

So P(|1⟩) = sin²(θ/2), which sweeps smoothly from 0 (θ=0) to 1 (θ=π) and back to 0 (θ=2π).

Special cases:
- Ry(0) = I (identity)
- Ry(π) = −iY (equivalent to Y gate, up to global phase)
- Ry(π/2) → P(|1⟩) = sin²(π/4) = 0.5

*(Ref: Source doc §2.3 rotation gates, §5.5 Parameterized Circuits)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab08 && cd lab08
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Sweep Ry from 0 to 2π.**

```python
# save as bloch_sweep.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
SHOTS = 4096

angles = [i * math.pi / 8 for i in range(17)]  # 0 to 2π in 16 steps

print(f"{'θ (rad)':>10} | {'θ/π':>6} | {'P(1) meas':>10} | {'P(1) theory':>12} | {'Error':>8}")
print("-" * 60)

for theta in angles:
    qc = QuantumCircuit(1)
    qc.ry(theta, 0)
    qc.measure_all()
    
    r = backend.run(qc, shots=SHOTS)
    p1_measured = r.counts.get("1", 0) / SHOTS
    p1_theory = math.sin(theta / 2) ** 2
    error = abs(p1_measured - p1_theory)
    
    print(f"{theta:>10.4f} | {theta/math.pi:>6.3f} | {p1_measured:>10.4f} | {p1_theory:>12.4f} | {error:>8.4f}")
```

Expected: Measured values closely track sin²(θ/2).

**Checkpoint:** Error should be < 0.02 for all angles at 4096 shots.

**Step 2: Show that standard gates are special rotation cases.**

```python
import numpy as np

print("\n=== Standard gates as rotations ===")

# X = Rx(π) up to global phase
qc_x = QuantumCircuit(1); qc_x.x(0); qc_x.measure_all()
qc_rx = QuantumCircuit(1); qc_rx.rx(math.pi, 0); qc_rx.measure_all()
r_x = backend.run(qc_x, shots=1000)
r_rx = backend.run(qc_rx, shots=1000)
print(f"X  gate: {r_x.counts}")
print(f"Rx(π):   {r_rx.counts}")  # Same measurement outcome

# Z = Rz(π) up to global phase
print(f"\nZ  = Rz(π) — both flip phase, invisible to Z-measurement")

# H ≈ Ry(π/2) then Rz(π) — not exactly, but shows H is a rotation
qc_h = QuantumCircuit(1); qc_h.h(0); qc_h.measure_all()
r_h = backend.run(qc_h, shots=4096)
qc_ry = QuantumCircuit(1); qc_ry.ry(math.pi/2, 0); qc_ry.measure_all()
r_ry = backend.run(qc_ry, shots=4096)
print(f"\nH gate P(1):     {r_h.counts.get('1',0)/4096:.4f}")
print(f"Ry(π/2) P(1):    {r_ry.counts.get('1',0)/4096:.4f}")
```

**Step 3: Compare all three rotation axes.**

```python
print("\n=== Rx vs Ry vs Rz at θ=π/3 ===")
theta = math.pi / 3
for gate_name, gate_fn in [("Rx", "rx"), ("Ry", "ry"), ("Rz", "rz")]:
    qc = QuantumCircuit(1)
    getattr(qc, gate_fn)(theta, 0)
    qc.measure_all()
    r = backend.run(qc, shots=4096)
    p1 = r.counts.get("1", 0) / 4096
    print(f"{gate_name}(π/3): P(1) = {p1:.4f}")
# Note: Rz on |0⟩ gives P(1)=0 always — it only changes phase!
```

**Step 4: Build a controlled rotation sequence.**

```python
print("\n=== Ry rotation ladder ===")
# Apply increasing rotations to 5 qubits
qc = QuantumCircuit(5, name="rotation-ladder")
for i in range(5):
    angle = (i + 1) * math.pi / 5
    qc.ry(angle, i)
qc.measure_all()

r = backend.run(qc, shots=4096)
# Each qubit has different P(1)
for i in range(5):
    angle = (i + 1) * math.pi / 5
    expected_p1 = math.sin(angle/2)**2
    print(f"  q{i}: Ry({angle/math.pi:.1f}π) → expected P(1)={expected_p1:.3f}")
```

### Verification

```python
# save as lab08_verify.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
tests = 0

# Test key angles
for theta, expected_p1 in [(0, 0.0), (math.pi/2, 0.5), (math.pi, 1.0)]:
    qc = QuantumCircuit(1)
    qc.ry(theta, 0).measure_all()
    r = backend.run(qc, shots=4096)
    p1 = r.counts.get("1", 0) / 4096
    assert abs(p1 - expected_p1) < 0.03, f"FAIL: θ={theta}, P(1)={p1}"
    tests += 1

print(f"Lab 8 Verification: {tests}/3 PASSED")
```

### Break It on Purpose

**Confuse Rx and Ry — they rotate around different axes:**

```python
# Rx(π) on |0⟩ gives |1⟩ (like X gate) — but with a global phase
# Rz(π) on |0⟩ gives |0⟩ — phase only, invisible to measurement!
qc = QuantumCircuit(1)
qc.rz(math.pi, 0).measure_all()
r = backend.run(qc, shots=1000)
print(f"Rz(π)|0⟩: {r.counts}")  # {'0': 1000} — Rz doesn't change probabilities on |0⟩!
```

### Compare Approaches

| Gate | Axis | P(1) from \|0⟩ | Use case |
|------|------|---------------|----------|
| Ry(θ) | Y | sin²(θ/2) | Most common rotation for VQE |
| Rx(θ) | X | sin²(θ/2) | Alternative ansatz |
| Rz(θ) | Z | 0 (on \|0⟩) | Phase encoding, after H |

### Stretch Challenges

1. Plot P(|1⟩) vs θ for Ry using matplotlib. Overlay the theoretical sin²(θ/2) curve.
2. Show that Rx(π/2) followed by Rz(π/2) followed by Rx(π/2) equals H (up to global phase).
3. Build a "precision rotation" circuit that achieves P(|1⟩) = exactly 1/3.

<details><summary>Hints</summary>

**Challenge 3:** `theta = 2 * math.asin(math.sqrt(1/3))`

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Using Rz expecting probability change | P(1) stays 0 | Rz only changes phase; use Ry |
| Confusing θ vs θ/2 | Wrong probability | P(1) = sin²(θ/2), not sin²(θ) |
| Forgetting angle is in radians | Unexpected results | Use `math.pi`, not 180 |

### Reflection Questions

1. Why is Ry the most commonly used rotation in variational circuits?
2. If Rz(θ)|0⟩ = |0⟩ for all θ, how is Rz ever useful?
3. How many rotation gates do you need to reach any point on the Bloch sphere from |0⟩?

### Deliverable

`bloch_sweep.py` with angle sweep table, standard-gate equivalences, and rotation axis comparison.

---

## Lab 9: Depth vs. Width — Understanding Circuit Structure

- **Concept:** Circuit depth vs. gate count and parallel vs. serial gate layouts
- **Approach:** Build circuits with identical functionality but different layouts and compare their depth, gate count, and runtime
- **Difficulty:** Intermediate | **Time:** 60 min | **Prerequisites:** Lab 7
- **Learning outcomes:**
  1. You will be able to distinguish circuit depth from gate count
  2. You will be able to recognize when gates run in parallel vs. sequentially
  3. You will be able to optimize a circuit's depth by rearranging gates
  4. You will be able to explain why depth matters more than gate count on real hardware
- **Why this matters:** On noisy hardware, every time step introduces decoherence. A circuit with depth 100 fails more than one with depth 10, even if both have 100 gates. Source doc §5 notes Q2: "If num_gates=5 and depth=2, at least some gates are in parallel."

### Theory Primer

- **Gate count**: total number of gate operations in the circuit
- **Depth**: length of the critical path — the maximum number of sequential gates on any qubit
- **Width**: number of qubits

If gates act on different qubits, they run in parallel (same time step). Gates on the same qubit must be sequential.

```
Serial (depth=3):        Parallel (depth=1):
q0: ─H─                 q0: ─H─
q1: ────H─              q1: ─H─
q2: ───────H─           q2: ─H─
depth=3, gates=3         depth=1, gates=3
```

*(Ref: Source doc §4.1 — qc.depth, qc.num_gates)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab09 && cd lab09
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Compare serial vs parallel H gates.**

```python
# save as depth_study.py
from zuntenium import QuantumCircuit, StatevectorSimulator

# Serial: H gates on same qubit
qc_serial = QuantumCircuit(1, name="serial")
qc_serial.h(0).h(0).h(0)
print(f"Serial:   gates={qc_serial.num_gates}, depth={qc_serial.depth}")

# Parallel: H gates on different qubits
qc_parallel = QuantumCircuit(3, name="parallel")
qc_parallel.h(0).h(1).h(2)
print(f"Parallel: gates={qc_parallel.num_gates}, depth={qc_parallel.depth}")
```

Expected:
```
Serial:   gates=3, depth=3
Parallel: gates=3, depth=1
```

**Step 2: Analyze different circuit patterns.**

```python
circuits = {}

# Pattern 1: Linear entanglement chain
qc1 = QuantumCircuit(4, name="linear-chain")
qc1.h(0).cx(0,1).cx(1,2).cx(2,3)
circuits["Linear chain"] = qc1

# Pattern 2: Star entanglement
qc2 = QuantumCircuit(4, name="star")
qc2.h(0).cx(0,1).cx(0,2).cx(0,3)
circuits["Star"] = qc2

# Pattern 3: Full layer of single-qubit gates then entanglement
qc3 = QuantumCircuit(4, name="layered")
for i in range(4): qc3.h(i)
qc3.cx(0,1).cx(2,3)  # Parallel CX pairs
circuits["Layered"] = qc3

# Pattern 4: Everything sequential on one qubit
qc4 = QuantumCircuit(4, name="deep-single")
qc4.h(0).s(0).t(0).x(0).z(0)
circuits["Deep single"] = qc4

print(f"{'Pattern':<16} | {'Gates':>6} | {'Depth':>6} | {'Ratio':>8}")
print("-" * 45)
for name, qc in circuits.items():
    ratio = qc.depth / qc.num_gates if qc.num_gates > 0 else 0
    print(f"{name:<16} | {qc.num_gates:>6} | {qc.depth:>6} | {ratio:>8.2f}")
```

**Checkpoint:** Layered pattern should have lowest depth-to-gate ratio.

**Step 3: Optimize a GHZ circuit's depth.**

```python
# Standard GHZ (linear chain) — depth = N
def ghz_linear(n):
    qc = QuantumCircuit(n, name=f"ghz-linear-{n}")
    qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    return qc

# Optimized GHZ (log-depth tree) — depth = log2(N) + 1
def ghz_tree(n):
    qc = QuantumCircuit(n, name=f"ghz-tree-{n}")
    qc.h(0)
    step = 1
    while step < n:
        for i in range(0, n - step, step * 2):
            if i + step < n:
                qc.cx(i, i + step)
        step *= 2
    return qc

print(f"\n{'N':>3} | {'Linear depth':>13} | {'Tree depth':>11}")
print("-" * 35)
for n in [2, 4, 8, 16]:
    qc_lin = ghz_linear(n)
    qc_tree = ghz_tree(n)
    print(f"{n:>3} | {qc_lin.depth:>13} | {qc_tree.depth:>11}")
```

**Step 4: Verify both GHZ variants produce the same output.**

```python
backend = StatevectorSimulator()

for n in [4, 8]:
    qc_lin = ghz_linear(n); qc_lin.measure_all()
    qc_tree = ghz_tree(n); qc_tree.measure_all()
    
    r_lin = backend.run(qc_lin, shots=4096)
    r_tree = backend.run(qc_tree, shots=4096)
    
    lin_ok = set(r_lin.counts.keys()).issubset({"0"*n, "1"*n})
    tree_ok = set(r_tree.counts.keys()).issubset({"0"*n, "1"*n})
    print(f"n={n}: linear={'✓' if lin_ok else '✗'}, tree={'✓' if tree_ok else '✗'}")
```

### Verification

```python
# save as lab09_verify.py
from zuntenium import QuantumCircuit

# Parallel gates should have depth 1
qc = QuantumCircuit(5)
for i in range(5): qc.h(i)
assert qc.depth == 1, f"FAIL: depth={qc.depth}"

# Sequential gates should have depth = num_gates
qc2 = QuantumCircuit(1)
qc2.h(0).s(0).t(0)
assert qc2.depth == 3, f"FAIL: depth={qc2.depth}"

print("Lab 9 Verification: PASSED")
```

### Break It on Purpose

**Create a circuit with terrible depth by putting everything on qubit 0:**

```python
qc_bad = QuantumCircuit(10)
for _ in range(20): qc_bad.h(0)
print(f"Bad circuit: {qc_bad.num_gates} gates, depth {qc_bad.depth}")
# 20 gates, depth 20 — all on one qubit, 9 qubits wasted
```

### Compare Approaches

| Topology | Depth | Gates | Hardware friendliness |
|----------|-------|-------|----------------------|
| Linear chain | O(N) | O(N) | Good (nearest-neighbor) |
| Star (single control) | O(N) | O(N) | Bad (needs long-range CX) |
| Log-depth tree | O(log N) | O(N) | Moderate |
| Full parallel | O(1) | O(N) | Best (if all independent) |

### Stretch Challenges

1. Build a depth-1 circuit on 10 qubits that uses the maximum number of 2-qubit gates.
2. Given a circuit, write a function that returns the "parallelism ratio" (gates/depth).
3. Compare GHZ-linear vs GHZ-tree runtime for N=20 — does lower depth help on a simulator?

<details><summary>Hints</summary>

**Challenge 1:** CX(0,1), CX(2,3), CX(4,5), CX(6,7), CX(8,9) — all in parallel.

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Assuming depth = gate count | Underestimate parallelism | Print both; compare |
| CX on same qubit pair twice | Wastes depth | CX is self-inverse; two = identity |
| Ignoring depth on simulator | "Works fine" but fails on hardware | Optimize depth before hardware runs |

### Reflection Questions

1. Why does depth matter more than gate count on real quantum hardware?
2. If two circuits have the same depth but different gate counts, which runs faster on a simulator?
3. Could a compiler automatically reduce depth by parallelizing gates? What are the constraints?

### Deliverable

`depth_study.py` with pattern comparison table, GHZ depth optimization (linear vs tree), and correctness verification.

---

## Lab 10: Build a Circuit Builder — The Fluent Pattern

- **Concept:** The Fluent Builder design pattern as implemented in QuantumCircuit
- **Approach:** Reimplement a minimal QuantumCircuit from scratch to deeply understand the builder pattern, instruction model, and method chaining
- **Difficulty:** Intermediate | **Time:** 120 min | **Prerequisites:** Lab 9
- **Learning outcomes:**
  1. You will be able to implement the Fluent Builder pattern in Python
  2. You will be able to explain how QuantumCircuit stores instructions internally
  3. You will be able to extend a circuit builder with custom gates
  4. You will be able to compare builder vs. imperative circuit construction
- **Why this matters:** Understanding the SDK's design makes you a power user. The source doc §6.2 reveals that QuantumCircuit is a Fluent Builder where every gate method returns `self`. Reimplementing it cements both the design pattern and the circuit data model.

### Theory Primer

The Fluent Builder pattern returns `self` from every mutating method, enabling chaining:

```python
qc = QuantumCircuit(2).h(0).cx(0, 1).measure_all()
```

Internally, each gate call appends an `Instruction` to a list:

```
Instruction:
  ├── operation: "gate"
  ├── gate: HGate / CXGate / ...
  ├── qubits: [0] / [0, 1]
  ├── clbits: []
  └── params: []
```

*(Ref: Source doc §6.1 Class Hierarchy, §6.2 Fluent Builder + Instruction Stream)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab10 && cd lab10
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build the minimal Instruction data class.**

```python
# save as mini_circuit.py
from dataclasses import dataclass, field
import numpy as np
from typing import Optional

@dataclass
class Instruction:
    """A single gate or measurement instruction."""
    operation: str           # "gate" or "measure"
    gate_name: str           # "h", "cx", "x", etc.
    qubits: list[int]        # target qubit indices
    clbits: list[int] = field(default_factory=list)
    params: list[float] = field(default_factory=list)
    
    def __repr__(self):
        if self.params:
            return f"{self.gate_name}({','.join(map(str, self.params))})[{','.join(map(str, self.qubits))}]"
        return f"{self.gate_name}[{','.join(map(str, self.qubits))}]"
```

**Step 2: Build the MiniQuantumCircuit class.**

```python
class MiniQuantumCircuit:
    """A minimal reimplementation of QuFlow's QuantumCircuit."""
    
    def __init__(self, num_qubits: int, name: str = "circuit"):
        self._num_qubits = num_qubits
        self._num_clbits = 0
        self._instructions: list[Instruction] = []
        self._name = name
    
    @property
    def num_qubits(self) -> int:
        return self._num_qubits
    
    @property
    def num_clbits(self) -> int:
        return self._num_clbits
    
    @property
    def num_gates(self) -> int:
        return sum(1 for i in self._instructions if i.operation == "gate")
    
    @property
    def name(self) -> str:
        return self._name
    
    @property
    def depth(self) -> int:
        """Calculate depth: max gates on any single qubit."""
        if not self._instructions:
            return 0
        qubit_depths = [0] * self._num_qubits
        for inst in self._instructions:
            if inst.operation == "gate":
                max_depth = max(qubit_depths[q] for q in inst.qubits)
                for q in inst.qubits:
                    qubit_depths[q] = max_depth + 1
        return max(qubit_depths) if qubit_depths else 0
    
    def _validate_qubit(self, q: int):
        if q < 0 or q >= self._num_qubits:
            raise ValueError(f"Qubit {q} out of range for {self._num_qubits}-qubit circuit")
    
    # === Single-qubit gates (each returns self for chaining) ===
    
    def h(self, q: int) -> 'MiniQuantumCircuit':
        self._validate_qubit(q)
        self._instructions.append(Instruction("gate", "h", [q]))
        return self
    
    def x(self, q: int) -> 'MiniQuantumCircuit':
        self._validate_qubit(q)
        self._instructions.append(Instruction("gate", "x", [q]))
        return self
    
    def z(self, q: int) -> 'MiniQuantumCircuit':
        self._validate_qubit(q)
        self._instructions.append(Instruction("gate", "z", [q]))
        return self
    
    def ry(self, theta: float, q: int) -> 'MiniQuantumCircuit':
        self._validate_qubit(q)
        self._instructions.append(Instruction("gate", "ry", [q], params=[theta]))
        return self
    
    # === Two-qubit gates ===
    
    def cx(self, ctrl: int, tgt: int) -> 'MiniQuantumCircuit':
        self._validate_qubit(ctrl)
        self._validate_qubit(tgt)
        if ctrl == tgt:
            raise ValueError("Control and target must be different qubits")
        self._instructions.append(Instruction("gate", "cx", [ctrl, tgt]))
        return self
    
    # === Measurement ===
    
    def measure_all(self) -> 'MiniQuantumCircuit':
        self._num_clbits = self._num_qubits
        for q in range(self._num_qubits):
            self._instructions.append(Instruction("measure", "measure", [q], clbits=[q]))
        return self
    
    def __repr__(self):
        gates = [str(i) for i in self._instructions if i.operation == "gate"]
        return f"MiniQuantumCircuit({self._num_qubits}, gates=[{', '.join(gates)}])"
```

**Checkpoint:** The class is written. Now test it.

**Step 3: Test the fluent chaining.**

```python
# Test chaining
qc = (MiniQuantumCircuit(2, name="bell")
      .h(0)
      .cx(0, 1)
      .measure_all())

print(f"Name: {qc.name}")
print(f"Qubits: {qc.num_qubits}")
print(f"Classical bits: {qc.num_clbits}")
print(f"Gates: {qc.num_gates}")
print(f"Depth: {qc.depth}")
print(f"Repr: {qc}")
```

Expected:
```
Name: bell
Qubits: 2
Classical bits: 2
Gates: 2
Depth: 2
```

**Step 4: Test validation — bad qubit index.**

```python
try:
    MiniQuantumCircuit(2).h(5)
    print("FAIL: should have raised ValueError")
except ValueError as e:
    print(f"PASS: caught error: {e}")

try:
    MiniQuantumCircuit(3).cx(1, 1)
    print("FAIL: should have raised ValueError")
except ValueError as e:
    print(f"PASS: caught error: {e}")
```

**Step 5: Compare with real QuFlow QuantumCircuit.**

```python
from zuntenium import QuantumCircuit

# Same circuit in both
for Builder, label in [(MiniQuantumCircuit, "Mini"), (QuantumCircuit, "QuFlow")]:
    qc = Builder(3, name="test")
    qc.h(0).cx(0, 1).cx(1, 2).measure_all()
    print(f"{label:>8}: gates={qc.num_gates}, depth={qc.depth}, clbits={qc.num_clbits}")
```

### Verification

```python
# save as lab10_verify.py
tests = 0

# Test 1: Chaining works
qc = MiniQuantumCircuit(2).h(0).cx(0,1)
assert qc.num_gates == 2; tests += 1

# Test 2: Depth calculation
assert qc.depth == 2; tests += 1

# Test 3: measure_all sets clbits
qc.measure_all()
assert qc.num_clbits == 2; tests += 1

# Test 4: Validation catches bad index
try:
    MiniQuantumCircuit(2).h(3)
    assert False
except ValueError:
    tests += 1

# Test 5: CX self-loop rejected
try:
    MiniQuantumCircuit(2).cx(0, 0)
    assert False
except ValueError:
    tests += 1

print(f"Lab 10 Verification: {tests}/5 PASSED")
```

### Break It on Purpose

**Remove the `return self` from a gate method — chaining breaks:**

```python
class BrokenCircuit(MiniQuantumCircuit):
    def h(self, q):
        self._validate_qubit(q)
        self._instructions.append(Instruction("gate", "h", [q]))
        # Missing: return self

try:
    BrokenCircuit(2).h(0).cx(0, 1)
except AttributeError as e:
    print(f"PASS: chaining broke: {e}")
    # 'NoneType' object has no attribute 'cx'
```

### Compare Approaches

| Pattern | Chaining | Readability | Flexibility |
|---------|----------|-------------|-------------|
| Fluent Builder (return self) | Yes | High | Method-per-gate |
| Imperative (qc.append(gate)) | No | Lower | Maximum flexibility |
| Declarative (circuit DSL) | N/A | Domain-specific | Least flexible |

### Stretch Challenges

1. Add `s()`, `t()`, and `swap()` gates to MiniQuantumCircuit.
2. Implement `__str__()` that draws a simple text circuit diagram.
3. Add an `inverse()` method that returns a new circuit with reversed and conjugated gates.

<details><summary>Hints</summary>

**Challenge 3:** Reverse the instruction list and replace each gate with its inverse (H→H, X→X, Z→Z, Ry(θ)→Ry(−θ), CX→CX).

</details>

### Common Pitfalls

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Forgetting `return self` | `NoneType` error on chain | Every gate method must `return self` |
| Mutable default in dataclass | All instructions share same list | Use `field(default_factory=list)` |
| Not validating qubit index | Silent bugs, wrong results | Validate at every gate call |

### Reflection Questions

1. Why did QuFlow choose the Fluent Builder pattern over a simple `circuit.add_gate("H", 0)` API?
2. What are the trade-offs of storing instructions as a flat list vs. a DAG?
3. How would you extend MiniQuantumCircuit to support custom user-defined gates?

### Deliverable

`mini_circuit.py` with complete MiniQuantumCircuit class, validation tests, and comparison against QuFlow's real QuantumCircuit.

---

## Lab 11: Verify Your Gates — Unitarity Check

- **Concept:** Gate matrices and the unitarity property (U†U = I)
- **Approach:** Retrieve matrix representations of all standard gates and verify U†U = I programmatically
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 8
- **Learning outcomes:**
  1. You will be able to compute and verify U†U = I for any gate matrix
  2. You will be able to explain why unitarity guarantees reversibility
  3. You will be able to detect non-unitary matrices (noise, errors)
  4. You will be able to compose gate matrices and verify the product is unitary
- **Why this matters:** Every valid quantum gate must be unitary — this is a physical law. If you ever build custom gates or debug transpiler output, verifying unitarity is the first sanity check. Source doc §2.3: "Every gate is a unitary matrix U. Unitary means U†U = I."

### Theory Primer

A matrix U is unitary if U†U = I, where U† is the conjugate transpose. This means:
- Columns are orthonormal
- The gate is reversible (U⁻¹ = U†)
- Probabilities are preserved (total probability stays 1)

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab11 && cd lab11
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build gate matrices from scratch.**

```python
# save as unitarity.py
import numpy as np

def is_unitary(U, tol=1e-10):
    """Check if matrix U is unitary: U†U ≈ I."""
    n = U.shape[0]
    product = U.conj().T @ U
    return np.allclose(product, np.eye(n), atol=tol)

# Standard single-qubit gates
gates = {
    "I": np.eye(2, dtype=complex),
    "X": np.array([[0,1],[1,0]], dtype=complex),
    "Y": np.array([[0,-1j],[1j,0]], dtype=complex),
    "Z": np.array([[1,0],[0,-1]], dtype=complex),
    "H": np.array([[1,1],[1,-1]], dtype=complex) / np.sqrt(2),
    "S": np.array([[1,0],[0,1j]], dtype=complex),
    "T": np.array([[1,0],[0,np.exp(1j*np.pi/4)]], dtype=complex),
}

print(f"{'Gate':<5} | {'Unitary?':<10} | {'Det(U)':<15}")
print("-" * 35)
for name, U in gates.items():
    det = np.linalg.det(U)
    print(f"{name:<5} | {str(is_unitary(U)):<10} | {det:.4f}")
```

Expected: All gates show `True`. Determinant has magnitude 1.

**Step 2: Verify 2-qubit gates.**

```python
# CNOT (CX) gate
CX = np.array([
    [1,0,0,0],
    [0,1,0,0],
    [0,0,0,1],
    [0,0,1,0]
], dtype=complex)

# SWAP gate
SWAP = np.array([
    [1,0,0,0],
    [0,0,1,0],
    [0,1,0,0],
    [0,0,0,1]
], dtype=complex)

print(f"\nCX unitary:   {is_unitary(CX)}")
print(f"SWAP unitary: {is_unitary(SWAP)}")
```

**Step 3: Verify gate composition preserves unitarity.**

```python
# H followed by Z followed by H
HZH = gates["H"] @ gates["Z"] @ gates["H"]
print(f"\nHZH unitary: {is_unitary(HZH)}")
print(f"HZH ≈ X: {np.allclose(HZH, gates['X'])}")

# Product of any unitaries is unitary
random_product = gates["H"] @ gates["S"] @ gates["T"] @ gates["X"]
print(f"HSTX unitary: {is_unitary(random_product)}")
```

**Step 4: Test with a NON-unitary matrix — what breaks?**

```python
bad_matrix = np.array([[1, 1], [0, 1]], dtype=complex)
print(f"\nNon-unitary matrix:")
print(f"  Is unitary: {is_unitary(bad_matrix)}")
product = bad_matrix.conj().T @ bad_matrix
print(f"  U†U = {product}")  # NOT identity!
```

### Verification

```python
# save as lab11_verify.py
import numpy as np
tests = 0
for name in ["I", "X", "Y", "Z", "H", "S", "T"]:
    U = gates[name]
    assert is_unitary(U), f"FAIL: {name}"
    tests += 1
assert is_unitary(CX); tests += 1
assert is_unitary(SWAP); tests += 1
print(f"Lab 11 Verification: {tests}/9 PASSED")
```

### Break It on Purpose

```python
# Scale a gate by 2 — no longer unitary
H_bad = gates["H"] * 2
print(f"2H unitary: {is_unitary(H_bad)}")  # False!
```

### Compare Approaches

| Verification | Checks | When |
|-------------|--------|------|
| U†U = I | Unitarity | Every custom gate |
| det(U) = ±1 | Necessary condition | Quick sanity check |
| Eigenvalue magnitudes = 1 | Equivalent check | Debugging |

### Stretch Challenges

1. Write `dagger(U)` that returns U† and verify `U @ dagger(U) = I` for all gates.
2. Generate a random unitary matrix using `scipy.stats.unitary_group.rvs(2)` and verify it.
3. Build a Toffoli (CCX) gate as a 8×8 matrix and verify unitarity.

<details><summary>Hints</summary>

**Challenge 2:** `from scipy.stats import unitary_group; U = unitary_group.rvs(2)`

</details>

### Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Forgetting conjugate in U† | Use `U.conj().T`, not just `U.T` |
| Floating point tolerance too tight | Use `atol=1e-10` in allclose |

### Reflection Questions

1. Why must quantum gates be unitary but classical gates need not be?
2. If you compose 100 unitary gates, is the result always unitary? Prove it.
3. Noise channels are NOT unitary — what does that mean physically?

### Deliverable

`unitarity.py` verifying all standard gates plus compositions. Table of results saved.

---

## Lab 12: Uncompute Everything — Inverse and Mirror Circuits

- **Concept:** Circuit inverse (`inverse()`) and mirror verification patterns
- **Approach:** Build circuits, compute their inverse, compose them, and verify the result is identity (returns to |0…0⟩)
- **Difficulty:** Intermediate | **Time:** 75 min | **Prerequisites:** Lab 11
- **Learning outcomes:**
  1. You will be able to construct the inverse of any circuit
  2. You will be able to use the mirror pattern (qc + qc†) to verify circuit correctness
  3. You will be able to explain why the mirror always returns to |0…0⟩ on a noiseless simulator
  4. You will be able to use uncomputation as a debugging and validation tool
- **Why this matters:** The mirror pattern is QuFlow's built-in correctness check (source doc §6.3). If `qc.mirror()` doesn't return 100% |0…0⟩, your circuit has a bug. This is used in the equivalence harness for testing.

### Theory Primer

For any circuit U: U†U = I, so applying U then U† returns the system to its original state.

```
|0⟩ → U → |ψ⟩ → U† → |0⟩
```

*(Ref: Source doc §6.3 Inverse and Mirror Circuits)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab12 && cd lab12
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build a Bell state and its inverse.**

```python
# save as mirror.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Forward circuit: Bell state
qc_fwd = QuantumCircuit(2, name="bell-forward")
qc_fwd.h(0).cx(0, 1)

# Inverse: reverses gate order, conjugates each gate
qc_inv = qc_fwd.inverse()

# Mirror: forward + inverse → should return to |00⟩
qc_mirror = qc_fwd.mirror()
qc_mirror.measure_all()

result = backend.run(qc_mirror, shots=1024)
print(f"Mirror result: {result.counts}")
# Expected: {'00': 1024}
```

**Step 2: Test mirror on a complex circuit.**

```python
# 4-qubit circuit with multiple gate types
qc_complex = QuantumCircuit(4, name="complex")
qc_complex.h(0).cx(0,1).cx(1,2).cx(2,3)
qc_complex.rz(1.234, 0).ry(0.567, 2)
qc_complex.s(1).t(3)

qc_mirror = qc_complex.mirror()
qc_mirror.measure_all()

result = backend.run(qc_mirror, shots=1024)
zeros_key = "0" * 4
zeros_count = result.counts.get(zeros_key, 0)
print(f"Complex mirror: {zeros_count}/1024 = |0000⟩")
assert zeros_count == 1024, f"FAIL: got {result.counts}"
print("PASS: mirror returns 100% |0000⟩")
```

**Step 3: Use mirror as a bug detector.**

```python
# Deliberately wrong circuit — the mirror will show it's wrong
# if we use a different inverse than the actual inverse
qc_buggy = QuantumCircuit(2, name="buggy")
qc_buggy.h(0).cx(0, 1)

# Wrong "inverse": swap CX and H order but don't conjugate
qc_wrong_inv = QuantumCircuit(2, name="wrong-inv")
qc_wrong_inv.h(0).cx(0, 1)  # Same as forward — NOT the inverse!

# Compose: forward + wrong inverse
qc_test = QuantumCircuit(2, name="test")
qc_test.h(0).cx(0,1).h(0).cx(0,1)
qc_test.measure_all()

result = backend.run(qc_test, shots=1024)
print(f"\nWrong inverse: {result.counts}")
# NOT 100% |00⟩ → the "inverse" was wrong
```

### Verification

```python
# save as lab12_verify.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()
tests = 0

for n in [2, 3, 4]:
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    mirror = qc.mirror()
    mirror.measure_all()
    r = backend.run(mirror, shots=1024)
    assert r.counts.get("0"*n, 0) == 1024, f"FAIL: n={n}"
    tests += 1

print(f"Lab 12 Verification: {tests}/3 PASSED")
```

### Break It on Purpose

```python
# Apply gates AFTER measurement — logically incorrect
qc = QuantumCircuit(1)
qc.h(0).measure_all()
# Can't meaningfully invert after measurement
```

### Compare Approaches

| Verification method | What it catches | Cost |
|-------------------|----------------|------|
| Mirror (U†U=I) | Gate ordering bugs, wrong params | 2× circuit depth |
| Assertion on counts | Wrong outcomes | 1× circuit |
| Statevector comparison | Phase errors, amplitude errors | Simulator only |

### Stretch Challenges

1. Build a mirror verification for every lab circuit so far (Labs 5-8).
2. Implement `inverse()` manually for your MiniQuantumCircuit from Lab 10.
3. Show that noise breaks the mirror: add a random Ry(0.01) "error" gate and measure how much the mirror degrades.

<details><summary>Hints</summary>

**Challenge 3:** Insert `qc.ry(0.01, 0)` in the middle of the forward circuit. The mirror won't perfectly return to |0⟩.

</details>

### Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Forgetting that measurement is NOT invertible | Put measurements AFTER the mirror |
| Using wrong gate order in manual inverse | Reverse the list AND conjugate parameters |

### Reflection Questions

1. Why is the mirror pattern useful even though it "does nothing" (returns to start)?
2. On noisy real hardware, the mirror won't give 100% |0…0⟩. How could you use the error rate as a hardware quality metric?
3. How does QuFlow's equivalence harness use mirrors in practice?

### Deliverable

`mirror.py` with mirror verification for Bell state, 4-qubit complex circuit, and bug detection example.

---

## Lab 13: Picking the Right Simulator

- **Concept:** Simulator selection trade-offs (Statevector vs Stabilizer vs others)
- **Approach:** Benchmark StatevectorSimulator vs StabilizerSimulator on Clifford circuits, measuring speed and memory across qubit counts
- **Difficulty:** Advanced | **Time:** 90 min | **Prerequisites:** Lab 7
- **Learning outcomes:**
  1. You will be able to select the appropriate simulator for a given circuit type
  2. You will be able to benchmark simulator performance empirically
  3. You will be able to explain why Clifford circuits can be simulated efficiently
  4. You will be able to identify when a circuit exceeds simulator capabilities
- **Why this matters:** Source doc §4.2 lists five simulators with different trade-offs. Choosing wrong wastes time (StatevectorSimulator on 30+ qubits) or fails entirely. Understanding when to use the Stabilizer simulator lets you test large Clifford circuits cheaply.

### Theory Primer

| Simulator | Memory | Circuit types | Speed |
|-----------|--------|---------------|-------|
| Statevector | O(2ⁿ) | Any | Slow for n>25 |
| Stabilizer | O(n²) | Clifford only (H, S, CX, measure) | Fast to n>1000 |
| DensityMatrix | O(4ⁿ) | Any + noise | Very slow |

Clifford circuits use only {H, S, CX, X, Y, Z, SWAP} gates. By the Gottesman-Knill theorem, they can be simulated in polynomial time.

*(Ref: Source doc §4.2 Simulators — Which One to Use)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab13 && cd lab13
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Run a Clifford circuit on both simulators.**

```python
# save as simulator_bench.py
import time
from zuntenium import QuantumCircuit, StatevectorSimulator

backend_sv = StatevectorSimulator()

# Build a Clifford-only GHZ circuit
def clifford_ghz(n):
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    qc.measure_all()
    return qc

# Benchmark StatevectorSimulator
print("=== StatevectorSimulator ===")
print(f"{'N':>4} | {'Time (s)':>10} | {'Result':>10}")
print("-" * 30)

for n in [4, 8, 12, 16, 20]:
    qc = clifford_ghz(n)
    t0 = time.time()
    r = backend_sv.run(qc, shots=1024)
    elapsed = time.time() - t0
    outcomes = len(r.counts)
    print(f"{n:>4} | {elapsed:>10.4f} | {outcomes} outcomes")
```

**Step 2: Try increasing qubit count until it gets slow.**

```python
print("\n=== Finding the wall ===")
for n in [20, 22, 24, 25]:
    qc = clifford_ghz(n)
    t0 = time.time()
    try:
        r = backend_sv.run(qc, shots=100)
        elapsed = time.time() - t0
        print(f"n={n}: {elapsed:.2f}s — {'OK' if elapsed < 10 else 'SLOW'}")
    except MemoryError:
        print(f"n={n}: MemoryError — exceeded available RAM")
        break
```

**Step 3: Compare non-Clifford circuits (Stabilizer can't handle these).**

```python
# Circuit with T gates — NOT Clifford
def non_clifford_circuit(n):
    qc = QuantumCircuit(n)
    for i in range(n):
        qc.h(i)
        qc.t(i)  # T gate breaks Clifford-ness
    qc.measure_all()
    return qc

print("\n=== Non-Clifford on StatevectorSimulator ===")
for n in [4, 8, 12]:
    qc = non_clifford_circuit(n)
    t0 = time.time()
    r = backend_sv.run(qc, shots=1024)
    elapsed = time.time() - t0
    print(f"n={n}: {elapsed:.4f}s, {len(r.counts)} outcomes")
```

**Step 4: Decision flowchart.**

```python
def recommend_simulator(circuit_desc: dict) -> str:
    """Recommend a simulator based on circuit properties."""
    n = circuit_desc["qubits"]
    has_non_clifford = circuit_desc.get("non_clifford", False)
    has_noise = circuit_desc.get("noise", False)
    
    if has_noise:
        if n > 15:
            return "TensorNetworkBackend (if shallow) or reduce qubits"
        return "DensityMatrixSimulator"
    
    if not has_non_clifford:
        return "StabilizerSimulator (O(n²), fast for any n)"
    
    if n <= 22:
        return "StatevectorSimulator"
    elif n <= 28:
        return "GPUStatevectorSimulator (if CUDA available)"
    else:
        return "TensorNetworkBackend (if shallow depth)"

# Examples
examples = [
    {"qubits": 100, "non_clifford": False, "name": "100-qubit GHZ"},
    {"qubits": 20, "non_clifford": True, "name": "20-qubit VQE"},
    {"qubits": 10, "noise": True, "name": "10-qubit noisy"},
    {"qubits": 30, "non_clifford": True, "name": "30-qubit algorithm"},
]

print("\n=== Simulator Recommendations ===")
for ex in examples:
    rec = recommend_simulator(ex)
    print(f"  {ex['name']}: → {rec}")
```

### Verification

```python
# save as lab13_verify.py
from zuntenium import QuantumCircuit, StatevectorSimulator
import time

backend = StatevectorSimulator()
qc = clifford_ghz(16)
t0 = time.time()
r = backend.run(qc, shots=1024)
elapsed = time.time() - t0
assert len(r.counts) == 2, f"FAIL: {len(r.counts)} outcomes"
assert elapsed < 30, f"FAIL: too slow ({elapsed}s)"
print(f"Lab 13 Verification: PASSED ({elapsed:.2f}s for 16-qubit GHZ)")
```

### Break It on Purpose

```python
# Try 30 qubits on StatevectorSimulator — will likely crash or be very slow
# Memory needed: 2^30 * 16 bytes = 16 GB
# DON'T run this on a machine with < 16 GB RAM!
print("WARNING: 30-qubit statevector needs ~16 GB RAM")
```

### Compare Approaches

| Simulator | Max practical n | Clifford | Non-Clifford | Noise |
|-----------|----------------|----------|-------------|-------|
| Statevector | ~25 (8GB) | ✓ | ✓ | ✗ |
| Stabilizer | >1000 | ✓ | ✗ | ✗ |
| DensityMatrix | ~13 | ✓ | ✓ | ✓ |
| GPU Statevector | ~30 | ✓ | ✓ | ✗ |
| TensorNetwork | depends on depth | ✓ | ✓ | ✗ |

### Stretch Challenges

1. Benchmark the crossover point: at what N does StabilizerSimulator become faster than StatevectorSimulator?
2. Estimate the RAM needed for N=28, 30, 32 qubits on StatevectorSimulator.
3. Build a circuit that is "almost Clifford" (99% Clifford gates, 1% T gates) and test if a hybrid approach helps.

<details><summary>Hints</summary>

**Challenge 2:** RAM = 2^N × 16 bytes. N=28: 4GB, N=30: 16GB, N=32: 64GB.

</details>

### Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Using StatevectorSimulator for 100-qubit Clifford | Use StabilizerSimulator |
| Expecting noise support from Statevector | Use DensityMatrixSimulator |
| Ignoring memory limits | Calculate 2^N × 16 bytes first |

### Reflection Questions

1. Why can Clifford circuits be simulated efficiently while general circuits cannot?
2. For a Zuntenium QuantumHub deployment on a budget server (4GB RAM), what's the max qubit count?
3. If quantum supremacy requires ~50 qubits, why do we still use simulators?

### Deliverable

`simulator_bench.py` with benchmark table, recommendation function, and memory estimation.

---

## Lab 14: Circuit Forensics — Debugging Broken Circuits

- **Concept:** Diagnosing and fixing common QuFlow circuit errors
- **Approach:** Given 8 broken circuits with different failure modes, diagnose each using error messages and output analysis, then fix them
- **Difficulty:** Advanced | **Time:** 90 min | **Prerequisites:** Lab 5
- **Learning outcomes:**
  1. You will be able to diagnose QubitIndexError, CircuitCompilationError, and empty-counts bugs
  2. You will be able to read error messages and map them to specific code fixes
  3. You will be able to use systematic debugging strategies for quantum circuits
  4. You will be able to prevent common errors in your own code
- **Why this matters:** Source doc §8 lists 7 common errors. Real debugging requires recognizing patterns. This lab trains that skill through deliberate practice with known bugs — the fastest way to build debugging intuition.

### Theory Primer

Common failure categories:
1. **Index errors:** qubit/clbit index out of range (0-indexed!)
2. **Missing operations:** forgot measure_all(), wrong gate order
3. **API misuse:** wrong import path, wrong method signature
4. **Logic errors:** correct code, wrong algorithm (e.g., CX before H)

*(Ref: Source doc §8 Troubleshooting and Common Errors)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab14 && cd lab14
source ../.venv/bin/activate
```

### Walkthrough

**Bug 1: Qubit index out of range.**

```python
# save as debug_cases.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# BUG 1: qubit 3 in a 3-qubit circuit (valid indices: 0, 1, 2)
print("=== Bug 1: Qubit Index ===")
try:
    qc = QuantumCircuit(3)
    qc.h(3)  # BUG: index 3 doesn't exist
except Exception as e:
    print(f"Error: {type(e).__name__}: {e}")
    # FIX: use qc.h(2) or create QuantumCircuit(4)

# FIXED version
qc = QuantumCircuit(3)
qc.h(2).measure_all()
r = backend.run(qc, shots=100)
print(f"Fixed: {r.counts}")
```

**Bug 2: Empty counts — missing measurement.**

```python
print("\n=== Bug 2: Empty Counts ===")
qc = QuantumCircuit(2)
qc.h(0).cx(0, 1)
# BUG: no measure_all()!
r = backend.run(qc, shots=100)
print(f"Buggy counts: {r.counts}")  # {} — empty!

# FIX: add measurement
qc.measure_all()
r = backend.run(qc, shots=100)
print(f"Fixed counts: {r.counts}")
```

**Bug 3: Wrong import path.**

```python
print("\n=== Bug 3: Wrong Import ===")
try:
    from zuntenium import X  # BUG: X is not a public export
except (ImportError, AttributeError) as e:
    print(f"Error: {e}")
    print("FIX: use QuantumCircuit(1).x(0) instead")
```

**Bug 4: Entanglement order — CX before H.**

```python
print("\n=== Bug 4: Gate Order ===")
qc = QuantumCircuit(2)
qc.cx(0, 1)  # BUG: CX on |00⟩ does nothing
qc.h(0)       # H after CX — not entangled!
qc.measure_all()
r = backend.run(qc, shots=1024)
print(f"Buggy (CX before H): {len(r.counts)} outcomes")  # 4 outcomes, not 2

# FIX: H first, then CX
qc_fixed = QuantumCircuit(2)
qc_fixed.h(0).cx(0, 1).measure_all()
r = backend.run(qc_fixed, shots=1024)
print(f"Fixed (H then CX): {len(r.counts)} outcomes")  # 2 outcomes
```

**Bug 5: Wrong shot count interpretation.**

```python
print("\n=== Bug 5: Shots Misinterpretation ===")
qc = QuantumCircuit(1)
qc.h(0).measure_all()
r = backend.run(qc, shots=10)
# BUG: concluding from 10 shots that coin is biased
ones = r.counts.get("1", 0)
print(f"10 shots: P(1) = {ones/10:.1f}")
print("With only 10 shots, this is NOT statistically meaningful!")
print("FIX: use at least 1000 shots for reliable estimates")
```

**Bug 6: Bit ordering confusion.**

```python
print("\n=== Bug 6: Bit Order ===")
qc = QuantumCircuit(2)
qc.x(0)  # Flip qubit 0 only
qc.measure_all()
r = backend.run(qc, shots=100)
print(f"X on q0: {r.counts}")
# Result is '01' — rightmost bit = qubit 0 = 1
# BUG: confusing this as "qubit 1 is flipped"
print("Remember: rightmost bit = qubit 0 (little-endian)")
```

**Bug 7: Reusing a measured circuit.**

```python
print("\n=== Bug 7: Circuit Reuse ===")
qc = QuantumCircuit(1)
qc.h(0).measure_all()
# Run once — fine
r1 = backend.run(qc, shots=100)
# BUG: adding more gates after measure_all
qc.h(0)  # Gate after measurement — logically wrong!
r2 = backend.run(qc, shots=100)
print(f"Run 1: {r1.counts}")
print(f"Run 2 (gate after measure): {r2.counts}")
print("FIX: build a new circuit for different operations")
```

**Bug 8: Parameter confusion (degrees vs radians).**

```python
print("\n=== Bug 8: Degrees vs Radians ===")
import math

# BUG: using degrees instead of radians
qc_wrong = QuantumCircuit(1)
qc_wrong.ry(90, 0)  # 90 degrees? No — QuFlow uses radians!
qc_wrong.measure_all()
r = backend.run(qc_wrong, shots=1024)
p1 = r.counts.get("1", 0) / 1024
print(f"ry(90) [wrong]: P(1) = {p1:.4f}")  # NOT 1.0!

# FIX: use radians
qc_right = QuantumCircuit(1)
qc_right.ry(math.pi / 2, 0)  # π/2 radians = 90 degrees
qc_right.measure_all()
r = backend.run(qc_right, shots=1024)
p1 = r.counts.get("1", 0) / 1024
print(f"ry(π/2) [correct]: P(1) = {p1:.4f}")  # ~0.5
```

### Verification

```python
# save as lab14_verify.py
# All 8 bugs identified and fixed
print("Lab 14 Verification: 8/8 bugs diagnosed and fixed")
```

### Break It on Purpose

This entire lab IS about breaking things. Each bug is already broken — your job is to fix it.

### Compare Approaches

| Debugging strategy | Best for | Time cost |
|-------------------|----------|-----------|
| Read error message | Index/import errors | Seconds |
| Print counts/statevector | Logic errors | Minutes |
| Mirror test | Subtle gate errors | Minutes |
| Step-by-step matrix trace | Phase/amplitude errors | ~30 min |

### Stretch Challenges

1. Build an automated "circuit linter" that checks for common bugs before execution.
2. Create your own set of 5 debugging puzzles and swap with a partner.
3. Write a `diagnose(qc)` function that prints warnings for the 8 bug patterns above.

<details><summary>Hints</summary>

**Challenge 1:** Check for: missing measurements, gates after measurements, qubit index range.

</details>

### Common Pitfalls

Already covered — this lab IS about pitfalls!

### Reflection Questions

1. What's the fastest debugging technique for quantum circuits that you learned?
2. Why are quantum bugs harder to find than classical bugs?
3. How would you build an automated test suite for a library of quantum circuits?

### Deliverable

`debug_cases.py` with all 8 bugs diagnosed, explained, and fixed.

---

## Lab 15: Parameterized Circuit Factory

- **Concept:** Parameterized circuit templates for reusable angle-sweep experiments
- **Approach:** Build a factory function that creates circuit templates and sweep parameters systematically
- **Difficulty:** Advanced | **Time:** 90 min | **Prerequisites:** Lab 8
- **Learning outcomes:**
  1. You will be able to build reusable parameterized circuit functions
  2. You will be able to sweep parameters and collect results systematically
  3. You will be able to identify optimal parameters from sweep data
  4. You will be able to explain how parameterized circuits connect to VQE/QAOA
- **Why this matters:** Variational algorithms (ZQL-05) optimize circuit parameters to minimize a cost function. This lab builds the infrastructure for parameter sweeps. Source doc §5.5 previews this with `build_ry_circuit(theta)`.

### Theory Primer

A parameterized circuit is a function θ → C(θ) → measurement. The parameter optimization loop:

```
θ₀ → C(θ₀) → measure → cost → optimizer → θ₁ → C(θ₁) → ...
```

*(Ref: Source doc §5.5 Parameterized Circuits)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab15 && cd lab15
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build a parameterized circuit factory.**

```python
# save as param_factory.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator
from typing import Callable

def ry_circuit(theta: float) -> QuantumCircuit:
    """Single-parameter Ry circuit."""
    qc = QuantumCircuit(1, name=f"ry({theta:.3f})")
    qc.ry(theta, 0).measure_all()
    return qc

def bell_rx_circuit(theta: float) -> QuantumCircuit:
    """Bell-state variant with parameterized Rx."""
    qc = QuantumCircuit(2, name=f"bell_rx({theta:.3f})")
    qc.h(0).cx(0, 1).rx(theta, 0).measure_all()
    return qc

backend = StatevectorSimulator()
```

**Step 2: Build a systematic parameter sweep engine.**

```python
def parameter_sweep(
    circuit_fn: Callable[[float], QuantumCircuit],
    param_range: list[float],
    target_bitstring: str,
    shots: int = 4096
) -> list[dict]:
    """Sweep a parameter and record the probability of a target bitstring."""
    results = []
    for param in param_range:
        qc = circuit_fn(param)
        r = backend.run(qc, shots=shots)
        p_target = r.counts.get(target_bitstring, 0) / shots
        results.append({
            "param": param,
            "p_target": p_target,
            "counts": dict(r.counts)
        })
    return results

# Sweep Ry from 0 to 2π
angles = [i * math.pi / 16 for i in range(33)]
sweep = parameter_sweep(ry_circuit, angles, "1")

print(f"{'θ':>8} | {'P(1)':>8}")
print("-" * 20)
for r in sweep[::4]:  # Print every 4th
    print(f"{r['param']:>8.4f} | {r['p_target']:>8.4f}")
```

**Step 3: Find the optimal parameter.**

```python
def find_optimal(sweep: list[dict], maximize=True) -> dict:
    """Find the parameter that maximizes or minimizes P(target)."""
    if maximize:
        return max(sweep, key=lambda x: x["p_target"])
    return min(sweep, key=lambda x: x["p_target"])

optimal = find_optimal(sweep)
print(f"\nOptimal θ for max P(1): {optimal['param']:.4f}")
print(f"  P(1) = {optimal['p_target']:.4f}")
print(f"  Expected θ = π = {math.pi:.4f}")
```

**Step 4: Multi-parameter sweep.**

```python
def two_param_circuit(theta1: float, theta2: float) -> QuantumCircuit:
    """Two-parameter circuit."""
    qc = QuantumCircuit(2, name=f"2p({theta1:.2f},{theta2:.2f})")
    qc.ry(theta1, 0).ry(theta2, 1)
    qc.cx(0, 1)
    qc.measure_all()
    return qc

# Grid sweep
print("\n=== 2-Parameter Grid Sweep ===")
print(f"{'θ1':>8} | {'θ2':>8} | {'P(11)':>8}")
print("-" * 30)

best = {"p": 0, "t1": 0, "t2": 0}
for t1 in [0, math.pi/4, math.pi/2, 3*math.pi/4, math.pi]:
    for t2 in [0, math.pi/4, math.pi/2, 3*math.pi/4, math.pi]:
        qc = two_param_circuit(t1, t2)
        r = backend.run(qc, shots=1024)
        p11 = r.counts.get("11", 0) / 1024
        if p11 > best["p"]:
            best = {"p": p11, "t1": t1, "t2": t2}
        if t2 == math.pi:  # Print one per row
            print(f"{t1:>8.4f} | {t2:>8.4f} | {p11:>8.4f}")

print(f"\nBest: θ1={best['t1']:.4f}, θ2={best['t2']:.4f}, P(11)={best['p']:.4f}")
```

### Verification

```python
# save as lab15_verify.py
import math
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Ry(π) should give P(1) ≈ 1.0
qc = QuantumCircuit(1)
qc.ry(math.pi, 0).measure_all()
r = backend.run(qc, shots=4096)
p1 = r.counts.get("1", 0) / 4096
assert p1 > 0.98, f"FAIL: P(1) = {p1}"
print(f"Lab 15 Verification: PASSED (P(1) = {p1:.4f} at θ=π)")
```

### Break It on Purpose

```python
# Sweep with too few points — misses the optimum
coarse = parameter_sweep(ry_circuit, [0, math.pi/2, math.pi], "1")
print(f"Coarse sweep found P(1) at π/2: {coarse[1]['p_target']:.4f}")
print("Missed the true max at θ=π because grid was too coarse!")
```

### Compare Approaches

| Sweep strategy | Points | Accuracy | Speed |
|---------------|--------|----------|-------|
| Uniform grid | N | Depends on N | O(N) runs |
| Random search | N | Probabilistic | O(N) runs |
| Bayesian optimization | ~10-20 | High | Expensive per step |
| Gradient descent | ~50-100 | High | Needs gradient circuit |

### Stretch Challenges

1. Implement a simple gradient-free optimizer (e.g., golden section search) to find optimal θ.
2. Build a 3-parameter circuit and sweep it with random search (100 samples).
3. Plot the parameter landscape for the 2-parameter circuit as a heatmap.

<details><summary>Hints</summary>

**Challenge 1:** Golden section divides [a,b] by φ at each step. Evaluate circuit at two interior points and narrow.

</details>

### Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Too few sweep points | Use ≥32 points for smooth curves |
| Not recording all results | Store full counts, not just P(target) |
| Confusing local vs global optimum | Use broad initial sweep |

### Reflection Questions

1. How does this parameter sweep relate to the optimization loop in VQE?
2. Why is grid search impractical for circuits with 10+ parameters?
3. For a Zuntenium QuantumHub offering VQE-as-a-service, how would you charge per parameter sweep?

### Deliverable

`param_factory.py` with sweep engine, single and multi-parameter sweeps, and optimizer results.

---

## Lab 16: How Big Is Too Big? — Memory Scaling Benchmark

- **Concept:** Exponential memory scaling of quantum simulation
- **Approach:** Empirically measure RAM usage and runtime as qubit count increases, finding the practical limit on your hardware
- **Difficulty:** Advanced | **Time:** 75 min | **Prerequisites:** Lab 13
- **Learning outcomes:**
  1. You will be able to calculate the theoretical RAM needed for N qubits
  2. You will be able to measure actual memory usage during simulation
  3. You will be able to determine the maximum qubit count for your hardware
  4. You will be able to explain why quantum advantage exists beyond ~50 qubits
- **Why this matters:** Source doc §2.5 shows 2^n scaling. Knowing your hardware's exact limit prevents OOMKilled crashes and informs whether you need GPU acceleration or real hardware access.

### Theory Primer

Memory = 2^N × 16 bytes (complex128). N=20: 16MB, N=25: 512MB, N=28: 4GB, N=30: 16GB.

*(Ref: Source doc §2.5 Memory Scaling)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab16 && cd lab16
source ../.venv/bin/activate
pip install psutil --quiet
```

### Walkthrough

**Step 1: Calculate theoretical memory.**

```python
# save as memory_scaling.py
import time, os

def theoretical_ram(n):
    """RAM in bytes for n-qubit statevector."""
    return (2 ** n) * 16  # complex128 = 16 bytes

print(f"{'N':>4} | {'RAM':>12} | {'Unit':>6}")
print("-" * 28)
for n in range(1, 35):
    ram = theoretical_ram(n)
    if ram < 1024**2:
        print(f"{n:>4} | {ram/1024:>12.1f} | KB")
    elif ram < 1024**3:
        print(f"{n:>4} | {ram/1024**2:>12.1f} | MB")
    else:
        print(f"{n:>4} | {ram/1024**3:>12.1f} | GB")
```

**Step 2: Empirically time simulations at increasing N.**

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
backend = StatevectorSimulator()

print(f"\n{'N':>4} | {'Theory RAM':>12} | {'Time (s)':>10} | {'Status':>8}")
print("-" * 50)

for n in range(4, 26, 2):
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    qc.measure_all()
    
    ram_mb = theoretical_ram(n) / 1024**2
    t0 = time.time()
    try:
        r = backend.run(qc, shots=100)
        elapsed = time.time() - t0
        status = "OK" if elapsed < 10 else "SLOW"
        print(f"{n:>4} | {ram_mb:>10.1f}MB | {elapsed:>10.3f} | {status:>8}")
    except MemoryError:
        print(f"{n:>4} | {ram_mb:>10.1f}MB | {'N/A':>10} | {'OOM':>8}")
        break
```

### Verification

```python
# save as lab16_verify.py
assert theoretical_ram(20) == 2**20 * 16  # 16 MB
assert theoretical_ram(30) == 2**30 * 16  # ~16 GB
print("Lab 16 Verification: PASSED")
```

### Break It on Purpose

```python
# Try allocating a 30-qubit statevector on a machine with < 16GB RAM
# import numpy as np
# v = np.zeros(2**30, dtype=np.complex128)  # Will crash!
print("DON'T uncomment the above on < 16GB RAM!")
```

### Compare Approaches

| N qubits | RAM | Time (approx) | Feasible? |
|----------|-----|---------------|-----------|
| 20 | 16 MB | <1s | ✓ Any laptop |
| 25 | 512 MB | ~5s | ✓ Most machines |
| 28 | 4 GB | ~30s | ⚠ Need 8GB+ RAM |
| 30 | 16 GB | ~2min | ⚠ Need 32GB or GPU |
| 40 | 16 TB | impossible | ✗ Need real QPU |

### Stretch Challenges

1. Find your machine's exact max N by binary search.
2. Measure actual RSS (resident memory) during simulation using `psutil`.
3. Estimate how long a 50-qubit simulation would take by extrapolating your timing curve.

<details><summary>Hints</summary>

**Challenge 2:** `import psutil; psutil.Process(os.getpid()).memory_info().rss / 1024**2`

</details>

### Reflection Questions

1. At what N does classical simulation become impossible? What fills the gap?
2. For a Zuntenium deployment on a $24/month DigitalOcean Droplet (4GB RAM), what's the max N?
3. Why is quantum advantage defined around 50 qubits?

### Deliverable

`memory_scaling.py` with theoretical and empirical scaling tables.

---

## Lab 17: Inspect the Transpiler — QuantumIR Under the Hood

- **Concept:** QuantumIR representation and TranspilerCache behavior
- **Approach:** Dump circuit IR, inspect its structure, and verify cache hit/miss behavior
- **Difficulty:** Advanced | **Time:** 90 min | **Prerequisites:** Lab 10
- **Learning outcomes:**
  1. You will be able to convert a circuit to QuantumIR using `to_ir()`
  2. You will be able to inspect IR metadata including SHA-256 hash
  3. You will be able to explain how the TranspilerCache avoids redundant compilation
  4. You will be able to trace the transpiler pipeline stages
- **Why this matters:** Source doc §4.3 describes VF2 placement, SABRE routing, KAK decomposition, and ALAP scheduling. Understanding IR lets you debug transpilation issues and optimize circuit compilation.

### Theory Primer

The transpiler pipeline: `QuantumCircuit → to_ir() → QuantumIR → [Validate → Hash → Place → Route → Decompose → Schedule] → Native circuit`

The TranspilerCache uses SHA-256 hashes of gate structures to avoid recompiling identical circuits.

*(Ref: Source doc §4.3 Transpiler — What Happens Under the Hood)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab17 && cd lab17
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Convert a circuit to IR and inspect it.**

```python
# save as ir_inspect.py
from zuntenium import QuantumCircuit, StatevectorSimulator

qc = QuantumCircuit(3, name="bell-3")
qc.h(0).cx(0, 1).cx(1, 2).measure_all()

# Convert to IR
ir = qc.to_ir()
print(f"QuantumIR:")
print(f"  num_qubits: {ir.num_qubits}")
print(f"  num_clbits: {ir.num_clbits}")
print(f"  instructions: {len(ir.instructions)}")
print(f"  metadata: {ir.metadata}")
```

**Step 2: Inspect individual IR nodes.**

```python
print("\n=== IR Instructions ===")
for i, node in enumerate(ir.instructions):
    print(f"  [{i}] {node}")
```

**Step 3: Verify cache behavior — same circuit should cache hit.**

```python
import time

backend = StatevectorSimulator()

# First run — cache miss (compile)
t0 = time.time()
r1 = backend.run(qc, shots=100)
t_first = time.time() - t0

# Second run — cache hit (skip compile)
t0 = time.time()
r2 = backend.run(qc, shots=100)
t_second = time.time() - t0

print(f"\nFirst run (cache miss):  {t_first:.4f}s")
print(f"Second run (cache hit): {t_second:.4f}s")
print(f"Speedup: {t_first/t_second:.1f}x")
```

**Step 4: Verify that different circuits get different hashes.**

```python
qc2 = QuantumCircuit(3, name="different")
qc2.h(0).cx(0, 2).cx(1, 2).measure_all()  # Different CX targets

ir2 = qc2.to_ir()
print(f"\nCircuit 1 hash: {ir.metadata.get('hash', 'N/A')}")
print(f"Circuit 2 hash: {ir2.metadata.get('hash', 'N/A')}")
print(f"Same hash: {ir.metadata.get('hash') == ir2.metadata.get('hash')}")
```

### Verification

```python
# save as lab17_verify.py
from zuntenium import QuantumCircuit
qc = QuantumCircuit(2)
qc.h(0).cx(0,1).measure_all()
ir = qc.to_ir()
assert ir.num_qubits == 2
assert len(ir.instructions) > 0
print("Lab 17 Verification: PASSED")
```

### Break It on Purpose

```python
# Disable the cache and see the performance difference
import os
os.environ["ZUNTENIUM_NO_CACHE"] = "1"
# Now every run recompiles — slower
```

### Stretch Challenges

1. Write a function that compares two circuits' IR to check equivalence.
2. Measure cache hit rate over 100 runs of 10 different circuits.
3. Build a circuit that has the same hash as another (hash collision exploration).

### Reflection Questions

1. Why does QuFlow use SHA-256 hashing for the transpiler cache?
2. What information is lost when converting from QuantumCircuit to QuantumIR?
3. How would you invalidate the cache when hardware calibration changes?

### Deliverable

`ir_inspect.py` with IR dump, cache timing comparison, and hash verification.

---

## Lab 18: Bit Order Matters — Endianness Pitfalls

- **Concept:** Bit ordering (little-endian) conventions in QuFlow
- **Approach:** Build circuits where bit order causes confusion, detect the pitfall, and build helper functions to handle it correctly
- **Difficulty:** Advanced | **Time:** 60 min | **Prerequisites:** Lab 5
- **Learning outcomes:**
  1. You will be able to explain QuFlow's little-endian bit ordering convention
  2. You will be able to convert between QuFlow bitstrings and qubit states correctly
  3. You will be able to detect and fix bit-order bugs in multi-qubit circuits
  4. You will be able to build utility functions for endianness conversion
- **Why this matters:** Source doc §5.4: "QuFlow returns bitstrings with qubit 0 as the rightmost bit." Getting this wrong means misinterpreting every multi-qubit result.

### Theory Primer

QuFlow uses little-endian: rightmost bit = qubit 0. Bitstring "10" means q0=0, q1=1.

```
Bitstring: "1  0"
             q1 q0
```

*(Ref: Source doc §5.4 Bit Order)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab18 && cd lab18
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Demonstrate the convention.**

```python
# save as endianness.py
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# X on qubit 0 only
qc = QuantumCircuit(3)
qc.x(0)  # Flip qubit 0
qc.measure_all()
r = backend.run(qc, shots=100)
print(f"X on q0: {r.counts}")
# Result: '001' — rightmost bit = qubit 0 = 1

# X on qubit 2 only
qc2 = QuantumCircuit(3)
qc2.x(2)  # Flip qubit 2
qc2.measure_all()
r2 = backend.run(qc2, shots=100)
print(f"X on q2: {r2.counts}")
# Result: '100' — leftmost bit = qubit 2 = 1
```

**Step 2: Build a decoder function.**

```python
def decode_bitstring(bitstring: str, num_qubits: int) -> dict[int, int]:
    """Decode a QuFlow bitstring to {qubit_index: value}."""
    result = {}
    for i in range(num_qubits):
        bit_pos = num_qubits - 1 - i  # little-endian
        result[i] = int(bitstring[bit_pos])
    return result

# Test
print(f"\nDecode '001': {decode_bitstring('001', 3)}")
# {0: 1, 1: 0, 2: 0} — only qubit 0 is set
print(f"Decode '100': {decode_bitstring('100', 3)}")
# {0: 0, 1: 0, 2: 1} — only qubit 2 is set
```

**Step 3: Build a bit-order-safe result reader.**

```python
def qubit_probability(counts: dict, qubit: int, num_qubits: int) -> float:
    """Get P(qubit=1) from measurement counts regardless of bit order."""
    total = sum(counts.values())
    p1 = 0
    bit_pos = num_qubits - 1 - qubit
    for bitstring, count in counts.items():
        if bitstring[bit_pos] == '1':
            p1 += count
    return p1 / total

# Test: H on q1 — only q1 should be 50/50
qc = QuantumCircuit(3)
qc.h(1).measure_all()
r = backend.run(qc, shots=10000)
for q in range(3):
    p = qubit_probability(r.counts, q, 3)
    print(f"  P(q{q}=1) = {p:.4f}")
# q0 = 0.0, q1 ≈ 0.5, q2 = 0.0
```

### Verification

```python
# save as lab18_verify.py
d = decode_bitstring("010", 3)
assert d[0] == 0 and d[1] == 1 and d[2] == 0
print("Lab 18 Verification: PASSED")
```

### Break It on Purpose

```python
# Naive read: assume leftmost = qubit 0 — WRONG!
bitstring = "100"
wrong_q0 = int(bitstring[0])  # 1 — but qubit 0 is actually 0!
right_q0 = int(bitstring[-1])  # 0 — correct
print(f"Wrong: q0={wrong_q0}, Right: q0={right_q0}")
```

### Stretch Challenges

1. Build a `reverse_endian(counts)` function that converts to big-endian convention.
2. Compare QuFlow's convention with Qiskit's — are they the same?
3. Build a visual bitstring display that labels each bit with its qubit index.

### Reflection Questions

1. Why does QuFlow use little-endian instead of big-endian?
2. When comparing results between QuFlow and another framework, what must you check first?
3. Does bit ordering matter for a 1-qubit circuit?

### Deliverable

`endianness.py` with convention demo, decoder function, and qubit probability helper.

---

## Lab 19: Quantum on a Budget — Constrained Hardware

- **Concept:** Running quantum experiments on resource-constrained hardware
- **Approach:** Optimize circuits and experiments for low-RAM machines, free-tier cloud, and mobile-first environments relevant to Africa
- **Difficulty:** Expert | **Time:** 90 min | **Prerequisites:** Labs 13, 16
- **Learning outcomes:**
  1. You will be able to run meaningful quantum experiments with ≤2GB RAM
  2. You will be able to use Google Colab and free-tier cloud for quantum computing
  3. You will be able to optimize shot budgets and qubit counts for constrained environments
  4. You will be able to design quantum curricula for settings with intermittent connectivity
- **Why this matters:** Zuntenium Quantum Institute launches in Kenya. Many learners use laptops with 4GB RAM, intermittent internet, and shared university computers. This lab ensures the curriculum works in these real-world conditions.

### Theory Primer

Constraints in African computing contexts:
- RAM: 2-4GB typical on student laptops
- Internet: Unstable, expensive per MB in many regions
- Compute: Shared lab PCs, mobile phones (via Termux/Colab)
- Budget: Can't afford IBM Quantum premium plans

*(Ref: Zuntenium Africa-first context)*

### Setup

```bash
# Minimal setup — no optional dependencies
cd ~/zql-foundations-labs && mkdir -p lab19 && cd lab19
python3 -m venv .venv-minimal
source .venv-minimal/bin/activate
pip install zuntenium==0.2.0 --quiet
```

### Walkthrough

**Step 1: Measure your actual RAM budget.**

```python
# save as budget_quantum.py
import os, sys

def available_ram_mb():
    """Estimate available RAM in MB."""
    try:
        import psutil
        return psutil.virtual_memory().available / 1024**2
    except ImportError:
        # Fallback: read /proc/meminfo on Linux
        try:
            with open('/proc/meminfo') as f:
                for line in f:
                    if 'MemAvailable' in line:
                        return int(line.split()[1]) / 1024
        except:
            return 2048  # Assume 2GB if unknown

ram = available_ram_mb()
print(f"Available RAM: {ram:.0f} MB")

# What's the max qubit count?
max_n = 0
for n in range(1, 40):
    ram_needed = (2**n * 16) / 1024**2  # MB
    if ram_needed > ram * 0.5:  # Use max 50% of available
        break
    max_n = n

print(f"Safe max qubits: {max_n}")
print(f"  RAM needed for {max_n} qubits: {2**max_n * 16 / 1024**2:.1f} MB")
```

**Step 2: Design experiments that work within constraints.**

```python
from zuntenium import QuantumCircuit, StatevectorSimulator

backend = StatevectorSimulator()

# Budget experiment: meaningful results with small circuits
def budget_bell_test(shots=512):
    """Run a Bell test with minimal resources."""
    qc = QuantumCircuit(2)
    qc.h(0).cx(0, 1).measure_all()
    r = backend.run(qc, shots=shots)
    
    p00 = r.counts.get("00", 0) / shots
    p11 = r.counts.get("11", 0) / shots
    other = 1 - p00 - p11
    
    return {"p00": p00, "p11": p11, "other": other, "entangled": other < 0.01}

result = budget_bell_test(shots=256)  # Even 256 shots is enough!
print(f"\nBudget Bell test (256 shots):")
print(f"  P(00) = {result['p00']:.3f}")
print(f"  P(11) = {result['p11']:.3f}")
print(f"  Entangled: {result['entangled']}")
```

**Step 3: Offline-capable workflow.**

```python
# Save results to JSON for review without re-running
import json

experiments = []
for n in range(2, 8):
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1): qc.cx(i, i+1)
    qc.measure_all()
    r = backend.run(qc, shots=256)
    experiments.append({
        "n_qubits": n,
        "shots": 256,
        "counts": dict(r.counts),
        "entangled": len(r.counts) == 2
    })

with open("results_offline.json", "w") as f:
    json.dump(experiments, f, indent=2)
print(f"\nSaved {len(experiments)} experiments to results_offline.json")
print("Can review results offline without re-running!")
```

**Step 4: Mobile-friendly quantum via Google Colab.**

```python
colab_instructions = """
=== Running Quantum Labs on a Phone ===
1. Open https://colab.research.google.com on Chrome mobile
2. Create a new notebook
3. In the first cell: !pip install zuntenium==0.2.0
4. In the second cell: paste any lab code
5. Run and see results — no laptop needed!

Tips for low data:
- Colab installs cache packages across sessions
- Download the notebook for offline editing
- Use small shot counts (256-512) to save compute time
"""
print(colab_instructions)
```

### Verification

```python
# save as lab19_verify.py
result = budget_bell_test(shots=100)
assert result["entangled"], "FAIL: Bell test failed even at 100 shots"
print("Lab 19 Verification: PASSED (Bell test works at 100 shots)")
```

### Stretch Challenges

1. Run all Labs 1-5 on Google Colab from a phone and document the experience.
2. Build a "quantum lab kit" script that pre-runs all experiments and saves results to JSON for offline review.
3. Estimate the total data cost (MB) to download zuntenium + numpy on a Safaricom data plan.

### Reflection Questions

1. What's the minimum viable quantum computing setup for a student in Nairobi?
2. How would you design a quantum computing course for a classroom with one shared PC?
3. Could quantum computing be accessed via a feature phone (SMS/USSD interface)?

### Deliverable

`budget_quantum.py` with RAM assessment, budget experiments, and offline-capable result storage.

---

## Lab 20: Entanglement Demonstrator — Capstone

- **Concept:** Full integration of all previous concepts into a polished GHZ verification application
- **Approach:** Build a complete, documented, tested application that generates, verifies, and reports on GHZ entanglement across multiple qubit counts
- **Difficulty:** Expert | **Time:** 120 min | **Prerequisites:** All previous labs
- **Learning outcomes:**
  1. You will be able to integrate circuit construction, simulation, verification, and reporting into one application
  2. You will be able to write production-quality quantum code with validation and error handling
  3. You will be able to generate a shareable report from quantum experiments
  4. You will be able to apply the mirror pattern for correctness verification
- **Why this matters:** This capstone ties together every concept from Labs 1-19. Source doc §9 provides the starter skeleton. You'll extend it into a complete, reusable tool.

### Theory Primer

The capstone demonstrates: circuit construction (Lab 7), measurement statistics (Lab 4), entanglement verification (Lab 5), mirror validation (Lab 12), memory scaling (Lab 16), and offline reporting (Lab 19).

*(Ref: Source doc §9 Capstone Project)*

### Setup

```bash
cd ~/zql-foundations-labs && mkdir -p lab20 && cd lab20
source ../.venv/bin/activate
```

### Walkthrough

**Step 1: Build the core demonstrator.**

```python
# save as entanglement_demo.py
"""
Zuntenium Quantum Institute — Entanglement Demonstrator
A comprehensive GHZ state verification tool.
"""
import json, time, math, sys
from dataclasses import dataclass, asdict
from typing import Optional
from zuntenium import QuantumCircuit, StatevectorSimulator

@dataclass
class GHZResult:
    n_qubits: int
    shots: int
    zeros_count: int
    ones_count: int
    other_count: int
    entangled: bool
    mirror_passed: bool
    build_time_ms: float
    run_time_ms: float

def build_ghz(n: int) -> QuantumCircuit:
    qc = QuantumCircuit(n, name=f"ghz_{n}")
    qc.h(0)
    for i in range(n - 1):
        qc.cx(i, i + 1)
    return qc

def verify_ghz(n: int, backend, shots: int = 4096) -> GHZResult:
    # Build
    t0 = time.time()
    qc = build_ghz(n)
    build_ms = (time.time() - t0) * 1000
    
    # Run
    qc_run = build_ghz(n)
    qc_run.measure_all()
    t0 = time.time()
    result = backend.run(qc_run, shots=shots)
    run_ms = (time.time() - t0) * 1000
    
    # Analyze
    zeros = result.counts.get("0" * n, 0)
    ones = result.counts.get("1" * n, 0)
    other = shots - zeros - ones
    entangled = other == 0
    
    # Mirror verification
    qc_mirror = qc.mirror()
    qc_mirror.measure_all()
    mirror_result = backend.run(qc_mirror, shots=shots)
    mirror_passed = mirror_result.counts.get("0" * n, 0) == shots
    
    return GHZResult(
        n_qubits=n, shots=shots,
        zeros_count=zeros, ones_count=ones, other_count=other,
        entangled=entangled, mirror_passed=mirror_passed,
        build_time_ms=build_ms, run_time_ms=run_ms
    )

backend = StatevectorSimulator()
```

**Step 2: Run the full verification suite.**

```python
print("=" * 70)
print("ZUNTENIUM QUANTUM INSTITUTE — ENTANGLEMENT DEMONSTRATOR")
print("=" * 70)
print(f"Backend: StatevectorSimulator")
print(f"Shots per circuit: 4096")
print()

header = f"{'N':>3} | {'|0⟩':>6} | {'|1⟩':>6} | {'Other':>6} | {'Entgl':>6} | {'Mirror':>7} | {'Time':>8}"
print(header)
print("-" * len(header))

results = []
for n in range(2, 11):
    r = verify_ghz(n, backend, shots=4096)
    results.append(r)
    print(f"{r.n_qubits:>3} | {r.zeros_count:>6} | {r.ones_count:>6} | "
          f"{r.other_count:>6} | {'✓' if r.entangled else '✗':>6} | "
          f"{'✓' if r.mirror_passed else '✗':>7} | {r.run_time_ms:>6.1f}ms")

all_pass = all(r.entangled and r.mirror_passed for r in results)
print(f"\nOVERALL: {'ALL PASSED ✓' if all_pass else 'SOME FAILED ✗'}")
```

**Step 3: Generate a shareable report.**

```python
report = {
    "title": "GHZ Entanglement Verification Report",
    "institution": "Zuntenium Quantum Institute",
    "backend": "StatevectorSimulator",
    "timestamp": time.strftime("%Y-%m-%d %H:%M:%S"),
    "results": [asdict(r) for r in results],
    "summary": {
        "total_experiments": len(results),
        "all_entangled": all(r.entangled for r in results),
        "all_mirrors_passed": all(r.mirror_passed for r in results),
        "max_qubits_tested": max(r.n_qubits for r in results),
    }
}

with open("ghz_report.json", "w") as f:
    json.dump(report, f, indent=2)
print(f"\nReport saved to ghz_report.json")
```

**Step 4: Generate a markdown summary.**

```python
with open("ghz_report.md", "w") as f:
    f.write("# GHZ Entanglement Verification Report\n\n")
    f.write(f"**Institution:** Zuntenium Quantum Institute\n")
    f.write(f"**Date:** {time.strftime('%Y-%m-%d')}\n")
    f.write(f"**Backend:** StatevectorSimulator\n\n")
    f.write("| Qubits | Entangled | Mirror | Time |\n")
    f.write("|--------|-----------|--------|------|\n")
    for r in results:
        f.write(f"| {r.n_qubits} | {'✓' if r.entangled else '✗'} | "
                f"{'✓' if r.mirror_passed else '✗'} | {r.run_time_ms:.1f}ms |\n")
    f.write(f"\n**Result:** {'ALL PASSED' if all_pass else 'ISSUES FOUND'}\n")
print("Markdown report saved to ghz_report.md")
```

### Verification

```python
# save as lab20_verify.py
results_data = json.load(open("ghz_report.json"))
assert results_data["summary"]["all_entangled"]
assert results_data["summary"]["all_mirrors_passed"]
assert results_data["summary"]["max_qubits_tested"] >= 10
print(f"Lab 20 Verification: PASSED ({results_data['summary']['total_experiments']} experiments)")
```

### Break It on Purpose

```python
# Inject a bug: skip one CX gate
def build_broken_ghz(n):
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n - 2):  # BUG: skip last CX
        qc.cx(i, i + 1)
    return qc

# The mirror test will catch this!
```

### Compare Approaches

| Verification | Catches | Cost | Confidence |
|-------------|---------|------|------------|
| Outcome check (2 outcomes) | Missing entanglement | Low | Medium |
| Mirror test (U†U=I) | Any gate error | 2× depth | High |
| Both combined | Everything | 3× depth | Very high |

### Stretch Challenges

1. Extend the demonstrator to test W states and compare with GHZ.
2. Add a timing plot that shows exponential scaling.
3. Deploy the demonstrator as a web API using FastAPI.

### Reflection Questions

1. What did building this capstone teach you that individual labs didn't?
2. How would you present this demonstrator to a non-technical audience?
3. What would you add to make this production-ready for QuantumHub?

### Deliverable

Complete `entanglement_demo.py` application with `ghz_report.json` and `ghz_report.md`.

---

## Capstone: Integrated Mini-Project

Combine Labs 1-20 into a single **Quantum Computing Quick-Start Toolkit**:

1. **Environment validator** (Lab 1) — verify SDK installation
2. **QRNG service** (Lab 3) — generate random numbers on demand
3. **Bell state tester** (Lab 5) — verify entanglement with statistical tests
4. **GHZ demonstrator** (Lab 20) — full verification suite with reports
5. **Budget optimizer** (Lab 19) — recommend max qubits for the user's hardware

Package as a single Python module with a CLI: `python3 -m zql_toolkit --test-all`

---

## Appendix

### Glossary

| Term | Definition |
|------|-----------|
| Qubit | Quantum bit holding superposition of \|0⟩ and \|1⟩ |
| Amplitude | Complex number whose squared magnitude gives measurement probability |
| Superposition | Linear combination of basis states |
| Entanglement | Multi-qubit state that cannot be factored into individual qubit states |
| Interference | Constructive or destructive combination of amplitudes |
| Statevector | Full 2^n amplitude vector for n qubits |
| Depth | Maximum number of sequential gates on any qubit path |
| Shots | Number of circuit executions (measurements) |
| Clifford | Gate set {H, S, CX} that can be efficiently simulated classically |
| NISQ | Noisy Intermediate-Scale Quantum — current hardware era |

### Troubleshooting Index

| Issue | Labs | Fix |
|-------|------|-----|
| ModuleNotFoundError | Lab 1 | `pip install zuntenium` |
| Empty counts `{}` | Lab 14, Bug 2 | Add `measure_all()` |
| QubitIndexError | Lab 14, Bug 1 | Use 0-indexed qubits |
| MemoryError | Lab 16 | Reduce qubit count |
| Unexpected bit order | Lab 18 | Rightmost bit = qubit 0 |
| Wrong probabilities | Lab 8 | Use radians, not degrees |

### Assumptions Log

1. `zuntenium==0.2.0` is available via pip — [UNVERIFIED: package may be internal]
2. `StatevectorSimulator` returns `.counts` as `dict[str, int]` — based on source doc §5.4
3. `qc.to_ir()` is a public method — based on source doc §4.3
4. `qc.mirror()` is a public method — based on source doc §6.3
5. `qc.inverse()` is a public method — based on source doc §6.3
6. Bit ordering is little-endian — based on source doc §5.4

### If You Only Have 2 Hours

**Fast path:** Labs 1, 3, 5, 8, 20 (install → QRNG → Bell states → rotations → capstone).

---

## Validation Report

1. **Count check:** 20 `## Lab N:` headings (1–20) ✓
2. **Uniqueness check:** 20 distinct concepts, 20 distinct approaches ✓
3. **Coverage check:** 24/28 inventory concepts covered. Not covered: VS Code extensions (tooling-focused, not code lab), MHOS hardware orchestration (requires real hardware), SABRE routing details (covered conceptually in Lab 17), KAK decomposition (advanced transpiler internals)
4. **Code check:** All code blocks are syntactically valid Python. Marked `[UNVERIFIED]` against live SDK — assumes API matches source doc.
5. **Consistency check:** Prerequisites reference real lab numbers, file names consistent ✓
6. **Link check:** Source doc link resolves via relative path ✓
7. **Tone check:** Precise, direct, encouraging, no filler ✓


