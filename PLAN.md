# CortX Development Plan (Alpha Roadmap)

**Project:** CortX
**Organisation:** cortxai
**Current Version:** v0.4.0
**Status:** Alpha

---

# Purpose of this Document

This document provides a **structured implementation roadmap** for CortX
that can be ingested by an LLM coding assistant. It explains:

* The **current architectural state**
* The **next implementation phase (v0.4.0)**
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
   * Example: `cortx_local`

---

# Architectural Rules (Must Never Be Violated)

1. **Core must remain small** — Runtime contains primitives only
2. **Everything must be replaceable** — Models, tools, classifiers, workers, routers
3. **Modules extend the runtime** — Runtime must never depend on modules
4. **Intelligence must be explicit pipelines** — Avoid opaque agent frameworks
5. **Distributions are separate from the runtime**

---

# Current Implementation Summary (v0.3.x Stabilisation)

* COREtex runtime with `coretex/` (runtime, interfaces, registries, executor, pipeline, loader, context, events, config)
* `modules/` implementing classifier_basic, router_simple, worker_llm, tools_filesystem, model_provider_ollama
* `distributions/cortx/` with FastAPI endpoints and module bootstrap
* **Pipeline:** User Input → Classifier → Router → Worker → ToolExecutor
* Fully modular runtime, deterministic two-stage LLM calls, structured logging, ModuleLoader validation, registry safety, 106 tests, documentation

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

# v0.4.0 --- Pipeline System

## Goal

Introduce **configurable execution pipelines**. The current hardcoded pipeline will be replaced by a structured, modular pipeline system that allows future pipelines to include planners, evaluators, or alternative tool graphs without modifying the runtime core.

---

## Implementation Plan

### 1 — PipelineRegistry Enhancements

File:

```
coretex/registry/pipeline_registry.py
```

Tasks:

1. Implement `PipelineRegistry` as a fully validated registry.
2. Support `register(name: str, pipeline: PipelineDefinition)` and `get(name: str) -> PipelineDefinition`.
3. Enforce:

   * Duplicate registration raises `ValueError("Pipeline already registered: <name>")`
   * Unknown lookup raises `ValueError("Unknown pipeline: <name>")` and logs `event=registry_lookup_failed`
4. Add `list()` to enumerate registered pipelines.

### 2 — Pipeline Definition Objects

Create `PipelineDefinition` dataclass in `coretex/runtime/pipeline.py`:

```python
@dataclass
class PipelineStep:
    component_type: Literal['classifier','router','worker','tool_executor']
    name: str  # Name in the registry

@dataclass
class PipelineDefinition:
    steps: list[PipelineStep]
```

Requirements:

* Must support validation of step types and existence in registry.
* Runtime will raise descriptive error on invalid steps.

### 3 — PipelineRunner Refactor

File:

```
coretex/runtime/pipeline.py
```

Changes:

* Accept `pipeline: PipelineDefinition` as argument.
* Dynamically iterate through `steps`, executing each by type:

  * Classifier → retrieve from ModuleRegistry
  * Router → retrieve from ModuleRegistry
  * Worker → retrieve from ModuleRegistry
  * ToolExecutor → always same executor
* Preserve structured logging and failure categories for each step.
* Maintain existing deterministic behaviour.

### 4 — Backward Compatibility

* Provide a default `default_pipeline` matching existing behaviour:

  ```
  User Input → ClassifierBasic → RouterSimple → WorkerLLM → ToolExecutor
  ```
* Ensure `POST /ingest` and OpenWebUI continue working with default pipeline.

### 5 — Observability

* Log pipeline name: `event=pipeline_selected pipeline=<name>`
* Step-level logging remains: `event=step_start`, `event=step_complete` with duration_ms
* Preserve existing latency metrics for classifier, worker, total pipeline

### 6 — Tests

* Add unit tests for PipelineRegistry (duplicate, unknown lookup)
* PipelineRunner tests with multiple pipeline definitions
* Verify full log lifecycle for custom pipelines
* Ensure backward compatibility with default pipeline
* Update test count target: 120+ tests

---

# High-Level Roadmap After v0.4.0

## v0.4.X — Model Provider System

* Introduce `ModelProvider` abstraction
* Implement `model_provider_ollama` module
* Enable hybrid model strategies (future: OpenAI, Anthropic, llama.cpp)
* Maintain backward-compatible worker behaviour

## v0.5.0 — Distribution Layer

* Introduce first **CortX distributions** (e.g., `cortx_local`)
* Assemble runtime + modules + FastAPI + OpenWebUI
* Support bootstrapped module loading

## v0.6.0 — Event System

* Runtime-wide **event bus**
* Every important action emits structured events
* Enable debugging, observability, metrics, future monitoring integrations

## v0.7.0 — Move Tool System into Modules

* Refactor tool system into modules
* Example: `tools_filesystem`, `tools_shell`, `tools_http`, `tools_git`
* Users can add, disable, or create third-party tools
* Completes **core modular architecture**

---

# Non-Goals (v0.4.x)

* Memory / conversation history
* Multi-agent coordination
* Task Graph / Planner orchestration (outside pipeline system)
* Authentication / authorization
* Async tool execution
* Additional model providers (beyond Ollama)
* Distributed runtime or inference

---

# Acceptance Criteria for v0.4.0

* PipelineRegistry supports dynamic pipelines with validation
* PipelineRunner executes all steps according to pipeline definition
* Full structured logging preserved, including step-level latency
* Default pipeline behaves identically to pre-v0.4.0 pipeline
* Test suite expanded to 120+ tests covering custom pipelines
* No regressions in /ingest or OpenWebUI endpoints

---

# Expected Outcome

After v0.4.0, COREtex will:

* Support **configurable pipelines**
* Maintain full determinism and observability
* Remain fully modular, extensible, and testable
* Provide a foundation for hybrid model providers and pipeline extensions in future phases

