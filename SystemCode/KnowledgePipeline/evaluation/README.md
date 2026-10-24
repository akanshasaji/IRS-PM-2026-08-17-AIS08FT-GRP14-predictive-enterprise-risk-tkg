# Knowledge Pipeline Evaluations

These notebooks audit extraction, entity resolution, source identity, temporal information and graph construction. They are evaluation/review workflows, not prerequisites for building the main pipeline outputs. Run each against the matching saved extraction or graph artifacts described inside that notebook.

| Notebook | Overview |
|---|---|
| [entityextract_evaluation_FINAL_UPDATED_PATHS.ipynb](./entityextract_evaluation_FINAL_UPDATED_PATHS.ipynb) | Evaluates the entity-extraction outputs using the notebook's configured data paths. |
| [Entity_Resolution_Evaluation_V1_Colab.ipynb](./Entity_Resolution_Evaluation_V1_Colab.ipynb) | Reviews entity-resolution results against evaluation cases. |
| [ER_Evidence_Review_Two_Roots_Ready.ipynb](./ER_Evidence_Review_Two_Roots_Ready.ipynb) | Supports evidence review for entity-resolution decisions spanning two source roots. |
| [Temporal_KG_Source_Identity_Verification_Colab.ipynb](./Temporal_KG_Source_Identity_Verification_Colab.ipynb) | Checks source identity and provenance in temporal graph records. |
| [Full_Graph_Deterministic_Evaluation_V2_Colab.ipynb](./Full_Graph_Deterministic_Evaluation_V2_Colab.ipynb) | Runs deterministic checks over the full graph evaluation data. |
| [Full_Graph_Deterministic_V2_Timestamp_Reconciliation.ipynb](./Full_Graph_Deterministic_V2_Timestamp_Reconciliation.ipynb) | Reconciles timestamps used by the deterministic graph evaluation. |
| [experiments/](./experiments/README.md) | Contains the separate V4 temporal-graph evaluation methodology experiment. |

These evaluations depend on external or saved datasets that are not all committed here. Their checks and metrics have the scope defined by each notebook and should not be presented as a general guarantee of extraction correctness.
