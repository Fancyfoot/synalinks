# Autonomous Agent Modules

Agents that autonomously (or interactively) decompose goals into multi-turn tool calling sequences. All agents inherit from or extend `FunctionCallingAgent`, which is the root of the agent family.

## FunctionCallingAgent

**Purpose**: Base trainable agent that picks tools from a set, executes them (often concurrently), and loops until done or `max_iterations` reached. The "parallel function-calling" agent.

**Signature**:
```
FunctionCallingAgent(
    schema=None,              # Output DataModel schema
    data_model=None,          # Alternative to schema
    language_model=None,      # LM to pick tools
    prompt_template=None,     # Custom prompt
    examples=None, instructions=None,  # Trainable
    final_instructions=None,  # Instructions for final answer formatting
    temperature=None, max_tokens=None, ...  # LM parameters
    use_inputs_schema=False,  # Include input/output schemas
    use_outputs_schema=False,
    reasoning_effort=None,    # LM extended thinking parameter
    use_chain_of_thought=False,  # Prepend thinking field
    tools=None,               # List of Tool or callable definitions
    autonomous=True,          # Auto-execute tools vs. interactive (human-in-loop)
    return_inputs_with_trajectory=True,  # Include trajectory in output
    max_iterations=5,         # Max tool-call turns
    workdir=None,             # Working directory (for file-based tools)
    skills=None,              # Agent Skills roots for discovery
    streaming=False,          # Stream output
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module` (directly; base of agent family).

**Distinctive behavior**:
- **Two modes**: `autonomous=True` executes tools automatically; `autonomous=False` is interactive (human-in-loop, one step at a time, ignores `max_iterations`)
- **Parallel tool calling**: A single turn's tool calls are dispatched concurrently via `asyncio.gather`
- **MCP compatible**: Supports MCP (Model Context Protocol) tools natively
- **Agent Skills**: Can auto-discover Agent Skills from `skills=` roots; exposes `read_skill` tool
- **AGENTS.md discovery**: Auto-loads `AGENTS.md` from `workdir` if present (convention for agent instructions)
- **Template method pattern**: Subclasses override hooks (`_get_builtin_tools`, `_native_tools`, `_dispatch_tool_calls`, `_final_result`, `_begin_call`, etc.) rather than reimplementing `call()`
- **Trajectory tracking**: Full turn-by-turn log of tool calls/outputs included in output or available as `self.trajectory`

## DeepAgent

**Purpose**: Coding agent whose tools are a sandboxed, copy-on-write mirror of a working directory (read/write/edit files, run bash commands).

**Signature**:
```
DeepAgent(
    # All FunctionCallingAgent params, plus:
    workdir,                  # Host directory to mirror into sandbox; required
    sub_language_model=None,  # Cheaper LM for subtasks
    timeout=30.0,             # Per-tool timeout
    sandbox=None,             # Provide a MirageSandbox (optional; one is created if omitted)
    max_subagent_depth=0,     # Depth of nested subagents allowed
    max_iterations=10         # Default higher than base FunctionCallingAgent
)
```

**Inherits**: `FunctionCallingAgent`.

**Distinctive behavior**:
- **Host-safe by design**: All tools operate on a `MirageSandbox` mount, never the real filesystem
- **Copy-on-write**: Changes made by the agent stay in the sandbox; `workdir` is never modified
- **Subagents**: Can spawn isolated subagents (`spawn_subagents`/`merge_subagent`/`discard_subagent`), each with its own forked sandbox
- **File tools**: Built-in `read_file`, `list_files`, `search_files`, `write_file`, `edit_file`, `run_bash`
- **Conflict detection**: Merging a subagent's results detects file conflicts and requires explicit resolution

## RecursiveLanguageModelAgent (RLM)

**Purpose**: "Recursive Language Model" agent — the LM writes/executes Python code snippets in a persistent sandbox REPL instead of calling native tools; can recursively delegate semantic sub-tasks to a cheaper sub-LM.

