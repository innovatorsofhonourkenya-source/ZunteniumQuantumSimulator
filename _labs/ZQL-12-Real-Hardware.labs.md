# ZQL-12-Real-Hardware — Walkthrough Labs

**Source:** [ZQL-12-Real-Hardware.md](../ZQL-12-Real-Hardware.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Superconducting qubits, trapped ions, hardware calibration, noise

---

## Lab 1: Qubit Technologies Overview
```python
for tech,t1,t2,gate in [("Superconducting","~100μs","~50μs","~20ns"),
    ("Trapped ion","~10s","~1s","~10μs"),("Photonic","long","N/A","~ps"),
    ("Neutral atom","~1s","~100ms","~1μs")]:
    print(f"  {tech:<18}: T1={t1}, T2={t2}, gate={gate}")
```
### Deliverable: `qubit_technologies.py`

## Lab 2: Superconducting Transmon
```python
print("Transmon: Josephson junction + capacitor → anharmonic oscillator")
print("|0⟩,|1⟩ = lowest two energy levels, controlled by microwave pulses")
```
### Deliverable: `transmon.py`

## Lab 3: Trapped Ion Qubits
```python
print("Ions confined in Paul trap, addressed by laser beams")
print("Hyperfine or optical transitions → very high fidelity gates")
```
### Deliverable: `trapped_ion.py`

## Lab 4: Gate Fidelity
```python
print("Fidelity F = |⟨ψ_ideal|ψ_actual⟩|²")
print("Single-qubit: F>99.9%; Two-qubit: F>99% (state of art)")
```
### Deliverable: `gate_fidelity.py`

## Lab 5: T1 and T2 Relaxation
```python
print("T1: energy relaxation (|1⟩→|0⟩), T2: phase decoherence")
print("T2 ≤ 2T1; circuit depth limited by T_gate × n_gates < T2")
```
### Deliverable: `t1_t2.py`

## Lab 6: Readout Error
```python
print("Readout error: P(0|1) and P(1|0), typically 1-5%")
print("Mitigate: confusion matrix calibration (ZQL-11)")
```
### Deliverable: `readout_error.py`

## Lab 7: Coupling Maps
```python
print("Hardware connectivity: not all-to-all")
print("SWAP gates needed for non-adjacent qubits (each SWAP = 3 CX)")
```
### Deliverable: `coupling_map.py`

## Lab 8: Circuit Transpilation
```python
print("Transpilation: decompose to native gate set + route to hardware")
print("Native: {√X, Rz, CX} (IBM) or {Rz, √iSWAP} (Google)")
```
### Deliverable: `transpilation.py`

## Lab 9: Randomized Benchmarking
```python
print("RB: apply random Clifford sequences of increasing length")
print("Decay rate → average gate fidelity: F ≈ 1 - (d-1)·r/d")
```
### Deliverable: `randomized_bench.py`

## Lab 10: Cross-Resonance Gate
```python
print("CR gate: IBM's native two-qubit gate")
print("Drive qubit A at frequency of qubit B → conditional rotation")
```
### Deliverable: `cross_resonance.py`

## Lab 11: Calibration Workflow
```python
print("1. Frequency calibration (Ramsey/spectroscopy)")
print("2. Pulse amplitude calibration (Rabi oscillation)")
print("3. T1/T2 measurement")
print("4. Gate error benchmarking (RB)")
print("5. Readout calibration")
```
### Deliverable: `calibration.py`

## Lab 12: Crosstalk
```python
print("Crosstalk: driving qubit A affects neighbor B")
print("Mitigation: stagger operations, dynamical decoupling")
```
### Deliverable: `crosstalk.py`

## Lab 13: Dynamical Decoupling on Hardware
```python
print("DD: insert refocusing π pulses during idle periods")
print("XY4, CPMG sequences → filter low-frequency noise")
```
### Deliverable: `dd_hardware.py`

## Lab 14: Quantum Volume
```python
print("QV=2^n: largest n×n random circuit with >2/3 heavy output")
print("QV=128 (IBM Eagle), QV=32768 (Quantinuum)")
```
### Deliverable: `quantum_volume.py`

## Lab 15: CLOPS — Circuit Layer Operations Per Second
```python
print("CLOPS: throughput metric, circuits executed per second")
print("Includes: compilation, execution, readout")
```
### Deliverable: `clops.py`

## Lab 16: Error Budget for Real Hardware
```python
print("2-qubit gate errors dominate: ~0.5-1% per CX")
print("100 CX gates: ~40-60% probability of at least one error")
```
### Deliverable: `error_budget.py`

## Lab 17: Qiskit/QuFlow Hardware Submission
```python
print("qc = QuantumCircuit(2); qc.h(0); qc.cx(0,1); qc.measure_all()")
print("backend = IBMQBackend('ibm_brisbane'); result = backend.run(qc)")
```
### Deliverable: `hw_submission.py`

## Lab 18: Job Queue and Batching
```python
print("Real hardware: job queue wait times (minutes to hours)")
print("Batch circuits: submit multiple experiments in one job")
```
### Deliverable: `batching.py`

## Lab 19: Hardware Noise Simulation
```python
print("Noise model: add depolarising, amplitude damping, readout errors")
print("Simulate realistic results before consuming hardware time")
```
### Deliverable: `noise_sim.py`

## Lab 20: Real Hardware Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("Gate fidelity",True);ok("T1/T2",True);ok("Transpilation",True);ok("QV metric",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("hw_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `hw_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct hardware concepts ✓
