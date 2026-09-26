# Retriever Modules

25 specialized retriever modules for single-purpose knowledge base searches. All share similar structure: an embedded `Generator` turns inputs into a query (optionally LM-inferred and KB-constrained), execute against a `KnowledgeBase` method, return results as `GenericResult` or `KnowledgeGraph`.

## Common Parameters

All retrievers accept:
- `knowledge_base=None` — KnowledgeBase; required
- `language_model=None` — LM to infer query (for LM-driven variants)
- `return_inputs=True` — Include input in output
- `return_query=True` — Include generated query in output
- `k=N` — Number of results (default varies: 10 for similarity/fulltext, 5 for general RetrieveKnowledge)
- `threshold=None` — Similarity/fulltext cutoff
- `ef_search=None` — HNSW recall/speed tradeoff
- `output_format="json"|"csv"` — Result format
- `name=None, description=None`
- `trainable=True` — LM params are trainable

## Table-Level Retrievers (5 variants)

Search a single table across all rows using five strategies:

- **`SimilaritySearch`** — Vector similarity (HNSW), `k=10`, `threshold` supported
- **`FullTextSearch`** — BM25 (Tantivy), `k=10`
- **`RegexSearch`** — RE2 pattern matching on string fields, `k=10`
- **`HybridFTSSearch`** — RRF fusion of `SimilaritySearch` + `FullTextSearch`
- **`HybridRegexSearch`** — RRF fusion of `SimilaritySearch` + `RegexSearch`

**Parameters**: All accept `schema`/`data_model` and `table_name` (**optional** — LM infers per call, constrained to real KB tables, if omitted).

## Entity-Level Retrievers (5 variants)

Search entities (graph nodes) of a single label:

- **`EntitySimilaritySearch`**
- **`EntityFullTextSearch`**
- **`EntityRegexSearch`**
- **`EntityHybridFTSSearch`**
- **`EntityHybridRegexSearch`**

**Parameters**: Accept `label` (**optional** — LM infers per call if omitted).

## Relation-Level Retrievers (5 variants)

Search relations (graph edges) of a single label:

- **`RelationSimilaritySearch`**
- **`RelationFullTextSearch`**
- **`RelationRegexSearch`**
- **`RelationHybridFTSSearch`**
- **`RelationHybridRegexSearch`**

## Path-Level Retrievers (5 variants)

Variable-length path search where **both endpoints match** a given entity (AND semantics on subj/obj):

- **`PathSimilaritySearch`**
- **`PathFullTextSearch`**
- **`PathRegexSearch`**
- **`PathHybridFTSSearch`**
- **`PathHybridRegexSearch`**

**Parameters**: Accept two entity endpoint specs (`subj`/`obj` sides), each with `schema`/`entity_model`/`label` triple.

## Graph-Level Retrievers (2 classes, no variants)

### GlobalGraphSearch

**Purpose**: Theme-centric — return `k` most important pre-built communities by aggregate PageRank as `KnowledgeGraph`s.

**Signature**:
```
GlobalGraphSearch(
    knowledge_base=None,      # Required
    node_labels=None,         # Restrict communities to these node types
    rel_labels=None,          # Restrict to these relation types
    k=10,                     # Number of communities
    members_per_community=None,  # Size limit per community
    return_inputs=True,       # Include input
    name=None, description=None,
    trainable=True
)
```

**Distinctive behavior**:
- **No LM**: Runs no language model at all; returns whole-graph communities
- **Requires pre-built communities**: `KnowledgeBase.build_communities()` must have run; otherwise returns empty
- **Global, not query-seeded**: Ignores input *content* (input is structural only)

### GlobalGraphMapReduce

**Purpose**: LM orchestration on top of `GlobalGraphSearch` — **map**: answer against each community in parallel + score; **reduce**: combine and synthesize final answer.

**Signature**:
```
GlobalGraphMapReduce(
    knowledge_base=None,      # Required
    language_model=None,      # Required
    schema=None, data_model=None,  # Output schema
    score_threshold=0.0,      # Keep answers scoring >= threshold
    map_instructions=None,    # Instructions for per-community LM step
    reduce_instructions=None, # Instructions for final synthesis
    return_community_answers=False,  # Include partial answers in output
    # Plus all GlobalGraphSearch params (node_labels, rel_labels, k, etc.)
    # Plus all standard Trainer/LM params (temperature, max_tokens, ...)
)
```

**Distinctive behavior**:
- **Two-stage LM pipeline**: Full orchestration of map/reduce phases
- **Classic GraphRAG**: Implements the global search pattern from GraphRAG (arXiv:2404.16130)

### LocalGraphSearch

**Purpose**: Entity-centric — vector-seed `k` entities of a label, expand `max_hops` undirected neighborhood, return the deduped union as one `KnowledgeGraph`.

**Signature**:
```
LocalGraphSearch(
    knowledge_base=None,      # Required
    language_model=None,      # LM to infer entity label/query
    schema=None,              # Query/output schema
    entity_model=None,        # Entity DataModel
    label=None,               # (**Optional** — LM infers per call)
    max_hops=2,               # Neighborhood expansion depth
    k=10,                     # Initial entities to seed
    threshold=None,           # Similarity threshold
    rel_label=None,           # Restrict expansion to these relations
    ef_search=None,           # HNSW parameter
    return_inputs=True,
    return_query=True,
    name=None, description=None,
    trainable=True
)
```

**Distinctive behavior**:
- **Entity-centric**: Start from seeded entities; expand via relations
- **Undirected**: Follows both incoming and outgoing edges
- **Counterpart to GlobalGraphSearch**: Local is query-driven; global is structural

## Implementation Notes

- **Shared infrastructure**: `retrievers/_path_helpers.py` (endpoint schema/label resolution), `retrievers/infer_helpers.py` (LM-inferred, KB-enum-constrained `table_name`/`label` fields)
- **Composition**: `retrievers/rrf_reranker.py` (`RRFReranker`) combines outputs from multiple `retrievers/*` modules via Reciprocal Rank Fusion

## Related pages
- [Knowledge Modules](./knowledge-modules.md) — Higher-level RetrieveKnowledge which wraps retrievers
- [Rerankers](./rerankers.md) — RRFReranker for fusing multiple retrieval results
- [Knowledge Bases](../infrastructure/knowledge-bases.md) — KnowledgeBase backing all retrievers
