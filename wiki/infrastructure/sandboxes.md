# Sandboxes (Container-Free Code Execution)

## Sandbox (Base Class)

**Purpose**: Stateful, restricted Python execution environment. Public surface mirrors E2B `AsyncSandbox` SDK for API compatibility.

**Key Methods**:
- `run_code(code_string)` → `Execution` (result + logs + output messages)
- `reset()` — Wipe state
- `dump()`/`load()` — Serialize/restore namespace
- `run_python_code`, `run_python_file`, `list_files`, `read_file`, `write_file`, `edit_file`, `search_files` — Ready-made tool methods (paginated, 1-based offsets)
- `create()`, `kill()`, `pause()`/`connect()`, `create_snapshot()`/`load()`, `fork()` — Full E2B lifecycle

**Implementation**: Provides shared machinery (history, bound functions, exceptions, result types); subclasses implement backends.

## MirageSandbox

**Purpose**: Production backend via Mirage (virtual filesystem + Python runtime binding).

**Constructor**:
```
MirageSandbox(
    workdir=None,             # Host directory to mirror (copy-in only)
    confine=True,             # Enable namespace/microVM confinement
    confine_network=False,    # Restrict egress
    allowed_hosts=None,       # For http_fetch tool
    memory_limit_mb=None,     # rlimit
    cpu_limit_seconds=None,   # rlimit
    max_processes=None,       # rlimit
    sandbox=None,             # Provide existing workspace
    # ... more options
)
```

**Distinctive Confinement** (fail-closed, non-configurable):
- **Linux**: User/mount/PID/net namespaces + seccomp + `pivot_root`
- **macOS Apple Silicon**: MicroVM (libkrun/Hypervisor.framework) + Linux confinement inside + Seatbelt profile on host
- **macOS Intel**: Seatbelt (TrustedBSD MAC) only (weaker)
- **Windows**: Unsupported natively (raises; use WSL2)
- `require_confinement=True` makes failure hard error

**State Persistence**: `dill`-serializes Python namespace after each snippet, restores before next — REPL-like persistence without Docker.

**Copy-on-write**: `workdir` copy-in only — real filesystem never modified.

**granted_capabilities()** — Self-audit method returning JSON (confined, backend, network mode, tools, mounts) — verify privilege surface before trusting with untrusted code.

## Filesystem / Commands Namespaces

**MirageFilesystem** / **MirageCommands** — Concrete implementations of the `Filesystem`/`Commands` abstract interface.

Subclasses (E2B compat) can override `Pty`, `Git` namespaces.

## Related pages
- [DeepAgent](../modules/agents.md) — Uses MirageSandbox for file/bash tools
- [RecursiveLanguageModelAgent](../modules/agents.md) — Uses for Python execution
- [PythonSynthesis](../modules/synthesis.md) — Executes learned scripts in sandbox
