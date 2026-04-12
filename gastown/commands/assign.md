---
title: gt assign
type: command
status: partial
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/internal/cmd/assign.go
  - /home/kimberly/repos/gastown/internal/cmd/root.go
tags: [command, work, bd-wrapper, crew, hook]
---

# gt assign

Creates a bead and immediately hooks it to a named crew member — a
one-shot shortcut for `bd create` + `gt hook`.

**Parent:** [gt](../binaries/gt.md) (root command)
**Group:** `GroupWork` ("Work Management") (`assign.go:19`)
**Polecat-safe:** no (no `AnnotationPolecatSafe` annotation)
**Beads-exempt:** no (not in `beadsExemptCommands` on
`/home/kimberly/repos/gastown/internal/cmd/root.go:44-77`)
**Branch-check-exempt:** no

## What it actually does

Source: `/home/kimberly/repos/gastown/internal/cmd/assign.go:17-192`.

### Invocation

```
gt assign <crew-member> <title> [flags]
```

`args[0]` is a short crew name (just the leaf, e.g. `monet`), and
remaining args are joined with a space to form the bead title
(`assign.go:68-69`).

### Behavior

Five-step flow in `runAssign` at `assign.go:67-192`:

1. **Find town root** via `workspace.FindFromCwd()` (`assign.go:72-75`).
2. **Resolve rig** (`assign.go:78-88`):
   1. `--rig` flag if set.
   2. Otherwise `inferRigFromCwd(townRoot)`.
   3. Otherwise scan all rigs for a crew member matching the given name
      via `inferRigFromCrewName`. This means `gt assign dave ...` works
      from anywhere in the town if exactly one rig has a `dave`.
3. **Validate crew member exists** — the directory
   `<townRoot>/<rig>/crew/<crewName>` must exist or assign errors
   (`assign.go:91-94`). Agent ID becomes
   `<rig>/crew/<crewName>` (`assign.go:96`).
4. **Dry-run short-circuit** — `--dry-run` prints what would happen and
   exits (`assign.go:98-111`).
5. **Create bead** via `BdCmd("create", ...)` with `WithAutoCommit()`
   run in the town root (`assign.go:114-137`). Flags passed through:
   `--title`, `--type`, `--priority`, `--silent`, optional
   `--description`, and each `--label`. Output is a trimmed bead ID.
6. **Hook the bead** — runs `bd update <beadID> --status=hooked
   --assignee=<agentID>` with `WithAutoCommit()`, retrying up to 5 times
   with `slingBackoff` exponential backoff starting at 500 ms, capped at
   10 s (`assign.go:142-161`). If all 5 attempts fail, returns an error.
7. **Update agent hook_bead field** via `updateAgentHookBead` — the
   comment says "currently a no-op but maintains contract"
   (`assign.go:164-166`).
8. **Log a feed event** of type `events.TypeHook` with the bead ID as
   the hook payload (`assign.go:169-171`). A failure here only prints a
   warning.
9. **Nudge or warn** (`assign.go:176-189`) — if `--nudge` was passed,
   shells out to `gt nudge <agentID> -m "New work on your hook: <title>"`
   (`assign.go:180`). Otherwise prints "Agent won't be notified (use
   --nudge to wake them)".

### Subcommands

None.

### Flags

Defined in `init()` at `assign.go:54-65`:

| flag | short | type | default | description |
|------|-------|------|---------|-------------|
| `--description` | `-d` | string | `""` | Bead description |
| `--type` | `-t` | string | `"task"` | Bead type |
| `--priority` | `-p` | string | `"2"` | Priority 0-4 |
| `--label` | `-l` | []string | `nil` | Labels (repeatable) |
| `--nudge` | — | bool | `false` | Wake the agent after hooking |
| `--rig` | — | string | `""` | Override rig inference |
| `--dry-run` | `-n` | bool | `false` | Show what would happen |
| `--force` | — | bool | `false` | Replace existing hooked work |

Note: `--force` is defined on the flag set (`assign.go:62`) but not
directly consumed by `runAssign` in the current file — presumably
forwarded via an out-of-file helper or captured by the hook retry path.
Worth confirming.

### Related commands

- [hook](hook.md) — the hook side of the assign (`bd update
  --status=hooked --assignee=...` is what `gt hook` does too).
- [bead](bead.md) — underlying bead data plane; `bd create` is what this
  command shells out to.
- [nudge](../commands/) (not yet mapped) — optional wake on
  `--nudge`, called via `exec.Command("gt", "nudge", ...)`
  (`assign.go:180`).

## Notes / open questions

- **`--force` appears unused in-file.** The flag is defined on
  `assign.go:62` but no reference to `assignForce` appears inside
  `runAssign`. Either it's a stub, or force handling is delegated to
  `updateAgentHookBead` / the retry loop. Verify before claiming it
  doesn't do anything.
- **`slingBackoff`** — the exponential backoff helper is shared with
  `gt sling`. Defined elsewhere in the package. Worth documenting.
- **`updateAgentHookBead`** is commented as a "currently a no-op" —
  confirm whether the contract-maintenance comment still applies or
  whether the function has been fleshed out since.
- See [README.md](README.md) for the full command index and
  [../binaries/gt.md](../binaries/gt.md) for root-level context.
