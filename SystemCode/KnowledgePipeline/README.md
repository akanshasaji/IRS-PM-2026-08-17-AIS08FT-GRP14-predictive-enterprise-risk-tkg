# Entity Extraction Pipeline

## 1. Overview

This notebook transforms heterogeneous project records into a common, provenance-aware JSONL representation for subsequent entity resolution and Temporal Knowledge Graph construction. The inputs are curated public Jira issue records and aligned, generated product requirements documents (PRDs) and meeting transcripts. **The Jira records are the public-data source; the PRDs and meetings are generated artifacts, not independently collected enterprise records.**

The approach combines deterministic extraction of structured fields and explicit identifiers, rule-based interpretation of meeting evidence, and **GLiNER** for semantic entity mentions in free text. It preserves time, source identifiers, original evidence and extraction method. This notebook performs **extraction, not canonical entity resolution, graph loading, risk-model training or GraphRAG inference**.

## Notebook map

| Notebook | Role |
|---|---|
| [Entity_Extraction.ipynb](./Entity_Extraction.ipynb) | Converts Jira records and generated PRDs/meetings into provenance-aware entity, relationship and event JSONL. The remainder of this README describes this development-history notebook and its final extraction outputs. |
| [Entity_Resolution.ipynb](./Entity_Resolution.ipynb) | Resolves extracted entity mentions across records before graph construction. |
| [Neo4j_Temporal_KG_Final.ipynb](./Neo4j_Temporal_KG_Final.ipynb) | Builds and validates the temporal knowledge graph from the upstream resolved/extracted data. |
| [evaluation/](./evaluation/README.md) | Contains extraction, entity-resolution, source-identity, timestamp and graph evaluation notebooks; one separate methodology experiment is documented under `evaluation/experiments/`. |

```text
Curated public Jira records        Official generated PRDs       Official generated meetings
             |                                |                                |
  Structured field/changelog      Section and requirement        Header/utterance parsing
  parsing + timestamp filtering   parsing + PRD chunking         + action/decision rules
             |                                |                                |
             |                          GLiNER semantic NER               GLiNER semantic NER
             |                                |                                |
             +--------------------------------+--------------------------------+
                                              |
                         Common entity / relationship / event schema
                                              |
                           Per-instance JSONL + validation report
                                              |
                         Global persisted-output audit (30 instances)
                                              |
                      Entity resolution / Temporal KG (downstream)
```

## 2. Notebook contents and review scope

The supplied notebook contains **222 code cells**, including exploratory pilot cells, generic helper definitions, production extraction functions, validation, batch processing and later corrective patches. It has no Markdown narrative cells; step headings are embedded in code comments. The implementation was reviewed cell by cell for its code structure, step descriptions, function definitions, paths, and saved execution outputs. **This review is static: the full Colab pipeline was not re-executed in this environment**, because the referenced Drive datasets and GLiNER runtime are external to the uploaded notebook.

| Notebook phase | Relevant labeled steps | Main purpose |
|---|---|---|
| Setup and pilot | 3–22 | Drive paths, source inspection, instance selection, temporal alignment |
| Common schema and Jira pilot | 10–45 | Local IDs, entity/relationship/event helpers, Jira extraction, deduplication, pilot JSONL |
| PRD pilot | 46–85 | Section parsing, explicit requirements, GLiNER chunks, PRD JSONL |
| Meeting pilot | 86–122 | Speakers, references, decisions, actions, dependencies, blockers, GLiNER |
| Pilot integration | 123–129 | Cross-source anchors, temporal and downstream-information checks |
| Production discovery | 130–137 and intermediate 135A–135C cells | Discover 30 instances; use manifest-selected official artifacts |
| Production extractors | 138–177 | Global Jira field mapping; JBEAP validation; format-tolerant PRD/meeting extraction |
| Batch processing | 178–190 | JSONL I/O, per-instance runner, validation, 28-instance batch |
| Edge-case corrections | 191–206 | Timestamp parsing, source-aware checks, SPR end-of-day boundary |
| Final audit and correction | 207–215 | Persisted-output totals, audit save, JBEAP meeting-date correction |

**Important execution note:** This is a development-history notebook, **not a clean linear script**. Several functions are redefined later to fix discovered edge cases; the *last executed definition* is authoritative. In particular, the final Jira alignment/extractor and validator are defined in steps **201–203**, and the final meeting extractor/runner in **212–213**. Do not use the earlier versions of those functions in a fresh production run.

## 3. Input data and Google Drive layout

The notebook mounts Google Drive and configures these paths:

