# Metrics (Monitoring Signals)

Metrics track training progress; they monitor rewards, predictions, and operational costs.

## Metric (Base)

**Purpose**: Base class for all metrics.

**Key Attributes**: `direction` ("up"/"down"/None) tells consumers which way is better.

**Methods**: `add_variable()` (create backend state), `reset_state()`, `update_state()`, `result()`.

## Simple Metrics

**Sum** — Running sum (one variable `total`).

**Mean** — Running mean (variables `total`/`count`).

**MeanMetricWrapper** — Wraps stateless function (often a reward) as a `Mean` metric; auto-sets `direction="up"` for known rewards.

## Accuracy Family

**Accuracy** — Per-field token Jaccard-index accuracy, `average=None|"micro"|"macro"|"weighted"`.

**BinaryAccuracy**, **CategoricalAccuracy** — Per-class variants.

## F-Score Family

**FBetaScore** — Weighted harmonic mean of precision/recall (SQuAD-style token multisets), `beta=1.0`.

**F1Score**, **Precision**, **Recall** — Token-level variants.

**BinaryF1Score**, **CategoricalF1Score** — Per-class variants.

## Regression Metrics

**CosineSimilarity** — Wraps `rewards.cosine_similarity` as a metric.

## Agent Sampling Metrics (BatchMetric)

All require `repeat=K` in dataset so each batch is one problem's K rollouts.

**PassAtK** — `1 - C(n-c,k)/C(n,k)` (optimistic, "solved ≥1 of k").

**PassHatK** — `C(c,k)/C(n,k)` (consistency, "solved ALL k").

**GapK** — `PassAtK - PassHatK` (reliability gap).

References HumanEval (`pass@k`) and tau-bench.

## Operational Metrics (Per-Phase)

All track calls/tokens/cost separately across `inference`/`reward`/`optimizer` phases via `op_scope`.

**LM Metrics** (~28): `InputTokens`, `OutputTokens`, `TotalTokens`, `AvgInputTokensPerCall`, ..., `Cost`, `ErrorRate`, `CacheHitRate`, etc.

**EM Metrics** (~15): `EmbeddingTokens`, `EmbeddingVectors`, `EmbeddingCost`, etc.

**Program Metrics**: `ProgramCalls`, `ProgramElapsedTime`, `ProgramCost` (wall-clock at program boundary, not summed nested).

**Rewarder/Optimizer Variants**: Parallel metric families with `Reward*`/`Optimizer*` prefixes isolate spend by training phase.

## BatchMetric

Base for metrics needing the whole batch at once (receives all samples together, returns list[float]).

## Related pages
- [Trainer](./trainer.md) — `compile(metrics=[...])` and metric compilation
- [Callbacks](./callbacks.md) — Monitor metrics during training via EarlyStopping, etc.
- [Rewards](./rewards.md) — Rewards wrapped as metrics
