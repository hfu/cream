# state-model.md

## Overview

This document defines the state model used by C.R.E.A.M.

C.R.E.A.M. is a cache-first orchestration system.

The purpose of the system is not to generate PMTiles.

The purpose of the system is not to run Martin.

The purpose of the system is to achieve and maintain a desired serving state through cache reconciliation.

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

A resource is available through a remote PMTiles archive.

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
  ↓
SERVING
```

States represent observations.

They do not represent commands.

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

- metadata
- assets
- PMTiles availability

---

## MATERIALIZED

A PMTiles archive exists.

Materialization may occur through Tilepack.

Examples:

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

The resource can be incorporated into a Martin configuration using a remote PMTiles archive.

Example:

```
STAC PMTiles URL
  → config.json
```

The PMTiles archive is not stored locally.

The system remains dependent on remote infrastructure.

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

## SERVING

The resource is actively being served by Martin.

Example:

```
PMTiles
  → config.json
  → Martin
  → tiles endpoint
```

A resource reaches SERVING when it is available through an active Martin process.

SERVING is an operational state.

---

## State Discovery

The current state is derived dynamically.

No persistent state database exists.

State is inferred through observation.

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
Martin process active
```

```
Martin serving tileset <id>
```

---

## Desired State Model

Commands describe desired states.

They do not describe procedures.

Example:

```
just cache <id>
```

does not mean:

```
run Tilepack
```

or

```
start Martin
```

Instead it means:

```
ensure resource is SERVING
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
  → serve
```

For a resource in:

```
MATERIALIZED
```

the same command may perform:

```
shallow cache
  → serve
```

For a resource in:

```
DEEP_CACHED
```

the same command may perform:

```
generate config
  → serve
```

No unnecessary transitions should occur.

---

## Multi-Resource State

A cache operation may target multiple resources.

Example:

```
just cache <id1> <id2> <id3>
```

The desired state becomes:

```
SERVING SET
```

rather than:

```
SERVING RESOURCE
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

The configuration exists only to support the SERVING state.

---

## Preferred States

The preferred storage state is:

```
DEEP_CACHED
```

The preferred operational state is:

```
SERVING
```

These goals are complementary.

Deep Cache improves resilience.

SERVING fulfills user intent.

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

Execution flows downward.

Authority flows upward.

Cache remains the controlling abstraction.

```
C.R.E.A.M.
Cache Rules Everything Around Me.
```
