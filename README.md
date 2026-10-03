# Galactus

**A reproducible experimentation system for building pretraining datasets and training small language models from scratch.**

Galactus is a reproducible experimentation system for constructing LLM pretraining datasets and training small language models from scratch. It connects data acquisition, extraction, deduplication, filtering, mixing, tokenization, packing, training, and evaluation behind one typed Python API and command-line interface.

> [!IMPORTANT]
> Galactus is under development. The commands and APIs below define the intended interface and are not yet implemented.

The project treats the dataset as part of the model system rather than as an anonymous input file. Its central question is:

> How do pretraining-data construction choices change the quality and efficiency of a language model when architecture, training budget, and evaluation are held constant?

Galactus is intended for two closely related roles:

- **Pretraining research engineers** who want to test hypotheses about deduplication, filtering, data mixtures, tokenization, or packing without assembling a new pipeline for every experiment.
- **Software engineers working on data infrastructure** who build reliable pipelines for collecting, extracting, deduplicating, filtering, mixing, versioning, and auditing the corpora used in model research.

**Features:**

- Build complete pretraining pipelines from multiple raw text sources.
- Remove exact, near-document, and repeated-line duplicates.
- Combine heuristic rules and learned classifiers in cost-aware filter cascades.
- Detect overlap between training documents and evaluation benchmarks.
- Create explicit, versioned data mixtures.
- Train byte-level BPE tokenizers and produce packed training shards.
- Train small decoder-only models from random initialization.
- Compare dataset variants under matched model and token budgets.
- Trace final sequences back to their sources and processing decisions.
- Measure corpus quality, model quality, throughput, storage, memory, GPU usage, and cost.
- Run the same recipe locally or through a parallel execution backend.
- Resume interrupted pipelines and reuse content-addressed artifacts.

## Installation

Galactus will require Python 3.12 or newer. From a source checkout, the planned development installation is:

```shell
uv sync --all-extras
uv run galactus --help
```

CPU execution will support pipeline development and small sample runs. A CUDA-capable GPU will be optional for learned filters and required for practical model-training experiments. Distributed execution will remain optional.

## Quick start

The demo processes a small web corpus, creates minimally processed, deduplicated, and fully curated variants, and trains the same small model on each.

Plan the experiment without executing it:

```shell
uv run galactus plan examples/demo.yaml
```

The plan resolves the sources, stages, dependencies, artifact fingerprints, estimated storage, and available execution backend.

Run the pipeline and its training variants:

```shell
uv run galactus run examples/demo.yaml
```

Galactus will provide terminal visualizations for immediate feedback. An illustrative result looks like this:

```text
GALACTUS  demo                                                COMPLETE

CORPUS FUNNEL
  WARC records acquired                              10,000 |####################|
  primary text extracted                              9,760 |###################-|
  URL and exact duplicates removed                    8,660 |#################---|
  MinHash/LSH near-duplicates removed                 7,760 |################----|
  heuristic filters passed                            7,160 |##############------|
  language and quality filters passed                 6,800 |##############------|
  benchmark decontamination passed                    6,796 |##############------|

  Repeated boilerplate lines removed                 84,200
  Packed-token utilization                            98.4%

CONTROLLED TRAINING
  50M parameters | 100M tokens per variant | seed 42

  Validation loss (lower is better)
  minimally processed   3.50 |####################
  deduplicated          3.31 |###################
  fully curated         3.18 |##################

  Loss over training
  minimally processed   4.8 4.2 3.9 3.7 3.5 | █▆▄▃▂
  deduplicated          4.7 4.1 3.7 3.5 3.3 | █▅▄▃▂
  fully curated         4.7 4.0 3.6 3.3 3.2 | █▅▃▂▁

PIPELINE TIME
  extract       00:41 |########
  deduplicate   01:36 |####################
  filter        00:58 |############
  tokenize      00:27 |######
  pack          00:11 |##

CONCLUSION
  The fully curated corpus improved validation loss by 9.1% under
  the same model architecture and training-token budget.

ARTIFACTS
  dataset       .galactus/runs/demo/datasets/fully-curated/
  tokenizer     .galactus/runs/demo/tokenizer/
  checkpoint    .galactus/runs/demo/checkpoints/fully-curated-final/
  lineage       .galactus/runs/demo/lineage.json
  report        .galactus/runs/demo/report.html
```

