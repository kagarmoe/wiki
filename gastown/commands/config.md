---
title: gt config
type: command
status: verified
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/internal/cmd/config.go
  - /home/kimberly/repos/gastown/internal/cmd/root.go
tags: [command, configuration, town-settings, agents, scheduler, lifecycle, cost-tier]
---

# gt config

Manages Gas Town town-scoped configuration: agent aliases, default
agent, cost-optimization tier, agent email domain, and a broad set of
dot-notation keys (scheduler, convoy, lifecycle, maintenance, Dolt
port, CLI theme). The single largest file in the Configuration group
at 1291 lines.

**Parent:** [gt](../binaries/gt.md) (root command)
**Group:** `GroupConfig` ("Configuration")
**Polecat-safe:** no (no `AnnotationPolecatSafe` on the cobra.Command at
`/home/kimberly/repos/gastown/internal/cmd/config.go:21-38`)
**Beads-exempt:** **yes** — `config` is in `beadsExemptCommands` at
`/home/kimberly/repos/gastown/internal/cmd/root.go:64`; runs without
requiring `bd` to be installed, so a broken beads install can still
be repaired via `gt config`.
**Branch-check-exempt:** no (not in `branchCheckExemptCommands` on
`root.go:81-91`)

## What it actually does

Source: `/home/kimberly/repos/gastown/internal/cmd/config.go:21-1291`.

### Invocation

```
gt config agent list [--json]
gt config agent get <name>
gt config agent set <name> <command> [--provider <preset>]
gt config agent remove <name>
gt config default-agent [<name>]
gt config default-agent list [--json]
gt config agent-email-domain [<domain>]
gt config cost-tier [<tier>]
gt config set <key> <value>
gt config get <key>
```

Parent `configCmd` uses `RunE: requireSubcommand`
(`config.go:25`) — bare invocation returns an error.

### Subcommand tree

Registration in `init()` (`config.go:1257-1291`):

```
config
├── agent            (intermediate group; defined inline at :1264-1272)
│   ├── list
│   ├── get <name>
│   ├── set <name> <command> [--provider]
│   └── remove <name>
├── cost-tier [tier]
├── default-agent [name]
│   └── list
├── agent-email-domain [domain]
├── set <key> <value>
└── get <key>
```

**Unusual structure**: the `configAgentCmd` parent-group is not a
package-level `var` like the others; it is constructed inline
inside `init()` at `config.go:1264-1272`. All other parent commands
in this file (`configCmd`, `configDefaultAgentCmd`) are top-level
vars.

### Storage

Everything in this command reads/writes town-level settings via
`config.TownSettingsPath(townRoot)` →
`config.LoadOrCreateTownSettings` / `config.SaveTownSettings`. Every
subcommand begins with `workspace.FindFromCwd()` to resolve
`townRoot`. Exceptions:

- `dolt.port` under `config set` routes to a separate
  `daemon.DaemonPatrolConfig` file via `daemon.LoadPatrolConfig` and
  `daemon.SavePatrolConfig` (`config.go:802-830` — inferred from the
  surrounding context; see Notes).
- `maintenance.*` keys dispatch to `setMaintenanceConfig(townRoot,
  key, value)` (`config.go:919`) which presumably writes a different
  file.

### Subcommands

#### `agent list` (`config.go:42-54`, `runConfigAgentList` at `:268-355`)

1. Loads town settings and calls `config.LoadAgentRegistry(
   DefaultAgentRegistryPath(townRoot))` (`config.go:281-285`).
2. Collects built-in presets via `config.ListAgentPresets()` and
   custom agents from `townSettings.Agents`.
3. Builds `[]AgentListItem` (`config.go:259-266`) with Name,
   Command, Args, Type ("built-in" or "custom"), IsCustom.
4. Sorts by name, outputs as JSON (`--json` flag read via
   `cmd.Flags().GetBool("json")`) or text.

#### `agent get <name>` (`config.go:56-69`, `runConfigAgentGet` at `:357`)

Shows the full configuration for a single agent — built-in or
custom — including command, args, and other settings. Implementation
details beyond the declaration were not read in full for this page.

#### `agent set <name> <command>` (`config.go:71-95`, `runConfigAgentSet` at `:425`)

Writes a custom agent definition into `townSettings.Agents`. The
command can include arguments; provider preset is inferred from the
binary basename unless `--provider` is supplied explicitly
(`config.go:83-91`). The provider preset controls session handling,
tmux detection, hooks, and other runtime defaults — see the Long
text.

