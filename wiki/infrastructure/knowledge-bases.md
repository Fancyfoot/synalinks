# Knowledge Bases (SQL/Graph/Vector Storage)

## KnowledgeBase

**Purpose**: Unified façade over **two orthogonal stores**: SQL (DuckDB/LanceDB) and property graph (Ladybug).

**Constructor**:
```
KnowledgeBase(
    uri=None,                 # SQL database URI (duckdb:///file.db or lancedb://)
    graph_uri=None,           # Graph database URI (ladybug://)
    data_models=None,         # SQL table schemas
    entity_models=None,       # Graph node schemas
    relation_models=None,     # Graph edge schemas
    embedding_model=None,     # For vector search
    metric="cosine",          # Vector distance metric
    wipe_on_start=False,      # Clear DBs on init
    encryption_key=None,      # Deliberately not stored on self
    name=None
)
```

**SQL Methods**: `update()`, `from_csv/json/parquet/jsonl()`, `rename`, `get/getall/delete/drop_table`, `sql()` (read-only), `similarity_search`, `fulltext_search`, `regex_search`, `hybrid_fts/regex_search`.

**Graph Methods** (if `graph_uri` provided): `update_entities/update_relations`, `get_entity/delete_entity`, `cypher()` (read-only), parallel search variants, `detect_communities`, `pagerank`, `local_graph_search`, `global_graph_search`.

**Distinctive**:
- **Live schema**: `from_csv` etc. are **native bulk loaders** bypassing Python row pipeline — two orders of magnitude faster
- **Auto-pairing**: Both `uri` and `graph_uri` omitted → auto-creates both under `synalinks_home()`
- **Primary key convention**: First declared field (skipping `label`/`subj`/`obj`)

## DatabaseAdapter

**Purpose**: Backend interface for SQL storage.

**Implementations**:
- **DuckDBAdapter** — Default; uses DuckDB VSS (HNSW), Tantivy (BM25), RE2 (regex), parser-enforced read-only `sql(read_only=True)`
- **LanceDBAdapter** — Vector-native columnar alternative (Lance files); mirrors DuckDB surface; uses LanceDB Tantivy + DuckDB as SQL engine

## GraphDatabaseAdapter

**Purpose**: Backend interface for graph storage.

**Implementations**:
- **LadybugAdapter** — Cypher graph DB (Ladybug); supports schema constraints, dedup via nearest-neighbor lookup, resilient extension install with timeout/cache

## Related pages
- [Knowledge Modules](../modules/knowledge-modules.md) — Modules wrapping KB operations
- [Retrievers](../modules/retrievers.md) — Specialized searchers (25 variants)
- [Data Model Hierarchy](../core/data-model-hierarchy.md) — Entity/Relation/KnowledgeGraph primitives
