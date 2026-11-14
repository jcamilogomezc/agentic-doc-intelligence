# ADR 0004: Load all model layers onto the GPU (num_gpu 99)

- **Status:** Accepted, to be re-validated in Phase 4 with long prompts
- **Date:** 2026-11-13
- **Related:** NFR-03, NFR-04; [ADR 0003](0003-native-ollama.md); measurements in [environment.md](../environment.md)

## Context

With a 4K-token context, `qwen3:4b` needs about 3.2 GB of memory. On the benchmark laptop's 4 GB GTX 1650, Ollama's default placement put 34% of the model on the CPU, and generation ran at 16.7 to 17.2 tokens per second. At the same time, `nvidia-smi` reported about 2.5 GB in use, leaving roughly 1.5 GB free. That suggested Ollama's memory estimate was more cautious than necessary.

Experiments on 2026-11-10 changed one setting at a time and compared eval rate rather than total duration, because answer lengths varied:

| Change | Placement | Eval rate (tokens/s) | Result |
| --- | --- | --- | --- |
| Ollama defaults | 34% CPU / 66% GPU | 16.7 | Baseline |
| GPU-using applications closed | 34% CPU / 66% GPU | 17.2 | No meaningful change |
| Flash attention and 8-bit KV cache | 28% CPU / 72% GPU | 13.6 | Slower; reverted |
| `num_gpu 99` (all layers on the GPU) | 100% GPU | 39.5 | About 2.3 times faster |

Repeat runs with all layers on the GPU measured 36.0 to 36.8 tokens per second.

## Decision

We will load the answer model with every layer on the GPU by sending `num_gpu: 99` with each request. The value comes from the application's configuration, so it can differ between machines. Flash attention and the KV cache type stay at Ollama's defaults.

## Alternatives considered

- **Ollama's default placement.** Safe, but about 2.3 times slower.
- **Flash attention with an 8-bit KV cache.** Moved more of the model onto the GPU but made generation about 20% slower on this card, which lacks tensor cores. Rejected.
- **A smaller model that fits by default, such as `llama3.2:3b`.** Possibly faster, but likely lower answer quality. That trade-off needs evaluation on real questions, so it is deferred to Phase 4. If a smaller model wins, a new ADR will supersede this one.
- **A shorter context window (2K tokens).** Frees memory, but leaves less room for retrieved passages, which hurts answer quality and citations.

## Consequences

**Positive:**

- About 2.3 times faster generation (roughly 17 to 37–40 tokens per second), which makes the provisional latency target in NFR-04 realistic.
- The decision rests on recorded measurements, not on defaults or assumptions.

**Negative:**

- Very little VRAM headroom remains. If another program takes GPU memory while the model is loaded, the NVIDIA driver on Windows may silently use system RAM instead, with a large slowdown and no error. Detection: watch for a sudden drop in eval rate in the query log (FR-11), then check `nvidia-smi`.
- The setting is specific to this hardware. On another GPU it may be unnecessary or prevent the model from loading, so it must stay configurable.
- It was validated only with short prompts and single runs. The 2.3-times gap is far larger than normal run-to-run variation, but Phase 4 must confirm the behavior with realistic 2,000 to 3,000-token RAG prompts.