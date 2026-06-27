# SLPO: Simulation Launcher & Pareto Optimization for OpenFOAM CFD

<p align="center">
  <img src="images/logo.jpg" width="50%">
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3-blue?logo=python&logoColor=white">
  <img alt="OpenFOAM" src="https://img.shields.io/badge/OpenFOAM-11-crimson?logo=openfoam">
  <img alt="Gmsh" src="https://img.shields.io/badge/Gmsh-mesh-orange">
  <img alt="Docker" src="https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white">
</p>

SLPO is a Python workflow that automates parametric CFD simulations and identifies Pareto-optimal designs. It uses Monte Carlo sampling to explore a geometric design space, runs each configuration through OpenFOAM's `icoFoam` solver, and applies Pareto optimization to find designs that best balance two competing objectives: minimizing inlet pressure and minimizing material cost.

The test case is a simplified air injector modeled as a curved pipe with a small secondary inlet.

---

This project was submitted as the final project for the Bachelor's degree in Industrial Electronics and Automatic Control Engineering at the Manresa School of Engineering (EPSEM), Universitat Politècnica de Catalunya (UPC), directed by Enrique Bonet and Francesc Perez.

## How it works

1. **Monte Carlo sampling** — random geometric and physical parameters are drawn for each simulation (pipe radii, flow speeds, temperature, kinematic viscosity).
2. **Mesh generation** — Gmsh builds a 3D mesh of the curved-pipe geometry from those parameters.
3. **CFD simulation** — OpenFOAM (`icoFoam`) solves incompressible, isothermal flow on the mesh.
4. **Result extraction** — pressure and velocity fields are read from VTK output; average inlet pressure and surface area are recorded.
5. **Cost computation** — material cost is estimated from surface area, material density, and price (304 stainless steel).
6. **Pareto optimization** — non-dominated points in the (cost, pressure) space are identified and plotted.

```
                       ┌────────────────────────────┐
                       │               n iterations │
                       ▼                            │
                ┌─────────────┐                     │
                │ Monte Carlo │                     │
                └──────┬──────┘                     │
                       │                            │
              Input    │                            │
            parameters │                            │
                       │                            │
         ┌─────────────┴──────────────┐             │
         │                            │             │
         │                            │             │
         ▼                            ▼             │
┌──────────────────┐         ┌─────────────────┐    │
│ Simulation files │         │ Mesh generation │    │
│    (PyFoam)      │         │     (gmsh)      │    │
└───────┬──────────┘         └────────┬────────┘    │
        │                             │             │
        └──────────────┬──────────────┘             │
                       │                            │
                       ▼                            │
             ┌────────────────────┐                 │
             │   Run simulation   ├─────────────────┘
             │ (OpenFOAM/icoFoam) │
             └─────────┬──────────┘
                       │
                       ▼
             ┌─────────────────────┐
             │ Extract and collect ├──┐
             │       results       │  │
             └─────────┬───────────┘  │
                       │              │
                       ▼              │
             ┌─────────────────────┐  │
             │ Pareto optimization ├──┤
             └─────────────────────┘  │
                                      │
                                      ▼
                               ┌──────────────┐
                               │ Plot results │
                               │ (matplotlib) │
                               └──────────────┘
```

## Repository structure

```
.
├── src/
│   ├── main/
│   │   ├── run.py       # entry point; orchestrates the full pipeline
│   │   ├── func.py      # all functions (mesh, simulation, results, Pareto, plots)
│   │   └── test.py      # single-simulation run for development and testing
│   ├── docker/
│   │   ├── Dockerfile   # Ubuntu + OpenFOAM 11 + Python dependencies
│   │   ├── build.sh
│   │   └── run.sh
│   └── mesh_time_test/  # standalone mesh timing benchmarks
├── docs/
│   ├── project_report.pdf
│   └── project_presentation.pdf
└── example_results/     # sample outputs from a completed run
```

## Dependencies

| Tool | Role |
|---|---|
| OpenFOAM 11 | CFD solver (`icoFoam`, `gmshToFoam`, `foamToVTK`) |
| Gmsh | Parametric 3D mesh generation |
| PyFoam | OpenFOAM case setup and runner interface |
| VTK (Python) | Result extraction from VTK output files |
| pandas, matplotlib | Data processing and plotting |

## Usage

### With Docker (recommended)

```bash
cd src/docker
bash build.sh
bash run.sh
```

Inside the container, mount your working directory to `/home/foam/code` and run:

```bash
python3 run.py
```

### Directly

With OpenFOAM 11, Python 3, and all dependencies installed:

```bash
cd src/main
python3 run.py
```

The number of simulations is set in `run.py` at the bottom:

```python
main(2)  # replace 2 with the desired number of simulations
```

For a single test run:

```bash
python3 test.py
```

## Outputs

Each run produces:

| File | Contents |
|---|---|
| `results1_sim_vtk.csv` | Raw VTK data (pressure, velocity, coordinates) for all simulations |
| `results2_avgp_cost.csv` | Per-simulation summary: average inlet pressure and material cost |
| `results3_pareto.csv` | Pareto-optimal subset of simulations |
| `statistics_<timestamp>.json` | Descriptive statistics for cost and pressure across all runs |
| `plot_*.pdf` | Scatter plots of all points, Pareto points, and the Pareto frontier |
| `s<n>_params.json` | Input parameters saved for each simulation run |

## Abstract

This project presents the development of the SLPO (Simulation Launcher & Pareto Optimization) algorithm, designed to automate fluid dynamics simulations and optimize design parameters using open-source computational fluid dynamics tools. The system studied is a simplified air injector, modeled as a curved pipe with a small inlet. Using Monte Carlo sampling, random parameter sets are generated, and simulations are conducted in OpenFOAM with the icoFoam solver for incompressible, isothermal fluid flow. Pareto optimization is applied to perform multi-objective optimization, identifying designs that balance the two competing objectives of minimizing inlet pressure and material cost. The algorithm explores the design space automatically and identifies Pareto optimal solutions. The workflow, built on Python, Gmsh, and PyFoam, offers a scalable approach to multi-objective optimization in CFD. While the case model is considered too simplistic, the methodology demonstrates significant potential for more complex and practical engineering applications.
