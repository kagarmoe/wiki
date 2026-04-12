---
title: gt hooks
type: command
status: partial
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/internal/cmd/hooks.go
  - /home/kimberly/repos/gastown/internal/cmd/root.go
tags: [command, configuration, hooks, claude-code, stub-parent]
---

# gt hooks

Parent command for centralized management of Claude Code hooks
(`.claude/settings.json`) across the Gas Town workspace with a shared
base config and per-role/per-rig overrides. In the mapped source file,
`hooks` is declared as a parent-only shell — the eight subcommands its
`Long` text advertises (`base`, `override`, `sync`, `diff`, `list`,
`scan`, `registry`, `install`) are not wired up here.

**Parent:** [gt](../binaries/gt.md) (root command)
**Group:** `GroupConfig` ("Configuration")
**Polecat-safe:** no (no `AnnotationPolecatSafe` on the cobra.Command at
`/home/kimberly/repos/gastown/internal/cmd/hooks.go:7-40`)
**Beads-exempt:** no (not in `beadsExemptCommands` on `root.go:44-77` —
note that `hook` singular **is** exempt (`root.go:54`), but `hooks`
plural is a different command and is not listed)
**Branch-check-exempt:** no (not in `branchCheckExemptCommands` on
`root.go:81-91`)

## What it actually does

Source: `/home/kimberly/repos/gastown/internal/cmd/hooks.go:7-44`.

### Invocation

```
gt hooks <subcommand>
```

### Behavior

Parent-only. `RunE: requireSubcommand` (`hooks.go:39`) — without a
subcommand, returns an error and prints help. Registration is a
single call at `hooks.go:42-44`:

```go
func init() {
    rootCmd.AddCommand(hooksCmd)
}
```

No `hooksCmd.AddCommand(...)` calls appear in this file. The
subcommand wiring lives elsewhere or is not yet implemented.

### Subcommands

Advertised by `Long` text (`hooks.go:17-25`), not wired here:

- `base` — edit the shared base hook config
- `override` — edit overrides for a role or rig
- `sync` — regenerate all `.claude/settings.json` files
- `diff` — show what `sync` would change
- `list` — show all managed `settings.json` locations
- `scan` — scan workspace for existing hooks
- `registry` — list hooks from the registry
- `install` — install a hook from the registry

### Flags

None declared on the parent command.

### Config structure (per Long text)

`hooks.go:27-31`:

- Base: `~/.gt/hooks-base.json`
- Overrides: `~/.gt/hooks-overrides/<target>.json`
- Merge strategy: base → role → rig+role (more specific wins)

This differs from the `~/.local/state/gastown/` and
`~/.config/gastown/` state paths used by `enable`/`disable`/`uninstall`
(see [uninstall.md](uninstall.md)). The hooks system has its own
`~/.gt/` namespace.

## Related

- [gt](../binaries/gt.md) — parent binary.
- [README.md](README.md) — command tree index.
- [doctor.md](doctor.md) — `doctor` runs a hooks-sync health check
  (`hooks sync` drift) and a hook-attachment check — see the
  "Hooks sync" and "Hook attachment" check blocks at
  `doctor.go:259-265`.
- [prime.md](prime.md) — `gt prime` is invoked from a Claude Code
  `SessionStart` hook; `hooks sync` regenerates the `settings.json`
  files that wire that hook up.
- [upgrade.md](upgrade.md) — post-install migration; likely the
  consumer that invokes `hooks sync` automatically after upgrade.
- [config.md](config.md) — sibling Configuration-group command for
  town settings. Hooks config is explicitly a separate system
  (`~/.gt/hooks-*.json`) from town settings (`settings/config.json`).

## Notes / open questions

- **Missing subcommand wiring.** `hooks.go` only declares the parent.
  All eight advertised subcommands must be registered somewhere else
  under `internal/cmd/` (or not at all). Follow-up: grep for
  `hooksCmd.AddCommand` across the tree.
- **`hook` vs `hooks`.** `root.go:54` exempts `hook` from the beads
  check, which is described as "Hook signal handlers must be fast,
  handle beads internally". That exempt-map entry is for a different
  command — likely a signal-handler command registered elsewhere, not
  this `hooks` management command. Two separate command trees.
- `~/.gt/hooks-base.json` is a new path not in the Layer-(c) file
  layer yet; add `files/hooks-base-json.md` when the shape is mapped.
- `registry` and `install` subcommands imply a hooks registry — worth
  its own `concepts/hook-registry.md` page when source is mapped.
