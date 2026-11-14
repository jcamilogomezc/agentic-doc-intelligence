# Development environment

Recorded: 2026-11-10

## Hardware
- OS: Windows 11 (build 26300)
- CPU: Intel Core i5, 9th gen
- RAM: 16 GB (WSL 2 capped at 6 GB)
- GPU: NVIDIA GeForce GTX 1650, 4 GB, driver 581.29

## Tools
- Git: 2.56.0
- VS Code: 1.145.3
- uv: 0.12.17
- Python (via uv): 3.12.14
- WSL: 3.0.5.0
- Docker Engine: 29.9.3
- Ollama: 0.40.15
- Docker Desktop: 4.99.0

## Local models
| Model | Download Size | Size in memory |Processor (ollama ps) |
| --- | --- | --- | --- |
| qwen3:4b | 2.5 GB | 3.2 GB | 100% GPU |
| nomic-embed-text | 274 MB | 335 MB | 100% GPU |

## qwen3:4b measurements (same question, 4K context)

| Run | Setting | Placement (ollama ps) | Eval count | Eval rate (tokens/s) | Notes |
| --- | --- | --- | --- | --- | --- |
| Baseline, thinking off | Ollama defaults | 34% CPU / 66% GPU | 254 | 16.7 | |
| Free VRAM, thinking off | GPU apps closed | 34% CPU / 66% GPU | 306 | 17.2 | |
| 8-bit KV cache, thinking off | flash attention + q8_0 KV cache | 28% CPU / 72% GPU | 283 | 13.6 | |
| All layers on GPU, thinking off | num_gpu 99 | 100% GPU | 338 | 39.5 | |
| All layers on GPU, thinking on (first try) | num_gpu 99 | 100% GPU | 299 | 39.6 | Invalid: thinking did not appear to run |
| All layers on GPU, thinking on (repeat) | num_gpu 99 | 100% GPU | 439 | 36.0 | Reasoning shown separately |
| All layers on GPU, thinking off (repeat) | num_gpu 99 | 100% GPU | 309 | 36.8 | Model still reasoned; text leaked into answer with a stray </think> |

Decision: use num_gpu 99 (2.3× faster than the default placement).
Flash attention and the 8-bit KV cache were reverted because they made generation slower on this GPU.
Thinking mode: decide in Phase 4. It costs about 40% more tokens, and turning it off can leak reasoning into the answer.

## Settings
- UV_CACHE_DIR = D:\dev\.uv-cache
- OLLAMA_MODELS = D:\dev\ollama-models
- Conda base auto-activation disabled
- qwen3:4b runs with num_gpu 99.