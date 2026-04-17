---
title: Wiki validation round 4 (final)
type: drift
status: active
topic: gastown
created: 2026-04-17
updated: 2026-04-17
phase: 7
---

# Wiki validation round 4 (final)

Five open gastown issues scored against the wiki to measure how well it
supports issue investigation without reading gastown source. Final round
of the 20-issue validation series.

## Summary

| Issue | Title | Score | Gap type |
|---|---|---|---|
| #3554 | daemon: 'no wisp config' warning spams log every 5s for rigs never parked | partial | daemon heartbeat loop documented; wisp config per-rig layer documented; spam-on-missing-config failure mode not flagged |
| #3539 | gt sling fails: bd update --type=agent rejected after beads type extraction | full | beads custom types, registration flow, and the v0.46.0 extraction from core are all documented |
| #3538 | Windows is not a viable platform: tmux hard dependency, build failures | full | tmux hard dependency pervasive in wiki; BuiltProperly ldflag gate documented verbatim; platform shims documented |
| #3537 | gt crew start launches Claude session outside tmux — no session to attach to | partial | crew Start() tmux session creation documented in detail; socket isolation and `-L` flag mechanics documented; specific failure mode of session spawning outside tmux not flagged |
| #3516 | Docs: rig names must use underscores, dolt is undocumented install prereq | full | rig name validation (reject hyphens) documented at AddRig step 1; dolt as external prerequisite documented in deps package |

**Scores: 3 full, 2 partial, 0 miss.**

---

### #3554 -- daemon: 'no wisp config' warning spams log every 5s for rigs that have never been parked

**Issue summary:** The daemon logs "Warning: no wisp config for <rig> -
parked state may have been lost" every ~5 seconds for each rig that has
never been parked. This creates ~54MB/day of log noise. The missing wisp
config is normal for rigs that have never been parked -- the warning
should either be suppressed for rigs without parking history or fired
once at startup instead of continuously.

**Wiki pages checked:**
- [internal/daemon](../packages/daemon.md) -- heartbeat loop, maintenance tickers
- [internal/wisp](../packages/wisp.md) -- wisp config per-rig layer
- [wisp concept](../concepts/wisp.md) -- wisp domain model
- [rig concept](../concepts/rig.md) -- rig parking states

**What the wiki knows:** The wiki documents the daemon's heartbeat loop
at `daemon.go:724-879` with its 3-minute default interval (not 5s as the
issue implies -- the 5s interval corresponds to the `eventPollInterval`
used by the feed curator ticker, not the heartbeat). The daemon page also
documents the `wispReaper` ticker at 30m intervals. The `internal/wisp`
package page thoroughly documents the `.beads-wisp/config/<rig>.json`
storage layer, including `Config.load()` which "returns an empty
`ConfigFile` if the file doesn't exist, so the caller never has to check
for non-existence." The rig concept page documents the `parked` and
`docked` lifecycle states. The wiki also documents the `compact` command's
reference to "rig wisp config" as Layer 2 of the compaction config chain.

