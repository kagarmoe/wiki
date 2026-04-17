---
title: Phase 6 upstream correction drafts
type: drift
status: active
topic: gastown
created: 2026-04-16
updated: 2026-04-16
phase: 6
---

# Phase 6 upstream correction drafts

Drafted corrections for all drift findings from the [drift index](README.md) Section 1.
Grouped by meta-pattern for efficient PR batching. Kimberly submits these as PRs
against `~/repos/gastown/`.

**Authority hierarchy:** Code > Cobra Long text > docs/*.md. All corrections
align the lower-authority layer with the code.

---

## Group 1: Hand-maintained Long text enumerations

These are all the same fix pattern: a Cobra `Long` text has a hand-maintained
list of subcommands/capabilities that omits entries added later via sibling
files. A single PR can fix all of them.

**Recommended PR approach:** Either update each Long text to include the missing
entries, or replace the hand-maintained lists with a note directing users to
`--help` for the auto-generated `Available Commands:` block.

---

### account.md — Parent Long text omits `switch` subcommand

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/account.go:34-38`
**Current text:**
> Commands:
>   gt account list              List registered accounts
>   gt account add <handle>      Add a new account
>   gt account default <handle>  Set the default account
>   gt account status            Show current account info

**Should be:**
> Commands:
>   gt account list              List registered accounts
>   gt account add <handle>      Add a new account
>   gt account default <handle>  Set the default account
>   gt account status            Show current account info
>   gt account switch <handle>   Switch to a different account

**Rationale:** `accountSwitchCmd` is registered at `account.go:513-518` and is the only path that actually flips the live account via symlink rewrite. Fully functional at v1.0.0.

---

### crew.md — Parent Long text lists 8 subcommands; 11 visible registered

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/crew.go:50-58`
**Current text:**
> Commands:
>   gt crew start <name>     Start session (creates workspace if needed)
>   gt crew stop <name>      Stop session(s)
>   gt crew add <name>       Create workspace without starting
>   gt crew list             List workspaces with status
>   gt crew at <name>        Attach to session
>   gt crew remove <name>    Remove workspace
>   gt crew refresh <name>   Context cycle with handoff mail
>   gt crew restart <name>   Kill and restart session fresh

**Should be (add 3 lines):**
> Commands:
>   gt crew start <name>     Start session (creates workspace if needed)
>   gt crew stop <name>      Stop session(s)
>   gt crew add <name>       Create workspace without starting
>   gt crew list             List workspaces with status
>   gt crew at <name>        Attach to session
>   gt crew remove <name>    Remove workspace
>   gt crew refresh <name>   Context cycle with handoff mail
>   gt crew restart <name>   Kill and restart session fresh
>   gt crew status [<name>]  Show workspace status
>   gt crew rename <old> <new> Rename workspace
>   gt crew pristine [<name>]  Check workspace cleanliness

**Rationale:** `crew.go:412-429` registers 13 subcommands; 2 are hidden (`next`, `prev`). The 3 omitted visible subcommands (`status`, `rename`, `pristine`) are all present at v1.0.0.

---

### doctor.md — Long text enumerates a selective subset of ~80 registered checks

**Finding:** cobra-drift | wrong | in-release | dev
**Source:** `internal/cmd/doctor.go:26-118`
**Current text:** ~93-line bulleted catalog of check names grouped by category (too long to quote).

**Should be:** Either:
(a) Replace the bulleted catalog with a category summary: "Runs ~80 diagnostic checks across workspace, infrastructure, rig, routing, lifecycle, Dolt, and patrol categories. Use `gt doctor -v` for the full check list." Or:
(b) Auto-generate the catalog at runtime from the registered check set so it cannot drift.

**Rationale:** `runDoctor` at `doctor.go:154-279` registers ~80 checks. 35+ checks are absent from the Long text catalog. The Long text is a stale partial snapshot.

---

### hooks.md — Parent Long text lists 8 subcommands; `init` is wired but unlisted

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/hooks.go:17-25`
**Current text:**
> Subcommands:
>   base       Edit the shared base hook config
>   override   Edit overrides for a role or rig
>   sync       Regenerate all .claude/settings.json files
>   diff       Show what sync would change
>   list       Show all managed settings.json locations
>   scan       Scan workspace for existing hooks
>   registry   List hooks from the registry
>   install    Install a hook from the registry

**Should be (add 1 line):**
> Subcommands:
>   base       Edit the shared base hook config
>   override   Edit overrides for a role or rig
>   sync       Regenerate all .claude/settings.json files
>   diff       Show what sync would change
>   list       Show all managed settings.json locations
>   scan       Scan workspace for existing hooks
>   registry   List hooks from the registry
>   install    Install a hook from the registry
>   init       Bootstrap base config from existing settings.json files

**Rationale:** `hooks_init.go:14-29` defines and registers `hooksInitCmd` on `hooksCmd` via `hooks_init.go:31-34`. Present at v1.0.0.

---

### mail.md — COMMANDS section lists 4 subcommands; 22 are registered

**Finding:** cobra-drift | wrong | in-release | agent
**Source:** `internal/cmd/mail.go:92-95`
**Current text:**
> COMMANDS:
>   inbox     View your inbox
>   send      Send a message
>   read      Read a specific message
>   mark      Mark messages read/unread

**Should be:** Replace the hand-maintained COMMANDS block with either the full 22-subcommand list or a summary directing to `gt mail --help`. The full set: `send`, `inbox`, `read`, `peek`, `delete`, `archive`, `mark-read`, `mark-unread`, `check`, `thread`, `reply`, `claim`, `release`, `clear`, `search`, `announces`, `drain`, `channel`, `directory`, `group`, `hook`, `queue`.

**Rationale:** `mail.go:524-541` registers 17 subcommands; 5 more from sibling files (`mail_channel.go`, `mail_directory.go`, `mail_group.go`, `mail_hook.go`, `mail_queue.go`). The COMMANDS list is 82% incomplete. All present at v1.0.0.

---

### molecule.md — Parent Long text omits step-group members and await-signal shortcut

**Finding:** cobra-drift | wrong | in-release | agent
**Source:** `internal/cmd/molecule.go:22-43`
**Current text:** The "WORKING ON STEPS" section lists only `step done`. No mention of `step await-event`, `step await-signal`, `step emit-event`, or the top-level `await-signal` shortcut.

**Should be:** Add a "SYNCHRONIZATION:" heading for `step await-event`, `step await-signal`, `step emit-event`, and the `await-signal` shortcut. Or remove the hand-maintained category list.

**Rationale:** `molecule_await_event.go:115`, `molecule_await_signal.go:113,132`, `molecule_emit_event.go:64` register these subcommands. All present at v1.0.0.

---

### namepool.md — Examples show 6 operations; 8 subcommands registered

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/namepool.go:31-38`
**Current text:** Examples block demonstrates: bare, `--list`, `themes`, `set`, `add`, `reset`.

**Should be:** Add examples for `create` and `delete`:
> gt namepool create mytheme    # Create a custom theme
> gt namepool delete mytheme    # Delete a custom theme

**Rationale:** `namepool.go:111-117` registers `create` (`namepool.go:83-97`) and `delete` (`namepool.go:99-108`). Present at v1.0.0.

---

### quota.md — COMMANDS block lists 4 subcommands; 5 registered

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/quota.go:47-54`
**Current text:**
> COMMANDS:
>   gt quota status            Show quota status for all accounts
>   gt quota scan              Detect rate-limited sessions
>   gt quota rotate            Swap blocked sessions to available accounts
>   gt quota clear             Mark account(s) as available again

**Should be (add 1 line):**
> COMMANDS:
>   gt quota status            Show quota status for all accounts
>   gt quota scan              Detect rate-limited sessions
>   gt quota rotate            Swap blocked sessions to available accounts
>   gt quota clear             Mark account(s) as available again
>   gt quota watch             Continuously poll and auto-rotate

**Rationale:** `quota.go:974-978` registers `watch` (`quota.go:812-869`), a long-running foreground process with `--interval` and `--dry-run` flags. Present at v1.0.0.

---

### reaper.md — "When run by a Dog" block lists 4 subcommands; 6 registered

**Finding:** cobra-drift | wrong | in-release | dev
**Source:** `internal/cmd/reaper.go:40-45`
**Current text:** Example block enumerates: `scan`, `reap`, `purge`, `auto-close`.

**Should be:** Add `databases` and `run`:
> gt reaper databases --db=gastown   # List databases available for reaping
> gt reaper run --db=gastown         # Full scan-reap-purge-auto-close cycle

**Rationale:** `reaper.go:566-571` registers `databases` (`reaper.go:45-59`) and `run` (`reaper.go:408-520`). Present at v1.0.0.

---

### tap.md — Long text omits `list` and `polecat-stop-check`, advertises 3 unimplemented subcommands

**Finding (1):** cobra-drift | wrong | in-release | agent
**Source:** `internal/cmd/tap.go:15-18`
**Current text:** Subcommands list names `guard`, `audit [planned]`, `inject [planned]`, `check [planned]`.

**Should be:**
> Subcommands:
>   guard              Guard commands via PreToolUse/PostToolUse hooks
>   list               List registered tap handlers
>   polecat-stop-check Check polecat stop conditions

**Finding (2):** cobra-drift | wrong | in-release | agent — Remove or clearly mark `audit`, `inject`, `check` as `[NOT YET IMPLEMENTED]`. No source files exist for these.

**Rationale:** `tap_list.go:31` and `tap_polecat_stop.go:38` register `list` and `polecat-stop-check`. No `tap_audit.go`, `tap_inject.go`, or `tap_check.go` exist.

---

## Group 2: Stale Long text claims

These are individually distinct Cobra `Long` text errors where the text makes a
specific factual claim that contradicts the code. Each needs a targeted text edit.

---

### activity.md — Refinery event type names wrong in Long text

**Finding:** cobra-drift | wrong | in-release | dev
**Source:** `internal/cmd/activity.go:52-56`
**Current text:**
> Supported event types for refinery:
>   merge_started    - When refinery starts a merge
>   merge_complete   - When merge succeeds
>   merge_failed     - When merge fails
>   queue_processed  - When refinery finishes processing queue

**Should be:**
> Supported event types for refinery:
>   merge_started    - When refinery starts a merge
>   merged           - When merge succeeds
>   merge_failed     - When merge fails
>   merge_skipped    - When merge is skipped

**Rationale:** Event constants at `internal/events/events.go:67-70` resolve to `"merged"` and `"merge_skipped"`, not `"merge_complete"` and `"queue_processed"`. The switch at `activity.go:136` uses these constants.

---

### assign.md — `--force` flag defined but never consumed

**Finding:** cobra-drift | wrong | in-release | agent
**Source:** `internal/cmd/assign.go:62`
**Current text:**
> `assignCmd.Flags().BoolVar(&assignForce, "force", false, "Replace existing hooked work")`

**Should be:** Either implement `--force` to clear existing hooked beads before re-hooking, or remove the flag registration at `assign.go:62` and the variable at `assign.go:51`.

**Rationale:** `assignForce` is declared at `assign.go:51` and bound at `:62`, but `runAssign` (`assign.go:67-192`) never reads it. No reference to `assignForce` exists anywhere in the file.

---

### boot.md — "Special dog" framing contradicts dog-manager exclusion

**Finding:** cobra-drift | wrong | in-release | agent
**Source:** `internal/cmd/boot.go:33`
**Current text:**
> Boot is a special dog that runs fresh on each daemon tick.

**Should be:**
> Boot is a daemon-tick watchdog that runs fresh on each daemon tick.

Or:
> Boot is a short-lived triage agent that runs fresh on each daemon tick.

**Rationale:** `internal/dog/manager.go:300` explicitly excludes Boot from the kennel: "No .dog.json means this isn't a valid dog worker (e.g., 'boot' is the boot watchdog using .boot-status.json, not a dog)." Boot has no `AgentDog` type, no `.dog.json`, and is excluded from dog enumeration.

---

### callbacks.md — `SLING_REQUEST` handler description says "spawn" but only logs

**Finding:** cobra-drift | wrong | in-release | agent
**Source:** `internal/cmd/callbacks.go:98`
**Current text:**
> SLING_REQUEST:     - Spawn polecat for the work

**Should be:**
> SLING_REQUEST:     - Log the request (Deacon executes the actual sling)

**Rationale:** `handleSling` at `callbacks.go:471-502` only logs. Comment at `callbacks.go:498-499`: "We don't actually spawn here - that would be done by the Deacon executing the sling command based on this request."

---

### changelog.md — `--week` flag defined but never consulted

**Finding:** cobra-drift | wrong | in-release | dev
**Source:** `internal/cmd/changelog.go:39` (Examples line) and `changelog.go:49` (flag registration)
**Current text:**
> gt changelog --week     # This week's completions
> `changelogCmd.Flags().BoolVar(&changelogWeek, "week", false, "Show this week's completions")`

**Should be:** Either wire `changelogWeek` into `changelogSinceTime` (add an `if changelogWeek { ... }` branch before the default), or remove both the flag registration at `changelog.go:49` and the Examples line at `changelog.go:39`.

**Rationale:** `changelogSinceTime` (`changelog.go:99-121`) never reads `changelogWeek`. The cascade is `--since` > `--today` > default-this-week. `--week` coincidentally matches the default but has no effect.

---

### config.md — `config get` Long text omits `dolt.port` from Supported keys

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/config.go:695-705` (the Supported keys block of `configGetCmd`)
**Current text:** The Supported keys list does not include `dolt.port`.

**Should be:** Add to the Supported keys block:
> dolt.port                    Dolt server port (default: 3307)

**Rationale:** `runConfigGet` at `config.go:896-905` has an explicit `case "dolt.port":` that loads the patrol config and reads `GT_DOLT_PORT`. The `default` branch's error message at `config.go:911` even lists `dolt.port` in its "Supported keys:" text.

---

### down.md — Long text recommends `gt start` as complement; `gt up` is correct

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/down.go:66`
**Current text:**
> This is a "pause" operation - use 'gt start' to bring everything back up.

**Should be:**
> This is a "pause" operation - use 'gt up' to bring everything back up.

**Rationale:** `gt start` does NOT start the daemon. `gt up` (`up.go:120-448`) is the actual complement: it starts Dolt, daemon, deacon, mayor, witnesses, and refineries.

---

### git-init.md — Long text lists 3 steps; code performs 4

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/gitinit.go:25-30`
**Current text:**
> This command:
>   1. Creates a comprehensive .gitignore for Gas Town
>   2. Initializes a git repository if not already present
>   3. Optionally creates a GitHub repository (private by default)

**Should be:**
> This command:
>   1. Creates a comprehensive .gitignore for Gas Town
>   2. Initializes a git repository if not already present
>   3. Installs branch-protection hook (auto-reverts non-main checkouts)
>   4. Optionally creates a GitHub repository (private by default)

**Rationale:** `runGitInit` at `gitinit.go:184-186` calls `InstallPreCheckoutHook` — the hook auto-reverts the HQ to `main` on any non-main branch checkout.

---

### init.md — Long text lists 4 directories with wrong paths; code creates 5

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/init.go:26-27`
**Current text:**
> This creates the standard agent directories (polecats/, witness/, refinery/,
> mayor/) and updates .git/info/exclude to ignore them.

**Should be:**
> This creates the standard agent directories (polecats/, crew/, witness/,
> refinery/rig/, mayor/rig/) and updates .git/info/exclude to ignore them.

**Rationale:** `rig.AgentDirs` at `internal/rig/types.go:48-54` lists 5 entries: `polecats`, `crew`, `refinery/rig`, `witness`, `mayor/rig`. The Long text omits `crew` entirely and uses wrong paths for `refinery` and `mayor`.

---

### install.md (1) — HQ structure lists 3 items; install creates at least 5 directories

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/install.go:55-58`
**Current text:**
> It contains:
>   - CLAUDE.md            Mayor role context (Mayor runs from HQ root)
>   - mayor/               Mayor config, state, and rig registry
>   - .beads/              Town-level beads DB (hq-* prefix for mayor mail)

**Should be:**
> It contains:
>   - CLAUDE.md            Mayor role context (Mayor runs from HQ root)
>   - mayor/               Mayor config, state, and rig registry
>   - deacon/              Deacon config and state (daemon lifecycle management)
>   - plugins/             Town-level plugin directory
>   - .beads/              Town-level beads DB (hq-* prefix for mayor mail)

**Rationale:** `runInstall` at `install.go:307-318` creates `deacon/` and at `install.go:327-334` creates `plugins/`. Both unconditionally created.

---

### install.md (2) — Long text references non-existent `docs/hq.md`

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/install.go:61-62`
**Current text:**
> See docs/hq.md for advanced HQ configurations including beads
> redirects, multi-system setups, and HQ templates.

**Should be:** Remove this line entirely (the file does not exist at HEAD or v1.0.0), or create `docs/hq.md`.

**Rationale:** `docs/hq.md` does not exist in the gastown repo.

---

### plugin.md — Gate check is cooldown-only; other types silently fall through

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/plugin.go:94-95`
**Current text:**
> By default, checks if the gate would allow execution and informs you
> if it wouldn't. Use --force to bypass gate checks.

**Should be:**
> By default, checks cooldown gates and informs you if the cooldown
> hasn't elapsed. Other gate types (cron, condition, event, manual) are
> treated as always-open by the manual-run path. Use --force to bypass
> cooldown checks.

**Rationale:** `runPluginRun` at `plugin.go:425-442` only checks `GateCooldown`. For any other gate type, `gateOpen = true` stands unchanged.

---

### repair.md — Long text lists 6 repair targets; code registers 2 checks

**Finding:** cobra-drift | wrong | in-release | dev
**Source:** `internal/cmd/repair.go:22-28`
**Current text:** "What it repairs" lists 6 targets (metadata.json, missing config.json, prefix mismatches, missing Dolt databases, missing identity beads, stale Dolt port).

**Should be:** Either register the missing checks in the `checks` slice at `repair.go:51-54`, or rewrite the Long text to describe only what the two registered checks actually cover (rig config sync and stale Dolt port).

**Rationale:** `runRepair` at `repair.go:51-54` constructs a fixed two-check suite: `doctor.NewRigConfigSyncCheck()` and `doctor.NewStaleDoltPortCheck()`.

---

### shell.md — `shell install` silently re-enables Gas Town

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/shell.go:31-37`
**Current text:** Long text describes only the cd hook installation and its side behaviors. No mention of `state.Enable(Version)`.

**Should be:** Add to the Long text:
> Also sets the global 'enabled' state (equivalent to 'gt enable').

**Rationale:** `runShellInstall` at `shell.go:72-74` calls `state.Enable(Version)`. A user who previously ran `gt disable` and then `gt shell install` will find their kill-switch silently flipped back to enabled.

---

### status.md — `--fast` help text understates what it skips

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/status.go:50` and `status.go:57`
**Current text:**
> Skip mail lookups for faster execution

**Should be:**
> Skip mail, hooks, and merge-queue lookups for faster execution

**Rationale:** `--fast` gates four skips: overseer mail count (`status.go:776-781`), per-agent mail counts, per-rig hook discovery (`status.go:891-893`), and per-rig merge-queue summary (`status.go:907-909`).

---

### uninstall.md — `--workspace` silently no-ops for non-standard paths

**Finding:** cobra-drift | wrong | in-release | user
**Source:** `internal/cmd/uninstall.go:38` and `uninstall.go:50-51`
**Current text:**
> The workspace (e.g., ~/gt) is NOT removed unless --workspace is specified.
> Also remove the workspace directory (DESTRUCTIVE)

**Should be:** Add to the Long text:
> With --workspace, removes ~/gt or ~/gastown if present. Workspaces at
> other paths are not detected and must be removed manually.

**Rationale:** `findWorkspaceForUninstall` at `uninstall.go:152-171` tries only `$HOME/gt` and `$HOME/gastown`. Any other path silently skips with no warning.

---

### warrant.md — Long text hardcodes `~/gt/warrants/` but code uses `<townRoot>/warrants/`

**Finding:** cobra-drift | wrong | in-release | dev
**Source:** `internal/cmd/warrant.go:52`
**Current text:**
> Warrants are stored in ~/gt/warrants/ as JSON files.

**Should be:**
> Warrants are stored in <town-root>/warrants/ as JSON files.

**Rationale:** `getWarrantDir()` at `warrant.go:122-128` returns `filepath.Join(townRoot, "warrants")` where `townRoot` is resolved dynamically.

---

### witness.md — `--foreground` advertised as functional but is a no-op notice

**Finding:** cobra-drift | wrong | in-release | agent
**Source:** `internal/cmd/witness.go:63` (Examples) and `witness.go:124` (flag description)
**Current text:**
> gt witness start greenplace --foreground
> "Run in foreground (default: background)"

**Should be:** Either remove the `--foreground` example row and rewrite the flag description to: `"(deprecated no-op - patrol logic moved to mol-witness-patrol molecule)"`, or remove the flag entirely after auditing callers.

**Rationale:** `runWitnessStart` at `witness.go:179-183` intercepts `--foreground` and prints a notice: "Foreground mode no longer runs patrol loop."

---

### wrappers.md — ABOUTME header omits `gt-gemini` wrapper

**Finding:** cobra-drift | wrong | in-release | n/a
**Source:** `internal/wrappers/wrappers.go:1-2`
**Current text:**
> ABOUTME: Manages wrapper scripts for non-Claude agentic coding tools.
> ABOUTME: Provides gt-codex and gt-opencode wrappers that run gt prime first.

**Should be:**
> ABOUTME: Manages wrapper scripts for non-Claude agentic coding tools.
> ABOUTME: Provides gt-codex, gt-gemini, and gt-opencode wrappers that run gt prime first.

**Rationale:** Code installs three wrappers (`wrappers.go:26`): `gt-codex`, `gt-gemini`, `gt-opencode`.

---

## Group 3: Docs file corrections

`docs/*.md` files with drift findings from Sweep 2. These affect upstream docs
files, not Cobra Long text.

---

### done.md — `docs/CLEANUP.md` claims self-nuke but code transitions to IDLE

**Finding:** drift | wrong | in-release | agent
**Source:** `docs/CLEANUP.md:28`
**Current text:**
> gt done | Polecat self-cleaning: pushes branch, submits MR (by default), self-nukes worktree, kills own session. MR skipped for `--status ESCALATED|DEFERRED` or `no_merge` paths

**Should be:**
> gt done | Polecat self-cleaning: pushes branch, submits MR (by default), syncs worktree to main, transitions session to IDLE (sandbox preserved for reuse). MR skipped for `--status ESCALATED|DEFERRED` or `no_merge` paths

**Rationale:** The persistent polecat model (gt-hdf8) shipped in v1.0.0. `done.go:1209-1263` transitions to IDLE; `selfNukePolecat` at `done.go:1673-1680` has no call sites.

---

### install.md — `docs/INSTALLING.md` shows `rigs/` but rigs are top-level

**Finding:** drift | wrong | in-release | user
**Source:** `docs/INSTALLING.md:112-117`
**Current text:** Tree diagram shows `├── rigs/  # Project containers (initially empty)`.

**Should be:** Remove `rigs/` line. Add a note: "Rigs are created at the top level via `gt rig add <name> <url>`."

**Rationale:** `runInstall` does not create a `rigs/` directory. Rigs are `~/gt/<rigname>/` as top-level directories.

---

### convoy.md (1) — Event-driven feeder misattributed to operations.go

**Finding:** drift | wrong | in-release | n/a
**Source:** `docs/skills/convoy/SKILL.md:59`
**Current text:**
> Event-driven (`operations.go`): Polls beads stores every ~5s for close events.

**Should be:**
> Event-driven (`convoy_manager.go`): Polls beads stores every ~5s for close events.

**Rationale:** The poll loop is `ConvoyManager.runEventPoll()` at `internal/daemon/convoy_manager.go:186-247`. `operations.go` provides `CheckConvoysForIssue` (synchronous close-event handler).

---

### convoy.md (2) — `docs/concepts/convoy.md` omits `staged_ready` and `staged_warnings` states

**Finding:** drift | wrong | in-release | n/a
**Source:** `docs/concepts/convoy.md:66-76`
**Current text:** Lifecycle shows only `open` and `closed` states.

**Should be:** Add `staged_ready` and `staged_warnings` to the lifecycle diagram and state table. Full lifecycle: stage > launch > feed > land.

**Rationale:** `convoy.go:97-102` defines four statuses: `open`, `closed`, `staged_ready`, `staged_warnings`.

---

### identity.md — GIT_AUTHOR_NAME docs use full BD_ACTOR path but code sets agent name only

**Finding:** drift | wrong | in-release | n/a
**Source:** `docs/concepts/identity.md:102,116`
**Current text:**
> GIT_AUTHOR_NAME="gastown/polecats/toast"
> GIT_AUTHOR_NAME="gastown/crew/joe"

**Should be:**
> GIT_AUTHOR_NAME="toast"
> GIT_AUTHOR_NAME="joe"

**Rationale:** `internal/config/env.go:118` (polecat) and `:130` (crew): `env["GIT_AUTHOR_NAME"] = cfg.AgentName` — just the agent name, not the full slash path.

---

### go-mod.md — Go version disagrees across three build paths

**Finding:** drift | wrong | in-release | n/a
**Source:** `go.mod:3`, `Dockerfile:5`, `Dockerfile.e2e:12`
**Current text:** `go.mod` says 1.25.8; Dockerfile pins `GO_VERSION=1.25.6`; Dockerfile.e2e uses `golang:1.26-alpine`.

**Should be:** Align Dockerfile `GO_VERSION` to `1.25.8` to match `go.mod`. Dockerfile.e2e using 1.26 is a forward-compatibility choice and less concerning.

**Rationale:** Three build paths use three different Go versions.

---

### goreleaser-yml.md — Release header advertises `brew install` but no `brews:` block exists

**Finding:** drift | wrong | in-release | n/a
**Source:** `.goreleaser.yml:164-167`
**Current text:**
> Homebrew (macOS/Linux):
> ```bash
> brew install gastown
> ```

**Should be:** Either add a `brews:` block to `.goreleaser.yml` or remove the Homebrew install line from the release header template.

**Rationale:** No `brews:` block in `.goreleaser.yml`. GoReleaser cannot produce a Homebrew formula without it.

---

### health.md — Package doc claims Doctor Dog shares this package; only gt health imports it

**Finding:** drift | wrong | in-release | n/a
**Source:** `internal/health/health.go:2-3`
**Current text:**
> Package health provides reusable health check functions for the Gas Town data plane.
> These checks are shared between the Doctor Dog (daemon/doctor_dog.go) and the
> gt health CLI command (cmd/health.go).

**Should be:**
> Package health provides reusable health check functions for the Gas Town data plane.
> These checks are used by the gt health CLI command (cmd/health.go).

**Rationale:** `rg '"github.com/steveyegge/gastown/internal/health"'` finds exactly one importer: `internal/cmd/health.go`. No file in `internal/daemon/` or `internal/doctor/` imports it.

---

### telemetry.md — otel-architecture.md claims functions don't exist but they do

**Finding:** drift | wrong | in-release | n/a
**Source:** `docs/design/otel/otel-architecture.md:54,80-81`
**Current text:** Implementation status table marks molecule lifecycle, bead creation, and agent instantiation telemetry as "Roadmap" with "no RecordMol* functions exist." Says "18 metric instruments."

**Should be:** Mark these as "Main" (implemented). Update metric count from 18 to 24.

**Rationale:** `internal/telemetry/recorder.go` has `RecordMolCook` (line 692), `RecordMolWisp` (710), `RecordMolSquash` (731), `RecordMolBurn` (749), `RecordBeadCreate` (766), `RecordAgentInstantiate` (414). 24 total instruments.

---

### Sweep 2 docs-only corrections (no wiki page annotation)

#### `docs/design/persistent-polecat-pool.md` — Hyphenated subcommand name wrong

**Finding:** drift | wrong | in-release | n/a | Batch 11c
**Source:** `docs/design/persistent-polecat-pool.md`
**Current text:** `gt polecat pool init <rig>` (space-separated)
**Should be:** `gt polecat pool-init <rig>` (hyphenated, matching code registration)

#### `docs/HOOKS.md` — Known Gap #2 claims `tap guard dangerous-command` doesn't exist

**Finding:** drift | wrong | in-release | n/a | Batch 10e
**Source:** `docs/HOOKS.md`
**Current text:** Known Gap #2 claims `gt tap guard dangerous-command` doesn't exist.
**Should be:** Remove this known gap — `tap_guard_dangerous.go` exists at HEAD and v1.0.0.

#### `docs/design/directives-and-overlays.md` — "CLI commands are being added" label stale

**Finding:** drift | wrong | in-release | n/a | Batch 11e
**Source:** `docs/design/directives-and-overlays.md`
**Current text:** "CLI commands are being added" label
**Should be:** Update to reflect that `gt directive show/edit/list` and `gt formula overlay show/edit/list` all exist.

#### `docs/design/architecture.md` (1) — MQ implementation phases table stale

**Finding:** drift | wrong | in-release | n/a | Batch 11f
**Source:** `docs/design/architecture.md`
**Current text:** GatesParallel "In progress" and batch-then-bisect "Blocked by Phase 1."
**Should be:** Both fully implemented — mark as complete.

#### `docs/design/architecture.md` (2) — Dead `[Watchdog Chain](watchdog-chain.md)` reference

**Finding:** drift | wrong | in-release | n/a | Batch 11f
**Source:** `docs/design/architecture.md`
**Current text:** `[Watchdog Chain](watchdog-chain.md)`
**Should be:** `[Dog Infrastructure](dog-infrastructure.md)` (closest equivalent)

#### `docs/design/property-layers.md` — Dead `[Watchdog Chain](watchdog-chain.md)` reference

**Finding:** drift | wrong | in-release | n/a | Batch 11g
**Source:** `docs/design/property-layers.md`
**Should be:** `[Dog Infrastructure](dog-infrastructure.md)`

#### `docs/design/scheduler.md` — Dead `[Watchdog Chain](watchdog-chain.md)` reference

**Finding:** drift | wrong | in-release | n/a | Batch 11h
**Source:** `docs/design/scheduler.md`
**Should be:** `[Dog Infrastructure](dog-infrastructure.md)`

#### `docs/design/polecat-self-managed-completion.md` — Status label stale

**Finding:** drift | wrong | in-release | n/a | Batch 11i
**Source:** `docs/design/polecat-self-managed-completion.md`
**Current text:** "Status: Design proposal"
**Should be:** "Status: Implemented" — all three migration phases are shipped.

#### `docs/design/mail-protocol.md` — Stale completion flow diagram

**Finding:** drift | wrong | in-release | n/a | Batch 11l
**Source:** `docs/design/mail-protocol.md`
**Current text:** Shows Polecat > Witness > Refinery completion flow.
**Should be:** Polecat now nudges refinery directly (self-managed completion). Witness is kept for observability, not as a required relay.

#### `docs/design/polecat-lifecycle-patrol.md` — Stale cleanup pipeline flow

**Finding:** drift | wrong | in-release | n/a | Batch 11o
**Source:** `docs/design/polecat-lifecycle-patrol.md`
**Current text:** Same stale Polecat > Witness > Refinery flow as mail-protocol.md.
**Should be:** Witness no longer required checkpoint in completion flow.

---

## Group 4: Implementation-status preservation callouts

These findings are NOT bugs. They describe aspirational or partially-implemented
features in design docs. The correction is adding `[Status: Not Yet Implemented]`
or similar callouts so readers know the feature is not built.

---

### `docs/design/dog-infrastructure.md` — Dog Pool Architecture unbuilt

**Finding:** implementation-status: unbuilt | wrong | in-release | n/a | Batch 11j
**Source:** `docs/design/dog-infrastructure.md`
**Aspirational features:** ShutdownDanceState, DogPool struct, warrant queue, `gt dog dances/warrants/pool status`
**Suggested callout:**
> **[Status: Design Only]** The Dog Pool Architecture described in this document
> (ShutdownDanceState, DogPool struct, warrant queue, pool status commands) has
> not been implemented. Current dog infrastructure uses a simpler per-dog model.

---

### `docs/design/model-aware-molecules.md` — Entire feature unbuilt

**Finding:** implementation-status: unbuilt | wrong | in-release | n/a | Batch 11m
**Source:** `docs/design/model-aware-molecules.md`
**Aspirational features:** Per-step model constraints, subscription routing, OpenRouter pricing. `internal/models/` doesn't exist.
**Suggested callout:**
> **[Status: Design Only]** This feature (per-step model constraints, subscription
> routing, OpenRouter pricing integration) has not been implemented. The
> `internal/models/` package referenced here does not exist.

---

### `docs/design/ledger-export-triggers.md` — Entire feature unbuilt

**Finding:** implementation-status: unbuilt | wrong | in-release | n/a | Batch 11n
**Source:** `docs/design/ledger-export-triggers.md`
**Aspirational features:** Bead closure triggers, convoy triggers, skill derivation engine, HOP economy.
**Suggested callout:**
> **[Status: Design Only]** This feature (bead closure triggers, convoy triggers,
> skill derivation engine, HOP economy) has not been implemented.

---

### `docs/design/mol-mall-design.md` — Mol Mall registry system unbuilt

**Finding:** implementation-status: unbuilt | wrong | in-release | n/a | Batch 12a
**Source:** `docs/design/mol-mall-design.md`
**Aspirational features:** Public/private/federated registries, HOP URI scheme, formula publishing.
**Suggested callout:**
> **[Status: Design Only]** The Mol Mall registry system (public/private/federated
> registries, HOP URI scheme, formula publishing) has not been implemented beyond
> embedded formulas.

---

### `docs/design/witness-at-team-lead.md` — Witness-as-AT-team-lead unbuilt

**Finding:** implementation-status: unbuilt | wrong | in-release | n/a | Batch 12b
**Source:** `docs/design/witness-at-team-lead.md`
**Aspirational features:** Delegate mode, AT teammate spawning. Depends on Claude Code Agent Teams.
**Suggested callout:**
> **[Status: Design Only]** This architecture (delegate mode, AT teammate spawning)
> has not been implemented. It depends on Claude Code Agent Teams, which is not
> yet available.

---

### `docs/design/sandboxed-polecat-execution.md` — Sandboxed execution unbuilt

**Finding:** implementation-status: unbuilt | wrong | in-release | n/a | Batch 12d
**Source:** `docs/design/sandboxed-polecat-execution.md`
**Aspirational features:** Exitbox container backend, Daytona cloud backend, SandboxAdapter interface.
**Suggested callout:**
> **[Status: Design Only]** Sandboxed execution (container and cloud backends,
> SandboxAdapter interface) has not been implemented. All polecat execution
> currently runs in local tmux sessions.

---

### `docs/design/factory-worker-api.md` — Factory Worker API unbuilt

**Finding:** implementation-status: unbuilt | wrong | in-release | n/a | Batch 12f
**Source:** `docs/design/factory-worker-api.md`
**Aspirational features:** 7 structured endpoints for agent interaction.
**Suggested callout:**
> **[Status: Design Only]** The Factory Worker API (7 structured endpoints) has
> not been implemented. All agent interaction currently uses tmux-based
> communication (28 touch points in the codebase).

---

### `docs/design/plugin-system.md` — Status label stale; advanced features not implemented

**Finding:** implementation-status: partial | incomplete | in-release | n/a | Batch 12c
**Source:** `docs/design/plugin-system.md`
**Current status label:** "not yet implemented"
**Should be:**
> **[Status: Partially Implemented]** Core dispatch is shipped (`gt dog dispatch
> --plugin`, `gt plugin` CLI). Advanced features (versioning, marketplace,
> sandboxed execution) are not implemented.

---

### `docs/design/federation.md` — Core features not implemented

**Finding:** implementation-status: partial | incomplete | in-release | n/a | Batch 12e
**Source:** `docs/design/federation.md`
**Suggested callout:**
> **[Status: Partially Implemented]** Federation infrastructure exists (Dolt
> remotes, wasteland commands). Core features (HOP URI, cross-workspace queries,
> sovereignty model) are not implemented.

---

### `docs/design/formula-resolution.md` — Tier 1 and Mol Mall not implemented

**Finding:** implementation-status: partial | incomplete | in-release | n/a | Batch 12g
**Source:** `docs/design/formula-resolution.md`
**Suggested callout:**
> **[Status: Partially Implemented]** Tier 2 (town/rig overrides) and Tier 3
> (embedded formulas) work. Tier 1 (project-committed formulas) and Mol Mall
> integration are not implemented.

---

### witness.md — `--foreground` patrol loop moved to molecule (vestigial)

**Finding:** implementation-status: vestigial | wrong | in-release | agent | Batch 1d
**Note:** This is paired with the Group 2 cobra-drift correction above. The `--foreground` flag was kept as a runtime notice at `witness.go:179-183`. The implementation-status finding tracks the vestigial state; the cobra-drift correction handles the Long text fix.

---

### keepalive package — Fully implemented but zero importers

**Finding:** implementation-status: partial | incomplete | in-release | n/a | Batch 2b
**Note:** This is a wiki-only note. The `internal/keepalive` package has a complete Touch/Read/Age API but zero in-tree importers. No upstream correction needed unless the intent is to wire it into the persistent prerun.

---

## Summary

| Group | Description | Count |
|---|---|---|
| 1 | Hand-maintained Long text enumerations | 11 corrections |
| 2 | Stale Long text claims | 19 corrections |
| 3 | Docs file corrections | 19 corrections |
| 4 | Implementation-status preservation callouts | 12 corrections |
| **Total** | | **61 corrections** |
