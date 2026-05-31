# cache-semantics.md

## Overview

This document defines the semantics of cache operations in C.R.E.A.M.

Cache is the primary concern of the system.

All commands are evaluated according to their effect on cache state.

This document defines what cache means.

It does not define how cache is implemented.

---

## Design Principle

```
Cache Rules Everything Around Me.
```

The system exists to reconcile resources into a desired cache state.

Tilepack, STAC, PMTiles, and Martin are supporting mechanisms.

The user interacts with cache.

Cache interacts with everything else.

---

## Cache Ownership

C.R.E.A.M. owns cache state.

The following components do not own cache state:

- STAC
- Tilepack
- PMTiles
- Martin

These systems provide information or capabilities.

Cache remains the controlling abstraction.

---

## Cache Layers

Two cache layers are defined.

### Shallow Cache

A resource is available through a remote PMTiles archive.

Example:

```
STAC
  → PMTiles URL
```

Characteristics:

- no local PMTiles file
- depends on remote availability
- fast to establish
- minimal storage consumption

---

### Deep Cache

A resource exists locally.

Example:

```
./cache/<id>.pmtiles
```

Characteristics:

- locally owned artifact
- reduced network dependency
- reproducible operation

---

## Cache Precedence

Deep Cache always takes precedence over Shallow Cache.

Given:

```
local PMTiles exists
```

and

```
remote PMTiles exists
```

the local PMTiles archive must be used.

---

## Desired State

The primary desired state is:

```
SERVING
```

A resource is considered available when it is actively being served through Martin.

The implementation path is irrelevant.

Examples:

```
local PMTiles
  → Martin
```

and

```
remote PMTiles
  → Martin
```

both satisfy SERVING.

---

## The cache Command

The primary user operation is:

```
just cache <id...>
```

The command describes intent.

It does not describe procedure.

The command means:

```
ensure all specified resources are SERVING
```

The implementation strategy is determined dynamically.

---

## Reconciliation Principle

Cache reconciliation is state-driven.

The current state is observed.

The desired state is:

```
SERVING
```

The system computes the required transitions.

The user does not need to specify:

- STAC lookup
- Tilepack execution
- PMTiles discovery
- Martin installation
- configuration generation
- Martin startup

These are implementation details.

---

## Cache Resolution Order

For each resource:

### Step 1

Check Deep Cache.

Example:

```
./cache/<id>.pmtiles
```

If present:

```
DEEP_CACHED
```

is achieved.

The local artifact becomes authoritative.

---

### Step 2

Resolve the STAC Item.

Discover PMTiles availability.

Example:

```
assets.pmtiles.href
```

If available:

```
SHALLOW_CACHED
```

may be established.

---

### Step 3

If PMTiles is unavailable:

```
Tilepack materialization
```

may be triggered.

Materialization continues until PMTiles becomes discoverable.

---

### Step 4

Ensure Martin is available.

If Martin is not installed:

```
cargo install martin --locked
```

may be executed automatically.

Installation is considered part of reconciliation.

---

### Step 5

Generate runtime configuration.

Example:

```
./config.json
```

The configuration is generated dynamically from the reconciled cache state.

---

### Step 6

Start Martin.

The resource enters:

```
SERVING
```

---

## Multi-Resource Semantics

A cache operation may target multiple resources.

Example:

```
just cache <id1> <id2> <id3>
```

The desired state becomes:

```
SERVING SET
```

Each resource is reconciled independently.

The resulting Martin configuration is generated from the combined state.

---

## Runtime Configuration

The file:

```
./config.json
```

is a runtime artifact.

It is generated every time cache reconciliation occurs.

It is not part of the repository.

It should normally be ignored by version control.

Example:

```
.cache/
config.json
```

may be entirely reconstructed from resource identifiers.

---

## The get Command

The command:

```
just get <id...>
```

has procedural semantics.

The command means:

```
materialize and store PMTiles locally
```

Result:

```
./cache/<id>.pmtiles
```

The command targets Deep Cache.

Unlike cache, it does not imply serving.

---

## The forget Command

The command:

```
just forget <id...>
```

removes Deep Cache artifacts.

Example:

```
./cache/<id>.pmtiles
```

may be removed.

Future cache operations may fall back to Shallow Cache.

The command affects cache ownership only.

It does not modify:

- STAC
- Tilepack
- PMTiles on remote storage

---

## The install Command

The command:

```
just install
```

is optional.

It exists primarily for maintenance and diagnostics.

Example:

```
cargo install martin --locked
```

The cache command may perform the same action automatically.

Users are not expected to run install during normal operation.

---

## Process Lifecycle

The cache command is responsible for reaching the SERVING state.

Example:

```
just cache <id>
```

may eventually result in:

```
martin --config config.json
```

running interactively.

Martin remains active until terminated.

Example:

```
Ctrl-C
```

No dedicated shutdown command is required.

---

## Idempotency

Cache operations should be idempotent.

Examples:

```
just cache <id>
```

```
just get <id>
```

```
just forget <id>
```

may be executed repeatedly.

Repeated execution should converge toward the same state.

---

## Observability

Cache reconciliation should be observable.

Examples:

```
[cache] deep cache found
```

```
[cache] pmtiles discovered via STAC
```

```
[cache] materialization required
```

```
[cache] installing martin
```

```
[cache] generating config.json
```

```
[cache] starting martin
```

```
[cache] serving
```

The system should explain how the desired state was achieved.

---

## Design Summary

The cache command is the center of the system.

Everything else exists to support cache reconciliation.

```
STAC
  ↓

Tilepack
  ↓

PMTiles
  ↓

Cache
  ↓

Martin
```

Execution flows downward.

Authority flows upward.

Cache remains the controlling abstraction.

```
C.R.E.A.M.
Cache Rules Everything Around Me.
```
