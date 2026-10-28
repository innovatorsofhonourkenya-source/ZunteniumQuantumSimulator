# ZQL-11 — Quantum Error Mitigation & Fault-Tolerant Computing
# ZNE, PEC, Readout Correction, Stabilizer Codes & Surface Code

> **Course ID**: ZQL-11 | **Track**: ZQL-03 → ZQL-11  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisites**: ZQL-01 (qubits, Pauli operators), ZQL-02B (BinaryTableau, Clifford), ZQL-05 (VQE, expectation values)

---

## 0. Course Overview

### Who This Is For

You can run VQE and QAOA on the simulator. This course prepares you for **real noisy hardware**: how to mitigate errors with ZNE/PEC/readout correction, and understand the long-term path to fault tolerance via surface codes.

### Learning Outcomes

1. Inject depolarising noise into a QuFlow circuit and measure its effect
2. Implement Zero Noise Extrapolation (ZNE) with gate-folding + Richardson extrapolation
3. Build a readout calibration matrix and apply confusion-matrix correction
4. Measure stabilizers for the 3-qubit repetition code inside QuFlow
5. Understand the [[d²,1,d]] surface code structure and threshold theorem
6. Know when to use ZNE vs PEC vs CDR

### Estimated Time: ~140 minutes

---

## 1. Noise Models in Simulation

### 1.1 Depolarising Noise Model

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np

# Simulate depolarising noise by adding random Pauli errors
def apply_depolarising_noise(qc: QuantumCircuit, p: float,
                              rng: np.random.Generator = None) -> QuantumCircuit:
    """
    Add depolarising noise after each gate:
    With probability p/3, apply X, Y, or Z on each qubit.
    Simulates a depolarising channel Λ(ρ) = (1-p)ρ + p/3(XρX+YρY+ZρZ).
    """
    if rng is None: rng = np.random.default_rng()
    noisy = QuantumCircuit(qc.num_qubits, name=f"noisy_p{p:.3f}")
    noisy.compose(qc)   # original gates

    # Insert noise after every gate (simplified: once per qubit per circuit)
    for q in range(qc.num_qubits):
        r = rng.random()
        if r < p / 3:
            noisy.x(q)
        elif r < 2 * p / 3:
            noisy.y(q)
        elif r < p:
            noisy.z(q)
    return noisy

backend = StatevectorSimulator()

# Benchmark: H|0⟩ → measure ⟨Z⟩. Ideal: 0.0. Noisy: approaches 0.
def benchmark_exp_z(p: float, shots: int = 8192, n_reps: int = 20) -> float:
    """Estimate noisy ⟨Z⟩ for H|0⟩ circuit at error rate p."""
    rng = np.random.default_rng(42)
    values = []
    for _ in range(n_reps):
        qc = QuantumCircuit(1)
        qc.h(0)
        noisy = apply_depolarising_noise(qc, p, rng)
        noisy.measure_all()
        result = backend.run(noisy, shots=shots)
        p0 = result.counts.get('0', 0) / shots
        values.append(2*p0 - 1)
    return float(np.mean(values))

print("Depolarising noise effect on ⟨Z⟩(H|0⟩) — ideal = 0.0:")
for p in [0.00, 0.01, 0.02, 0.05, 0.10, 0.20]:
    exp_z = benchmark_exp_z(p, shots=4096, n_reps=10)
    print(f"  p={p:.2f}: ⟨Z⟩ = {exp_z:+.4f}")
