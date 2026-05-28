# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

This is the **authoritative parent document** for the whole system — the
high-level map. It deliberately stays shallow: anything detailed lives in the
per-project docs, which this file points to. Don't duplicate those details
here; update them there and link.

## The system

Equator is one system in two layers, each a git submodule with its own repo
(`github.com:the-bokya/{dadar,nyc}.git`):

| Layer | Submodule | What it is | Start reading |
|---|---|---|---|
| Platform | `dadar/` | A distributed web **framework**: FastAPI app over an rqlite raft cluster, plus an ORM, node supervisor, and `dadar` CLI. | `dadar/CLAUDE.md` |
| Application | `nyc/` | A distributed **Firecracker microVM manager** built *on* dadar. | `nyc/README.md`, `nyc/spec.md` |

Dependency is strictly one-directional: **nyc depends on dadar; dadar knows
nothing about nyc.** dadar is a reusable library; nyc is one app consuming it —
others could be built the same way. The equator root holds no code, only the
submodule pointers (`.gitmodules`), the shared style guide (`smell.md`), and
this file.

## Shared model: what a "node" is

The one concept both layers share. A **node** is a single OS process tree in a
**node folder** on disk, running an `rqlited` subprocess (joining one shared
raft cluster — the replicated SQLite all nodes agree on) plus a `uvicorn`
FastAPI app talking to its own local rqlite. The cluster is the source of
truth; each folder is its own runtime instance even on the same host, which is
what lets `dadar stage N` / nyc's `stage.sh N` emulate an N-node cluster on one
machine. Full mechanics: `dadar/CLAUDE.md` and `dadar/src/dadar/node/spec.md`.

## Where things are documented

Read the doc nearest the code; everything below is covered there, not here.

| Topic | Doc |
|---|---|
| dadar framework, ORM, CLI, node lifecycle, downstream `DadarApp` contract | `dadar/CLAUDE.md` |
| dadar internals (orm, api, node, cli, tables) | `dadar/src/dadar/*/spec.md` |
| nyc overview, domain model, REST API, proxying, staging | `nyc/README.md`, `nyc/spec.md` |
| nyc routers / reconciler / tables | `nyc/nyc/{routers,reconciler,tables}/spec.md` |
| Firecracker client (env, vm, network, volume, lifecycle, backends) | `nyc/nyc/client/**/spec.md` |
| Coding style (shared, authoritative) | `smell.md` |

Convention worth knowing up front: **every directory carries its own
`spec.md`** that could rebuild it from scratch if the code vanished. Keep them
current with the code — they are load-bearing, not decoration.

## Working with the submodules

This is equator-specific and lives only here:

- Each submodule is a full git repo with its own history, branches, remote.
  `cd dadar` or `cd nyc` before committing — commits land in *that* repo.
- equator only tracks *which commit* of each submodule is checked out. After
  advancing one, `git add dadar` (or `nyc`) at the root and commit to move the
  pointer. A root `M dadar` status means the pointer is out of sync, not that a
  file changed.
- Fresh clone: `git submodule update --init --recursive`. Each submodule is its
  own `uv` project — `uv sync` once per submodule.

## Naming

Names are taken seriously here (`dadar`, `nyc`); each encodes what the
component does. Keep new names in the same spirit.
