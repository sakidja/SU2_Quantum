# SU(2) Neutrino QC

Simulation and analysis code for the preprint:

**"Dynamical Generation of Neutrino Mass from Local SU(2) Gauge Variance"**
R. Sakidja, Zenodo (2026). DOI: [10.5281/zenodo.23111769](https://doi.org/10.5281/zenodo.23111769)

The paper asks whether the neutrino mass has a geometric origin in the SU(2) gauge field itself. The imaginary components of the gauge field average to zero, and that zero has long been read as absence. Their variance does not vanish. It is the Yang–Mills energy density, term by term, and it is fixed by the geometry of the group through the exact relation

    Σ_k Var(c_k) = 1 − ⟨c_0²⟩

The results are presented in two stages:

* **Stage 1, the bound.** A collective observable such as Z⊗N takes only the values ±1, so Var(Z⊗N) = 1 − ⟨Z⊗N⟩². With the mean at zero the variance sits at 1, the maximal fluctuation: the zero mean hides the fluctuation completely rather than removing it. This is also the value Σ_k Var(c_k) reaches for pure curvature, ⟨c_0²⟩ = 0.
* **Stage 2, the complete calculation.** Reading the Pauli coefficients directly from plaquettes built from Haar random links gives ⟨c_k⟩ ≈ 0 and Σ_k Var(c_k) = 3/4 exactly, on 3×3, 4×4, 5×5, 2×2×2 and 3×3×3 lattices, and the same values are recovered on qubits with a Hadamard test. In the classical Wilson ensembles at β = 2.2 the same quantity is 0.6172 ± 0.0010, identical at L = 16, 24 and 32. A neutrino probe qubit entangled with the gauge register shows exactly zero variance with the field off and a variance that grows with Σ_k Var(c_k) with the field on.

## Repository layout

```
analysis/    Post-processing and figure scripts (laptop-scale, no MPI)
data/        Summary JSONs from the production campaigns
quantum/     Quantum simulation scripts (planar and cubic lattices)
classical/   Classical SU(2) Wilson studies: the S6 twist campaign (twist/), 3D vortex lock (topology/), 4D (topology4d/)
```

## Quantum simulations (`quantum/`)

| Script | Description |
|--------|-------------|
| `su2_lattice_scaling.py` | MPI scaling script for L×L planar lattices (9/16/25 qubits); `--cubic` mode for 2×2×2 and 3×3×3 (8/27 qubits); `--alpha` sets the excitation amplitude; seeded, checkpoint every 50 samples, lossless resumption (`--resume`), recovery of timed out runs (`--merge-checkpoints`) (Sections 4.5, S5) |
| `submit_su2_scaling.sh` | Slurm template for the scaling runs: L = 3, 4, 5 with 10,000 samples each, 32 MPI ranks on one CPU node (Section S5) |
| `su2_paths_collapse.py` | Energy dependence: three injection protocols (uniform, Gaussian, binary) over an α grid, and the collapse of mass onto variance; Figures 14 and 15 (Sections 4.6, S5) |
| `SU2_Nutrino_Mass_QC_3x3_v2.ipynb` | Stage 1, 9 qubit 3×3 lattice, Figures 1, 2, 6, 7 and 9 of the paper (the notebook titles use earlier numbers); seed 42 |
| `SU2_Nutrino_Mass_QC_2x2x2_v2.ipynb` | Stage 1, 8 qubit 2×2×2 lattice, Figures S1 to S6 (S6 drawn as a schematic) |
| `stage2/su2_stage2_haar.ipynb` | Stage 2: Haar sampled plaquettes on five lattices, Hadamard test readout of c_k, neutrino probe with the field on and off; Figures 3, 4, 5 and 8 |
| `stage2/su2_haar_check.py`, `stage2/figs.py` | Stage 2 as plain scripts; `python su2_haar_check.py` then `python figs.py` |
| `stage3/su2_stage3_topological_lock.ipynb` | Stage 3: topological lock test (Section 10.1). Neutrino probe transported around a loop; random versus winding locked links with identical link statistics |
| `stage3/su3_lock_sim.py`, `stage3/su3_lock_figs.py` | SU(3) version: Z₃ center vortex lock, phase 2π/3 per loop, local statistics identical (1 − ⟨\|Tr U/3\|²⟩ = 8/9) |

Backend: PennyLane with `lightning.qubit`. The 27-qubit cubic case needs ~2 GB statevector per rank.

## Classical Wilson companion (`classical/`)

| Script | Description |
|--------|-------------|
| `twist/su2_classical_twist.py` | The classical companion study of Section S6: 3D Creutz heat bath in the quaternion representation, Frobenius twist density split into gradient and orientation channels, hedgehog charge per cell, m*_CTC per site; MPI chains, checkpoints and `--resume`. Source of `summary_cl_L*.json` |
| `twist/twist_scaling_fit.py` | Cross size analysis for S6: hedgehog susceptibility against 1/L with constant and linear fits, and the volume stable floor quantiles (Figure S9) |
| `twist/submit_su2_twist.sh` | Slurm script for the S6 production scan, L = 16, 24, 32 at β = 2.2, 2,000 configurations per size on 32 chains, then the scaling fit |
| `topology/` | Stage 3 in the thermal vacuum: hedgehog charge correlator, Wilson loops, neutrino probe around loops, center vortices after maximal center gauge. MPI heat bath for NERSC Perlmutter with Slurm scripts; also runs on the S6 checkpoints. See `topology/README.md` |

### Stage 3 results on NERSC Perlmutter (`classical/topology/`)

| Job | Script | Result | Paper |
|---|---|---|---|
| 01 | `su2_topology_heatbath.py` | L = 16, 24, 32: ⟨c0⟩ = 0.4933, Σ Var c_k = 0.6172, hedgehog charge screened within one spacing, χ(0) = 0 | Figs 19, 20 |
| 10 | `vortex_lock_production.py` | Center vortex lock W_odd/W_even = −1.00 from R = 4 at L = 32 and 40 | Figs 21, 22 |
| 11 | `vortex_lock_production.py --gribov 3` | Lock unchanged by the gauge copy | Section 10.2 |
| 12 | `vortex_lock_production.py`, `twist_modes_analysis.py` | Three twist modes, split by level repulsion | Fig 25 |
| 13 | `locked_modes.py` | Populations 0.6924, 0.2664, 0.0412 at the lock scale; parity sign of the axis correlation | Fig 24 |
| 14 | `locked_modes.py`, `beta_scan_summary.py` | β = 3.0, 4.0, 5.0: same fixed point, lock at the same physical size (about 7/g²) | Fig 23 |
| 15 | `mode_correlators.py` | Correlation of each mode in space, β = 3.0, 4.0, 5.0: ordering by population at one lattice step, rates track the lattice spacing | Fig 27 |
| laptop | `pmns_test.py` | Measured electron neutrino row against the SU(2) populations; δ_CP = π allowed | Fig 26 |

Run order and times: `classical/topology/RUN_ORDER.md`. Summary JSONs of every job: `data/nersc_summaries/`.

### Four dimensions on NERSC Perlmutter (`classical/topology4d/`)

The 3D campaign repeated in 4D SU(2), the setting of Creutz (1980): same Wilson action, same heat bath, same center vortex and mode analysis (Section 10.5). 128 independent chains per run. Details and run order: `classical/topology4d/README_4D.md`.

| Job | Script | Result | Paper |
|---|---|---|---|
| 21 | `su2_4d_production.py` (16⁴, β = 2.3, 7,680 configurations) | plaquette 0.60227; populations 0.6924, 0.2664, 0.0412 at R = 3 to 8, real and null; W_odd/W_even −0.087, −0.414, −0.723 at R = 3 to 5, −0.925 ± 0.021 at R = 6, in spatial and temporal planes | Section 10.5 |
| 21 | multi turn fingerprint, same runs | one and three turns: −0.414 and −0.415 at R = 4, −0.723 and −0.717 at R = 5; the held phase is π | Section 10.5 |
| 21 | 20⁴, β = 2.3 (5,120) and β = 2.4 | volume independence: the 16⁴ values; β = 2.4 crossing R = 3.71 at both sizes, −0.68 ± 0.04 at R = 7 | Section 10.5 |
| 21 | β = 2.2, 2.3, 2.4, 2.5 | crossing at R = 2.20, 2.73, 3.71, 5.64 in lattice units | Section 10.5 |
| 23 | `su2_4d_production.py --gribov 3` (2,560) | best of three gauge copies: −0.094, −0.424, −0.725, −0.96 ± 0.03 at R = 3 to 6 | Section 10.5 |
| 24 | `su2_4d_potential.py` (multihit, Parisi, Petronzio and Rapuano 1983) | a√σ = 0.499(9), 0.376(5), 0.272(2), 0.188(2) at the time plateau; crossing 1.10, 1.03, 1.01, 1.06 in units of 1/√σ, the same physical size within 5 percent of 1.05 | Section 10.5 |
| 26 | `su2_4d_mode_masses.py` (16⁴, β = 2.4) | time correlators of the three twist modes | Section 10.5 |
| 27, 28 | `su2_4d_production.py` 32⁴, `su2_4d_potential.py --tmax 12` 24⁴, β = 2.6 | fifth coupling, in progress; not used in the paper | |

Summary JSONs: `data/nersc_summaries/4d/`. Progress of every run: `python check_status.py`.

## Analysis (`analysis/`)

| Script | Input | Output |
|--------|-------|--------|
| `floor_quantiles.py` | results npz | Figure S9: the volume-stable quantile ladder |
| `twist_visualization.py` | rank checkpoints | Figures S10–S11: twist slices, per-config means, correlation function |
| `postprocess_topology.py` | checkpoints + production `measure_fields` | Site-level topology field dumps |

## Data

`data/` holds the campaign summary JSONs, including `stage2_results.json` with the Stage 2 numbers used in the paper. In the Stage 1 circuit summaries (`summary_L3.json`, `summary_L5.json`) the reconstructed identity component satisfies ⟨c_0²⟩ ≈ 0.98, so Var ≈ 1 there comes from the vanishing mean (Stage 1), not from c_0 = 0. Large products — the results npz files with pooled site-level histograms and the 32 gauge-field checkpoints (L = 32) — are attached to this repository's **Releases** rather than the source tree, and will be updated there as further campaigns complete. The Zenodo record holds the manuscript of record.

## Reproducibility

All runs are seeded. Identical results across platforms and rank layouts were verified to the fourth decimal between HPC runs and laptop-scale notebooks.

Requirements: `numpy`, `scipy`, `matplotlib`, `pennylane` (+ `lightning.qubit`), `mpi4py` for the MPI scripts.

## Acknowledgment of method

This work was assisted by LLMs for scripts and drafting. The concept, its development, and the verification of every result remain the responsibility of the author.

## Citation

If you use this code, cite the Zenodo preprint above.