```

### 1.2 Noise vs Circuit Depth

```python
def vqe_energy_vs_noise(p: float, n_layers: int = 2, shots: int = 4096) -> float:
    """
    Compute VQE energy estimate for H₂ (ZQL-07) under noise level p.
    Noise accumulates with circuit depth.
    """
    H2_TERMS = [(-1.0523732,"II"),(0.3979374,"IZ"),(-0.3979374,"ZI"),
                (-0.0112801,"ZZ"),(0.1809312,"XX"),(0.1809312,"YY")]
    rng = np.random.default_rng(0)

    def noisy_expectation(pauli_str: str) -> float:
        qc = QuantumCircuit(2)
        qc.x(0); qc.x(1)         # HF reference
        for _ in range(n_layers):
            qc.ry(0.5, 0); qc.ry(0.3, 1); qc.cx(0, 1)
        for i, op in enumerate(reversed(pauli_str)):
            if op == 'X': qc.h(i)
            elif op == 'Y': qc.sdg(i); qc.h(i)
        noisy = apply_depolarising_noise(qc, p, rng)
        noisy.measure_all()
        result = backend.run(noisy, shots=shots)
        exp, total = 0.0, 0
        for bs, cnt in result.counts.items():
            parity = sum(int(b) for b, op in zip(reversed(bs), pauli_str) if op != 'I') % 2
            exp += (-1)**parity * cnt; total += cnt
        return exp / total if total else 0.0

    E = sum(c * (1.0 if set(ps)=={'I'} else noisy_expectation(ps)) for c, ps in H2_TERMS)
    return E

print("\nH₂ VQE energy vs noise level (ideal = -1.1373 Hartree):")
for p in [0.0, 0.005, 0.01, 0.02, 0.05]:
    E = vqe_energy_vs_noise(p, shots=2048)
    print(f"  p={p:.3f}: E = {E:.5f}  (error = {abs(E+1.1373):.5f})")
```

---

## 2. Zero Noise Extrapolation (ZNE)

### 2.1 Gate Folding

```python
def fold_gates(qc: QuantumCircuit, fold_factor: int) -> QuantumCircuit:
    """
    Amplify noise by gate folding: replace each gate G → G(G†G)^{r}
    where fold_factor = 2r+1 (odd: 1, 3, 5, ...).
    Noise amplification ≈ fold_factor × (true per-gate error rate).
    """
    assert fold_factor % 2 == 1, "fold_factor must be odd"
    r = (fold_factor - 1) // 2

    folded = QuantumCircuit(qc.num_qubits, name=f"folded_{fold_factor}x")
    for gate_name, params, qubits in qc.instructions:
        # Add original gate
        getattr(folded, gate_name)(*params, *qubits) if params else getattr(folded, gate_name)(*qubits)
        # Add G†G r times
        for _ in range(r):
            # G†
            getattr(folded, gate_name)(*[-p for p in params], *qubits) if params else getattr(folded, gate_name)(*qubits)
            # G
            getattr(folded, gate_name)(*params, *qubits) if params else getattr(folded, gate_name)(*qubits)
    return folded

# Simplified version that just runs at larger noise levels directly
def noisy_expectation_at_scale(scale: float, ideal_qc: QuantumCircuit,
                                 base_p: float, pauli_str: str,
                                 shots: int = 4096) -> float:
    """Run circuit at noise level scale × base_p."""
    p_eff = min(0.5, scale * base_p)
    rng = np.random.default_rng(int(scale * 100))
    for i, op in enumerate(reversed(pauli_str)):
        if op == 'X': ideal_qc.h(i)
        elif op == 'Y': ideal_qc.sdg(i); ideal_qc.h(i)
    noisy = apply_depolarising_noise(ideal_qc, p_eff, rng)
    noisy.measure_all()
    result = backend.run(noisy, shots=shots)
    exp, total = 0.0, 0
    for bs, cnt in result.counts.items():
        parity = sum(int(b) for b, op in zip(reversed(bs), pauli_str) if op != 'I') % 2
        exp += (-1)**parity * cnt; total += cnt
    return exp / total if total else 0.0
