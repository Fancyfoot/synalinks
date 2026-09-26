# Optimizers (In-Context RL)

Optimizers refine trainable `Variable` state (prompts, examples, code) by testing candidate variations and selecting based on reward.

## Optimizer (Base)

**Purpose**: Base class implementing shared candidate-population machinery (propose → validate → select).

**Key Methods**: `optimize(step, trainable_variables, x, y, val_x, val_y)` — per-step protocol: predict training batch, compute rewards, propose child candidates, score on validation minibatch, select.

**Lifecycle**: Hooks `on_train_begin/on_epoch_begin/on_batch_begin/on_batch_end/on_epoch_end/on_train_end` manage `candidates`, `best_candidates`, `seed_candidates` per variable.

**Distinctive**: Candidates/predictions live **inside trainable `Variable`'s JSON** (via `Trainable` keys), not separate optimizer state — fully serializable with the program.

## GreedyOptimizer (Base)

**Purpose**: Abstract base for optimizers that select few-shot examples from best past predictions (no LLM mutation).

**Key Constructor**: `(nb_min_examples=1, nb_max_examples=3, sampling="softmax"|"random"|"best", sampling_temperature=0.3, population_size=10, ...)`

## RandomFewShot

**Purpose**: Few-shot learning optimizer — samples best past predictions into the variable's `examples`. Implements "Language Models are Few-Shot Learners" paper.

**Signature**:
```
RandomFewShot(
    nb_min_examples=1, nb_max_examples=3,
    sampling="softmax"|"random"|"best",
    sampling_temperature=0.3,
    population_size=10,
    # ... more optional params
)
```

**Distinctive**: Trivial `build()`; `propose_new_candidates` only picks a variable and assigns examples — no LM call.

## EvolutionaryOptimizer (Base)

**Purpose**: Abstract base for LLM-driven genetic optimizers (mutation + crossover of JSON variables).

**Key Abstract Methods**: `mutate_candidate()`, `merge_candidate()` (subclasses implement).

**Distinctive**: `select_evolving_strategy()` picks crossover with constant probability `merging_rate`; `_operator_kwargs` inspects method signature for backward compatibility.

## OMEGA

**Purpose**: "OptiMizEr as Genetic Algorithm" — Dominated Novelty Search (DNS): quality-diversity GA that removes candidates only if *both* inferior in reward *and* similar in embedding-space.

**Signature** (subset):
```
OMEGA(
    instructions=None,        # LM system instructions (trainable)
    language_model=None,      # LM for mutations/crossover; required
    embedding_model=None,     # For diversity calculation
    mutation_temperature=0.3,
    crossover_temperature=0.3,
    merging_rate=0.05,        # Probability of crossover vs mutation
    algorithm="dns"|"ga",     # DNS is default; GA for ablation
    use_chain_of_thought=True,  # Use ChainOfThought or plain Generator
    nb_best_predictions=1, nb_worst_predictions=3,  # Context examples
    nb_hard_examples=1,       # Recurring failed examples
    k_nearest_fitter=5,       # KNN for dedup
    reasoning_effort=None,
    # ... more optional params
)
```

**Distinctive behavior**:
- **Dominated Novelty Search**: Diversity measured by embedding cosine distance; removed only if dominated *and* similar
- **Hard example memory**: Tracks cross-batch poorly-scored inputs (`observe_training_batch()`)
- **Embedding strategy**: Embeds each trainable field separately, averages/renormalizes unit vectors
- **Input echo removal**: Strips fields that echo the input (avoids redundant LM context)
- **Builds MutationInputs/CrossoverInputs** with best/worst predictions for LM context
- **Research origins**: Dominated Novelty Search paper, DSPy's GEPA, DeepMind's AlphaEvolve

## Related pages
- [Trainer](./trainer.md) — Where optimizers are compiled and used
- [Rewards](./rewards.md) — How optimizers score candidates via rewards
- [Callbacks](./callbacks.md) — Monitoring optimizer progress
