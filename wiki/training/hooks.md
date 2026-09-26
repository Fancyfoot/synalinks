# Hooks (Per-Call Observability)

Hooks intercept **module `__call__()`** events, NOT the training loop. Do NOT confuse with `callbacks/` which hook training.

## Hook (Base)

**Purpose**: Base class for per-call lifecycle hooks.

**Events**: `on_call_begin(call_id, parent_call_id=None, inputs=None, kwargs=None)`, `on_call_end(call_id, parent_call_id=None, outputs=None, exception=None)`.

**Attributes**: `params`, `module`.

## HookList

**Purpose**: Fan-out container for hooks.

**Distinctive**: `_add_default_hooks` auto-injects `Logger()`, `Monitor()` (if observability enabled), `Recorder()` (if tracing enabled) — so `synalinks.enable_observability()`/`record_traces()` transparently wire hooks without code changes.

## Built-In Hooks

**Logger** — Default hook; logs module calls to stdlib `logging` module (debug for symbolic, info for concrete, error on exception).

**Recorder** — Writes every concrete `LanguageModel` call (messages + completion) as one JSONL line (compatible with OpenAI chat format, NVIDIA NeMo Customizer, etc.).
- `base_dir=None` (defaults to `synalinks_home()`)
- Organizes: `<base_dir>/<program_name>/<module_name>/<module>_<timestamp>.jsonl`
- Each record includes `synalinks_version`, `call_id`, `token usage`, `cost`, `inputs_hash`/`config_hash` (SHA-256 for dedup)
- Foldsanthropicthinking `thinking_blocks` into `reasoning_content` for compatibility

**Monitor** — MLflow distributed tracing hook.
- Creates one Span per module call
- `trace_context` context manager attaches `user_id`/`session_id`/`metadata`/`tags` (contextvars-based, safe under async)
- Maintains `_GLOBAL_SPANS_REGISTRY` and `_ROOT_TRACES` deque so `callbacks.Monitor` can attach reward assessments as feedback traces after each batch
- Kept module-level `_LOGGER` (no instance attribute) to avoid breaking `deepcopy` on tracked dicts

**trace_context** — Context manager for trace metadata.
```python
with trace_context(user_id="...", session_id="...", metadata={...}):
    ... # Spans created here carry the metadata
```

## Distinctive Patterns

- **Two parallel systems**: `callbacks/` (training loop phase), `hooks/` (per-call tracing) — used together, not mutually exclusive
- **Op-scope phases**: `op_scope` context sets `"inference"` / `"reward"` / `"optimizer"` phase, used by both hooks and metrics to attribute calls correctly
- **Automatic wiring**: `synalinks.enable_observability()` and `record_traces()` auto-wire hooks without explicit construction

## Related pages
- [Callbacks](./callbacks.md) — Training-phase observability (different system)
- [Module Call Lifecycle](../core/module-and-call-lifecycle.md) — How hooks are invoked
- [Trainer](./trainer.md) — Training loop that coordinates op_scope phases
- [Two Systems of Observability](../patterns/observability-two-systems.md) — Detailed explanation of both systems
