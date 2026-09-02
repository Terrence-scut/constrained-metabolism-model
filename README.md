# Constrained Metabolism Model

This repository contains the Python notebook and model-ready inputs for the constrained metabolism model described in Supplementary Methods 5. The repository covers the regional material-flow model and its scenario inputs. It does not include the upstream building-stock projection, Monte Carlo generation, or life-cycle inventory preprocessing workflows.

## Repository contents

```text
notebooks/
  constrained_metabolism_model.ipynb
data/
  network/
  model/
  baseline/
  scenarios/
  scenario_manifest.csv
  file_manifest.csv
  codebook.csv
  DATA_SCHEMA.md
requirements.txt
CITATION.cff
LICENSE
```

- `notebooks/constrained_metabolism_model.ipynb` constructs and runs the model for one selected scenario.
- `data/network/facility_project_arcs_km.csv` contains the 11 candidate arc types linking material-supply, processing, recycling, and construction-project nodes.
- `data/network/grid_to_recycling_arcs_km.csv` contains the `NDGRD -> NDWRP` candidate arcs from demolition-waste grid cells to recycling plants.
- `data/model/node_parameters_2026.csv` contains project-demand and facility-capacity inputs.
- `data/baseline/` contains the all-virgin baseline WC and WB inputs.
- `data/scenarios/` contains 18 paired WC/WB scenario realizations.
- `data/scenario_manifest.csv` is the runtime registry for the baseline and integrated scenarios.
- `data/file_manifest.csv` records the size, schema, and SHA-256 digest of every manifest-tracked model input and the scenario manifest.
- `data/codebook.csv` defines the canonical node, material, parameter, recycling-scheme, and constraint-group vocabulary.
- `data/DATA_SCHEMA.md` documents the physical CSV schemas and validation rules.

Together, the two network files define 5,197,128 candidate transport arcs. They are loaded in the order listed above and concatenated without reordering.

Generated files are written under `results/<scenario_id>/`, which is excluded from version control.

## Interpretation of the scenario inputs

WC and WB files contain model-ready annual network inputs at the grid-cell level. Each integrated input pair reflects a lifespan scenario, one selected percentile-representative Monte Carlo realization, and the applicable city-specific policy targets.

WB inputs additionally include an absorption adjustment where applicable. The corresponding rate is recorded in `waste_brick_absorption_rate_pct` in `data/scenario_manifest.csv`; adjusted WB filenames also contain an `_ar<percentage>` suffix. A value of 100 indicates that the model-ready WB input equals the scenario-targeted quantity.

The baseline WC and WB files contain zero inputs and represent the all-virgin case.

## Scenario selection

Set `SCENARIO_ID` in the first code cell of the notebook. The notebook defaults to `l_n_p50`; valid values are `baseline` and the 18 integrated IDs listed in `data/scenario_manifest.csv`.

Integrated IDs use compact lookup labels:

- lifespan: `s`, `m`, or `l` for short, medium, or long;
- policy: `n` or `ml` for the near-term or medium-to-long-term policy target;
- input realization: `p2.5`, `p50`, or `p97.5`.

For example, `m_n_p50` selects the medium-lifespan case, the near-term policy target, and the P50 input realization. The manifest is authoritative for the full labels and paired file paths.

The 19 valid scenario IDs are:

```text
baseline
s_n_p2.5   s_n_p50   s_n_p97.5
s_ml_p2.5  s_ml_p50  s_ml_p97.5
m_n_p2.5   m_n_p50   m_n_p97.5
m_ml_p2.5  m_ml_p50  m_ml_p97.5
l_n_p2.5   l_n_p50   l_n_p97.5
l_ml_p2.5  l_ml_p50  l_ml_p97.5
```

## Software environment

The model was run in an Anaconda environment on Windows with Python 3.10.9, Gurobi 12.0.2, and gurobipy 12.0.2. The tested package versions were NumPy 1.23.5, pandas 1.5.3, SciPy 1.10.0, JupyterLab 3.5.3, and ipykernel 6.19.2. The system had a 13th-generation Intel Core i7-13700K processor and 64 GB of memory. Running the model requires a valid Gurobi license.

`requirements.txt` lists the direct notebook dependencies and their tested versions. It is not a complete export of the surrounding Anaconda environment.

## Running the notebook

1. Create and activate an environment with Python 3.10.9.
2. Install the listed packages:

   ```bash
   python -m pip install -r requirements.txt
   ```

3. Confirm that a valid Gurobi license is available in the activated environment.
4. Start Jupyter from the repository root or from `notebooks/`:

   ```bash
   python -m jupyter lab
   ```

5. Open `notebooks/constrained_metabolism_model.ipynb`, set `SCENARIO_ID`, and run the cells in order.

The full model is computationally demanding. Each scenario uses 5,197,128 candidate transport arcs and constructs 16,710,231 material-flow variables, 1,734 recycling-scheme-throughput variables, and 7,186,946 constraints. At least 64 GB of memory is recommended for a complete run.

## Verification status

The notebook was run successfully for all 19 scenarios registered in `data/scenario_manifest.csv` using the software environment described above.

## Outputs

The notebook exports:

- positive material-flow records on candidate arcs; and
- active recycling-scheme-throughput records by waste recycling plant.

The output fields are documented in `data/DATA_SCHEMA.md`. Output units follow the one-year model basis.

## Citation and contact

Citation metadata for this software are provided in `CITATION.cff`. Bibliographic details for the associated article will be updated after publication.

For questions about the model, source code, or model inputs, please contact Bo Wu at bowu@scut.edu.cn.

## License

The source code and associated software documentation authored for this repository are released under the MIT License; see [LICENSE](LICENSE). This code license does not cover the model input data. Gurobi and gurobipy are third-party dependencies, are not distributed as part of this repository, and remain subject to Gurobi's own licensing terms. A valid Gurobi license is required to run the full model.