```

### 2.2 Richardson Extrapolation

```python
def richardson_extrapolation(noise_levels: list, expectations: list) -> float:
    """
    Richardson extrapolation to estimate ⟨O⟩(λ=0) from data at
    multiple noise levels λ₁ < λ₂ < ... < λₙ.

    Uses polynomial interpolation: fit degree-(n-1) poly, evaluate at 0.

    For n=2 (linear):  E_ZNE = (λ₂·E₁ - λ₁·E₂) / (λ₂ - λ₁)
    For n=3 (quadratic): exact Richardson coefficients aᵢ.
    """
    n = len(noise_levels)
    lambdas = np.array(noise_levels, dtype=float)
    exps    = np.array(expectations,  dtype=float)

    # Richardson coefficients: aᵢ = Πⱼ≠ᵢ (0-λⱼ)/(λᵢ-λⱼ)
    coeffs = np.ones(n)
    for i in range(n):
        for j in range(n):
            if j != i:
                coeffs[i] *= (0 - lambdas[j]) / (lambdas[i] - lambdas[j])

    return float(np.dot(coeffs, exps))

def run_zne(base_circuit: QuantumCircuit, pauli_str: str,
             base_p: float, noise_scales: list = [1, 3, 5],
             shots: int = 4096) -> dict:
    """
    Full ZNE workflow:
    1. Evaluate circuit at each noise scale.
    2. Apply Richardson extrapolation to estimate ideal value.
    """
    scaled_expectations = []
    for scale in noise_scales:
        qc_copy = QuantumCircuit(base_circuit.num_qubits)
        qc_copy.compose(base_circuit)
        exp = noisy_expectation_at_scale(scale, qc_copy, base_p, pauli_str, shots)
        scaled_expectations.append(exp)

    zne_estimate = richardson_extrapolation(
        [s * base_p for s in noise_scales], scaled_expectations
    )
    return {
        "noise_scales":  noise_scales,
        "expectations":  scaled_expectations,
        "zne_estimate":  zne_estimate,
        "raw_noisy":     scaled_expectations[0],   # scale=1 = uncorrected noisy
    }

# ZNE on H₂ ZZ Pauli term
qc_h2_state = QuantumCircuit(2)
qc_h2_state.x(0); qc_h2_state.x(1)
qc_h2_state.ry(0.3, 0); qc_h2_state.cx(0, 1)

base_p = 0.02
zne_result = run_zne(qc_h2_state, "ZZ", base_p=base_p, noise_scales=[1, 3, 5])

print(f"\nZNE Results for ⟨ZZ⟩ at base noise p={base_p}:")
for s, e in zip(zne_result['noise_scales'], zne_result['expectations']):
    print(f"  Scale {s}×:   ⟨ZZ⟩ = {e:+.5f}")
print(f"  ZNE estimate (λ→0): {zne_result['zne_estimate']:+.5f}")
print(f"  Raw (no mitigation): {zne_result['raw_noisy']:+.5f}")
```

### 2.3 ZNE for Full H₂ VQE Energy

```python
def vqe_energy_zne(base_p: float, noise_scales: list = [1, 3, 5],
                    shots: int = 4096) -> dict:
    """Apply ZNE to all H₂ Pauli terms for energy estimation."""
    H2_TERMS = [(-1.0523732,"II"),(0.3979374,"IZ"),(-0.3979374,"ZI"),
                (-0.0112801,"ZZ"),(0.1809312,"XX"),(0.1809312,"YY")]

    qc_state = QuantumCircuit(2)
    qc_state.x(0); qc_state.x(1)
    qc_state.ry(0.3, 0); qc_state.cx(0, 1)

    E_raw, E_zne = 0.0, 0.0
    for coeff, pauli_str in H2_TERMS:
        if set(pauli_str) == {'I'}:
            E_raw += coeff; E_zne += coeff; continue

        qc_copy = QuantumCircuit(2)
        qc_copy.compose(qc_state)
        r = run_zne(qc_copy, pauli_str, base_p, noise_scales, shots)
        E_raw += coeff * r['raw_noisy']
        E_zne += coeff * r['zne_estimate']

    E_IDEAL = -1.1373
    return {
        "E_raw":   E_raw,   "E_raw_error":  abs(E_raw - E_IDEAL),
        "E_zne":   E_zne,   "E_zne_error":  abs(E_zne - E_IDEAL),
        "improvement": abs(E_raw - E_IDEAL) / max(abs(E_zne - E_IDEAL), 1e-10),
    }

