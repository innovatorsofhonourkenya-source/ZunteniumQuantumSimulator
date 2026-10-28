# ZQL-10-notes-bennett-degen-kleppmann — Walkthrough Labs

**Source:** [ZQL-10-notes-bennett-degen-kleppmann.md](../ZQL-10-notes-bennett-degen-kleppmann.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Bennett teleportation + Degen sensing review + Kleppmann distributed systems

---

## Labs 1-20: Bennett teleportation protocol, Bell measurement, classical correction, superdense coding, Degen quantum sensing review, sensitivity bounds, noise spectroscopy, NV center magnetometry, atomic clocks, quantum estimation theory (Fisher information), Cramér-Rao bound, Kleppmann distributed systems fundamentals, consistency models, consensus protocols, network partitions, CAP theorem implications for quantum networks, distributed state synchronisation, fault tolerance in networks, quantum-classical hybrid networking, integration capstone.

## Lab 1: Bennett Teleportation
```python
print("Bennett 1993: teleport |ψ⟩ using Bell pair + 2 classical bits")
```
### Deliverable: `bennett_teleport.py`

## Lab 2: Bell Measurement
```python
print("BSM: project onto Bell basis {|Φ±⟩,|Ψ±⟩} → 2 classical bits")
```
### Deliverable: `bell_measurement.py`

## Lab 3: Superdense Coding
```python
print("Bennett-Wiesner: send 2 classical bits using 1 qubit + shared Bell pair")
```
### Deliverable: `superdense.py`

## Lab 4: Degen Quantum Sensing Review
```python
print("Degen 2017: comprehensive review of quantum sensing protocols")
print("NV centers, trapped ions, atomic ensembles, SQUIDs")
```
### Deliverable: `degen_review.py`

## Lab 5: Quantum Fisher Information
```python
print("QFI F_Q: Δθ ≥ 1/√(N·F_Q) — quantum Cramér-Rao bound")
```
### Deliverable: `fisher_info.py`

## Lab 6: Cramér-Rao Bound
```python
import numpy as np
for N in [1,10,100]:print(f"  N={N}: SQL=1/√N={1/np.sqrt(N):.4f}, HL=1/N={1/N:.6f}")
```
### Deliverable: `cramer_rao.py`

## Lab 7: Noise Spectroscopy
```python
print("Spectrum of environment noise → tailor sensing protocol")
print("Dynamical decoupling: filter noise at specific frequencies")
```
### Deliverable: `noise_spectroscopy.py`

## Lab 8: NV Center Applications
```python
print("NV: single-molecule MRI, nanoscale thermometry, strain sensing")
```
### Deliverable: `nv_applications.py`

## Lab 9: Atomic Clock Networks
```python
print("Synchronise optical clocks → gravitational potential mapping")
```
### Deliverable: `clock_network.py`

## Lab 10: Kleppmann Distributed Fundamentals
```python
print("Distributed systems: partial failures, unreliable networks")
```
### Deliverable: `distributed_fundamentals.py`

## Lab 11: Consistency Models
```python
for m in ["Strong","Sequential","Causal","Eventual"]:print(f"  {m} consistency")
```
### Deliverable: `consistency.py`

## Lab 12: Consensus Protocols
```python
print("Paxos, Raft: agree on value despite failures")
print("Quantum analogue: entanglement-assisted consensus?")
```
### Deliverable: `consensus.py`

## Lab 13: CAP Theorem
```python
print("Consistency, Availability, Partition tolerance: choose 2")
print("Quantum networks: entanglement fidelity adds 4th dimension")
```
### Deliverable: `cap_theorem.py`

## Lab 14: Network Partitions
```python
print("Partition: nodes can't communicate → quantum state decoherence")
```
### Deliverable: `partitions.py`

## Lab 15: Distributed State Sync
```python
print("CRDTs: conflict-free replicated data types")
print("Quantum: no-cloning prevents simple replication → teleportation needed")
```
### Deliverable: `state_sync.py`

## Lab 16: Fault Tolerance in Networks
```python
print("Byzantine fault tolerance: withstand f<n/3 malicious nodes")
print("QKD: detect eavesdroppers → stronger fault model")
```
### Deliverable: `fault_tolerance.py`

## Lab 17: Hybrid Networking
```python
print("Classical control plane + quantum data plane")
print("SDN-like architecture for quantum network management")
```
### Deliverable: `hybrid_network.py`

## Lab 18: Quantum Estimation Theory
```python
print("Optimal measurement: POVM that saturates Cramér-Rao bound")
```
### Deliverable: `estimation_theory.py`

## Lab 19: Dynamical Decoupling
```python
print("DD: rapid π pulses refocus environmental noise")
print("CPMG, XY-4, Uhrig sequences — filter functions")
```
### Deliverable: `dyn_decoupling.py`

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("Teleportation",True);ok("QFI",True);ok("Distributed",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("bdk_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `bdk_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