```python
from google.colab import drive
drive.mount('/content/drive')

from pathlib import Path
GC1_ROOT = Path('/content/drive/MyDrive/GC1')
JIRA_ROOT = GC1_ROOT / 'GC1_Jira_Dataset'
PRD_MEETING_ROOT = GC1_ROOT / 'GC1_Generated_PRD_Meetings'
EXTRACTION_ROOT = GC1_ROOT / 'GC1_Entity_Extraction'
```

Expected relevant layout (example instance shown):

```text
MyDrive/GC1/
├── GC1_Jira_Dataset/
│   ├── 01_JBEAP/
│   │   └── jira_tickets.jsonl
│   ├── 24_DOCS/
│   │   └── jira_tickets.jsonl
│   └── ... (30 repository folders)
├── GC1_Generated_PRD_Meetings/
│   ├── 01_JBEAP/
│   │   ├── repository_instance_manifest.json
│   │   └── JBEAP-I01/
│   │       ├── generation_manifest.json
│   │       ├── snapshot_manifest.json
│   │       ├── prds/
│   │       │   ├── JBEAP-I01-PRD-V1.txt
│   │       │   └── JBEAP-I01-PRD-V1.meta.json
│   │       └── meetings/
│   │           ├── JBEAP-I01-M001.txt
│   │           ├── JBEAP-I01-M002.txt
│   │           └── corresponding .meta.json files
│   └── ...
└── GC1_Entity_Extraction/
    ├── 01_JBEAP/JBEAP-I01/
    │   ├── jira_extracted.jsonl
    │   ├── prd_extracted.jsonl
    │   ├── meetings_extracted.jsonl
    │   └── validation_report.json
    ├── ... (30 repository / instance folders)
    ├── production_batch_summary.json
    └── final_extraction_audit.json
```

The input discovery function selects **only PRDs and meetings listed in each instance's `generation_manifest.json`**, rather than indiscriminately processing all text files in a directory. This matters for JBEAP, whose folder contains additional generated versions not selected by the official schedule. Manifests are **control metadata for selection and validation**, not semantic evidence or model features.

The curated Jira input used by the extraction notebook is `jira_tickets.jsonl` per repository; **this notebook does not download the original public Jira corpus**. Dataset download and repository curation are separate upstream notebooks.

## 4. Methods

### 4.1 Jira: deterministic entities, links and temporal events

Each selected Jira ticket is parsed into structured entities (e.g., `PROJECT`, `ISSUE`, `PERSON`, `COMPONENT`), relationships (e.g., `HAS_ISSUE`, `ASSIGNED_TO`, `REPORTED_BY`, `CREATED_BY`, `HAS_COMPONENT`, `DEPENDS_ON`, `RELATED_TO`), and timestamped events from `status_history` and `full_changelog`. The extractor also preserves relevant issue attributes, comments, dependency and blocker evidence.

- Ticket inclusion is based on activity in the selected instance's time window, not simply creation date.
- Status transitions repeated across `status_history` and `full_changelog` are deduplicated.
- Common changelog fields are mapped to meaningful event types (`STATUS_CHANGE`, `SPRINT_CHANGE`, `ASSIGNEE_CHANGE`, etc.).
- Repository-specific fields use `CUSTOM_FIELD_CHANGE` while retaining `jira_field`, original old/new values, timestamp, actor and field metadata; unknown fields are **not** collapsed into `OTHER_CHANGE`.
- The final boundary helper interprets an instance end timestamp exactly at midnight as an **inclusive final calendar day**. This fixed SPR-I01's initial 47/300 ticket mismatch.

### 4.2 PRD: deterministic parsing plus GLiNER

PRD text is parsed for metadata, headings, explicit requirement IDs, milestones, ownership, dependencies, scope, acceptance criteria, risks and open questions. The parser accommodates different document layouts (e.g., heading aliases, table formats and numbered/bulleted sections). Explicit Jira issue keys are extracted separately from `REQ-*` requirement identifiers.

For semantic extraction, PRD sections are split into **section-aware chunks** (production setting `PRODUCTION_PRD_MAX_CHARS = 1800`) and passed to GLiNER. Chunks carry a section label and chunk ID during inference; they are intermediate processing objects, **not separately saved as a full `chunks.jsonl` corpus**.

### 4.3 Meetings: speakers, evidence states and GLiNER

Meeting transcripts are parsed into metadata, participant names/roles, speaker utterances, and explicit Jira/requirement references. Rule-based candidate detection retains the distinction between:

- decisions that are `MADE`, `TENTATIVE`, `PROPOSED`, `DEFERRED` or historical;
- dependencies that are `CONFIRMED`, `UNCERTAIN`, `CONDITIONAL` or questions;
- blocker evidence that is `ACTIVE`, `NEGATED`, `UNBLOCKING` or a question.

