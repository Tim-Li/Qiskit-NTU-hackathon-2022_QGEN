# Quantum Hackathon 2022
## QGEN: An Improved Approach to VLSI Placement Optimization

### Overview

QGEN explores quantum algorithms for a graph-based VLSI placement optimization problem. The repository contains an executable Jupyter notebook (`code/VLSI.ipynb`) that implements problem encoding to Ising Hamiltonians and compares several quantum optimization approaches (VQE, QAOA, Grover-based optimization and CVaR-enhanced VQE) against a classical exact eigensolver baseline.

### Technical Challenges

- Mapping VLSI placement constraints to a compact Ising/Quadratic program formulation that is amenable to near-term quantum algorithms.
- Balancing problem terms (placement vs. penalty terms) requires normalization/ratio hyperparameters; the notebook implements both an "unnormalized" Hamiltonian and a parameterized `normalized_h(adj_mtx, ratio)` to study trade-offs.
- Noisy hardware/imperfect optimizers and limited circuit depth affect solution quality; the notebook uses simulators for experiments and compares multiple classical optimizers.

### Method design

- Problem encoding: an input adjacency/list representation is converted to an adjacency matrix and then into Pauli operators (Ising Hamiltonian). Two Hamiltonian variants are provided:
	- `unnormalized_h(adj_mtx)` — direct Ising encoding used as a baseline.
	- `normalized_h(adj_mtx, ratio)` — mixes a penalty term and a weighted interaction term controlled by `ratio` to tune relative strengths.
- Solvers and algorithms evaluated:
	- Classical exact solver via `NumPyMinimumEigensolver` (used through `MinimumEigenOptimizer`).
	- VQE experiments using `VQE` + `MinimumEigenOptimizer` with a `TwoLocal` ansatz; multiple classical optimizers are tested (`SLSQP`, `COBYLA`, `NELDER_MEAD`, `POWELL`) and circuit depths (reps).
	- QAOA experiments via `QAOA` wrapped in `MinimumEigenOptimizer`, again sweeping optimizers and depths.
	- Grover-based optimization using `GroverOptimizer`.
	- CVaR-based VQE (confidence levels `alpha`), implemented with `CVaRExpectation` to evaluate risk-aware objectives.
- Metrics and outputs: the notebook records solution probabilities, expectation (cost) values and computes an error-rate metric (relative to a classical expectation value). Target bitstrings used in analysis are `10011010` and `01100101` (domain-specific examples in the demo graph).

### Installation

Prerequisites:
- Python 3.8+ (recommended)
- A working Python environment with Jupyter Notebook/Lab

Basic Python dependencies (install with pip):

```bash
pip install 'qiskit<1.0' qiskit-optimization qiskit-aer ibm-quantum-widgets numpy matplotlib
```

> **Note:** The notebook uses Qiskit 0.x APIs (`qiskit.opflow`, `qiskit.algorithms`, `IBMQ`). These were refactored in Qiskit 1.0, so pinning to `qiskit<1.0` is recommended for compatibility.

Notes:
- The notebook calls `IBMQ.load_account()` — to run IBMQ-related cells you must configure your IBM Quantum account credentials (see Qiskit docs) or skip those cells if running locally on Aer simulators.
- Some experiments save/load data under `data/` and figures under `graph/`; ensure the notebook's working directory contains those folders or create them before running the full sweep.

### Results (summary)

- The notebook performs parameter sweeps over optimizer types, circuit depth (p-depth / reps), and a family of normalization `ratio` values. For each experiment it saves per-run probabilities and cost values and produces plots comparing:
	- Probability of finding the intended solutions (`10011010` and `01100101`).
	- Expectation value (cost) vs. depth.
	- Error rate = (expectation - classical_expectation) / |classical_expectation| as a normalized performance metric.
- Output artifacts: example graphs are saved under `graph/` (for VQE/QAOA/Grover comparisons) and numeric results under `data/` and `data_normalized/` folders.
- High-level observation from the experiments (as implemented): normalized Hamiltonians and varying the `ratio` parameter can change the convergence behaviour; different classical optimizers and circuit depths produce measurable differences in probability and error metrics. The notebook includes plotting code to visualize these comparisons.

### Usage

- Open and run `code/VLSI.ipynb` with Jupyter. Start by running cells in order; they are grouped as:
	1. imports and helper functions (`adjacency_matrix`, `unnormalized_h`, `normalized_h`, `output`)
	2. classical baseline solving
	3. VQE experiments and plotting
	4. QAOA experiments and plotting
	5. Grover and CVaR experiments
- To reproduce full sweeps and plots, ensure `data/`, `data_normalized/`, and `graph/` directories exist and have write permission.

### Repository Structure

```
.
├── README.md
└── code/
    └── VLSI.ipynb   # main experiment notebook
```
