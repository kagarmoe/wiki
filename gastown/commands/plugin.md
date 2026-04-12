---
title: gt plugin
type: command
status: verified
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/internal/cmd/plugin.go
  - /home/kimberly/repos/gastown/internal/cmd/root.go
tags: [command, configuration, plugins, deacon, patrol]
---

# gt plugin

Manages plugins that run during Deacon patrol cycles. Plugins are
periodic automation tasks defined by `plugin.md` files with TOML
frontmatter, discovered from `<townRoot>/plugins/` and from each
`<rig>/plugins/` directory.

**Parent:** [gt](../binaries/gt.md) (root command)
**Group:** `GroupConfig` ("Configuration")
**Polecat-safe:** no (no `AnnotationPolecatSafe` on the cobra.Command at
`/home/kimberly/repos/gastown/internal/cmd/plugin.go:32-56`)
**Beads-exempt:** no (not in `beadsExemptCommands` on `root.go:44-77`).
Note that `plugin history` records runs as ephemeral wisp beads
(`plugin.go:476-487`), so the command actively requires beads to be
available despite not being explicitly exempt.
**Branch-check-exempt:** no (not in `branchCheckExemptCommands` on
`root.go:81-91`)

## What it actually does

Source: `/home/kimberly/repos/gastown/internal/cmd/plugin.go:32-630`.

### Invocation

```
gt plugin list [--json]
gt plugin show <name> [--json]
gt plugin run <name> [--force] [--dry-run]
gt plugin sync [--source DIR] [--clean] [--dry-run]
gt plugin history <name> [--json] [--limit N]
```

Parent `pluginCmd` uses `RunE: requireSubcommand`
(`plugin.go:55`) — bare invocation returns an error.

### Plugin model

From the Long text (`plugin.go:36-54`):

- **Locations**: `~/gt/plugins/` (town-level, universal) and
  `<rig>/plugins/` (rig-specific, takes precedence over town).
- **Gate types**: `cooldown`, `cron`, `condition`, `event`, `manual`.
- **Execution**: `printPluginSummary` (`plugin.go:277-297`) marks
  plugins with `p.IsExecWrapper()` as `[exec-wrapper]` instead of the
  gate type — implying some plugins wrap other binaries rather than
  running as pure instructions.

### `getPluginScanner` helper (`plugin.go:169-191`)

Shared by every subcommand except `sync`. Builds a
`plugin.Scanner` by:

1. `workspace.FindFromCwdOrError()` — fail if not in a workspace.
2. Loading the rigs config from `constants.MayorRigsPath(townRoot)`
   via `config.LoadRigsConfig`. Failures are non-fatal (synthesizes
   an empty config).
3. Extracting and sorting rig names.
4. `plugin.NewScanner(townRoot, rigNames)`.

### Subcommands

#### `list` (`plugin.go:58-73`, `runPluginList` at `:193-214`)

1. `scanner.DiscoverAll()` walks town + rig plugin directories.
2. Sorts by name.
3. **JSON mode** (`plugin.go:209-211`, `outputPluginListJSON`
   `:216-225`): maps each `plugin.Plugin` to a `plugin.PluginSummary`
   via `p.Summary()`, encodes as JSON.
4. **Text mode** (`outputPluginListText` `:227-275`): prints
   `● Discovered N plugin(s)`, groups by `LocationTown` vs rig,
   prints bold section headers and calls `printPluginSummary` per
   plugin.

#### `show <name>` (`plugin.go:75-87`, `runPluginShow` at `:299-317`)

1. `scanner.GetPlugin(name)` (fails with error if not found).
2. JSON → encode the full plugin struct.
3. Text (`outputPluginShowText` `:325-410`): prints bold-labeled
   fields: Plugin, Path, Description, Location (plus rig name if
   rig-scoped), Version, Gate (Type + any of Duration/Schedule/
   Check/On), Tracking (Labels, Digest), Execution (Type, Wrapper,
   Timeout, NotifyOnFailure, Severity), and the first 10 lines of
   `Instructions`.

#### `run <name>` (`plugin.go:89-103`, `runPluginRun` at `:412-490`)

1. Fetch the plugin via the shared scanner.
2. **Gate check** (`plugin.go:425-442`): only implemented for
   `GateCooldown`. Instantiates `plugin.NewRecorder(townRoot)` and
   calls `recorder.CountRunsSince(p.Name, duration)` to see if any
   runs exist within the cooldown window. Defaults duration to
   `"1h"` if blank.
3. **`--dry-run`** (`plugin.go:444-456`): prints what would happen
   based on gate status, returns.
4. **Gate-closed short circuit** (`plugin.go:458-462`): if cooldown
   gate is closed and `--force` not set, print warning and return
   nil.
