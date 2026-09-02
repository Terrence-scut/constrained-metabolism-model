# Data Schema

This document describes the CSV files used by the constrained metabolism model. All paths are relative to the repository root unless stated otherwise.

## General conventions

- Files are comma-delimited UTF-8 text with one header row.
- Decimal values use a period as the decimal separator.
- Material quantities use metric tonnes on the modeled one-year basis.
- Annual demand, capacity, and grid-cell inputs use `t yr^-1`.
- Distances use kilometers.
- Canonical node identifiers consist of a node-type code, an underscore, and a positive integer.
- Controlled vocabularies are defined in `data/codebook.csv`.
- Within `data/scenario_manifest.csv`, empty categorical fields occur only in the baseline row.

## 1. Candidate transport arcs

Files:

- `data/network/facility_project_arcs_km.csv`: 2,382,778 rows covering the 11 allowed arc types among material-supply, processing, recycling, and construction-project nodes.
- `data/network/grid_to_recycling_arcs_km.csv`: 2,814,350 rows covering only `NDGRD -> NDWRP` arcs.

The notebook loads the files in this order and concatenates them without reordering. The combined transport network contains 5,197,128 rows.

| Column | Type | Unit | Definition | Validation |
| --- | --- | --- | --- | --- |
| `origin_node_id` | string | - | Canonical identifier of the directed arc origin. | Required; valid node identifier. |
| `destination_node_id` | string | - | Canonical identifier of the directed arc destination. | Required; valid node identifier. |
| `distance_km` | floating point | km | Network distance assigned to the directed arc. | Required; finite and non-negative. |

Both files use the same schema. The composite key (`origin_node_id`, `destination_node_id`) must be unique across the two files together. The allowed directed node-type pairs are:

```text
NDCEM -> NDCWV   NDSND -> NDCWV   NDQRY -> NDCWV
NDCWV -> NDCWP
NDCWP -> NDCBP   NDCWP -> NDBMP
NDCBP -> NDPRJ   NDBMP -> NDPRJ
NDGRD -> NDWRP
NDWRP -> NDPRJ   NDWRP -> NDCBP   NDWRP -> NDBMP
```

Transport mode is determined by the directed node-type pair in the notebook.

### 1.1 Material-route registry

The notebook registry contains 17 allowed material-route templates:

| Material | Directed route template |
| --- | --- |
| `CEM` | `NDCEM->NDCWV->NDCWP->NDCBP->NDPRJ` |
| `CEM` | `NDCEM->NDCWV->NDCWP->NDBMP->NDPRJ` |
| `NS` | `NDSND->NDCWV->NDCWP->NDCBP->NDPRJ` |
| `NS` | `NDSND->NDCWV->NDCWP->NDBMP->NDPRJ` |
| `NCA` | `NDQRY->NDCWV->NDCWP->NDCBP->NDPRJ` |
| `NCA` | `NDQRY->NDCWV->NDCWP->NDBMP->NDPRJ` |
| `WAT` | `NDCBP->NDPRJ` |
| `WAT` | `NDBMP->NDPRJ` |
| `WC` | `NDGRD->NDWRP` |
| `WB` | `NDGRD->NDWRP` |
| `RCS` | `NDWRP->NDPRJ` |
| `RCL` | `NDWRP->NDPRJ` |
| `RCC` | `NDWRP->NDCBP->NDPRJ` |
| `RCF` | `NDWRP->NDBMP->NDPRJ` |
| `RBC` | `NDWRP->NDBMP->NDPRJ` |
| `RBF` | `NDWRP->NDBMP->NDPRJ` |
| `RBP` | `NDWRP->NDBMP->NDPRJ` |

After repeated segments shared by routes for the same material are deduplicated, the registry contains 34 unique material/arc-type combinations. These two counts provide static checks on the route registry before a model run.

## 2. Static node parameters

File: `data/model/node_parameters_2026.csv`

Expected rows: 13,880.

