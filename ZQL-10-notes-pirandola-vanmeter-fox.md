# ZQL-10 Step 1 Notes — Pirandola (Sensing) + Van Meter (Networking) + Fox (Quantum Optics)

---

# A. Pirandola & Lupo — *Quantum Metrology, Imaging, and Communication* (De Gruyter, 2016)

## Quantum Metrology

### Quantum Fisher Information (QFI)
- Classical Cramér-Rao bound: Var(θ̂) ≥ 1/[n·F_cl(θ)].
- Quantum Cramér-Rao bound: Var(θ̂) ≥ 1/[n·F_Q(θ)] where F_Q ≥ F_cl.
- QFI for a pure state |ψ(θ)⟩: F_Q = 4(⟨∂_θψ|∂_θψ⟩ − |⟨∂_θψ|ψ⟩|²).

### Standard Quantum Limit (SQL) vs Heisenberg Limit (HL)
- **SQL** (shot noise limit): using n independent qubits/photons → σ_θ ≥ 1/√n.
- **Heisenberg Limit**: using entanglement → σ_θ ≥ 1/n (quadratic improvement).
- GHZ state |GHZ_n⟩ = (|0ⁿ⟩ + |1ⁿ⟩)/√2 achieves Heisenberg Limit.
- **[SDK]** GHZ state preparation: qc.h(0) + qc.cx(0,1) + ... + qc.cx(0,n-1).

### Ramsey Interferometry
- Setup: atom initialised in |↑⟩ (or |0⟩), exposed to unknown frequency ω for time T.
- Circuit analogue:
  1. H gate → creates superposition |+⟩
  2. Phase accumulation: Rz(ωT) → e^{-iωT/2}|0⟩ + e^{iωT/2}|1⟩
  3. H gate → measurement
- Measured expectation: ⟨Z⟩ = cos(ωT)
- Estimate ω = arccos(⟨Z⟩)/T.

## Quantum Sensing Applications

| Sensor | Physical quantity | Quantum advantage |
|---|---|---|
| Atomic clock | Frequency / time | Heisenberg limit via Ram-sey |
| Gravimeter | Gravity gradient | Matter-wave interferometry |
| Magnetometer | Magnetic field | Spin squeezing |
| LIDAR (quantum) | Range / velocity | Entangled photons, 6 dB SNR gain |
| MRI (quantum-enhanced) | Molecular structure | Improved T₁/T₂ resolution |

---

# B. Rodney Van Meter — *Quantum Networking* (Wiley-ISTE, 2013)

## Quantum Network Stack

### Layers
1. **Physical**: photon transmission, qubit encoding in fibre/free-space.
2. **Link**: entanglement generation between adjacent nodes.
3. **Network**: entanglement distribution over multi-hop paths (quantum repeaters).
4. **Transport**: end-to-end reliability for quantum state transfer.
5. **Application**: QKD, distributed quantum computing, blind QC.

### Quantum Repeaters
- Classical repeaters: amplify signal. **Impossible** for quantum signals (no-cloning).
- Quantum repeaters: use entanglement swapping to extend range.
  1. Generate entanglement A–B and B–C.
  2. Perform Bell measurement at B: swaps entanglement → A–C entangled.
  3. Repeat for longer chains.
- Key requirement: quantum memory (store entanglement while waiting for swapping).

### Entanglement Swapping Protocol
```
Initial:  A⊗B entangled, B'⊗C entangled (B and B' at same node)
Bell meas on (B, B') → classical 2-bit result
Correct A and C based on result → A⊗C entangled
```

### Quantum Internet Architecture (Wehner, Elkouss, Hanson 2018)
Six stages:
1. Trusted repeater (QKD + classical repeaters)
2. Prepare-and-measure (BB84 over fibre)
3. Entanglement distribution (heralded, EPR pairs)
4. Quantum memory (store entanglement, repeat)
5. Fault-tolerant (logical qubits in network)
6. Quantum computing cloud (blind quantum computing)

---

# C. Mark Fox — *Quantum Optics: An Introduction* (Oxford, 2004)

## Photon Statistics
- **Coherent state** (laser): Poissonian photon statistics, Fano factor F=1.
- **Fock state** |n⟩: exactly n photons, sub-Poissonian (F<1), non-classical.
- **Thermal state**: super-Poissonian (F>1), bunching.
- **[Quantum comms]** Photons as flying qubits: polarisation encoding |H⟩=|0⟩, |V⟩=|1⟩.

## Entangled Photon Pairs (SPDC)
- Spontaneous Parametric Down-Conversion (SPDC): pump photon → signal + idler.
- Energy conservation: ωₚ = ωₛ + ωᵢ.
- Momentum conservation (phase matching): kₚ = kₛ + kᵢ.
- Type-II SPDC: orthogonal polarisations → polarisation-entangled pairs |Φ+⟩.
- **[SDK]** Simulated as Bell pair: qc.h(0) + qc.cx(0,1).

## Quantum Teleportation Theory
- Bennett et al. (1993): transfer quantum state using EPR pair + 2 classical bits.
- No faster-than-light communication (classical bits required).
- Teleports the *state* not the *particle*.
- Fidelity in experiment: >90% for photon polarisation states.
