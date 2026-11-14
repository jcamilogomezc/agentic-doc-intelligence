# ADR 0002: Store documents, chunks and vectors in PostgreSQL with pgvector

- **Status:** Accepted
- **Date:** 2026-11-13
- **Related:** FR-04, FR-05, FR-10, FR-11, FR-14, NFR-01, NFR-07; [ADR 0001](0001-modular-monolith.md)

## Context

The system must store chunks together with their vectors and metadata (FR-04), retrieve them with both semantic and lexical search (FR-05), and delete a document together with its chunks, vectors and metadata (FR-10). It also logs every query (FR-11), and Phase 6 agents will run approved, read-only queries over document metadata (FR-14).

Retrieval must be reproducible for evaluation (NFR-07), and everything must run locally at no cost (NFR-01). The corpus is small: 20 to 30 documents, so thousands to tens of thousands of chunks. Docker's Linux VM is limited to 6 GB of RAM.

## Decision

We will use PostgreSQL 17 with the pgvector extension (Docker image `pgvector/pgvector:pg17`) as the single data store:

- Relational tables for documents, chunks, ingestion runs and the query log.
- A 768-dimension vector column for chunk embeddings, with an HNSW index for similarity search.
- A full-text search column with a GIN index for lexical search.

Application code accesses the database through SQLAlchemy 2.0, and the schema evolves only through Alembic migrations.

## Alternatives considered

- **Chroma for vectors, plus a relational database for everything else.** Chroma has a simple API, but two stores cannot share a transaction. Deleting a document (FR-10) would mean coordinating two systems, with a risk of orphaned vectors if one step fails. Lexical search would need a third component.
- **FAISS.** Very fast in-memory similarity search, but it is a library, not a database: no persistence, metadata, transactions or full-text search. We would have to build storage around it.
- **A dedicated vector database server (Qdrant, Weaviate, Milvus).** Strong vector features, and some include hybrid search. But each is another service to run inside the 6 GB limit and another technology to learn, and their advantages appear at scales this project will not reach.

## Consequences

**Positive:**

- One transaction covers a document, its chunks and their vectors. Deleting a document can cascade to everything derived from it, which satisfies FR-10 without coordination code.
- Hybrid search runs inside one database engine: vector similarity and full-text search can be combined in SQL.
- Metadata filters, the query log and agent queries all use standard SQL.
- Mature tooling for migrations, backups and inspection (psql, DBeaver), and only one data service to run.

**Negative:**

- pgvector is not a specialized vector engine. At millions of vectors, a dedicated database would likely be faster and easier to tune.
- HNSW search is approximate, and building the index takes memory and time. For NFR-07, index parameters must be fixed and recorded. Because the corpus is small, evaluation can also run exact search to guarantee identical results.
- The vector dimension is fixed in the schema. Changing the embedding model requires a migration and re-embedding every chunk (this is true of any vector store).
- The project depends on relational modelling and SQL skills, which take time to learn.