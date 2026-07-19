# SU(2) Neutrino QC

Simulation and analysis code for the preprint:

**"SU(2) Neutrino QC: Dynamical Neutrino Mass Generation from the Geometric Variance of the Weak Gauge Field"**
R. Sakidja, Zenodo (2026). DOI: https://doi.org/10.5281/zenodo.21437873

The paper asks whether the neutrino mass has a geometric origin in the SU(2) gauge field itself. The imaginary components of the gauge field average to zero, and that zero has long been read as absence. Their variance does not vanish: it locks to the geometric bound of the group, in every simulation, at every lattice size, on quantum circuits and classical lattices alike. That variance is the Yang–Mills energy density, term by term.

## Repository layout

```
analysis/    Post-processing and figure scripts (laptop-scale, no MPI)
data/        Summary JSONs from the production campaigns
quantum/     Quantum simulation scripts (planar and cubic lattices)
classical/   Classical SU(2) Wilson companion study (production + notebook)
```

## Quantum simulations (`quantum/`)

| Script | Description |
|--------|-------------|
| `su2_9qubit_notebook.ipynb` | 3×3 planar lattice, 9 qubits; the laptop-scale run of Section 3 |
| `su2_scaling_mpi.py` | MPI scaling script for L×L planar lattices (9/16/25 qubits); `--cubic` mode for 2×2×2 and 3×3×3 (8/27 qubits); seeded, checkpoint every 50 samples, lossless resumption (Sections 4.5, S5) |
| `u1_control.py` | U(1) control demonstrating the constraint is SU(2)-specific (Section S3) |
| `energy_dependence.py` | Uniform/Gaussian/binary amplitude-scan protocol (Figures 13–14) |

Backend: PennyLane with `lightning.qubit`. The 27-qubit cubic case needs ~2 GB statevector per rank.

## Classical Wilson companion (`classical/`)

| Script | Description |
|--------|-------------|
| `su2_wilson_production.py` | 3D SU(2) Creutz heat-bath in the quaternion representation; 32 MPI ranks; hedgehog charge per cell, m\*\_CTC per site, pooled histograms, per-rank checkpoints. Source of `results_cl_L*.npz` and `ckpt_cl_L32_rank*.npz` |
| `su2_wilson_colab.ipynb` | Resumable notebook version (L = 4–16, multi-seed); reproduces the study on a free Colab instance in under an hour (Section S6) |

## Analysis (`analysis/`)

| Script | Input | Output |
|--------|-------|--------|
| `floor_quantiles.py` | results npz | Figure S9: the volume-stable quantile ladder |
| `twist_visualization.py` | rank checkpoints | Figures S10–S11: twist slices, per-config means, correlation function |
| `postprocess_topology.py` | checkpoints + production `measure_fields` | Site-level topology field dumps |

## Data

`data/` holds the campaign summary JSONs. Large products — the results npz files with pooled site-level histograms and the 32 gauge-field checkpoints (L = 32) — are attached to this repository's **Releases** rather than the source tree, and will be updated there as further campaigns complete. The Zenodo record holds the manuscript of record.

## Reproducibility

All runs are seeded. Identical results across platforms and rank layouts were verified to the fourth decimal between HPC runs and laptop-scale notebooks.

Requirements: `numpy`, `scipy`, `matplotlib`, `pennylane` (+ `lightning.qubit`), `mpi4py` for the MPI scripts.

## Acknowledgment of method

This work was assisted by LLMs for scripts and drafting. The concept, its development, and the verification of every result remain the responsibility of the author.

## Citation

If you use this code, cite the Zenodo preprint above.
