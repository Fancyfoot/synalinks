# Synthesis Modules

Modules that learn trainable transformations (code or plans) rather than prompts.

## PythonSynthesis

**Purpose**: Learn a trainable Python script that transforms JSON input → JSON output, executed inside a `MirageSandbox`.

**Signature**:
```
PythonSynthesis(
    schema=None, data_model=None,
    python_script=None,       # The script (trainable variable)
    seed_scripts=None,        # Initial script options
    default_return_value=None,  # Return value if script doesn't set result
    return_python_script=False,  # Include the script in output
    timeout=5,                # Per-execution timeout
    tools=None,               # Sandbox-accessible tools (exposed as global functions)
    sandbox=None,             # Provide a MirageSandbox (optional)
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **The script itself is trainable**: Stored as a `PythonScript` (trainable `Variable`) — optimizers rewrite it, not prompts
- **Sandbox execution**: Runs inside a real sandbox REPL, not simulated
- **Result variable required**: Script must assign its result to a variable literally named `result`
- **Advanced optimizers only**: Requires OMEGA or EvolutionaryOptimizer, not RandomFewShot
- **Sandbox tools**: `tools=` become plain global functions inside the script's namespace

## SequentialPlanSynthesis

**Purpose**: Learn a step-by-step plan (trainable list of step strings) executed sequentially by a pluggable `runner` (typically a `Generator`, `ChainOfThought`, or `FunctionCallingAgent`).

**Signature**:
```
SequentialPlanSynthesis(
    schema=None, data_model=None,
    language_model=None,      # LM to execute each step (via runner)
    steps=None,               # Initial plan steps (trainable variable)
    seed_steps=None,          # Alternative initial steps
    runner=None,              # Module to execute each step; required
    return_inputs=True,       # Include input in output
    reasoning_effort=None,    # LM parameter
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Plan is trainable**: `steps` list is optimized by advanced optimizers (OMEGA/EvolutionaryOptimizer)
- **Sequential chaining**: Each step's output feeds as input to the next step
- **Runner requirement**: `runner` must have `return_inputs=False` (SequentialPlanSynthesis re-concatenates inputs+previous-output each iteration)
- **Advanced optimizers only**: Requires OMEGA or EvolutionaryOptimizer
- **Empty plan edge case**: With `steps=[]`, degenerates to a single runner call

## Related pages
- [Optimizers](../training/optimizers.md) — OMEGA and EvolutionaryOptimizer that train these modules
- [Sandboxes](../infrastructure/sandboxes.md) — MirageSandbox backing PythonSynthesis
- [Core Modules](./core.md) — Runner modules used by SequentialPlanSynthesis
