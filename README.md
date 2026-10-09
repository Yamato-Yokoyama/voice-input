# voice-input

Japanese–English code-switched speech input that actually understands how I talk.

Off-the-shelf dictation breaks when Japanese speech contains English technical terms ("C++のビルドが通らない" comes out as "シープラプラ"). This project builds a dictation tool around whisper.cpp that is measured against my own speech and used every day.

**Status:** in progress (started 2026-10). Results are added weekly under `results/`.

## Architecture

| Component | Language | Role |
|---|---|---|
| `cpp/` | C++17 | Recognition engine wrapping whisper.cpp (RAII, Strategy pattern, streaming), exposed over HTTP |
| `java/` | Java 21, Spring Boot 3 | Service layer: REST API, swappable recognizers via dependency injection, dictionary and LLM correction, Actuator metrics |
| `eval/` | Python | Evaluation harness: MER, CER, B-WER/U-WER, latency p50/p95, RTF |
| `mcp/` | Python | MCP server exposing `transcribe` and `evaluate` to coding agents |

## Evaluation

Standard ASR metrics only; see `docs/EVAL.md`. My own recordings are never committed; CI runs on a small public fixture set.

## Results

(Added weekly.)
