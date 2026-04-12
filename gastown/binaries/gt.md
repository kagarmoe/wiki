---
title: gt
type: binary
status: partial
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/cmd/gt/
  - /home/kimberly/repos/gastown/internal/cmd/root.go
  - /home/kimberly/repos/gastown/Makefile
  - /home/kimberly/gt/GASTOWN-README.md
tags: [cli, build, self-kill]
---

# gt

The main gastown CLI binary. Primary entry point for all gt workflows.

## What it actually does

**Version observed:** `1.0.0` (as of 2026-04-11, installed via
`go install github.com/steveyegge/gastown/cmd/gt@latest` inside the
`gt-gastown` Docker image built from `~/gt/Dockerfile`).

**Source:** `/home/kimberly/repos/gastown/cmd/gt/` in
`github.com/steveyegge/gastown`.

**Install path (go install default):** `$GOPATH/bin/gt`. In the upstream
`golang:1.26-bookworm` Docker image, `GOPATH=/go`, so the binary lands at
`/go/bin/gt` — NOT `/root/go/bin/gt` (which is what `$HOME/go/bin` would
resolve to on a typical Linux host, and what `~/gt/Dockerfile` was
originally assuming).

**Self-kill check at startup:**
`/home/kimberly/repos/gastown/internal/cmd/root.go:96-106` — if
`BuiltProperly == "" && Build == "dev"`, `gt` prints an error to stderr and
calls `os.Exit(1)`. Any command (including `gt --version` and
`gt completion bash`) hits this gate on its first action.

The check is bypassed by setting the `BuiltProperly` ldflag at build time.
The canonical recipe lives in [../files/makefile.md](../files/makefile.md):

```
-ldflags "-X github.com/steveyegge/gastown/internal/cmd.BuiltProperly=1"
```

`make build` applies this automatically. Raw `go install` /
`go build` (without the ldflag) produce binaries that refuse to run.

The Homebrew formula sets `Build` via ldflags to `"Homebrew"`, which also
bypasses the check (the condition is `BuiltProperly == "" && Build == "dev"`
— setting `Build` to anything non-`"dev"` is enough on its own).

## Docs claim

`/home/kimberly/gt/GASTOWN-README.md:141` (Installation section):

> `$ go install github.com/steveyegge/gastown/cmd/gt@latest  # From source (macOS/Linux)`

And a few lines later:

> `# If using go install, add Go binaries to PATH (add to ~/.zshrc or ~/.bashrc)`
> `export PATH="$PATH:$HOME/go/bin"`

Both claims imply `go install` produces a working binary on `macOS/Linux`
and that `$HOME/go/bin` is where the resulting binary lives.

## Drift

1. **`go install` is a broken install path.** The README tells users to
   `go install github.com/steveyegge/gastown/cmd/gt@latest`. The resulting
   binary self-kills on first run with:

   > ERROR: This binary was built with 'go build' directly.
   >        macOS will SIGKILL unsigned binaries. Use 'make build' instead.

   No workaround is mentioned in the README. The binary works only if built
   with `make build` or an equivalent
   `-ldflags "-X ...BuiltProperly=1"`.

2. **Error message misattributes the platform.** The message mentions
   `macOS will SIGKILL unsigned binaries`, but the check is unconditional
   — it fires on Linux (and presumably Windows, WSL, any OS) even though
   macOS code signing has no relevance there.

3. **Stale comment claims warning-only behavior.**
   `/home/kimberly/repos/gastown/internal/cmd/root.go:97` comment reads
   "Warning only - doesn't block execution." The code at line 106 calls
   `os.Exit(1)` — it blocks. The comment was never updated when the
   behavior changed.

4. **`$HOME/go/bin` path is environment-specific.** README says binaries
   land in `$HOME/go/bin`. In the upstream `golang:1.26-bookworm` Docker
   image (a common container base), `GOPATH=/go`, so binaries land in
   `/go/bin` instead. `~/gt/Dockerfile` was originally wrong about this
   assumption.

## Notes / open questions

- `gt` has a large command surface (`mayor`, `rig`, `crew`, `convoy`,
  `sling`, `escalate`, `feed`, `seance`, `wl`, `dashboard`, `formula`,
  `completion`, ...). None of these have been verified against the code
  yet — they are all currently unverified README claims.
- Two sibling binaries, `gt-proxy-server` and `gt-proxy-client`, are built
  by the Makefile but not documented in the README. Purpose unknown; see
  [Makefile](../files/makefile.md).
- Need to investigate: does the `brew install gastown` path pass its own
  ldflags, and does the npm package (`@gastown/gt`) wrap a prebuilt binary
  or rebuild from source?
- Runtime dependencies to verify from code: README lists `dolt`, `bd`,
  `git`, `sqlite3`, `tmux`, and optional `claude`/`codex`/`copilot` CLIs
  — which of these are actually exec'd by `gt` at runtime, and which are
  aspirational?
