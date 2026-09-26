# Trainer and Training Loop

## Trainer

**Purpose**: Mixin providing Keras-style `compile()`/`fit()`/`evaluate()`/`predict()`/`train_on_batch()`/`test_on_batch()`/`predict_on_batch()` for any `Program` (ported from `keras/src/trainers/trainer.py`).

**Key Methods**:
```
compile(optimizer=None, reward=None, reward_weights=None, metrics=None, 
         run_eagerly=False, steps_per_execution=1)
  → Resolves optimizer/reward/metrics via registries, wraps them in CompileReward/CompileMetrics
  
fit(x=None, y=None, batch_size=1, minibatch_size=4, epochs=1, 
    validation_split=0.1, validation_data=None, shuffle=True, callbacks=None, ...)
  → Builds EpochIterator, auto-builds program on first batch, manages optimizer/epoch/batch lifecycle
  
evaluate(x=None, y=None, batch_size=32, verbose="auto", steps=None, callbacks=None, ...)
evaluate_batch(x, y, callbacks=None) → Single batch
  
predict(x, batch_size=None, verbose="auto", steps=None, callbacks=None, ...)
predict_batch(x, callbacks=None) → Single sample
  
train_on_batch(x, y) → Single-batch training primitive
test_on_batch(x, y) → Single-batch evaluation
predict_on_batch(x) → Single-batch prediction
```

**Distinctive behavior**:
- **Stratified validation**: `stratified_minibatch_indices()` draws validation minibatch that spans distinct target classes (not naive random)
- **Auto-build reuse**: First `fit`/`evaluate` batch's predictions aren't thrown away — reused to avoid duplicate expensive forward pass
- **Metrics cloning**: Metric instances are cloned per-program so nothing leaks across tuner sweeps
- **Warning on empty optimizer**: Returns empty `History` if optimizer isn't compiled rather than burning LM calls uselessly
- **Reward tracking**: Creates a `Mean` metric `_reward_tracker` with `direction="up"` for monitoring

## CompileUtils

**MetricsList** — Groups multiple metrics under one output name; `update_state_batch` batches `BatchMetric`s.

**CompileMetrics** — Wraps user's `metrics=` list/dict into per-output `MetricsList`s.

**CompileReward** — Wraps user's `reward=` with `reward_weights`, tracks per-reward metrics.

## EpochIterator

**Purpose**: Iterates a dataset in batches for `fit`/`evaluate`/`predict`.

**Features**: Wraps generator/array data via `DataAdapter`, supports `steps_per_epoch`, `steps_per_execution`.

## DataAdapters

**DataAdapter** (base), **ArrayDataAdapter**, **GeneratorDataAdapter**; plus `train_validation_split()` and `unpack_x_y()` utilities for data pipeline management.

## Related pages
- [Optimizers](./optimizers.md) — Optimizer lifecycle hooks called by Trainer
- [Rewards](./rewards.md) — Reward compilation
- [Metrics](./metrics.md) — Metric compilation and tracking
- [Callbacks](./callbacks.md) — Callback hooks during training
- [Module Call Lifecycle](../core/module-and-call-lifecycle.md) — How modules execute within training loop
