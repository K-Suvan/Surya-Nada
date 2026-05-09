# Surya-Nada

Surya-Nada is a lightweight 2D/3D solar MHD framework for generating **synthetic helioseismic observables** from simulations of a small patch of the upper convection zone and lower solar atmosphere.[file:107]

The code is designed to be:
- **Simple to read and modify** – aimed at students and researchers who want to experiment with helioseismic forward models.
- **Modular** – separate modules for synthetic acoustic events, 2D MHD tests, and 3D MHD simulations.
- **Practical for analysis** – produces time series, slices and diagnostics that can be fed directly into helioseismic pipelines.

---

## Project structure

*(Adjust names to match your actual repo)*

```text
Surya-Nada/
├─ README.md
├─ environment.yml / requirements.txt
├─ src/
│  ├─ synthetic_acoustic.py      # 2D time–distance synthetic module
│  ├─ mhd2d.py                   # 2D nonlinear MHD testbed
│  ├─ mhd3d.py                   # 3D finite-volume MHD solver
│  ├─ utils_io.py                # I/O, diagnostics, helpers
│  └─ config_example.yaml        # Example configuration file
├─ notebooks/
│  ├─ demo_synthetic.ipynb       # Example: synthetic time–distance diagrams
│  ├─ demo_3d_mhd.ipynb          # Example: 3D run + figures from the paper
├─ data/
│  ├─ input/                     # Initial conditions, background profiles
│  └─ output/                    # Simulation outputs (created at runtime)
└─ figures/
   ├─ 3d_extrema_evolution.png   # Max |v|, |B| vs time
   ├─ density.png
   ├─ bz.png
   └─ vz.png
```

---

## Physical model in a nutshell

- Solves the **ideal MHD equations** in a Cartesian box with constant gravity.[file:107]
- Background atmosphere is vertically stratified and in hydrostatic equilibrium; 3D runs use a simple quiet-Sun–like profile based on VAL‑C, slightly simplified.[file:107]
- Magnetic field consists of vertical flux concentrations plus weaker surrounding fields, mimicking network/sunspot-like structures.[file:107]
- Near-surface drivers (velocity/pressure perturbations) excite waves in the lower layers, representing granular-scale convection.[file:107]

The focus is **wave propagation and mode conversion** in the near-surface layers, not full radiative transfer or coronal eruptions.

---

## Modules and what they do

### 1. Synthetic acoustic event generator (`synthetic_acoustic.py`)

- Builds 2D acoustic wave fields on an \(x\)–\(t\) grid.
- Each event is a **Ricker wavelet** with specified:
  - acoustic speed \(v\),
  - start time \(t_0\),
  - propagation angle \(\theta\),
  - amplitude \(A\).[file:107]
- Superposes events to produce time–distance diagrams with known “true” travel times.

Use cases:
- Testing time–distance helioseismology pipelines.
- Validating cross-correlation and filtering steps against known ground truth.

---

### 2. 2D nonlinear MHD testbed (`mhd2d.py`)

- Solves the same ideal MHD equations in a 2D \((x, y)\) slice with:
  - stratified isothermal atmosphere,
  - uniform inclined background field.[file:107]
- Numerics:
  - Finite-difference scheme with **minmod slope limiter** for shock capturing.[file:107]
  - Third-order SSP Runge–Kutta for time stepping.
  - Small explicit viscosity \(\nu\) and resistivity \(\eta\) for controlled dissipation.
  - CFL timestep based on local sound + Alfvén speeds.
- Purpose:
  - “Sandbox” for tuning reconstruction, dissipation and boundary conditions before running 3D.
  - Quick checks of stability, shock capturing and boundary behaviour.

---

### 3. 3D solar MHD solver (`mhd3d.py`)

- Full 3D box: \(L_x = L_y = 6\) Mm, \(L_z = 1.4\) Mm on a \(128 \times 128 \times 64\) grid (example setup).[file:107]
- Numerics:
  - **Finite-volume** scheme for conservation.
  - **MP5** reconstruction (monotonicity-preserving, 5th order) for interface states.[file:107]
  - **HLLD** Riemann solver for ideal MHD fluxes.[file:107]
  - Third-order SSP Runge–Kutta in time.
  - Ghost cells with open/reflecting boundary conditions; analysis excludes a few cells near edges.
  - Global CFL timestep based on maximum characteristic speeds.

