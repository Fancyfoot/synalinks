# Module Base Class and Call Lifecycle

## Module (Base)

**Purpose**: Base class every module inherits from; defines the calling convention and state management for composable units.

**Constructor**:
```
Module(name=None, description=None, trainable=True, hooks=None, **kwargs)
```

**Key Attributes**:
- `trainable` — Whether optimizers can rewrite `Variables` in this module
- `built` — Whether `build()` has completed
- `variables`, `modules`, `metrics` — Tracked sub-components (via `Tracker`)

**Key Methods**:
- `build(input_spec, ...)` — Create state depending on input shape; wrapped in `build_wrapper` (auto name-scopes, locks state)
- `call(inputs, training=False, ...)` (async) — The forward pass; abstract, implemented by subclasses
- `__call__(inputs, training=None, ...)` (async) — Public calling interface (see lifecycle below)
- `compute_output_spec(input_spec, ...)` — Symbolic execution (for tracing)
- `get_config()` / `build_from_config()` — Serialization

## Call Lifecycle

When `module(inputs)` is invoked:

1. **Data conversion** — Convert DataModel args to `JsonDataModel` if needed
2. **Input validation** — Check against input schema
3. **Auto-build** — Call `build()` if not already built
4. **CallContext setup** — Set `ContextVar` with `training`, phase, call_id
5. **Hook dispatch** — `hooks.on_call_begin()` (before execution)
6. **Actual execution** — Call `call()` with inputs
7. **Hook dispatch** — `hooks.on_call_end()` (after execution, or with exception)
8. **Output conversion** — Convert output to declared `DataModel` type (raises if mismatch)

## CallContext

**Purpose**: Task-local context carrying per-call metadata (accessible via `ContextVar`).

**Contents**: `training` flag, `phase` ("inference"/"reward"/"optimizer"), `call_id` (UUID), `parent_call_id`, `module_name`, timestamps.

Used to:
- Thread the `training` flag through nested calls
- Attribute LM calls / wall-clock time to the correct training phase
- Trace nested module call hierarchies

## Operation / Node / Function (Graph Tracing)

**Operation** — Base class every module inherits from (via `Module`). `__call__` branches:
- **Symbolic path**: If any input is `SymbolicDataModel`, delegate to `compute_output_spec()` and record a `Node` in the ops graph
- **Eager path**: Otherwise, call the real `call()`

**Node** — Records one `operation.__call__()` event (the "edge bookkeeping" of a DAG): inputs, operation, outputs. Used to reconstruct a full computation graph.

**Function** — Captures a reusable subgraph between given input/output `SymbolicDataModel`s. Stateless analogue of `Functional` (which itself uses `Function` internally).

**SynalinksHistory** — Metadata on `SymbolicDataModel` storing backward references to `Node`s that created it; enables full graph reconstruction via `SymbolicDataModel._synalinks_history`.

## Distinctive Patterns

- **State is locked after build()** — `_lock_state()` prevents new `Variable`s from being created post-build (except in `Functional`, which overrides to no-op)
- **Symbolic building** — Calling a module with symbolic inputs (during graph tracing) doesn't execute `call()`; it just records schema transformation and returns a symbolic output
- **Auto tracking via __setattr__** — When a `Variable`/`Module`/`Metric` is assigned as an attribute, the `Tracker` auto-registers it (no need for manual tracking)
- **Trainable propagation** — The `training` context var is set by the outer trainer and flows through all nested calls automatically

## Related pages
- [Programs](./programs.md) — How modules are composed into trainable programs
- [Scopes](./scopes.md) — How `name_scope` and `op_scope` coordinate state and phase tracking
- [Hooks](../training/hooks.md) — Per-call observability
