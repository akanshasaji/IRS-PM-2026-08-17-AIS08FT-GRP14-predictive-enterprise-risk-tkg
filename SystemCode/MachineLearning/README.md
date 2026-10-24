# GC-1 — Temporal Snapshots, Feature Engineering and Risk Prediction

This README covers the three notebooks that prepare historical project checkpoints, build a numerical risk-feature dataset, train risk models, explain selected predictions and schedule investigations. 

## Notebooks and execution order

| Order | Notebook | Purpose | Main output |
|---|---|---|---|
| 1 | `GroundTruth_TemporalSnapshots.ipynb` | Reconstruct Jira and temporal-graph history; generate 14-day Schedule, Scope and Dependency Risk labels; validate point-in-time snapshots. | Historical labels and five final snapshot tables. |
| 2 | `Feature_Engineering.ipynb` | Calculate historical Jira, event, dependency, graph and scope features; select 34 numerical predictors; join and validate the three labels. | Labelled project-week feature matrix and feature list. |
| 3 | `Risk_Model_SHAP_Trigger.ipynb` | Train TabPFN 2.5 (Schedule) and CatBoost (Scope/Dependency); choose validation thresholds; compute on-demand SHAP explanations; replay and schedule investigations. | Model artifacts, evaluation records, SHAP outputs and investigation queues. |

**Pipeline:** Phase 1 temporal KG and Jira data → historical snapshots and labels → 34-feature dataset → three risk models → alert scheduling → event-focused retrieval.

## Execution environment

- Developed for **Google Colab** with Google Drive mounted at `/content/drive`.
- Expected project root: `/content/drive/MyDrive/GC1`.
- Python with `pandas`, `numpy`, `pyarrow` and the other imports used in each notebook. The modelling notebook installs `tabpfn==9.1.0`, `catboost>=1.2.8`, `shap>=0.46`, `pyarrow>=15`, `scikit-learn>=1.5` and `joblib>=1.3`.
- A **CUDA GPU** is required by the modelling notebook; it explicitly checks for one. TabPFN v2.5 checkpoint access may require acceptance of its license and a `TABPFN_TOKEN` in Colab secrets.
- Input files from the upstream Phase 1 knowledge extraction, entity resolution and temporal KG pipeline must already be present. The notebooks do **not** recreate Phase 1.

## 1. GroundTruth_TemporalSnapshots.ipynb

**Inputs:** Phase 1 temporal KG exports under `GC1/entity_resolution_output/` (including `temporal_kg_v2_output` or its ZIP), source Jira records and checkpoint metadata. The notebook searches and validates the expected files; later cells use existing in-memory variables created by earlier cells.

**Processing:**
1. Load graph nodes, links, source Jira histories and project checkpoints.
2. Reconstruct historical dependency intervals and screen suspect bulk changes.
3. Generate future-14-day Dependency, Schedule and Scope labels using eligible outcome events.
4. Reconstruct checkpoint-time Jira status, active dependencies, graph nodes and graph edges.
5. Check temporal availability, identifiers, duplicate records and exclusion of future information from predictive inputs.

**Key output directory:** `GC1/Temporal_snapshots/final_snapshots/`

```text
final_snapshots/
  nodes.parquet
  edges.parquet
  jira_states.parquet
  active_dependencies.parquet
  snapshot_manifest.parquet
  validation_report.csv
  dataset_metadata.json
```

Additional intermediate label and audit files are written under `GC1/Temporal_snapshots/`, including `schedule_risk_labels.csv`, `scope_risk_labels.csv`, `dependency_escalation_labels.csv`, `historical_structural_scope_events.parquet` and consolidated risk labels.

The configured final dataset comprises **30 project instances and 987 weekly checkpoints** with a **14-day prediction horizon**. Decision Risk is not a supervised target.

## 2. Feature_Engineering.ipynb

**Inputs:** The five `.parquet` tables from `final_snapshots/`, historical structural scope events, and the consolidated three-risk label files in `GC1/Temporal_snapshots/`.

