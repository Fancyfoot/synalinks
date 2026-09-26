# Synalinks LLM Wiki

A persistent, agent-maintained knowledge base for the Synalinks framework. This wiki is built following Andrej Karpathy's "LLM Wiki" pattern: instead of rederiving understanding of the codebase from scratch, agents and humans incrementally build and maintain a structured, interlinked collection of pages that compounds over time.

## What is this?

Synalinks is a neuro-symbolic Language Model framework inspired by Keras. It has 100+ modules, a rich training system (optimizers, rewards, metrics, callbacks), and layered infrastructure (backends, knowledge bases, sandboxes). This wiki documents all of it:

- **What each module does** — purpose, constructor signature, inheritance, distinctive behavior
- **How the training system works** — the full training loop, optimizer families, reward/metric types, observability hooks
- **Core abstractions** — how `Module`, `Program`, `DataModel`, and scopes work together
- **Infrastructure** — knowledge bases (SQL/graph), sandboxes (code execution), serialization
- **Design patterns** — why certain choices were made, common anti-patterns to avoid

Unlike a codebase README or API docs, the wiki is **living knowledge**: when you discover something about the framework, you update the wiki so the next session (or agent) benefits from that understanding.

## How to use this wiki

### For learning

Start at [index.md](./index.md) to browse topics by category. For a quick introduction to the framework itself, read:
1. [Module and Call Lifecycle](./core/module-and-call-lifecycle.md) — the foundation
2. [Programs](./core/programs.md) — how to build a trainable application
3. [Data Model Hierarchy](./core/data-model-hierarchy.md) — how I/O works
4. Then explore modules, training system, or infrastructure as needed

### For developing

When you modify or add a module/feature:
1. **Update the relevant wiki page** — add a new subsection or edit an existing one
2. **Append to [log.md](./log.md)** — one line describing what changed and why
3. **Commit with the code** — `git add wiki/` and commit it alongside your changes (or as a follow-up commit in the same PR)

**Don't skip the wiki**: Writing it down forces clarity and saves the next person (or future-you) from re-discovering it.

### For querying

When an agent (Claude Code, a research tool, or a future maintainer) needs to understand something:
1. **Check the wiki first** — it should have the answer organized by subsystem
2. **If the answer isn't there, add it** — the wiki grows when you use it and find gaps
3. **If the answer is wrong, fix it** — pages drift from code; lint and update when you spot issues

## Navigation

- **[index.md](./index.md)** — catalog of all pages by category
- **[log.md](./log.md)** — chronological changelog of edits
- **[schema.md](./schema.md)** — how to maintain this wiki (Ingest/Query/Lint workflows, page format)

## Directory structure

```
wiki/
  README.md             ← you are here
  schema.md             ← maintenance contract
  index.md              ← catalog
  log.md                ← changelog
  
  modules/              ← all user-facing module classes
    core.md
    agents.md
    ttc.md
    synthesis.md
    knowledge-modules.md
    masking-merging.md
    retrievers.md
    rerankers.md
    models.md
  
  training/             ← training system
    trainer.md
    optimizers.md
    rewards.md
    metrics.md
    callbacks.md
    hooks.md
  
  core/                 ← core framework concepts
    module-and-call-lifecycle.md
    programs.md
    data-model-hierarchy.md
    scopes.md
  
  infrastructure/       ← backend/storage/serialization
    knowledge-bases.md
    sandboxes.md
    saving.md
    datasets.md
  
  patterns/             ← cross-cutting patterns
    observability-two-systems.md
    design-decisions.md
```

## Contributing to the wiki

See [schema.md](./schema.md) for the detailed maintenance guidelines. In short:

- **Ingest**: When code changes, update the wiki
- **Query**: When you learn something not in the wiki, add it
- **Lint**: Periodically check for drift between wiki and code

Write pages as if explaining to another AI or engineer unfamiliar with the codebase. Assume readers know Python/Keras/neural networks but not Synalinks yet.

---

**Last updated**: See [log.md](./log.md) for full history.

**Maintained by**: Claude Code and future agents/maintainers.
