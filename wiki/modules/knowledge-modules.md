# Knowledge and Retrieval Modules

Modules for embedding knowledge, retrieving from knowledge bases, and querying structured data.

## EmbedKnowledge

**Purpose**: Extract masked fields (including knowledge primitives `Entity`/`Relation`/`KnowledgeGraph`) and attach `embedding` vectors via a single batched embedding-model call.

**Signature**:
```
EmbedKnowledge(
    embedding_model=None,     # EmbeddingModel; required
    in_mask=None,             # Fields to extract
    out_mask=None,            # Fields to keep
    name=None, description=None,
    trainable=False
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Batch efficiency**: Walks nested graph structures (relation endpoints, communities) to gather all embeddings in one round-trip instead of N calls

## RetrieveKnowledge

**Purpose**: LM generates search queries, then retrieves from a `KnowledgeBase` using configurable strategy (similarity/fulltext/regex/hybrid).

**Signature**:
```
RetrieveKnowledge(
    knowledge_base=None,      # KnowledgeBase; required
    language_model=None,      # LM to generate queries
    data_models=None,         # Target table schemas
    search_type="hybrid_fts",  # "similarity"|"fulltext"|"regex"|"hybrid_fts"|"hybrid_regex"
    k=10,                     # Results per search
    similarity_threshold=None,  # Cutoff
    fulltext_threshold=None,  # Cutoff
    k_rank=60,                # RRF parameter
    fields=None,              # Which fields to search
    case_sensitive=True,      # For regex search
    prompt_template=None, examples=None, instructions=None,  # Trainable
    temperature=None, max_tokens=None, ...  # LM parameters
    return_inputs=True,       # Include input
    return_query=True,        # Include generated query
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **LM-driven search**: LM generates the query; constraints enforce it matches KB-known tables/labels
- **Dynamic schema**: Output schema changes for `hybrid_regex` (adds `patterns` field)
- **Flexible strategy**: Swap search types without changing the module

## StampKnowledge

**Purpose**: Add a `created_at` ISO-timestamp field to each input via logical AND.

**Signature**:
```
StampKnowledge(
    name=None, description=None,
    trainable=False
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Trivial but ubiquitous**: Used everywhere for provenance tracking

## UpdateKnowledge

**Purpose**: Insert/upsert arbitrary data (or knowledge-graph primitives like `Entity`/`Relation`/`KnowledgeGraph`) into a `KnowledgeBase`.

**Signature**:
```
UpdateKnowledge(
    knowledge_base=None,      # KnowledgeBase; required
    name=None, description=None,
    trainable=False
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Dispatch ordering**: `KnowledgeGraph` before wrapped `Entities`/`Relations` (since graph node dedup needs them); `Relation` before bare `Entity` (so subj/obj endpoints exist before edges)
- **Upsert semantics**: Uses embedding-based dedup (nearest neighbor lookup) to avoid duplicates

## Text2SQL

**Purpose**: Translate natural language → read-only `SELECT` query, execute against a `KnowledgeBase`, return `{sql_query, result}`.

**Signature**:
```
Text2SQL(
    knowledge_base=None,      # KnowledgeBase with SQL adapter; required
    language_model=None,      # LM to generate SQL
    k=50,                     # Row limit
    output_format="json"|"csv",  # Result format
    prompt_template=None, examples=None, instructions=None,  # Trainable
    temperature=None, max_tokens=None, ...  # LM parameters
    return_inputs=False,      # Include input
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **Live schema**: Schema re-fetched every call (not cached) so DDL changes are picked up
- **Safety net**: Wraps LM's SQL in outer `SELECT * FROM (...) LIMIT k`
- **Safety enforcement**: DuckDB parser validates (read-only enforced at adapter level, not string filtering)

## Text2Cypher

**Purpose**: Graph analogue of `Text2SQL` — natural language → read-only Cypher query, execute against a graph-backed `KnowledgeBase`, return `{cypher_query, result}`.

**Signature**:
```
Text2Cypher(
    knowledge_base=None,      # KnowledgeBase with graph adapter; required
    language_model=None,      # LM to generate Cypher
    k=50,                     # Result limit
    output_format="json"|"csv",  # Result format
    prompt_template=None, examples=None, instructions=None,  # Trainable
    temperature=None, max_tokens=None, ...  # LM parameters
    return_inputs=False,      # Include input
    name=None, description=None,
    trainable=True
)
```

**Inherits**: `Module`.

**Distinctive behavior**:
- **ASCII schema**: Graph schema rendered with ASCII art (`(:Node)-[:REL]->(:Node)`) so LM sees exact syntax
- **Safety**: Enforced by graph adapter (rejects write keywords), not query wrapping (Cypher has no generic subquery-LIMIT)

## Related pages
- [Knowledge Bases](../infrastructure/knowledge-bases.md) — KnowledgeBase used by all these modules
- [Retrievers](./retrievers.md) — Specialized single-search retrievers (more granular than RetrieveKnowledge)
- [Core Modules](./core.md) — Generator which these modules wrap