**Processing:**
1. Calculate Jira workflow and recent status-change features.
2. Derive rolling event counts and stabilised growth/activity measures, with bulk-update screening.
3. Construct checkpoint-active dependency graph features.
4. Compute temporal KG structure and entity-activity features.
5. Derive past-only structural scope-change features.
6. Consolidate the feature groups and select **34 numerical predictors**.
7. Attach Schedule, Scope and Dependency labels, cross-check them against consolidated ground truth and export only if validation passes.

**Output directory:** `GC1/Temporal_snapshots/feature_engineering_output/`

```text
feature_engineering_output/
  tabpfn_labelled_dataset.parquet
  predictive_features.parquet
  selected_feature_list.csv
  feature_validation_report.csv
```

The labelled matrix has one row per project-week checkpoint. Future outcome labels are included for supervised training/evaluation, **not** as inference features.

## 3. Risk_Model_SHAP_Trigger.ipynb

**Required inputs:**

```text
GC1/Temporal_snapshots/feature_engineering_output/
  tabpfn_labelled_dataset.parquet
  selected_feature_list.csv
GC1/Temporal_snapshots/model_training_output/
  project_split_manifest.csv
```

**Important:** `project_split_manifest.csv` is an existing upstream artifact; these three notebooks do not create it. It must assign each project to `train`, `validation` or `test` without project overlap. The modelling notebook expects 987 rows split into 592 training, 198 validation and 197 test checkpoints (18/6/6 projects).

**Processing:**
1. Validate the 34-feature schema and frozen project split.
2. Train **TabPFN 2.5** for Schedule Risk and **CatBoost** for Scope and Dependency Risks.
3. Run training-only grouped cross-validation and choose alert thresholds using validation data.
4. Evaluate on the previously examined exploratory test partition, without tuning against those results.
5. Compute local SHAP explanations when needed: permutation SHAP for TabPFN (probability units) and TreeSHAP for CatBoost (log-odds units).
6. Apply stateful alert policies, chronological replay, on-demand explanation and persistent investigation scheduling.
7. Export model artifacts and structured requests for downstream retrieval.

**Output directory:** `GC1/Temporal_snapshots/model_training_efficient_investigation/`

```text
model_training_efficient_investigation/
  models/
  metrics/
  predictions/
  shap/
  triggers/
    validation_scheduled_investigations.jsonl
    exploratory_test_scheduled_investigations.jsonl
  audits/
  efficient_investigation_contract.json
```

CatBoost is configured with 250 iterations, depth 3, learning rate 0.035 and L2 regularisation 8. TabPFN uses the explicitly resolved v2.5 checkpoint and four estimators.

## How to run

1. Place the Phase 1 data and existing split manifest under the expected Google Drive paths.
2. Open `GroundTruth_TemporalSnapshots.ipynb` in Colab and run its cells **in order**, checking the final validation report and exported snapshot files.
3. Run `Feature_Engineering.ipynb` and confirm all validation checks pass before using the labelled matrix.
4. Enable a Colab GPU, configure TabPFN checkpoint access if necessary, and run `Risk_Model_SHAP_Trigger.ipynb` in order.
5. Inspect `efficient_investigation_contract.json`, the model metrics and the scheduled investigation JSONL files. Continue with the separate event-focused retrieval notebook.

## Reproducibility and interpretation

- The notebooks depend on saved upstream files, exact schemas, mounted Drive paths and, in places, state from preceding cells. They are **not** a standalone one-command application.
- Checkpoint features are constructed from historical information; future events are used only to define supervised outcomes.
- A model risk score and validation-selected alert threshold are **not** a calibrated probability of project failure.
- SHAP describes feature attribution to a model prediction; it does not establish a causal root cause.
- The exploratory test set was previously inspected during development and is not an untouched external holdout.
- If an assertion fails, inspect the missing file, source schema, checkpoint alignment or validation output rather than bypassing the check.
