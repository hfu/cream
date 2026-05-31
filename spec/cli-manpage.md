# cli-manpage.md

## NAME

cream — single-command cache-driven tile serving system

---

## SYNOPSIS

```
just cache <id...>
```

---

## DESCRIPTION

C.R.E.A.M. (Cache Rules Everything Around Me) is a cache-driven system for serving geospatial tiles.

It resolves OpenAerialMap STAC items, materializes PMTiles when required, manages local and remote cache layers, and starts an interactive Martin tile server.

The system is intentionally designed so that the user interacts with only one concept:

```
cache
```

Everything else is implementation detail.

---

## CORE INTERFACE

There is exactly one primary command:

```
just cache <id...>
```

This command expresses intent, not procedure.

It means:

```
ensure the specified resources are served
```

The system determines all required steps.

---

## BEHAVIOR MODEL

The cache command performs state reconciliation toward:

```
SERVING
```

Internally, this may involve:

- STAC resolution
- PMTiles discovery
- Tilepack materialization
- Deep cache lookup
- Shallow cache fallback
- runtime configuration generation
- Martin startup

These steps are not user-facing.

---

## USER EXPERIENCE

From a user perspective:

```
just cache <id>
```

results in:

- a local tile server starts
- tiles become immediately available
- execution remains interactive
- termination occurs via Ctrl-C

---

## INTERACTIVE LIFECYCLE

```
cache command
  ↓
state reconciliation
  ↓
config generation
  ↓
Martin start
  ↓
SERVING
  ↓
Ctrl-C
  ↓
exit
```

No daemon mode is defined.

---

## MULTI-RESOURCE MODE

Multiple identifiers are supported:

```
just cache id1 id2 id3
```

This creates a single serving session with a combined tileset configuration.

Each resource is independently resolved and composed into the runtime state.

---

## FAILURE MODEL

If a resource is unavailable:

- STAC resolution may fail
- PMTiles may be materialized via Tilepack
- remote fallback may be used when possible

Failures are handled internally by the system.

User intervention is not required unless resolution is impossible.

---

## CONFIGURATION

The system generates an ephemeral runtime configuration:

```
./config.json
```

This file:

- is regenerated on every execution
- is not stored in version control
- exists only to configure the Martin runtime

---

## INTERNAL COMMANDS

The following commands exist for maintenance and diagnostics.

They are not part of the primary interface.

---

### just install

Installs runtime dependencies:

```
cargo install martin --locked
```

May be executed automatically during `just cache`.

---

### just get <id...>

Downloads PMTiles into deep cache:

```
./cache/<id>.pmtiles
```

Used for offline preparation or prefetching.

---

### just forget <id...>

Removes local cached artifacts:

```
./cache/<id>.pmtiles
```

Does not affect STAC or remote sources.

---

### just serve <id...>

Legacy compatibility command.

Equivalent in outcome to:

```
just cache <id...>
```

May bypass some reconciliation steps.

---

## DESIGN PRINCIPLE

C.R.E.A.M. enforces a single mental model:

```
cache → serve
```

The user does not manage infrastructure.

The user expresses intent.

---

## SYSTEM RESPONSIBILITY

The system is responsible for:

- dependency installation
- STAC resolution
- cache selection (deep vs shallow)
- PMTiles materialization
- configuration generation
- Martin startup
- serving lifecycle management

---

## ARCHITECTURAL MODEL

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
  ↓
SERVING
```

Authority flows upward.

Execution flows downward.

Cache is the only user-facing abstraction.

---

## PHILOSOPHY

```
Cache Rules Everything Around Me.
```

C.R.E.A.M. is designed so that a single command replaces an entire geospatial tile serving stack:

```
just cache <id>
```

This is the only concept the user needs to operate the system.
