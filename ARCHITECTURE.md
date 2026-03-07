# CortX Architecture

**Project:** CortX\
**Organisation:** cortxai\
**Status:** Alpha\
**Purpose:** Runtime platform for building local-first intelligent
systems.

------------------------------------------------------------------------

# Overview

CortX is designed as a **modular runtime platform** for building
intelligent software systems.

The architecture separates the system into three primary layers:

1.  **Runtime**
2.  **Modules**
3.  **Distributions**

This separation allows CortX to support both:

-   simple local deployments
-   highly customised intelligent systems

while maintaining a stable core.

------------------------------------------------------------------------

# Architectural Layers

## 1. Runtime

The runtime is the **core execution environment** of CortX.

It provides the primitives required to execute intelligent pipelines but
does **not contain application logic**.

Responsibilities:

-   request lifecycle management
-   pipeline execution
-   module loading
-   registry management
-   configuration management
-   event emission

The runtime should remain **small, stable, and highly testable**.

Runtime code lives in:

cortx/runtime/

------------------------------------------------------------------------

## 2. Interfaces

Interfaces define the **contracts** that modules implement.

They ensure the runtime is independent from specific implementations.

Examples:

Classifier\
Router\
Worker\
Tool\
ModelProvider\
MemoryStore (future)

Interfaces live in:

cortx/interfaces/

------------------------------------------------------------------------

## 3. Registries

Registries allow modules to **register capabilities with the runtime**.

Examples:

ModuleRegistry\
ToolRegistry\
ModelProviderRegistry\
PipelineRegistry

Registries live in:

cortx/registry/

------------------------------------------------------------------------

# Modules

Modules are the **extensibility mechanism** of CortX.

They implement platform capabilities and register themselves with the
runtime.

Examples:

classifier_basic\
router_simple\
worker_llm\
tools_filesystem\
model_provider_ollama

Modules live in:

modules/

Each module should expose a single entrypoint file:

module.py

This file registers the module's capabilities with the runtime.

Example capabilities:

-   classifiers
-   routers
-   workers
-   tools
-   model providers

Modules must **never modify runtime behaviour directly**.

------------------------------------------------------------------------

# Pipelines

Pipelines define **how intelligent tasks are executed**.

Example pipeline:

Input\
→ Classifier\
→ Router\
→ Worker\
→ ToolExecutor

Pipelines should be configurable so different distributions can
implement different behaviour.

------------------------------------------------------------------------

# Distributions

Distributions are **complete CortX systems** assembled from:

-   runtime
-   modules
-   pipelines
-   infrastructure

Example:

distributions/cortx_local

This distribution includes:

-   FastAPI ingress
-   OpenWebUI integration
-   Ollama model provider
-   filesystem tools

Distributions allow CortX to support both:

-   beginners who want a ready-to-run system
-   advanced users who want to assemble custom stacks

------------------------------------------------------------------------

# Repository Structure

The repository should follow this structure:

cortx/ runtime/ interfaces/ registry/ config/

modules/

distributions/

tests/

docs/

------------------------------------------------------------------------

# Design Principles

CortX architecture is guided by several core principles:

-   minimal runtime
-   modular extensibility
-   explicit pipelines
-   replaceable components
-   strong observability

------------------------------------------------------------------------

# Runtime Responsibilities

The runtime must remain responsible for only a small number of core
systems:

-   execution lifecycle
-   pipeline orchestration
-   module loading
-   event emission
-   registry management

Everything else must exist as modules.

------------------------------------------------------------------------

# Event System

CortX will emit structured runtime events to support:

-   debugging
-   tracing
-   monitoring
-   observability

Example events:

request_received\
intent_detected\
model_invoked\
tool_called\
response_generated

------------------------------------------------------------------------

# Model Strategy

CortX supports flexible model strategies.

Models are accessed through **ModelProvider modules**.

Examples:

model_provider_ollama\
model_provider_openai (future)

Different model roles may use different providers:

classifier model\
worker model\
planner model (future)

This enables hybrid local/cloud architectures.

------------------------------------------------------------------------

# Long-Term Architecture

As CortX evolves, the architecture will support:

-   module ecosystems
-   custom pipelines
-   hybrid model infrastructure
-   advanced tool integrations

The core runtime should remain stable across all future versions.