The displayed numbers demonstrate the output format; actual results will be measured from the selected data, pipeline, model, and hardware. Terminal reporting may also include tables, histograms, progress bars, filter-score distributions, duplicate-cluster summaries, sparklines, and resource-usage charts.

Open the interactive report or trace one document:

```shell
uv run galactus report demo --open
uv run galactus lineage demo --document DOC_ID
```

The lineage view reports where the document originated, how it was extracted, which duplicate cluster it belonged to, every filter score and decision, its mixture assignment, token offsets, packed sequence, and the model runs that consumed it.


## Pipeline Scope

Galactus will implement the principal stages of a modern pretraining-data pipeline:

### 1. Acquisition, extraction, and integration

Ingest text from multiple open sources, preserve licenses and source metadata, extract primary content from source formats, normalize encodings and fields, and map records into a shared document schema.

### 2. Multi-level deduplication

Remove exact URL and content duplicates; detect near-duplicate documents with word shingles, MinHash, and locality-sensitive hashing; resolve duplicate clusters with connected components; and identify frequently repeated boilerplate lines. The system will record cluster membership and the rule used to select each retained representative.

### 3. Filtering and decontamination

Apply configurable quality rules for document length, repetition, duplicate n-grams, punctuation, character composition, malformed text, and related failure modes. Add language, domain, and learned quality classifiers through a cost-aware cascade in which cheap deterministic filters precede expensive model-based filters. Detect and report exact or approximate overlap with evaluation datasets.

### 4. Corpus analysis and data mixing

Profile retained documents by source, domain, language, quality score, duplication, and token count. Construct explicit, versioned source mixtures instead of relying on accidental source proportions.

### 5. Tokenization, packing, and sharding

Train a byte-level BPE tokenizer, evaluate tokenization efficiency across sources, tokenize the corpus, and create deterministic training shards. Use length-aware packing to reduce padding and unnecessary truncation while retaining document-boundary and lineage metadata.

### 6. Small-model training

Use an open-source training runtime to train small decoder-only language models from random initialization. The models are experimental instruments for measuring dataset decisions, not an attempt to compete with frontier-scale models. Downloaded pretrained models may assist individual filtering stages, but they are not the final evaluation artifact.

### 7. Evaluation and reporting

Generate one comparison covering:

- corpus retention, duplication, diversity, and contamination;
- validation loss, perplexity, and selected downstream evaluations;
- memorization and qualitative model behavior;
- stage and end-to-end runtime;
- records, bytes, and tokens processed per second;
- CPU, memory, storage, and GPU utilization;
- estimated monetary cost and cost per useful training token;
- failures, retries, rejected documents, and reproducibility checks.

## Experimental Design

The principal demonstration will compare at least three dataset variants:

| Variant | Data treatment |
| --- | --- |
| Minimally processed | Extraction, normalization, tokenization, and packing only |
| Deduplicated | Minimal pipeline plus exact, near-document, and line-level deduplication |
| Fully curated | Deduplication, filtering, decontamination, explicit mixing, and final preparation |

Each model comparison will hold architecture, parameter count, training-token budget, optimizer settings, evaluation procedure, and—where practical—random seed constant. This isolates the effect of the data pipeline. Additional ablations may vary individual filters, MinHash/LSH parameters, mixture weights, tokenizer vocabulary size, or packing policy.

The final demonstration is therefore not simply a chatbot or a downloaded model. It is a reproducible execution showing how raw records became training sequences, how those sequences produced a model, which resources were consumed, and which engineering choices improved or harmed the result.

## Scope and Non-Goals

Galactus reproduces the structure and experimental discipline of a production-style system at a scale a three-person course team can execute. Corpus size, model size, and cluster size will be reduced; stage boundaries, data contracts, lineage, controls, and systems measurements will remain representative.

The course project will build a CLI/library rather than a hosted multi-tenant training service. Cluster scheduling, hardware provisioning, custom GPU kernels, a general ML compiler, and training frontier-scale models are outside the core scope. Multilingual specialization, code fill-in-the-middle transformation, long-context extension, and annealing are optional extensions.

## Course Project Fit

Galactus primarily falls under the course's **end-to-end data engineering pipeline** category: it acquires, prepares, cleans, integrates, analyzes, and versions data, then evaluates whether the result supports a downstream predictive model.

