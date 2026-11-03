# Engineering decisions and trade-offs

These decisions describe implemented behavior, with trade-offs relevant to a backend or applied-AI review.

| Decision | Why it helps | Trade-off visible in the implementation |
| --- | --- | --- |
| **Domain-oriented NestJS modules with ports/adapters** for identity, patient journeys, educational content and AI assistance | Separates controllers and use cases from Prisma and outbound HTTP; keeps ownership and tests focused | Several service/repository interfaces and HTTP hops to maintain |
| **Python services for generation and embeddings; NestJS owns retrieval** | Model-specific dependencies live in Python while the application API retains document/search responsibilities | Cross-service availability and timeout behavior matter |
| **Documents → versions → chunks** in PostgreSQL | Connects returned evidence to language, publication date and source metadata rather than anonymous text | A version status is not a clinical approval workflow; indexing is a separate step |
| **pgvector cosine search with an unindexed-corpus keyword path** | Enables semantic matching while allowing local use before vectors are populated | If *any* eligible embeddings exist, semantic search is used; partial indexing can reduce coverage |
| **Structured generation with source-ID checks and abstention** | Makes references and no-evidence responses explicit and testable | Citations indicate model-selected context, not claim-level verification |
| **Case-level owner and delegated permissions** | Keeps shared journey access decisions on the backend rather than only in the UI | More authorization states to model and test than a single global role |
| **Synthetic technical evaluations** | Supports reproducible retrieval and AI-control regressions | Does not measure clinical quality, live provider quality or patient outcomes |

For implementation-level details, see [architecture](ARCHITECTURE.md), [RAG pipeline](RAG_PIPELINE.md) and [proposed excerpts](../examples/retrieval-strategy.md).
