# Wiki Maintenance Schema

This document defines how the Synalinks LLM Wiki is maintained. Following Andrej Karpathy's "LLM Wiki" pattern, the wiki is an evolving, agent-maintained knowledge base where understanding compounds over time rather than being rederived from scratch each session.

## Three Core Operations

### Ingest
When the Synalinks codebase changes (a module is added, removed, or its behavior meaningfully shifts):
1. Update the relevant wiki page under `wiki/modules/`, `wiki/training/`, `wiki/core/`, or `wiki/infrastructure/`
2. Add one line to `wiki/log.md` with: date, page touched, one-line reason (e.g., "2025-09-26 | modules/core.md | Generator now supports decision_model parameter")
3. If a page becomes orphaned (class removed from codebase) or contradicts the source, update or remove it

### Query
When an agent (Claude Code, a future human reader, or an external tooling agent) digs into some part of the framework:
- If the understanding is **not yet in the wiki**, add a new page or extend an existing one — don't let insight be lost to the conversation
- If the understanding **corrects or expands** existing wiki content, update the relevant page
- Append the update to `wiki/log.md`

### Lint
Periodically (at least once per major version release):
1. Check that every page listed in `wiki/index.md` actually exists
2. Verify no pages are orphaned (documenting removed classes)
3. Spot-check 3–5 pages against the source code for drift (class signature changes, renamed methods, moved files)
4. Cross-check links within pages (relative paths like `../core/data-model-hierarchy.md` should exist)

## Page Format Convention

Every **content page** (not meta pages like `index.md` or `log.md`) follows this structure:

```markdown
# Page Title (e.g., "Generator and Core Inference Modules")

**Overview paragraph** (1–2 sentences): What does this subsystem do? Why would you use it?

## ClassName

**Purpose**: One sentence describing what this class does.

**Signature**: Constructor parameters only, grouped logically. Example:
```
ClassName(
    schema=None,              # The DataModel of the output
    language_model=None,      # The LM to use (required)
    prompt_template=None,     # Custom prompt format
    examples=None,            # Few-shot examples (trainable)
    instructions=None,        # System instructions (trainable)
    temperature=None,         # LM parameter passthrough
    ...
)
```

**Inherits**: What base class(es) it extends, and via what. Example: `Module` (directly) or `Generator` → indirectly inherits `Module`.

**Distinctive behavior**: 
- Bullet list of non-obvious characteristics, quirks, or important implementation details
- This is where the "why" goes — e.g., "trainable state is stored as an `Instructions` variable, rewritten by optimizers"
- Include references to related classes or files when helpful (e.g., "uses `synalinks/src/modules/core/generator.py` internally")

## Related pages
- [Link to related subsystem](../other/page.md)
- [Another context](../core/foundational-concept.md)
```

**Key conventions:**
- Paths are always **relative** (e.g., `../training/optimizers.md`, not absolute file paths)
- Class names are in **bold** (`**ClassName**`) for scannability
- Keep one class per `##` heading (don't nest classes under a category heading)
- "Signature" sections list only **key parameters**, not every optional kwarg
- Link to other wiki pages when explaining concepts (e.g., explaining what a `Variable` is → link to `../core/data-model-hierarchy.md`)

## Directory Structure

```
wiki/
  README.md             # Entry point for humans and agents
  schema.md             # This file
  index.md              # Catalog of all pages
  log.md                # Chronological changelog
  modules/              # All user-facing module classes
  training/            # Training system (Trainer, optimizers, rewards, metrics, callbacks, hooks)
  core/                # Core framework concepts (Module, Program, DataModel, scopes)
  infrastructure/      # Backend/storage (KnowledgeBase, Sandbox, serialization, datasets)
  patterns/            # Cross-cutting patterns and design decisions
```

## When NOT to create a page

- **Per-class pages are not created lightly**: grouping by subsystem keeps the wiki scannable. Only create a dedicated page if a class is truly central and complex (e.g., `Module`, `OMEGA`, `MirageSandbox`). Otherwise, include it in a subsystem page.
- **Thin utilities** like `initializers/`, `ops/`, or `tree/` are documented *inline* within the pages they're used in, not as separate pages.
- **Examples and tutorial content** belong in the repo's `examples/`, `guides/`, or `docs/` directories, not the wiki.

## Evolution and Maintenance

- **Commit frequency**: Pages should be edited and committed atomically with the code change that prompted them (or as a follow-up commit in the same PR)
- **Agent updates**: When Claude Code (or future agents) discovers something new about the framework, they should add/update the wiki and commit it — this is part of the development workflow, not separate
- **History is in git**: The wiki pages are version-controlled. Blame/log on each file shows its evolution. Don't worry about keeping a detailed edit history *within* a page — git does that

---

## Example workflow

**Scenario**: A new `ChainOfReasoning` module is added to `synalinks/src/modules/core/`.

1. **Ingest**: Update `wiki/modules/core.md` to add a `## ChainOfReasoning` section with Purpose/Signature/Inherits/Distinctive behavior
2. **Log**: Append to `wiki/log.md`: `2025-09-26 | modules/core.md | Added ChainOfReasoning (new module for chain-of-reasoning patterns)`
3. **Index**: If relevant, update `wiki/index.md` to mention the new module under the Modules category
4. **Commit**: `git add wiki/` && `git commit -m "docs(wiki): add ChainOfReasoning to modules/core.md"`

**Scenario**: An agent discovers that `OMEGA` supports a new `novelty_search_metric` parameter not documented in the wiki.

1. **Query**: Update `wiki/training/optimizers.md`, OMEGA section, Signature to include `novelty_search_metric=...`
2. **Log**: `2025-09-27 | training/optimizers.md | Added novelty_search_metric parameter to OMEGA signature`
3. **Commit**: Similar to above

---

This schema is not rigid. As the wiki grows, agents and maintainers should feel free to:
- Reorganize pages if a subsystem becomes too large (split it)
- Create cross-cutting pages under `patterns/` if a concept recurs across multiple subsystems
- Update this schema itself if a better structure emerges
