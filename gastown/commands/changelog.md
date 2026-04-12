---
title: gt changelog
type: command
status: verified
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/internal/cmd/changelog.go
  - /home/kimberly/repos/gastown/internal/cmd/root.go
tags: [command, work, reporting, bd-wrapper]
---

# gt changelog

Aggregate closed beads across all rigs in the current town into a
human- or JSON-readable changelog, filtering out ephemeral/internal
beads.

**Parent:** [gt](../binaries/gt.md) (root command)
**Group:** `GroupWork` ("Work Management") (`changelog.go:30`)
**Polecat-safe:** no
**Beads-exempt:** no
**Branch-check-exempt:** no

## What it actually does

Source: `/home/kimberly/repos/gastown/internal/cmd/changelog.go:28-306`.

### Invocation

```
gt changelog [--today|--week|--since <YYYY-MM-DD>] [--rig <name>] [--json]
```

### Behavior

`runChangelog` (`changelog.go:76-96`) is a four-step pipeline:

1. **Find town root** via `workspace.FindFromCwdOrError()`
   (`changelog.go:77-80`).
2. **Determine cutoff time** via `changelogSinceTime()`
   (`changelog.go:99-121`):
   - `--since YYYY-MM-DD` parses the explicit date via
     `time.ParseInLocation("2006-01-02", ...)`.
   - `--today` uses 00:00 local today.
   - Default (no flag): current week starting at Monday 00:00 local
     (handles Sunday specifically — `weekday == 0` becomes 7, so
     Monday-start is computed correctly).
3. **Collect entries** via `collectChangelogEntries(townRoot, since)`
   (`changelog.go:124-165`):
   - Always includes the town root as a `hq` location.
   - If no `--rig` filter: loads `mayor/rigs.json` via
     `config.LoadRigsConfig` and for each rig that has a `.beads`
     directory, adds it as a location (`changelog.go:132-142`).
   - If `--rig` was given: validates that `<rig>/.beads` exists,
     otherwise errors out (`changelog.go:143-149`).
   - For each location, calls `fetchClosedBeads(dir, rig, since)`,
     collecting entries and tolerating per-location errors
     silently (`changelog.go:151-158`).
   - Sorts results descending by `ClosedAt` (`changelog.go:161-163`).
4. **Render** via `printChangelog` or `printChangelogJSON`
   (`changelog.go:92-95`).

### Fetch + filter (`fetchClosedBeads`)

`changelog.go:168-203`:

- Shells out to `bd list --status=closed --all --limit=0 --json` with
  `cmd.Dir = dir`.
- Decodes into `[]closedBead` (`changelog.go:66-74`: `ID`, `Title`,
  `IssueType`, `Ephemeral`, `ClosedAt`, `CloseReason`, `Labels`).
- For each bead:
  - Skips if `isInternalBead(b)` returns true (see below).
  - Parses `ClosedAt` as RFC3339; skips on parse failure.
  - Skips if `ClosedAt` is before the cutoff.
  - Emits a `ChangelogEntry` (`changelog.go:56-63`: `ID`, `Title`,
    `Type`, `Rig`, `ClosedAt`, `CloseReason`).

### `isInternalBead` filter (`changelog.go:206-223`)

Excludes from the changelog:

- `b.Ephemeral == true` (wisps).
- `b.IssueType == "event"`.
- Any bead whose lowercased title starts with `mol-`, `wisp-`, `plugin
  run:`, or `cost report`.

This is the "filter out noise" layer — the wisp/patrol/molecule-step
ephemera should show up in [compact](compact.md) territory, not here.

### Rendering (`printChangelog`, `changelog.go:225-273`)

- Groups entries by rig (preserving the order they first appeared in
  the sorted list).
- Prints a bold label + count per rig.
- Per entry: type icon (`🐛` bug, `✨` feature, `✓` task, `🏔` epic,
  `·` default), dim bead ID, dim `Jan 02` date, title
  (`changelog.go:246-265`).
- Footer: `Total: N issues closed across M rig(s)`.

The period header is computed by `formatPeriod` (`changelog.go:285-306`)
which distinguishes "Week of …", "Today", and "Since …" labels.

### JSON output

`printChangelogJSON` (`changelog.go:275-282`) marshals
`[]ChangelogEntry` with `json.MarshalIndent` and writes to stdout.

### Subcommands

None.

### Flags

Defined in `init()` at `changelog.go:46-53`:

| flag | type | default | description |
|------|------|---------|-------------|
| `--today` | bool | `false` | Show today's completions |
| `--week` | bool | `false` | Show this week's completions |
| `--since` | string | `""` | Show completions since date (YYYY-MM-DD) |
| `--rig` | string | `""` | Filter by rig name |
| `--json` | bool | `false` | Output as JSON |

Note: `--week` is defined but never consulted in `changelogSinceTime()`
— the week-of-Monday logic is the default, so `--week` is currently
redundant. This is worth flagging.

### Related commands

- [compact](compact.md) — handles the wisp/molecule-step ephemeral
  beads that this command filters out.
- [bead](bead.md), [cat](cat.md), [close](close.md) — the bd-wrapper
  family. `bd close` is what populates the `closed` status this
  command counts.
- [../binaries/gt.md](../binaries/gt.md) — root.

## Notes / open questions

- **`--week` is a no-op.** The flag is defined (`changelog.go:49`) but
  `changelogSinceTime` (`changelog.go:99-121`) never reads
  `changelogWeek`. Week-of-Monday is already the default. Either
  remove the flag or wire it up — neutral observation.
- **`isInternalBead` hard-codes title prefixes.** `mol-`, `wisp-`,
  `plugin run:`, `cost report` are baked into the filter
  (`changelog.go:216-221`). A configuration-driven approach would be
  cleaner — but out of scope for a mapping pass.
- **Per-rig errors silently dropped.** `collectChangelogEntries`
  swallows errors with "Non-fatal: rig may have no beads db"
  (`changelog.go:154-156`). This is fine for the happy path but
  obscures real failures.
