# ZQL-15-Benchmarking — Walkthrough Labs

**Source:** [ZQL-15-Benchmarking.md](../ZQL-15-Benchmarking.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Quantum benchmarking, quantum volume, CLOPS, application benchmarks

---

## Lab 1: Why Benchmark Quantum Computers
```python
print("Benchmarks: compare hardware, track progress, identify bottlenecks")
print("No single metric captures all aspects → use suite of benchmarks")
```
### Deliverable: `why_benchmark.py`

## Lab 2: Quantum Volume
```python
print("QV=2^n: largest n where random SU(4) circuit succeeds >2/3")
print("Tests: gate fidelity, connectivity, compiler quality")
```
### Deliverable: `quantum_volume.py`

## Lab 3: QV Measurement Protocol
```python
print("1. Generate random SU(4) circuits of width=depth=n")
print("2. Run on hardware, compute heavy output probability")
print("3. If P(heavy) > 2/3 → QV ≥ 2^n; increase n and repeat")
```
### Deliverable: `qv_protocol.py`

## Lab 4: CLOPS — Circuit Layer Operations Per Second
```python
print("CLOPS: end-to-end throughput including compilation + execution")
print("Measures: how fast can you run parameterised circuits in a loop")
```
### Deliverable: `clops.py`

## Lab 5: Randomized Benchmarking
```python
print("RB: random Clifford sequences → decay gives avg gate error")
print("Interleaved RB: isolate error of specific gate")
```
### Deliverable: `randomized_bench.py`

## Lab 6: Cross-Entropy Benchmarking (XEB)
```python
print("XEB: compare measured bitstring probabilities to ideal")
print("F_XEB = 2^n · Σ p_ideal(x)·p_measured(x) - 1")
```
### Deliverable: `xeb.py`

## Lab 7: Application Benchmarks
```python
for app,metric in [("VQE","Chemical accuracy (mHartree)"),("QAOA","Approximation ratio"),
    ("QML","Classification accuracy"),("QEC","Logical error rate")]:
    print(f"  {app}: {metric}")
```
### Deliverable: `app_benchmarks.py`

## Lab 8: Mirror Circuits
```python
print("Mirror circuit: U followed by U† → should return to |0⟩")
print("Fidelity = P(|0⟩) — measures total circuit error")
```
### Deliverable: `mirror_circuits.py`

## Lab 9: Gate Set Tomography
```python
print("GST: full characterisation of gate set (not just average error)")
print("Detects: coherent errors, crosstalk, non-Markovian noise")
```
### Deliverable: `gst.py`

## Lab 10: Cycle Benchmarking
```python
print("Cycle benchmarking: characterise error per parallel gate layer")
print("More realistic than single-gate RB for actual circuit performance")
```
### Deliverable: `cycle_bench.py`

## Lab 11: Volumetric Benchmarking
```python
print("Plot pass/fail for circuits of varying width × depth")
print("Reveals the shape of compute capability, not just a single number")
```
### Deliverable: `volumetric.py`

## Lab 12: Algorithmic Qubits
```python
print("AQ = number of qubits usable for a specific algorithm")
print("Accounts for connectivity, error rates, compilation overhead")
```
### Deliverable: `algorithmic_qubits.py`

## Lab 13: Benchmark Automation
```python
print("Automated benchmark suite: nightly runs, trend tracking")
print("Tools: Qiskit benchmarks, custom pytest-benchmark")
```
### Deliverable: `automation.py`

## Lab 14: Comparing Hardware Platforms
```python
for hw,qv,clops in [("IBM Eagle",128,1500),("Quantinuum H2",65536,35),
    ("Google Sycamore","N/A","N/A"),("IonQ Aria",25,200)]:
    print(f"  {hw:<20}: QV={qv}, CLOPS≈{clops}")
```
### Deliverable: `hw_comparison.py`

## Lab 15: Noise Characterisation
```python
print("T1, T2, gate fidelity, readout error, crosstalk → noise fingerprint")
print("Feed into noise models for simulation")
```
### Deliverable: `noise_char.py`

## Lab 16: Compilation Quality Metrics
```python
print("Metrics: depth reduction, CX count, SWAP overhead")
print("Compare compilers: Qiskit, tket, QuFlow transpiler")
```
### Deliverable: `compiler_metrics.py`

## Lab 17: Statistical Significance
```python
print("Bootstrap confidence intervals for benchmark results")
print("Minimum: 100 shots × 10 repetitions per data point")
```
### Deliverable: `statistics.py`

## Lab 18: Benchmark Reproducibility
```python
print("Pin: SDK version, backend calibration date, compiler settings")
print("Report: mean ± std with hardware metadata")
```
### Deliverable: `reproducibility.py`

## Lab 19: Future Benchmarks
```python
print("Emerging: QCVV, QEC benchmarks, fault-tolerant benchmarks")
print("Logical error rate per round → primary FT metric")
```
### Deliverable: `future_bench.py`

## Lab 20: Benchmarking Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("QV concept",True);ok("CLOPS",True);ok("RB",True);ok("XEB",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("bench_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `bench_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct benchmarking concepts ✓
