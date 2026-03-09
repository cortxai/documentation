# CortX Development Plan (Alpha Roadmap)

**Project:** CortX\
**Organisation:** cortxai\
**Current Version:** v0.3.15\
**Status:** Alpha

------------------------------------------------------------------------

# Purpose of this Document

This document provides a **structured implementation roadmap** for CortX
that can be ingested by an LLM coding assistant. It explains:

-   The **current architectural state**
-   The **next implementation phase**
-   A **versioned roadmap** for upcoming phases
-   **Architectural constraints** that must remain true
-   **Non-goals** (features intentionally deferred)

The goal is to guide development toward the long‑term vision of CortX as
a **local‑first runtime platform for intelligent systems**.

------------------------------------------------------------------------

# Architectural Vision

CortX is evolving from a proof‑of‑concept **local agentic system** into
a **platform runtime for intelligent software systems**.

The long‑term architecture will consist of three conceptual layers:

1.  **CortX Runtime**
    -   Core execution environment
    -   Module loading
    -   Pipeline orchestration
    -   Event system
2.  **CortX Modules**
    -   Extensions that implement platform capabilities
    -   Classifiers
    -   Workers
    -   Routers
    -   Model providers
    -   Tools
    -   Infrastructure integrations
3.  **CortX Distributions**
    -   Opinionated system builds
    -   Provide a complete runnable platform
    -   Example: `cortx_local`

The existing implementation (v0.3.15) will eventually become the first
**distribution**.

------------------------------------------------------------------------

# Architectural Rules (Must Never Be Violated)

1.  **Core must remain small**
    -   Runtime contains primitives only
    -   No integrations or application logic
2.  **Everything must be replaceable**
    -   Models
    -   tools
    -   classifiers
    -   workers
    -   routers
3.  **Modules extend the runtime**
    -   Runtime must never depend on modules
4.  **Intelligence must be explicit pipelines**
    -   Avoid opaque agent frameworks
5.  **Distributions are separate from the runtime**

------------------------------------------------------------------------

# Current Implementation Summary (v0.3.0)

The current implementation introduces the **COREtex runtime platform**, separating the system into a reusable runtime, pluggable modules, and assembled distributions.

Pipeline:

User Input\
→ Classifier (deterministic prefix checks → LLM fallback)\
→ Deterministic Router\
→ Worker LLM\
→ Agent JSON Action\
→ Tool Execution Layer

Key properties:

-   Modular runtime architecture (`coretex/`, `modules/`, `distributions/`)
-   Stateless request pipeline
-   Exactly two LLM calls per successful request
-   Deterministic routing with no LLM involvement
-   Agent outputs structured JSON action envelopes
-   Tools executed only via `ToolExecutor`
-   Local model inference via Ollama
-   Module loading via a runtime module registry
-   Structured logging with request correlation IDs
-   OpenAI-compatible API shim for OpenWebUI
-   Fully mocked test suite covering the entire runtime

The system now behaves as a **runtime platform** capable of supporting
multiple modules and distributions.

The next phase focuses on **runtime stabilisation, module validation,
and observability improvements**.

------------------------------------------------------------------------

# Versioning Strategy

Version numbers follow the format:

0.N.X

Where:

N = **major architectural changes**\
X = **minor improvements**

Examples:

v0.3.0 → major architectural change\
v0.3.1 → minor improvements or fixes

Version **v1.0.0** marks the end of the alpha/beta period.

------------------------------------------------------------------------

# Next Phase

# v0.3.X --- Stablisation

Goal:

Harden the runtime, validate the module architecture, improve observability, and expand test coverage

No major architectural additions are permitted in this phase.

------------------------------------------------------------------------

# Objectives

Focus on:
- runtime stability
- module system validation
- logging improvements
- documentation
- expanded test coverage

------------------------------------------------------------------------

# Section 1 — Runtime Stability

The runtime layer in coretex/runtime/ must become robust against all failure modes.

## 1.1 Harden PipelineRunner

File:
```
coretex/runtime/pipeline.py
```

### Required Improvements

#### Explicit Failure Categories

Introduce explicit error categories inside the pipeline:
```
ClassificationFailure
WorkerFailure
ToolExecutionFailure
AgentParseFailure
```

These should not introduce new classes unless necessary — simple exception categorisation is acceptable.

Failures should be logged consistently.

Example:
```
event=pipeline_classifier_failure
event=pipeline_worker_failure
event=pipeline_tool_failure
event=pipeline_agent_parse_failure
```

### Defensive Behaviour

Ensure the pipeline:

