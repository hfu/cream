# state-model.md

## Overview

This document defines the state model used by C.R.E.A.M.

C.R.E.A.M. is a cache-first orchestration system.

The purpose of the system is not to generate PMTiles.

The purpose of the system is not to run Martin.

The purpose of the system is to achieve and maintain a desired cache state.

All other components exist in support of cache.

---

## Core Principle

```
Cache Rules Everything Around Me.
```

Cache is the primary concern.

All operations are evaluated according to their contribution to cache state.

---

## Canonical Identity

A resource is identified by an OpenAerialMap STAC Item ID.

Example:

```
6a18bf8e8a50e594a322d68a
```

This identifier is used consistently throughout the system.

```
STAC Item ID
  = cache key
  = PMTiles identity
  = Martin tileset name
```

---

## Cache Layers

C.R.E.A.M. defines two cache layers.

### Shallow Cache

A resource is servable through a remote PMTiles URL.

Example:

```
STAC Item
  → PMTiles asset URL
  → Martin
```

The PMTiles archive is not stored locally.

The resource remains dependent on external infrastructure.

---

### Deep Cache

A resource is stored locally.

Example:

```
./cache/6a18bf8e8a50e594a322d68a.pmtiles
```

The PMTiles archive becomes locally available.

The resource may continue to be served even if the original source becomes unavailable.

---

## State Model

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
```

---

## UNRESOLVED

The system knows only the resource identifier.

Example:

```
6a18bf8e8a50e594a322d68a
```

No STAC lookup has occurred.

---

## RESOLVED

The STAC Item has been resolved.

Example:

```
STAC Item available
```

The system can inspect:

- assets
- metadata
- PMTiles availability

---

## MATERIALIZED

A PMTiles archive exists.

Materialization may occur through Tilepack.

Example:

```
Tilepack READY
```

or

```
assets.pmtiles exists
```

A PMTiles URL is available.

---

## SHALLOW_CACHED

The resource is servable through Martin.

Example:

```
remote PMTiles URL
  → Martin config.json
  → Martin runtime
```

The PMTiles archive is not stored locally.

The system depends on remote availability.

---

## DEEP_CACHED

The resource exists locally.

Example:

```
./cache/<id>.pmtiles
```

The local artifact becomes the preferred source.

Deep Cache always takes precedence over Shallow Cache.

---

## Preferred State

The preferred steady state is:

```
DEEP_CACHED
```

because:

- network dependency is reduced
- startup becomes deterministic
- cache ownership becomes local

---

## State Discovery

The current state is discovered dynamically.

No persistent state database exists.

C.R.E.A.M. derives state through observation.

Examples:

```
STAC lookup
```

```
assets.pmtiles exists
```

```
./cache/<id>.pmtiles exists
```

```
Martin runtime available
```

---

## Desired State Model

Commands describe desired states.

They do not directly describe procedures.

Example:

```
just cache <id>
```

does not mean:

```
run Tilepack
```

Instead it means:

```
make resource servable
```

The system determines the required transitions.

---

## State Reconciliation

For a resource in:

```
UNRESOLVED
```

the command:

```
just cache <id>
```

may internally perform:

```
resolve
  → materialize
  → shallow cache
```

For a resource in:

```
MATERIALIZED
```

the same command may perform:

```
shallow cache only
```

For a resource in:

```
DEEP_CACHED
```

the same command may perform:

```
generate config
  → run Martin
```

No unnecessary transitions should occur.

---

## Multi-Resource State

A cache operation may target multiple identifiers.

Example:

```
just cache <id1> <id2> <id3>
```

The desired state becomes:

```
SERVABLE SET
```

rather than:

```
SERVABLE RESOURCE
```

Each resource is reconciled independently.

The resulting Martin configuration is generated from the combined state.

---

## Runtime Configuration

The file:

```
./config.json
```

is generated dynamically.

It is not considered part of system state.

It is a runtime artifact.

Example:

```
Deep Cache PMTiles
```

and

```
Shallow Cache PMTiles URLs
```

may coexist within the same generated configuration.

---

## State Ownership

Tilepack does not own state.

STAC does not own state.

Martin does not own state.

Cache owns state.

C.R.E.A.M. exists to reconcile observed reality with desired cache state.

Everything else is implementation detail.

---

## Design Summary

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

The direction of execution flows downward.

The direction of authority flows upward.

Cache remains the controlling layer.

This principle defines the entire system.

```
C.R.E.A.M.
Cache Rules Everything Around Me.
```
