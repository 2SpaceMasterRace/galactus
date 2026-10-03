# Galactus

Galactus is an implementation of a modern LLM pretraining data pipeline. It transforms raw, heterogeneous web data into documented, deduplicated, filtered, mixed, tokenized, and packed training datasets. Small language models trained under controlled conditions provide a downstream evaluation of whether each data-engineering decision improves the resulting corpus.

## What the Final System Does

Galactus starts with raw text from multiple sources and produces versioned, training-ready dataset shards through a reproducible pipeline:

```text
Raw sources
  -> acquisition and extraction
  -> normalization and integration
  -> exact, document-level, and line-level deduplication
  -> heuristic and model-based filtering
  -> quality analysis and data mixing
  -> tokenizer training and application
  -> sequence packing and training shards
  -> controlled small-model pretraining
  -> data, systems, cost, and model evaluation
```

The final demonstration is not a downloaded pretrained model. Galactus uses an open-source training implementation to train small language models from scratch on controlled variants of the produced data. Holding the architecture, training-token budget, and other experimental settings constant makes it possible to measure whether pipeline choices such as deduplication or filtering improve downstream model behavior. Pretrained open-source models may still be used as components of model-based filtering, but they are not the final evaluation artifact.

## Pipeline Scope

Galactus will implement the principal stages used in modern LLM pretraining-data systems:

- **Acquisition and extraction:** Ingest raw web and text data, preserve source metadata, extract primary text from source formats, and normalize the result into a shared document schema.
- **Multi-level deduplication:** Remove exact URL and content duplicates; identify near-duplicate documents with word shingles, MinHash, and locality-sensitive hashing; form duplicate clusters with graph or connected-component methods; and remove frequently repeated boilerplate lines.
- **Heuristic filtering:** Apply configurable document-quality rules covering length, repetition, duplicate n-grams, punctuation, character composition, malformed text, and other low-quality patterns. Evaluate established rule sets and project-specific extensions rather than treating their thresholds as universally correct.
- **Benchmark decontamination:** Detect and record overlap between the training corpus and evaluation data using exact and approximate n-gram matching.
- **Model-based filtering:** Apply language identification, topic or domain classification, and learned quality scoring. Organize filters as a cost-aware cascade so inexpensive deterministic checks run before more expensive models.
- **Data analysis and mixing:** Measure the retained corpus by source, domain, language, quality, duplication, and token count; then construct explicit source mixtures for controlled experiments.
- **Tokenization:** Train and evaluate a byte-level BPE tokenizer, record its vocabulary and pre-tokenization configuration, and compare tokenization efficiency across the retained data sources.
- **Packing and formatting:** Convert tokenized documents into deterministic, training-ready shards and use length-aware packing to reduce padding and unnecessary truncation while preserving document-boundary metadata.
- **Small-model pretraining:** Train small decoder-only language models from random initialization on controlled corpus variants using the same architecture, token budget, optimization settings, and evaluation procedure.
- **Ablation and systems evaluation:** Remove or vary individual stages and measure their effect on corpus quality, model quality, throughput, memory, storage, and cost.

The implementation reproduces the architecture and engineering decisions of a production-style pipeline at a scale that a three-person course team can execute and evaluate. Corpus size and model size are reduced; the stages, interfaces, provenance, and experimental controls remain representative. Multilingual specialization, code-specific fill-in-the-middle transformations, long-context extension, and annealing are optional extensions rather than core deliverables.

## Course Project Fit

Galactus primarily falls under the course's **end-to-end data engineering pipeline** category: acquire and clean data, explore and analyze it, and evaluate the results through predictions.

| Course requirement | Galactus |
| --- | --- |
| Acquire data | Collect raw web-text datasets from multiple sources. |
| Transform and prepare | Extract, normalize, tokenize, and pack text. |
| Clean data | Deduplicate and apply heuristic and model-based quality filters. |
| Integrate sources | Standardize and combine heterogeneous text collections. |
| Build a scalable pipeline | Execute stages with parallel or distributed processing. |
| Track provenance | Record sources, versions, transformations, and filtering decisions. |
| Explore and visualize | Analyze duplication, quality, domains, token distributions, and pipeline performance. |
| Support downstream analysis | Train small language models on the resulting datasets. |
| Evaluate results | Compare model quality across controlled pipeline variants. |
| Ensure reproducibility | Publish code, configurations, manifests, and experiment instructions. |

