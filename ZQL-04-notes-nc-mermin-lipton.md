# ZQL-04 Step 1 Notes — Nielsen & Chuang Ch.5–6 + Mermin + Lipton & Regan

---

# A. Nielsen & Chuang — Ch. 5: The Quantum Fourier Transform and Phase Estimation

## 5.1 The Quantum Fourier Transform (QFT)

The QFT is the quantum analogue of the Discrete Fourier Transform (DFT):

**Classical DFT**: xₖ → X_j = (1/√N) Σₖ xₖ · e^{2πijk/N}

**Quantum QFT**: |j⟩ → (1/√N) Σₖ e^{2πijk/N} |k⟩

For n qubits (N=2ⁿ):
```
QFT|j⟩ = (1/√2ⁿ) Σₖ₌₀^{2ⁿ-1} e^{2πijk/2ⁿ} |k⟩
```

### Product Representation (N&C §5.1)
QFT has an efficient product form:
```
QFT|j₁j₂...jₙ⟩ = (|0⟩+e^{2πi·0.jₙ}|1⟩) ⊗ (|0⟩+e^{2πi·0.jₙ₋₁jₙ}|1⟩) ⊗ ... ⊗ (|0⟩+e^{2πi·0.j₁j₂...jₙ}|1⟩)
```

Gates required: n Hadamards + n(n-1)/2 controlled-phase gates → **O(n²) gates**, O(n) depth with parallelism.

Classical FFT: O(n·2ⁿ) operations. QFT: O(n²) gates. Exponential speedup in gate count.

**Circuit construction** (3-qubit example):
1. H on q[0], then R₂ controlled from q[1], R₃ from q[2]
2. H on q[1], then R₂ controlled from q[2]
3. H on q[2]
4. SWAP q[0] ↔ q[2] (bit reversal)

```python
# SDK: build 3-qubit QFT
from zuntenium import QuantumCircuit
import math

def qft_circuit(n: int) -> QuantumCircuit:
    """Build n-qubit QFT circuit (without final SWAP)."""
    qc = QuantumCircuit(n, name=f"QFT_{n}")
    for j in range(n):
        qc.h(j)
        for k in range(j+1, n):
            angle = 2 * math.pi / (2 ** (k - j + 1))
            qc.crz(angle, k, j)   # Controlled-Rz (phase gate)
    # Bit reversal
    for i in range(n // 2):
        qc.swap(i, n - 1 - i)
    return qc
```

## 5.2 Phase Estimation (QPE)

**Problem**: Given unitary U and eigenstate |u⟩ (U|u⟩ = e^{2πiφ}|u⟩), estimate φ.

**Circuit**: t control qubits + n eigenstate qubits.
1. Initialise: t control qubits in |0⟩, eigenstate qubits in |u⟩
2. Apply H⊗t to control register
3. Apply controlled-U^{2^j} for j=0...t-1
4. Apply inverse QFT to control register
5. Measure control register → binary approximation of φ

**Precision**: t control qubits → φ accurate to 2⁻ᵗ.

**[SDK]** `qc.cx`, `qc.crz`, `qc.crx`, `qc.cry`, `qct.ccx` are the building blocks. QPE is a key subroutine in ZQL-05 (VQE) — the PauliHamiltonian eigenvalue estimate uses related ideas.

---

# B. Nielsen & Chuang — Ch. 6: Quantum Search

## 6.1 Grover's Algorithm

**Problem**: Find marked item in unsorted database of N=2ⁿ items. Classical: O(N). Quantum: O(√N).

**Setup**: Oracle Uf: |x⟩|y⟩ → |x⟩|y⊕f(x)⟩ where f(x)=1 only for the target.

**Phase oracle** (using kickback with |y⟩=|−⟩): Uf|x⟩|−⟩ = (−1)^{f(x)}|x⟩|−⟩. Marks target with −1 phase.

**Grover iteration** (one amplitude amplification step):
1. Apply oracle O (marks target state with −1)
2. Apply diffusion operator D = 2|ψ⟩⟨ψ|−I (reflect about uniform superposition)
   - D = H⊗n · (2|0⟩⟨0| − I) · H⊗n

**Number of iterations**: ⌊π√N/4⌋ ≈ π√N/4.

**SDK building blocks**:
```python
# Diffusion operator for n qubits
def diffusion(n: int) -> QuantumCircuit:
    qc = QuantumCircuit(n, name="diffusion")
    qc.h(list(range(n)))    # H⊗n
    qc.x(list(range(n)))    # Flip
    # Multi-controlled Z (phase on |11...1⟩)
    qc.h(n-1)
    if n == 2:
        qc.cx(0, 1)
    elif n == 3:
        qc.ccx(0, 1, 2)
    qc.h(n-1)
    qc.x(list(range(n)))    # Flip back
    qc.h(list(range(n)))    # H⊗n
    return qc
```

---

# C. Mermin — *Quantum Computer Science: An Introduction*

## Clean Notation for QFT
Mermin uses the "hat" notation: let ω_N = e^{2πi/N}. Then:
- QFT: |j⟩ → N^{-1/2} Σₖ ω_N^{jk} |k⟩
- Compact, clean — good for pencil-and-paper verification.
- **[Teaching]** Great for ZQL-04 exercises: students write out QFT on 2–3 qubits by hand using ω notation.

## Shor's Algorithm Framework (Ch. 6)
- Shor's problem: factor N into p·q. Reduces to: find the *order* r of a∈ℤ_N (smallest r s.t. aʳ≡1 mod N).
- Once r is found: gcd(a^{r/2}−1, N) gives a factor with high probability.
- Order finding uses QPE with unitary U: |y⟩ → |ay mod N⟩.
- **[Teaching]** ZQL-04 treats Shor conceptually — quantum circuit arithmetic is ZQL-03 content.

---

# D. Lipton & Regan — *Quantum Algorithms via Linear Algebra*

## Linear-Algebra-First Approach
- Every algorithm step is a matrix multiplication. No physics.
- QFT = Fourier matrix F_N. Grover = product of two orthogonal reflections.
- Grover diffusion matrix D = 2vvᵀ−I where v = N^{-1/2}·(1,1,...,1)ᵀ.

## Key Results
- **Grover analysis**: after k iterations, amplitude of target = sin((2k+1)θ) where sin(θ) = 1/√N.
- Optimal k ≈ π/(4θ) ≈ π√N/4.
- **[SDK]** We can verify this amplitude amplification formula in code using the statevector simulator.

## Amplitude Amplification (generalised Grover)
- Works for any quantum algorithm A where P(success) = p. Reduces queries from O(1/p) to O(1/√p).
- Used in quantum Monte Carlo, QAOA warm starts, and quantum optimisation.
