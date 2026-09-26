# Scopes

Scopes manage execution context, state mutation, graph tracing, and phase tracking.

## name_scope

**Purpose**: Build `/`-joined variable paths for hierarchical state organization.

**Usage**:
```python
with name_scope("layer1"):
    var1 = Variable(...)  # path = "/layer1/var1"
    with name_scope("sublayer"):
        var2 = Variable(...)  # path = "/layer1/sublayer/var2"
```

**Features**:
- `deduplicate=True` — Skip re-entering identical scope from same caller (avoids redundant nesting)
- `override_parent=True` — Absolute path override (ignores parent stack)

**Implementation**: Thread-local `name_scope_stack` (via `global_state`).

## StatelessScope

**Purpose**: Prevent in-place `Variable` mutation for a scope's duration (functional evaluation semantics).

**Usage**:
```python
with StatelessScope(state_mapping=[(var1, value1), (var2, value2)]):
    y_pred = program(x)  # reads overridden values, mutations queued
    # On exit: initializes uninitialized variables, pending mutations are recorded
```

**Key Methods**:
- `add_update((variable, value))` — Queue a pending write
- `get_current_value(variable)` — Read override value (if present)

**Distinctive**: `collect_rewards` dict collects reward signals emitted mid-graph.

## SymbolicScope

**Purpose**: Marker indicating "we are tracing the Functional graph" (no mutation prevention).

**Usage**: Used internally during `Program(inputs=..., outputs=...)` construction.

**Checked via**: `in_symbolic_scope()` / `get_symbolic_scope()`.

## op_scope (Operation Scope)

**Purpose**: Track execution phase for attributing LM calls and wall-clock time.

**Values**: `"inference"` (prediction), `"reward"` (computing reward), `"optimizer"` (optimizer step).

**Usage**:
```python
with op_scope("inference", clock=phase_clock):
    y_pred = program.predict_on_batch(x)  # All LM calls attributed to "inference"
```

**PhaseClock**: Ensures nested phases (e.g., reward-inside-optimizer) don't double-count time. Only the innermost active phase is credited.

**Implementation**: `contextvars.ContextVar` (safe across `asyncio.gather` and greenlets).

## trajectory_scope

**Sub-scope of op_scope**: Marks the start of an agent's whole multi-turn trajectory.

**Distinctive**: Set-once semantics — nested sub-agents don't reset the outer timer. Used for "time to first token" measurement across a trajectory.

## global_state

**Purpose**: Thread-local key/value store backing all scopes.

**Public API**: `synalinks.clear_session(gc.collect=False)` resets all scope state (used between tests).

## Related pages
- [Data Model Hierarchy](./data-model-hierarchy.md) — StatelessScope's role in variable initialization
- [Module Call Lifecycle](./module-and-call-lifecycle.md) — How CallContext and op_scope coordinate
- [Trainer](../training/trainer.md) — op_scope used by Trainer to attribute costs
- [Hooks](../training/hooks.md) — Hooks receive phase from op_scope via CallContext