Galactus may also include a research contribution through a new or extended filtering, deduplication, data-selection, or pipeline-optimization method. Its primary framing remains data engineering: small-model training is the evaluation layer, while the central objective is to determine whether the engineered pipeline produces cleaner and more useful pretraining data.

## Galactus Deliverables

The project deliverables combine the working system, its evaluation, and its reproducibility artifacts:

- **Pipeline and datasets:** A working, scalable pipeline covering data acquisition, extraction, normalization, integration, deduplication, filtering, mixing, tokenization, and packing. It will produce documented intermediate datasets and final training shards from multiple raw sources.
- **Data quality and provenance:** Dataset manifests, lineage records, stage-level statistics, rejection reasons, configuration snapshots, and source/version metadata that make each output traceable to its inputs and transformations.
- **Downstream evaluation:** Small language models trained from scratch on controlled dataset variants, with experiments measuring the effect of pipeline choices on corpus quality and model performance.
- **Systems evaluation:** Measurements of runtime, throughput, memory, storage, scaling behavior, and monetary cost, together with ablations and failure analysis for important stages.
- **Reproducible repository:** A public GitHub repository containing source code, environment and dependency definitions, configurations, orchestration commands, tests, sample data or acquisition scripts, and instructions sufficient to reproduce the results.
- **Research report:** A paper describing the problem, related work, methods, architecture, implementation, experiments, results, limitations, references, and each team member's contributions.
- **Presentation:** A seven-minute presentation followed by five minutes of questions and answers. The slides will include the project name, team members, and GitHub link, and will focus on the problem, approach, key findings, technical decisions, challenges, and limitations.
- **Team accountability:** A three-person collaboration with clearly defined responsibilities and a contribution statement in the final report.

## Official Course Requirements

As a custom project, Galactus must meet the following requirements:

- Apply and extend methods learned in the course to solve a real big-data problem.
- Be completed collaboratively by a team of three, with each member's contributions documented in the final report.
- Define the problem and its significance using the Heilmeier Catechism.
- Explain how course methods will be applied to complete the project.
- Establish concrete milestones that support completion and evaluation.
- Submit a proposal PDF with the project title, team name, team members' names, and NetIDs on its title page.
- Produce a research-style final report containing an introduction, problem formulation, related work, methods/architecture/design, results, and references.
- Make the results reproducible through a GitHub repository containing sufficient code, documentation, configurations, and instructions.
- Prepare a seven-minute presentation followed by five minutes of questions and answers, with all team members present and prepared to answer questions.
- Include the project name, team members, and GitHub link in the presentation slides.
- Focus the presentation on the problem, approach, key findings, challenges, and limitations rather than repeating the report.

## Proposed Research Paper Structure

The final report will use the required research-paper format with a systems-oriented organization:

1. **Abstract** — The problem, system, evaluation, and principal findings.
2. **Introduction** — Motivation, research questions, scope, and contributions.
3. **Background and Related Work** — LLM pretraining data pipelines and the relevant data-engineering methods.
4. **Problem Formulation** — Inputs, outputs, constraints, quality objectives, cost objectives, and evaluation questions.
5. **Methods, Architecture, and Design** — End-to-end architecture and the design of every pipeline stage.
6. **Implementation** — Data models, execution framework, storage, orchestration, provenance, and reproducibility mechanisms.
7. **Experimental Methodology** — Datasets, baselines, controlled pipeline variants, model configuration, metrics, and experimental controls.
8. **Data and Model Quality Results** — Corpus measurements and downstream small-model results.
9. **Systems Performance and Cost Analysis** — Runtime, throughput, memory, storage, scaling behavior, and monetary cost.
10. **Ablations and Failure Analysis** — The effect of individual pipeline stages, errors, tradeoffs, and negative results.
11. **Limitations and Future Work** — Constraints of course-project scale and directions for a production system.
12. **Team Contributions and Reproducibility** — Individual responsibilities and exact reproduction instructions.
13. **References** — All work cited in the report and related-work discussion.
