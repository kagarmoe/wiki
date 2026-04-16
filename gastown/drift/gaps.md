---
title: Phase 3 gap findings
type: drift
status: active
topic: gastown
created: 2026-04-16
updated: 2026-04-16
phase: 3
---

# Phase 3 Gap Findings

Systematic code-to-wiki gap enumeration produced by Batch 14 (Sweeps G1 + G2). Complements the drift index at [README.md](README.md), which covers what's *wrong*; this file covers what's *missing*.

---

## Section A: Missing entity pages

### A1: Go packages with no wiki page

Enumerated all 68 Go package directories under `internal/` and compared against 61 existing `gastown/packages/*.md` pages.

| Code path | Description | Classification | Rationale |
|---|---|---|---|
| `internal/agent` | Shared types and StateManager generic for all agents (witness, refinery, deacon) | gap: missing | Own API surface (generic StateManager[T]), imported by multiple agents; 62 lines but distinct abstraction |
| `internal/agent/provider` | JSON-RPC provider types for LLM agent communication (roles, messages, tool definitions) | gap: missing | 799 lines, own type system (Role, Message, Tool, ToolResult), core agent protocol layer |
| `internal/boot` | Boot watchdog package — daemon entry point for Deacon triage decisions | gap: missing | 237 lines, distinct from `gt boot` command page; package has its own daemon-tick logic |
| `internal/checkpoint` | Session checkpointing for crash recovery (save/load polecat state) | gap: missing | 350 lines, own file format (.polecat-checkpoint.json), recovery API |
| `internal/cmd` | Cobra command registration — all 111 top-level commands + 384 subcommands | deliberately excluded | The `cmd` package is the CLI wiring layer (239 files, 96,500 lines). Every command already has its own wiki page under `gastown/commands/`. A package-level page would be a meta-index that duplicates `gastown/commands/README.md`. |
| `internal/connection` | Address parsing for agent/rig/polecat addresses (`[machine:]rig[/polecat]`) | gap: missing | 689 lines, own Address type, 4 files; networking/routing abstraction |
| `internal/proxy` | mTLS CA management and proxy server for sandboxed polecat execution | gap: missing | 1,363 lines, 5 files; crypto/TLS infrastructure, distinct from the `gt-proxy-*` binary pages |
| `internal/scheduler/capacity` | Pure types and functions for capacity-controlled dispatch scheduler | deliberately excluded | 540 lines, but parent `gastown/packages/scheduler.md` already exists and covers the scheduler. This sub-package holds types extracted for testability; a separate page would fragment the scheduler narrative. |
| `internal/templates/commands` | Agent-agnostic command provisioning (embedded markdown bodies for agent CLAUDE.md generation) | deliberately excluded | 175 lines, 1 file; helper sub-package of `internal/templates` which already has a wiki page (`gastown/packages/templates.md`). Provides embedded `bodies/*.md` files used by the parent. |
| `internal/tui/convoy` | Bubbletea TUI model for convoy display (key bindings, view rendering) | deliberately excluded | 624 lines, but parent `gastown/packages/tui.md` already exists and covers both TUI sub-packages. This is a UI component sub-package, not an independent API surface. |
| `internal/tui/feed` | Bubbletea TUI model for feed display (convoy issues, formatting) | deliberately excluded | 3,665 lines (largest sub-package), but parent `gastown/packages/tui.md` covers both TUI sub-packages. If the TUI page grows too large, this could be split out, but currently the parent page is adequate. |

**Summary:** 6 gap findings (packages that need wiki pages), 5 deliberately excluded.

### A2: Binaries

Compared `cmd/` directory (3 binaries) against `gastown/binaries/` (3 pages).

| Binary | Wiki page | Status |
|---|---|---|
| `gt` | `gt.md` | Covered |
| `gt-proxy-client` | `gt-proxy-client.md` | Covered |
| `gt-proxy-server` | `gt-proxy-server.md` | Covered |

**Summary:** Full coverage. No gaps.

### A3: Root / config files

Compared notable gastown root files against `gastown/files/` pages.

