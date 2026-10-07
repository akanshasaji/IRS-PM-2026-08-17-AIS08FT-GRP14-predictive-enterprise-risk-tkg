# [ Practice Module ] Project Submission
---

## SECTION 1 : PROJECT TITLE

# Predictive Enterprise Project Risk Analysis using Temporal Knowledge Graphs


---

## SECTION 2 : EXECUTIVE SUMMARY / PAPER ABSTRACT

Software projects generate large volumes of heterogeneous information across issue tracking systems, requirements documents, meeting discussions, and other project artefacts. Important knowledge about requirements, dependencies, risks, decisions, project progress, and potential delivery problems is often distributed across these different sources and evolves over time.

This project investigates the use of **Large Language Models (LLMs), Temporal Knowledge Graphs, Machine Learning, and Agentic Retrieval-Augmented Generation (RAG)** to construct an intelligent software project analysis and decision-support system.

The project uses historical software project data derived primarily from public Jira repositories. To create a richer multi-source project environment, selected Jira project instances are supplemented with synthetically generated Project Requirements Documents (PRDs) and meeting transcripts. Generated artefacts are temporally constrained by the Jira information available at the corresponding point in the project timeline and maintain provenance information linking them to their supporting source records.

An LLM-based information extraction pipeline is then used to extract structured entities, relationships, events, and temporal information from Jira issues, PRDs, and meeting transcripts. Entity resolution is performed before the extracted information is integrated into a **temporal knowledge graph**, allowing the state and evolution of software projects to be represented over time.

In parallel, temporal project features are constructed for a **machine-learning prediction pipeline**, with XGBoost used to investigate predictive signals associated with software project outcomes and risks.

Finally, an **Agentic Graph RAG** layer combines knowledge retrieval, graph-based reasoning, machine-learning outputs, and LLM-based interaction to support project-related queries and explanations.

The overall system is organised into four major components:

1. **Data Generation and Preparation**
2. **Knowledge Extraction and Temporal Knowledge Graph Construction**
3. **Machine Learning and Predictive Modelling**
4. **Agentic Graph RAG and Decision Support**

The project will also evaluate the quality of the generated dataset, information extraction and knowledge graph construction, machine-learning performance, and the final retrieval and reasoning system.

---

## SECTION 3 : PROJECT OBJECTIVES

The primary objective of this project is to investigate how structured knowledge representation, machine learning, and LLM-based reasoning can be combined to support software project intelligence.

The project aims to:

- Construct a temporally organised, multi-source software project dataset from public Jira records and generated project artefacts.
- Extract project entities, relationships, events, and temporal information using LLM-based information extraction.
- Perform entity resolution across heterogeneous project documents.
- Construct a temporal knowledge graph representing the evolution of software projects.
- Engineer temporal project features suitable for machine-learning models.
- Train and evaluate an XGBoost-based predictive model.
- Develop a Graph RAG retrieval mechanism over the temporal knowledge graph.
- Develop an agentic reasoning layer capable of combining retrieved project knowledge and predictive outputs.
- Evaluate the individual system components as well as the overall end-to-end architecture.

---

## SECTION 4 : SYSTEM ARCHITECTURE

The project currently follows the high-level pipeline below:

```text
Public Jira Dataset
        |
        v
+-----------------------------+
| 1. Data Generation          |
|                             |
| - Dataset profiling         |
| - Project selection         |
| - Temporal snapshots        |
| - PRD generation            |
| - Meeting generation        |
| - Provenance manifests      |
+-------------+---------------+
              |
              | Jira + PRDs + Meetings
              v
+-----------------------------+
| 2. Knowledge Graph          |
|                             |
| - Entity extraction         |
| - Relation extraction       |
| - Temporal extraction       |
| - Entity resolution         |
| - Temporal KG construction  |
| - KG evaluation             |
+-------------+---------------+
              |
              | Temporal Knowledge Graph
              |
       +------+------+
       |             |
       v             v
+-------------+  +------------------+
| 3. Machine  |  | 4. Agentic      |
| Learning    |  | Graph RAG       |
|             |  |                  |
| - Features  |  | - Graph retrieval|
| - XGBoost   |  | - Context        |
| - Evaluation|  | - Agent reasoning|
| - XAI       |  | - Explanations   |
+------+------+  +--------+---------+
       |                  |
       +---------+--------+
                 |
                 v
       Project Intelligence /
         Decision Support
```

> The architecture may be refined as implementation and evaluation progress.

---

## SECTION 5 : REPOSITORY STRUCTURE

The repository follows the project submission structure provided for the Practice Module.

```text
Project/
│
├── Miscellaneous/
│
├── ProjectReport/
│
├── SystemCode/
│   │
│   ├── 01_DataGeneration/
│   │
│   ├── 02_KnowledgeGraph/
│   │
│   ├── 03_MachineLearning/
│   │
│   ├── 04_AgenticRAG/
│   │
│   └── requirements.txt
│
├── Video/
│
└── README.md
```

### SystemCode Components

#### `01_DataGeneration`

Contains the data preparation and synthetic artefact generation pipeline.

This includes:

