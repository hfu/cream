# consistency-check.md

## Overview

This document defines consistency rules for the C.R.E.A.M. specification set.

C.R.E.A.M. is a tightly coupled system of:

- state model
- cache semantics
- STAC binding
- Tilepack integration
- Martin runtime
- CLI interface

These components must remain internally consistent.

This document exists to enforce that consistency.

---

## Core Principle

```
Cache Rules Everything Around Me.
```

Consistency is defined relative to cache behavior.

If a rule conflicts with cache semantics, cache semantics takes precedence.

---

## Primary Consistency Axis

All system behavior must align along a single axis:

```
STAC ID → Cache → SERVING
```

Any deviation is considered a specification defect.

---

## Forbidden Inconsistencies

### 1. Multiple Sources of Truth

Invalid:

- STAC is authoritative
- Cache is authoritative
- Martin is authoritative

Valid:

- STAC describes
- Cache owns state
- Martin serves

---

### 2. Command Explosion

Invalid:

```
just install
just get
just serve
just start
just run
```

Valid:

```
just cache <id...>
```

All other commands must be subordinate or internal.

---

### 3. State Leakage into CLI

Invalid:

- user must choose shallow vs deep cache
- user must decide materialization strategy
- user must start Martin manually

Valid:

- system decides all state transitions
- user only expresses intent via cache

---

### 4. External System Authority Confusion

Invalid:

- Tilepack decides final state
- STAC decides cache state
- Martin decides availability

Valid:

- Tilepack produces artifacts
- STAC describes resources
- Martin serves artifacts
- Cache reconciles everything

---

## State Model Consistency Rule

All documents must agree on final operational state:

```
SERVING
```

Any document that terminates at:

- DEEP_CACHED
- SHALLOW_CACHED
- MATERIALIZED

without reaching SERVING is considered incomplete from a system perspective.

---

## CLI Consistency Rule

All user-facing behavior must reduce to:

```
just cache <id...>
```

Any additional command must satisfy at least one:

- debugging utility
- internal orchestration step
- backward compatibility

No additional command may introduce new conceptual primitives.

---

## STAC Binding Consistency Rule

All identifiers must remain stable across:

- STAC
- Tilepack
- Cache
- Martin

Rule:

```
STAC Item ID = universal system key
```

No translation layer is allowed.

---

## Cache Semantics Consistency Rule

Cache semantics must always imply:

```
intent → SERVING
```

Any definition of cache that does not terminate in SERVING is invalid.

---

## Martin Integration Consistency Rule

Martin must remain:

- stateless
- configuration-driven
- externally controlled

Martin must never:

- decide cache strategy
- resolve STAC
- trigger Tilepack

---

## Lifecycle Consistency Rule

System lifecycle must follow:

```
cache command → SERVING → Ctrl-C
```

No alternative lifecycle paths are permitted in the primary model.

---

## Multi-Resource Consistency Rule

Multiple IDs must behave as a single SERVING session:

```
just cache id1 id2 id3
```

must result in:

- unified configuration
- single Martin instance
- combined serving state

No per-ID runtime isolation at CLI level.

---

## Configuration Consistency Rule

`config.json` must always be:

- ephemeral
- reproducible
- derived from cache state only

It must not encode:

- user decisions
- historical state
- external system state

---

## Observability Consistency Rule

Logs across all components must reflect:

```
cache → reconciliation → serving
```

No component may introduce alternative narratives.

---

## System Integrity Statement

If all rules in this document are satisfied:

- system is considered internally consistent
- SERVING state is valid and stable

If any rule is violated:

- specification must be corrected before implementation

---

## Final Principle

C.R.E.A.M. is consistent only when:

```
cache is the only concept required to explain the system
```

Everything else must collapse into that abstraction.

```
Cache Rules Everything Around Me.
```
