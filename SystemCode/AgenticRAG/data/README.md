# AgenticRAG Demo Inputs

This folder contains the local inputs used by the final demonstration notebook:

| File | Purpose |
|---|---|
| [agentic_graphrag_retrieval_queue.jsonl](./agentic_graphrag_retrieval_queue.jsonl) | Checkpoint-specific investigation requests and retrieved evidence used for the single-case demonstration. |
| [test_predictions.parquet](./test_predictions.parquet) | Saved test predictions used to select and explain a demonstration case. |
| [llm_pilot_reports.jsonl](./llm_pilot_reports.jsonl) | Saved V5 report collection used by the 139-report browser. The notebook validates its 139 records and unique trigger IDs. |

The report browser can run directly from these local inputs without rerunning the full retrieval or report-generation pipelines. See [../README.md](../README.md) for complete run and hardware instructions.
