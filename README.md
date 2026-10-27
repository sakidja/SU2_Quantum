# SU(2) Neutrino QC

Simulation and analysis code for the preprint:

**"Dynamical Generation of Neutrino Mass from Local SU(2) Gauge Variance"**
R. Sakidja, Zenodo (2026). Version 3. DOI: [10.5281/zenodo.23646621](https://doi.org/10.5281/zenodo.23646621)

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
hamiltonian/ Version 3: exact Kogut Susskind Hamiltonian treatment, twisted boundary Monte Carlo, results and the Section 11 addendum
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

## Version 3: the Hamiltonian vacuum (`hamiltonian/`)

Version 3 of the paper adds an exact quantum treatment of the same SU(2) plaquette field: the Kogut Susskind Hamiltonian diagonalised on small periodic lattices (single plaquette, open 2×1, 1×1, 2×1, 3×1 and 2×2 tori), with the electric basis truncated at j ≤ jmax and convergence shown in jmax, tied to the Monte Carlo campaign above through a twist free energy computed with the same heat bath code. All runs on NERSC Perlmutter. The folder has its own `README.md` with the layout, the script to section map and the memory rule for the large builds.

| Script | Result | Paper |
|---|---|---|
| `nersc/run.py` | exact ground state: Σ Var(c_k) = 0.614 on 3×1 at β = 2.2 against 0.617 from the Monte Carlo; twist energy on every closed lattice, zero on the open pair | 11.2, 11.4, Table 11.1 |
| `nersc/probe_phase.py` | probe coupled to Σ c_k²: energy shift λ Σ Var(c_k), Lamb correction 1 to 3 percent, coherence loss below one percent at λ = 0.5 | 11.3, Table 11.2, Fig 29 |
| `twistmc/su2_twist_free_energy.py` | twist cost per unit time 0.1275 at the 2×2 cross section, zero from 4³ to 32³: a core of about 7/g², free beyond | 11.4, Table 11.3, Fig 30 |
| `twistmc/vortex_eye.py` | the imposed flux has no location in boxes of L ≥ 8: twisted and untwisted ensembles identical slice by slice | 11.4, Fig 31 |
| `nersc/flux_quench.py` | twist released from one plaquette delocalises in about one lattice time; variance moves with the energy (correlation 0.82 to 0.95) | 11.5, Fig 32 |
| `nersc/emergent_symmetry.py` | no singlet projection: the ground state has total charge K = 0 on every lattice, gap 0.84 to 2.84 to the first charged sector | 11.6, Fig 33 |
| `nersc/ungauged.py` | no gauge fixing, no constraint: ⟨G_s²⟩ = 0 at every site; breaking term κ induces ⟨G²⟩ = χ κ² with χ × gap² = 0.36 to 0.37 constant across couplings, sectors and truncations; gap constant in units of g²; thermal occupation with activation energy equal to the gap; random per link breaking with χ′ = χ; the 1×1×1 cube in 3+1 dimensions | 11.6, Table 11.5, Fig 33 |
| `nersc/angle_distribution.py` | twist populations of the 3×1 ground state, 0.728 / 0.234 / 0.037, converged in jmax; a geometry limited result | 11.7, Table 11.4, Fig 34 |

Results and job logs of every run are in `hamiltonian/nersc/results`, `hamiltonian/nersc/logs` and `hamiltonian/twistmc/results`; the saved Monte Carlo configurations and the quench movies stay on Perlmutter. Added since the first v3 snapshot: 3×1 at jmax 2.5, 2×2 at jmax 1.5, the 2×2 quench at jmax 1.5, the thermal and random breaking readouts of the ungauged vacuum, the 3+1 dimensional 1×1×1 cube, the torus geometry generator, the general Lx×Ly×Lt heat bath and the centre vortex analysis. Added in the second snapshot: the eight cross section boxes (2×3 to 5×5 at Lₜ = 32) that fill Figure 30 between 2×2 and 4×4, with the twist cost falling as e^(−0.51 A) in the area, and the 3+1 dimensional 2×1×1 box in `ungauged.py`. Also added: the centre vortex analysis of the twisted against the untwisted ensembles (Figure 35). This is the release snapshot for version 3, including the 3+1 dimensional 2×1×1 box (7.5 million states, Gauss law as an outcome at both sites, gap/g² and χ × gap² in three spatial dimensions). Still queued: 3×1 at jmax 3; the 3×2 torus at jmax 1 needs a coupled basis builder; both go into version 3.1. The eight teaching lectures built from the paper (about 140 slides each, 1,124 in all, with typeset derivations, falsifier slides, exercises and solutions) are built from the sources in `lectures/`; see “Building the lecture decks” below.

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

AI (LLM) assistance (Claude, Anthropic) was used in preparing code and text. The framework, calculations and analysis are the author's own, and the author takes full responsibility for the scientific content.

## Citation

If you use this code or build on its results, please cite:

R. Sakidja, "Dynamical Generation of Neutrino Mass from Local SU(2) Gauge Variance," version 3, Zenodo (2026). https://doi.org/10.5281/zenodo.23646621

A machine readable citation is provided in `CITATION.cff`; GitHub shows it through the "Cite this repository" button.

## License

The code in this repository is released under the MIT License (see `LICENSE`). Any copy or substantial portion of the code must retain the copyright notice and this permission notice.

The scientific results and the paper are the work of the author and are dated by the Zenodo record above. Work that extends these results is asked to cite that record.

`render_eqs.py` regenerates `eqimg/`, which is a build cache and is therefore not
tracked. Requirements: node with `pptxgenjs`, and a LaTeX installation for the
equation images. The built decks, 1,124 slides across the eight lectures, are
attached to the Zenodo record rather than to this repository.
