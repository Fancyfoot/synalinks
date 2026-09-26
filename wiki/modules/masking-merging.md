# Masking and Merging Modules

Modules for structural manipulation of data models.

## InMask

**Purpose**: Keep only specified fields (allow-list); drop everything else.

**Signature**:
```
InMask(
    mask=None,                # List of field names to keep; required, non-empty
    pattern=None,             # Regex pattern to match field names (alternative to explicit list)
    name=None, description=None,
    trainable=False
)
```

**Inherits**: `Module`.

## OutMask

**Purpose**: Remove specified fields (deny-list); keep everything else.

**Signature**:
```
OutMask(
    mask=None,                # List of field names to drop; required, non-empty
    pattern=None,             # Regex pattern to match field names
    name=None, description=None,
    trainable=False
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Common use case**: Strip `thinking` field from `ChainOfThought` output via `OutMask(mask=["thinking"])`

## Concat (alias Concatenate)

**Purpose**: Concatenate fields of multiple data models field-wise; raises if any input is `None`.

**Signature**:
```
Concat(**kwargs)  # Arbitrary number of inputs
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Operator backing**: Implements the `+` operator on data models: `output1 + output2`

## And

**Purpose**: Concatenate like `Concat`, but return `None` if *any* input is `None`.

**Signature**:
```
And(**kwargs)  # Arbitrary number of inputs
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Operator backing**: Implements `&` operator: `output1 & output2`
- **Short-circuit**: Returns `None` immediately if any input is `None`

## Or

**Purpose**: Concatenate all non-`None` inputs; return single non-`None` value if only one present; return `None` if all are `None`.

**Signature**:
```
Or(**kwargs)  # Arbitrary number of inputs
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Operator backing**: Implements `|` operator: `output1 | output2`
- **Ubiquitous in graphs**: Used pervasively to recombine `Branch` outputs (unselected branches emit `None`, which are filtered out)

## Xor

**Purpose**: Exactly-one-of semantics — return `None` if more than one input is non-`None`; pass through the single non-`None` input otherwise; return `None` if all are `None`.

**Signature**:
```
Xor(**kwargs)  # Arbitrary number of inputs
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Operator backing**: Implements `^` operator: `output1 ^ output2`
- **Guard patterns**: Used for guard/stop-condition logic paired with `Not`/`Identity`

## Related pages
- [Core Modules](./core.md) — Branch which produces outputs you often mask or merge
- [Data Model Hierarchy](../core/data-model-hierarchy.md) — The operator DSL (+ & | ^ ~ in) these modules implement
- [Masking and Merging Patterns](../patterns/design-decisions.md) — Design rationale for the operator DSL
