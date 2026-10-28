# ZQL-02B Step 1 Notes — Tanenbaum (Memory) + Gottesman (Stabiliser) + Johansson/Gorelick (NumPy)

---

# A. Tanenbaum — *Computer Organization and Architecture*

**Relevant: memory layouts, cache hierarchy, O(2^n) analysis for quantum simulators**

## Memory Hierarchy
- L1 cache: ~4 cycles, 32–64 KB. L2: ~12 cycles, 256 KB. L3: ~40 cycles, 4–32 MB. DRAM: ~200 cycles.
- Cache line: 64 bytes = 8 doubles = 4 complex128 amplitudes.
- **[SDK]** For n=4 qubits: statevector = 16 complex128 = 256 bytes — fits in L1. For n=17: 128MB — cache-unfriendly. This is why `MAX_DM_QUBITS=15` and simulator performance degrades sharply above n~20.

## Memory Layouts for Quantum Simulation

| Simulator | Memory | Layout | RAM at n=20 |
|---|---|---|---|
| `StatevectorSimulator` | O(2ⁿ) complex128 | 1D C-contiguous array | 8 MB |
| `DensityMatrixSimulator` | O(4ⁿ) complex128 | 2D C-contiguous matrix | 8 GB (n=20 impossible) |
| `TensorNetworkBackend` | O(2ᵏ·m) | Compressed tensors | Much smaller for low-entanglement |
| `StabilizerSimulator` | O(n²) bits | BinaryTableau | ~negligible |

## Data Types and Precision
- `complex64` (float32): 8 bytes/amplitude. Halves memory; but ~1e-7 precision. Risk: unitarity drift.
- `complex128` (float64): 16 bytes/amplitude. Standard choice. Precision: ~1e-15.
- **[SDK]** QuFlow uses `np.complex128` (confirmed in `GateLibrary` and `DensityMatrix`). Switching to float32 would double capacity but risk numerical errors.

## Cache-Aware Gate Application
- For large circuits (n > 15), cache misses dominate. Strategies:
  1. Process amplitudes in blocks that fit in L3 cache.
  2. Reorder qubit axes so target qubit is innermost (stride-1 access).
  3. Use SIMD via `np.einsum` (numpy calls BLAS/LAPACK with AVX2).

---

# B. Aaronson & Gottesman — *Improved Simulation of Stabilizer Circuits* (Phys. Rev. A, 2004)

**Primary reference for `BinaryTableau` / Clifford stabilizer simulator**

## The Gottesman-Knill Theorem
- **Theorem**: Any quantum circuit consisting only of Clifford gates (H, S, CX, Pauli X/Y/Z, measurement) can be simulated classically in O(n²) time and O(n²) space.
- Clifford gates map Pauli operators to Pauli operators under conjugation (they stabilise the Pauli group).

## Stabilizer Formalism
- A state |ψ⟩ is described by its **stabilizer group**: the set of Pauli operators P such that P|ψ⟩ = |ψ⟩.
- n-qubit state → n independent stabilizers, each an n-qubit Pauli string.
- Instead of tracking 2ⁿ amplitudes, track n Pauli strings → O(n²) bits.

## Binary Tableau (Aaronson & Gottesman's contribution)
- Represent the stabilizer group as a 2n×(2n+1) binary matrix (the "tableau"):
  - Each row = one stabilizer generator (n X-bits + n Z-bits + 1 phase bit)
  - Row operations (elementary) implement gate actions

```python
# BinaryTableau storage for n qubits:
# Shape: (2n, 2n+1) bool/uint8
# Columns: x0..x_{n-1} z0..z_{n-1} phase
# n=4: 8 rows × 9 columns = 72 bits ≈ 9 bytes!
# Compare: statevector for n=4 = 16 complex128 = 256 bytes
```

## Gate Operations on Tableau
- H on qubit k: swap the x and z columns for qubit k (+ phase update). O(n).
- CX(c,t): add ctrl x-column to targ x-column; add targ z-column to ctrl z-column. O(n).
- Measurement: check if target Pauli commutes with each stabilizer. O(n²).

## **[SDK]** `StabilizerSimulator` / `BinaryTableau`
- Located in `backends/stabilizer.py` (18,582 bytes).
- Only Clifford circuits can run here — non-Clifford (T gate, Rz with non-Clifford angle) must use statevector.
- Benefit: simulates 1000+ qubit Clifford circuits on a laptop.

---

# C. Johansson + Gorelick & Ozsvald — *Numerical Python* + *High Performance Python*

## NumPy Memory Layouts (Johansson — Numerical Python, 3rd ed.)

### Array Ordering
- **C-order (row-major)**: default in NumPy. `arr[i,j]` — j is innermost → row elements are contiguous.
- **Fortran-order (column-major)**: `arr[i,j]` — i is innermost.
- **[SDK]** Statevectors are 1D C-contiguous. Density matrices are 2D C-contiguous (2^n × 2^n).

### dtype Management
- `np.zeros(2**n, dtype=np.complex128)` — allocates 16·2^n bytes.
- Creating wrong dtype and casting is expensive: always specify dtype at creation.
- `np.ascontiguousarray(arr)` — ensures C-contiguous layout (important before BLAS calls).

### np.einsum and Broadcasting
- `np.einsum('ij,jk->ik', A, B)` — matrix multiply via contraction.
- `np.einsum('ab,bc->ac', gate, state)` — gate contraction (avoids building full matrix).
- Performance: numpy's einsum uses optimised BLAS routines for large matrices.

## High Performance Python (Gorelick & Ozsvald)

### Memory Bandwidth Bottleneck
- For n=25 statevector: 256MB — won't fit in L3 cache. Each gate access = DRAM load.
- Solution: **re-ordering qubits** to make the target qubit innermost → stride-1 access pattern.
- `np.transpose()` reorders axes at cost of likely making array non-contiguous.

### `np.einsum` vs Manual Loops
- Manual Python loop: ~100 ns/iteration. `np.einsum`: ~5 ns/operation (20x faster).
- **[SDK]** All gate applications should use `np.einsum` or vectorised NumPy — never Python-level loops over amplitude indices for n > 4.

### Profiling Memory Allocations
```python
import tracemalloc
tracemalloc.start()
# ... run simulation ...
current, peak = tracemalloc.get_traced_memory()
print(f"Peak: {peak / 1024**2:.1f} MiB")
tracemalloc.stop()
```

## Summary: Memory Scaling Cheatsheet

| n qubits | Statevector | Density Matrix | BinaryTableau |
|---|---|---|---|
| 10 | 16 KB | 16 MB | 80 bytes |
| 20 | 16 MB | 16 TB (!) | 320 bytes |
| 30 | 16 GB | impossible | 720 bytes |
| 1000 | impossible | impossible | ~125 KB |
