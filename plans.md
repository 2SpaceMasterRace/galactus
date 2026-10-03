It would be a managed pretraining experimentation platform where the dataset—not merely the model—is a first-class, versioned object.

  source = galactus.source("common-crawl", snapshot="2026-01")

  dataset = (
      source.extract()
            .normalize()
            .deduplicate(method="minhash")
            .filter(["gopher", "language", "quality-model"])
            .decontaminate(evaluations)
            .mix(wikipedia=0.15, code=0.10)
            .tokenize(vocab_size=32_000)
            .pack(sequence_length=2_048)
  )

  run = galactus.pretrain(
      model="decoder-150m",
      dataset=dataset,
      token_budget="1B",
  )

  run.compare(baseline="minimally-processed")

  It would inherit:

  - From Tinker: a simple researcher-facing API while the platform manages compute allocation, failures, checkpoints and distributed execution.
  - From NeMo Curator: scalable extraction, classification, filtering, deduplication and CPU/GPU execution.
  - From Dolma: composable curation operators, dataset mixing, statistics, CLI workflows and open reproducibility.
  - Its own contribution: connecting a particular dataset recipe to controlled from-scratch training and evaluation.

  Underneath, it would contain:

  Research API / CLI
          |
  Experiment control plane
          |
  Data DAG -------- Training DAG -------- Evaluation DAG
          |              |                     |
  Artifact store    Model checkpoints      Comparison reports
          \____________ Provenance graph ____________/

  The most important capabilities would be:

  - immutable, addressable dataset versions;
  - token-level lineage back to source documents;
  - cached and resumable processing stages;
  - local, cloud or cluster execution without changing the recipe;
  - data and model experiment matrices;
  - matched-compute comparisons between dataset variants;
  - automatic runtime, throughput, storage, GPU and monetary-cost reports;
  - extensible Python operators for new research ideas;
  - shareable experiment recipes that another researcher can reproduce.

  The user experience might collapse into four commands:

  galactus build dataset.yaml
  galactus train experiment.yaml
  galactus evaluate RUN_ID
  galactus compare RUN_ID_A RUN_ID_B

Useful features to borrow:

  - Composable pipeline stages: readers, extractors, taggers, filters, deduplicators, mixers,
    tokenizers and writers.

  - Tag before filtering: preserve quality signals so researchers can test new thresholds
    without reprocessing raw data.

  - Pluggable execution backends: run the same pipeline locally or through Ray/Slurm.
  - Exact and MinHash/LSH deduplication: with optional CPU/GPU implementations.
  - Dataset mixing: reproducibly combine sources using explicit weights.
  - Resumable execution: cache completed shards and retry only failed tasks.
  - Stage statistics: track retention, rejection reasons, throughput and resource consumption.
  - Extensible filters/classifiers: allow custom Python operators alongside built-in rules.
  - Configuration-driven CLI: reproducible tag, dedupe, filter, mix, tokenize and pack
    operations.

  Galactus would then add its distinguishing layer: train controlled models from scratch and
  compare how those pipeline choices affect model quality and cost.