| Root file | Wiki page | Status | Rationale |
|---|---|---|---|
| `Makefile` | `makefile.md` | Covered | |
| `Dockerfile` | `dockerfile.md` | Covered | |
| `Dockerfile.e2e` | `dockerfile-e2e.md` | Covered | |
| `docker-compose.yml` | `docker-compose.md` | Covered | |
| `docker-entrypoint.sh` | `docker-entrypoint.md` | Covered | |
| `go.mod` | `go-mod.md` | Covered | |
| `.goreleaser.yml` | `goreleaser-yml.md` | Covered | |
| `.golangci.yml` | `golangci-yml.md` | Covered | |
| `flake.nix` | `flake-nix.md` | Covered | |
| `.claude/` | `claude-dir.md` | Covered | |
| `.opencode/` | `opencode-dir.md` | Covered | |
| `codecov.yml` | — | deliberately excluded | CI config (108 lines); defines coverage thresholds and flags. Low wiki value — rarely referenced, no runtime behavior, and content is self-documenting YAML. |
| `renovate.json` | — | deliberately excluded | Dependency bot config (74 lines); Renovate auto-update rules. Low wiki value — no runtime behavior, self-documenting JSON. |
| `go.sum` | — | deliberately excluded | Auto-generated dependency checksums (322 lines). Never manually edited; no conceptual content. |
| `.gitignore` | — | deliberately excluded | Standard gitignore (86 lines). Self-documenting, no runtime behavior. |

**Summary:** Full coverage for files with wiki value. 4 deliberately excluded (CI/tooling config with no runtime behavior).

### A4: Services

Checked for long-running processes not covered by existing categories.

| Service | Coverage | Notes |
|---|---|---|
| Daemon | `gastown/commands/daemon.md` + `gastown/packages/daemon.md` | Covered |
| Dolt server | `gastown/commands/dolt.md` + `gastown/packages/doltserver.md` | Covered |
| Proxy server | `gastown/binaries/gt-proxy-server.md` + `gastown/packages/proxy.md` (gap) | Package page missing (see A1) |

**Summary:** No service-level gaps beyond the proxy package gap already captured in A1.

---

## Section B: Subcommand coverage

### B1: Overall statistics

| Metric | Count |
|---|---|
| Total `AddCommand` registrations | 495 |
| Root-level commands (top-level `gt <cmd>`) | 111 |
| Non-root subcommands | 384 |
| Parent commands with subcommands | 62 |

### B2: Per-parent coverage for commands with >5 subcommands

| Parent | Code subcommands | Wiki covered | Gap | Notes |
|---|---|---|---|---|
| `mail` | 22 | 22 | 0 | All 22 documented after Phase 3 Sweep 1 fix |
| `dolt` | 21 | 21 | 0 | `init-rig` and `migrate-wisps` both documented |
| `rig` | 20 | 20 | 0 | Full coverage |
| `deacon` | 18 | 18 | 0 | Full coverage |
| `crew` | 13 | 13 | 0 | Full coverage |
| `polecat` | 12 | 12 | 0 | Full coverage |
| `molecule` | 12 | 12 | 0 | `attach-from-mail` documented |
| `convoy` | 12 | 12 | 0 | Full coverage |
| `wl` | 11 | 11 | 0 | Full coverage |
| `refinery` | 11 | 11 | 0 | Full coverage |
| `session` | 9 | 9 | 0 | Full coverage |
| `hooks` | 9 | 9 | 0 | Full coverage |
| `dog` | 9 | 9 | 0 | Full coverage |
| `mq` | 8 | 8 | 0 | Full coverage |
| `daemon` | 8 | 8 | 0 | `clear-backoff` and `enable-supervisor` both documented |
| `mail channel` | 7 | 7 | 0 | Nested under mail.md |
| `scheduler` | 6 | 6 | 0 | Full coverage |
| `role` | 6 | 6 | 0 | Full coverage |
| `namepool` | 6 | 6 | 0 | Full coverage |
| `mayor` | 6 | 6 | 0 | Full coverage |
| `mail group` | 6 | 6 | 0 | Nested under mail.md |
| `config` | 6 | 6 | 0 | Full coverage |

### B3: Per-parent coverage for commands with 3-5 subcommands

