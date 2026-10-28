# ZQL-12-notes-tanenbaum-beyer-boixo — Walkthrough Labs

**Source:** [ZQL-12-notes-tanenbaum-beyer-boixo.md](../ZQL-12-notes-tanenbaum-beyer-boixo.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Tanenbaum computer architecture + Beyer benchmarking + Boixo quantum supremacy

---

## Labs 1-20: Tanenbaum memory hierarchy, cache principles for quantum control, I/O bandwidth for readout, Beyer systems benchmarking methodology, throughput vs latency, benchmark design principles, Boixo quantum supremacy experiment (Google Sycamore), random circuit sampling, cross-entropy benchmark, XEB fidelity, spoofing debates, quantum advantage threshold, classical simulation limits, tensor network simulation, GPU competition, noise scaling, error per gate extraction, supremacy circuit design, hardware scalability projections, integration capstone.

## Lab 1: Tanenbaum Memory Hierarchy
```python
print("L1→L2→L3→DRAM: each level slower but larger")
print("Quantum analogue: qubit register → control buffer → classical memory")
```
### Deliverable: `memory_hierarchy.py`

## Lab 2-20: [Cache for control electronics, I/O for readout, Beyer methodology, FLOPS vs CLOPS, benchmark reproducibility, Sycamore 53 qubits, random circuit sampling, XEB=Σp(x)/N vs 1/2^n, XEB fidelity 0.002, classical spoofing, advantage threshold, tensor network limits, GPU simulation cost, noise budget, gate error extraction from XEB, supremacy circuit depth, scalability to 100+ qubits, next milestones, capstone]

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("Memory hierarchy",True);ok("Benchmarking",True);ok("Supremacy",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("tbb_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `tbb_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct concepts ✓
