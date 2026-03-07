# MODULE_SPEC.md

## CortX Module Specification

**Status:** Draft\
**Applies to:** CortX v0.6.x and later\
**Purpose:** Define the required structure and contract for all CortX
modules.

This document ensures that all modules added to the CortX ecosystem
follow a consistent interface so they can be discovered, loaded, and
executed safely by the runtime.

------------------------------------------------------------------------

# 1. Overview

A **CortX Module** is a self-contained capability package that can be
registered with the CortX runtime.

Modules may provide:

-   Tools
-   Agents
-   Workflows
-   Integrations
-   Data adapters
-   Domain logic

Modules must follow strict structural and interface requirements so
that:

-   The runtime can **discover them automatically**
-   LLM coding assistants can **generate them reliably**
-   The system remains **local-first and composable**

------------------------------------------------------------------------

# 2. Core Principles

Every module must follow these design principles:

1.  **Self Contained**
    -   Modules should not rely on global runtime state.
2.  **Explicit Registration**
    -   Modules must register themselves through a module entrypoint.
3.  **Clear Capability Declaration**
    -   Modules must explicitly declare what capabilities they provide.
4.  **No Hidden Side Effects**
    -   Modules must not perform work during import.
5.  **Local First**
    -   Modules must function without requiring external services.

------------------------------------------------------------------------

# 3. Standard Module Directory Layout

Example module layout:

    modules/
      example_module/
        module.py
        tools/
          tool_a.py
          tool_b.py
        agents/
          helper_agent.py
        workflows/
          workflow_x.py
        config/
          defaults.yaml
        README.md

Required file:

    module.py

------------------------------------------------------------------------

# 4. Required Entry Point

Each module must expose a `register()` function.

Example:

``` python
def register(registry):
    registry.register_tool(my_tool)
    registry.register_agent(my_agent)
    registry.register_workflow(my_workflow)
```

This function is called by the CortX module loader.

The registry object will provide:

-   `register_tool()`
-   `register_agent()`
-   `register_workflow()`
-   `register_capability()`

------------------------------------------------------------------------

# 5. Module Metadata

Modules must define metadata:

``` python
MODULE_METADATA = {
    "name": "example_module",
    "version": "0.1.0",
    "description": "Example CortX module",
    "author": "CortX",
    "capabilities": [
        "tools",
        "workflows"
    ]
}
```

Fields:

  Field          Description
  -------------- ------------------------------
  name           Unique module name
  version        Module version
  description    Short description
  author         Module author
  capabilities   Declared module capabilities

------------------------------------------------------------------------

# 6. Tool Definition Standard

Tools must follow this structure:

``` python
def example_tool(input_text: str) -> str:
    \"\"\"
    Description of tool for the LLM.
    \"\"\"
    return "result"
```

Optional metadata:

``` python
example_tool.metadata = {
    "name": "example_tool",
    "description": "Demonstration tool"
}
```

Tools must be:

-   Pure functions when possible
-   Deterministic
-   Side-effect minimal

------------------------------------------------------------------------

# 7. Agent Definition Standard

Agents must expose:

    run(context)

Example:

``` python
class ExampleAgent:

    def run(self, context):
        return "result"
```

Agents should:

-   Accept structured context
-   Return structured output

------------------------------------------------------------------------

# 8. Workflow Definition Standard

Workflows orchestrate multiple tools or agents.

Example:

``` python
def example_workflow(context):

    result = tool_a(context)

    return result
```

Workflows should:

-   Be deterministic where possible
-   Avoid hidden global state

------------------------------------------------------------------------

# 9. Capability Declaration

Modules may declare capabilities such as:

-   tools
-   agents
-   workflows
-   integrations
-   data_sources

Example:

``` python
MODULE_METADATA["capabilities"] = [
    "tools",
    "agents"
]
```

------------------------------------------------------------------------

# 10. Module Loader Expectations

The CortX runtime will:

1.  Scan the `modules/` directory
2.  Import `module.py`
3.  Read `MODULE_METADATA`
4.  Call `register()`

Modules must **not perform work at import time**.

------------------------------------------------------------------------

# 11. Version Compatibility

Modules must remain compatible with the CortX runtime version they
target.

Example:

    requires_cortx >= 0.6.0

------------------------------------------------------------------------

# 12. Security Rules

Modules must not:

-   Open network connections automatically
-   Execute shell commands without explicit user request
-   Modify runtime internals
-   Access filesystem outside allowed paths

------------------------------------------------------------------------

# 13. Testing Expectations

Each module should include tests:

    tests/
      test_tools.py
      test_agents.py

Tests should validate:

-   Tool behavior
-   Agent outputs
-   Workflow logic

------------------------------------------------------------------------

# 14. Example Minimal Module

    modules/
      hello/
        module.py
        tools/
          hello.py

module.py:

``` python
from .tools.hello import hello

MODULE_METADATA = {
    "name": "hello",
    "version": "0.1.0",
    "capabilities": ["tools"]
}

def register(registry):
    registry.register_tool(hello)
```

------------------------------------------------------------------------

# 15. Future Extensions (Not Yet In Scope)

Planned module features:

-   Module dependency graph
-   Version resolution
-   Sandboxed execution
-   Remote module registries
-   Capability permissions
-   Module isolation environments

These will be introduced after **CortX v0.7.x**.

------------------------------------------------------------------------

# 16. Summary

Modules are the core extensibility mechanism of CortX.

Strict adherence to this specification ensures:

-   predictable runtime behavior
-   LLM-friendly development
-   safe modular expansion

This specification will evolve as CortX approaches **v1.0**.
