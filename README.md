# Percolation without trapping

Quasi-static two-phase pore-network modelling of percolation without trapping, developed by **Ademola Isaac Adebimpe** ([AIAdebimpe](https://github.com/AIAdebimpe)) as an extension of [pnflowPy](https://github.com/ImperialCollegeLondon/pnflowPy).

## Model purpose

The model computes drainage and imbibition on a pore network, including capillary-pressure and relative-permeability curves. It investigates the equilibrium effect of Ostwald ripening on two-phase displacement using percolation without trapping. It first computes a single-phase baseline, then follows the displacement cycles specified in the input file.

These are three separate codebases:

| Repository | Purpose |
| --- | --- |
| [pnflowPy](https://github.com/ImperialCollegeLondon/pnflowPy) | The underlying quasi-static pore-network flow model and supporting input/network files. |
| **percolation_without_trapping** (this repository) | Extensions for quasi-static percolation without trapping. |
| [OstRipening](https://github.com/ImperialCollegeLondon/OstRipening) | Time-dependent pore-network modelling of Ostwald ripening. |

This model does not resolve the elapsed-time mass-transfer dynamics of ripening. Its extensions modify the underlying phase-displacement algorithms; the drainage implementation retains wetting-phase trapping checks. The repository name should therefore not be interpreted as disabling every trapping check in both phases.

## Dependencies

Both this repository and **pnflowPy** are required. They must be available as sibling Python packages, as shown below. This repository does not contain its own `environment.yml` or example network data.

The current pnflowPy [environment file](https://github.com/ImperialCollegeLondon/pnflowPy/blob/main/environment.yml) specifies Python 3.10 and PETSc 3.20, together with the scientific Python stack. Packages used directly by this model or its pnflowPy dependency include:

| Dependency | Role |
| --- | --- |
| NumPy, SciPy, pandas | Numerical operations, sparse matrices and tabular data. |
| sortedcontainers, Numba | Ordered invasion queues and compiled numerical routines. |
| PETSc / petsc4py | Flow-system solution; the default solver uses GMRES with Hypre/BoomerAMG, so PETSc needs Hypre support. |
| dill, joblib | Network-state serialization and loading. |
| IPython | Imported by pnflowPy cluster routines. |

Use the dependency's environment file as the starting point rather than installing only the packages in this table. It also lists supporting packages such as MPI, HDF5, matplotlib and scikit-learn. `joblib` is imported directly but is not explicitly listed in that file; the setup below installs it explicitly.

## Installation

Run the following commands on the machine where the simulations will be executed, from a parent directory of your choice:

```bash
git clone https://github.com/ImperialCollegeLondon/percolation_without_trapping.git
git clone https://github.com/ImperialCollegeLondon/pnflowPy.git
conda env create -f pnflowPy/environment.yml
conda activate flowenv
conda install -c conda-forge joblib
```

Keep the repository directory names unchanged. The layout should be:

```text
parent-directory/
├── percolation_without_trapping/
│   ├── __init__.py
│   ├── main.py
│   ├── twoPhaseD.py
│   └── twoPhaseImb.py
└── pnflowPy/
    ├── environment.yml
    ├── inputData.py
    ├── network.py
    └── data/
        ├── input_pnflow.dat
        ├── Bentheimer_link1.dat
        ├── Bentheimer_link2.dat
        ├── Bentheimer_node1.dat
        └── Bentheimer_node2.dat
```

PETSc and its external solvers must be available for your platform. The environment specification is a setup starting point, not a guarantee that every operating system can resolve the same packages.

## Running the model

With `flowenv` active, stay in the **parent directory containing both repositories** and run:

```bash
python -m percolation_without_trapping.main pnflowPy/data/input_pnflow.dat
```

This invokes this repository's displacement model using the Bentheimer input and four network files supplied by pnflowPy. It is a general example input, not a claim to reproduce the publication's exact cases or results. Check its fluid properties, contact angles and displacement schedule before using it for a research case.

Use the package command above: `main.py` relies on relative imports, so running it directly as a script is not the supported entry point. Start a fresh Python process for each model run; these extensions replace functions in pnflowPy's drainage and imbibition modules during model initialization.

### Preparing a different network or case

Copy an input file and its matching network files into a case directory. For example, the input entry

```text
NETWORK F Bentheimer;
```

selects these files **in the same directory as the input file**:

```text
Bentheimer_link1.dat
Bentheimer_link2.dat
Bentheimer_node1.dat
Bentheimer_node2.dat
```

Then launch the case from the same parent directory:

```bash
python -m percolation_without_trapping.main path/to/case/input_pnflow.dat
```

Networks can be prepared with tools such as [pnextract](https://github.com/ImperialCollegeLondon/pnextract). In the input file, review at least `NETWORK`, `FLUID`, `INIT_CONT_ANG`, `EQUIL_CON_ANG`, `SAT_CONTROL` and `RAND_SEED`. `INIT_CONT_ANG` controls initial drainage contact angles; `EQUIL_CON_ANG` supplies the imbibition contact angles. Each `SAT_CONTROL` row contains 11 entries:

```text
finalSat Pc deltaSw minDeltaPc deltaPcFraction calcKr calcI InjectLeft InjectRight EscapeLeft EscapeRight
```

Input records end with `;`, and `%` introduces a comment. See pnflowPy's [input documentation](https://github.com/ImperialCollegeLondon/pnflowPy/blob/main/doc/input_pnflow.pdf) and the [pnflow documentation](https://github.com/ImperialCollegeLondon/pnflow) for the input format; only keywords implemented in the Python code apply. The shared example also contains `RIPENING` and `RES_DIR` entries, but this repository's driver does not use them to run time-dependent ripening or select its output directory.

### Outputs

The driver sets its model results directory to `percolation_without_trapping_results/`, relative to the directory from which Python is launched. Drainage/imbibition CSV output includes wetting-phase saturation (`Sw`), phase flow rates (`qW`, `qNW`), relative permeabilities (`krw`, `krnw`), capillary pressure (`Pc`) and invasion counts. Cycle-end network states are saved as `.pkl` files. Inherited pnflowPy initialization may also create a `quasi_static_results/` directory.

### Documentation validation

These instructions were checked by reading this repository at commit [`1a777b6`](https://github.com/ImperialCollegeLondon/percolation_without_trapping/commit/1a777b62abfa952ffdfc7b138e187e62d76a980f) and pnflowPy at commit [`f90dd00`](https://github.com/ImperialCollegeLondon/pnflowPy/commit/f90dd008d14688665d11633e27a9dc6eedf4357f). Environment installation, model execution and reproduction of the paper's results have not been validated as part of this documentation update.

## Reference

Ademola Isaac Adebimpe, Sajjad Foroughi, Branko Bijeljic and Martin J. Blunt (2024). **Percolation without trapping: How Ostwald ripening during two-phase displacement in porous media alters capillary pressure and relative permeability.** *Physical Review E* **110**, 035105. [doi:10.1103/PhysRevE.110.035105](https://doi.org/10.1103/PhysRevE.110.035105).

## Authorship, maintenance and provenance

- **Original model and code:** Ademola Isaac Adebimpe ([AIAdebimpe](https://github.com/AIAdebimpe)).
- **Imperial repository setup and documentation maintenance:** Linqi Zhu ([LinqiZhu](https://github.com/LinqiZhu)). This maintenance role does not change the original model authorship.
- The code was imported from [AIAdebimpe/percolation_without_trapping](https://github.com/AIAdebimpe/percolation_without_trapping), retaining the original commits and the `main` and `v1.0` branch histories. The original imported README remains available in [the import snapshot](https://github.com/ImperialCollegeLondon/percolation_without_trapping/blob/1a777b62abfa952ffdfc7b138e187e62d76a980f/README.md). Documentation maintenance is added through new commits without rewriting that history.

For model questions, contact Ademola Adebimpe:

- **a.adebimpe21@imperial.ac.uk**
- **aisaac.adebimpe@gmail.com**

See also the Imperial College London [Pore-scale Modelling research group](https://www.imperial.ac.uk/earth-science/research/research-groups/pore-scale-modelling).

## Contributing

Please open an issue or submit a pull request to this repository with a description of the proposed change. Keep model changes distinguishable from documentation updates, and include the relevant input data and validation evidence when proposing numerical changes.

## License

The original author's README states: "Distributed under the MIT License." This repository currently has no standalone `LICENSE` file; the original statement is retained here. The separately required pnflowPy repository carries its own [license](https://github.com/ImperialCollegeLondon/pnflowPy/blob/main/LICENSE).
