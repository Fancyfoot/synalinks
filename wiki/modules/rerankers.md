# Reranker Modules

## RRFReranker

**Purpose**: Fuse multiple ranked result lists using Reciprocal Rank Fusion (RRF) — combine heterogeneous retrieval types (similarity/fulltext/regex/graph) without normalizing incompatible score scales.

**Signature**:
```
RRFReranker(
    k_rank=60,                # RRF denominator (score = sum of 1/(k + rank))
    k=None,                   # Limit output to top k results (optional)
    id_key=None,              # Key to match rows across lists
    name=None, description=None,
    trainable=False
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Fusion formula**: Each result's final score = Σ 1/(k_rank + rank_in_list_i) across all input lists
- **Ignores None inputs**: Silently handles optional retrieval branches (combines whatever's present)
- **Identity matching**: Rows matched across lists either by `id_key` or canonical JSON signature
- **Common pattern**: Used to combine outputs of `SimilaritySearch` + `FullTextSearch` or any heterogeneous retrievers

## Related pages
- [Retrievers](./retrievers.md) — Retriever modules whose outputs RRFReranker combines
- [Merging Modules](./masking-merging.md) — Or module (|) which also combines outputs