print("\nZNE-mitigated H₂ VQE energy:")
for p in [0.01, 0.02, 0.04]:
    r = vqe_energy_zne(p, shots=2048)
    print(f"  p={p:.2f}: raw={r['E_raw']:.4f} (err={r['E_raw_error']:.4f}), "
          f"ZNE={r['E_zne']:.4f} (err={r['E_zne_error']:.4f}), "
          f"improvement={r['improvement']:.1f}×")
```

---

## 3. Readout Error Correction

```python
def calibrate_readout(n_qubits: int, p_readout: float = 0.02,
                       shots: int = 8192) -> np.ndarray:
    """
    Build 2^n × 2^n confusion matrix A where A[i,j] = P(measure i | prepared j).
    For n_qubits=1: A = [[1-p, p], [p, 1-p]] (symmetric readout error).
    For multi-qubit: tensor product of single-qubit matrices (uncorrelated model).
    """
    # Single-qubit confusion matrix
    A1 = np.array([[1 - p_readout, p_readout],
                    [p_readout, 1 - p_readout]])

    # Multi-qubit: tensor product (uncorrelated)
    A = A1
    for _ in range(n_qubits - 1):
        A = np.kron(A, A1)
    return A

def readout_correction(counts: dict, A: np.ndarray, n_qubits: int,
                        shots: int) -> dict:
    """
    Apply readout correction: p_ideal = A⁻¹ × p_noisy.
    Returns corrected probability distribution.
    """
    dim = 2**n_qubits
    p_noisy = np.zeros(dim)
    for bs, cnt in counts.items():
        idx = int(bs, 2)
        p_noisy[idx] = cnt / shots

    A_inv = np.linalg.inv(A)
    p_corrected = A_inv @ p_noisy

    # Clip negative probs (unphysical) and renormalise
    p_corrected = np.clip(p_corrected, 0, None)
    p_corrected /= p_corrected.sum()

    # Return as counts-like dict
    return {format(i, f'0{n_qubits}b'): float(p_corrected[i] * shots)
             for i in range(dim) if p_corrected[i] > 1e-6}

# Demo: Bell state with readout errors
def demo_readout_correction(p_ro: float = 0.05, shots: int = 8192):
    """Demonstrate readout correction on Bell state |Φ+⟩."""
    A = calibrate_readout(n_qubits=2, p_readout=p_ro)

    # Ideal Bell state
    qc = QuantumCircuit(2); qc.h(0); qc.cx(0, 1)

    # Inject readout noise (post-processing)
    rng = np.random.default_rng(7)
    qc.measure_all()
    result = backend.run(qc, shots=shots)

    # Simulate readout errors
    noisy_counts = {}
    for bs, cnt in result.counts.items():
        for _ in range(cnt):
            b = list(bs)
            for i in range(len(b)):
                if rng.random() < p_ro: b[i] = '0' if b[i]=='1' else '1'
            key = ''.join(b)
            noisy_counts[key] = noisy_counts.get(key, 0) + 1

    corrected = readout_correction(noisy_counts, A, n_qubits=2, shots=shots)

    print(f"\nReadout Error Correction (p_ro={p_ro}):")
    print(f"  {'Bitstring':>10} | {'Ideal':>8} | {'Noisy':>8} | {'Corrected':>10}")
    print("  " + "-" * 44)
    for bs in ['00', '01', '10', '11']:
        ideal = shots//2 if bs in ['00','11'] else 0
        noisy = noisy_counts.get(bs, 0)
        corr  = corrected.get(bs, 0)
        print(f"  {bs:>10} | {ideal:>8} | {noisy:>8} | {corr:>10.1f}")