**What the wiki doesn't know:** The wiki does not document which daemon
code path reads the wisp config and emits this warning. The daemon page
catalogs the heartbeat steps and the ~10 maintenance tickers but does not
mention a "wisp config" check or a "parked state" warning in either
location. The wisp package page documents the config store's API
(Get/Set/Block) but not any daemon-side consumer that reads it on a loop.
The specific warning message ("no wisp config for <rig> - parked state
may have been lost") is not surfaced anywhere in the wiki. An
investigator would find the wisp config mechanism and the daemon's
periodic loops but would need source access to identify which loop
emits this specific warning and why it fires repeatedly instead of once.

**Score:** partial

**Remediation:** Identify which daemon ticker or heartbeat step reads
the wisp config and add a note about it to the daemon page. Document the
warning message and its trigger condition (missing
`.beads-wisp/config/<rig>.json`). Flag that the warning fires on every
cycle rather than once-at-startup, and that rigs without parking history
should be exempt.

---

### #3539 -- gt sling fails: bd update --type=agent rejected after beads type extraction

**Issue summary:** `gt sling` fails when spawning a polecat because it
calls `bd update <polecat-id> --type=agent`, but beads v1.0.0 removed
Gas Town-specific types (`agent`, `role`, `rig`, `convoy`, etc.) from
its built-in type set. The `bd update` validation uses `IsValid()`
(built-in only) rather than `IsValidWithCustom()`, rejecting `agent` as
invalid even when `types.custom` is configured. Three patterns are
affected: polecat spawn, polecat creation, and agent lookups.

**Wiki pages checked:**
- [gt sling](../commands/sling.md) -- work dispatch command
- [internal/beads](../packages/beads.md) -- beads library, `beads_types.go`
- [internal/constants](../packages/constants.md) -- `BeadsCustomTypes` registration
- [gt install](../commands/install.md) -- custom type registration at install time
- [internal/deps](../packages/deps.md) -- version pinning for bd and dolt

**What the wiki knows:** The wiki provides comprehensive coverage of the
custom type mechanism. The `internal/constants` page documents
`BeadsCustomTypes = "agent,role,rig,convoy,slot,queue,event,message,
molecule,gate,merge-request"` with the explicit note: "Extracted from
beads core in v0.46.0, hence the explicit registration." The
`internal/beads` page documents `beads_types.go` (481 lines) as "custom
type management: `FindTownRoot`, `EnsureCustomTypes` with a two-level
(in-memory + sentinel-file) cache, and the `bd config set types.custom`
plumbing." The `gt install` command page documents step 5: "Register
custom beads types via `registerCustomTypes`." The `gt sling` command
page documents the full dispatch pipeline including polecat spawning.
The `internal/deps` page documents `MinBeadsVersion = "0.57.0"`.

An investigator reading the wiki would understand: (1) Gas Town has
custom types that were extracted from beads core, (2) these types must
be explicitly registered via `bd config set types.custom`, (3) the
registration happens at install time and is cached, and (4) the sling
command dispatches polecat creation through the beads library. The wiki
provides enough context to diagnose that a beads version upgrade that
changes type validation behavior would break Gas Town's custom type
usage.

**Score:** full

**Remediation:** None needed for diagnosis. The wiki documents the
custom type extraction, the registration mechanism, and the version
dependency. Once the fix lands (either beads accepting custom types in
`update` validation, or Gas Town switching to
`--type=task --labels=gt:agent`), update the relevant pages.

---

### #3538 -- Windows is not a viable platform: tmux hard dependency, build failures, and no viable workaround

**Issue summary:** A comprehensive investigation concluding that Windows
is not viable for Gas Town due to: (1) the `BuiltProperly` ldflag gate
rejecting `go install` binaries, (2) the pervasive tmux hard dependency
throughout session management, daemon, crew, peek/nudge, (3) beads
cross-compilation complexity requiring ICU/CGO/MinGW, (4) version
mismatch between embedded and standalone beads, (5) MinGW PATH
pollution, and (6) orphan process/file locking issues. Standalone `bd`
works on Windows.

**Wiki pages checked:**
- [gt](../binaries/gt.md) -- BuiltProperly ldflag gate
- [internal/tmux](../packages/tmux.md) -- platform shims, hard dependency
- [internal/session](../packages/session.md) -- session substrate for all roles
- [internal/daemon](../packages/daemon.md) -- daemon depends on tmux
- [internal/crew](../packages/crew.md) -- crew depends on tmux sessions
- [Makefile](../files/makefile.md) -- build recipe, Windows powershell path
- [.goreleaser.yml](../files/goreleaser-yml.md) -- Windows build target
- [internal/git](../packages/git.md) -- `copy_windows.go` "not tested on Windows"
- [internal/mayor](../packages/mayor.md) -- `process_windows.go`
- [internal/doltserver](../packages/doltserver.md) -- `sysproc_windows.go`
- [Dockerfile](../files/dockerfile.md) -- bd/dolt install
- [internal/deps](../packages/deps.md) -- external prereqs

**What the wiki knows:** The wiki provides extensive evidence of both
Windows support attempts and the tmux hard dependency:

1. **BuiltProperly gate:** The `gt` binary page documents the exact
   self-kill check verbatim (`root.go:94-107`), including the misleading
   macOS-specific error message. This was validated as a "full" score in
   round 2 (#3626).

2. **tmux pervasiveness:** The `internal/tmux` page documents 11 files
   including 8 platform-specific pairs (`flock_windows.go`,
   `process_group_windows.go`, `descendants_windows.go`,
   `sysproc_windows.go`). The `internal/session` page states it is "the
   session substrate for ALL agent roles." The daemon, crew, polecat,
   witness, refinery, deacon, and dog packages all import tmux.

3. **Windows platform shims exist but are stubs:** The `internal/git`
   page explicitly notes that `copy_windows.go:15-38` "explicitly notes
   'This Windows implementation has not been tested on Windows' -- a
   drift flag." The `internal/mayor` page documents
   `process_windows.go` with `windows.OpenProcess`. The goreleaser page
   documents the `gt-windows-amd64` build target with MinGW cross-
   compilation.

4. **No platform abstraction:** The wiki makes clear that tmux is NOT
   behind an interface -- every session-adjacent package directly imports
   `internal/tmux`. The session package's description ("agent session
   lifecycle: naming... starting and stopping sessions via tmux")
   confirms tmux is baked into the foundation.

An investigator reading the wiki would quickly conclude that tmux is a
hard dependency woven through every agent lifecycle path, that Windows
platform shims exist but are untested, and that the `BuiltProperly`
gate blocks `go install` on all platforms. The wiki provides all the
architectural context needed to understand why Windows is not viable.

**Score:** full

**Remediation:** None needed for diagnosis. Consider adding a
cross-cutting "Platform support" concept page that consolidates the
Windows-specific findings from across the wiki (tmux dependency, untested
shims, BuiltProperly gate, goreleaser cross-compilation). Once any
platform abstraction lands, document it.

---

### #3537 -- gt crew start launches Claude session outside tmux -- no session to attach to

**Issue summary:** `gt crew start <name> --rig <rig>` spawns the Claude
process on a detached pty rather than inside a tmux session. `tmux
list-sessions` shows no entry for the crew member. `gt crew at` reports
readiness but has no attachable session. `gt peek` fails with "session
not found." The Claude process IS running (visible via `ps aux`). Related
to #3042 (tmux socket isolation revert) and #3514 (missing `-L` socket
flag).

**Wiki pages checked:**
- [internal/crew](../packages/crew.md) -- `Manager.Start()` at `manager.go:680-895`
- [gt crew](../commands/crew.md) -- CLI surface, `runCrewStart`
- [internal/tmux](../packages/tmux.md) -- socket management, `NewSessionWithCommandAndEnv`
- [internal/session](../packages/session.md) -- session naming, lifecycle

**What the wiki knows:** The wiki documents crew `Start()` in extensive
detail, including the specific tmux session creation call:
"`NewSessionWithCommandAndEnv` (not `NewSession + SendKeys`)" and the
subsequent `GT_PANE_ID` storage for ZFC liveness checks. The tmux package
page documents `SetDefaultSocket`/`GetDefaultSocket` and the per-town
socket isolation (`tmux -L <socket>`). The session package documents
`CrewSessionName` as the naming function. The crew page documents
`SetCrewCycleBindings` for tmux keybindings.

The tmux package page also documents `killSplitBrainSession`
(`tmux.go:771-787`) which "kills stale same-named sessions on other
sockets during `NewSessionWithCommandAndEnv`" -- directly relevant to
socket isolation issues.

**What the wiki doesn't know:** The wiki does not flag the specific
failure mode where session creation succeeds at the process level but
the session is not visible via `tmux list-sessions` because of a socket
mismatch (the `-L` flag issue from #3514). The wiki documents that
Gas Town uses per-town sockets (`tmux -L <socket>`) but does not explain
what happens when different parts of the system use different socket
arguments -- e.g., if `gt crew start` creates the session on one socket
but `gt crew at` / `tmux list-sessions` queries the default socket. The
related issues (#3042 socket isolation revert, #3514 missing `-L` flag)
point to a socket-argument inconsistency that the wiki's tmux coverage
doesn't surface.

**Score:** partial

**Remediation:** Add a note to the `internal/tmux` or `internal/crew`
package page about the socket-consistency requirement: every tmux
operation on a Gas Town session must use the same `-L <socket>` argument.
Document the failure mode where a session exists on a non-default socket
but is invisible to `list-sessions` without the matching `-L` flag.
Reference #3042 and #3514.

---

### #3516 -- Docs: rig names must use underscores (not hyphens), and dolt is an undocumented install prerequisite

**Issue summary:** Two documentation gaps: (1) `gt rig add cc-mem` fails
silently while `gt rig add cc_mem` succeeds -- the hyphen restriction is
not documented in README.md or INSTALLING.md. (2) `gt dolt start` fails
when `dolt` is not installed, but `dolt` is not listed as a prerequisite
in the INSTALLING.md quickstart.

**Wiki pages checked:**
- [internal/rig](../packages/rig.md) -- `AddRig` validation, name rules
- [gt rig](../commands/rig.md) -- CLI surface
- [internal/crew](../packages/crew.md) -- `validateCrewName` (same pattern)
- [internal/deps](../packages/deps.md) -- dolt as external prerequisite
- [gt install](../commands/install.md) -- Dolt preflight, prerequisites
- [gt doctor](../commands/doctor.md) -- `NewDoltBinaryCheck`
- [gastown/drift/corrections.md](corrections.md) -- INSTALLING.md drift

**What the wiki knows:** The wiki documents both issues with precision:

1. **Rig name validation:** The `internal/rig` package page documents
   `AddRig` step 1 (`manager.go:306-321`): "reject hyphens, dots,
   spaces, path separators. Suggest an underscored alternative. Reject
   reserved names (`hq`)." The `internal/crew` package page documents
   the identical pattern: "`validateCrewName` rejects hyphens, dots, and
   spaces because they collide with agent ID parsing." The page also
   notes that "the error message includes a sanitized suggestion."

2. **Dolt as prerequisite:** The `internal/deps` package page documents
   `MinDoltVersion = "1.82.4"`, `DoltInstallURL`, and `DoltStatus`
   including `DoltNotFound`. The page explicitly states: "Beads can
   auto-install; dolt cannot. The asymmetry is intentional." The
   `gt install` page documents Dolt preflight at step 6: "Verify `dolt`
   is on `PATH` (soft -- if absent, the whole block is skipped)." The
   `gt doctor` page lists `NewDoltBinaryCheck` as an infrastructure
   prerequisite.

3. **INSTALLING.md drift already documented:** The `gt install` page
   contains a Phase 3 drift finding about INSTALLING.md showing `rigs/`
   incorrectly. The `gastown/drift/corrections.md` page has the
   correction draft. The upstream documentation gaps are a known pattern.

An investigator reading the wiki would find the hyphen rejection rule,
the dolt prerequisite and its non-auto-install behavior, and the existing
INSTALLING.md drift findings. The wiki provides all the context needed
to both understand the problem and draft documentation fixes.

**Score:** full

**Remediation:** None needed for diagnosis. File the INSTALLING.md gaps
(dolt prerequisite, rig naming rules) as new drift findings in the
corrections page if not already there. Once upstream docs are fixed,
verify the wiki's references still align.

---

## Aggregate Analysis (All 20 Issues)

Combined results from validation rounds 1-4, covering 20 open gastown
issues scored against the wiki.

### Overall Score Distribution

| Score | Round 1 | Round 2 | Round 3 | Round 4 | Total | Pct |
|---|---|---|---|---|---|---|
| Full | 0 | 2 | 2 | 3 | 7 | 35% |
| Partial | 4 | 3 | 3 | 2 | 12 | 60% |
| Miss | 1 | 0 | 0 | 0 | 1 | 5% |

**7 full, 12 partial, 1 miss across 20 issues.**

### All Issues Scored

| # | Issue | Title | Score | Round |
|---|---|---|---|---|
| 1 | #3665 | Centralize cross-platform sleep command strings | miss | 1 |
| 2 | #3661 | `bd` only has `--repo` flag, not `--rig` | partial | 1 |
| 3 | #3658 | gt polecat remove leaves zombie tmux sessions | partial | 1 |
| 4 | #3653 | Dead code: HandlePolecatDone / HandlePolecatDoneFromBead | partial | 1 |
| 5 | #3652 | witness and refinery Start() never create agent beads | partial | 1 |
| 6 | #3651 | Incorrect install instructions on 1.0 release | partial | 2 |
| 7 | #3648 | doctor: claude-settings never converges for polecats | full | 2 |
| 8 | #3626 | `go install` method on Linux complains about macOS | full | 2 |
| 9 | #3623 | Dolt connection exhaustion and slow mail/beads queries | partial | 2 |
| 10 | #3614 | Dashboard doesn't show pi/omp polecats | partial | 2 |
| 11 | #3604 | Refinery stalls during branch consolidation | partial | 3 |
| 12 | #3570 | gt daemon doesn't clean up legacy tmux sockets on upgrade | partial | 3 |
| 13 | #3565 | tmux keybindings use stale prefix pattern | full | 3 |
| 14 | #3563 | gt nudge fails with stale tmux pane ID | partial | 3 |
| 15 | #3562 | Rig init can create duplicate Dolt databases | full | 3 |
| 16 | #3554 | daemon: 'no wisp config' warning spams log every 5s | partial | 4 |
| 17 | #3539 | gt sling fails: bd update --type=agent rejected | full | 4 |
| 18 | #3538 | Windows is not a viable platform | full | 4 |
| 19 | #3537 | gt crew start launches Claude session outside tmux | partial | 4 |
| 20 | #3516 | Docs: rig names must use underscores, dolt undocumented prereq | full | 4 |

### Category Performance

**Best-performing wiki areas (most "full" scores):**

- **Build system / binary identity** -- the `BuiltProperly` ldflag,
  goreleaser config, and build pipeline are documented with verbatim
  code and error messages. Issues touching this area (#3626, #3538)
  scored full.
- **Rig lifecycle / beads infrastructure** -- the rig creation pipeline,
  beads prefix routing, custom type registration, and identity
  verification are exhaustively documented. Issues here (#3562, #3539,
  #3516) scored full.
- **Doctor health checks** -- the `ClaudeSettingsCheck` convergence
  finding (#3648) was diagnosed entirely from wiki content because both
  the doctor and hooks sides of the conflict were documented.
- **tmux keybindings** -- the binding staleness mechanism (#3565) was
  fully documented with function names and line numbers.

**Worst-performing wiki areas (most "partial" scores):**

- **Daemon internals / log-level behaviors** -- the daemon page catalogs
  heartbeat steps and tickers at a structural level but does not
  document specific warning messages, log spam patterns, or per-tick
  failure modes (#3554, #3570).
- **tmux socket management** -- socket isolation is documented at the
  API level but not at the consistency-requirement level. Issues where
  different parts of the system use mismatched socket arguments (#3537,
  #3570) fall through the cracks.
- **Cross-system interaction bugs** -- issues where two subsystems
  disagree (e.g., doctor vs hooks in #3648 which scored full because
  both sides were documented, vs refinery cherry-pick vs conflict
  classification in #3604 which scored partial because only the happy
  path was documented).
- **Process detection / agent-type assumptions** -- the dashboard
  fetcher (#3614) and zombie detection rely on Claude-specific process
  patterns, which the wiki documents at the interface level but not at
  the filtering-logic level.

### Common Gap Patterns

1. **Happy-path documentation, missing failure modes.** The most
   frequent partial-score pattern (8 of 12 partials). The wiki documents
   what code intends to do but not where it silently fails, stalls, or
   produces misleading output. Examples: refinery conflict stall (#3604),
   daemon legacy socket gap (#3570), Remove missing tmux kill (#3658),
   wisp config warning spam (#3554).

2. **Cross-page inference required.** Some issues can technically be
   diagnosed from wiki content, but only by comparing descriptions
   across 2-3 pages and noticing asymmetries. Examples: witness/refinery
   missing agent bead creation (#3652), dead code in witness handlers
   (#3653). These are partial because the signal is present but not
   synthesized.

3. **Implementation-detail depth.** The wiki documents function names,
   line numbers, and high-level step lists but sometimes stops one
   level short of the specific parameter, flag, or condition that
   matters. Examples: `--rig` vs `--repo` flag mismatch (#3661),
   `wait_timeout` vs read/write timeouts (#3623), socket `-L` flag
   consistency (#3537).

4. **Warning/error message coverage.** Specific warning and error
   messages are rarely documented in wiki pages (the `BuiltProperly`
   gate is the notable exception). When an issue's primary symptom is
   a log message, the wiki provides structural context but not the
   message itself.

### Top 5 Remediation Priorities

1. **Add failure-mode annotations to critical paths.** The highest-
   value improvement is to add "what happens when this fails" notes to
   the documented step lists for `Remove`, `Start`, heartbeat, and
   conflict-resolution paths. This addresses the dominant gap pattern
   across all 20 issues.

2. **Document tmux socket consistency requirements.** A dedicated
   section in the `internal/tmux` page (or a cross-cutting concept
   page) explaining that all tmux operations on a Gas Town session
   must use the same `-L <socket>` argument, with the failure mode
   when they don't. Addresses #3537, #3570, and related socket issues.

3. **Surface daemon warning/error messages.** The daemon page should
   catalog the warning messages the heartbeat and tickers emit, with
   their trigger conditions and expected frequency. Addresses #3554
   directly and would improve future daemon-related investigations.

4. **Synthesize cross-page asymmetries.** Where the wiki documents
   two parallel paths (e.g., polecat vs witness/refinery Start,
   HandlePolecatDone vs DiscoverCompletions), add explicit comparison
   notes that flag the differences. This converts "cross-page inference
   required" partials into fulls.

5. **Expand INSTALLING.md drift coverage.** The wiki already has one
   INSTALLING.md drift finding (rigs/ directory). Add the two from
   #3516 (dolt prerequisite, rig naming rules) to the corrections
   page for upstream PR batching.

### Validation Methodology Assessment

The 20-issue sample spans a representative range of issue types:

- **Bug reports:** 14 (process/session bugs, data-plane bugs, log bugs)
- **Documentation issues:** 3 (#3651, #3516, #3626)
- **Feature requests / platform support:** 2 (#3665, #3538)
- **Dead code / cleanup:** 1 (#3653)

The 35% full-score rate indicates the wiki provides sufficient depth for
roughly one in three open issues to be diagnosed without source access.
The 60% partial rate (with zero misleading content) indicates the wiki
consistently points investigators to the right code locations, even when
it doesn't document the specific failure mode. The single miss (#3665)
was a feature request for a package that doesn't exist yet -- the wiki
correctly had no coverage.

**Overall assessment:** The wiki is a strong navigational aid and a
moderate diagnostic tool. Its primary limitation is depth at the
failure-mode level, not breadth or accuracy.
