# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Synalinks is a neuro-symbolic Language Model (LM) framework inspired by Keras. It provides a declarative API for building, training, and deploying LM-based applications including RAGs, autonomous agents, and self-evolving reasoning systems.

## Requirements

- **Python 3.12+** (3.13 and 3.14 also supported)
- **Platform**: Unix/macOS only. Windows has no native support — **WSL2 is required**.
- **Sandbox system dependencies** (for agent code execution):
  - **Linux**: `fuse3`, `libfuse3-3`, and (Ubuntu 23.10+) disabling `kernel.apparmor_restrict_unprivileged_userns` and `kernel.unprivileged_userns_clone`
  - **macOS (Apple Silicon)**: `libkrun` via `slp/krun` Homebrew tap
  - **Windows**: WSL2 + dependencies above for the WSL2 Linux environment

## Development Commands

```bash
# Install dependencies
./shell/install.sh

# Run tests with coverage (installs dev dependencies first)
./shell/test.sh

# Run a single test file
uv run pytest synalinks/src/path/to/test_file.py -v

# Run a specific test
uv run pytest synalinks/src/path/to/test_file.py::test_function_name -v

# Lint check
./shell/lint.sh

# Format code
./shell/format.sh

# Build documentation
./shell/doc.sh
```

### Development Notes

- `./shell/test.sh` runs `uv pip install --group dev` first — without dev deps, optional-dependency tests silently skip instead of failing. Don't assume a green run is valid if dev deps weren't installed.
- `./shell/format.sh` runs `black` scoped only to `synalinks/src/`, then `ruff check --fix` and `ruff format` across the whole repo. Both tools are applied, not just one.
- `./shell/lint.sh` runs `ruff check` then `ruff format --check`, both reading from `pyproject.toml` config.
- pytest treats all warnings as errors by default (`filterwarnings = ["error", ...]` in `pyproject.toml`) — a new warning anywhere can fail unrelated tests.

## Architecture

### Core Abstractions

The framework follows a Keras-like pattern with these core abstractions:

- **Module** (`synalinks/src/modules/module.py`): Base class for all composable units, similar to Keras Layers. Modules have:
  - `__init__()`: Define attributes and create variables
  - `build()`: Create state that depends on input shapes
  - `call()`: The forward pass logic (async)
  - `get_config()`/`build_from_config()`: Serialization support

- **Program** (`synalinks/src/programs/program.py`): Groups modules into trainable/deployable objects (like Keras Models). Inherits from both `Trainer` and `Module`. Supports:
  - Functional API: Chain module calls from `Input` to outputs
  - Subclassing: Override `call()` method
  - Sequential: Stack of single-input/single-output modules

- **DataModel**: Pydantic-based structured data with JSON schema support. All module I/O uses DataModels.

### Key Components

- **Generator** (`synalinks/src/modules/core/generator.py`): Core module for LM inference with structured outputs
- **FunctionCallingAgent** (`synalinks/src/modules/agents/function_calling_agent.py`): Autonomous agent with parallel tool calling
- **ChainOfThought** (`synalinks/src/modules/ttc/chain_of_thought.py`): Generator with thinking field for step-by-step reasoning

### Training System

- **Trainer** (`synalinks/src/trainers/trainer.py`): Provides `compile()` and `fit()` methods
- **Optimizers** (`synalinks/src/optimizers/`): In-context RL optimizers for prompt/example optimization
- **Rewards** (`synalinks/src/rewards/`): Reward functions like `ExactMatch`, `CosineSimilarity`, `LMAsJudge`, `RubricReward`
- **Metrics** (`synalinks/src/metrics/`): Training metrics (accuracy, precision/recall, F1Score, EM, regression metrics)
- **Callbacks** (`synalinks/src/callbacks/`): Keras-style `fit()` callbacks (EarlyStopping, ProgramCheckpoint, BackupAndRestore, CSVLogger, etc.)
- **Hooks** (`synalinks/src/hooks/`): Lightweight event system for runtime observability (separate from callbacks; used by agents for logging and monitoring)

### Additional Key Subsystems

- **Sandboxes** (`synalinks/src/sandboxes/`): Container-free code execution via `mirage-ai`; used by agents for tool execution and code evaluation
- **Knowledge Bases** (`synalinks/src/knowledge_bases/`): Embedded graph DB (Ladybug) and vector/SQL DB (LanceDB/DuckDB) abstraction for RAG systems — no external DB server required
- **Testing**: No `conftest.py` — tests subclass `synalinks.src.testing.test_case.TestCase` (extends `IsolatedAsyncioTestCase` + `parameterized.TestCase`), which handles `clear_session()`, loading `.env`, zeroing retry backoff for mocked tests, and resetting default LM/embedding/decision-model globals

### Backend and Data Models

The framework has Keras-style multi-backend scaffolding, but **Pydantic is currently the only implemented backend**. The backend layer is split:
- **`synalinks/src/backend/common/`** (backend-agnostic): `Variable`, `SymbolicDataModel`, `JsonDataModel`, name scoping, and state management utilities
- **`synalinks/src/backend/pydantic/`** (concrete implementation): Pydantic-based `DataModel` with JSON schema support

All module I/O uses `DataModel`. DataModels support combinator operators (`+`, `&`, `|`, `^`, `~`, `in`) for combining and branching data structures.

### Serialization

All objects are JSON-serializable via `synalinks.saving`. Custom objects need `@synalinks.saving.register_synalinks_serializable()` decorator.

## Code Conventions

- Use `uvx ruff` for linting with config in `pyproject.toml`
- Use `black` for formatting with 90 char line length (applied via `./shell/format.sh`, scoped to `synalinks/src/`)
- Tests are colocated with source files using `*_test.py` suffix
- All module `call()` methods are async
- New metrics, optimizers, or reward functions require core team approval — keep the library deliberately minimal. Discuss additions in Discord before submitting PRs.

## API Structure

Public API is exported via `synalinks/api/` directory. The `./shell/api_gen.sh` script (wrapping root `api_gen.py`) generates `synalinks/__init__.py` and `synalinks/api/` from exports decorated with `@synalinks_export()`. After adding or removing decorated exports, run `./shell/api_gen.sh` to regenerate — do not hand-edit `synalinks/__init__.py` or `synalinks/api/` (both are marked "DO NOT EDIT").

## CLI and Scaffolding

The `synalinks` CLI (`synalinks/cli/main.py`, click-based) provides `synalinks init` to scaffold new projects from bundled templates. Example usage patterns (RAGs, agents, SQL, MCP, training, callbacks, deployment) are in `examples/` and `guides/` at repo root — each has a `.py` file paired with a `.log` file showing expected output.