Only decisions classified as **`MADE`** create `DECISION_MADE` temporal events. The action, decision, dependency and blocker evidence remains available in dedicated arrays, including speaker and source text. GLiNER is applied to utterances for contextual semantic entities; **person mentions are excluded from the final meeting GLiNER labels**, because speaker and participant identities are extracted more reliably from the transcript structure.

### 4.4 GLiNER configuration

| Setting | Notebook value |
|---|---|
| Library | `gliner` |
| Model checkpoint | `urchade/gliner_medium-v2.1` |
| Inference threshold | `0.40` |
| Production acceptance threshold | `0.55` |
| Production semantic labels | `feature`, `system`, `component`, `team`, `person`, `milestone` |
| Final meeting semantic labels | `feature`, `system`, `component`, `team` |

GLiNER is used for flexible label-based extraction of free-text mentions without requiring a separate fine-tuned NER classifier for each label. It **complements** deterministic extraction; structured Jira fields and explicit identifiers are not replaced by model guesses. Predicted mentions are filtered for confidence, type suitability and structured-ID conflicts. The notebook installs GLiNER with `!pip install -q gliner` and loads the checkpoint through `GLiNER.from_pretrained(...)`. No hosted LLM API is required for this extraction stage.

## 5. Common JSONL schema

Each JSONL line is one complete source record. The high-level schema is:

```json
{
  "repository_project": "24_DOCS",
  "project_key": "DOCS",
  "instance_id": "DOCS-I01",
  "source": {"source_type": "JIRA", "source_id": "DOCS-9947", "source_timestamp": null},
  "entities": [],
  "relationships": [],
  "events": [],
  "decisions": [],
  "action_items": [],
  "blockers": [],
  "dependencies": [],
  "temporal": {},
  "attributes": {},
  "provenance": {}
}
```

*This is a schematic shape, not an actual extracted ticket. The arrays and dictionaries above are populated in the saved records.*

Entities receive **source-scoped local IDs** (e.g., `JIRA_DOCS-9947_E001`) and may retain `name`, `type`, `attributes`, `extraction_method`, and `confidence`. They are **occurrences, not globally canonical entities**. Relationships and events refer to these local entities. Provenance retains source type/ID, instance, file name and evidence text; some semantic mentions also retain chunk/section or speaker/utterance information.

## 6. Temporal integrity and special cases

The final implementation applies **source-aware validation**:

1. **Jira events:** must have parseable timestamps within the selected Jira instance window, using inclusive end-of-day handling where applicable.
2. **Official PRD and meeting artifacts:** are checked against the generation schedule rather than automatically rejected when their official date is beyond the Jira activity window.
3. **Human-readable PRD dates:** e.g., `24 March 2019`, are normalized to UTC-aware datetimes.
4. **JBEAP meeting-date discrepancy:** the transcript for `JBEAP-I01-M002` displays `2015-12-13`, while the official schedule and artifact metadata identify `2016-01-17`. The final meeting extractor uses the **official timestamp for canonical temporal placement** and retains the displayed transcript date as raw provenance.

These controls prevent the temporal graph from treating stale or inconsistently formatted dates as authoritative event times.

## 7. Outputs and validated results

The final persisted-output audit reported:

| Metric | Validated count |
|---|---:|
| Project instances passing validation | **30 / 30** |
| Jira source records | **5,737** |
| PRD source records | **30** |
| Meeting source records | **60** |
| Total source records | **5,827** |
| Entity occurrences | **61,392** |
| Relationships | **63,305** |
| Temporal events | **82,577** |
| Dependency evidence | **7,709** |
| Blocker evidence | **443** |
| Action evidence | **207** |
| Decision evidence | **414** |
| `CUSTOM_FIELD_CHANGE` events | **14,138** |
| `OTHER_CHANGE` events | **0** |

These are **extraction occurrence/evidence counts**, not the number of unique canonical entities, unique dependencies or verified causal relationships. The global figures are supported by the notebook's saved Step 208 output. After the final JBEAP correction, its local validation also passed (Step 215); the project's final saved audit was subsequently reconfirmed as 30/30 in the project workflow.

## 8. How to inspect saved files

Open any `*.jsonl` file with a text editor, or inspect individual records in Colab/Python without loading the entire file:

