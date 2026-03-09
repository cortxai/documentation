# CortX Development Plan (Alpha Roadmap)

**Project:** CortX  
**Organisation:** cortxai  
**Current Version:** v0.5.0  
**Status:** Alpha

---

# Purpose of this Document

This document provides a **structured implementation roadmap** for CortX
that can be ingested by an LLM coding assistant. It explains:

* The **current architectural state**
* The **next implementation phase (v0.5.X)**
* A **versioned roadmap** for upcoming phases
* **Architectural constraints** that must remain true
* **Non-goals** (features intentionally deferred)

The goal is to guide development toward the long‑term vision of CortX as
a **local‑first runtime platform for intelligent systems**.

---

# Architectural Vision

CortX is evolving from a proof‑of‑concept **local agentic system** into
a **platform runtime for intelligent software systems**.

The long‑term architecture consists of three conceptual layers:

1. **CortX Runtime**

   * Core execution environment
   * Module loading
   * Pipeline orchestration
   * Event system
2. **CortX Modules**

   * Extensions that implement platform capabilities
   * Classifiers
   * Workers
   * Routers
   * Model providers
   * Tools
   * Infrastructure integrations
3. **CortX Distributions**

   * Opinionated system builds
   * Provide a complete runnable platform
   * Example: `cortx`

---

# Architectural Rules (Must Never Be Violated)

1. **Core must remain small** — Runtime contains primitives only
2. **Everything must be replaceable** — Models, tools, classifiers, workers, routers
3. **Modules extend the runtime** — Runtime must never depend on modules
4. **Intelligence must be explicit pipelines** — Avoid opaque agent frameworks
5. **Distributions are separate from the runtime**

---

# Current Implementation Summary (v0.4.4 Pipeline System)

* COREtex runtime with `coretex/` (runtime, interfaces, registries, executor, pipeline, loader, context, events, config)
* `modules/` implementing classifier_basic, router_simple, worker_llm, tools_filesystem, model_provider_ollama
* `distributions/cortx/` with FastAPI endpoints and module bootstrap
* **Pipeline System:** `PipelineStep` and `PipelineDefinition` dataclasses with runtime validation; `PipelineRegistry` with pipeline-specific error messages; `PipelineRunner` accepts `PipelineDefinition`; `make_default_pipeline()` factory; `event=pipeline_selected` observability logging; default pipeline registered in bootstrap; 129 tests

---

# Versioning Strategy

Version numbers follow the format:

0.N.X

Where:

* N = **major architectural changes**
* X = **minor improvements**

Examples:

* v0.3.0 → major architectural change
* v0.3.1 → minor improvements or fixes

Version **v1.0.0** marks the end of the alpha/beta period.

---

# v0.4.0 — Pipeline System ✅ COMPLETED

## Summary

Configurable execution pipelines were introduced. The hardcoded pipeline was replaced by a structured, modular pipeline system.

### What was delivered

* `PipelineStep` dataclass — `component_type` (validated) + `name`
* `PipelineDefinition` dataclass — `name` + ordered `steps` list + `get_step()`
* `make_default_pipeline()` factory — returns 4-step default pipeline
* `PipelineRegistry` — pipeline-specific error messages (`"Pipeline already registered: <name>"`, `"Unknown pipeline: <name>"`)
* `PipelineRunner` refactor — accepts `PipelineDefinition`, emits `event=pipeline_selected pipeline=<name>`
* Bootstrap updated — `pipeline_registry` singleton, default pipeline registered
* `distributions/cortx/main.py` updated — pipeline retrieved from registry
* Test suite expanded: 106 → 129 tests
* Documentation updated: README, IMPLEMENTATION, TESTING, DEVELOPMENT

---

# v0.5.0 — Model Provider System

## Goal

Introduce a **formal `ModelProvider` abstraction** so that the classifier and worker can be configured to use different models or backends. Currently both components call Ollama directly using hardcoded client logic. The v0.5.0 phase will route all model inference through registered `ModelProvider` instances, enabling hybrid model strategies (e.g., different models for classifier vs worker, or future OpenAI/Anthropic support) without any change to pipeline logic.

---

## Implementation Plan

### 1 — ModelProvider Interface

File: `coretex/interfaces/model_provider.py`

```python
class ModelProvider(ABC):
    @abstractmethod
    async def generate(self, model: str, prompt: str, **kwargs) -> str: ...

    @abstractmethod
    async def chat(self, model: str, messages: list, **kwargs) -> str: ...
```

---

### 2 — ModelProviderRegistry Enhancements

File: `coretex/registry/model_registry.py`

Tasks:

* Ensure `ModelProviderRegistry` is a fully validated registry.
* Support `register(name: str, provider: ModelProvider)` and `get(name: str) -> ModelProvider`.
* Enforce:
  - Duplicate registration raises `ValueError("Model provider already registered: <name>")`
  - Unknown lookup raises `ValueError("Unknown model provider: <name>")` and logs `event=registry_lookup_failed`