| Column | Type | Unit | Definition | Validation |
| --- | --- | --- | --- | --- |
| `node_id` | string | - | Canonical identifier of a project or facility node. | Required; valid node identifier. |
| `parameter_name` | string | - | Canonical parameter name. | Required; listed for the node type below. |
| `value_t_per_year` | floating point | t yr^-1 | Annual demand or capacity value. | Required; finite and non-negative. |

The composite key is (`node_id`, `parameter_name`). Permitted combinations are:

| Node type | Permitted parameter names |
| --- | --- |
| `NDPRJ` | `concrete_demand`, `block_demand` |
| `NDCBP` | `batching_plant_capacity` |
| `NDBMP` | `block_plant_capacity` |
| `NDCEM` | `cement_supply_capacity` |
| `NDSND` | `marine_sand_supply_capacity` |
| `NDQRY` | `natural_coarse_aggregate_supply_capacity` |
| `NDWRP` | `recycling_plant_capacity` |

## 3. Grid-cell WC and WB inputs

Integrated files are stored under `data/scenarios/`; baseline files are stored under `data/baseline/`. Every WC or WB file contains 56,287 rows and the same canonical `NDGRD` grid-cell identifier set.

### 3.1 WC files

| Column | Type | Unit | Definition | Validation |
| --- | --- | --- | --- | --- |
| `grid_cell_id` | string | - | Canonical demolition-waste grid-cell identifier. | Required; unique within the file; node type `NDGRD`. |
| `waste_concrete_network_input_t_per_year` | floating point | t yr^-1 | Model-ready annual WC input entering the network from the grid cell. | Required; finite and non-negative. |

### 3.2 WB files

| Column | Type | Unit | Definition | Validation |
| --- | --- | --- | --- | --- |
| `grid_cell_id` | string | - | Canonical demolition-waste grid-cell identifier. | Required; unique within the file; node type `NDGRD`. |
| `waste_brick_network_input_t_per_year` | floating point | t yr^-1 | Model-ready annual WB input entering the network from the grid cell. | Required; finite and non-negative. |

WC and WB values are model-ready annual network inputs at the grid-cell level, rather than unadjusted supply potentials. They reflect the lifespan scenario, the selected percentile-representative Monte Carlo realization, and the applicable city-specific policy targets. WB values additionally include an absorption adjustment where applicable. The adjustment rate is stored in `data/scenario_manifest.csv` and, for adjusted files, in the filename suffix. Baseline WC and WB values are zero.

## 4. Scenario manifest

File: `data/scenario_manifest.csv`

Expected rows: 19, comprising 18 integrated scenario–realization combinations and one baseline.

| Field | Type | Definition | Validation |
| --- | --- | --- | --- |
| `scenario_id` | string | Compact runtime lookup key. | Unique; one of the manifest IDs. |
| `scenario_type` | string | Scenario class. | `integrated` or `baseline`. |
| `lifespan_scenario` | string | Building-lifespan case. | `short`, `medium`, or `long`; empty for baseline. |
| `policy_scenario` | string | City policy-target case. | `near` or `mid-long`; empty for baseline. |
| `input_realization` | string | Percentile-representative Monte Carlo realization. | `P2.5`, `P50`, or `P97.5`; empty for baseline. |
| `waste_concrete_file` | string | Path to the paired WC file, relative to `data/`. | Required; existing path. |
| `waste_brick_file` | string | Path to the paired WB file, relative to `data/`. | Required; existing path. |
| `waste_brick_absorption_rate_pct` | floating point | Percentage of the scenario-targeted WB quantity represented by the model-ready input. | For integrated rows, greater than zero and at most 100; empty for baseline. |

Integrated IDs use compact forms such as `s_n_p2.5` and `m_ml_p50`. Full categorical labels are stored in their dedicated manifest fields. Adjusted WB filenames include `_ar<percentage>`; the decimal point is retained, for example `_ar89.5`.

## 5. File manifest

File: `data/file_manifest.csv`

Expected rows: 42, covering 41 model inputs and `data/scenario_manifest.csv`.

