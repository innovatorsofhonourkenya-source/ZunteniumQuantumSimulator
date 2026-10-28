# ZQL-09 Step 1 Notes — BB84 Protocol + NIST Standards + Gisin QKD Survey

---

# A. Bennett & Brassard — *Quantum Cryptography: Public Key Distribution and Coin Tossing* (1984)

## BB84 Protocol — Full Steps

### Setup
- Alice and Bob share a quantum channel (fibre/free-space) and a classical authenticated channel.
- No shared secret initially.

### Steps
1. **Alice prepares** n qubits randomly in one of four states:
   - Basis Z={|0⟩,|1⟩}: bit 0 → |0⟩, bit 1 → |1⟩
   - Basis X={|+⟩,|−⟩}: bit 0 → |+⟩, bit 1 → |−⟩
2. **Alice sends** qubits to Bob over quantum channel.
3. **Bob measures** each qubit in a randomly chosen basis (Z or X).
4. **Sifting**: Alice and Bob announce their basis choices (NOT the bits) over classical channel. Keep only bits where bases matched → raw key (≈50% of bits).
5. **QBER check**: compare a random subset of raw key bits. If QBER (Quantum Bit Error Rate) > threshold (typically 11%), eavesdropping detected → abort.
6. **Error correction + Privacy amplification**: reconcile remaining errors, apply hash to distil secure key.

### Why Eavesdropping is Detectable
- Eve must measure to learn information.
- If Eve guesses wrong basis: her measurement disturbs the qubit.
- Bob detects errors: theoretical QBER from Eve ≈ 25% (each mismatched measurement has 50% error rate).
- Threshold: QBER > 11% → abort (above 11%, no secure key possible after error correction).

### No-Cloning Theorem Security
- Eve cannot copy qubits (no-cloning theorem) — any intercept-and-resend strategy is detectable.
- Security proof: information-theoretic (no computational hardness assumption).

---

# B. NIST Post-Quantum Cryptography — FIPS 203, 204, 205 Parameters

## CRYSTALS-Kyber / ML-KEM (FIPS 203)
| Parameter set | Module rank k | Security level | pk size | sk size | ct size |
|---|---|---|---|---|---|
| Kyber-512 / ML-KEM-512 | 2 | L1 (AES-128) | 800 B | 1632 B | 768 B |
| Kyber-768 / ML-KEM-768 | 3 | L3 (AES-192) | 1184 B | 2400 B | 1088 B |
| Kyber-1024 / ML-KEM-1024 | 4 | L5 (AES-256) | 1568 B | 3168 B | 1568 B |

## CRYSTALS-Dilithium / ML-DSA (FIPS 204)
| Parameter set | Module rank | Security level | pk size | sig size |
|---|---|---|---|---|
| Dilithium2 / ML-DSA-44 | (4,4) | L2 | 1312 B | 2420 B |
| Dilithium3 / ML-DSA-65 | (6,5) | L3 | 1952 B | 3293 B |
| Dilithium5 / ML-DSA-87 | (8,7) | L5 | 2592 B | 4595 B |

## SPHINCS+ / SLH-DSA (FIPS 205)
| Parameter set | Hash | Security level | pk size | sig size |
|---|---|---|---|---|
| SPHINCS+-128s | SHAKE-256 | L1 | 32 B | 7856 B |
| SPHINCS+-128f | SHAKE-256 | L1 | 32 B | 17088 B |
| SPHINCS+-256s | SHAKE-256 | L5 | 64 B | 49856 B |

## Size Comparison vs Classical
| Algorithm | Type | Public key | Signature/ct |
|---|---|---|---|
| RSA-2048 | Classical (broken) | 256 B | 256 B |
| ECDSA-256 | Classical (broken) | 33 B | 64 B |
| Dilithium3 | PQC lattice | 1952 B | 3293 B |
| SPHINCS+-128s | PQC hash | 32 B | 7856 B |
| Kyber-768 | PQC KEM | 1184 B | 1088 B |

---

# C. Gisin et al. — *Quantum Cryptography* (Rev. Mod. Phys., 2002)

## QKD Practical Implementations
- **Fibre-based QKD**: 1550 nm telecom wavelength. Loss: ~0.2 dB/km → ≈100 km range.
- **Free-space QKD**: satellite-based (China's Micius satellite, 2017) → 1200 km demonstrated.
- **Repeater problem**: quantum no-cloning → no classical repeater. Quantum repeaters (entanglement swapping) are still research.

## E91 Protocol (Ekert, 1991)
- Uses entangled pairs |Φ+⟩ = (|00⟩ + |11⟩)/√2.
- Alice and Bob each hold one qubit of the entangled pair.
- Measurement correlations (Bell inequality violation) detect eavesdropping.
- **[SDK]** E91 circuit: Bell pair preparation → distribute → measure in random bases.

## QBER and Security Parameters
| QBER | Interpretation |
|---|---|
| 0% | Perfect channel, no eavesdropping |
| 3–5% | Typical noisy fibre channel |
| 11% | Shannon limit — no secure key below here |
| 25% | Eve randomly intercepts all qubits |
| >25% | Eve uses more sophisticated attack |

## [SDK] BB84 Simulation
BB84 can be fully simulated in QuFlow:
- Alice's qubit preparation: `qc.x(0)` (bit 1, Z-basis), `qc.h(0)` (X-basis)
- Bob's measurement: `qc.h(0)` then measure (X-basis), or just measure (Z-basis)
- Eavesdropping: insert intermediate measure → disturbs state → QBER≈25%
