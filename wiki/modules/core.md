# Core Inference and Routing Modules

Core modules for generating structured outputs, routing decisions, and executing actions. These are the foundational building blocks for most Synalinks programs.

## Generator

**Purpose**: Use a language model (or decision model) to generate a structured data model from arbitrary input.

**Signature**:
```
Generator(
    schema=None,              # The DataModel schema of the output
    data_model=None,          # Alternative to schema (for type hints)
    language_model=None,      # The LM to use; required unless decision_model given
    prompt_template=None,     # Custom prompt format (default: f-string template)
    prompt_variables=None,    # Variables interpolated into prompt_template
    examples=None,            # Few-shot examples (trainable; stored in Instructions variable)
    instructions=None,        # System instructions (trainable)
    seed_instructions=None,   # Immutable seed instructions
    use_inputs_schema=False,  # Include input schema in prompt
    use_outputs_schema=False, # Include output schema in prompt
    return_inputs=False,      # Return input with output
    temperature=None,         # LM sampling parameter
    max_tokens=None,          # Max output tokens
    top_p, top_k, reasoning_effort=None,  # More LM parameters
    streaming=False,          # Stream output incrementally
    tools=None,               # Tool definitions (optional, merged with per-call ones)
    tool_schemas=None,        # Tool JSON schemas
    decision_model=None,      # Use DecisionModel instead of LanguageModel (for 5-category classification)
    name=None,                # Module name
    description=None,
    trainable=True            # Allow optimizers to rewrite instructions/examples
)
```

**Inherits**: `Module` (directly).

**Distinctive behavior**:
- **Trainable state**: `instructions` and `examples` are stored as a single `Instructions` variable — this is the unit rewritten by optimizers (OMEGA, GreedyOptimizer, etc.)
- **Tool support**: Native function-calling is supported; tools passed here are merged with per-call tools
- **Streaming**: Returns a `StreamingIterator` if `streaming=True`
- **Decision model fallback**: Can delegate to a `DecisionModel` (fast/cheap) instead of an LM when the output schema is all yes/no/choice questions
- **Schema inference**: Exports `default_prompt_template()` and `default_instructions()` helpers

## Action

**Purpose**: Use a language model to infer a tool's arguments from input, execute the tool, and return both inputs and tool outputs.

