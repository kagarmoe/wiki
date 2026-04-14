---
title: internal/keepalive
type: package
status: verified
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/internal/keepalive/keepalive.go
tags: [package, diagnostics, keepalive, daemon, sentinel-pattern]
---

# internal/keepalive

Best-effort agent-activity signaling via a single JSON file at
`<townRoot>/.runtime/keepalive.json`. Every `gt` invocation touches this
file so the daemon can tell whether an agent is actively running commands
or idle.

**Go package path:** `github.com/steveyegge/gastown/internal/keepalive`
**File count:** 1 go file, 1 test file
**Imports (notable):** stdlib only (`encoding/json`, `os`, `path/filepath`,
`strings`, `time`) plus [`internal/workspace`](workspace.md) for town-root
discovery.
**Imported by:** only `internal/web/api.go` at present. Despite the
package existing, the expected `gt`-root persistent prerun integration
that would call `Touch` on every command does not currently import this
package (see "Notes / open questions"). So "every `gt` invocation touches
this file" is the intended design; what the code actually does today is
narrower.

## What it actually does

Source: `/home/kimberly/repos/gastown/internal/keepalive/keepalive.go`.

### Public API

**Type:**

- `State struct { LastCommand string; Timestamp time.Time }`
  (`keepalive.go:46-49`) — the wire format persisted as
  `keepalive.json`. `last_command` is a free-form string, not a
  structured `{cmd, args}` tuple.

**Touch functions (writer side):**

- `Touch(command string)` (`keepalive.go:53-55`) — thin convenience
  wrapper that calls `TouchWithArgs(command, nil)`.
- `TouchWithArgs(command string, args []string)`
  (`keepalive.go:59-72`) — resolves the town root via
  `workspace.FindFromCwd`, silently returns when not inside a workspace,
  and then builds a space-joined `"command arg1 arg2 ..."` string before
  delegating to `TouchInWorkspace`. No shell-quoting.
- `TouchInWorkspace(workspaceRoot, command string)`
  (`keepalive.go:76-96`) — ensures `<root>/.runtime/` exists, marshals
  `State{LastCommand: command, Timestamp: time.Now().UTC()}`, and
  `os.WriteFile`s it to `<root>/.runtime/keepalive.json` with mode
  `0644`. Every error path is silently swallowed with `return` — no
  logging, no panic, no error return. Comment at `keepalive.go:95`:
  "non-fatal: status file for debugging".

**Read side:**

- `Read(workspaceRoot string) *State` (`keepalive.go:103-117`) — reads
  and unmarshals `<root>/.runtime/keepalive.json`. **Returns `nil` (not
  an error) when the file doesn't exist, can't be read, or contains
  invalid JSON.** This is the nil-sentinel pattern documented at length
  in the package doc comment.
- `(s *State) Age() time.Duration` (`keepalive.go:132-137`) — `nil`-
  receiver-safe. On a non-nil `State` it returns `time.Since(s.Timestamp)`.
  **On a nil receiver it returns `24 * time.Hour * 365`**, a sentinel
  "maximally stale" value chosen specifically to exceed any practical
  idle threshold without risking `MaxInt64` overflow arithmetic.

### Internals / Notable implementation

- **The nil-sentinel pattern is the load-bearing design choice.** The
  package doc (`keepalive.go:14-32`) explicitly motivates it: daemon code
  wants to write `if keepalive.Read(root).Age() > 5*time.Minute { ... }`
  without a nil guard, treating "no keepalive" and "stale keepalive" as
  the same idle condition. Returning `365 days` from `Age()` on a nil
  receiver makes that idiom compile and do the right thing.
- **All writes are fire-and-forget.** The rationale at `keepalive.go:5-9`:
  keepalive signals are non-critical; failures (disk full, permission
  errors) should not interrupt `gt` commands; the daemon already tolerates
  missing signals via timeout. This is why `TouchInWorkspace` has three
  separate silent-`return` error paths and one `_ = os.WriteFile(...)`
  explicit discard.
- **Timestamps are UTC** (`keepalive.go:86`). The daemon's consumer side
  must also work in UTC to avoid tz-drift false positives.

### Usage pattern

Writer side: any CLI command or daemon-adjacent tool that wants to assert
"an agent was active" calls `keepalive.Touch("command name")`. The
intended integration point is the root `gt` cobra `PersistentPreRun`, so
that **every** subcommand implicitly signals activity — but as of this
ingest the only in-tree importer is `internal/web/api.go`.

Reader side: the daemon loop (seen in `internal/daemon/*` and
`internal/doctor/*` — not yet mapped) would read the state file on a
tick, compute `Age()`, and use the result as input to exponential-backoff
logic during idle periods.

## Related wiki pages

- [gt](../binaries/gt.md) — the binary expected to call `Touch` on every
  invocation.
- [internal/workspace](workspace.md) — used to locate the town root
  before writing the keepalive file.
- [internal/doctor](doctor.md) — the diagnostic package that reports
  agent-idle conditions; `Age()` is designed to feed directly into
  doctor-style threshold checks.
- [gt doctor](../commands/doctor.md), [gt health](../commands/health.md),
  [gt vitals](../commands/vitals.md) — sibling diagnostic CLI surfaces
  that expose agent-activity state to the user.
- [go-packages inventory](../inventory/go-packages.md).

## Notes / open questions

- **Why only `web/api.go`?** `Touch` is designed to be called from every
  `gt` invocation via a cobra `PersistentPreRun` wire-up, but the grep
  for `"github.com/steveyegge/gastown/internal/keepalive"` finds only
  `internal/web/api.go`. Either the preRun integration lives outside a
  direct import path (via a small shim in another package), or the
  integration never landed. Worth checking when mapping `internal/cmd`
  root wire-up.
- **Who actually consumes `Read`?** The daemon-side consumer of the
  365-day sentinel isn't in this package. Its real user must live in
  `internal/daemon/` or `internal/doctor/` — verify when those pages
  are built out.
- The full-command string built by `TouchWithArgs` is not shell-quoted
  and is stored as-is. Arguments containing spaces or newlines will
  corrupt the `last_command` field. Acceptable because this field is
  documented as debugging-only, but worth noting if anyone ever tries
  to reconstruct invocations from it.