| Course objective | Galactus implementation |
| --- | --- |
| Acquire heterogeneous data | Versioned connectors for multiple open text sources |
| Model and integrate data | Shared document, decision, artifact, and lineage schemas |
| Transform and prepare data | Extraction, normalization, tokenization, packing, and sharding |
| Diagnose and clean data | Exact and approximate deduplication plus configurable filters |
| Build scalable pipelines | Partitioned execution and parallelizable expensive stages |
| Track provenance | Source manifests, stage fingerprints, decisions, and artifact lineage |
| Explore and visualize data | Corpus, quality, retention, and performance reports |
| Ensure reproducibility | Immutable artifacts, configurations, seeds, environment snapshots, and rerun instructions |
| Deliver downstream value | Small language models trained on the resulting datasets |
| Evaluate results | Controlled data, model, systems, and cost comparisons |

The project may add a research contribution through a new filtering, deduplication, data-selection, or pipeline-optimization method. Its primary contribution remains the data system and its experimental interface; small-model training is the downstream evaluation layer.

## Galactus Deliverables

- **Researcher-facing CLI and library:** A canonical typed Python API for defining and extending experiments, plus validated YAML support and CLI commands for executing, inspecting, resuming, and comparing them.
- **Working end-to-end pipeline:** Acquisition, extraction, normalization, integration, deduplication, filtering, decontamination, mixing, tokenization, packing, sharding, training, and evaluation.
- **Versioned data artifacts:** Documented intermediate datasets and final training shards derived from multiple raw sources.
- **Provenance and reproducibility:** Manifests, lineage records, stage fingerprints, configuration snapshots, seeds, source versions, filtering decisions, and environment definitions.
- **Controlled small-model experiments:** Models trained from scratch on dataset variants under matched training conditions.
- **Data and model evaluation:** Measurements demonstrating how processing decisions affect corpus properties and model behavior.
- **Systems and cost evaluation:** Runtime, throughput, resource utilization, storage, scaling behavior, and monetary-cost measurements, including ablations and failure analysis.
- **Reproducible public repository:** Source code, tests, configurations, acquisition scripts or sample data, dependency definitions, architecture documentation, and complete reproduction instructions.
- **Research paper and presentation:** A final report and presentation communicating the problem, system, experiments, results, limitations, and team contributions.

## Official Course Requirements

As a custom project, Galactus must:

- apply or extend course methods to solve a real big-data problem;
- be completed by a team of three, with each member's contributions documented;
- define the problem and its significance using the Heilmeier Catechism;
- explain how course methods will be applied;
- establish concrete milestones;
- submit a proposal PDF whose title page contains the project title, team name, members' names, and NetIDs;
- produce a research-style report with an introduction, problem formulation, related work, methods/architecture/design, results, and references;
- provide a GitHub repository with sufficient code, configurations, documentation, and instructions to reproduce the results;
- prepare a seven-minute presentation followed by five minutes of questions and answers, with all members present and prepared to answer technical questions;
- include the project name, team members, and GitHub link in the slides; and
- present the problem, approach, findings, challenges, and limitations rather than repeating the written report.

## Proposed Research Paper Structure

The final report will retain the required research-paper sections while presenting Galactus as a systems project:

1. **Abstract** — Problem, system, evaluation, and principal findings.
2. **Introduction** — Motivation, users, research questions, scope, and contributions.
3. **Background and Related Work** — Pretraining-data systems and relevant data-engineering methods.
4. **Problem Formulation** — Inputs, outputs, invariants, constraints, and quality and cost objectives.
5. **Interface and System Design** — User workflow, core abstractions, architecture, and data contracts.
6. **Pipeline Methods** — Acquisition through training-shard construction.
7. **Implementation** — Execution engine, storage, parallelism, caching, recovery, provenance, and reproducibility.
8. **Experimental Methodology** — Data, baselines, variants, training configuration, metrics, and controls.
9. **Data and Model Quality Results** — Corpus measurements and downstream small-model findings.
10. **Systems Performance and Cost Analysis** — Runtime, throughput, memory, storage, utilization, scaling, and cost.
11. **Ablations and Failure Analysis** — Component effects, errors, tradeoffs, and negative results.
12. **Limitations and Future Work** — Course-scale constraints and the path toward a larger research platform.
13. **Team Contributions and Reproducibility** — Individual responsibilities and exact reproduction procedure.
14. **References** — Cited systems, methods, and related work.

## License

Galactus is licensed under the terms in [MIT](MIT). Input datasets and downloaded models retain their own licenses and usage restrictions.
