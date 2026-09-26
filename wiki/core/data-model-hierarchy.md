# Data Model Hierarchy and Operator DSL

Four parallel representations of "structured data + JSON Schema":

| Class | File | Role |
|---|---|---|
| `DataModel` | `backend/pydantic/core.py` | Concrete instances with real data + validation (Pydantic-based) |
| `SymbolicDataModel` | `backend/common/symbolic_data_model.py` | Placeholder carrying only schema (for graph tracing) |
| `JsonDataModel` | `backend/common/json_data_model.py` | Backend-independent runtime: schema + json dict |
| `Variable` | `backend/common/variables.py` | Mutable state container, optimizable by optimizers |

## DataModel

**Purpose**: Pydantic-based structured data with JSON schema support.

**Key Methods**: `get_schema()`, `get_json()`, `to_symbolic_data_model()`, `to_json_data_model()`, `get_config()`/`from_config()`.

**Distinctive**: `serialize_as_any=True` on dump gives duck-typed polymorphic serialization (subclasses like `Entity`/`Relation` keep their own fields inside a generic container).

## SymbolicDataModel

**Purpose**: Placeholder carrying only a JSON schema (no data) — the tracing token for Functional API.

**Distinctive**: Operations like `.get()`, `.get_json()` raise `ValueError` with guiding messages (you're probably calling in the wrong context).

## JsonDataModel

**Purpose**: Backend-independent runtime carrier: schema + json dict.

**Key Methods**: `to_symbolic_data_model()`, `.get()`, `.keys()`, `.values()`, `.items()`, `.update()`, full operator DSL.

**Distinctive**: `get_nested_entity(key)` — resolves label-discriminated nested objects against `$defs` in schema, producing properly-typed submodels.

## Variable

**Purpose**: Mutable, named state container, optimizable by optimizers.

**Constructor**: `Variable(initializer, data_model=, trainable=True, name=, ...)`.

**Deferred initialization**: If constructed inside `StatelessScope`, a callable initializer is required and initialization is deferred; actual init happens when the scope exits.

**Key Methods**: `get_json()`, `assign(value)`, `to_json_data_model()`, dict-style access.

## The Operator DSL

All four classes support the same operator set:

- **`+` (Concat)** — Merge fields of two data models
- **`&` (And)** — `None` if either is `None`, else concat
- **`|` (Or)** — Return the non-`None` side, or concat if both present
- **`^` (Xor)** — `None` if both present, else the present one
- **`~` (Not)** — Output `None`
- **`in` (Contains)** — Schema containment check

Plus non-operator combinators: `.factorize()`, `.in_mask()`/`.out_mask()`, `.prefix()`/`.suffix()`.

**Distinctive**: The same operators work on symbolic (graph-tracing) and eager (execution) — automatically dispatches to `ops.Concat.symbolic_call()` (graph) or `ops.Concat.call()`/`()` (eager) via `run_maybe_nested`.

## Scopes

**name_scope** — Context manager pushing onto thread-local stack; variables get `/`-joined paths.

**StatelessScope** — Prevents in-place variable mutation; tracks pending state updates; initializes deferred variables on exit.

**SymbolicScope** — Marker (no mutation prevention); indicates "we are tracing the graph."

**op_scope** — Phase tracker (`"inference"` / `"reward"` / `"optimizer"`); used for attributing LM calls and wall-clock time.

## Related pages
- [Scopes](./scopes.md) — Detailed scope behavior
- [Module Call Lifecycle](./module-and-call-lifecycle.md) — How data models flow through modules
- [Knowledge Bases](../infrastructure/knowledge-bases.md) — Extended data models (Entity, Relation, KnowledgeGraph)
