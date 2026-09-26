# Program, Functional, and Sequential

## Program

**Purpose**: Top-level trainable container grouping modules into a deployable unit (Keras `Model` equivalent).

**Constructor** (note: class dispatch via metaclass):
```
Program(...) or
Program(inputs=..., outputs=...) → transparently becomes Functional instance
```

**Key Methods**:
- `get_module(name/index=...)` — Access sub-modules
- `summary()` — Print architecture
- `save()` / `load()` — JSON file with version, config, full variable state
- `to_json()` / `program_from_json()` — JSON serialization
- `build_from_config()` — Deserialize from config dict

**Inherits**: `Trainer` (mixin for `.compile()`, `.fit()`, `.evaluate()`, `.predict()`) and `Module`.

**Distinctive**: Uses `inject_functional_program_class()` to dynamically rewrite `__bases__` at runtime when Functional-style construction detected.

## Functional

**Purpose**: DAG-based program defined by tracing calls from `Input()` to output `SymbolicDataModel`s.

**Constructor**:
```
Functional(inputs=..., outputs=...)  # inputs/outputs are SymbolicDataModel(s)
```

**Usage**:
```python
inp = Input(schema=InputSchema)
x = Generator(...)(inp)
out = Decision(...)(x)
program = Program(inputs=inp, outputs=out)  # Becomes Functional internally
```

**Key Methods**:
- `call()` runs `_run_through_graph()` (evaluates DAG)
- `compute_output_spec()` — Symbolic execution (returns schema)
- `input_schema` / `output_schema` properties
- `get_config()` / `functional_from_config()` — Full graph serialization (module list + node connectivity)

**Distinctive**:
- **State stays mutable** after `build()` (overrides normal `Module` behavior)
- **Clone-on-import**: If inputs aren't `Input()` placeholders, clones the subgraph (allows extraction from intermediate points)
- **Full serialization**: Graph structure (topological module list + inbound-node args) reconstructed by iterative "unprocessing" (Keras algorithm)

## Sequential

**Purpose**: Linear stack of single-input/single-output modules (Keras `Sequential` equivalent).

**Constructor**:
```
Sequential(description="...", modules=[...])  # description required (unlike Keras)
```

**Key Methods**:
- `add(module, rebuild=True)` — Append module
- `pop(rebuild=True)` — Remove last module
- Lazily builds internal `Functional` once an `Input` is known
- Falls back to iterative calling if no `Functional` possible (nested-structure inputs)

**Distinctive**: `modules` list is the single source of truth; `input_schema`, `output_schema`, `inputs`, `outputs` all delegate to lazy `_functional`.

## Related pages
- [Module Call Lifecycle](./module-and-call-lifecycle.md) — How modules execute
- [Trainer](../training/trainer.md) — `.compile()`, `.fit()` on programs
- [Data Model Hierarchy](./data-model-hierarchy.md) — Input/output schemas