#### `agent remove <name>` (`config.go:97-109`, `runConfigAgentRemove` at `:491`)

Removes a custom agent from `townSettings.Agents`. Built-in agents
(`claude`, `gemini`, `codex`) cannot be removed — per the Long text,
though the enforcement code was not read line-by-line.

#### `cost-tier [tier]` (`config.go:113-132`, `runConfigCostTier` at `:134-188`)

1. Loads town settings.
2. **No args** (`config.go:146-159`): reads the current tier via
   `config.GetCurrentTier(townSettings)`. If blank, reports
   `Cost tier: custom (manual role_agents configuration)` — meaning
   `role_agents` has been manually populated without matching a
   preset. Otherwise prints the tier name, description, and role
   assignment table.
3. **With arg** (`config.go:162-187`): validates via
   `config.IsValidTier(tierName)`, warns if overwriting a custom
   `role_agents` setup, calls `config.ApplyCostTier(townSettings,
   tier)`, saves. Prints confirmation + role table.

Tier presets per Long text (`config.go:121-124`):
- `standard` — all roles Opus (default)
- `economy` — patrol roles Sonnet/Haiku, workers Opus
- `budget` — patrol roles Haiku, workers Sonnet

#### `default-agent [name]` (`config.go:192-213`, `runConfigDefaultAgent` at `:531`)

Gets or sets `townSettings.DefaultAgent`. Used by rigs that don't
specify their own agent.

#### `default-agent list` (`config.go:215-227`, delegates to `runConfigAgentList`)

An alias for `config agent list` — same implementation, separate
flag variable `configDefaultAgentListJSON`.

#### `agent-email-domain [domain]` (`config.go:232-251`, `runConfigAgentEmailDomain` at `:594`)

Gets or sets `townSettings.AgentEmailDomain` — the domain used when
`gt commit` builds a git commit email from an agent identity. E.g.,
`gastown/crew/jack` becomes `gastown.crew.jack@<domain>`. Default is
`gastown.local`.

#### `set <key> <value>` (`config.go:643-687`, `runConfigSet` at `:728-917`)

The kitchen-sink setter. Dispatches on `key` via a large `switch`
(`config.go:743-...`) to validate and write different fields of
`townSettings` or adjacent config files. Supported keys (from both
the Long text at `config.go:647-683` and the switch cases observed
at `config.go:744-830`):

| Key                             | Target                                      | Notes                                     |
|---------------------------------|---------------------------------------------|-------------------------------------------|
| `convoy.notify_on_complete`     | `townSettings.Convoy.NotifyOnComplete`      | bool (parseBool)                          |
| `cli_theme`                     | `townSettings.CLITheme`                     | `dark|light|auto`                         |
| `default_agent`                 | `townSettings.DefaultAgent`                 | string                                    |
| `dolt.port`                     | `daemon.DaemonPatrolConfig` env section     | 1024–65535; writes `GT_DOLT_PORT`         |
| `scheduler.max_polecats`        | `townSettings.Scheduler.MaxPolecats`        | int ≥ -1                                  |
| `scheduler.batch_size`          | `townSettings.Scheduler.BatchSize`          | positive int                              |
| `scheduler.spawn_delay`         | `townSettings.Scheduler.SpawnDelay`         | Go duration                               |
| `maintenance.window`            | delegated to `setMaintenanceConfig`         | `HH:MM`                                   |
| `maintenance.interval`          | same                                        | `daily|weekly|monthly` or duration        |
| `maintenance.threshold`         | same                                        | commit count                              |
| `lifecycle.reaper.enabled`      | `townSettings.Lifecycle.Reaper.Enabled`     | bool (per Long text)                      |
| `lifecycle.reaper.interval`     | ...Reaper.Interval                          | duration (default 30m)                    |
| `lifecycle.reaper.delete_age`   | ...Reaper.DeleteAge                         | duration (default 168h / 7d)              |
| `lifecycle.compactor.enabled`   | ...Compactor.Enabled                        | bool                                      |
| `lifecycle.compactor.interval`  | ...Compactor.Interval                       | duration (default 24h)                    |
| `lifecycle.compactor.threshold` | ...Compactor.Threshold                      | commit count (default 500)                |
| `lifecycle.doctor.enabled`      | ...Doctor.Enabled                           | bool                                      |
| `lifecycle.doctor.interval`     | ...Doctor.Interval                          | duration (default 5m)                     |
| `lifecycle.backup.enabled`      | ...Backup.Enabled                           | bool                                      |
| `lifecycle.backup.interval`     | ...Backup.Interval                          | duration (default 15m)                    |

