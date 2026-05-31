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

The system exists to achieve and maintain desired cache states.

Tilepack, STAC, PMTiles, and Martin are supporting mechanisms.

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
  → Martin
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
- reproducible startup behavior

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
SERVABLE
```

A resource is servable when Martin can expose it as a tileset.

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

both satisfy SERVABLE.

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
ensure all specified resources are SERVABLE
```

---

## State Reconciliation

The cache command reconciles observed state with desired state.

Example:

```
desired state:
  SERVABLE
```

Current state may be:

```
UNRESOLVED
```

```
RESOLVED
```

```
MATERIALIZED
```

```
SHALLOW_CACHED
```

```
DEEP_CACHED
```

The system determines the required transitions.

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

No remote lookup is required.

---

### Step 2

Resolve STAC Item.

Discover PMTiles availability.

Example:

```
assets.pmtiles.href
```

If present:

```
SHALLOW_CACHED
```

may be established.

---

### Step 3

If PMTiles does not exist:

```
Tilepack materialization
```

may be triggered.

The resulting PMTiles archive is then discovered through STAC.

---

### Step 4

Generate Martin runtime configuration.

---

### Step 5

Ensure Martin serves the resource.

Result:

```
SERVABLE
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
SERVABLE SET
```

Each resource is evaluated independently.

The resulting Martin configuration is generated from the combined result.

---

## Runtime Configuration

The file:

```
./config.json
```

is generated dynamically.

It is not persistent system state.

It is a runtime artifact.

The file may contain:

- Deep Cache PMTiles
- Shallow Cache PMTiles

simultaneously.

---

## The get Command

The command:

```
just get <id...>
```

has different semantics.

It is procedural.

The command means:

```
download PMTiles into Deep Cache
```

Example:

```
./cache/<id>.pmtiles
```

Unlike cache, get is concerned with storage.

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

The command affects cache ownership.

It does not affect STAC or Tilepack state.

---

## Idempotency

Cache operations should be idempotent.

Examples:

```
just cache <id>
```

may be executed repeatedly.

```
just get <id>
```

may be executed repeatedly.

Repeated execution should not change the resulting cache state.

---

## Observability

Cache reconciliation should be observable.

Example:

```
[cache] deep cache found
```

```
[cache] pmtiles discovered via STAC
```

```
[cache] tilepack materialization required
```

```
[cache] generating config.json
```

```
[cache] martin ready
```

The system should explain how SERVABLE was achieved.

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
