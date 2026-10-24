# System Code

The notebooks implement a staged research prototype for point-in-time project risk analysis. Run the stages in dependency order; later notebooks expect data and outputs produced upstream.

| Stage | Folder | What it does |
|---|---|---|
| 1. Prepare data | [DataGeneration/](./DataGeneration/README.md) | Download/profile public Jira data, select project instances, and generate checkpoint-aware synthetic PRDs and meeting transcripts. |
| 2. Build knowledge | [KnowledgePipeline/](./KnowledgePipeline/README.md) | Extract entities/events, resolve mentions and create temporal graph data; evaluation notebooks review extraction and graph quality. |
| 3. Predict risks | [MachineLearning/](./MachineLearning/README.md) | Reconstruct historical snapshots, engineer features, train risk models and schedule investigations. |
| 4. Retrieve and demonstrate | [AgenticRAG/](./AgenticRAG/README.md) | Retrieve historical evidence and browse saved investigation reports or run the single-case Qwen demonstration. |

## Execution notes

- Most notebooks were designed for Google Colab and use Google Drive paths. Their datasets and intermediate outputs are not all included in this repository.
- Use the component README for each notebook's inputs, expected outputs, runtime and execution order.
- The stages are not wired into a single command or a pinned environment. Run notebook cells in order and retain their validation checks.
- The final runnable demonstration and its input/runtime requirements are documented in [AgenticRAG/README.md](./AgenticRAG/README.md).