| Parent | Code subcommands | Wiki covered | Gap | Notes |
|---|---|---|---|---|
| `witness` | 5 | 5 | 0 | |
| `quota` | 5 | 5 | 0 | |
| `polecat identity` | 5 | 5 | 0 | |
| `plugin` | 5 | 5 | 0 | |
| `krc` | 5 | 5 | 0 | `auto-prune-status` documented |
| `hook` | 5 | 5 | 0 | |
| `formula` | 5 | 4 | 1 | `overlay` sub-tree (3 sub-subcommands) not mentioned |
| `escalate` | 5 | 5 | 0 | |
| `agents` | 5 | 5 | 0 | |
| `account` | 5 | 5 | 0 | |
| `tap guard` | 4 | 4 | 0 | |
| `patrol` | 4 | 3 | 1 | `scan` subcommand not mentioned |
| `mountain` | 4 | 4 | 0 | |
| `molecule step` | 4 | 4 | 0 | |
| `mail queue` | 4 | 4 | 0 | |
| `config agent` | 4 | 4 | 0 | |
| `warrant` | 3 | 3 | 0 | |
| `trail` | 3 | 3 | 0 | |
| `tap` | 3 | 3 | 0 | |
| `synthesis` | 3 | 3 | 0 | |
| `shell` | 3 | 3 | 0 | |
| `rig settings` | 3 | 3 | 0 | |
| `rig config` | 3 | 3 | 0 | |
| `mq integration` | 3 | 3 | 0 | |
| `issue` | 3 | 3 | 0 | |
| `formula overlay` | 3 | 0 | 3 | `show`, `edit`, `list` — entire sub-tree missing from formula.md |
| `directive` | 3 | 3 | 0 | |
| `checkpoint` | 3 | 3 | 0 | |
| `boot` | 3 | 3 | 0 | |
| `bead` | 3 | 3 | 0 | |

### B4: Commands with 1-2 subcommands

| Parent | Code subcommands | Wiki covered | Gap | Notes |
|---|---|---|---|---|
| `worktree` | 2 | 2 | 0 | |
| `town` | 2 | 2 | 0 | |
| `theme` | 2 | 2 | 0 | |
| `orphans procs` | 2 | 2 | 0 | |
| `orphans` | 2 | 2 | 0 | |
| `krc config` | 2 | 2 | 0 | |
| `cycle` | 2 | 2 | 0 | |
| `costs` | 2 | 2 | 0 | |
| `start` | 1 | 1 | 0 | |
| `sling` | 1 | 1 | 0 | |
| `signal` | 1 | 1 | 0 | |
| `log` | 1 | 1 | 0 | |
| `config default-agent` | 1 | 1 | 0 | |
| `compact` | 1 | 1 | 0 | |
| `callbacks` | 1 | 1 | 0 | |
| `activity` | 1 | 1 | 0 | |

### B5: Subcommand gap detail

Two parent commands have subcommand coverage gaps:

#### `gt formula overlay` (3 missing sub-subcommands)

- **`formula overlay show`** — display active overlay for a formula (`formula_overlay_show.go`, 109 lines). Classification: **gap: needs parent expansion**. The formula.md page should add an Overlay section describing the sub-tree.
- **`formula overlay edit`** — open overlay in $EDITOR (`formula_overlay_edit.go`, 103 lines). Classification: **gap: needs parent expansion**.
- **`formula overlay list`** — list all overlay files (`formula_overlay_list.go`, 99 lines). Classification: **gap: needs parent expansion**.

The `overlay` parent itself (`formula_overlay.go`, 36 lines) is a grouping stub. All three leaf commands are simple enough to document in formula.md's subcommand table rather than needing their own pages.

#### `gt patrol scan` (1 missing subcommand)

- **`patrol scan`** — scan polecats for zombies, stalls, and completions (`patrol_scan.go`, ~120 lines). Bridges witness library detection to CLI for the survey-workers patrol step. Classification: **gap: needs parent expansion**. Has 4 flags (`--json`, `--notify`, `--rig`, `--verbose`); should be documented in patrol.md's subcommand table.

---

## Summary statistics

| Category | Count |
|---|---|
| **Missing entity pages (Sweep G1)** | |
| Go packages: gap (needs page) | 6 |
| Go packages: deliberately excluded | 5 |
| Binaries: gap | 0 |
| Root files: gap | 0 |
| Root files: deliberately excluded | 4 |
| Services: gap | 0 |
| **Subcommand coverage (Sweep G2)** | |
| Total non-root subcommands | 384 |
| Covered in parent wiki page | 380 |
| Gap: needs parent expansion | 4 |
| Gap: needs own page | 0 |
| Deliberately excluded | 0 |
| **Totals** | |
| Total gap findings (entities + subcommands) | 10 |
| Total deliberately excluded | 9 |