| Field | Type | Definition | Validation |
| --- | --- | --- | --- |
| `release_path` | string | Repository-root-relative path to the file. | Starts with `data/`; unique; existing path. |
| `file_role` | string | Functional role of the file. | `network_input`, `model_input`, `baseline_input`, `scenario_input`, or `scenario_manifest`. |
| `scenario_id` | string | Associated scenario lookup key when applicable. | Valid manifest ID or empty. |
| `material_code` | string | Associated demolition-waste material when applicable. | `WC`, `WB`, or empty. |
| `row_count` | integer | Number of data rows excluding the header. | Non-negative and equal to the physical file. |
| `column_names` | string | Ordered header names separated by `|`. | Equal to the physical header. |
| `size_bytes` | integer | Physical file size in bytes. | Positive and equal to the file. |
| `sha256` | string | SHA-256 digest of the file. | 64 lowercase hexadecimal characters. |

## 6. Codebook

File: `data/codebook.csv`

The codebook contains 48 canonical entries: 10 node types, 13 materials, 10 parameters, 6 recycling schemes, and 9 constraint groups.

| Field | Definition |
| --- | --- |
| `domain` | Controlled-vocabulary domain. |
| `canonical_code` | Canonical machine-readable code or parameter name. |
| `descriptive_name` | Descriptive snake-case label. |
| `definition` | Concise scientific meaning. |
| `unit` | Unit associated with the code, when applicable. |
| `paper_reference` | Reference to Supplementary Methods 5. |
| `notes` | Additional interpretation required for correct use. |

`canonical_code` is unique within each domain.

### 6.1 Relationship to Supplementary Methods 5

The flow carbon coefficients and constraint groups C1–C9 correspond to the model formulation described in Supplementary Methods 5. Non-negative bounds are applied separately to material-flow and recycling-scheme-throughput variables.

## 7. Generated outputs

Generated outputs are written under `results/<scenario_id>/` and follow the schemas below.

The notebook sets `REPORTING_THRESHOLD_T_PER_YEAR = 1e-6`. Only values strictly greater than this threshold are exported. The scheme-throughput file is created for every successful run and contains only its header when no scheme exceeds the threshold.

### 7.1 Positive material flows

File: `results/<scenario_id>/flow_results_<scenario_id>.csv`

| Field | Type | Unit | Definition |
| --- | --- | --- | --- |
| `material_code` | string | - | Canonical material code. |
| `origin_node_id` | string | - | Canonical arc-origin identifier. |
| `destination_node_id` | string | - | Canonical arc-destination identifier. |
| `distance_km` | floating point | km | Network distance. |
| `arc_type` | string | - | Directed canonical node-type pair. |
| `origin_node_type` | string | - | Canonical origin node type. |
| `destination_node_type` | string | - | Canonical destination node type. |
| `unit_carbon_impact_kgco2e_per_t` | floating point | kg CO2-eq t^-1 | Flow carbon coefficient assigned to the material and directed arc. |
| `flow_t_per_year` | floating point | t yr^-1 | Modeled annual material flow on the directed arc. |

### 7.2 Recycling-scheme throughput

File: `results/<scenario_id>/scheme_throughput_<scenario_id>.csv`

| Field | Type | Unit | Definition |
| --- | --- | --- | --- |
| `recycling_plant_id` | string | - | Canonical waste-recycling-plant identifier. |
| `recycling_scheme` | string | - | Canonical recycling-scheme code. |
| `throughput_t_per_year` | floating point | t yr^-1 | Active scheme throughput at the plant. |

## 8. Integrity checks

The following checks verify the integrity of the repository inputs:

1. Every file listed in `data/file_manifest.csv` exists and matches its recorded row count, ordered header, byte size, and SHA-256 digest.
2. Every manifest path resolves inside `data/`.
3. The 18 integrated rows form a complete set of lifespan, policy, and realization combinations, with one WC/WB pair per row.
4. Every scenario and baseline input contains the same unique grid-cell identifier set.
5. All node, material, parameter, and recycling-scheme values occur in `data/codebook.csv`.
6. Node-parameter pairs are unique and compatible with their node type.
7. Distances and annual values are finite and non-negative.
