# Wiki Index

Complete catalog of all pages organized by topic.

## Meta Pages
- [README](./README.md) — What is this wiki and how to use it
- [Schema](./schema.md) — Maintenance guidelines (Ingest/Query/Lint, page format)
- [Log](./log.md) — Chronological changelog of all edits

## Modules (User-Facing Classes)

### Core Inference & Routing
- [Core Modules](./modules/core.md) — Generator, Action, Decision, MultiDecision, Branch, Identity, Not, InputModule/Input, Lambda, Tool

### Agents & Autonomous Systems
- [Agents](./modules/agents.md) — FunctionCallingAgent, DeepAgent, RLM (RecursiveLanguageModelAgent), SQLAgent, CypherAgent, VectorRAGAgent

### Test-Time Compute
- [Test-Time Compute (TTC)](./modules/ttc.md) — ChainOfThought, SelfCritique

### Synthesis & Learning
- [Synthesis Modules](./modules/synthesis.md) — PythonSynthesis, SequentialPlanSynthesis

### Knowledge & Retrieval
- [Knowledge Modules](./modules/knowledge-modules.md) — EmbedKnowledge, RetrieveKnowledge, StampKnowledge, UpdateKnowledge, Text2SQL, Text2Cypher
- [Retrievers](./modules/retrievers.md) — 25 retriever classes (table/entity/relation/path/graph level, similarity/fulltext/regex/hybrid variants)
- [Rerankers](./modules/rerankers.md) — RRFReranker (reciprocal rank fusion)

### Structural Manipulation
- [Masking & Merging](./modules/masking-merging.md) — InMask, OutMask, Concat, And, Or, Xor

### Model Wrappers
- [Language/Embedding/Decision Models](./modules/models.md) — LanguageModel, EmbeddingModel, DecisionModel

## Training System

### Core Training Loop
- [Trainer](./training/trainer.md) — Trainer mixin, compile()/fit()/evaluate()/predict() methods, data adapters

### Optimization
- [Optimizers](./training/optimizers.md) — Optimizer base, GreedyOptimizer, RandomFewShot, EvolutionaryOptimizer, OMEGA (Dominated Novelty Search)

### Evaluation
- [Rewards](./training/rewards.md) — Reward base, ExactMatch, CosineSimilarity, BatchReward, ComposableReward, LM-based judges (LMAsJudge, RLMAsJudge, AgentAsJudge, DeepAgentAsJudge), RubricsAsJudge + 28 rubric presets
- [Metrics](./training/metrics.md) — Metric base, accuracy/F-score/precision-recall/regression, PassAtK sampling metrics, operational metrics (LM/EM/program), pass@k variants

### Observability (Training-Phase)
- [Callbacks](./training/callbacks.md) — Callback, CallbackList, History, ProgbarLogger, CSVLogger, EarlyStopping, ProgramCheckpoint, BackupAndRestore, BudgetStopping, callbacks.Monitor (MLflow)
- [Hooks](./training/hooks.md) — Hook, HookList, Logger, Recorder (JSONL tracing), hooks.Monitor (MLflow distributed tracing), trace_context

## Core Framework

### Foundations
- [Module and Call Lifecycle](./core/module-and-call-lifecycle.md) — Module base class, __call__ lifecycle, CallContext, Operation/Node/Function graph tracing

### High-Level Structures
- [Programs](./core/programs.md) — Program (trainable container), Functional API (DAG-based), Sequential (linear stack)

### Data Representation
- [Data Model Hierarchy](./core/data-model-hierarchy.md) — DataModel, SymbolicDataModel, JsonDataModel, Variable + the operator DSL (+ & | ^ ~ in)

### Execution Context
- [Scopes](./core/scopes.md) — name_scope, StatelessScope, SymbolicScope, op_scope (phase tracking), global_state

## Infrastructure

### Knowledge & Data
- [Knowledge Bases](./infrastructure/knowledge-bases.md) — KnowledgeBase façade, SQL/vector adapters (DuckDB/LanceDB), graph adapter (Ladybug)
- [Datasets](./infrastructure/datasets.md) — Dataset streaming, format loaders (CSV/JSON/Parquet/Text/Image/HuggingFace), built-in benchmark suite

### Execution Environment
- [Sandboxes](./infrastructure/sandboxes.md) — Sandbox base, MirageSandbox (container-free with namespaces/microVM confinement), e2b API compatibility

### Serialization & State
- [Saving](./infrastructure/saving.md) — SynalinksSaveable, object_registration, serialization_lib (round-trip JSON configs)

## Cross-Cutting Patterns

### Observability
- [Two Systems of Observability](./patterns/observability-two-systems.md) — callbacks/ (training-loop phase) vs hooks/ (per-call) — when to use each, how they interact

### Architecture
- [Design Decisions](./patterns/design-decisions.md) — why Pydantic-only backend, composition-over-inheritance for LM modules, JSON-native trainable state, op_scope phases, etc.

---

**Page count**: ~28 content pages + 3 meta pages = 31 total

**Last indexed**: 2025-09-26

Browse by reading [README](./README.md) or jump to a category above.
