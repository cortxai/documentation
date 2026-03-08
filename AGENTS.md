# AGENTS.md

## Purpose

This document defines the operational rules for both human developers and AI coding agents working in this repository.

All contributors **must follow these rules** when creating branches, commits, and versions.

The goals are:

- predictable versioning
- clean Git history
- small, atomic commits
- semantic commit messages
- traceable project evolution

---

# Versioning Rules

The project uses strict semantic-style version identifiers using the format:

`vX.Y.Z`

Where:

| Component | Meaning |
|-----------|--------|
| X | Major version (architectural milestone or major redesign) |
| Y | Feature branch version |
| Z | Commit iteration within the branch |

Example version:

`v1.4.7`

---

# Branching Model

## Creating New Branches

A new development branch **must increment the Y component**.

Example:

Current version:

`v1.4.3`

New branch version:

`v1.5`

Branch naming format:

`feature/vX.Y`

Example:

`feature/v1.5`

Rules:

- X remains unchanged when branching
- Y increments by **1**
- Z resets to **0** when the branch is created

---

# Commit Versioning

Every commit **must include a version identifier**.

Each commit **increments the Z component**.

Example commit progression:

`v1.5.0`  
`v1.5.1`  
`v1.5.2`  
`v1.5.3`

Rules:

- Only **one increment per commit**
- X and Y must **never change inside a branch**
- Z must always increment sequentially

---

# Commit Frequency

A commit **must be made after each unit of work**.

A "unit of work" is defined as:

- completing a function
- completing a module
- fixing a bug
- adding a test
- updating documentation
- refactoring a logical section

Avoid:

- large commits
- combining unrelated work
- multi-feature commits

Preferred pattern:

implement parser → commit  
add tests → commit  
fix parsing bug → commit

---

# Semantic Commit Messages

All commits must follow **Semantic Commit format**.

Commit message format:

`<type>: <description> <version>`

Example commits:

`feat: add intent router v1.5.1`  
`fix: correct classification threshold v1.5.2`  
`docs: update architecture documentation v1.5.3`  
`refactor: simplify router pipeline v1.5.4`  
`test: add router unit tests v1.5.5`

---

## Allowed Commit Types

| Type | Usage |
|-----|------|
| feat | New feature |
| fix | Bug fix |
| refactor | Internal code restructuring |
| docs | Documentation changes |
| test | Tests added or modified |
| chore | Maintenance tasks |
| perf | Performance improvements |

---

# Commit Requirements

Each commit must:

- include a semantic commit message
- include the version identifier
- represent a single logical change
- leave the project in a working state

---

# Example Development Flow

Starting version:

`v1.4.3`

Create branch:

`feature/v1.5`

Commit sequence:

`feat: implement tool execution layer v1.5.0`  
`test: add tool execution tests v1.5.1`  
`fix: correct tool timeout handling v1.5.2`  
`docs: update tool architecture docs v1.5.3`

---

# Major Version Changes

The X component increments only when:

- architecture significantly changes
- APIs break
- a new major system capability is introduced

Example:

`v1.9.12 → v2.0.0`

Major versions must be merged to the `main` branch.

---

# Pull Request Rules

Before merging:

- ensure all commits follow semantic format
- ensure version sequence is correct
- ensure documentation is updated
- ensure tests pass

---

# Agent Behavior Requirements

AI coding agents must:

1. Read `AGENTS.md` before performing any task.
2. Follow the versioning rules exactly.
3. Commit after every unit of work.
4. Never batch unrelated changes into one commit.
5. Ensure commit messages follow semantic format.
6. Increment version numbers correctly.

Agents must **never bypass these rules**.

---

# Summary

Core workflow:

create branch (increment Y)  
perform small unit of work  
commit (increment Z)  
repeat  
merge when feature complete

Version structure:

`vX.Y.Z`

Example lifecycle:

`v1.4.3`  
→ `feature/v1.5`  
→ `v1.5.0`  
→ `v1.5.1`  
→ `v1.5.2`  
→ merge

---

# Guiding Principle

**Small commits. Clear history. Deterministic versioning.**
