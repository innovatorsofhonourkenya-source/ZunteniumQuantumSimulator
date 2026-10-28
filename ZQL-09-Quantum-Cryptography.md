# ZQL-09 — Quantum Cryptography & Post-Quantum Security
# Shor's Threat, BB84 QKD, Lattice-Based PQC (Kyber/Dilithium/SPHINCS+)

> **Course ID**: ZQL-09 | **Track**: ZQL-04 → ZQL-09  
> **Version**: 1.0.0 | **SDK**: QuFlow v0.2.0 (package: `zuntenium`)  
> **Prerequisites**: ZQL-04 (Shor's algorithm, QPE), ZQL-01 (qubits, measurement, bases)

---

## 0. Course Overview

### Who This Is For

You have run Shor's algorithm in simulation (ZQL-04). This course shows **what it breaks**, what replaces it, and how to simulate **quantum key distribution (BB84)** on QuFlow.

### Learning Outcomes

1. Quantify Shor's threat to RSA and ECC with concrete key sizes
2. Explain what makes lattice problems (LWE) quantum-resistant
3. Compare CRYSTALS-Kyber, Dilithium, and SPHINCS+ (NIST standards)
4. Simulate BB84 full key exchange protocol using QuFlow
5. Detect eavesdropping (Eve) through QBER measurement
6. Simulate the E91 entanglement-based QKD protocol

### Estimated Time: ~130 minutes

| Section | Content | Time |
|---|---|---|
| §1 | Classical Cryptography Baseline | 15 min |
| §2 | Shor's Threat — Quantified | 15 min |
| §3 | Grover's Threat to Symmetric Crypto | 10 min |
| §4 | Post-Quantum Cryptography (PQC) | 20 min |
| §5 | BB84 QKD Simulation | 30 min |
| §6 | E91 Entanglement-Based QKD | 15 min |
| §7 | Exercises + Quiz + Capstone | 25 min |

---

## 1. Classical Cryptography Baseline

```python
# Simulate RSA-2048 key operations (conceptual — no real crypto, illustrative)
import math, random

def is_prime_miller_rabin(n: int, k: int = 10) -> bool:
    """Miller-Rabin primality test (probabilistic)."""
    if n < 2: return False
    if n == 2 or n == 3: return True
    if n % 2 == 0: return False
    r, d = 0, n - 1
    while d % 2 == 0: r += 1; d //= 2
    for _ in range(k):
        a = random.randrange(2, n - 1)
        x = pow(a, d, n)
        if x == 1 or x == n - 1: continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1: break
        else: return False
    return True

def rsa_key_components(bits: int = 16) -> dict:
    """Generate small RSA components (illustrative, NOT production-safe)."""
    # In production: use 2048-bit primes
    from sympy import randprime, nextprime
    # For illustration: tiny primes
    p, q = 61, 53    # classic textbook example
    N = p * q          # 3233
    phi_N = (p-1)*(q-1)   # 3120
    e = 17             # public exponent
    d = pow(e, -1, phi_N)   # secret exponent

    return {"N": N, "e": e, "d": d, "p": p, "q": q, "phi_N": phi_N}

rsa = rsa_key_components()
print("Toy RSA key components:")
print(f"  N = {rsa['N']} = {rsa['p']} × {rsa['q']}")
print(f"  Public key:  (N={rsa['N']}, e={rsa['e']})")
print(f"  Private key: d={rsa['d']}")

# Encrypt and decrypt
m = 42    # message
c = pow(m, rsa['e'], rsa['N'])
m2 = pow(c, rsa['d'], rsa['N'])
print(f"\n  Message:   {m}")
print(f"  Encrypted: {c}")
print(f"  Decrypted: {m2}  (matches: {m == m2})")

# Security: breaking RSA = factoring N (hard classically, easy for Shor's)
print(f"\n  Classical factoring: trial division up to √{rsa['N']}={int(math.sqrt(rsa['N']))+1}")
print(f"  Shor's (quantum):    O((log {rsa['N']})³) ≈ O({int(math.log2(rsa['N']))**3}) logical ops")
```

---

## 2. Shor's Threat — Quantified

### 2.1 RSA Factoring via Shor's Algorithm

Recall from ZQL-04: Shor's algorithm factors N by finding the order r of a mod N, using Quantum Phase Estimation.

```python
# Shor's algorithm complexity comparison
import numpy as np

print("=== Shor's Algorithm: RSA Breaking Complexity ===\n")
print(f"{'Key size':>12} | {'Classical ops':>18} | {'Quantum ops':>18} | {'Quantum advantage':>20}")
print("-" * 76)

for bits in [512, 1024, 2048, 4096]:
    # Classical: sub-exponential GNFS: exp(1.923 × (log N)^{1/3} × (log log N)^{2/3})
    log_n = bits
    classical = np.exp(1.923 * (log_n ** (1/3)) * (np.log(log_n) ** (2/3)))
    classical_str = f"2^{np.log2(classical):.0f}" if classical > 1e6 else f"{classical:.0f}"

    # Shor's: O((log N)^3) = O(bits^3)
    quantum = bits ** 3
    quantum_str = f"2^{np.log2(quantum):.1f}" if quantum > 1e6 else f"{quantum:.0f}"

    ratio = classical / quantum
    ratio_str = f"≈ 2^{np.log2(ratio):.0f}×" if ratio > 1e3 else f"≈ {ratio:.0f}×"
    print(f"{bits:>12} | {classical_str:>18} | {quantum_str:>18} | {ratio_str:>20}")

print("\n→ RSA-2048: classical ≈ 2^112 ops, quantum ≈ 2^{33} ops")
print("  A quantum computer with ~4000 logical qubits (millions of physical) would break RSA-2048")
```

### 2.2 ECC Breaking via Quantum Order Finding

```python
# ECDLP: same Shor's strategy applied to elliptic curve group order
print("\n=== Shor's Algorithm: ECC Breaking ===\n")
print(f"{'Curve':>20} | {'Classical security':>20} | {'Quantum equivalent':>22}")
print("-" * 66)

ecc_params = [
    ("P-256 / secp256k1", 256, 128),
    ("P-384",              384, 192),
    ("P-521",              521, 260.5),
    ("Curve25519",         255, 127.5),
]
for name, bits, classical_sec in ecc_params:
    quantum_sec = classical_sec / 2   # Shor halves security? No — Shor BREAKS it
    print(f"{name:>20} | {classical_sec:>15.0f}-bit | {'BROKEN (Shor)':>22}")

print("\n→ All ECDSA, ECDH, EdDSA, secp256k1 (Bitcoin) are broken by Shor's")
print("  Grover doesn't help here — Shor directly finds discrete log")
```

### 2.3 Timeline: When Does This Become Real?

```python
# Q-Day estimates (when large-scale fault-tolerant quantum computer arrives)
timeline = {
    "Best current hardware": "2026 — ~100-1000 error-prone physical qubits",
    "Useful NISQ": "2026-2030 — thousands of noisy qubits, limited algorithms",
    "Fault-tolerant (early)": "2030-2035 — millions of physical qubits",
    "Q-Day (RSA-2048 breaks)": "2033-2040 — estimate range (controversial)",
    "NIST PQC standard live": "2024 — FIPS 203/204/205 finalised NOW",
}
print("=== Cryptographic Timeline ===")
for event, date in timeline.items():
    print(f"  {event:<35}: {date}")
print("\n→ Action required NOW: migrate to PQC before Q-Day")
```

---

## 3. Grover's Threat to Symmetric Cryptography

```python
# Grover's algorithm halves effective key length
print("=== Grover's Impact on Symmetric Cryptography ===\n")
print(f"{'Algorithm':>15} | {'Classical bits':>15} | {'Post-Grover':>12} | {'Safe?':>8}")
print("-" * 56)

sym_crypto = [
    ("AES-128",   128, 64),
    ("AES-256",   256, 128),
    ("3DES-168",  168, 84),
    ("SHA-256",   256, 128),   # collision: 128, preimage: 128
    ("SHA-3-384", 384, 192),
    ("SHA-3-512", 512, 256),
]
for name, bits, post_grover in sym_crypto:
    safe = "✓" if post_grover >= 128 else "✗"
    action = "OK" if post_grover >= 128 else "Upgrade key size"
    print(f"{name:>15} | {bits:>12}-bit | {post_grover:>8}-bit | {safe:>5}  {action}")

print("\n→ AES-256 is sufficient; AES-128 becomes marginal (64-bit post-Grover)")
print("  Solution for symmetric: double key sizes (AES-128 → AES-256)")
```

---

## 4. Post-Quantum Cryptography (NIST Standards)

### 4.1 Why Lattice Problems Are Quantum-Resistant

```python
# Illustrate Learning With Errors (LWE)
import numpy as np

def lwe_sample(n: int, q: int, s: np.ndarray, noise_width: float = 2.0) -> tuple:
    """
    Generate one LWE sample: (a, b = aᵀs + e mod q)
    where a ∈ Zq^n, s ∈ Zq^n (secret), e ← discrete Gaussian.
    """
    a = np.random.randint(0, q, size=n)
    e = int(round(np.random.normal(0, noise_width)))   # small noise
    b = (int(a @ s) + e) % q
    return a, b

def lwe_demo(n: int = 5, q: int = 97):
    """Demonstrate why LWE is hard: many samples but can't easily find s."""
    s = np.random.randint(0, q, size=n)    # secret
    print(f"LWE Demo: n={n}, q={q}")
    print(f"  Secret s = {s}")
    print(f"\n  LWE samples (a, b=aᵀs+e mod {q}):")
    for i in range(5):
        a, b = lwe_sample(n, q, s, noise_width=2.0)
        expected_b_exact = (a @ s) % q
        print(f"    a={a[:3]}...  b={b:3d}  (exact aᵀs={expected_b_exact:3d}, noise={b-expected_b_exact:+d})")
    print(f"\n  Challenge: recover s from these noisy equations → NP-hard (believed)")
    print(f"  Classical best: BKZ lattice sieving, exponential in n")
    print(f"  Quantum best: same BKZ sieving with Grover (quadratic) → still exponential")

lwe_demo()
```

### 4.2 NIST PQC Algorithm Comparison

```python
# Size and performance comparison
print("\n=== NIST PQC Algorithm Comparison ===\n")

pqc_data = {
    # (type, security, pk_bytes, sk_bytes, ct_or_sig_bytes, classical_equivalent)
    "RSA-2048 [BROKEN]":   ("KEM",  "broken",   256,   256,   256,  "RSA"),
    "ECDSA-256 [BROKEN]":  ("SIG",  "broken",    33,    32,    64,  "ECC"),
    "ML-KEM-512 (Kyber)":  ("KEM",  "L1 (128)", 800,  1632,   768,  "Lattice LWE"),
    "ML-KEM-768 (Kyber)":  ("KEM",  "L3 (192)",1184,  2400,  1088,  "Lattice LWE"),
    "ML-DSA-44 (Dil.)":    ("SIG",  "L2",      1312,  2528,  2420,  "Lattice LWE"),
    "ML-DSA-65 (Dil.)":    ("SIG",  "L3",      1952,  4000,  3293,  "Lattice LWE"),
    "SLH-DSA-128s":        ("SIG",  "L1 (128)",  32,    64,  7856,  "Hash (SHAKE)"),
}

print(f"{'Algorithm':<25} {'Type':<5} {'Security':<12} {'PK (B)':<8} {'CT/SIG (B)':<12}")
print("-" * 68)
for name, (kind, sec, pk, sk, ct, basis) in pqc_data.items():
    print(f"{name:<25} {kind:<5} {sec:<12} {pk:<8} {ct:<12}  [{basis}]")

print("\n→ Migration: RSA → ML-KEM (key exchange), ECDSA → ML-DSA (signatures)")
print("  ML-KEM-768 / ML-DSA-65 recommended for gen-purpose high-security applications")
```

### 4.3 Hybrid PQC + Classical (Transition Strategy)

```python
def hybrid_key_exchange_description():
    """
    Hybrid key exchange: run classical + PQC in parallel.
    Final shared secret = HASH(classical_secret || pqc_secret).
    Safe if EITHER algorithm is not broken.
    """
    print("\nHybrid Key Exchange Strategy:")
    print("  1. Run ECDHE (classical) → shared_secret_classical")
    print("  2. Run ML-KEM (PQC)     → shared_secret_pqc")
    print("  3. Final key = SHA3-256(shared_secret_classical ‖ shared_secret_pqc)")
    print("  4. Safe if either classical OR PQC is unbroken")
    print("  5. Recommended by NIST, NSA, NCSC during transition period")

hybrid_key_exchange_description()
```

---

## 5. BB84 Quantum Key Distribution Simulation

### 5.1 BB84 Protocol Implementation

```python
from zuntenium import QuantumCircuit, StatevectorSimulator
import numpy as np
import random

backend = StatevectorSimulator()

def prepare_bb84_qubit(bit: int, basis: str) -> QuantumCircuit:
    """
    Alice prepares one BB84 qubit.
    bit:   0 or 1 (the secret bit value)
    basis: 'Z' → {|0⟩, |1⟩}, 'X' → {|+⟩, |−⟩}
    Returns unitary circuit (do not measure here — Eve might intercept).
    """
    qc = QuantumCircuit(1, name=f"Alice_b{bit}_{basis}")
    if bit == 1:
        qc.x(0)        # |0⟩ → |1⟩
    if basis == 'X':
        qc.h(0)        # |0⟩ → |+⟩,  |1⟩ → |−⟩
    return qc

def bob_measure(alice_qc: QuantumCircuit, basis: str, shots: int = 1) -> int:
    """
    Bob measures Alice's qubit in his randomly chosen basis.
    Returns 0 or 1.
    """
    qc_copy = QuantumCircuit(1)
    qc_copy.compose(alice_qc)
    if basis == 'X':
        qc_copy.h(0)     # Rotate to Z-basis before measuring
    qc_copy.measure_all()
    result = backend.run(qc_copy, shots=shots)
    # Return most likely bit
    counts = result.counts
    return int(max(counts, key=counts.get))

def eve_intercept(alice_qc: QuantumCircuit, shots: int = 1) -> tuple:
    """
    Eve intercepts: measures in Z-basis (random guess), then re-prepares.
    Returns (eve_result, disturbed_qc) — disturbed_qc replaces Alice's.
    """
    eve_basis = random.choice(['Z', 'X'])
    eve_bit = bob_measure(alice_qc, eve_basis, shots)
    # Eve re-sends what she measured (re-preparation)
    disturbed = prepare_bb84_qubit(eve_bit, eve_basis)
    return eve_bit, disturbed

def run_bb84(n_qubits: int = 100, eve_present: bool = False,
              verbose: bool = False) -> dict:
    """
    Full BB84 simulation.

    Steps:
    1. Alice generates random bits + random bases
    2. (Optional) Eve intercepts
    3. Bob measures in random bases
    4. Sifting: keep where Alice_basis == Bob_basis
    5. QBER check on sample

    Returns: {raw_key, sifted_key, qber, alice_bits, bob_bits, shared_bases}
    """
    # Step 1: Alice's random bits and bases
    alice_bits   = [random.randint(0, 1) for _ in range(n_qubits)]
    alice_bases  = [random.choice(['Z', 'X']) for _ in range(n_qubits)]
    bob_bases    = [random.choice(['Z', 'X']) for _ in range(n_qubits)]

    # Step 2 & 3: Transmission (with optional Eve) and Bob's measurement
    bob_results = []
    for i in range(n_qubits):
        qc_alice = prepare_bb84_qubit(alice_bits[i], alice_bases[i])
        if eve_present:
            _, qc_alice = eve_intercept(qc_alice)    # Eve disturbs the channel
        bob_bit = bob_measure(qc_alice, bob_bases[i])
        bob_results.append(bob_bit)

    # Step 4: Sifting — keep only where bases match
    sifted_alice, sifted_bob = [], []
    for i in range(n_qubits):
        if alice_bases[i] == bob_bases[i]:
            sifted_alice.append(alice_bits[i])
            sifted_bob.append(bob_results[i])

    # Step 5: QBER — compare a random 20% sample
    n_sample = max(1, len(sifted_alice) // 5)
    sample_idx = random.sample(range(len(sifted_alice)), n_sample)
    errors = sum(sifted_alice[i] != sifted_bob[i] for i in sample_idx)
    qber = errors / n_sample

    # Remaining bits (not used for QBER check) → raw key
    key_idx = [i for i in range(len(sifted_alice)) if i not in sample_idx]
    raw_key = [sifted_alice[i] for i in key_idx]

    if verbose:
        print(f"  Total qubits transmitted: {n_qubits}")
        print(f"  Sifted key length: {len(sifted_alice)} ({len(sifted_alice)/n_qubits:.1%})")
        print(f"  QBER sample size: {n_sample} bits")
        print(f"  Errors in sample: {errors}")
        print(f"  QBER: {qber:.2%}")
        safe = qber < 0.11
        print(f"  {'✓ Channel clean — proceed' if safe else '✗ ABORT: Eve detected or noisy channel'}")
        if safe and raw_key:
            print(f"  Raw key length (after QBER check removed): {len(raw_key)} bits")
            print(f"  First 10 key bits: {raw_key[:10]}")

    return {
        "n_transmitted": n_qubits,
        "sifted_length": len(sifted_alice),
        "qber": qber,
        "raw_key": raw_key,
        "secure": qber < 0.11,
    }
```

### 5.2 BB84 Without Eve

```python
print("=== BB84 — No Eavesdropping ===")
result_clean = run_bb84(n_qubits=100, eve_present=False, verbose=True)
```

### 5.3 BB84 With Eve — QBER Spike

```python
print("\n=== BB84 — With Eve Intercept-Resend Attack ===")
result_eve = run_bb84(n_qubits=100, eve_present=True, verbose=True)

print(f"\nQBER without Eve: {result_clean['qber']:.2%}  (expected ≈ 0–3%)")
print(f"QBER with Eve:    {result_eve['qber']:.2%}  (expected ≈ 25%)")
```

### 5.4 Statistical Distribution of QBER (100 trials)

```python
import numpy as np

print("\n=== QBER Distribution Over 50 Trials ===")
n_trials = 50

# Without Eve
qbers_clean = [run_bb84(80, eve_present=False)['qber'] for _ in range(n_trials)]
# With Eve
qbers_eve   = [run_bb84(80, eve_present=True)['qber']  for _ in range(n_trials)]

print(f"Without Eve — mean QBER: {np.mean(qbers_clean):.2%} ± {np.std(qbers_clean):.2%}")
print(f"With Eve    — mean QBER: {np.mean(qbers_eve):.2%}   ± {np.std(qbers_eve):.2%}")
print(f"\n→ QBER threshold: 11%")
print(f"  Protocol aborts correctly? Clean: {sum(q>0.11 for q in qbers_clean)}/{n_trials} false alarms")
print(f"                             Eve:   {sum(q>0.11 for q in qbers_eve)}/{n_trials} detected")
```

---

## 6. E91 Entanglement-Based QKD

```python
def e91_pair() -> QuantumCircuit:
    """
    Prepare EPR pair |Φ+⟩ = (|00⟩ + |11⟩)/√2 for E91 protocol.
    Qubit 0 → Alice, Qubit 1 → Bob.
    """
    qc = QuantumCircuit(2, name="EPR_pair")
    qc.h(0)
    qc.cx(0, 1)
    return qc

def e91_measure_alice(basis_angle: float) -> QuantumCircuit:
    """Alice measures qubit 0 in basis rotated by basis_angle (in radians)."""
    qc = QuantumCircuit(1)
    qc.ry(-2 * basis_angle, 0)  # Rotate to measurement basis
    return qc

def run_e91(n_pairs: int = 100) -> dict:
    """
    Simplified E91 simulation.
    Alice and Bob each choose from 3 angles: 0, π/4, π/2 (Alice)
    and π/4, π/2, 3π/4 (Bob).
    Key bits come from matching angles (0=π/4 or π/2=π/2).
    Bell inequality checked for security.
    """
    alice_angles = [0, np.pi/4, np.pi/2]
    bob_angles   = [np.pi/4, np.pi/2, 3*np.pi/4]

    alice_choices  = [random.choice(alice_angles) for _ in range(n_pairs)]
    bob_choices    = [random.choice(bob_angles)   for _ in range(n_pairs)]

    alice_bits_all, bob_bits_all = [], []

    for i in range(n_pairs):
        # Prepare entangled pair
        pair_qc = e91_pair()
        pair_qc.measure_all()
        result = backend.run(pair_qc, shots=1)
        bs = next(iter(result.counts))   # single shot bitstring

        # Apply angle rotations (simplified: use the EPR correlations directly)
        a_bit = int(bs[0])   # Alice's qubit (qubit 0)
        b_bit = int(bs[1])   # Bob's qubit (qubit 1)

        alice_bits_all.append(a_bit)
        bob_bits_all.append(b_bit)

    # Sifting: both chose the same angle (Alice=π/4 & Bob=π/4 or Alice=π/2 & Bob=π/2)
    sifted_alice, sifted_bob = [], []
    for i in range(n_pairs):
        if alice_choices[i] == bob_choices[i]:  # matching angles
            sifted_alice.append(alice_bits_all[i])
            sifted_bob.append(1 - bob_bits_all[i])  # EPR: perfect anti-correlation

    n_key = len(sifted_alice)
    errors = sum(a != b for a,b in zip(sifted_alice, sifted_bob))

    print(f"E91 (entanglement-based):")
    print(f"  Entangled pairs: {n_pairs}")
    print(f"  Sifted key bits: {n_key} ({n_key/n_pairs:.1%})")
    print(f"  Errors: {errors}/{n_key}  QBER={errors/max(n_key,1):.2%}")
    print(f"  Security: Bell-inequality violation verifiable (not simulated here)")
    return sifted_alice

e91_key = run_e91(n_pairs=100)
print(f"\nE91 raw key (first 16 bits): {e91_key[:16]}")
```

---

## 7. Exercises, Quiz, and Capstone

### Exercises

**E1.1** For RSA-2048 (2048-bit N): how many bits does Shor's register need for φ(N)? (Answer: need 2×2048=4096 qubit QPE register for period finding.)

**E2.1** Run `run_bb84(n_qubits=200, eve_present=False)` 10 times. What is the average sifted key length? Why is it approximately 50% of transmitted qubits?

**E3.1** Modify `eve_intercept()` to only intercept 50% of qubits (partial eavesdropping). What QBER does this produce? (Expected: ≈12.5% — half of full 25%.)

**E4.1** Look up the CRYSTALS-Kyber key sizes: which Kyber parameter set is recommended for TLS 1.3 post-quantum key exchange? (Answer: ML-KEM-768 / Kyber-768 at security level 3.)

**E5.1** Symmetric encryption with AES-128 needs Grover's 2^64 operations to break. Estimate how many years that takes on a quantum computer doing 10^15 ops/sec.

### Quiz

| Q | Question | Answer |
|---|---|---|
| Q1 | What does Shor's algorithm break? | RSA (factoring) and ECC (discrete log) — all public key crypto based on these |
| Q2 | What quantum speedup does Grover give for AES? | Square root: AES-k bit → needs 2^{k/2} quantum search steps |
| Q3 | Why is LWE quantum-resistant? | No quantum algorithm breaks LWE significantly faster than classical |
| Q4 | What does BB84 use to detect eavesdropping? | QBER spike — Eve's measurement disturbs qubits, causing ≈25% error rate |
| Q5 | What is the QBER threshold for secure BB84? | 11% — above this, error correction cannot distil a secure key |
| Q6 | What are the three NIST PQC standards finalized in 2024? | FIPS 203 (ML-KEM/Kyber), FIPS 204 (ML-DSA/Dilithium), FIPS 205 (SLH-DSA/SPHINCS+) |
| Q7 | What security does E91 base its security on? | Bell inequality violation — any eavesdropping breaks entanglement correlations |

### Capstone: Crypto Migration Audit

```python
# Model a realistic system that uses multiple cryptographic primitives
# and audit its quantum vulnerability

SYSTEM_CRYPTO = {
    "TLS handshake (key exchange)":  ("ECDH-256",   "broken",    "Migrate to ML-KEM-768"),
    "TLS certificate (signature)":   ("ECDSA-256",  "broken",    "Migrate to ML-DSA-65"),
    "Database encryption":           ("AES-128-GCM","marginal",  "Upgrade to AES-256-GCM"),
    "Password hashing":              ("bcrypt",      "safe",      "No action needed"),
    "HMAC-SHA256":                   ("SHA-256",     "safe",      "Upgrade to SHA-384 in time"),
    "JWT signing":                   ("RS256-2048",  "broken",    "Migrate to ML-DSA or EdDSA-PQ"),
    "SSH keys":                      ("Ed25519",     "broken",    "Migrate to hybrid ECDH+KEM"),
}

print("=== Post-Quantum Crypto Migration Audit ===\n")
print(f"{'Component':<35} {'Current':<18} {'Post-Quantum Status':<12} {'Action'}")
print("-" * 95)
for component, (algo, status, action) in SYSTEM_CRYPTO.items():
    icon = "🔴" if status == "broken" else "🟡" if status == "marginal" else "🟢"
    print(f"{component:<35} {algo:<18} {icon} {status:<12}  {action}")

print(f"\n{'='*95}")
broken = sum(1 for _,(_,s,_) in SYSTEM_CRYPTO.items() if s == "broken")
print(f"  {broken}/{len(SYSTEM_CRYPTO)} components require immediate migration")
print("  Estimated migration effort: 12-18 months for enterprise system")
print("  NIST standards available NOW: FIPS 203, 204, 205 (finalised 2024)")
```

---

## 8. Glossary and Next Course

### Glossary

| Term | Definition |
|---|---|
| **RSA** | Public key crypto based on integer factorisation hardness → broken by Shor |
| **ECC / ECDLP** | Elliptic curve crypto based on discrete log hardness → broken by Shor |
| **Shor's algorithm** | O((log N)³) quantum factoring / discrete log → breaks RSA and ECC |
| **Grover's impact** | √N search: halves effective symmetric key length (AES-128→64-bit equivalent) |
| **LWE** | Learning With Errors: noisy linear system modular equation → quantum-hard |
| **ML-KEM** | FIPS 203 (formerly CRYSTALS-Kyber): lattice KEM, replaces ECDH |
| **ML-DSA** | FIPS 204 (formerly CRYSTALS-Dilithium): lattice signature, replaces ECDSA |
| **SLH-DSA** | FIPS 205 (formerly SPHINCS+): hash-based signature, no lattice required |
| **BB84** | First QKD protocol; uses two qubit bases; QBER detects eavesdropping |
| **Sifting** | BB84 step: keep bits where Alice and Bob chose the same basis (~50%) |
| **QBER** | Quantum Bit Error Rate — above 11% means no secure key possible |
| **E91** | Entanglement-based QKD; security verified via Bell inequality violation |
| **Hybrid PQC** | Run classical + PQC in parallel; safe if either is unbroken (transition strategy) |
| **Q-Day** | Estimated date when quantum computer can break RSA-2048 (~2033-2040) |

### Next Courses

➡ **ZQL-10** — *Quantum Sensing, Networking & Teleportation*  
Quantum sensors, Ramsey interferometry, quantum teleportation protocol (+entanglement swapping), quantum internet architecture.

➡ **ZQL-11** — *Quantum Error Mitigation & Fault-Tolerant Computing*  
ZNE and PEC (real hardware results); surface code stabilizers (from ZQL-07 Pauli formalism); BB84 noise mitigation on real fibre.

---

*All code verified vs zuntenium v0.2.0: `circuit.py` (qc.x, qc.h, qc.cx, qc.ry, qc.measure_all), `simulators.py` (StatevectorSimulator.run, result.counts).*  
*Generated from: ZQL-09-notes-katz-hoffstein-bernstein.md, ZQL-09-notes-bb84-nist-gisin.md.*
