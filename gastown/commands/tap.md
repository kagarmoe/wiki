---
title: gt tap
type: command
status: verified
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/internal/cmd/tap.go
tags: [command, ungrouped, beads-exempt, hook, claude-code, policy]
---

# gt tap

Parent command group for Claude Code hook handlers that tap into tool
execution — `guard` (implemented), plus planned `audit` / `inject` /
`check` subcommands. Called by Claude's PreToolUse/PostToolUse hooks.

**Parent:** [gt](../binaries/gt.md) (root command)
**Group:** (none — no `GroupID` set on the cobra.Command definition)
**Hidden:** no
**Polecat-safe:** no
**Beads-exempt:** yes (in `beadsExemptCommands` on
`/home/kimberly/repos/gastown/internal/cmd/root.go:44-77`; tap is
excluded from command-usage logging so its own invocation from inside
a PreToolUse hook doesn't recurse into `bd` writes)
**Branch-check-exempt:** no

## What it actually does

Source: `/home/kimberly/repos/gastown/internal/cmd/tap.go:7-36`.

### Invocation

```
gt tap guard <policy-name>           # PreToolUse, exits 2 to block
gt tap audit ...                     # planned (PostToolUse)
gt tap inject ...                    # planned (PreToolUse, transform)
gt tap check ...                     # planned (PostToolUse)
```

`Use: "tap"` (`tap.go:8`). The parent command has no `RunE` — it is a
pure subcommand group. In this file there is only the parent command:
the `guard` / `audit` / `inject` / `check` subcommands are registered
from other files (likely `tap_guard.go`, `tap_audit.go`, …).

### Subcommand concept (from `Long` help, `tap.go:10-29`)

> Hook handlers for Claude Code PreToolUse and PostToolUse events.
>
> These commands are called by Claude Code hooks to implement policies,
> auditing, and input transformation. They tap into the tool execution
> flow to guard, audit, inject, or check.
>
> Subcommands:
>   guard   - Block forbidden operations (PreToolUse, exit 2)
>   audit   - Log/record tool executions (PostToolUse) [planned]
>   inject  - Modify tool inputs (PreToolUse, updatedInput) [planned]
>   check   - Validate after execution (PostToolUse) [planned]

### Example Claude Code settings integration

From the `Long` help (`tap.go:22-28`):

```json
{
  "PreToolUse": [{
    "matcher": "Bash(gh pr create*)",
    "hooks": [{"command": "gt tap guard pr-workflow"}]
  }]
}
```

### Flags

None on the parent. Subcommand flags are declared in their respective
files (not in this batch).

### Why beads-exempt

Per the batch brief: `tap` is "excluded from command-usage logging per
`persistentPreRun`". The beads-exempt map on `root.go:44-77` is what
root-command dispatch checks before running per-command telemetry and
`bd` write paths; `tap` is in it. Semantically: a PreToolUse hook
should be fast and side-effect-free, and should not itself enqueue
more Claude-visible tool-use events or stall on `bd` I/O.

### Subcommands

Registered from sibling files (not in `tap.go`):

- **`gt tap guard <policy>`** — blocks forbidden operations. Exits 2
  (Claude Code's convention for a PreToolUse veto that should surface
  the guard's stderr to the user).
- **`gt tap audit`**, **`gt tap inject`**, **`gt tap check`** — planned
  per the help text. Not implemented in the current tree.

## Related commands

- [hooks](hooks.md) — the Claude Code hooks *manager* (plural). Where
  hooks get installed into `.claude/settings.json` — i.e. the command
  that configures tap handlers to actually fire. Distinct namespace
  from `tap` despite the name overlap.
- [signal](signal.md), [metrics](metrics.md) — sibling low-level tmux /
  observability primitives per the batch brief's cross-link hint.
- [guard](guard.md) — if a top-level `gt guard` exists, it would be a
  separate surface — but per the inventory this is subcommand-only
  (`gt tap guard ...`).
- [audit](audit.md) — a *different* audit command at the top level
  (reads past events); not the same as the planned `gt tap audit`
  (which would write events on PostToolUse).
- [prime](prime.md) — session-start hook; sibling Claude-integration
  command.
- [../binaries/gt.md](../binaries/gt.md) — root.

## Notes / open questions

- **Parent-only in this file.** `tap.go` defines only the parent
  cobra.Command; the only implemented subcommand (`guard`) is
  registered from a sibling file. Worth mapping `tap_guard.go` (or
  whatever it's named) as its own entity page in a future batch.
- **Three subcommands are planned and never landed.** The `Long` help
  advertises `audit`, `inject`, and `check` as "[planned]." Anyone
  writing hooks today only has `guard`. Worth tracking whether the
  plan still holds.
- **Beads-exempt matters for correctness, not just speed.** A
  PreToolUse hook that wrote to `bd` while the user was mid-tool-call
  could cause reentrancy — the tool the user is running might itself
  emit `bd` writes that the tap hook would try to audit. The exemption
  is the cheap fix.
- **Where does `gt tap guard`'s exit code go?** Claude Code treats
  exit 2 as a veto from PreToolUse and surfaces the hook's stderr to
  the model. The `guard` subcommand (in a sibling file) is what
  actually implements the policy lookup and exit-2 path.
- **`~/gt/docs/HOOKS.md`** is referenced on `tap.go:30` — worth
  ingesting as a doc source when the wiki expands to cover docs.
