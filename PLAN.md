# CortX Development Plan (Alpha Roadmap)

**Project:** CortX\
**Organisation:** cortxai\
**Current Version:** v0.2.0\
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

The existing implementation (v0.2.0) will eventually become the first
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

# Current Implementation Summary (v0.2.0)

The current PoC implements:

Pipeline:

User Input\
→ Classifier LLM\
→ Deterministic Router\
→ Worker LLM\
→ Tool Execution Layer

Key properties:

-   Stateless request pipeline
-   Exactly two LLM calls
-   Deterministic routing
-   Tools executed only via ToolExecutor
-   Local model inference via Ollama
-   Structured logging
-   Fully tested tool execution layer

The system currently behaves like an **application** rather than a
**platform runtime**.

The next phases focus on **architectural restructuring**.

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

# v0.3.0 --- Runtime Extraction

Goal:

Transform the current application architecture into a **CortX runtime
platform**.

This is the most important architectural step.

------------------------------------------------------------------------

## Objectives

Introduce a **runtime layer** that owns:

-   request lifecycle
-   pipeline execution
-   module loading
-   event system
-   registries

Current logic must be refactored so components become **modules**, not
hardcoded imports.

------------------------------------------------------------------------

## New Directory Structure

Target repository layout:

cortx/ runtime/ interfaces/ registry/ config/

modules/ distributions/ tests/

------------------------------------------------------------------------

## Runtime Responsibilities

Runtime should implement:

-   pipeline execution engine
-   module loader
-   system event bus
-   execution context
-   registry management

Runtime must **not contain implementations** of classifiers, tools, or
workers.

------------------------------------------------------------------------

## Interfaces Layer

Define interfaces for:

Classifier\
Worker\
Router\
Tool\
ModelProvider

These interfaces define how modules integrate with the runtime.

------------------------------------------------------------------------

## Registry Layer

Introduce registries for:

ModuleRegistry\
ToolRegistry\
ModelProviderRegistry\
PipelineRegistry

Modules register themselves during runtime initialization.

------------------------------------------------------------------------

## Refactor Existing Code into Modules

Current components become modules:

classifier.py → classifier_basic module

router.py → router_simple module

worker.py → worker_llm module

filesystem tool → tools_filesystem module

ollama integration → model_provider_ollama module

Modules must expose a **registration entrypoint**.

------------------------------------------------------------------------

# v0.3.X --- Stabilisation

Minor releases in the v0.3 series should focus on:

-   runtime stability
-   module system validation
-   logging improvements
-   documentation
-   expanded test coverage

No major architectural additions should occur here.

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
