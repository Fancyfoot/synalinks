# Datasets (Streaming and Benchmarks)

## Dataset (Base Class)

**Purpose**: Streaming dataset abstraction for `program.fit/evaluate/predict(x=...)` or `KnowledgeBase.update(dataset)`.

**Constructor**:
```
Dataset(
    input_template=None,      # Jinja2 template rendering raw rows to input DataModel/schema
    output_template=None,     # Template rendering to output DataModel/schema
    batch_size=32,            # Batch size
    limit=None,               # Max rows
    repeat=None,              # Repeats (used for GRPO-style rollout grouping)
    shuffle=True              # Shuffle rows
)
```

**Workflow**: `_iter_rows()` yields raw dicts → Jinja2 templates render → validate against schema → batch and yield `(x,)` or `(x, y)` object-array tuples.

## Format Loaders

All inherit from `Dataset`:

**CSVDataset**, **JSONDataset**, **JSONLDataset**, **ParquetDataset** — File format loaders.

**TextDataset**, **MarkdownDataset** — Text-specific loaders with built-in `TextDocument`/`MarkdownSection`/`MarkdownDocument` DataModels.

**ImageFolderDataset** — Image classification loader.

**HuggingFaceDataset** — Loads HuggingFace Hub datasets; subclasses override `_iter_rows()` for native loaders.

**MLflowDataset** — Loads MLflow dataset artifacts.

## Built-In Benchmarks

~25 standard NLP/reasoning benchmark wrappers under `datasets/built_in/`, each pairing a `HuggingFaceDataset` subclass with small DataModel schemas:

- **ARCAGITask** / `arc_challenge` (multiple choice science)
- **BBH** (Big Bench Hard)
- **BBQ**, **BoolQ**, **DROP**, **GSM8K**, **HellaSwag** (multiple choice)
- **HotpotQA** (QA with document retrieval; variant `HotpotQAHardOnly`)
- **HumanEval** (code generation)
- **IFEval** (instruction-following)
- **LAMBADA**, **LogiQA**, **MMLU**, **SQuAD** (reading comprehension)
- **TruthfulQA**, **WinoGrande** (reasoning)

Out-of-the-box benchmark suite for evaluating/training Synalinks programs.

## Related pages
- [Trainer](../training/trainer.md) — `fit(x=dataset)` consumption
- [Knowledge Bases](./knowledge-bases.md) — `kb.update(dataset)` consumption
