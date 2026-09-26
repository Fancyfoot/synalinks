# Test-Time Compute (TTC) Modules

Modules that add reasoning/critique at inference time (test-time compute).

## ChainOfThought

**Purpose**: Prepend a `thinking` field to the target schema so the LM can reason step-by-step before answering.

**Signature**:
```
ChainOfThought(
    schema=None, data_model=None,
    language_model=None,      # LM (required unless decision_model given)
    prompt_template=None,     # Custom prompt
    examples=None, instructions=None,  # Trainable
    temperature=None, max_tokens=None, ...  # LM parameters
    use_inputs_schema=False, use_outputs_schema=False,
    return_inputs=False,
    streaming=False,
    tools=None, tool_schemas=None,
    reasoning_effort=None,
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module` (thin wrapper delegating to internal `Generator` over `Thinking + data_model`).

**Distinctive behavior**:
- **Defaults reasoning_effort to "low"** — uses the LM's native extended-thinking when supported, otherwise it's a normal generated field
- **Implements the Chain-of-Thought paper** (arXiv:2201.11903)

## SelfCritique

**Purpose**: Critique given inputs and emit a normalized 0.0–1.0 `reward` score (the reward primitive for LM-based judges).

**Signature**:
```
SelfCritique(
    language_model=None,      # LM (required unless decision_model given)
    prompt_template=None,     # Custom prompt
    examples=None, instructions=None,  # Trainable
    temperature=None, max_tokens=None, ...  # LM parameters
    use_inputs_schema=False, use_outputs_schema=False,
    return_reward=True,       # Include reward field in output
    score_type=None,          # Grading scale enum (FineScore, Rating20, etc.)
    return_inputs=True,       # Include input in output
    decision_model=None,      # Use DecisionModel instead
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Score type**: Selects the LM's native grading scale (`FineScore` default 21 levels, `Rating20` etc.) but output `reward` is **always normalized to 0..1** regardless — this is the reward consumed by `LMAsJudge` and other judges
- **With DecisionModel**: No `critique` text field — only yes/no-style grading (output has no `thinking`)

## Related pages
- [Core Modules](./core.md) — Generator which ChainOfThought wraps
- [Rewards](../training/rewards.md) — SelfCritique is used by LM-based reward judges
- [Trainer](../training/trainer.md) — Where rewards are compiled and used during fit()
