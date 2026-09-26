# Callbacks (Training-Phase Observability)

Callbacks hook into `fit()`/`evaluate()`/`predict()` lifecycle. Do NOT confuse with `hooks/` which hook into per-call module execution.

## Callback (Base)

**Purpose**: Base class for training lifecycle hooks.

**Hooks**: `on_epoch_begin/end`, `on_train/test/predict_batch_begin/end`, `on_train/test/predict_begin/end`.

**Attributes**: `params`, `program` (set by trainer).

## CallbackList

**Purpose**: Fan-out container dispatching to registered callbacks.

**Distinctive**: `_configure_async_dispatch` — batch-end callbacks run concurrently via `ThreadPoolExecutor` if they declare `async_safe=True` (perf optimization).

## Built-In Callbacks

**History** — Records every epoch's logs; auto-added by `fit()`; returned as `program.fit()`'s return value.

**ProgbarLogger** — Prints progress bar to stdout.

**CSVLogger** — Streams per-epoch logs to CSV file (`filepath`, separator, append mode).

**EarlyStopping** — Stop when monitored quantity plateaus.
- `monitor="val_reward"`, `mode="auto"|"min"|"max"`, `patience=0`, `stop_at=1.0` (absolute threshold)
- `restore_best_variables=False` (deep-copy and restore trainable JSON on early stop)
- Auto-infers `mode` from `Metric.direction` on program

**ProgramCheckpoint** — Save program or variables at frequency.
- `monitor=None, save_best_only=False, mode="auto"`
- `.variables.json` suffix triggers variables-only save

**BackupAndRestore** — Fault-tolerant training resume.
- `backup_dir`, `save_freq="epoch"|int|False`, `double_checkpoint=False`
- Auto-restores on next `fit()` if checkpoint exists, deletes on success

**BudgetStopping** — Hard $/token spend cap across training.
- `max_cost=None, max_tokens=None` (at least one required)
- Warns if provider never reports cost (local models like Ollama)

**Monitor** — MLflow experiment tracking.
- Logs metrics, params, datasets, program (as MLflow pyfunc versioned on `val_reward` improvement)
- Renders per-epoch prompts to MLflow Prompt Registry
- `experiment_name`, `run_name`, `resume=True` for appending to existing runs
- `evaluate()` opens separate "assessment" run logging per-sample rewards as MLflow trace feedback

## Related pages
- [Trainer](./trainer.md) — Where callbacks are passed to `fit()`
- [Hooks](./hooks.md) — Per-call observability (different system)
- [Metrics](./metrics.md) — Metrics monitored by callbacks