5. **Execute**: prints `● Running plugin: <name>` and echoes
   `p.Instructions` to stdout for the agent/user to execute
   (`plugin.go:464-473`). Important code comment at `:465-466`:
   > For manual runs, we print the instructions for the agent/user to
   > execute. Automatic execution via dogs is handled by gt-n08ix.2
6. **Records the run** as an ephemeral wisp bead via
   `recorder.RecordRun(PluginRunRecord{Result: ResultSuccess, ...})`
   (`plugin.go:476-487`). The run is **always** marked Success —
   there is no failure reporting for manual runs.

#### `sync` (`plugin.go:105-121`, `runPluginSync` at `:492-574`)

Copies plugin directories from a source into
`<townRoot>/plugins/`:

1. Determines source: explicit `--source`, or
   `plugin.FindGastownSource(townRoot)` if omitted
   (`plugin.go:498-505`).
2. Resolves to absolute path (`plugin.go:508-514`).
3. **`--dry-run`** (`plugin.go:518-545`): `plugin.DetectDrift(source,
   target)` returns drifted, missing, and extra files; prints with
   `~ + -` markers (extras only shown with `--clean`).
4. **Live sync** (`plugin.go:547-573`): `plugin.SyncPlugins(source,
   target, clean)` returns Copied / Removed / Skipped / Errors.
   Prints per-file `↑` or `×` markers, a summary line, and any errors
   to stderr.

#### `history <name>` (`plugin.go:123-136`, `runPluginHistory` at `:576-630`)

1. Calls `recorder.GetRunsSince(name, "")` — empty duration means
   "all history".
2. Applies `--limit` client-side (`plugin.go:595-597`).
3. JSON mode encodes the full `PluginRunBead` list.
4. Text mode prints `● Execution history for <name> (N runs)` and
   per-run lines with a ✓ / ✗ / ○ marker depending on `run.Result`
   (`plugin.go:612-627`), timestamp, and dim bead ID.

### Flags

Declared in `init()` (`plugin.go:138-156`):

| Flag                | Scope              | Type   | Default | Purpose                                              |
|---------------------|--------------------|--------|---------|------------------------------------------------------|
| `--json`            | `list`/`show`/`history` | bool   | `false` | JSON output                                          |
| `--force`           | `run`              | bool   | `false` | Bypass gate check                                    |
| `--dry-run`         | `run` and `sync`   | bool   | `false` | Show what would happen                               |
| `--source`          | `sync`             | string | `""`    | Source plugins directory (auto-detected if omitted)  |
| `--clean`           | `sync`             | bool   | `false` | Remove target plugins that don't exist in source     |
| `--limit`           | `history`          | int    | `10`    | Max runs to show                                     |

## Related

- [gt](../binaries/gt.md) — parent binary.
- [README.md](README.md) — command tree index.
- [doctor.md](doctor.md) — `doctor.go:218-226` registers
  `PatrolPluginsAccessible` and `PatrolPluginDrift` health checks
  that read the same plugin layout. `PatrolPluginDrift` in
  particular overlaps with `plugin sync --dry-run`.
- [config.md](config.md) — sibling Configuration command. Plugin
  config is file-based (`plugin.md`) rather than in
  `settings/config.json`, so `config` and `plugin` operate on
  disjoint storage.
- [hooks.md](hooks.md) — both manage "things that run on events"
  but hooks target Claude Code `SessionStart`/etc., while plugins
  target Deacon patrol cycles. Different systems, different storage.
- [status.md](status.md) — status is likely where a user eyeballs
  which plugins have run recently; unverified.
- `gt-n08ix.2` — the in-code reference at `plugin.go:465-466`
  pointing at the automatic execution code path. Worth capturing as
  a bead reference.

## Notes / open questions

- **`run` always records Success** (`plugin.go:480`,
  `Result: plugin.ResultSuccess, // Manual runs are marked success`).
  If the printed instructions fail when the agent executes them,
  there is no code path from `gt plugin run` back to the recorder
  to amend the record. History will be falsely optimistic. Noted
  neutrally.
- **Gate semantics beyond cooldown are not enforced by `run`.**
  Only `GateCooldown` is checked (`plugin.go:428`). `cron`,
  `condition`, `event`, `manual` all fall through `gateOpen = true`
  and the plugin runs unconditionally. This may be intentional for
  the manual `gt plugin run` path (the user is explicitly bypassing
  gates) but is not documented.
- **Sync auto-detection** (`plugin.FindGastownSource`) is a black
  box from this file. A `packages/plugin.md` page should enumerate
  the search order and fallback rules.
- Plugin `instructions` are markdown body; the model that
  interprets them (Deacon? per-agent?) should be captured in a
  `concepts/plugin.md` page.
- The `Exec` wrapper type (`plugin.IsExecWrapper`) is advertised in
  list output but not explained here; worth chasing into
  `internal/plugin`.