**Signature**:
```
RecursiveLanguageModelAgent(  # Alias: RLM
    # All FunctionCallingAgent params, plus:
    sub_language_model=None,  # Cheaper LM for llm_query subtasks
    sandbox_tools=None,       # Tools available inside sandbox code
    native_tools=None,        # Tools called directly (not from code)
    timeout=60.0,             # Per-snippet timeout
    recursive=True,           # Allow sub-LM queries
    max_llm_calls=50,         # Limit recursive sub-LM calls per snippet
    max_output_chars=10_000,  # Truncate long snippet outputs
    sandbox=None,             # Provide a MirageSandbox (optional)
    sandbox_type=None,        # Sandbox backend hint
    max_subagent_depth=0,     # Nested subagent depth
    max_iterations=20         # Default higher than base
)
```

**Inherits**: `FunctionCallingAgent`.

**Distinctive behavior**:
- **Code-as-tool**: LM writes Python snippets; the agent executes them in a persistent sandbox REPL, not as isolated calls
- **Persistent state**: Snippets' side effects (variables, imports, file state) carry forward within one session
- **submit(result=...)**: Only termination signal — the LM must explicitly call this to end the agent
- **Recursive delegation**: Inside a snippet, `llm_query(question)` delegates to `sub_language_model` (e.g., asking a cheaper model "are these two dicts equal?")
- **Subagents**: Can fork both files and REPL state; only one subagent's REPL can be merged per batch (backend limitation)
- **Native vs. sandbox tools**: `native_tools` are called directly; `sandbox_tools` are globals inside snippets
- **Implements the RLM paper** (arXiv:2512.24601)

## SQLAgent

**Purpose**: Ready-to-use SQL analyst agent with three built-in tools: `get_database_schema`, `get_table_sample`, `run_sql_query` against a `KnowledgeBase`.

**Signature**:
```
SQLAgent(
    # All FunctionCallingAgent params, plus:
    knowledge_base,           # KnowledgeBase with SQL adapter; required
    k=50,                     # Row limit for result sets
    output_format="csv"|"json"  # Result format
)
```

**Inherits**: `FunctionCallingAgent`.

**Distinctive behavior**:
- **Safety**: SQL safety enforced by the KB's own parser (`SELECT`-only, `enable_external_access=false`), not string filtering
- **Result capping**: Wraps LM SQL in an outer `LIMIT k` to prevent unbounded queries

## CypherAgent

**Purpose**: Graph analyst counterpart to `SQLAgent` — three built-in tools for a graph-backed `KnowledgeBase`: `get_graph_schema`, `get_node_sample`, `run_cypher_query`.

**Signature**:
```
CypherAgent(
    # All FunctionCallingAgent params, plus:
    knowledge_base,           # KnowledgeBase with graph adapter; required
    k=50,                     # Node limit
    output_format="csv"|"json"  # Result format
)
```

**Inherits**: `FunctionCallingAgent`.

**Distinctive behavior**:
- **Read-only enforcement**: Cypher queries are validated to reject `CREATE`/`MERGE`/`SET`/`DELETE`/etc. by the graph adapter
- **Auto-schema**: Default instructions auto-generated from the KB's actual node/relation labels

## VectorRAGAgent

**Purpose**: Retrieval-augmented agent with three built-in tools: `get_knowledge_base_schema`, `search_knowledge_base`, `get_record_by_id`.

**Signature**:
```
VectorRAGAgent(
    # All FunctionCallingAgent params, plus:
    knowledge_base,           # KnowledgeBase; required
    search_type="hybrid_fts"|"similarity"|"fulltext",  # Search strategy
    k=5,                      # Results per search
    similarity_threshold=None,  # Cutoff for similarity search
    fulltext_threshold=None,  # Cutoff for fulltext search
    output_format="csv"|"json"  # Result format
)
```

**Inherits**: `FunctionCallingAgent`.

**Distinctive behavior**:
- **LM-driven retrieval**: The LM decides *if*, *which table*, and *how to phrase* the search (vs. always-retrieve RAG)
- **Multiple searches**: The agent can issue multiple searches in one turn

## Related pages
- [Core Modules](./core.md) — Foundational building blocks (Generator, Decision, Branch)
- [Module Call Lifecycle](../core/module-and-call-lifecycle.md) — How agents execute
- [Sandboxes](../infrastructure/sandboxes.md) — MirageSandbox used by DeepAgent and RLM
- [Knowledge Bases](../infrastructure/knowledge-bases.md) — KB used by SQLAgent, CypherAgent, VectorRAGAgent
