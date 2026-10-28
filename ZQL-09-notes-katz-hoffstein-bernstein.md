# ZQL-09 Step 1 Notes — Katz & Lindell (Cryptography) + Hoffstein (Lattices) + Bernstein (PQC)

---

# A. Katz & Lindell — *Introduction to Modern Cryptography* (3rd ed., 2020)

## Classical Cryptographic Hardness Assumptions

### RSA (Rivest-Shamir-Adleman)
- **Hardness**: integer factorisation of N=p·q (product of two large primes).
- Key sizes: 2048-bit (safe against classical), 4096-bit (comfortable margin).
- Encryption: c = mᵉ mod N; Decryption: m = cᵈ mod N where ed ≡ 1 (mod φ(N)).
- **Quantum threat**: Shor's algorithm factors N in O((log N)³) → RSA is broken.

### Elliptic Curve Cryptography (ECC)
- **Hardness**: Elliptic Curve Discrete Logarithm Problem (ECDLP).
- ECDLP: given P and Q=kP on elliptic curve, find k.
- Best classical: O(√N) (Pollard rho). ECDSA, ECDH use 256-bit keys.
- **Quantum threat**: Shor's algorithm (via quantum phase estimation) can solve ECDLP in O((log N)³). → ECC is broken.

### Symmetric Cryptography (AES, SHA)
- AES-256: Grover's algorithm halves effective key length → AES-256 → AES-128 equivalent.
- SHA-256, SHA-3: Grover reduces preimage resistance 2²⁵⁶ → 2¹²⁸. Still safe if doubled.
- **[Important]** Symmetric crypto needs larger keys (double bit length), not a full replacement.

## Threat Timeline
| Cryptosystem | Classical security | Quantum threat | Required action |
|---|---|---|---|
| RSA-2048 | 112-bit | Shor → broken | Replace with PQC |
| ECC-256 | 128-bit | Shor → broken | Replace with PQC |
| AES-128 | 128-bit | Grover → 64-bit | Upgrade to AES-256 |
| AES-256 | 256-bit | Grover → 128-bit | Still considered safe |
| SHA-256 | 256-bit | Grover → 128-bit | Still considered safe |
| SHA-3-384 | 384-bit | Grover → 192-bit | Safe |

---

# B. Hoffstein, Pipher & Silverman — *An Introduction to Mathematical Cryptography* (2nd ed.)

## Lattice-Based Cryptography

### What is a Lattice?
A lattice L ⊆ ℝⁿ is a discrete additive subgroup:
```
L = {a₁v₁ + a₂v₂ + ... + aₖvₖ : aᵢ ∈ ℤ}
```
where {v₁,...,vₖ} is a basis.

### Hard Lattice Problems
- **SVP (Shortest Vector Problem)**: find the shortest non-zero vector in L.
- **CVP (Closest Vector Problem)**: given a point t ∉ L, find the closest lattice point.
- **LWE (Learning With Errors)**: given (A, b = As + e) where e is small noise, find s.
- **[Key]** These problems are believed hard for both classical AND quantum computers. No quantum speedup beyond square-root Grover.

### LWE (Learning With Errors) — CRYSTALS-Kyber uses Module-LWE
```
Setup: A ∈ Zq^{m×n}, s ∈ Zq^n (secret), e ∈ small noise
Public: (A, b = As + e mod q)
Problem: find s given (A, b)

For typical params (Kyber-512):
  n=256, q=3329, k=2 (module rank)
```

### NTRU
- Original lattice-based cryptosystem (1996).
- Based on polynomial rings: ℤ[X]/(Xⁿ−1).
- Encrypt: c = r·h + m (mod q) where h = g/f (public key).
- Break: find short (f, g) satisfying fh ≡ g (mod q) — a lattice problem.

---

# C. Bernstein & Lange (eds.) — *Post-Quantum Cryptography* (Springer, 2009) + NIST PQC Standards

## PQC Algorithm Families
| Family | Based on | NIST standard | Key size | Signature size |
|---|---|---|---|---|
| Lattice (KEM) | Module-LWE | CRYSTALS-Kyber (FIPS 203) | 800B–1568B | N/A |
| Lattice (Sig) | Module-LWE | CRYSTALS-Dilithium (FIPS 204) | 1312B–2592B | 2420B–4595B |
| Hash (Sig) | Hash functions | SPHINCS+ (FIPS 205) | 32B–64B | 8B–49K |
| Lattice (Sig) | NTRU lattices | FALCON (FIPS optional) | 897B–1793B | 666B–1280B |

## CRYSTALS-Kyber (FIPS 203, ML-KEM)
- KEM (Key Encapsulation Mechanism): replace RSA/ECDH key exchange.
- Security: MLWE with module rank k (k=2: Kyber-512, k=3: Kyber-768, k=4: Kyber-1024).
- Operation:
  1. KeyGen: A←U(Rq^{k×k}), s,e←β_η → pk=(A,t=As+e), sk=s
  2. Encaps: r,e₁,e₂←β_η → u=Aᵀr+e₁, v=tᵀr+e₂+⌊q/2⌋m → (c=(u,v))
  3. Decaps: m = ⌊decode(v − sᵀu)⌉

## SPHINCS+ (FIPS 205, SLH-DSA)
- Hash-based signature — security from collision resistance only.
- No algebraic structure → most conservative (safest post-quantum choice).
- Uses hypertree of XMSS trees + FORS one-time signature scheme.
- Main drawback: large signatures (8KB–49KB) vs 64 bytes for ECDSA.

## BB84 Quantum Key Distribution (Bennett & Brassard, 1984)
- **Protocol**: distribute a cryptographic key using quantum states.
- No assumptions about computational hardness — security from physics.
- Uses two bases: {|0⟩,|1⟩} (Z basis) and {|+⟩,|−⟩} (X basis).
- Eavesdropping disturbs the quantum state → detectable.