* Add `list()` to enumerate registered providers.
* Update error messages to be model-provider-specific (parallel to v0.4.0 PipelineRegistry change).

---

### 3 — OllamaProvider Module

File: `modules/model_provider_ollama/provider.py`

* Implement `OllamaProvider` fully implementing the `ModelProvider` interface:
  - `generate(model, prompt, **kwargs)` — calls `POST /api/generate` with `stream=False`, returns response text
  - `chat(model, messages, **kwargs)` — calls `POST /api/chat` with `stream=False`, returns last message content
* Configure timeouts from `settings.classifier_timeout` / `settings.worker_timeout`
* Log `event=model_provider_generate_start` / `event=model_provider_generate_complete` with `duration_ms`

---

### 4 — Classifier Integration

File: `modules/classifier_basic/classifier.py`

* Remove direct `httpx` client calls from `ClassifierBasic`
* Inject `model_provider: ModelProvider` at construction or lookup by name from `ModelProviderRegistry`
* Route all LLM calls through:
```python
await model_provider.chat(model=settings.classifier_model, messages=[...])
```
* Preserve all existing retry logic, fallback, and structured logging

---

### 5 — Worker Integration

File: `modules/worker_llm/worker.py`

* Remove direct `httpx` client calls from `WorkerLLM`
* Inject `model_provider: ModelProvider` at construction or lookup from `ModelProviderRegistry`
* Route all LLM calls through:
```python
await model_provider.generate(model=settings.worker_model, prompt=...)
```
* Preserve existing intent-aware prompts and JSON action envelope behaviour

---

### 6 — Module Registration

Files:

```
modules/classifier_basic/module.py
modules/worker_llm/module.py
modules/model_provider_ollama/module.py
```

* `model_provider_ollama/module.py` — register `OllamaProvider()` as `"ollama"` in `model_registry`
* `classifier_basic/module.py` — pass model provider name or instance when creating `ClassifierBasic`
* `worker_llm/module.py` — pass model provider name or instance when creating `WorkerLLM`

---

### 7 — Backward Compatibility

* Ensure `POST /ingest` and OpenWebUI continue working with default Ollama configuration
* Default model provider is `"ollama"` — no configuration change required for existing deployments
* Environment variables `CLASSIFIER_MODEL` and `WORKER_MODEL` continue to control model selection

---

### 8 — Observability

* Log model provider name in classifier and worker events:
  - `event=classifier_start classifier=<name> model_provider=<name>`
  - `event=worker_start worker=<name> model_provider=<name>`
* Provider-level events:
  - `event=model_provider_generate_start`
  - `event=model_provider_generate_complete duration_ms=<int>`

---

### 9 — Tests

* Add unit tests for `ModelProviderRegistry` (duplicate, unknown lookup, list)
* Add unit tests for `OllamaProvider` (mocked httpx calls)
* Add integration tests confirming classifier and worker use injected provider
* Verify model provider events are emitted in full pipeline run
* Ensure backward compatibility with default pipeline and Ollama
* Update test count target: 150+ tests

---

# High-Level Roadmap After v0.5.X

## v0.6 — Distribution Layer

* Introduce first **CortX distributions** (e.g., `cortx`)
* Assemble runtime + modules + FastAPI + OpenWebUI
* Support bootstrapped module loading

## v0.7 — Event System

* Runtime-wide **event bus**
* Every important action emits structured events
* Enable debugging, observability, metrics, future monitoring integrations

## v0.8 — Move Tool System into Modules

* Refactor tool system into modules
* Example: `tools_filesystem`, `tools_shell`, `tools_http`, `tools_git`
* Users can add, disable, or create third-party tools
* Completes **core modular architecture**

---

# Non-Goals (v0.5.x)

* Memory / conversation history
* Multi-agent coordination
* Task Graph / Planner orchestration
* Authentication / authorization
* Async tool execution
* Additional model providers beyond Ollama (foundation only, not implementations)
* Distributed runtime or inference

---

# Acceptance Criteria for v0.5.X

* `ModelProvider` interface is complete and documented
* `ModelProviderRegistry` has model-provider-specific error messages
* `OllamaProvider` fully implements `ModelProvider` interface with observability events
* Classifier and worker route all LLM calls through `ModelProvider`
* Full structured logging preserved, including model_provider name in step events
* Default pipeline behaves identically to pre-v0.5.0 pipeline
* Test suite expanded to 150+ tests
* No regressions in /ingest or OpenWebUI endpoints

---

# Expected Outcome

After v0.5.X, COREtex will:

* Support **swappable model backends** through the `ModelProvider` abstraction
* Enable hybrid model strategies (different providers for classifier and worker)
* Maintain full determinism and observability
* Provide a foundation for multi-provider inference in future phases
