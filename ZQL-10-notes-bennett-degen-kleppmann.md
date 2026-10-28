# ZQL-10 Step 1 Notes — Bennett 1993 (Teleportation) + Degen 2017 (Quantum Sensing) + Kleppmann (CRDT)

---

# A. Bennett et al. — *Teleporting an Unknown Quantum State* (Phys. Rev. Lett., 1993)

## The Teleportation Protocol (Original Paper)

### Resources Required
- 1 EPR pair (shared between Alice and Bob) — established in advance.
- The unknown state |ψ⟩ = α|0⟩ + β|1⟩ at Alice's location.
- 2 classical bits communicated from Alice to Bob.

### Protocol Steps
1. **Alice's side**: she has qubit |ψ⟩ (to teleport) and her half of EPR pair.
2. **Alice's Bell measurement**: apply CNOT(ψ, EPR_A), then H(ψ), then measure both.
   - Result: 4 possible outcomes (00, 01, 10, 11), each with equal probability.
3. **Classical communication**: Alice sends 2 classical bits to Bob.
4. **Bob's correction**: Bob applies Pauli corrections based on Alice's bits:
   - 00 → I (nothing), 01 → X, 10 → Z, 11 → ZX.
5. **Result**: Bob's qubit is now in state |ψ⟩.

### Mathematical Verification
Start: (α|0⟩+β|1⟩)_A ⊗ (|00⟩+|11⟩)_{A'B}/√2

After Alice's CNOT + H + measure:
- If Alice gets 00: Bob has α|0⟩+β|1⟩ ← correct, no correction needed
- If Alice gets 01: Bob has α|1⟩+β|0⟩ ← apply X
- If Alice gets 10: Bob has α|0⟩−β|1⟩ ← apply Z
- If Alice gets 11: Bob has α|1⟩−β|0⟩ ← apply ZX

### No-Faster-Than-Light
- Bob cannot extract |ψ⟩ without Alice's classical bits.
- Classical channel has latency.
- No information transferred faster than light.

### **[SDK]** Teleportation Circuit
```
Qubit 0: |ψ⟩           ──[H]──●────────[measure]
                               │
Qubit 1: EPR(A)  |0⟩──[H]──●──⊕──[measure]
                             │
Qubit 2: EPR(B)  |0⟩────────⊕─────[condX]──[condZ]──→|ψ⟩
```
SDK: qc.h(0), qc.cx(0,1), qc.cx(1,2), qc.h(1) → measure → conditionals.
Note: QuFlow v0.2.0 does NOT yet support mid-circuit classical conditioning.
Workaround: use deferred measurement principle (measure at end).

---

# B. Degen, Reinhard & Cappellaro — *Quantum Sensing* (Rev. Mod. Phys., 2017)

## Classification of Quantum Sensors

### By Physical System
| Sensor system | Quantity sensed | Precision |
|---|---|---|
| Nitrogen-vacancy (NV) centres | Magnetic field, temperature | nT/√Hz at room temp |
| Trapped ions | Electric/magnetic fields, frequency | Sub-Hz atomic clocks |
| Superconducting qubits | Flux, charge, microwave field | Single photon |
| Rydberg atoms | Electric field | <100 nV/cm |
| Atom interferometer | Gravity, rotation | ppb precision |

### Ramsey Spectroscopy (Quantum Clock Protocol)
1. Prepare |+⟩ = H|0⟩.
2. Free evolution for time T under unknown Hamiltonian H = ωZ/2.
   State: |ψ(T)⟩ = e^{-iωTZ/2}|+⟩ = cos(ωT/2)|0⟩ + e^{-iωT}sin(ωT/2)|1⟩.
3. Apply H → measure Z.
4. ⟨Z⟩ = cos(ωT) → estimate ω.

Fisher information: F = T² (for Ramsey with 1 qubit).
With n qubits (Heisenberg): F = n²T².

### Dynamical Decoupling (DD)
- Problem: environmental noise (dephasing) limits coherence time T₂.
- DD solution: apply π pulses periodically to refocus decoherence.
- **Hahn echo**: |+⟩ → wait T/2 → X → wait T/2 → measure. Cancels static field noise.
- **CPMG**: n π-pulses. Extends T₂ by n².
- **[SDK]** Hahn echo: `qc.h(0); [evolution T/2]; qc.x(0); [evolution T/2]; qc.h(0); measure`.

---

# C. Martin Kleppmann — *Designing Data-Intensive Applications* (O'Reilly, 2017)

**Relevant section: CRDTs — underpins the Zuntenium collaboration server.**

## CRDTs (Conflict-free Replicated Data Types)

### Problem: Distributed State under Concurrent Edits
- Multiple users editing the same quantum circuit simultaneously.
- No central lock → concurrent edits may conflict.
- CRDT: an algebraic approach ensuring eventual consistency *without coordination*.

### Types of CRDTs
- **G-Counter**: grow-only counter. State: vector of n counters (one per node).
  Merge: take element-wise max. Commutative + associative.
- **LWW-Register** (Last Write Wins): each write has a timestamp; merge takes latest.
- **CRDT Set** (OR-Set): each add tagged with unique ID; remove only removes specific IDs.
- **[SDK Platform]** Quantum circuit collaboration: each gate has a globally unique ID (UUID).
  - Add gate: insert (gate_id, gate_data) into OR-Set.
  - Remove gate: remove specific gate_id.
  - Concurrent adds: both survive. Concurrent remove + add: add wins.

### Operational Transform vs CRDT
| Property | OT (Google Docs) | CRDT |
|---|---|---|
| Coordination | Requires server transform | Server-free |
| Latency | RTT to server | Peer-to-peer |
| Correctness proof | Complex, bugs common | Algebraic guarantee |
| Implementation | Complex | Standardised algorithms |

### CAP Theorem
- **C**onsistency: all nodes see same data at same time.
- **A**vailability: every request gets a response.
- **P**artition tolerance: system works even if nodes can't communicate.
- CRDTs choose **AP** (available + partition tolerant) over strict consistency.
- **[SDK Platform]** Acceptable for collaborative circuit editing — eventual consistency is fine.
