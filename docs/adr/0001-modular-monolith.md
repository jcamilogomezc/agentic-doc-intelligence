# ADR 0001: Build the system as a modular monolith

- **Status:** Accepted
- **Date:** 2026-11-13
- **Related:** NFR-01, NFR-03, NFR-08, NFR-10; [architecture.md](../architecture.md)

## Context

The system has one user, one developer and one machine: it runs locally on the benchmark laptop described in [environment.md](../environment.md) (NFR-03) with no paid infrastructure (NFR-01). Docker's Linux VM is capped at 6 GB of RAM.

NFR-08 requires extraction, chunking, embedding, retrieval, generation, storage and evaluation to be separate components with their own automated tests. NFR-10 requires a new machine to reach a working UI by following the README.

We need clear boundaries between components without paying the operational cost of running each one as a separate service.

## Decision

We will build one Python package, deployed as a single FastAPI application, with one internal module per component: ingestion, embeddings, storage, retrieval, generation and agents. Modules call each other through Python functions and small, explicit interfaces, never over the network.

Only three things run as separate processes: the Streamlit UI, the PostgreSQL database and Ollama. Evaluation runs as an offline harness on the host.

## Alternatives considered

- **Microservices, one service per component.** Each component could be deployed and scaled independently. Nothing in this project needs that: there is one user and one machine. The cost would be network calls and serialization between components, more containers inside the 6 GB limit, a harder setup (NFR-10), and debugging across several processes.
- **A single unstructured script or notebook.** The fastest way to start, but it fails NFR-08: no boundaries, no isolated tests, and every change risks breaking unrelated behavior.

## Consequences

**Positive:**

- One application to install, run, test and debug. A stack trace shows the whole path of a request across modules.
- Calls between modules are plain function calls: fast, typed, and easy to refactor.
- One test suite and one CI pipeline.
- The memory footprint fits the laptop's limits.

**Negative:**

- Module boundaries are enforced only by discipline. Nothing in Python stops one module from importing another's internals. Mitigation: each module exposes a small public interface, tests go through it, and code review checks imports. An import-checking tool can be added later if boundaries erode.
- Components cannot be scaled or deployed independently.
- A crash or memory leak in one module affects the whole API.
- Long-running agent workflows (Phase 6) share the process with ordinary requests and must not block them.
- Extracting a module into its own service later would take real work, although clear interfaces make it possible.