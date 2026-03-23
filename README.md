# Grover's Algorithm — 10-Qubit Implementation in Qiskit

A from-scratch implementation of Grover's quantum search algorithm on a 10-qubit (1024-state) search space. Built entirely with primitive Qiskit gates — no `qiskit.algorithms.Grover` used.

**Marked states:** `0110011010` and `1101010001`

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Installation](#installation)
- [How to Run](#how-to-run)
- [Implementation Details](#implementation-details)
- [Expected Output](#expected-output)
- [Changing the Marked States](#changing-the-marked-states)

---

## Overview

Grover's algorithm provides a quadratic speedup for unstructured search. For a search space of `N = 2^n` states with `M` marked states, the optimal number of iterations is approximately:

$$k_{\text{opt}} = \left\lfloor \frac{\pi}{4} \cdot \sqrt{\frac{N}{M}} \right\rfloor$$

For this project: `n = 10`, `N = 1024`, `M = 2` → **optimal ≈ 25 iterations**.

The notebook is divided into two parts that each implement Grover's algorithm independently, then compare results:

| | Part A | Part B |
|---|---|---|
| **Name** | IBM Reference Subroutine | Custom Subroutine |
| **Oracle** | `MCMTGate(ZGate())` wrapper | `H + mcx + H` decomposition |
| **Diffusion** | `MCMTGate(ZGate())` wrapper | `H + X + mcx + X + H` (primitive) |
| **Result** | Identical amplification | Identical amplification |

Both are mathematically equivalent and produce the same measurement histogram.

---

## Project Structure

```
.
├── Grover_10Qubit_Project.ipynb   # Main notebook (Parts A and B)
├── README.md                      # This file
```

Output files generated when the notebook is run:

```
grover_partA_histograms.png        # Histograms for Part A across iteration counts
grover_partB_histograms.png        # Histograms for Part B across iteration counts
grover_comparison.png              # Side-by-side Part A vs Part B at optimal iterations
```

---

## Requirements

- Python 3.8+
- qiskit >= 1.0
- qiskit-aer
- matplotlib
- pylatexenc *(for circuit drawing with `output="mpl"`)*

---

## Installation

**1. Clone the repository**

```bash
git clone <your-repo-url>
cd <repo-folder>
```

**2. (Recommended) Create a virtual environment**

```bash
python -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate           # Windows
```

**3. Install dependencies**

```bash
pip install qiskit qiskit-aer matplotlib pylatexenc
```

---

## How to Run

**Option A — Jupyter Notebook (recommended)**

```bash
jupyter notebook Grover_10Qubit_Project.ipynb
```

Then run all cells: **Kernel → Restart & Run All**

**Option B — JupyterLab**

```bash
jupyter lab Grover_10Qubit_Project.ipynb
```

**Option C — VS Code**

Open the `.ipynb` file in VS Code with the Jupyter extension installed and click **Run All**.

**Option D — Command line (convert and run)**

```bash
jupyter nbconvert --to script Grover_10Qubit_Project.ipynb
python Grover_10Qubit_Project.py
```

> The notebook uses `AerSimulator` (local simulation) — no IBM Quantum account or cloud access is required.

---

## Implementation Details

### 1. Superposition

All 10 qubits are initialized to `|0⟩` and Hadamard gates are applied to create an equal superposition over all 1024 states.

### 2. Oracle

The oracle applies a phase flip of `−1` to each marked state, leaving all others unchanged. For each target bit-string:

1. The bit-string is reversed to match Qiskit's qubit ordering (`q0` = LSB).
2. `X` gates are applied to qubits where the target has a `'0'` (open-control trick).
3. A multi-controlled-Z is applied across all 10 qubits.
4. The `X` gates are reversed to restore the register.

**Part A** implements the MCZ using `MCMTGate(ZGate(), n-1, 1)`.  
**Part B** decomposes it as `H(target) → mcx(controls, target) → H(target)`.

### 3. Diffusion Operator

The diffusion operator `D = H⊗ⁿ · U₀ · H⊗ⁿ` performs inversion about the mean.  
`U₀` flips the phase of every state except `|00...0⟩`, implemented as:

```
X(all) → Multi-controlled-Z → X(all)
```

### 4. Grover Iterations

One Grover iteration = Oracle + Diffusion. The circuit is run for several iteration counts to observe amplitude amplification over time:

| Iterations | Expected behaviour |
|---|---|
| 1 | Slight elevation above uniform background |
| 3 | Visible amplification, not dominant |
| 5 | Further amplification |
| 10 | Strong amplification, may overshoot |
| ~25 (optimal) | Marked states dominate with ~40–50% probability each |

### 5. Simulation

Circuits are transpiled and run on `AerSimulator` (local statevector/shot-based simulator) with **4096 shots**.

---

## Expected Output

When all cells are executed, the notebook will:

1. Print the optimal iteration count and per-iteration probabilities for both marked states.
2. Display and save histograms showing measurement results for each iteration count.
3. Confirm that at optimal iterations both `0110011010` and `1101010001` appear with the highest probability (~40–50% each, combined ~80–100%).

Example console output at optimal iterations:

```
Part A (IBM):
  0110011010  →  48.XX%
  1101010001  →  47.XX%
  Combined probability of marked states: 95.XX%

Part B (Custom):
  0110011010  →  48.XX%
  1101010001  →  47.XX%
  Combined probability of marked states: 95.XX%
```

Histogram bars for the two marked states will appear highlighted in red (Part A) and green (Part B), clearly dominating all other states.

---

## Changing the Marked States

To search for different bit-strings, edit the `TARGETS` list in the **Imports & Setup** cell:

```python
TARGETS = ["0110011010", "1101010001"]
```

Replace with any two (or more) 10-character binary strings, for example:

```python
TARGETS = ["1010101010", "0101010101"]
```

All downstream cells — oracle, diffusion, iteration count, histograms — update automatically. No other changes are needed.

---

## References

- [IBM Quantum Learning — Grover's Algorithm](https://quantum.cloud.ibm.com/learning/en/modules/computer-science/grovers)
- [Qiskit Documentation](https://docs.quantum.ibm.com/)
- Nielsen & Chuang, *Quantum Computation and Quantum Information*, Chapter 6
