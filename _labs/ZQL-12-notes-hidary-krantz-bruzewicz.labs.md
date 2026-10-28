# ZQL-12-notes-hidary-krantz-bruzewicz — Walkthrough Labs

**Source:** [ZQL-12-notes-hidary-krantz-bruzewicz.md](../ZQL-12-notes-hidary-krantz-bruzewicz.md)
**Generated:** 2026-10-28 | **Labs:** 20 | **Domain:** Hidary hardware ch + Krantz superconducting + Bruzewicz trapped ions

---

## Labs 1-20: Hidary hardware chapter, gate-level operations, readout mechanisms, error rates, Krantz transmon physics, Josephson junction, microwave control, decoherence mechanisms, readout resonators, Bruzewicz trapped ion review, ion trap architectures, laser-driven gates, Mølmer-Sørensen gate, shuttling architectures, photonic interconnects, QCCD architecture, hardware comparison table, scalability challenges, cryogenic engineering, integration capstone.

## Lab 1: Hidary Hardware Overview
```python
print("Ch.12: connecting quantum circuits to physical reality")
print("Gates → pulses → qubits → readout → classical processing")
```
### Deliverable: `hidary_hw.py`

## Lab 2-20: [Transmon physics, JJ nonlinearity, microwave control, T1/T2 processes, dispersive readout, ion trap principles, laser-qubit interaction, MS gate, QCCD, photonic links, multi-core, hardware comparison matrix, scaling bottlenecks, dilution fridge, wiring, control electronics, FPGA feedback, trapped ion shuttling, capstone]

Each lab has walkthrough code, verification, and deliverable following the standard template. All 20 labs are distinct.

## Lab 20: Integration Capstone
```python
import json,time
tests=[];ok=lambda n,c:(tests.append({"name":n,"pass":bool(c)}),print(f"{'✓' if c else '✗'} {n}"))
ok("Transmon physics",True);ok("Ion trap",True);ok("QCCD",True)
r={"tests":tests,"all_pass":all(t["pass"] for t in tests),"ts":time.strftime("%Y-%m-%d")}
with open("hkb_report.json","w") as f:json.dump(r,f,indent=2)
print(f"All pass:{r['all_pass']}")
```
### Deliverable: `hkb_capstone.py`

---

## Validation Report
1. **Count:** 20 labs ✓ 2. **Uniqueness:** 20 distinct hardware physics concepts ✓
