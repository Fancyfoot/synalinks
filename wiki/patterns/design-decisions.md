# Design Decisions and Architecture Rationale

Key architectural choices in Synalinks, their rationale, and implications.

## Pydantic-Only Backend

**Decision**: Only Pydantic is implemented as a backend, despite Keras-style multi-backend scaffolding (`backend/config.py`, `_AVAILABLE_BACKEND`).

**Rationale**: Pydantic provides battle-tested JSON schema generation + validation, seamless serialization to JSON (critical for storing prompts/examples in optimizers), and direct integration with LiteLLM's schema validation features.

**Implications**:
- No `backend.tensorflow` or `backend.jax` variants
- All variables are JSON, not tensors
- Full serialization/versioning of state is natural (not artificial)
- `Variable` state lives inside the program JSON, not in separate checkpoint files

## JSON-Native Trainable State

**Decision**: Optimizer candidates, mutations, and optimizable state (prompts, examples, code) are stored as JSON within `Variable`s, not as separate optimizer state.

**Rationale**: Every variation is serializable-by-default with the program. Checkpointing, resuming training, and versioning prompts is straightforward. Optimizers are stateless regarding the program.

**Implications**:
- `Variable` extends Python dict-like semantics with `.get()`, `.items()`, etc.
- Operators (`+`, `&`, `|`, etc.) work directly on JSON objects
- Candidates live in the variable's `Trainable` metadata (reserved keys `reward`, `reward_count`, `candidates`, `best_candidates`)

## Composition Over Inheritance for LM Modules

**Decision**: LM-driven modules (Generator, Decision, Branch, ChainOfThought, etc.) wrap an internal `Generator` (or derived module) via composition, rather than inheriting from it.

**Rationale**: Each module has different input/output contracts and schema transformation logic. Composition is more flexible than a deep inheritance tree and keeps concerns separated.

**Implications**:
- No `Generator` subclass hierarchy (except the rare `ChainOfThought` → `Module` direct inheritance for step-by-step reasoning)
- Each module's schema transformation is explicit in its constructor/` compute_output_spec()`
- Easier to understand what each module does (no "what did the parent class do?" questions)

## Async-First Call Interface

**Decision**: All `call()` methods are `async def`, even for synchronous modules.

**Rationale**: Allows agents to call tools and sub-agents concurrently without forcing all code to think about async. The async boundary is at the module level, not scattered throughout.

**Implications**:
- `await module(inputs)` is the normal calling convention
- `asyncio.gather` is used pervasively for parallel tool execution (agents, branches, reranking)
- Single-threaded async runtime naturally serializes most operations while parallelizing I/O

## Op-Scope Phases for Cost Attribution

**Decision**: Training is divided into three operational phases (`inference`, `reward`, `optimizer`), tracked via `op_scope` context variable.

**Rationale**: LM spend during training comes from different sources (forward pass prediction, reward computation, optimizer mutation/crossover). Attributing them correctly enables fine-grained cost monitoring and budgeting.

**Implications**:
- Every LM call records which phase triggered it
- Metrics and callbacks see per-phase token/cost breakdowns
- `BudgetStopping` can cap spend per phase or globally

## Keras Lineage

**Decision**: Architecture closely mirrors Keras (Module ≈ Layer, Program ≈ Model, Trainer mixin, Sequential, Functional API, callbacks, metrics).

**Rationale**: Engineers familiar with Keras feel at home. Keras's proven abstractions translate cleanly to LMs. Porting utilities (like `tree/` for nested structures) was straightforward.

**Implications**:
- `compile()`, `fit()`, `evaluate()`, `predict()` method names and signatures
- `get_config()` / `from_config()` serialization pattern
- `layers` analogy to `modules`, `Models` to `Programs`

## Deterministic Name Scoping

**Decision**: Variables get automatic path-based names (`/layer1/sublayer/var_name`) from name-scope context.

**Rationale**: Reproducible, collision-free naming without user boilerplate. Essential for multi-program state management (tuner sweeps) and serialization.

**Implications**:
- `name_scope()` must be properly nested during module construction
- Serialized state preserves structure via paths (useful for e.g., partial loading, merging)

## Related pages
- [Module and Call Lifecycle](../core/module-and-call-lifecycle.md) — How call() is invoked
- [Data Model Hierarchy](../core/data-model-hierarchy.md) — JSON-native state
- [Optimizers](../training/optimizers.md) — How optimizers leverage JSON trainable state
- [Programs](../core/programs.md) — Keras-inspired program API
