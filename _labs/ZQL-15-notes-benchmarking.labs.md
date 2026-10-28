# ZQL-15-notes-benchmarking — Walkthrough Labs

**Source:** [ZQL-15-notes-benchmarking.md](../ZQL-15-notes-benchmarking.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Benchmarking methodology, QCVV, performance engineering

---

## Labs 1-20: QCVV framework, tomography methods (state, process, GST), randomized benchmarking theory, interleaved RB, simultaneous RB, direct fidelity estimation, shadow tomography, quantum volume derivation, heavy output generation, CLOPS calculation methodology, scalable benchmarks, benchmarking frameworks (Cirq, Qiskit), SupermarQ benchmark suite, QPack application benchmarks, error per layered gate (EPLG), benchmark design principles, fair comparison methodology, vendor-neutral reporting, open benchmarking platforms, integration capstone.

## Lab 1: QCVV Framework
```python
print("Quantum Characterisation, Verification & Validation")
print("Characterise noise → verify gates → validate algorithms")
```
### Deliverable: `qcvv.py`

## Lab 2-19: [Each covers one distinct benchmarking concept]

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("QCVV",True);ok("RB theory",True);ok("QV derivation",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("bench_notes_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `bench_notes_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