**Signature**:
```
Action(
    tool,                     # The Tool to call; required
    language_model=None,      # LM to infer tool arguments
    prompt_template=None,     # Custom prompt
    examples=None,            # Few-shot examples (trainable)
    instructions=None,        # Instructions (trainable)
    temperature=None, max_tokens=None, ...  # LM parameters
    use_inputs_schema=False,  # Include input schema in prompt
    use_outputs_schema=False, # Include output schema in prompt
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module` (internally wraps a `Generator` targeting the tool's input schema).

**Distinctive behavior**:
- **Error handling**: Tool execution exceptions are caught and folded into the output as `{"error": str(e)}` — the module never raises
- **Tool signature**: Requires `tool.func` to have a complete docstring `Args:` section (raises `ValueError` otherwise)

## Decision

**Purpose**: Single-label classification via an LM or decision model — pick one of N labels with a constrained output guarantee (result is always one of the provided labels).

**Signature**:
```
Decision(
    question=None,            # The classification question
    labels=None,              # List of valid label strings
    language_model=None,      # LM to use; required unless decision_model given
    prompt_template=None,     # Custom prompt
    examples=None,            # Few-shot examples (trainable)
    instructions=None,        # Instructions (trainable)
    temperature=None, max_tokens=None, ...  # LM parameters
    use_inputs_schema=False,  # Include input schema
    use_outputs_schema=False, # Include output schema
    decision_model=None,      # Use DecisionModel instead (fast yes/no classifier)
    min_confidence=None,      # With DecisionModel: threshold to abstain (return None)
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **LM mode**: Output has a `thinking` field (chain-of-thought); `mode` inferred from LM capability
- **DecisionModel mode**: No `thinking` field; `min_confidence` can make it abstain (return `None`) if uncertainty is high

## MultiDecision

**Purpose**: Multi-label selection — pick zero, one, or multiple labels from a set (via a dynamic array-of-enum schema).

**Signature**:
```
MultiDecision(
    question=None,            # The selection question
    labels=None,              # List of valid label strings
    language_model=None,      # LM to use
    inline=True,              # Format labels inline in prompt (vs. one per line)
    prompt_template=None,     # Custom prompt
    examples=None, instructions=None,  # Trainable
    temperature=None, max_tokens=None, ...  # LM parameters
    use_inputs_schema=False,  # Include input/output schemas
    use_outputs_schema=False,
    decision_model=None,      # Use DecisionModel instead
    threshold=None,           # With DecisionModel: keep labels with prob >= threshold
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **LM mode**: Asks for all labels at once; output is `{"choices": ["label1", "label3", ...]}`
- **DecisionModel mode**: Each label becomes a separate yes/no "noul" question; `threshold` (default 0.5) decides which are kept

## Branch

**Purpose**: Route one or more inputs to sub-modules/programs based on a decision decision (single-label or multi-label); unselected branches emit `None`.

**Signature**:
```
Branch(
    question=None,            # The routing question
    labels=None,              # Possible branches (list of strings)
    branches=None,            # Dict {label: module_or_program} mapping
    inject_decision=True,     # Include decision output in branch inputs
    return_decision=True,     # Return the decision in module outputs
    language_model=None,      # LM for the decision
    prompt_template=None,     # Custom prompt
    examples=None, instructions=None,  # Trainable
    temperature=None, max_tokens=None, ...  # LM parameters
    use_inputs_schema=False,  # Include input/output schemas
    use_outputs_schema=False,
    decision_type=Decision,   # Use Decision or MultiDecision (for simultaneous branches)
    decision_model=None,      # Use DecisionModel instead
    min_confidence=None,      # With DecisionModel (Decision mode)
    threshold=None,           # With DecisionModel (MultiDecision mode)
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Pluggable decision type**: Set `decision_type=MultiDecision` to route to multiple branches in parallel (each with a fixed positional output slot)
- **Concurrent execution**: Branches run via `asyncio.gather` when using `MultiDecision`
- **None for unselected**: Branches not selected emit `None` at their fixed position

## Identity

**Purpose**: Pass-through placeholder — no-op module used to scaffold a program's architecture before implementation.

**Signature**:
```
Identity(**kwargs)  # Ignores all kwargs
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Structural clone**: Clones nested input structures so symbolic building works even though execution is a pass-through

## Not

**Purpose**: Placeholder that always outputs `None` — used to implement guards/stop-conditions combined with `|` (Or) or a `Branch` with `Xor`.

**Signature**:
```
Not(**kwargs)  # Ignores all kwargs
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Symbolic schema**: Still clones input schema during `compute_output_spec` so the graph can be traced even though `call()` always returns `None`

## InputModule and Input Factory

**Purpose**: `InputModule` defines an input placeholder (entry point) for a `Program`; `Input()` factory returns a `SymbolicDataModel` (the Keras `Input`-style entry point). `ImageInput()`/`AudioInput()` are multimodal variants using `ChatMessages` with image/audio content parts.

**Signature**:
```
# Factory functions (return SymbolicDataModel)
Input(schema=None, data_model=None, optional=False, name=None)
ImageInput(name=None)
AudioInput(name=None)

# The module class (used internally by Input())
InputModule(
    schema=None,              # The input DataModel schema
    input_data_model=None,    # Alternative to schema
    optional=False,           # Allow None inputs
    name=None, ...
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Entry point**: Immediately creates a `Node` in the ops graph when instantiated (not called like normal modules)
- **Built immediately**: Sets `self.built = True` at construction (no explicit `build()` call needed)

## Lambda

**Purpose**: Wrap an arbitrary sync/async callable as a stateless `Module` for custom data transforms, without subclassing `Module`.

**Signature**:
```
Lambda(
    function,                 # Callable(input: DataModel) -> DataModel | dict | None
    schema=None,              # Output DataModel schema (inferred if possible)
    data_model=None,          # Alternative to schema
    name=None, description=None,
    trainable=False
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Function flexibility**: Function may return a `dict`, `DataModel`, `JsonDataModel`, or `None` (to short-circuit via `|` / `Branch`)
- **Serialization requirement**: Function must be named (lambda functions cannot be saved); use `@synalinks.saving.register_synalinks_serializable()` if custom

## Tool

**Purpose**: Wrap an async function as a callable "tool" module, auto-deriving its input/output JSON schema from type hints and docstring.

**Signature**:
```
Tool(
    func,                     # Async callable with type hints and complete Args: docstring
    name=None,                # Tool name (default: func.__name__)
    description=None,         # Tool description (default: func.__doc__)
    trainable=False
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Docstring requirement**: Requires a complete `Args:` section in the docstring (raises `ValueError` if missing)
- **Retry**: Automatically retried (up to 3 attempts via `tenacity`) on execution failure
- **Media support**: Can surface `synalinks.Image`/`synalinks.Audio` return values as content parts in agent messages

## Related pages
- [Agents](./agents.md) — Higher-level autonomous agents built from these primitives
- [Test-Time Compute](./ttc.md) — ChainOfThought and SelfCritique which wrap core modules
- [Module Call Lifecycle](../core/module-and-call-lifecycle.md) — How the underlying `Module.__call__` works
- [Programs](../core/programs.md) — How to compose these modules into trainable programs
