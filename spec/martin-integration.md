# martin-integration.md

## Overview

This document defines how C.R.E.A.M. integrates with Martin.

Martin is the runtime tile server responsible for serving PMTiles resources.

Martin is not a cache system.

Martin is not a state system.

Martin is a serving runtime.

C.R.E.A.M. owns state and cache.

Martin consumes the reconciled result.

---

## Design Principle

```
Cache Rules Everything Around Me.
```

Martin exists only to serve the cache state produced by C.R.E.A.M.

Martin does not decide what to serve.

It only executes a generated configuration.

---

## Role Separation

### C.R.E.A.M.

- resolves STAC items
- materializes PMTiles (via Tilepack)
- manages deep cache
- constructs serving intent
- generates runtime configuration

### Martin

- loads config.json
- serves PMTiles tilesets
- exposes tile endpoints
- runs as an interactive process

---

## Installation

Martin is installed via Rust cargo:

```
cargo install martin --locked
```

This step is considered part of cache reconciliation.

It may be executed automatically when required.

Users are not expected to manually install Martin in normal operation.

---

## Runtime Model

Martin is executed as an interactive process:

```
martin --config ./config.json
```

This process runs until explicitly terminated.

Termination is performed via:

```
Ctrl-C
```

No separate shutdown command exists.

---

## Configuration File

C.R.E.A.M. generates:

```
./config.json
```

This file is a runtime artifact.

It is not stored in version control.

It is regenerated on every cache reconciliation.

---

## Configuration Structure

The configuration describes a set of tileset sources.

Each source is derived from a cache resource:

### Deep Cache Source

```
./cache/<id>.pmtiles
```

### Shallow Cache Source

```
https://.../pmtiles
```

Both may coexist in the same configuration.

---

## Tileset Naming

Each tileset is named using the STAC Item ID:

```
6a18bf8e8a50e594a322d68a
```

Rationale:

- unique
- stable
- globally consistent across system layers

No human-readable alias is required.

---

## Multi-Resource Composition

When multiple resources are provided:

```
just cache <id1> <id2> <id3>
```

C.R.E.A.M. generates a combined configuration:

```
{
  "tilesets": {
    "<id1>": {...},
    "<id2>": {...},
    "<id3>": {...}
  }
}
```

Each resource is independently resolved before inclusion.

---

## Integration Flow

The integration pipeline is:

```
STAC Item ID
  ↓
Tilepack (if needed)
  ↓
PMTiles (remote or local)
  ↓
Cache reconciliation
  ↓
config.json generation
  ↓
Martin startup
  ↓
SERVING state
```

Martin is the final execution layer in the pipeline.

---

## Serving Semantics

A resource is considered SERVING when:

- Martin is running
- config.json includes the resource
- PMTiles source is reachable (local or remote)
- tile endpoint is exposed

Serving is an operational condition, not a stored state.

---

## Lifecycle Ownership

C.R.E.A.M. controls lifecycle transitions.

Martin does not persist state across runs.

Typical lifecycle:

```
cache command
  ↓
reconciliation
  ↓
config generation
  ↓
Martin start
  ↓
interactive serving session
  ↓
Ctrl-C
  ↓
termination
```

---

## Failure Handling

Martin does not recover missing resources.

If a tileset is unavailable:

- C.R.E.A.M. must resolve it
- C.R.E.A.M. must regenerate config
- Martin is restarted with corrected state

Martin is stateless with respect to recovery.

---

## Deep vs Shallow Cache Behavior

### Deep Cache Preferred

```
./cache/<id>.pmtiles
```

- fastest startup
- fully local
- deterministic

### Shallow Cache Fallback

```
remote PMTiles URL
```

- used when deep cache is absent
- depends on external availability

C.R.E.A.M. decides which source is used.

Martin only consumes the result.

---

## Auto-Start Behavior

When executing:

```
just cache <id...>
```

C.R.E.A.M. may automatically:

1. install Martin (if missing)
2. generate config.json
3. start Martin

This behavior is implicit.

It is not user-configurable in normal operation.

---

## Observability

Martin integration should produce observable logs:

```
[martin] loading config.json
[martin] tileset registered: <id>
[martin] serving on localhost
```

C.R.E.A.M. may prepend orchestration logs:

```
[cache] generating config.json
[cache] starting martin
```

---

## Operational State

Martin introduces the only runtime state in the system:

```
SERVING
```

All other states exist in C.R.E.A.M. orchestration layer.

SERVING is transient and process-bound.

---

## Design Summary

C.R.E.A.M. produces configuration.

Martin executes configuration.

```
C.R.E.A.M.
  ↓
config.json
  ↓
Martin
  ↓
tiles API
```

Authority remains with C.R.E.A.M.

Execution responsibility remains with Martin.

```
Cache Rules Everything Around Me.
```
