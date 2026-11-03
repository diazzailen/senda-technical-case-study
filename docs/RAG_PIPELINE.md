# Educational RAG pipeline

SENDA uses a single retrieval-and-generation path: **versioned content → chunk embeddings → source search → context-bound generation → references or abstention**. It is not a multi-agent workflow.

## Indexing and retrieval

1. The API stores document metadata and versions with language, status, publication date and content hash. A deterministic paragraph-oriented chunker creates linked chunks and normalized keyword-search text.
2. Index/backfill is a separate operation: a FastAPI Sentence Transformers service encodes chunks as normalized 384-dimensional vectors; the NestJS adapter validates the shape and stores them in PostgreSQL/pgvector. Version creation does not automatically embed its chunks.
3. On an authenticated educational question, NestJS trims and bounds input (up to 2,000 characters) and sends an educational request to FastAPI. FastAPI requests up to five matching sources from the API for the supplied locale and France country scope.
4. If eligible active-version embeddings exist, NestJS embeds the question and uses cosine similarity with a configurable threshold. Otherwise it uses a deterministic keyword-match heuristic. **There is no keyword fallback for zero vector hits or an embedding-service error.** Returned records carry chunk text and document/version metadata.

## Generation and answer handling

5. Before retrieval, the AI use case can short-circuit a fixed list of medical-related terms to a care-team message. This is an explicitly development-only heuristic, not medical-intent classification.
6. With retrieved sources, a prompt asks Mistral to answer only from supplied context, treat source text as evidence rather than instructions, and report insufficient evidence when necessary. The provider returns structured JSON: answer, cited source IDs and an insufficiency flag.
7. The adapter validates the result and rejects unknown or repeated source IDs. The use case turns cited chunk IDs into document/version source references; the mobile UI displays titles, organizations and optional sections/dates. A no-source or model-declared insufficient-evidence response has no educational answer. Provider failures are handled as technical errors.

## Why this shape, and what it can demonstrate

Keeping retrieval behind NestJS preserves knowledge-base ownership; version/chunk IDs support traceability; response validation makes no-evidence behavior testable. Retrieval tests cover both semantic and keyword branches; synthetic fixture cases calculate precision@k/recall@k. Python tests and a deterministic fake-dependent evaluator check routing, source mapping and abstention. None of those checks establishes medical correctness or field performance. Partial indexing can omit unembedded content, and valid citations do not prove every generated statement follows from the source.

Related: [architecture](ARCHITECTURE.md) · [technical decisions](ENGINEERING_DECISIONS.md).
