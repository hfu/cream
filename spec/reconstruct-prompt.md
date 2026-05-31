# reconstruct-prompt.md

## Purpose

This document defines how to reconstruct the C.R.E.A.M. system from minimal knowledge.

C.R.E.A.M. is a cache-driven tile serving system.

If all implementation, code, or repository context is lost, the system can be rebuilt from the principles defined here.

---

## Core Principle

```
Cache Rules Everything Around Me.
```

This is the only required invariant.

All reconstruction must preserve this principle.

---

## Minimal Input

The system can be reconstructed from:

- A single command concept:
  ```
  just cache <id>
  ```

- A data identity model:
  - OpenAerialMap STAC Item ID

Example:

```
6a18bf8e8a50e594a322d68a
```

---

## Target System Behavior

The reconstructed system must satisfy:

### 1. Single Command Interface

Only one user-facing command exists:

```
just cache <id...>
```

No other primary commands are required.

---

### 2. Operational Goal

The system must always converge to:

```
SERVING
```

SERVING means:

- a tile server is running
- resources are accessible via HTTP tiles API
- system is in an interactive state

---

### 3. Hidden Complexity Rule

All of the following must be internalized:

- STAC resolution
- PMTiles discovery
- Tilepack materialization
- local cache management
- remote fallback handling
- server configuration generation
- Martin startup

None of these are user-facing concepts.

---

### 4. Architecture Reconstruction

The system must be reconstructed as a layered pipeline:

```
STAC ID
  ↓
STAC resolution
  ↓
PMTiles discovery / materialization
  ↓
Cache layer (local or remote)
  ↓
Runtime configuration
  ↓
Martin tile server
  ↓
SERVING
```

---

### 5. Cache as Central Abstraction

Cache is the only meaningful system abstraction.

Rules:

- cache owns state
- STAC only describes data
- Tilepack only produces artifacts
- Martin only serves results

---

### 6. State Model Reconstruction

The system must support the following state progression:

```
UNRESOLVED
  ↓
RESOLVED
  ↓
MATERIALIZED
  ↓
SHALLOW_CACHED
  ↓
DEEP_CACHED
  ↓
SERVING
```

SERVING is the only terminal state.

---

### 7. CLI Reconstruction Rules

The reconstructed CLI must:

- expose only `just cache <id...>` as primary interface
- hide all auxiliary commands by default
- embed installation, startup, and reconciliation logic inside cache

Optional commands may exist only as debugging tools:

- get (materialization only)
- forget (local cache cleanup)
- install (dependency management)

These must not be part of the primary mental model.

---

### 8. Configuration Principle

The system must generate ephemeral runtime configuration:

```
config.json
```

Rules:

- must be derived from cache state only
- must not be user-authored
- must be regenerated on every execution

---

### 9. Determinism Requirement

Given the same STAC ID set, the system must always converge to the same SERVING behavior, regardless of:

- prior state
- local cache presence
- remote availability (fallback allowed but must converge)

---

### 10. User Mental Model

The reconstructed system must be explainable as:

```
cache → serve
```

No additional mental model is allowed.

---

## Reconstruction Algorithm (Conceptual)

To reconstruct C.R.E.A.M.:

1. Define STAC Item ID as canonical identity
2. Define cache as the only user-facing abstraction
3. Define SERVING as terminal state
4. Define `just cache <id>` as the only primary command
5. Map all infrastructure (Tilepack, PMTiles, Martin) as internal layers
6. Ensure all flows converge into SERVING
7. Ensure system is explainable as a single-command UX

---

## Validation Criteria

A reconstructed implementation is valid if:

- user only needs to know `just cache <id>`
- system reaches SERVING state reliably
- STAC, Tilepack, PMTiles, Martin are not exposed as concepts
- cache reconciliation is fully internal
- system behavior is deterministic given same inputs

---

## Final Principle

If reconstruction is correct, the system collapses into:

```
just cache <id>
```

and nothing more.

```
Cache Rules Everything Around Me.
```
