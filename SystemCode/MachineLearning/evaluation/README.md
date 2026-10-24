# Machine Learning Evaluations

The notebooks in this folder validate risk labels, prediction methodology, dependency evidence and temporal-leakage controls. They consume saved Phase 2 snapshots, labels, predictions or source-review cases; consult the input cells in each notebook before running.

| Notebook | Overview |
|---|---|
| [Phase2_Risk_Label_Validation_Audit_V1_Colab.ipynb](./Phase2_Risk_Label_Validation_Audit_V1_Colab.ipynb) | Audits the risk-label construction and validation cases. |
| [Phase2_Dependency_GroundTruth_Validation_Final.ipynb](./Phase2_Dependency_GroundTruth_Validation_Final.ipynb) | Checks dependency-risk ground truth against its supporting records. |
| [Phase2_Independent_Risk_Prediction_Evaluation_V1_Colab.ipynb](./Phase2_Independent_Risk_Prediction_Evaluation_V1_Colab.ipynb) | Evaluates risk predictions in a separate evaluation workflow. |
| [Phase2_Scope_Dependency_60_Case_Source_Audit_V3_Colab .ipynb](./Phase2_Scope_Dependency_60_Case_Source_Audit_V3_Colab%20.ipynb) | Performs a source-level audit of the 60 scope/dependency review cases. The filename includes a trailing space before `.ipynb`. |
| [Phase2_Temporal_Leakage_Early_Warning_Audit_V1_Colab.ipynb](./Phase2_Temporal_Leakage_Early_Warning_Audit_V1_Colab.ipynb) | Audits point-in-time and temporal-leakage behavior for early-warning predictions. |

These notebooks complement, but do not replace, the model training and threshold-selection workflow in [MachineLearning/README.md](../README.md). A review sample or audit result is limited to its cases and criteria; it is not automatically a population-wide performance estimate.
