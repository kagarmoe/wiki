---
title: gastown topic landing
type: note
status: stub
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/
tags: [landing, index]
---

# gastown

Topic root for the gastown project.

- **Source under investigation:** `/home/kimberly/repos/gastown/`
- **Upstream:** https://github.com/gastownhall/gastown
- **Go module path declared in `go.mod`:** `github.com/steveyegge/gastown`
  (the `gastownhall` repo self-identifies under the `steveyegge` namespace,
  so `go install` must use the `steveyegge` path)

## Scope

Gastown is a multi-agent orchestration system for AI coding runtimes,
managed by the `gt` CLI. This topic tracks:

- The `gt` binary and its CLI surface
- Build system, Makefile recipes, Docker build stages
- Docker runtime environment (entrypoint, compose, volumes)
- Internal packages, roles, services, workflows
- Drift between documentation and code reality

## Investigation state

**Current focus:** getting `gt` to build and run inside Docker at `~/gt/`.
`/home/kimberly/repos/gastown/` is reference only — we read from it but
never build or run anything there.

**Known drift (as of 2026-04-11):**

1. README's `go install` instructions produce a self-killing binary. Must
   use `make build` or pass `-ldflags` with `BuiltProperly=1`. See
   [gt](binaries/gt.md).
2. Makefile builds three binaries (`gt`, `gt-proxy-server`,
   `gt-proxy-client`); README only documents `gt`. See
   [Makefile](files/makefile.md).
3. Error message at `/home/kimberly/repos/gastown/internal/cmd/root.go:101`
   claims "macOS will SIGKILL" but the check fires on Linux too.
4. Stale comment at
   `/home/kimberly/repos/gastown/internal/cmd/root.go:97` claims the
   self-kill check is "Warning only - doesn't block execution" — but the
   code below it calls `os.Exit(1)`.

## Sub-index

### Binaries

- [gt](binaries/gt.md) — main CLI

### Files

- [Makefile](files/makefile.md) — canonical build recipe