The `scheduler.*` cases lazily initialize `townSettings.Scheduler`
via `capacity.DefaultSchedulerConfig()` if nil
(`config.go:773-775`, `:783-785`, `:794-796`). The `convoy.*` case
does the same with an empty struct.

#### `get <key>` (`config.go:690-726`, `runConfigGet` at `:837`)

Reads the same key set as `set`, minus `dolt.port` (which is not
listed in the `get` Long text — asymmetric). Implementation body not
read in full.

### Flags

Declared in `init()` (`config.go:1259-1261`):

| Flag         | Scope                          | Type   | Default | Purpose                                                       |
|--------------|--------------------------------|--------|---------|---------------------------------------------------------------|
| `--json`     | `agent list`, `default-agent list` | bool   | `false` | JSON output                                                   |
| `--provider` | `agent set`                    | string | `""`    | Agent provider preset (inferred from command basename if set) |

## Related

- [gt](../binaries/gt.md) — parent binary. `persistentPreRun` calls
  `initCLITheme()` which consumes the `cli_theme` key this command
  writes.
- [README.md](README.md) — command tree index.
- [theme.md](theme.md) — **`theme cli <mode>` is a second code path
  that writes the same `townSettings.CLITheme` field**
  (`theme.go:414`). Two writers, one store — drift risk noted on
  both pages.
- [account.md](account.md) — sibling Configuration command using a
  separate `accounts.json` store. Account selection and default
  agent are related but stored independently.
- [doctor.md](doctor.md) — `doctor` registers several config-arch
  checks at `doctor.go:231-238` (settings, session hook, runtime
  gitignore, legacy gastown, deprecated merge-queue keys) that
  consume the same town settings file.
- [dashboard.md](dashboard.md) — the dashboard reads scheduler /
  lifecycle / convoy state that `config set` writes.
- [status.md](status.md) — status reads the configured state as
  well.
- [disable.md](disable.md) / [enable.md](enable.md) — these manipulate
  a separate state file, not the town settings this command edits.

## Notes / open questions

- **Inline-defined `configAgentCmd`** (`config.go:1264-1272`) — the
  only subcommand group in the file that is NOT a package-level
  `var`. Possibly an oversight; harmless but inconsistent with the
  rest of the file.
- **Drift risk: two writers for `cli_theme`**. `config set cli_theme`
  validates via a `switch` over `{dark, light, auto}`
  (`config.go:754-760`), while `theme cli <mode>` validates via
  `isValidCLITheme` against the slice `validCLIThemes`
  (`theme.go:23-24`). The two sets agree today, but any future
  extension must update both sites.
- **`get` and `set` key sets are asymmetric**. `dolt.port` appears
  in the Long text at `config.go:653-655` for `set` but is absent
  from the `get` Long text at `config.go:695-705`. Intentional (port
  lives in `daemon.DaemonPatrolConfig`, not town settings) or
  oversight?
- **`default-agent list` is an alias** that reuses the same handler
  as `agent list`. This works but produces a misleading
  `{ "flag": "json", "defaultValue": "false" }` tie across two
  separate flag variables (`configAgentListJSON` vs
  `configDefaultAgentListJSON`). Minor.
- **Lifecycle config lives under `townSettings.Lifecycle`** — this
  is a nested config struct that deserves its own
  `config-file/town-settings.md` page once the config package is
  mapped. The default durations printed in the Long text are
  authoritative reference for that page.
- **`dolt.port` routes to `daemon.LoadPatrolConfig`** — a separate
  config file under `mayor/daemon.json`. Changing the port affects
  the entire Dolt layer; coordinate with `gt dolt` commands (see
  the Dolt operational section in the repo-level CLAUDE.md).
- **`setMaintenanceConfig`** (`config.go:919`) writes maintenance
  settings through a separate helper. Target file not yet mapped.
- This file is the largest in the Configuration group; future reads
  should focus on `runConfigSet` (`:728-917`) and `runConfigGet`
  (`:837-917`) in full to complete the key-→-file mapping.