- Outputs:
  - 3D snapshots of \(\rho\), \(\mathbf{v}\), \(\mathbf{B}\) at regular times.
  - Diagnostics like global min/max of \(\rho\), \(|\mathbf{v}|\), \(|\mathbf{B}|\), used for Fig. “3d_extrema_evolution”.[file:107]
  - Derived “observables”: horizontal and vertical slices, synthetic Doppler-like maps, height-dependent power spectra.

Use cases:
- Forward modelling for local helioseismology.
- Studying basic wave–magnetic interactions in a simplified solar atmosphere.

---

## Installation

Assuming a Python-based implementation:

```bash
# clone the repository
git clone https://github.com/your-username/surya-nada.git
cd surya-nada

# using conda (recommended)
conda env create -f environment.yml
conda activate surya-nada

# or, using pip
pip install -r requirements.txt
```

Typical dependencies:

- `numpy`, `scipy`
- `numba` or `jax` (if you use acceleration)
- `matplotlib`
- `h5py` / `netCDF4` (if you store data in those formats)
- `tqdm` for progress bars
- `jupyter` for the example notebooks

Update `environment.yml` / `requirements.txt` to match exactly what you use.

---

## Running the examples

### 1. Synthetic acoustic module

```bash
python src/synthetic_acoustic.py --config configs/synthetic_example.yaml
```

This will:

- generate a set of acoustic events,
- save time–distance data under `data/output/synthetic/`,
- optionally create quicklook plots (x–t diagrams).

You can then open `notebooks/demo_synthetic.ipynb` to reproduce the figures and travel-time analysis.

---

### 2. 3D MHD run

```bash
python src/mhd3d.py --config configs/mhd3d_example.yaml
```

This will:

- set up the stratified atmosphere and embedded flux tubes,
- apply near-surface drivers,
- evolve the box up to \(t = 1200\) s (or whatever you set),
- write snapshot files to `data/output/mhd3d/`.

Use `notebooks/demo_3d_mhd.ipynb` to:

- compute global min/max summaries (Table 1),
- plot \(|\mathbf{v}|_{\max}\) and \(|\mathbf{B}|_{\max}\) vs time (Fig. 3d_extrema_evolution),
- produce the density/Bz/vz slices at \(t = 1200\) s (Fig. 3d_slices).

---

## Configuration

Example `config_example.yaml` might look like:

```yaml
# domain
nx: 128
ny: 128
nz: 64
Lx: 6.0e6     # m
Ly: 6.0e6
Lz: 1.4e6

# physics
gamma: 5./3.
g: 274.0
B0_strength: 100.0   # G
driver_amplitude: 200.0  # m/s

# numerics
cfl: 0.3
t_end: 1200.0
output_interval: 12.0
reconstruction: MP5
riemann_solver: HLLD

# I/O
output_dir: data/output/mhd3d_run1
```

Document the most important knobs in comments:

- grid resolution vs runtime,
- CFL number and stability,
- driver amplitude and spectrum.

---

## Reproducing paper figures

If you want others to reproduce your manuscript results:

1. Tag the commit that matches the paper, e.g. `v1.0-paper`.
2. Provide a short script/notebook per figure, for example:

   - `notebooks/figure1_synthetic_xt.ipynb`
   - `notebooks/figure2_3d_extrema.ipynb`
   - `notebooks/figure3_slices.ipynb`

3. In each notebook:
   - Load the appropriate output files,
   - Generate the figure with the same colour scales and labels as in the paper,
   - Save to `figures/`.

Mention this in the README:

> To reproduce the figures in Kadakol (2026), run the notebooks in `notebooks/` using the `v1.0-paper` tag.

---

## License


```text
This project is licensed under the MIT License – see the LICENSE file for details.
```

## Contact

- **Author:** Suvan Kadakol  
- **Affiliation:** Department of Physics, Dr Vishwanath Karad MIT World Peace University, Pune, India  
- **Email:** suvan.kadakol@mitwpu.edu.in
