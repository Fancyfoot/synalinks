# Rewards (Training Signals)

Rewards score predictions against targets; they guide optimizer candidate selection and are tracked as metrics.

## Reward (Base)

**Purpose**: Base class for all rewards.

**Signature**:
```
Reward(name=None, reduction="mean"|"sum"|"min"|"max"|"none",
       in_mask=None, out_mask=None, 
       in_mask_pattern=None, out_mask_pattern=None)
```

**Key Methods**: `call(y_true, y_pred)` (abstract), `reduce_values()` (applies reduction).

**Utilities**: `apply_masks()`, `reduce_rewards()` (per-sample → scalar for trackers/candidate scoring).

## Simple Rewards

**ExactMatch** (`exact_match(y_true, y_pred)`) — 1.0 if `get_json()` equal else 0.0.

**CosineSimilarity** (`cosine_similarity(...)`) — Embeds both sides, returns `(cosine_sim + 1) / 2` ∈ [0,1].

## Composite Rewards

**BatchReward** — Base for rewards needing the whole batch at once (e.g., cross-sample ranking). `compute_batch()` returns `list[float]`.

**ComposableReward** — Combine multiple child rewards (list or dict) weighted, with per-child reduction.

## LM-Based Judge Rewards

All judge rewards inherit from `ProgramAsJudge` (wraps a `Program` as reward), normalize output to 0..1 regardless of scale.

**LMAsJudge** — Single LM read (no tools), scores via `score_type` enum (FineScore, Rating20, etc.), normalizes.

**RLMAsJudge** — Recursive LM judge (writes/executes Python in sandbox to verify prediction), more flexible scoring.

**AgentAsJudge** — Judges via `FunctionCallingAgent` with arbitrary tools (run code, query DB, fetch web).

**DeepAgentAsJudge** — Judges via `DeepAgent` in sandboxed `workdir` (can apply patches, run tests, diff). Arguments: `workdir`, `sandbox`, `reset_sandbox=True`, `max_iterations=10`, `timeout=30`.

## Rubric Rewards

**RubricsAsJudge** — Grade against named criteria (`Rubric` = name + description + weight), combine weighted mean.

**28 Rubric Presets** — Concrete subclasses hard-coding one preset each: `AnswerRelevancy`, `Faithfulness`, `Hallucination`, `Summarization`, `ArgumentCorrectness`, ..., `TurnRelevancy` (built from ~28 DeepEval-style rubrics in `RUBRICS` dict).

## RewardFunctionWrapper

Wraps a bare `async def fn(y_true, y_pred, **kwargs)` function into a `Reward`; `compile(reward=fn)` auto-wraps.

## Distinctive Patterns

- **Normalization**: All judges normalize to [0,1] so training loop can combine rewards across scales
- **With DecisionModel**: `RubricsAsJudge` can use a `decision_model` instead of LM (grades via ordered levels, no `score_type`); `score_type` must be None in this case
- **Masking**: All rewards support `in_mask`/`out_mask` to select fields for evaluation

## Related pages
- [Trainer](./trainer.md) — `compile(reward=...)` and how rewards drive optimization
- [Metrics](./metrics.md) — How rewards become metrics for tracking
- [Test-Time Compute](../modules/ttc.md) — SelfCritique which is the base reward primitive for LM judges
