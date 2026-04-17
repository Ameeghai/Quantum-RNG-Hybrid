# 🔬 Hybrid Quantum Random Number Generator (QRNG)

> **B.S. Physics Project** | Qiskit AerSimulator | 32-Qubit System | NIST SP 800-22 Validated  
> **Author:** Amee Ghai — B23PH1003

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Motivation](#-motivation)
- [Circuit Architectures](#-circuit-architectures)
- [Project Structure](#-project-structure)
- [Installation & Setup](#-installation--setup)
- [Usage](#-usage)
- [Results — Report I (Model Comparison)](#-results--report-i-model-comparison)
- [Results — Report II (Noise Robustness)](#-results--report-ii-noise-robustness)
- [NIST SP 800-22 Validation](#-nist-sp-800-22-validation)
- [ANU Hardware Comparison](#-anu-hardware-comparison)
- [Key Findings](#-key-findings)
- [Limitations & Future Work](#-limitations--future-work)
- [References](#-references)

---

## 🧭 Overview

This project designs, implements, and rigorously evaluates a **Hybrid Quantum Random Number Generator (QRNG)** using Qiskit's `AerSimulator`. The system combines classical shot-noise seeding with quantum superposition (Hadamard gates) and entanglement (CNOT gates) to generate cryptographically-grade 32-bit random integers.

Four distinct circuit architectures (Models A–D) are designed and compared across entropy, correlation, and NIST statistical test compliance. The best-performing model (Model D) is then subjected to a full **five-layer noise robustness pipeline** to simulate real hardware error conditions.

**Key outcome:** Model D (Dice-CNOT + Dice-Measurement, 5000 shots) achieves maximum entropy (12.29 bits), zero Shannon–Min entropy gap, and lowest average off-diagonal correlation (0.0110), passing 15/16 NIST SP 800-22 tests and matching ANU real quantum hardware benchmarks.

---

## 💡 Motivation

Classical Pseudo-Random Number Generators (PRNGs) are **deterministic** — given a seed, the sequence is entirely reproducible. This makes them unsuitable for:

- Cryptographic key generation
- Quantum cryptography protocols (BB84, E91)
- Monte Carlo simulations requiring true unpredictability
- Hardware security modules (HSMs)

Quantum mechanics provides a fundamentally different approach. When a qubit is placed in superposition via a **Hadamard gate** and measured, the outcome is **intrinsically random** — not just computationally hard to predict, but physically impossible to predict even in principle. This project exploits that property to build a practical QRNG.

---

## ⚛️ Circuit Architectures

All four models share the same base pipeline:

```
Classical Shot-Noise Seed → X Gates → Hadamard Gates (Superposition) → [Optional CNOT] → Measure → Sample
```

| Model | Name | Dice Ordering | CNOT Entanglement | Description |
|-------|------|:---:|:---:|---|
| **A** | Baseline | ✗ | ✗ | Sequential Hadamard + direct measurement (q0–q31). Reference architecture. |
| **B** | CNOT Layer | ✗ | ✓ | Hadamard + CNOT gates with fixed shuffle-and-pair entanglement structure. |
| **C** | Dice Measure | ✓ | ✗ | Hadamard + randomised dice-based measurement ordering. No entanglement. |
| **D ★** | Full Hybrid | ✓ | ✓ | Dice-based CNOT pairing + dice measurement ordering. Both entanglement and readout fully randomised each run. **Best model.** |

### Model D — Circuit Design Detail

- **32 qubits** (q0–q31) each producing 1 bit → 32-bit integer output per shot
- **Classical seed:** NumPy-generated 32-bit string at `p = 0.5` applied via X gates
- **Hadamard layer:** Primary entropy source — puts each qubit into equal superposition
- **Dice-CNOT:** CNOT target pairs randomised each run using a dice roll — prevents fixed entanglement bias
- **Dice-Measurement:** Qubit readout order randomised each run — adds unpredictability in classical post-processing

---

## 📁 Project Structure

```
QRNG-Project/
│
├── notebooks/
│   ├── Quantum_random_number_generator.ipynb   # Report I: 4 model designs & comparison
│   └── noise_analysis.ipynb                    # Report II: Noise robustness pipeline
│
├── reports/
│   ├── QRNG_Model_Comparison_Report.docx       # Detailed Report I write-up
│   └── Noise_Analysis_Report.docx              # Detailed Report II write-up
│
├── presentation/
│   └── QRNG_Presentation.pdf                   # Full project presentation slides
│
├── results/
│   ├── nist_input.txt                          # Binary output for NIST test suite
│   └── correlation_matrices/                   # Saved correlation heatmaps (PNG)
│
└── README.md
```

---

## 🛠️ Installation & Setup

### Prerequisites

- Python 3.9+
- Qiskit (AerSimulator)
- NumPy, Matplotlib, SciPy

### Install Dependencies

```bash
pip install qiskit qiskit-aer numpy matplotlib scipy
```

### Optional: NIST SP 800-22 Test Suite

Download and compile from the official NIST source:

```bash
# Clone the NIST randomness test suite
git clone https://github.com/dj-on-github/sp800_22_tests.git
cd sp800_22_tests
make
```

Or use the GUI version from [https://csrc.nist.gov/projects/random-bit-generation](https://csrc.nist.gov/projects/random-bit-generation)

---

## 🚀 Usage

### Run Report I — Model Comparison

Open and run all cells in:

```
notebooks/Quantum_random_number_generator.ipynb
```

This notebook:
1. Defines all four circuit architectures (Models A–D)
2. Runs each model at 2000 and 5000 shots
3. Computes Shannon entropy, Min-entropy, and qubit-wise Pearson correlation matrices
4. Plots correlation heatmaps for visual inspection
5. Exports a binary output file for NIST testing

### Run Report II — Noise Analysis

Open and run all cells in:

```
notebooks/noise_analysis.ipynb
```

This notebook:
1. Takes Model D as the base architecture
2. Passes output through a 5-layer noise pipeline (see below)
3. Compares noisy vs. noiseless entropy and correlation metrics
4. Benchmarks against ANU real quantum hardware data

### Quick Example — Generate a Random 32-bit Integer (Model D)

```python
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
import numpy as np

def generate_qrng_modelD(shots=5000):
    n_qubits = 32
    seed_bits = np.random.choice([0, 1], size=n_qubits, p=[0.5, 0.5])
    
    # Dice-based CNOT pairs and measurement order
    qubits = list(range(n_qubits))
    np.random.shuffle(qubits)
    cnot_pairs = [(qubits[i], qubits[i+1]) for i in range(0, n_qubits-1, 2)]
    measure_order = list(range(n_qubits))
    np.random.shuffle(measure_order)
    
    qc = QuantumCircuit(n_qubits, n_qubits)
    
    # Apply seed via X gates
    for i, bit in enumerate(seed_bits):
        if bit == 1:
            qc.x(i)
    
    # Hadamard layer
    for i in range(n_qubits):
        qc.h(i)
    
    # Dice-CNOT entanglement
    for ctrl, tgt in cnot_pairs:
        qc.cx(ctrl, tgt)
    
    # Dice-ordered measurement
    for i, q in enumerate(measure_order):
        qc.measure(q, i)
    
    simulator = AerSimulator()
    job = simulator.run(transpile(qc, simulator), shots=shots)
    counts = job.result().get_counts()
    
    # Sample one bitstring and convert to integer
    bitstring = max(counts, key=counts.get)
    return int(bitstring, 2)

random_number = generate_qrng_modelD()
print(f"Generated 32-bit random number: {random_number}")
```

---

## 📊 Results — Report I (Model Comparison)

### Entropy & Correlation Summary

| Model | Shots | Shannon Entropy (bits) | Min-Entropy (bits) | Avg Off-Diag Correlation | Status |
|-------|:---:|:---:|:---:|:---:|:---:|
| Model A | 2000 | 10.97 | 10.97 | 0.0190 | ✅ Pass |
| Model A | 5000 | 12.29 | 12.29 | 0.0150 | ✅ Pass |
| Model B | 2000 | 10.97 | 10.97 | 0.0180 | ✅ Pass |
| Model B | 5000 | 12.29 | 11.29 | 0.0140 | ⚠️ Partial |
| Model C | 2000 | 10.97 | 10.97 | 0.0170 | ✅ Pass |
| Model C | 5000 | 12.29 | 12.29 | 0.0130 | ✅ Pass |
| **Model D ★** | **2000** | **10.97** | **10.97** | **0.0150** | ✅ **Pass** |
| **Model D ★** | **5000** | **12.29** | **12.29** | **0.0110** | 🏆 **Best** |

> ★ **Model D at 5000 shots** is selected as the best architecture — highest entropy, zero Shannon/Min gap, lowest correlation.

### Qubit-wise Correlation Matrices

All models produce 32×32 Pearson correlation matrices. In a good QRNG, only the diagonal (self-correlation = 1.0) should be red; all off-diagonal elements should be near zero (pale/white). All models achieve this, confirming qubit independence.

**Model B Note:** At 5000 shots, Model B shows a Shannon–Min entropy gap (12.29 vs 11.29 bits), indicating occasional distributional skew introduced by the **fixed** CNOT pairing structure. Randomising the CNOT pairs (Model D) eliminates this.

---

## 📊 Results — Report II (Noise Robustness)

### Five-Layer Noise Pipeline

```
L1: Classical Shot-Noise Seed
    └─ 32-bit NumPy seed, p=0.5 · Per-bit entropy ≈ 0.9994 bits
       ↓
L2: Initialisation Noise (Bit-Flip Error)
    └─ Seed bits flip with p_init = 0.02 (2%) · Models state preparation errors
       ↓
L3: Quantum Circuit — Model D
    └─ Hadamard + Dice-CNOT + Dice-Measurement · Per-bit entropy ≈ 0.9998 bits
       ↓
L4: Measurement Noise (Bit-Flip Error)
    └─ Output bits flip with p_meas = 0.02 (2%) · Models readout error
       ↓
L5: Correlation Analysis
    └─ 32×32 Pearson matrix · Avg off-diagonal ≈ 0.0115
```

### Noisy vs. Noiseless Results

| Metric | Noiseless (`p=0`) | Noisy (`p=0.02`) | Verdict |
|--------|:---:|:---:|:---:|
| Per-Bit Entropy | ≈ 0.9998 bits/bit | ≈ 0.9998 bits/bit | ✅ Identical |
| Min-Entropy | ≈ 31.99 bits | ≈ 31.99 bits | ✅ Identical |
| Shannon Entropy (strings) | ≈ 12.29 bits | ≈ 12.29 bits | ✅ Identical |
| Avg Off-Diagonal Correlation | ≈ 0.0112 | ≈ 0.0115 | ✅ Negligible difference |

> **Key Finding:** The Hadamard + CNOT structure effectively "washes out" low-level (2%) bit-flip noise. The pipeline is noise-robust.

---

## ✅ NIST SP 800-22 Validation

The binary output of Model D (5000 shots) was tested against the full NIST SP 800-22 statistical test suite for random number generators.

| # | Test | Result |
|---|------|:---:|
| 01 | Frequency (Monobit) | ✅ Random |
| 02 | Frequency within Block | ✅ Random |
| 03 | Runs Test | ✅ Random |
| 04 | Longest Run of 1s | ✅ Random |
| 05 | Binary Matrix Rank | ✅ Random |
| 06 | DFT (Spectral) | ✅ Random |
| 07 | Non-overlapping Template Matching | ✅ Random |
| 08 | Overlapping Template Matching | ✅ Random |
| **09** | **Maurer's Universal Statistical** | **⚠️ Non-Random** |
| 10 | Linear Complexity | ✅ Random |
| 11 | Serial Test | ✅ Random |
| 12 | Approximate Entropy | ✅ Random |
| 13 | Cumulative Sums (Forward) | ✅ Random |
| 14 | Cumulative Sums (Backward) | ✅ Random |
| 15 | Random Excursions | ✅ Random |
| 16 | Random Excursions Variant | ✅ Random |

**Result: 15/16 PASS**

### Note on Test 09 (Maurer's Universal)

Test 09 returned `p-value = −1.0`, flagged as Non-Random. This is **not a randomness failure**. A p-value of exactly `−1.0` is a **sentinel value** indicating the test could not execute — the bit sequence was too short. Maurer's Universal test requires a minimum of **387,840 bits** to produce meaningful results. The current output (~160,000 bits at 5000 shots × 32 bits) is insufficient for this specific test. All other 15 tests passed with `p > 0.05`.

To pass Test 09, significantly more quantum shots or real hardware execution would be required.

---

## 🔬 ANU Hardware Comparison

The simulated QRNG output was benchmarked against the **Australian National University (ANU) QRNG**, which generates true random numbers from quantum vacuum fluctuations in a real optical setup. 1024 `uint8` values (8192 bits) were fetched from the ANU API for comparison.

| Metric | Simulated QRNG (Model D) | ANU Real QRNG | Assessment |
|--------|:---:|:---:|:---:|
| Per-Bit Entropy | ≈ 0.9999 bits/bit | ≈ 0.9997 bits/bit | ✅ Both near-ideal |
| Sequential (Lag-1) Autocorrelation | ≈ −0.003 | ≈ +0.009 | ✅ Both near-zero |
| Source | Qiskit AerSimulator | Real optical hardware | ✅ Comparable quality |

> The simulation faithfully models real QRNG behaviour at both per-bit entropy and autocorrelation metrics. The difference between simulated and real hardware is statistically negligible at these metrics.

---

## 🔑 Key Findings

1. **Shot count is the primary driver of entropy.** Increasing shots from 2000 → 5000 consistently improved Shannon entropy from ~10.97 to ~12.29 bits across all models. At 2000 shots, all models appear nearly identical.

2. **The Hadamard layer does the heavy lifting.** The superposition layer generates the majority of randomness. CNOT entanglement and dice-based ordering provide secondary improvements in correlation and distributional uniformity.

3. **Fixed CNOT structure introduces distributional skew.** Model B (fixed CNOT pairs) showed a Shannon–Min entropy gap at 5000 shots (12.29 vs 11.29 bits), indicating bias from fixed entanglement patterns. Randomising CNOT pairs (Model D) eliminates this.

4. **Model D wins on all three metrics.** Maximum entropy (12.29 bits), zero Shannon–Min gap, and lowest correlation (0.0110). Randomised CNOT + randomised measurement ordering prevents systematic patterns.

5. **The pipeline is noise-robust at 2% error rates.** Noisy and noiseless configurations produce virtually identical statistical outputs, confirming that the quantum circuit absorbs low-level hardware noise.

6. **Simulation matches real quantum hardware.** Per-bit entropy ≥ 0.9997 and near-zero autocorrelation in both simulated and ANU real hardware outputs confirm the validity of the simulation approach.

---

## ⚠️ Limitations & Future Work

### Current Limitations

- Noise levels tested (2%) are mild — real quantum hardware error rates can be 5–20%
- Independent bit-flip noise model used; real hardware noise is **correlated** across qubit pairs (crosstalk, coherence decay)
- Per-bit entropy near 1.0 is necessary but not sufficient for full output uniformity
- ANU comparison limited to only 8,192 bits — too small for rigorous statistical inference
- Test 09 (Maurer's Universal) could not be evaluated due to insufficient sample size

### Future Work

- [ ] Re-run noise analysis at 5%, 10%, 20% error rates to identify degradation threshold
- [ ] Implement correlated, structured noise models matching real IBM Quantum chip topologies
- [ ] Run full NIST SP 800-22 suite with a much larger sample (1M+ bits)
- [ ] Deploy on real IBM Quantum hardware (via IBM Quantum Network) for direct physical validation
- [ ] Explore qubit count scaling (64, 128-qubit systems) for higher-bit output
- [ ] Investigate post-processing extractors (e.g., Toeplitz hashing) to further whiten output

---

## 📚 References

1. Nielsen, M. A., & Chuang, I. L. (2010). *Quantum Computation and Quantum Information*. Cambridge University Press.
2. NIST SP 800-22 Rev 1a — *A Statistical Test Suite for Random and Pseudorandom Number Generators for Cryptographic Applications*. [https://csrc.nist.gov/publications/detail/sp/800-22/rev-1a/final](https://csrc.nist.gov/publications/detail/sp/800-22/rev-1a/final)
3. Qiskit Documentation — AerSimulator. [https://qiskit.github.io/qiskit-aer/](https://qiskit.github.io/qiskit-aer/)
4. ANU Quantum Random Numbers Server. [https://qrng.anu.edu.au/](https://qrng.anu.edu.au/)
5. Maurer, U. M. (1992). A universal statistical test for random bit generators. *Journal of Cryptology*, 5(2), 89–105.
6. Ma, X., et al. (2016). Quantum random number generation. *npj Quantum Information*, 2, 16021.

---

## 📄 License

This project was developed as a B.S. Physics project at IIT Jodhpur. For academic and research use only.

---

<div align="center">
  <sub>Built with ⚛️ Qiskit · Python · NumPy · Matplotlib</sub>
</div>
