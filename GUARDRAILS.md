# CortX Architectural Guardrails

This document defines **non-negotiable architectural rules** for the
CortX codebase.

Any modification to the repository must respect these guardrails.

These rules protect the long-term maintainability of the platform.

------------------------------------------------------------------------

# Guardrail 1 --- Core Runtime Must Remain Small

The CortX runtime is a **platform kernel**, not an application.

The runtime must only contain:

-   execution lifecycle
-   pipeline orchestration
-   module loading
-   registry management
-   event emission
-   configuration

The runtime must **never contain**:

-   integrations
-   tools
-   model providers
-   infrastructure logic
-   application features

If a feature can exist as a module, it must **not be added to the
runtime**.

------------------------------------------------------------------------

# Guardrail 2 --- Runtime Must Never Depend on Modules

Dependency direction must always be:

modules → runtime

Never:

runtime → modules

The runtime must not import or reference specific modules.

All modules must register themselves dynamically.

------------------------------------------------------------------------

# Guardrail 3 --- Everything Must Be Replaceable

Major system components must be replaceable through interfaces.

Replaceable systems include:

-   classifiers
-   routers
-   workers
-   model providers
-   tools
-   pipelines

The runtime must interact with these systems through **interfaces**, not
concrete implementations.

------------------------------------------------------------------------

# Guardrail 4 --- Intelligence Must Be Explicit

CortX must avoid opaque agent frameworks.

Intelligent behaviour must be expressed through **explicit pipelines**.

Example:

Input\
→ Classifier\
→ Router\
→ Worker\
→ ToolExecutor

Developers must be able to understand system behaviour by inspecting the
pipeline.

Avoid hidden orchestration or implicit agent behaviour.

------------------------------------------------------------------------

# Guardrail 5 --- Distributions Are Separate From the Platform

CortX itself is a runtime platform.

Application systems must exist as **distributions**.

Example:

distributions/cortx_local

The runtime must remain independent from:

-   FastAPI
-   web interfaces
-   infrastructure environments

Distributions may include those components.

------------------------------------------------------------------------

# Guardrail 6 --- Tools Must Execute Through the ToolExecutor

Agents must **never execute tools directly**.

All tool execution must occur through the ToolExecutor system.

This ensures:

-   observability
-   logging
-   security
-   validation

------------------------------------------------------------------------

# Guardrail 7 --- Pipelines Must Be Deterministic

Pipeline orchestration must be deterministic.

LLMs may be used for:

classification\
reasoning\
content generation

LLMs must **not be used to control core orchestration logic**.

Routing and execution decisions must remain deterministic.

------------------------------------------------------------------------

# Guardrail 8 --- Observability Is Mandatory

All important system operations must emit structured events.

Examples:

request_received\
intent_detected\
model_invoked\
tool_called\
response_generated

This ensures the system remains debuggable as complexity increases.

------------------------------------------------------------------------

# Guardrail 9 --- Avoid Premature Complexity

The following systems must **not be implemented prematurely**:

-   multi-agent orchestration
-   autonomous planning loops
-   memory systems
-   distributed inference scheduling

These systems will only be introduced after the core runtime
architecture stabilises.

------------------------------------------------------------------------

# Guardrail 10 --- Prefer Simple Pipelines

Complex orchestration frameworks should be avoided.

The preferred model is:

simple pipelines\
clear execution steps\
deterministic behaviour

Simplicity improves:

debugging\
observability\
maintainability

------------------------------------------------------------------------

# Architectural Philosophy

CortX should evolve as:

A minimal runtime platform for building intelligent systems.

Not:

A monolithic AI application or experimental agent framework.

Maintaining architectural discipline is critical to achieving this goal.
