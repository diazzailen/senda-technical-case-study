# Candidate code sample: selecting a retrieval strategy

**Status: proposed for manual approval; not published.** Source: `apps/api/src/modules/educational-content/features/search-official-sources/search-official-sources.use-case.ts`. This presentation reproduces the **entire 44-line source file verbatim**. No lines are omitted, redacted, reordered or modified; the headings and explanation outside the code block are new portfolio prose. The original application file is unchanged.

## Portfolio presentation

In SENDA's NestJS backend, this use case chooses how to retrieve educational source chunks. It uses injected ports for storage and embeddings, allowing retrieval policy to remain separate from the Prisma/pgvector and HTTP implementations. If eligible chunks have embeddings, it embeds the question and requests similarity search; otherwise, it uses the repository's keyword search. The small, explicit branch makes the behavior testable without starting the embedding service or a database.

```ts
import { Inject, Injectable } from "@nestjs/common";
import {
  DOCUMENT_CHUNK_REPOSITORY,
  EMBEDDING_PROVIDER,
} from "../../educational-content.tokens";
import type { DocumentChunkRepository } from "../../ports/document-chunk.repository";
import type { EmbeddingProviderPort } from "../../ports/embedding-provider.port";
import type {
  RetrievedOfficialSource,
  SearchOfficialSourcesQuery,
} from "./search-official-sources.query";

@Injectable()
export class SearchOfficialSourcesUseCase {
  private readonly minimumSimilarity = Number(
    process.env.SEMANTIC_RETRIEVAL_MIN_SIMILARITY ?? "0.33",
  );

  constructor(
    @Inject(DOCUMENT_CHUNK_REPOSITORY)
    private readonly chunks: DocumentChunkRepository,
    @Inject(EMBEDDING_PROVIDER)
    private readonly embeddings: EmbeddingProviderPort,
  ) {}

  async execute(
    query: SearchOfficialSourcesQuery,
  ): Promise<RetrievedOfficialSource[]> {
    if (!(await this.chunks.hasSearchableEmbeddings(query))) {
      return this.chunks.searchOfficialSources(query);
    }

    const embedding = await this.embeddings.embedQuery({
      text: query.query,
      locale: query.locale,
    });

    return this.chunks.searchOfficialSourcesByEmbedding(
      query,
      embedding,
      this.minimumSimilarity,
    );
  }
}
```

**What this demonstrates:** dependency injection against interfaces, a single-purpose application use case, and a clear retrieval decision with infrastructure-specific work behind ports. The companion unit test (`search-official-sources.use-case.spec.ts`) checks the semantic branch, the no-index keyword branch, locale forwarding and the deliberate absence of keyword fallback after zero semantic results. This file itself does **not** implement vector ranking, source approval, input validation or end-to-end RAG generation.

**Minimum context for a Tech Lead:** `SearchOfficialSourcesQuery` carries question text, locale, country and result limit; `RetrievedOfficialSource` carries chunk content and source metadata. `DocumentChunkRepository` exposes the eligible-embedding check and keyword/vector searches; `EmbeddingProviderPort` embeds the query. NestJS injection tokens bind the interfaces to adapters. In the actual repository adapter, eligible vectors are associated with active document versions matching locale/country, and the vector search applies a minimum-similarity cutoff. These interfaces/adapters are described here, **not copied** into this candidate.

## Review for publication

**Assessment:** Strong, appropriately sized backend sample. It shows orchestration and boundaries rather than claiming the entire RAG implementation. At 44 lines, the complete file is preferable to a shorter excerpt: its imports and constructor show how the ports are wired, and the whole decision fits in one screen. A shorter excerpt would need explicit omission markers and a prose explanation of the missing DI/type context. The full file can reasonably be shared **if the creator approves this exact text and its disclosure**; this document is not that approval.

**Sensitive information:** No credentials, access tokens, personal/patient data, internal hostnames or private source content appear in the file. `SEMANTIC_RETRIEVAL_MIN_SIMILARITY` and its default `0.33` are a tuning detail, not a secret. Module names, relative imports and the branch behavior reveal limited implementation structure; confirm that disclosing these and the threshold is acceptable. Confirm ownership and redistribution rights before placing even a short private-repository file in a public portfolio.

**Trade-off worth stating accurately:** The keyword path is chosen **only when no eligible embeddings exist**. If some eligible chunks are indexed, unembedded chunks may be missed; zero vector hits or an embedding-provider failure do not trigger keyword fallback. The threshold is configurable but this excerpt does not validate its value. Unit tests demonstrate branch behavior with fakes, not production retrieval quality or clinical correctness. Any portfolio caption should keep those distinctions.