|Failure                 |Behaviour                        |
|------------------------|---------------------------------|
|Classifier HTTP failure |fallback → ambiguous             |
|Worker HTTP failurere   |turn worker failure response     |
|Tool lookup failure     |worker failure response          |
|Tool runtime exception  |worker failure response          |
|Agent JSON parse failure|treat raw output as text response|

This behaviour already partially exists but must be consistent and logged.

## 1.2 Standardise Pipeline Logging

Every request should produce a complete traceable log chain.

Expected log lifecycle:
```
event=request_received
event=classifier_start
event=classifier_complete
event=router_selected
event=worker_start
event=worker_complete
event=agent_output_received
event=tool_execute
event=tool_execute_complete
event=request_complete
```

All logs must include:
```
request_id
intent (when known)
handler
duration_ms (when applicable)
```

## 1.3 ExecutionContext Expansion (Minor Only)

File:
```
coretex/runtime/context.py
```

Add optional metadata fields:
```
metadata: dict[str, Any] | None
timestamp: float
```

These should not change pipeline behaviour.

Purpose: improve observability.

------------------------------------------------------------------------

# Section 2 — Module System Validation

The module system introduced in v0.3.0 must be validated to ensure:
- modules cannot corrupt runtime state
- duplicate registrations are prevented
- missing dependencies are detected

## 2.1 Registry Validation

Registries must enforce strict duplicate detection.

Affected files:
```
coretex/registry/module_registry.py
coretex/registry/tool_registry.py
coretex/registry/model_registry.py
coretex/registry/pipeline_registry.py
```

Requirements:

### Duplicate Registration

All `register_*` functions must raise:
```
ValueError("Component already registered: <name>")
Unknown Lookup
```

All `get()` methods must raise:
```
ValueError("Unknown component: <name>")
```

And log:
```
event=registry_lookup_failed
component=<type>
name=<name>
```

## 2.2 ModuleLoader Validation

File:
```
coretex/runtime/loader.py
```

The loader must:
- Import the module
- Verify `register()` exists
- Execute `register()`

Add validation steps:

### Validate Signature

`register()` must accept:
```
register(module_registry, tool_registry, model_registry)
```

If incorrect:
```
ValueError("Invalid module register() signature")
```

### Detect Partial Registration

Log warnings when a module registers nothing.
```
event=module_loaded
module=<name>
registered_components=<count>
```

## 2.3 Module Loading Logs

Startup logs should show:
```
event=module_loading_start
event=module_loaded
event=module_loading_complete
```

Example:
```
module=classifier_basic
components=1
```

------------------------------------------------------------------------

# Section 3 — Logging & Observability Improvements

Logging must be **machine-readable**.

## 3.1 Structured Logging Format

Use structured key=value logs.

Example:
```
event=classifier_complete request_id=abc intent=execution confidence=0.92 duration_ms=312
```

Do NOT introduce external logging libraries.

Use Python `logging`.

## 3.2 Request Duration Metrics

Add timing measurements to:
- classifier
- worker
- pipeline total

Example:
```
duration_ms
```

## 3.3 Router Debug Improvements

File:
```
modules/router_simple/router.py
```

When settings.debug_router == True log:
```
event=router_decision
intent=<intent>
handler=<handler>
```

------------------------------------------------------------------------

# Section 4 — Test Coverage Expansion

Current tests: 64

Target: 100+ tests

All tests remain in:
```
tests/test_smoke.py
```

Do not split files yet.

## 4.1 Registry Tests

Add tests for:
```
duplicate tool registration
duplicate module registration
unknown tool lookup
unknown classifier lookup
unknown worker lookup
```

## 4.2 ModuleLoader Tests

Test cases:

|Scenario                   |Expected Result|
|---------------------------|---------------|
|module missing `register()`|failure        |
|register wrong signature   |failure        |
|module registers nothing   |warning        |
|module registers components|success        |

## 4.3 Tool Executor Tests

Add tests for:
```
tool execution success
unknown tool
tool runtime exception
respond action bypass
invalid action
```

## 4.4 Pipeline Failure Tests

Mock scenarios:
```
classifier HTTP failure
worker HTTP failure
invalid JSON output
tool lookup failure
tool runtime exception
```

Verify correct fallback responses.

## 4.5 Logging Tests

Capture logs and assert presence of:
```
request_received
classifier_complete
router_selected
worker_complete
request_complete
```

------------------------------------------------------------------------

# Section 5 — Documentation

The runtime extraction introduced new architectural concepts.

Documentation must reflect them.

## 5.1 Add COREtex Runtime Documentation

Create:
```
docs/runtime.md
```

Contents:
- runtime responsibilities
- module architecture
- registries
- pipeline execution flow
- failure behaviour

## 5.2 Module Development Guide

Create:
```
docs/module_development.md
```

