# Data Generation and Preparation

This folder contains the first stage of the project workflow. It prepares public Jira data and creates selected project instances with generated, temporally aligned PRDs and meeting transcripts. The generated documents are synthetic research inputs, not independently collected company records.

## Notebooks

| Notebook | Overview |
|---|---|
| [Download_dataset.ipynb](./Download_dataset.ipynb) | Retrieves the public Jira dataset used by later preparation steps. |
| [Inspect and Select Projects JIRA.ipynb](./Inspect%20and%20Select%20Projects%20JIRA.ipynb) | Profiles Jira projects and selects suitable project histories/instances for the study. |
| [Jira_PRD_Meeting_Generation_Stage_Instance_Aware.ipynb](./Jira_PRD_Meeting_Generation_Stage_Instance_Aware.ipynb) | Generates PRDs and meeting transcripts for selected project instances while carrying Jira context, dates and provenance forward. |

## Order and handoff

1. Run the dataset-download notebook and verify that the dataset is available at the path expected by the notebook.
2. Run the project-inspection/selection notebook to choose and curate the project records.
3. Run the instance-aware generation notebook for the selected instances. Review its manifests and temporal/provenance checks.
4. Pass the curated Jira records and generated artifacts to [KnowledgePipeline](../KnowledgePipeline/README.md).

The notebooks use external files and may require Google Drive or Colab configuration. Paths and generated outputs are defined within each notebook; this repository does not contain the full raw dataset or all generated artifacts. Run cells in order and do not bypass failed data-quality checks.