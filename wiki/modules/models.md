# Model Wrapper Modules

Modules wrapping external LM/embedding/decision model APIs.

## LanguageModel

**Purpose**: Central API wrapper (litellm-backed) for chat/completion + structured output across many providers (OpenAI, Anthropic, Gemini, Mistral, local Ollama, etc.).

**Signature**:
```
LanguageModel(
    model=None,               # Model identifier (required); provider aliases auto-rewritten
    api_base=None,            # Custom endpoint (default: provider-specific)
    timeout=600,              # Seconds
    retry=5,                  # Auto-retry attempts
    retry_max_wait=60,        # Max exponential backoff
    fallback=None,            # Chain to fallback LM if this fails
    caching=False,            # On-disk response caching
    cache_dir=None,           # Cache directory (optional)
    name=None, description=None,
    hooks=None,               # Lifecycle hooks
    **default_kwargs          # temperature, top_p, top_k, max_tokens, reasoning_effort, etc.
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Constrained output support**: Handles Azure/Ollama/Mistral-style structured output constraints
- **Constrained-tool-only providers**: Groq, Anthropic (forces tool-only, not full function calling)
- **reasoning_effort semantics**: Three-way: `"none"` (default, say nothing), `"disable"` (turn off native reasoning), any other value (forwarded reasoning level)
- **Provider aliases**: `doubleword/*` → `openai/*`, `mirai/*` → `hosted_vllm/*`
- **Caching**: On-disk response cache (including cached streaming replay)
- **Observability**: `current_call_usage()` exposes per-task token/cost/latency via `ContextVar`
- **Per-phase tracking**: Tracks calls/tokens/cost separately across `inference`/`reward`/`optimizer` phases via `op_scope`

## EmbeddingModel

**Purpose**: API wrapper for text → dense embeddings (litellm-backed, many providers).

**Signature**:
```
EmbeddingModel(
    model=None,               # Model identifier (required)
    api_base=None,            # Custom endpoint
    retry=5,                  # Auto-retry attempts
    retry_max_wait=60,        # Max exponential backoff
    fallback=None,            # Fallback EmbeddingModel
    caching=True,             # On-disk caching
    cache_dir=None,           # Cache directory (optional)
    name=None, description=None,
    hooks=None,               # Lifecycle hooks
    **default_kwargs          # Provider-specific options
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Provider list**: azure, bedrock, cohere, gemini, huggingface, mistral, ollama, openai, together_ai, vertex_ai, voyage_ai — verified/documented providers
- **Ollama auto-config**: `api_base` defaults to `http://localhost:11434` for `ollama/*` models
- **Per-phase tracking**: Tracks calls/tokens/cost separately per phase (`inference`/`reward`/`optimizer`)

## DecisionModel

**Purpose**: API wrapper around "System One" decision models (currently TypeSafe's `jev-*`) that answer typed yes/no, choice, or score questions with calibrated probabilities.

**Signature**:
```
DecisionModel(
    model=None,               # Model identifier (required, e.g. "typesafe/jev-latest")
    api_base=None,            # Custom endpoint
    timeout=30.0,             # Per-request timeout
    retry=5,                  # Auto-retry
    retry_max_wait=60,        # Max exponential backoff
    fallback=None,            # Fallback DecisionModel
    cache_dir=None,           # Cache directory for responses
    cost_per_token=None,      # Manual cost override
    name=None, description=None,
    hooks=None                # Lifecycle hooks
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Schema validation**: Every output field must be decision-answerable (`bool` → noul, enum → choice, scale → score) or raises `UnsupportedSchemaError`
- **Where it's used**: As `decision_model=` argument to `Generator`, `Decision`, `MultiDecision`, `Branch`, `SelfCritique`, `RubricsAsJudge`; never as `language_model`
- **Module-level helpers**: `score_schema()`, `noul_schema()`, `choice_schema()`, `questions_from_schema()` — schema inference machinery
- **Fallback chain**: Supports `fallback` chaining like `LanguageModel`
- **Caching**: On-disk response cache

## Related pages
- [Core Modules](./core.md) — Generator, Decision, etc. which wrap these models
- [Agents](./agents.md) — Agents which use these models
- [Language Model Parameters](../training/trainer.md) — How LM parameters flow through training
