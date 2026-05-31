# stac-binding.md

## Overview

This document defines the relationship between C.R.E.A.M. and STAC.

C.R.E.A.M. uses STAC as the canonical catalog layer.

STAC is not cache.

STAC is not storage.

STAC is the authoritative description of a resource.

---

## Canonical Identity

A resource is identified by a STAC Item ID.

Example:

```
6a18bf8e8a50e594a322d68a
```

Within C.R.E.A.M., this identifier is the canonical identity.

The identifier is used consistently across:

- STAC
- Tilepack
- PMTiles
- Cache
- Martin

No additional identifier translation is performed.

---

## STAC Endpoint

The canonical OpenAerialMap STAC endpoint is:

```
https://api.imagery.hotosm.org/stac/collections/openaerialmap/items/{id}
```

Example:

```
https://api.imagery.hotosm.org/stac/collections/openaerialmap/items/6a18bf8e8a50e594a322d68a
```

---

## Source of Truth

C.R.E.A.M. treats STAC as the source of truth.

The system should prefer information discovered through STAC over information obtained from implementation-specific APIs.

Example:

Preferred:

```
STAC
  → assets.pmtiles.href
```

Less preferred:

```
Tilepack response
  → url
```

Both may refer to the same PMTiles archive.

The STAC representation is considered authoritative.

---

## PMTiles Discovery

A PMTiles archive becomes discoverable through the STAC Item.

Before materialization:

```
assets.pmtiles
```

may not exist.

After materialization:

```json
"pmtiles": {
  "href": "...",
  "type": "application/vnd.pmtiles"
}
```

appears within the STAC Item.

This transition is observable through STAC.

---

## Mutable Catalog State

STAC metadata is generally descriptive.

However, PMTiles availability introduces observable state.

Example:

```
PMTiles absent
```

and

```
PMTiles present
```

represent distinct resource conditions.

C.R.E.A.M. therefore treats the STAC Item as a state-bearing document.

---

## Binding Rules

### Rule 1

A STAC Item ID uniquely identifies a cache resource.

---

### Rule 2

A STAC Item ID is used as the Martin tileset name.

Example:

```
6a18bf8e8a50e594a322d68a
```

No human-readable alias is required.

---

### Rule 3

A Deep Cache artifact uses the STAC Item ID as filename.

Example:

```
./cache/6a18bf8e8a50e594a322d68a.pmtiles
```

---

### Rule 4

A cache lookup always begins with a STAC Item ID.

Example:

```
just cache 6a18bf8e8a50e594a322d68a
```

---

## Asset Selection

When multiple assets exist, C.R.E.A.M. evaluates them according to cache needs.

Preferred order:

```
local PMTiles
  ↓
STAC PMTiles asset
  ↓
materialization through Tilepack
```

This ordering follows the cache-first principle.

---

## Relationship to Tilepack

Tilepack is not a source of truth.

Tilepack is a materialization mechanism.

Example:

```
STAC Item
  ↓
Tilepack
  ↓
PMTiles
  ↓
STAC updated
```

After materialization, future discovery should occur through STAC.

---

## Relationship to Cache

STAC describes resources.

Cache owns resources.

This distinction is fundamental.

STAC answers:

```
What exists?
```

Cache answers:

```
What is available?
```

C.R.E.A.M. is primarily concerned with availability.

---

## Design Summary

```
STAC
  ↓ describes

PMTiles
  ↓ stored by

Cache
  ↓ served by

Martin
```

STAC provides identity.

Cache provides availability.

The two concepts must remain separate.

---

## Design Principle

The authoritative description of a resource belongs to STAC.

The authoritative availability of a resource belongs to Cache.

This separation allows C.R.E.A.M. to remain cache-first while continuing to rely on STAC as a shared catalog.

```
Cache Rules Everything Around Me.
```