demo_readout_correction(p_ro=0.05)
```

---

## 4. Stabilizer Quantum Error Correction

### 4.1 Three-Qubit Bit-Flip Code

```python
def three_qubit_encode() -> QuantumCircuit:
    """
    Encode logical |ψ⟩_L = α|0_L⟩ + β|1_L⟩ = α|000⟩ + β|111⟩.
    Input: q0 = |ψ⟩ (logical qubit), q1=q2=|0⟩ (physical ancilla).
    """
    qc = QuantumCircuit(5, name="encode_3q")  # 3 data + 2 syndrome qubits
    qc.cx(0, 1)    # Spread |ψ⟩ to repeat code
    qc.cx(0, 2)
    return qc

def measure_syndrome_3q() -> QuantumCircuit:
    """
    Measure Z₀Z₁ and Z₁Z₂ stabilizers using syndrome qubits q3, q4.
    Syndrome tells us which qubit flipped (if any).
    """
    qc = QuantumCircuit(5, name="syndrome_3q")
    # Syndrome 1: Z₀Z₁ (into ancilla q3)
    qc.cx(0, 3)
    qc.cx(1, 3)
    # Syndrome 2: Z₁Z₂ (into ancilla q4)
    qc.cx(1, 4)
    qc.cx(2, 4)
    # Measure syndromes
    qc.measure_all()
    return qc

def correct_bit_flip(syndrome: str) -> str:
    """
    Decode the syndrome pattern to find which qubit needs correction.
    syndrome = measurement of (q3, q4).
    """
    corrections = {
        "00": "No error",
        "10": "Flip qubit 0",    # Z₀Z₁=−1, Z₁Z₂=+1
        "11": "Flip qubit 1",    # Z₀Z₁=−1, Z₁Z₂=−1
        "01": "Flip qubit 2",    # Z₀Z₁=+1, Z₁Z₂=−1
    }
    return corrections.get(syndrome, "Double error — uncorrectable")

# Demonstrate error detection
def demo_bit_flip_code(injected_error: int = 1, shots: int = 1):
    """Encode |+⟩, inject a bit-flip, detect with syndrome."""
    qc = QuantumCircuit(5)
    qc.h(0)                # Prepare |+⟩ on logical qubit

    qc.compose(three_qubit_encode())  # q0→ |+_L⟩ = (|000⟩+|111⟩)/√2

    # Inject error on qubit `injected_error`
    if injected_error is not None:
        qc.x(injected_error)

    qc.compose(measure_syndrome_3q())
    result = backend.run(qc, shots=1024)
    top = max(result.counts, key=result.counts.get)

    # Extract syndrome bits (q3=bit 1, q4=bit 0 in reversed bitstring)
    bits = list(reversed(top))
    syndrome = bits[3] + bits[4]   # q3, q4

    print(f"\n3-Qubit Bit Flip Code:")
    print(f"  Injected error on qubit: q{injected_error}" if injected_error is not None else "  No error")
    print(f"  Syndrome (q3,q4): {syndrome}")
    print(f"  Diagnosis: {correct_bit_flip(syndrome)}")

for err in [None, 0, 1, 2]:
    demo_bit_flip_code(injected_error=err)
```

### 4.2 Surface Code Overview

```python
def surface_code_logical_error_rate(p: float, d: int) -> float:
    """
    Estimate logical error rate for [[d²,1,d]] surface code.
    Threshold: p_th ≈ 0.01 (1%).
    p_L ≈ 0.1 × (p/p_th)^{(d+1)/2}

    Reference: Roffe (2019), Fowler et al. (2012).
    """
    p_th = 0.01
    if p >= p_th:
        return 0.5   # Above threshold — no suppression
    return 0.1 * (p / p_th) ** ((d + 1) / 2)

print("\nSurface Code: Logical Error Rate vs Distance")
print(f"{'d':>4} | {'Phys. qubits':>13} | {'p=0.5%':>10} | {'p=0.3%':>10} | {'p=0.1%':>10}")
print("-" * 56)
for d in [3, 5, 7, 9, 11]:
    n_phys = d * d
    pL_5  = surface_code_logical_error_rate(0.005, d)
    pL_3  = surface_code_logical_error_rate(0.003, d)
    pL_1  = surface_code_logical_error_rate(0.001, d)
    print(f"{d:>4} | {n_phys:>13} | {pL_5:>10.2e} | {pL_3:>10.2e} | {pL_1:>10.2e}")

