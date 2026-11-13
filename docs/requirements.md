# Requirements

## 1. Problem

Clinical, regulatory, and pharmaceutical research teams spend substantial time finding and validating information in long technical PDFs. Conventional keyword search does not reliably provide relevant context or page-level evidence, making research slow and increasing the risk of unsupported conclusions.

## 2. Users

| User | Goal |
| --- | --- |
| Clinical data manager | Find what a regulatory guideline says about a topic, with the exact page to verify it. |
| Regulatory affairs specialist | Compare requirements across public guidance documents and identify relevant differences with source citations. |
| Pharmaceutical research analyst | Search, summarize, and interrogate public drug labels and technical documents without manually reading every page. |

## 3. Use cases

- **UC-01:** As a clinical data manager, I want to ask a question about ICH E6(R3) and get an answer that cites the document and page, so I can check it in the source.
- **UC-02:** As a user, I want to upload a text-based English PDF through the API or simple UI and see whether processing completed successfully, so I know when it is ready for questions.
- **UC-03:** As a regulatory affairs specialist, I want to compare two public drug labels on a topic, so I can identify similarities and differences with supporting citations.
- **UC-04:** As a user, I want the system to explicitly say when the uploaded documents do not contain the answer, so I do not treat an unsupported answer as evidence.
- **UC-05:** As a pharmaceutical research analyst, I want a cited summary of a document, so I can understand its main contents quickly.
- **UC-06:** As a user, I want to inspect the retrieved chunks, configured model, and response latency for a query, so I can audit the system’s behavior.
- **UC-07:** As a user, I want an agent to complete a controlled multi-step research task using approved tools, so I can automate repeatable analyses. *(Phase 6)*
- **UC-08:** As the maintainer, I want to run a repeatable evaluation suite before each release, so I can detect quality regressions.

## 4. Functional requirements

| ID | Requirement | Phase | Use cases |
| --- | --- | --- | --- |
| FR-01 | The system accepts English, text-based PDF uploads of up to 200 pages through an API and a simple UI. | MVP | UC-02 |
| FR-02 | The system extracts text on a page-by-page basis and records the document title, page number, and extraction status. | MVP | UC-02 |
| FR-03 | The system cleans extracted text and splits it into chunks that retain document, title, page, and section-heading metadata. | MVP | UC-01, UC-02 |
| FR-04 | The system generates semantic vector representations for chunks and stores them with their text and metadata in a local persistent data store. | MVP | UC-01, UC-03 |
| FR-05 | The system uses both semantic and lexical search to retrieve and rank relevant document chunks for a question. | MVP | UC-01, UC-03 |
| FR-06 | The system accepts natural-language questions over processed documents. | MVP | UC-01, UC-04 |
| FR-07 | The system generates answers using a locally running language model and the retrieved document content. | MVP | UC-01, UC-04 |
| FR-08 | Each grounded answer displays citations containing the source document, page number, and a supporting text snippet. | MVP | UC-01, UC-03 |
| FR-09 | When retrieved context does not support an answer, the system responds explicitly that the information was not found in the documents. | MVP | UC-04 |
| FR-10 | The user can list uploaded documents and delete a document together with its chunks, vector representations, and metadata. | MVP | UC-02 |
| FR-11 | The system logs each query locally with timestamp, retrieved chunks, configured model, answer or refusal, errors, and latency. | MVP | UC-06 |
| FR-12 | The system generates cited document summaries. | Phase 6 | UC-05 |
| FR-13 | The system compares two selected documents and presents cited similarities and differences. | Phase 6 | UC-03 |
| FR-14 | The system provides controlled agent access to approved tools, including database queries, and logs tool use. | Phase 6 | UC-07 |
| FR-15 | The system runs a repeatable evaluation suite and stores answer-quality and agent-behavior results. | Phase 7 | UC-08 |

## 5. Non-functional requirements

