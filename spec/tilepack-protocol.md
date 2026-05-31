# tilepack-protocol.md

## Overview

This document defines how C.R.E.A.M. interacts with the Tilepack API.

C.R.E.A.M. (Cache Rules Everything Around Me) is a cache-first orchestration system.

The primary user operation is:

```
just cache <id...>
```

All other operations exist to support the desired cache state.

Tilepack is treated as a PMTiles materialization service within the cache lifecycle.

---

## Scope

This document describes:

- Tilepack job initiation
- Tilepack job progression
- PMTiles materialization
- Discovery of PMTiles through STAC

This document does not describe:

- Martin configuration
- Cache reconciliation
- Runtime serving behavior

Those are specified elsewhere.

---

## Resource Identity

A resource is identified by an OpenAerialMap STAC Item ID.

Example:

```
6a18bf8e8a50e594a322d68a
```

This identifier is the canonical identity throughout the system.

It is used as:

- Tilepack input
- STAC Item lookup key
- Cache key
- Martin tileset name

---

## Endpoint

```
POST https://packager.imagery.hotosm.org/tilepacks/{id}?format=pmtiles
```

Example:

```
POST https://packager.imagery.hotosm.org/tilepacks/6a18bf8e8a50e594a322d68a?format=pmtiles
```

---

## Request Semantics

The request asks Tilepack to ensure that a PMTiles archive exists for the specified STAC Item.

Repeated requests are expected and normal.

The endpoint behaves as an asynchronous state machine.

---

## Tilepack State Model

```
NOT_STARTED
  ↓
STARTED
  ↓
IN_PROGRESS
  ↓
READY
```

---

## STARTED

Example:

```json
{
  "status": "started"
}
```

Meaning:

- Request accepted
- Packaging job created

---

## IN_PROGRESS

Example:

```json
{
  "status": "in_progress"
}
```

Meaning:

- Packaging is currently running

---

## READY

Example:

```json
{
  "status": "ready",
  "url": "https://...pmtiles"
}
```

Meaning:

- PMTiles archive exists
- PMTiles URL is available

---

## Materialization Contract

When Tilepack reaches READY:

- A PMTiles archive exists
- A PMTiles URL is returned
- The PMTiles archive becomes discoverable through STAC

---

## STAC Integration

The canonical STAC Item endpoint is:

```
https://api.imagery.hotosm.org/stac/collections/openaerialmap/items/{id}
```

Example:

```
https://api.imagery.hotosm.org/stac/collections/openaerialmap/items/6a18bf8e8a50e594a322d68a
```

Before PMTiles materialization:

```
assets.pmtiles
```

does not exist.

After PMTiles materialization:

```json
"pmtiles": {
  "href": "...",
  "type": "application/vnd.pmtiles",
  "roles": ["tiles"]
}
```

appears within the STAC Item.

---

## Discovery Rule

C.R.E.A.M. treats STAC as the source of truth.

The PMTiles URL returned by Tilepack may be used immediately.

However, long-term discovery should occur through the STAC Item.

Preferred flow:

```
STAC Item
  → PMTiles asset
  → PMTiles URL
```

rather than:

```
Tilepack response
  → PMTiles URL
```

---

## Relationship to Cache

Tilepack itself is not cache.

Tilepack only materializes PMTiles artifacts.

Within C.R.E.A.M.:

```
STAC Item
  → Tilepack
  → PMTiles
  → Cache
  → Martin
```

Tilepack is therefore a dependency of cache, not cache itself.

---

## Deep Cache

A PMTiles archive stored locally under:

```
./cache/
```

is considered a Deep Cache artifact.

Example:

```
./cache/6a18bf8e8a50e594a322d68a.pmtiles
```

Tilepack may be used to obtain such artifacts.

However, Deep Cache is managed by C.R.E.A.M., not by Tilepack.

---

## Design Principle

Tilepack does not control the system.

STAC does not control the system.

Martin does not control the system.

Cache controls the system.

Tilepack exists to support cache creation.

The existence of a PMTiles archive is valuable only insofar as it contributes to the desired cache state.

This principle is summarized by the project name:

```
C.R.E.A.M.
Cache Rules Everything Around Me.
```

---

## Observed Session

```
POST /tilepacks/6a18bf8e8a50e594a322d68a?format=pmtiles
→ started

POST /tilepacks/6a18bf8e8a50e594a322d68a?format=pmtiles
→ in_progress

POST /tilepacks/6a18bf8e8a50e594a322d68a?format=pmtiles
→ ready
→ PMTiles URL returned

GET STAC Item
→ pmtiles asset appears
```
