# AgenticRAG Evaluations and Audit Artifacts

This folder holds evaluation notebooks and saved diagnostic/review tables for the final retrieval and report-generation stage. These are bounded evaluation workflows, not a fully independent certification of the system.

## Notebooks

| Notebook | Overview |
|---|---|
| [Phase3_GraphRAG_Retrieval_Precision_at_5_V1_Colab.ipynb](./Phase3_GraphRAG_Retrieval_Precision_at_5_V1_Colab.ipynb) | Evaluates retrieval ranking at Precision@5 using its configured cases and labels. |
| [Phase3_Explanation_Faithfulness_Evaluation_V1 .ipynb](./Phase3_Explanation_Faithfulness_Evaluation_V1%20.ipynb) | Reviews explanation/evidence faithfulness using the notebook's configured cases. The filename has a trailing space before `.ipynb`. |

## Saved audit files

| File | Role |
|---|---|
| [queue_preflight.csv](./queue_preflight.csv) | Input request and queue validation. |
| [evidence_preflight.csv](./evidence_preflight.csv) | Evidence screening and historical-eligibility diagnostics. |
| [llm_report_audit.csv](./llm_report_audit.csv) | Report-format and citation diagnostics. |
| [claim_support_review.csv](./claim_support_review.csv) | Claim-level review worksheet; this alone does not reproduce the separate 40-claim and 155-claim AI-assisted evaluations. |
| [evidence_usefulness_diagnostic.csv](./evidence_usefulness_diagnostic.csv) | Diagnostic evidence-usefulness assessment. |

The frozen V5 full run completed 139 requests. The original retrieval Precision@5 (37.3%) and sampled claim support (70% on 40 claims) were **AI-assisted preliminary assessments**. Their corresponding evaluation notebooks and labelled datasets must be supplied separately for exact recomputation. Source-level checks of 155 claims did not establish independently verified overall factual accuracy. A valid evidence ID is not a guarantee of a true claim.
