# C.R.E.A.M.

Cache-driven tile serving system for OpenAerialMap STAC resources.

---

## What is this?

C.R.E.A.M. is a system that turns STAC item IDs into a running tile server.

You interact with a single command:

```
just cache <id>
```

Everything else is handled internally.

---

## Quick Start

```bash id="readme-clone"
git clone https://github.com/hfu/cream
cd cream
just cache 6a18bf8e8a50e594a322d68a
```

---

## What happens when you run it

When you execute:

```
just cache <id>
```

the system will:

- resolve STAC metadata
- locate or materialize PMTiles
- manage local cache if available
- fall back to remote sources if needed
- start a local tile server

A browser-accessible tile service is then available locally.

Stop with:

```
Ctrl-C
```

---

## Core Interface

There is only one meaningful user command:

```
just cache <id>
```

This command expresses intent, not procedure.

---

## Data Model

Input IDs are OpenAerialMap STAC Item IDs.

Example:

```
6a18bf8e8a50e594a322d68a
```

---

## System Overview (simplified)

```
STAC ID
  ↓
Tile data (remote or local)
  ↓
Cache system
  ↓
Martin tile server
```

---

## Documentation

Full system specification:

- spec/state-model.md
- spec/cache-semantics.md
- spec/stac-binding.md
- spec/tilepack-protocol.md
- spec/martin-integration.md
- spec/cli-manpage.md
- spec/consistency-check.md

---

## Philosophy

```
Cache Rules Everything Around Me.
```

---

## Notes

This system is intentionally designed so that:

- no installation steps are required in user thinking
- cache is the only interface
- all complexity is internalized into the system