| ID | Quality | Requirement (measurable) |
| --- | --- | --- |
| NFR-01 | Cost | The system must use $0 of paid services or APIs. Embeddings, generation, database storage, and logging must run locally. |
| NFR-02 | Privacy | After installation and model download, the complete upload-and-ask flow must work with the network disconnected. Only public or synthetically generated documents may be processed. |
| NFR-03 | Hardware | The MVP must run locally on the benchmark environment documented in `docs/environment.md`, without cloud infrastructure. Benchmark results must record the machine specification and model configuration. |
| NFR-04 | Latency | **Provisional until Phase 4:** On the benchmark environment, 90% of single-document benchmark questions must return an answer of up to 200 tokens within 20 seconds, measured from submission to the final generated token. Current generation speed is approximately 40 tokens per second, or about 5 seconds for 200 tokens. Phase 4 will measure retrieval and long-prompt processing overhead and revise this target. |
| NFR-05 | Citation precision | In a manually reviewed evaluation set, at least 90% of displayed citations must point to a page that supports the associated answer claim. |
| NFR-06 | Retrieval quality | At least 80% of answerable evaluation questions must contain a correct supporting chunk in the top five retrieved results. |
| NFR-07 | Reproducibility | Given the same document version, configuration, and query, retrieval must return identical ranked results. Evaluation runs must use fixed generation settings, including temperature 0 and a documented seed, to make generated answers as repeatable as possible. |
| NFR-08 | Maintainability | Extraction, chunking, embedding, retrieval, generation, storage, and evaluation must be separate components with automated unit and integration tests for their main success and failure paths. A 20-question smoke-test suite must be run before each MVP release. |
| NFR-09 | Reliability | Corrupt, empty, image-only, or unsupported PDFs must fail with a clear recorded error and must not prevent use of already processed documents. |
| NFR-10 | Reproducible setup | In Phase 8, a new machine with the documented prerequisites must reach a working local UI by following the README. |

## 6. Constraints

- Documents are limited to public documents or synthetically generated documents.
- The system must not be used with confidential information, personal information, patient data, or internal company information.
- The MVP supports English, text-based PDFs only.
- The application is designed for one local user.
- The project has no budget for paid APIs, hosted databases, or cloud infrastructure.
- Initial technical choices are PostgreSQL with pgvector for local storage, `nomic-embed-text` for embeddings, and `qwen3:4b` for answer generation. Changes to these choices must be recorded in an architecture decision record.
- The initial target domain is technical, regulatory, and pharmaceutical documentation.

## 7. Scope

- **MVP (end of Phase 5):** A local, single-user application with an API and simple UI for uploading English text-based PDFs of up to 200 pages. It extracts, cleans, chunks, stores, and retrieves document content; answers questions with page-level citations or an explicit “not found in the documents” response; lists and deletes documents; and logs every query with retrieved chunks, configured model, and latency.

- **After the MVP:** Cited document summaries, two-document comparisons, and agents with controlled tools in Phase 6; full answer-quality and agent-behavior evaluation in Phase 7; and containerized deployment with continuous integration in Phase 8.

- **Not planned unless time allows:** OCR for scanned PDFs, understanding tables or figures, non-English documents, multiple users, and cloud deployment.

## 8. Initial success targets

- **NFR-06:** Retrieval recall must meet the defined target.
- **Citation coverage:** At least 90% of factual claims in supported answers have at least one supporting citation.
- At least 90% of unanswerable evaluation questions receive an explicit insufficient-evidence response rather than an invented answer.
- At least 90% of valid PDF uploads of 200 pages or fewer complete indexing without manual intervention.
- At least 18 of 20 smoke-test questions produce the expected supported answer, supported refusal, or expected error behavior.
- The provisional latency target in NFR-04 is measured and revised in Phase 4.

## 9. Open questions

- How much time do hybrid retrieval and long-context prompt processing add to end-to-end latency?
- Is a reasoning or “thinking” mode worth its additional latency for grounded answers? This will be decided during Phase 4.
- What chunk size and overlap best preserve retrieval and page-level citation accuracy for regulatory and pharmaceutical PDFs?
- How should the system handle tables, figures, footnotes, and references within otherwise text-based PDFs?
- What questions and expected evidence should make up the evaluation set beyond the initial 20-question smoke test?
- What agent tools and database actions are safe and valuable enough to include in Phase 6?