```python
import json
from pathlib import Path

instance_dir = Path('/content/drive/MyDrive/GC1/GC1_Entity_Extraction/24_DOCS/DOCS-I01')

with (instance_dir / 'jira_extracted.jsonl').open(encoding='utf-8') as f:
    first_record = json.loads(next(line for line in f if line.strip()))

print(first_record['source'])
print('Entities:', len(first_record['entities']))
print('Relationships:', len(first_record['relationships']))
print('Events:', len(first_record['events']))
print('First entity:', first_record['entities'][0] if first_record['entities'] else None)

with (instance_dir / 'validation_report.json').open(encoding='utf-8') as f:
    report = json.load(f)
print('Instance validation:', report.get('success'))
```

Inspect the global audit:

```python
with Path('/content/drive/MyDrive/GC1/GC1_Entity_Extraction/final_extraction_audit.json').open(encoding='utf-8') as f:
    audit = json.load(f)
print(audit['projects_validated'], audit['jira_records'], audit['total_source_records'])
```

A basic cross-project count check (no model execution required):

```python
from pathlib import Path
root = Path('/content/drive/MyDrive/GC1/GC1_Entity_Extraction')
for name, expected in [('jira_extracted.jsonl', 5737),
                       ('prd_extracted.jsonl', 30),
                       ('meetings_extracted.jsonl', 60)]:
    files = list(root.glob(f'*/*/{name}'))
    actual = sum(sum(bool(line.strip()) for line in p.open(encoding='utf-8')) for p in files)
    print(name, 'files=', len(files), 'records=', actual, 'expected=', expected)
    assert actual == expected
```

## 9. Reproducing the extraction

**Environment:** Google Colab with Google Drive mounted, Python 3, the `gliner` package, and the notebook's standard Python dependencies. GLiNER model weights must be downloaded on first use; a GPU is helpful but the exact runtime/versions were not pinned in the notebook. The upstream curated Jira and generated-artifact folders must already exist at the configured Drive paths.

**Recommended reproduction procedure:**

1. Make a backup of `GC1_Entity_Extraction/` before running any cells that save or overwrite files.
2. Mount Drive and confirm the three root paths exist (initial setup cells).
3. Load the GLiNER model and run the helper definitions needed for the production pipeline.
4. Ensure the **final** definitions are active: manifest-based discovery (135C), timestamp parsing (195), official artifact schedule (196), boundary normalization (199), Jira alignment/extraction (201–202), source-aware validator (203), and meeting extractor/runner (212–213). Earlier definitions in the notebook are historical iterations.
5. Test on a single instance using `run_production_instance(inputs, save_outputs=False)` before any write. Note that the generic runner is defined at Step 183, while its helper functions are updated later.
6. Only if re-extraction is intended, run the batch on the required instances and save per-instance JSONL files.
7. Run the persisted-output audit (Steps 207–208) **after any corrections**, then save the definitive global audit (Step 209). Avoid trusting an earlier batch summary after subsequent fixes.

**Reproducibility limitation:** Because this notebook is an iterative Colab development record with later function redefinitions and no pinned dependency lockfile, a fresh top-to-bottom run is **not guaranteed to reproduce the final output unchanged** without following the final-function ordering above. For submission, the validated JSONL files and final audit are the primary reproducible artifacts; a future cleanup could extract the final function definitions into a separate production-only script/notebook.

## 10. Validation coverage and limitations

The production validator checks expected source counts, required top-level sections, project/instance consistency, event timestamp parseability, source-aware temporal rules, and absence of `OTHER_CHANGE`. The pipeline also checks duplicated Jira status transitions and preserves unknown Jira field identities.

These checks establish **structural and temporal consistency**, not a manually labeled precision/recall score for GLiNER. They do not guarantee that every semantic mention, action, decision or blocker has been correctly interpreted. Rule-based candidates may include repeated evidence, and `MADE` decisions are heuristic classifications. Entity resolution and cross-source canonical linking are intentionally left to the downstream pipeline.

## 11. Submission and attribution

Submit the following together where permitted:

- `Entity_Extraction.ipynb` — implementation and recorded exploratory/production workflow.
- `GC1_Entity_Extraction/` — final per-instance JSONL files and validation reports.
- `final_extraction_audit.json` — global audit of persisted outputs.
- This README — setup, data contract, interpretation and inspection guidance.

The extraction work covers structured Jira processing, semantic GLiNER extraction from PRD/meeting text, common JSONL construction, and validation. Generation of the synthetic PRDs/meetings, canonical entity resolution, Temporal KG loading, and downstream prediction/explanation models are separate components of the overall project.

---

**Note for reviewers:** Source files and generation manifests may not be packaged inside the extraction output directory. To re-run extraction rather than merely inspect its outputs, access to the original curated Jira and officially generated PRD/meeting folders is required.