Explain:
- module directory structure
- required register() function
- component registration
- best practices
- common errors

## 5.3 Distribution Guide

Create:
```
docs/distributions.md
```

Explain:
- how to build a distribution
- how bootstrap works
- how to load modules

------------------------------------------------------------------------

# Section 6 — Minor Code Quality Improvements

These are safe refactors.

## 6.1 Type Hint Improvements

Ensure full typing coverage in:
```
runtime/
registry/
interfaces/
```

Avoid Any unless unavoidable.

## 6.2 Docstrings

Add docstrings to:
```
PipelineRunner
ToolExecutor
ModuleLoader
ToolRegistry
ModuleRegistry
```

## 6.3 Consistent Naming

Ensure all event logs follow:
```
event=<name>
```

No mixed formats.

------------------------------------------------------------------------

# Non-Goals (Must Not Be Implemented)

The following must not be added in v0.3.x:
- memory systems
- conversation history
- task graphs
- planners
- multi-agent coordination
- streaming responses
- authentication
- distributed runtime
- additional model providers
- async tool execution
- plugin dependency graphs

------------------------------------------------------------------------

# Acceptance Criteria

The stabilisation phase is complete when:
- runtime behaviour is deterministic
- logging provides full request traces
- module loading is validated
- registry safety is guaranteed
- test count exceeds 100
- runtime documentation exists
- all tests pass with Ollama fully mocked

------------------------------------------------------------------------

# Expected Outcome

After v0.3.x stabilisation, COREtex will be:
- a stable agentic runtime kernel
- safely extensible through modules
- observable via structured logs
- thoroughly tested

------------------------------------------------------------------------

# v0.4.0 --- Pipeline System

Goal:

Introduce configurable **execution pipelines**.

The current pipeline is hardcoded.

Instead pipelines should be defined as structured execution graphs.

Example:

Input → Classifier → Router → Worker → ToolExecutor

Future pipelines may include planners, evaluators, or tool graphs.

Pipelines should be configurable without modifying runtime code.

------------------------------------------------------------------------

# v0.4.X --- Model Provider System

Introduce a **model provider abstraction layer**.

Define interface:

ModelProvider

Methods:

generate() chat()

Implement first provider module:

model_provider_ollama

Future providers may include:

OpenAI Anthropic llama.cpp other local inference engines

This enables hybrid model strategies.

------------------------------------------------------------------------

# v0.5.0 --- Distribution Layer

Goal:

Introduce **CortX distributions**.

Create first distribution:

cortx_local

This distribution includes:

runtime basic modules ollama provider filesystem tool FastAPI ingress
OpenWebUI integration

The existing system behaviour should remain unchanged.

The difference is that it is now assembled from runtime + modules.

------------------------------------------------------------------------

# v0.6.0 --- Event System

Goal:

Introduce a runtime **event bus**.

Every important system action should emit events.

Examples:

request_received intent_detected model_invoked tool_called
response_generated

This allows:

debugging observability metrics future monitoring integrations

------------------------------------------------------------------------

# v0.7.0 --- Move Tool System into Modules

Goal:

Refactor the tool system so tools are no longer defined inside core
runtime code.

Tools become modules.

Example modules:

tools_filesystem tools_shell tools_http tools_git

Each module registers tools with the runtime.

This allows users to:

add tools disable tools create third‑party tools

This completes the **core modular architecture**.

------------------------------------------------------------------------

# Non‑Goals (Out of Scope for Current Phases)

The following features are intentionally **not part of the current
roadmap**.

They may be explored after v0.7.

------------------------------------------------------------------------

## Memory Systems

Conversation memory vector stores long‑term agent memory

Memory architectures significantly increase complexity.

They should not be implemented until the runtime architecture
stabilizes.

------------------------------------------------------------------------

## Multi‑Agent Systems

Agent‑to‑agent collaboration autonomous agent networks cooperative
reasoning

Most frameworks introduce these prematurely.

CortX should remain **single pipeline execution** for now.

------------------------------------------------------------------------

## Autonomous Planning Loops

Recursive planning self‑directed agents goal‑seeking execution

These introduce instability and unpredictable execution.

They belong in **optional modules** later.

------------------------------------------------------------------------

## Distributed Inference Scheduling

Load balancing model inference across hosts.

Future possibility:

CortX model gateway multiple inference hosts

But this should not be built until the platform architecture is stable.

------------------------------------------------------------------------

# Long‑Term Vision

By the time CortX approaches v1.0 the system should provide:

-   a stable runtime platform
-   a module ecosystem
-   configurable pipelines
-   hybrid model support
-   distribution builds

At that point CortX becomes:

**A local‑first runtime for building intelligent systems.**
