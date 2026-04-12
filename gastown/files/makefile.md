---
title: Makefile
type: file
status: partial
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/Makefile
tags: [build, ldflags, makefile]
---

# Makefile

Canonical build recipe for the gastown project. Lives at
`/home/kimberly/repos/gastown/Makefile`.

## What it actually does

**Binaries produced** by the `build` target — one `go build` per binary:

1. `gt` — main CLI (see [../binaries/gt.md](../binaries/gt.md))
2. `gt-proxy-server` — purpose not yet investigated
3. `gt-proxy-client` — purpose not yet investigated

**LDFLAGS** (`/home/kimberly/repos/gastown/Makefile:18-22`):

```
-s -w \
-X github.com/steveyegge/gastown/internal/cmd.Version=$(VERSION) \
-X github.com/steveyegge/gastown/internal/cmd.Commit=$(COMMIT) \
-X github.com/steveyegge/gastown/internal/cmd.BuildTime=$(BUILD_TIME) \
-X github.com/steveyegge/gastown/internal/cmd.BuiltProperly=1
```

The critical one is `BuiltProperly=1`, which bypasses the self-kill check
at `/home/kimberly/repos/gastown/internal/cmd/root.go:96-106`. Without it,
the resulting binary exits on first run. See
[../binaries/gt.md](../binaries/gt.md) for the full story.

**VERSION, COMMIT, BUILD_TIME** (`Makefile:14-16`): derived from
`git describe --tags --always --dirty`, `git rev-parse --short HEAD`, and a
UTC timestamp respectively. These are informational; only `BuiltProperly`
gates execution.

**macOS ICU detection** (`Makefile:27-33`): auto-detects the Homebrew
`icu4c` prefix (a keg-only package) and exports `CGO_CPPFLAGS` /
`CGO_LDFLAGS` so the `go-icu-regex` transitive dependency can find headers
and libs. Not relevant on Linux — system ICU is on the default search path.

**Install directory:** `$(HOME)/.local/bin` (`Makefile:6`).

**Other targets referenced:** `desktop-build`, `desktop-run`, `install`,
`safe-install`, `check-forward-only`, `clean`, `test`, `test-e2e-container`,
`check-up-to-date`. These have not been investigated yet.

## Notes / open questions

- What do `gt-proxy-server` and `gt-proxy-client` actually do? Read
  `/home/kimberly/repos/gastown/cmd/gt-proxy-server/` and
  `/home/kimberly/repos/gastown/cmd/gt-proxy-client/`.
- Does the Homebrew formula invoke `make build`, or does it use its own
  `go build` with custom ldflags? The self-kill check is bypassed if
  `Build` is set to anything non-`"dev"`, so Homebrew could set
  `Build=Homebrew` and skip `BuiltProperly` entirely.
- Does the npm package (`@gastown/gt`) wrap a prebuilt binary, or does it
  build from source on install? If prebuilt, how are the binaries produced
  and signed?
- The `desktop-build` target produces `gt-desktop` — what is that? Another
  undocumented binary?
