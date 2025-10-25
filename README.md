# 🌀 Hybrid Quantum Random Number Generator (QRNG)

### 📘 Project Overview

This project implements a **Hybrid Quantum Random Number Generator (QRNG)** using a novel blend of **classical shot noise simulation**, **quantum superposition**, and **dice-based logical randomness**.
The goal is to enhance true randomness by integrating multiple independent layers of entropy within a quantum framework built using **Qiskit**.

---

### ⚛️ Approach

The hybrid QRNG operates on three levels of randomness:

1. **Classical Randomness (Shot Noise Simulation)**

   * The generator begins with a simulated 32-bit electronic shot-noise string, representing the random fluctuations of current at the microscopic level.
   * This classical randomness forms the initial state distribution for the quantum system.

2. **Quantum Superposition and Entanglement**

   * Each qubit is initialized according to the shot-noise bitstring and placed into a uniform superposition using Hadamard gates.
   * Controlled-NOT (CNOT) gates are optionally applied to introduce **quantum correlations**, further enhancing unpredictability.

3. **Dice-Based Logical Randomness**

   * A virtual “dice logic” layer randomly determines qubit-pair interactions (CNOT pairings) and measurement order, ensuring that circuit evolution itself is dynamically randomized during every run.

This **three-tier hybrid randomness generation** approach ensures that every execution produces new, statistically distinct random numbers.

---

### 💡 Why This Project is Unique

* Combines **three independent entropy sources** — classical noise, quantum mechanics, and algorithmic logic — in a single model.
* Implements **randomized circuit structure** (through dice logic) — an unconventional addition to typical QRNG architectures.
* Focuses on **quantitative analysis** through entropy, correlation heatmaps, and statistical distribution plots.
* Fully **simulation-based and reproducible**, yet adaptable for hardware deployment on IBM Quantum devices.
* Bridges **classical stochastic modeling** and **quantum indeterminacy**, serving as an educational as well as technical innovation.

---

### 🧩 Key Components

* **Shot Noise Generator** – Simulates electron-level current fluctuations using `numpy.random.choice()`.
* **Quantum Circuit (Qiskit)** – 32-qubit circuit implementing Hadamard superposition and CNOT-based entanglement.
* **Dice Logic Engine** – Randomizes qubit interactions and measurement sequences.
* **Statistical Analysis Suite** – Evaluates randomness using Shannon entropy, min-entropy, and correlation matrices.

---

### 🧬 Conclusion and Final Outcome

The Hybrid QRNG demonstrates that **layered randomness**—combining classical, quantum, and logical domains—produces more uniform, less correlated random sequences.
Through this integration, the generator showcases **quantum-verified unpredictability** and supports future work toward **quantum-secure randomness generation**.

This project stands as a strong example of using **simulation-based quantum modeling** to explore randomness at the intersection of physics and computation.
It highlights how classical noise can seed quantum processes to create high-quality random numbers suitable for applications in **cryptography, quantum communication, and secure computing**.

---

### 🧑‍🔬 Author

**Amee Ghai**
B.S. Physics (3rd Year), Indian Institute of Technology Jodhpur
Specialization: Quantum Technologies | Focus: Quantum Optics & Quantum Computing
📄 Project developed for Hackathon (QRNG Track)
