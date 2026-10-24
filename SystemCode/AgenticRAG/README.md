# Agentic GraphRAG, Investigation Reports and Demo

This folder is the final project stage. It connects checkpoint-specific risk investigations with graph/Jira evidence, generates or displays investigation reports, and provides the professor-facing demo. The notebooks are related but are not one fully automated workflow.

## Notebook map

| Notebook | Purpose | Execution status |
|---|---|---|
| [Event_Focused_Retrieval.ipynb](./Event_Focused_Retrieval.ipynb) | Retrieves and validates historical evidence for scheduled risk investigations using temporal graph traversal, BM25/dense retrieval, reciprocal-rank fusion and reranking. | Upstream notebook; requires Phase 2 snapshots and investigation queues. |
| [Agentic_GraphRAG_V5_Full_139_Free_GPU.ipynb](./Agentic_GraphRAG_V5_Full_139_Free_GPU.ipynb) | Full V5 report-generation workflow for the 139-request run using Qwen. | Batch-generation workflow; requires the complete upstream data and GPU runtime. |
| [GC1_Professor_Demo_Live_and_139_Report_Browser.ipynb](./GC1_Professor_Demo_Live_and_139_Report_Browser.ipynb) | Browses saved V5 reports and separately offers a single-case, checkpoint-safe Qwen evidence-summary demonstration. | **Final runnable demonstration notebook.** The report browser uses the included 139-report JSONL; live inference also needs a CUDA GPU. |
| [V5_Supplementary_Saved_Outputs.ipynb](./V5_Supplementary_Saved_Outputs.ipynb) | Presents supplementary saved-run outputs and analysis. | Archive/support material, not the standalone runnable demo. |
| [evaluation/](./evaluation/README.md) | Documents retrieval and report/claim-support evaluation notebooks and saved audit files. | Evaluation workflows; inputs and limitations are described in that folder. |

## Pipeline context

```text
Phase 2 snapshots + scheduled investigations
                  |
                  v
      Event_Focused_Retrieval.ipynb
                  |
                  v
     evidence queue (JSONL) --> V5 batch report generation
                  |                        |
                  +------------------------+
                                           v
               professor demo: saved-report browser
                    + separate single-case Qwen summary
```

The event-focused retrieval notebook is designed for Google Colab with Google Drive mounted at `/content/drive/MyDrive/GC1` (override with `GC1_ROOT`). It expects five Parquet snapshot tables and the validation/exploratory investigation queues from the Machine Learning stage. It writes a retrieval queue, manifest, validation and sufficiency summaries under `Temporal_snapshots/graph_hybrid_retrieval_event_focused_full_v2_output/`. See the notebook for dependency installation, retrieval settings and validation contracts.

The frozen V5 run is documented as 139 requests processed (137 local-Qwen reports and two rule-based abstentions). These counts describe outputs, not report accuracy. Retrieval metrics and claim-support reviews in the evaluation materials are preliminary or sample-specific; evidence IDs and a successful schema check do not prove factual correctness.

## Results

To inspect the saved run outputs and reports, see the [results/](./results/) folder.

## Final demo prerequisites

The [data/](./data/README.md) folder includes `llm_pilot_reports.jsonl`, `test_predictions.parquet` and `agentic_graphrag_retrieval_queue.jsonl`. The report browser validates that the saved report file contains exactly 139 records with unique trigger IDs.

## Run the professor demo in Colab

1. Open [GC1_Professor_Demo_Live_and_139_Report_Browser.ipynb](./GC1_Professor_Demo_Live_and_139_Report_Browser.ipynb) in Colab. For live Qwen inference, select a **T4 GPU** (or another compatible CUDA GPU). The saved-report browser itself is GPU-independent.
2. Ensure the notebook working directory is `SystemCode/AgenticRAG`, or set `REPO_ROOT` in its environment-configuration cell to the absolute path of that folder. The notebook resolves `data/` relative to `Path.cwd()` and writes `demo_outputs/` relative to `REPO_ROOT`.
3. Keep the three included demo inputs available under `data/`.
4. Run the notebook cells from the top, including the lightweight dependency setup. Confirm the input-discovery output resolves all intended files.
5. Run the report validation and browser cells. Search/filter the saved reports and export the summary CSV; the notebook writes it to `demo_outputs/`.
6. To demonstrate fresh single-case summarization, continue through the case/evidence display, Qwen initialization and generation cells. The notebook installs Transformers, Accelerate, bitsandbytes and SentencePiece and downloads `Qwen/Qwen2.5-3B-Instruct`. This phase requires a working CUDA runtime and model access.
7. Review the generated JSON and its cited evidence. The live summary is separate from the saved V5 report set and must not add claims unsupported by the checkpoint-safe evidence.

For a local notebook session, use a Python kernel with the notebook dependencies. The report browser does not require a GPU. The Qwen cells explicitly fail without CUDA, so a CPU-only run can use the browser but not live model inference.

## Evidence and interpretation safeguards

- Only use evidence available at or before the investigation checkpoint. Do not use later outcomes to support a historical explanation.
- Preserve evidence provenance and quality labels. Graph metadata or inferred checkpoint transitions are not quotations from original Jira text.
- A risk score is not necessarily a calibrated probability of project failure. SHAP attribution describes model behavior, not causality.
- A generated report or valid evidence ID is not independently verified truth. Review cited source records and the evaluation scope before making factual claims.
- Keep run artifacts, credentials and model tokens out of source control unless inclusion is explicitly permitted.