- Jira dataset profiling
- Selection of suitable project instances
- Construction of temporal snapshots
- Generation of PRDs
- Generation of meeting transcripts
- Generation of provenance and snapshot manifests

#### `02_KnowledgeGraph`

Contains the information extraction and knowledge graph construction pipeline.

This includes:

- Entity extraction
- Relationship extraction
- Temporal information extraction
- Entity resolution
- Knowledge graph population
- Temporal knowledge graph construction
- Knowledge graph evaluation

#### `03_MachineLearning`

Contains the predictive modelling pipeline.

This includes:

- Temporal feature engineering
- Training and validation datasets
- XGBoost modelling # needs to change
- Model evaluation
- Feature importance and explainability

#### `04_AgenticRAG`

Contains the retrieval and agentic reasoning components.

This includes:

- Graph-based retrieval
- Temporal knowledge retrieval
- Context construction
- Integration of predictive model outputs
- Agent tools and orchestration
- Response generation and explanation
- RAG/agent evaluation

---

## SECTION 6 : DATASET

The project uses publicly available historical Jira software project data as its primary source.

Selected Jira projects are organised into project instances and temporal snapshots. For selected project instances, additional project artefacts are generated using LLMs to simulate information that may normally exist outside an issue tracking system.

Generated artefacts include:

- Project Requirements Documents (PRDs)
- Meeting transcripts

Generated artefacts maintain provenance linking them to their supporting Jira records and are constrained according to the project information available at the corresponding point in time.

A typical generated project instance follows the structure:

```text
PROJECT-I01/
│
├── snapshot_manifest.json
├── generation_manifest.json
│
├── prds/
│   └── ...
│
└── meetings/
    └── ...
```

The manifests are used to maintain temporal consistency, provenance, reproducibility, and support later evaluation.

---

## SECTION 7 : TECHNOLOGIES

The current implementation is primarily Python-based and developed using Jupyter/Google Colab notebooks.

Technologies currently planned or under evaluation include:

- Python
- Jupyter Notebook / Google Colab
- OpenAI GPT models
- XGBoost 
- Knowledge Graph technologies
- Graph-based retrieval
- Large Language Models
- Agentic RAG

Additional libraries and infrastructure will be documented as implementation progresses.

---

## SECTION 8 : CREDITS / PROJECT CONTRIBUTION

| Official Full Name | Student ID | Work Items | Email (Optional) |
| Akansha Sajimon | A0352134N | To be updated | akanshasajimon@u.nus.edu |
| Cho Laqshya | A0357002R | To be updated | cho.laqshya@u.nus.edu |
| Rajamanickam Abirami | A0354513J | To be updated | abiramirajamanickam@u.nus.edu |
| Ravikumar Prithika | A0354287U | To be updated | prithika.ravikumar@u.nus.edu |

---

## SECTION 9 : VIDEO OF SYSTEM MODELLING & USE CASE DEMO

The final system modelling and demonstration video will be added here.

**Video:** To be added.

Supporting material will also be available under:

```text
Video/
```

---

## SECTION 10 : USER GUIDE

Detailed installation and usage instructions will be provided once the implementation environment has stabilised.

Current development is primarily performed using:

- Python
- Jupyter Notebook
- Google Colab

The final user guide will document:

1. Environment setup
2. Required dependencies
3. Dataset preparation
4. Data generation
5. Knowledge extraction
6. Knowledge graph construction
7. Machine-learning pipeline
8. Agentic RAG execution
9. Evaluation procedures

Refer to the final project report under:

```text
ProjectReport/
```

---

## SECTION 11 : PROJECT REPORT / PAPER

The final project report will be available under:

```text
ProjectReport/
```

Planned report sections include:

- Executive Summary / Abstract
- Problem Background
- Literature Review / Related Work
- Project Objectives
- Dataset and Data Generation
- System Architecture
- Information Extraction
- Temporal Knowledge Graph
- Machine Learning
- Agentic Graph RAG
- Experimental Methodology
- Evaluation
- Results and Discussion
- Limitations
- Future Work
- Conclusion
- References
- Appendices

---

## SECTION 12 : EVALUATION

The project is intended to evaluate the system at multiple levels rather than relying solely on final end-to-end performance.

Planned evaluation areas include:

### Data Generation

- Temporal consistency
- Provenance correctness
- Source grounding
- Generated artefact quality

### Information Extraction & Knowledge Graph

- Entity extraction quality
- Relationship extraction quality
- Entity resolution accuracy
- Temporal consistency
- Knowledge graph completeness and correctness

### Machine Learning

- Predictive performance
- Baseline comparison
- Temporal validation
- Feature importance
- Explainability

### Agentic Graph RAG

- Retrieval quality
- Groundedness
- Answer correctness
- Temporal reasoning
- Usefulness of predictive information
- End-to-end system performance

Detailed metrics and experimental protocols will be documented as the evaluation framework is finalised.

---

## SECTION 13 : MISCELLANEOUS

Additional project materials that do not belong directly to the implementation or report will be stored under:

```text
Miscellaneous/
```

This may include:

- Architecture diagrams
- Experimental notes
- Supporting documentation
- Presentation material
- Additional evaluation artefacts

---
