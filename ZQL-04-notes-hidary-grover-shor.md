# ZQL-04 Step 1 Notes — Hidary (Algorithms) + Grover 1996 Paper + Shor 1997 Paper

---

# A. Hidary — Algorithm Chapters

## QFT Chapter
- Hidary frames QFT as the "hammer" of quantum computing: once you have QFT, many problems become tractable.
- Key insight: QFT transforms from computational basis |j⟩ to Fourier basis; the periodicity of phase encodes arithmetic structure.
- Practical note: QFT alone doesn't give speedup — it must be paired with a state-preparation subroutine (e.g., QPE) that puts useful information in the phase.

## Grover's Search Chapter
- Hidary's framing: "Grover is a quantum flashlight — it amplifies the amplitude of what you're looking for."
- The oracle is the key: it's a black box that marks the solution. For Grover to work, evaluating the oracle must be efficient.
- Practical limit: Grover requires O(√N) quantum gates + O(√N) circuit executions. For N=2ⁿ qubits, √N = 2^{n/2}.
- Example applications: searching protein-folding databases, constraint satisfaction problems.

## Shor's Algorithm Chapter (conceptual)
- Hidary's framing: "Shor's algorithm is a magic trick that turns factoring into period-finding."
- Classical reduction: factor N → find order of random a∈ℤ_N. Classical hardness: best = sub-exponential.
- Quantum speedup: QPE finds the order in O((log N)³) gates. Breaks RSA-2048 with ~4000 logical qubits.
- SDK relevance: modular exponentiation circuit (`|x⟩→|aˣ mod N⟩`) uses quantum arithmetic from ZQL-03.

---

# B. Grover 1996 — *A Fast Quantum Mechanical Algorithm for Database Search*

**Primary source: arXiv:quant-ph/9605043**

## Core Idea
- Problem: N items, one marked item, oracle evaluations only.
- Classical: O(N) evaluations (must check each).
- Quantum: O(√N) evaluations using amplitude amplification.

## The Algorithm
1. Initialise: |ψ⟩ = H⊗n|0ⁿ⟩ (uniform superposition)
2. Repeat O(√N) times:
   a. Oracle: |x⟩ → −|x⟩ if x is target; else |x⟩ (phase flip)
   b. Inversion about average (diffusion): |ψ⟩ → 2⟨ψ|ψ⟩|ψ⟩−|ψ⟩
3. Measure: get marked item with probability ≈ 1.

## Grover's Key Insight
- Each iteration rotates the state vector by 2θ in the 2D subspace {|target⟩, |non-target⟩}.
- After ≈π/(4θ) ≈ π√N/4 iterations, the state vector points at |target⟩.

## Amplitude Formula
- Let |marked⟩ amplitude = sin(φₖ) after k iterations.
- φₖ = (2k+1)·arcsin(1/√N) ≈ (2k+1)/√N for large N.
- At optimum k*: φ_{k*} ≈ π/2 → P(success) ≈ 1.

## Historical Note
- Published in STOC 1996. First practical quantum speedup for unstructured search.
- Proved optimal: Ω(√N) queries are necessary for any quantum algorithm.

---

# C. Shor 1997 — *Polynomial-Time Algorithms for Prime Factorization and Discrete Logarithms*

**Primary source: SIAM J. Comput. 26(5):1484–1509, 1997 (arXiv:quant-ph/9508027)**

## The Problem
- **Factoring**: given N, find p, q such that N = p·q.
- Best classical: General Number Field Sieve — sub-exponential: exp(O(n^{1/3}·(log n)^{2/3})) where n=log N.
- Shor's quantum: polynomial in n = O((log N)³).

## The Reduction: Factoring → Order Finding
1. Pick random a with 1 < a < N and gcd(a,N)=1.
2. Find the **order** r: smallest r>0 with aʳ ≡ 1 (mod N).
3. If r is even and a^{r/2} ≢ −1 (mod N):
   - Let x = a^{r/2} mod N.
   - gcd(x−1, N) and gcd(x+1, N) give non-trivial factors with probability ≥ ½.

## Quantum Order Finding
- Define unitary U_a: |y⟩ → |ay mod N⟩.
- Eigenstates of U_a: |u_s⟩ = (1/√r) Σⱼ e^{-2πisj/r} |aʲ mod N⟩ with eigenvalue e^{2πis/r}.
- QPE on U_a with eigenstate → measures s/r → continued fractions → extract r.

## Circuit Resources
- n = ⌈log₂N⌉ bits. QPE register: 2n qubits. Modular exponentiation: O(n³) gates.
- Total: ~4000 logical qubits for RSA-2048.
- NISQ implication: Shor is beyond current NISQ hardware; needs fault tolerance (ZQL-11).

## SDK mapping
| Shor component | QuFlow element |
|---|---|
| Modular exponentiation circuit | `qc.ccx` chains (ZQL-03 arithmetic) |
| QPE phase kickback | `qc.crz` controlled-phase gates |
| Inverse QFT | `qft_circuit(n).inverse()` |
| Continued fractions | Classical post-processing (Python) |
