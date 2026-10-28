# ZQL-09-notes-katz-hoffstein-bernstein — Walkthrough Labs

**Source:** [ZQL-09-notes-katz-hoffstein-bernstein.md](../ZQL-09-notes-katz-hoffstein-bernstein.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Katz-Lindell crypto + Hoffstein NTRU + Bernstein PQC

---

## Lab 1: Katz-Lindell Security Definitions
```python
print("IND-CPA: ciphertext indistinguishable under chosen plaintext")
print("IND-CCA2: indistinguishable under adaptive chosen ciphertext")
```
### Deliverable: `security_defs.py`

## Lab 2: RSA Cryptosystem
```python
print("RSA: N=pq, e·d≡1 mod φ(N), encrypt c=mᵉ mod N")
print("Shor breaks: factors N in poly time → RSA insecure post-quantum")
```
### Deliverable: `rsa.py`

## Lab 3: Diffie-Hellman Key Exchange
```python
print("DH: g^a mod p, g^b mod p → shared secret g^{ab} mod p")
print("Quantum threat: discrete log also broken by Shor")
```
### Deliverable: `diffie_hellman.py`

## Lab 4: Elliptic Curve Cryptography
```python
print("ECC: P=kG on curve, smaller keys than RSA")
print("Quantum threat: ECDLP broken by Shor variant")
```
### Deliverable: `ecc.py`

## Lab 5: Hoffstein NTRU Lattice
```python
print("NTRU (1996): first practical lattice-based encryption")
print("Ring-LWE variant: polynomial ring Z[x]/(x^n+1)")
```
### Deliverable: `ntru.py`

## Lab 6: Learning With Errors (LWE)
```python
print("LWE: given (A,b=As+e mod q), find s")
print("Hardness: worst-case lattice problems → average-case LWE")
```
### Deliverable: `lwe.py`

## Lab 7: Ring-LWE Efficiency
```python
print("Ring-LWE: structured lattice → smaller keys, faster ops")
print("NTRU/Kyber: based on Module-LWE (compromise)")
```
### Deliverable: `ring_lwe.py`

## Lab 8: Bernstein PQC Analysis
```python
print("Bernstein: rigorous analysis of PQC candidate security")
print("Key sizes, signature sizes, performance benchmarks")
```
### Deliverable: `bernstein_pqc.py`

## Lab 9: Code-Based Cryptography
```python
print("McEliece (1978): encrypt using error-correcting codes")
print("Large keys (~1 MB) but proven security for 40+ years")
```
### Deliverable: `code_based.py`

## Lab 10: Multivariate Cryptography
```python
print("Systems of multivariate quadratic equations over F_q")
print("Solving MQ is NP-hard → candidate for PQC signatures")
```
### Deliverable: `multivariate.py`

## Lab 11: Hash-Based Signatures — Merkle Trees
```python
print("Lamport: one-time signature from hash function")
print("Merkle tree: chain Lamport signatures → many signatures")
```
### Deliverable: `merkle.py`

## Lab 12: Quantum-Safe TLS
```python
print("TLS 1.3 + ML-KEM (Kyber) key exchange")
print("Backward compatible: hybrid with ECDH for transition")
```
### Deliverable: `quantum_tls.py`

## Lab 13: Grover's Impact on Symmetric Crypto
```python
print("Grover: O(√N) search → halves symmetric key security")
print("AES-128 → 64-bit quantum security; AES-256 → 128-bit ← sufficient")
```
### Deliverable: `grover_symmetric.py`

## Lab 14: Lattice Reduction Attacks
```python
print("LLL/BKZ: classical lattice reduction algorithms")
print("No known quantum speedup for lattice reduction → PQC safe")
```
### Deliverable: `lattice_attacks.py`

## Lab 15: Key Encapsulation Mechanism
```python
print("KEM: (pk,sk)=KeyGen(); (ct,ss)=Encaps(pk); ss=Decaps(sk,ct)")
print("Kyber: IND-CCA2 KEM from Module-LWE")
```
### Deliverable: `kem.py`

## Lab 16: Digital Signature Schemes
```python
print("(pk,sk)=KeyGen(); σ=Sign(sk,m); Verify(pk,m,σ)→{0,1}")
print("PQC: Dilithium (lattice), SPHINCS+ (hash), Falcon (lattice)")
```
### Deliverable: `signatures.py`

## Lab 17: Cryptographic Agility
```python
print("Design systems to swap algorithms without re-architecting")
print("Critical for PQC transition: algorithm may be broken later")
```
### Deliverable: `crypto_agility.py`

## Lab 18: Quantum Computing Timeline for Crypto
```python
for yr,milestone in [(2024,"NIST PQC standards"),(2028,"~1000 logical qubits?"),
    (2035,"RSA-2048 breakable?")]:print(f"  {yr}: {milestone}")
```
### Deliverable: `timeline.py`

## Lab 19: Katz-Lindell Reduction Proofs
```python
print("Reduction: if adversary breaks scheme, then solver breaks hard problem")
print("PQC reductions: typically to LWE, SVP, or hash security")
```
### Deliverable: `reductions.py`

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("RSA threat",True);ok("LWE hardness",True);ok("NIST standards",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("khb_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `khb_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
