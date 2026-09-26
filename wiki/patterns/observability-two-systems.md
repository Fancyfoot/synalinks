# Two Parallel Systems of Observability

Synalinks has two distinct observability systems that serve different purposes and must NOT be confused:

## System 1: Callbacks (Training-Phase Observability)

**Scope**: The `fit()`/`evaluate()`/`predict()` training loop.

**Lifecycle**: Epoch-level, batch-level, and whole-run lifecycle hooks.

**Hooks**: `on_train_begin/end`, `on_epoch_begin/end`, `on_train_batch_begin/end`, `on_train_batch_end` (async dispatch optional).

**What they see**: Aggregate metrics across batches/epochs, program-level state, loss/reward trends.

**Built-in callbacks**: History, ProgbarLogger, CSVLogger, EarlyStopping, ProgramCheckpoint, BackupAndRestore, BudgetStopping, callbacks.Monitor (MLflow experiment tracking).

**Enable explicitly**: Pass `callbacks=[...]` to `fit()` or `callbacks.Monitor()` for MLflow.

## System 2: Hooks (Per-Call Observability)

**Scope**: Individual module `__call__()` invocations (any call, anywhere in the program, not just during training).

**Lifecycle**: `on_call_begin/end` — fires for every module call.

**What they see**: Per-call inputs/outputs, call hierarchy, LM messages, exceptions, fine-grained timing.

**Built-in hooks**: Logger (stdlib logging), Recorder (JSONL tracing), hooks.Monitor (MLflow distributed tracing).

**Enable transparently**: `synalinks.enable_observability()` / `synalinks.record_traces()` auto-wires hooks onto every module without code changes.

## When to Use Each

| Need | System | Why |
|------|--------|-----|
| Monitor training progress (loss, reward trend) | Callbacks | Aggregate epoch-level signals |
| Detect which batch failed | Callbacks | Batch-level lifecycle |
| Trace per-call LM messages for debugging | Hooks | Call-level detail |
| Collect fine-tuning data from LM calls | Hooks | JSONL recording of messages |
| Track inference cost per query | Hooks | Per-call cost tracking |
| Budget-cap the total training spend | Callbacks | BudgetStopping monitors cumulative cost |
| Trace nested agent sub-calls | Hooks | Hierarchy visible in on_call_begin call_id's |
| View token/latency per training phase | Both | Callbacks.Monitor shows aggregate; hooks show per-call |

## Integration Example

```python
# Training with both systems
program.compile(optimizer=..., reward=..., metrics=[...])

# Callbacks hook the training loop
program.fit(
    x=dataset,
    epochs=10,
    callbacks=[
        EarlyStopping(monitor="val_reward"),
        callbacks.Monitor(experiment_name="my_exp"),  # MLflow training metrics
    ]
)

# Hooks hook into every module call, including those during training
synalinks.enable_observability()          # Enables hooks.Monitor (MLflow tracing)
synalinks.record_traces(base_dir="...")  # Enables hooks.Recorder (JSONL collection)

# Now fit() runs with both systems active:
# - callbacks.Monitor logs epoch metrics to MLflow runs
# - hooks.Monitor creates MLflow traces for each module call
# - hooks.Recorder writes LM messages to JSONL
```

## Op-Scope Phases

Both systems use `op_scope` to attribute costs/time to the correct training phase:
- `"inference"` — `predict_on_batch`
- `"reward"` — Computing reward
- `"optimizer"` — Optimizer step

Operational metrics (LM tokens, latency) are tracked separately per phase.

## Related pages
- [Callbacks](../training/callbacks.md) — Training-phase observability
- [Hooks](../training/hooks.md) — Per-call observability
- [Module Call Lifecycle](../core/module-and-call-lifecycle.md) — Where hooks are invoked
- [Scopes](../core/scopes.md) — op_scope phase tracking