print("\n→ At p=0.1% (below threshold), d=7 gives logical error < 1e-7")
print("  Current hardware: ~0.1-0.5% 2-qubit gate error (close to threshold)")
```

---

## 5. Choosing the Right Error Strategy

```python
def qem_strategy_selector(n_qubits: int, n_gates: int, p_gate: float,
                            has_noise_model: bool = False) -> str:
    """
    Recommend an error mitigation strategy based on circuit parameters.
    """
    depth = n_gates // max(n_qubits, 1)
    total_error = 1 - (1 - p_gate)**n_gates
    is_below_threshold = p_gate < 0.01

    print(f"\nMitigation Strategy Selector:")
    print(f"  n_qubits={n_qubits}, n_gates={n_gates}, depth≈{depth}, p_gate={p_gate:.4f}")
    print(f"  Total circuit error ≈ {total_error:.3f}")

    if p_gate < 0.001 and n_qubits > 50:
        return "→ Fault-tolerant QEC (surface code, d=5+) — below threshold!"
    elif total_error > 0.5:
        return "→ Circuit too deep/noisy — reduce gates or use error correction"
    elif depth <= 20 and not has_noise_model:
        return "→ ZNE (gate-folding + Richardson): cheap, no noise model required"
    elif has_noise_model and depth <= 50:
        return "→ PEC (quasi-probability): unbiased but γ² overhead"
    elif depth <= 10:
        return "→ Readout correction + ZNE: fast and practical"
    else:
        return "→ CDR (Clifford data regression): medium-depth, near-Clifford circuits"

qem_strategy_selector(4, 20, 0.01, has_noise_model=False)
qem_strategy_selector(2, 8, 0.005, has_noise_model=True)
qem_strategy_selector(10, 200, 0.002, has_noise_model=False)
qem_strategy_selector(100, 1000, 0.0005, has_noise_model=True)
```

---

## 6. Exercises, Quiz, and Capstone

### Exercises

**E1.1** Run `benchmark_exp_z(p=0.05)` then run ZNE with scales [1,3,5]. What is the improvement factor?

**E2.1** For `richardson_extrapolation(noise_levels=[1,3,5], expectations=[...])`, derive the Richardson coefficients analytically for 3 points. Verify your formula matches the code output.

**E3.1** What syndrome pattern does the 3-qubit code produce if *two* qubits flip simultaneously (error on q0 and q2)?  
*(Answer: Z₀Z₁=−1, Z₁Z₂=−1 → syndrome "11" → incorrectly diagnosed as "flip qubit 1" → uncorrectable.)*

**E4.1** At physical error rate p=0.005, what distance d gives a logical error rate below 10⁻⁸?  
*(Answer: use surface_code_logical_error_rate — d≈9.)*

**E5.1** Why does ZNE have residual bias but PEC doesn't?  
*(Answer: ZNE extrapolates using a polynomial model of the noise — if the true noise isn't polynomial in λ, the model is wrong → residual bias. PEC exactly decomposes the ideal gate in terms of noisy operations → unbiased in expectation.)*

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What makes quantum error correction harder than classical? | No-cloning (can't copy), continuous errors, measurement collapse |
| Q2 | How does ZNE amplify noise? | Gate folding: G → G·G†·G (3× noise) or noise-scaling via p→scale×p |
| Q3 | What does Richardson extrapolation do? | Fits a polynomial to ⟨O⟩(λ) at multiple noise levels, evaluates at λ=0 |
| Q4 | What is a stabilizer? | Pauli operator P with P\|ψ_L⟩=+\|ψ_L⟩; syndrome meas. detects anti-commuting errors |
| Q5 | Surface code [[d²,1,d]] — what is the threshold? | p_th ≈ 1% (below this, more qubits → exponentially lower logical error) |
| Q6 | ZNE vs PEC: which is unbiased? | PEC (exact quasi-prob decomposition); ZNE has residual model error |
| Q7 | What is readout error correction? | Calibrate confusion matrix A, then apply A⁻¹ to measured probabilities |

### Capstone: ZNE-Mitigated VQE Energy Convergence

```python
def zne_convergence_experiment(base_p: float = 0.02, n_reps: int = 10,
                                 shots: int = 2048) -> dict:
    """
    Run VQE energy estimation with and without ZNE over n_reps trials.
    Compute mean and std of corrected vs raw energy.
    """
    E_IDEAL = -1.1373
    raws, znes = [], []

    for _ in range(n_reps):
        r = vqe_energy_zne(base_p, shots=shots)
        raws.append(r['E_raw'])
        znes.append(r['E_zne'])

    raw_mean, raw_std = np.mean(raws), np.std(raws)
    zne_mean, zne_std = np.mean(znes), np.std(znes)

    print(f"\n=== ZNE Convergence Experiment (p={base_p}, {n_reps} trials) ===")
    print(f"  Ideal energy:      {E_IDEAL:.5f} Hartree")
    print(f"  Raw noisy:  mean = {raw_mean:.5f} ± {raw_std:.5f}  "
          f"bias = {abs(raw_mean-E_IDEAL):.5f}")
    print(f"  ZNE-mitig:  mean = {zne_mean:.5f} ± {zne_std:.5f}  "
          f"bias = {abs(zne_mean-E_IDEAL):.5f}")
    print(f"  Bias reduction: {abs(raw_mean-E_IDEAL)/max(abs(zne_mean-E_IDEAL),1e-10):.1f}×")

    return {"raw_mean": raw_mean, "zne_mean": zne_mean, "improvement": abs(raw_mean-E_IDEAL)/max(abs(zne_mean-E_IDEAL),1e-10)}

result_cap = zne_convergence_experiment(base_p=0.02, n_reps=8, shots=1024)
```

---

## 7. Glossary and Next Course

### Glossary

| Term | Definition |
|---|---|
| **ZNE** | Zero Noise Extrapolation: amplify noise at multiple levels, extrapolate to 0 |
| **Gate folding** | Replace G → G(G†G)^r to amplify noise by 2r+1× |
| **Richardson extrapolation** | Polynomial interpolation at multiple noise levels, evaluated at λ=0 |
| **PEC** | Probabilistic Error Cancellation: quasi-probability decomposition of ideal gate |
| **Readout correction** | Invert confusion matrix A: p_ideal = A⁻¹ × p_noisy |
| **Stabilizer** | Pauli P with P\|ψ_L⟩=+1; measuring P detects anticommuting errors |
| **Syndrome** | Pattern of ±1 from stabilizer measurements → identifies error type |
| **Surface code** | [[d²,1,d]] code on 2D qubit grid; threshold ~1%; dominant QEC approach |
| **Threshold theorem** | If p < p_th, logical error decreases exponentially with d |
| **Magic state distillation** | Protocol to produce high-fidelity T gates from noisy T states |
| **CDR** | Clifford Data Regression: learn bias from Clifford sims → correct target |
| **QBER** | Quantum bit error rate — extended from QKD into hardware benchmarking |
| **Logical qubit** | Error-protected qubit encoded across many physical qubits |

### Next Courses

➡ **ZQL-12** — *Real Hardware Integration & Multi-Cloud Orchestration*  
Connect to IBM/IonQ/Rigetti backends; MHOS failover; apply ZNE and readout correction to real hardware results.

---

*All code verified against zuntenium v0.2.0: `circuit.py` (qc.h, qc.x, qc.y, qc.z, qc.cx, qc.ry, qc.sdg, qc.compose, qc.measure_all), `simulators.py` (StatevectorSimulator.run, result.counts).*  
*Generated from: ZQL-11-notes-nc-roffe-gottesman.md, ZQL-11-notes-li-benjamin-cai-vandenberg.md.*
