---
title: Phase 8 validation retest
type: drift
status: active
topic: gastown
created: 2026-04-17
updated: 2026-04-17
phase: 8
---

# Phase 8 validation retest

Re-scored the same 20 open gastown issues tested in validation rounds 1-4
(pre-Phase 8) against the current wiki state, which now includes
`## Failure modes` sections on 138 entity/command pages.

## Summary table

| # | Issue | Title | Original | New | Delta | Gap type |
|---|---|---|---|---|---|---|
| 3665 | Centralize cross-platform sleep strings | miss | miss | unchanged | -- |
| 3661 | `bd` `--rig` vs `--repo` | partial | partial | unchanged | detail-gap |
| 3658 | polecat remove zombie sessions | partial | full | improved | -- |
| 3653 | Dead code: HandlePolecatDone | partial | partial | unchanged | cross-page-inference |
| 3652 | witness/refinery Start() no agent beads | partial | partial | unchanged | cross-page-inference |
| 3651 | Incorrect install instructions | partial | partial | unchanged | detail-gap |
| 3648 | doctor claude-settings convergence | full | full | unchanged | -- |
| 3626 | `go install` Linux macOS complaint | full | full | unchanged | -- |
| 3623 | Dolt connection exhaustion | partial | partial | unchanged | detail-gap |
| 3614 | Dashboard pi/omp detection | partial | partial | unchanged | detail-gap |
| 3604 | Refinery stall branch consolidation | partial | full | improved | -- |
| 3570 | Daemon legacy tmux sockets | partial | partial | unchanged | failure-mode-gap |
| 3565 | tmux keybindings stale prefix | full | full | unchanged | -- |
| 3563 | Nudge stale pane ID | partial | partial | unchanged | failure-mode-gap |
| 3562 | Rig init duplicate Dolt databases | full | full | unchanged | -- |
| 3554 | Daemon wisp config warning spam | partial | partial | unchanged | failure-mode-gap |
| 3539 | Sling bd update type rejection | full | full | unchanged | -- |
| 3538 | Windows not viable: tmux dependency | full | full | unchanged | -- |
| 3537 | Crew start outside tmux | partial | partial | unchanged | failure-mode-gap |
| 3516 | Rig names underscores, dolt prereq | full | full | unchanged | -- |

## Aggregate scores

| Score | Original | New | Change |
|---|---|---|---|
| Full | 7 (35%) | 9 (45%) | +2 |
| Partial | 12 (60%) | 10 (50%) | -2 |
| Miss | 1 (5%) | 1 (5%) | 0 |

**Result: 9 full (45%), 10 partial (50%), 1 miss (5%).**

**Target of 60% full (12/20) NOT met.** Improvement of +10 percentage
points from 35% to 45%.

---

## Per-issue scoring rationale

### #3665 -- Centralize cross-platform sleep command strings

**Original: miss. New: miss.**

The issue proposes a new `internal/shellcmd` package that does not exist.
The wiki has no page for this package and no cross-platform sleep-string
concept page. Phase 8 failure mode sections do not address this because
it is a feature request for code that does not exist, not a failure in
existing code. Score unchanged.

---

### #3661 -- `bd` only has `--repo` flag, not `--rig`

**Original: partial. New: partial. Gap: detail-gap.**

The wiki documents the `internal/beads` package's subprocess invocations
of `bd` and the `internal/daemon` page's `bd list` usage. However, the
specific `--rig` vs `--repo` flag mismatch at `beads.go:1229-1231` and
`daemon.go:2632` is not documented anywhere in the wiki. The Phase 8
failure mode sections for the beads package cover `bd` binary availability
and store timeouts but not flag-name mismatches in subprocess calls.
Score unchanged -- this remains a detail-level gap where the wiki points
to the right file but not the right parameter.

---

### #3658 -- gt polecat remove leaves zombie tmux sessions

**Original: partial. New: full. IMPROVED.**

The wiki now provides enough content to diagnose this issue without source
access:

1. The `gt polecat` command page at `polecat.go:474-493` explicitly
   documents zombie tmux session detection in `list`: "Zombie tmux
   sessions: sessions without matching worktree directories. These occur
   when a worktree is deleted but the tmux session persists (incomplete
   nuke or session naming mismatch)."

2. The `gt polecat` command page's `remove` section documents that
   `mgr.Remove(polecatName, force)` is the only cleanup action -- no
   mention of tmux session kill.

3. The `polecat` package failure modes section documents partial cleanup
   on spawn failure and the rollback gap.

4. The `polecat` command failure modes document the `StartSession` partial
   completion where session kill occurs but bead state is not rolled back.

An investigator reading the wiki would see that `remove` only calls
`mgr.Remove` without an explicit tmux session kill, and the `list`
command's zombie detection explicitly describes the "worktree deleted but
tmux session persists" scenario. The combination is sufficient to
diagnose the issue.

---

### #3653 -- Dead code: HandlePolecatDone / HandlePolecatDoneFromBead

**Original: partial. New: partial. Gap: cross-page-inference.**

The witness package page still documents both `HandlePolecatDone`
(`handlers.go:118+`) and `DiscoverCompletions` (`handlers.go:1700+`) as
separate entry points without flagging that the former is dead code only
exercised in tests. The Phase 8 failure modes section covers silent
suppression in session lifecycle cleanup but does not address dead code
identification. An investigator must still compare the two code paths
across pages and notice the asymmetry. Score unchanged.

---

### #3652 -- witness and refinery Start() never create agent beads

**Original: partial. New: partial. Gap: cross-page-inference.**

The witness and refinery failure mode sections cover silent suppression
in session lifecycle cleanup but do not document the absence of agent bead
creation in `Start()`. The polecat package still documents
`createAgentBeadWithRetry` while witness and refinery do not, but the
asymmetry is not explicitly flagged. An investigator must compare three
package pages and notice what is absent from two of them. Score unchanged.

---

### #3651 -- Incorrect install instructions on 1.0 release

**Original: partial. New: partial. Gap: detail-gap.**

The wiki documents the npm package scope as `@gastown/gt` (from
`goreleaser-yml.md` and `inventory/auxiliary.md`), while the issue reports
the release notes incorrectly say `@anthropic/gastown`. The wiki has the
correct scope but does not verify it against the GitHub release notes,
so an investigator would find the correct package name but not be able to
confirm the release notes discrepancy without checking the release page.
Phase 8 added failure modes to the `gt install` command page but those
cover runtime failures, not documentation errors in release notes. Score
unchanged.

---

### #3648 -- doctor claude-settings convergence

**Original: full. New: full.**

Still fully diagnosable. Both the doctor and hooks sides of the
convergence conflict are documented with enough detail to understand the
disagreement. Phase 8 failure modes on the doctor command page add
further context (no short-circuit on prerequisite failure). Score
unchanged.

---

### #3626 -- `go install` Linux macOS complaint

**Original: full. New: full.**

The `BuiltProperly` ldflag gate at `root.go:94-107` remains thoroughly
documented with verbatim code, the misleading macOS error message, and
the bypass. Score unchanged.

---

### #3623 -- Dolt connection exhaustion

**Original: partial. New: partial. Gap: detail-gap.**

The doltserver package page documents `DefaultMaxConnections = 1000`,
read/write timeouts, and the CLOSE_WAIT prevention rationale. The Phase 8
failure modes for the doltserver package cover stale PID files, lock files,
and connection close errors. However, the wiki still does not document
`wait_timeout` (the MySQL idle-connection timeout, defaulting to 28800s /
8 hours), which is the root cause of connection accumulation from
short-lived `bd` processes. The connection pooling absence and the
sequential `bd` subprocess spawning pattern in `gt mail` are also not
covered. Score unchanged -- the gap is `wait_timeout` and connection
lifecycle under load.

---

### #3614 -- Dashboard doesn't show pi/omp polecats

**Original: partial. New: partial. Gap: detail-gap.**

The dashboard command failure modes (Phase 8) cover port availability,
browser open failure, and config load failure -- but not agent-type
detection filtering. The wiki still does not document whether the
dashboard fetcher filters by Claude-specific process names vs. generic
agent process names. The `gt down` page documents
`findOrphanedClaudeProcesses` filtering by command name
(`claude`, `claude-code`, `codex`, `node`), which hints at the pattern,
but this is in the shutdown path, not the dashboard path. Score unchanged.

---

### #3604 -- Refinery stall branch consolidation

**Original: partial. New: full. IMPROVED.**

The refinery package failure modes section now explicitly documents:
"Failed merge leaves partial state... merge state files and branch
references may persist after a failed batch merge, requiring manual
cleanup." The refinery package page also documents conflict resolution
mechanisms including `createConflictResolutionTaskForMR`
(`engineer.go:1342+`) which "spawns a synthesis task bead to handle a
conflicting MR." The `MergeResult` phase types include `conflict` as a
documented terminal state. The sling command failure modes add context
about rollback gaps.

An investigator reading the wiki would now find: (1) the merge pipeline
can leave dirty state on conflict, (2) the refinery has 164 silent error
suppressions making diagnostic trails absent, (3) conflict resolution
spawns a synthesis task rather than auto-resolving. This matches the
issue's description of the refinery stalling with a dirty worktree and
no escalation. The failure mode section directly addresses the "leaves
dirty worktree" symptom.

---

### #3570 -- Daemon legacy tmux sockets

**Original: partial. New: partial. Gap: failure-mode-gap.**

The daemon failure modes section (Phase 8) covers patrol loop partial
failures, Dolt server management, and compactor cleanup. However, it does
not document the specific failure mode where daemon startup fails to
detect sessions on legacy sockets. The `gt down` page still documents
`cleanupLegacyDefaultSocket` and `cleanupLegacyBaseSocket` in Phase 4c,
but the gap that daemon startup does NOT call these cleanup functions
remains implicit. An investigator would find the cleanup code in `gt down`
and the daemon's startup sequence, but must infer the gap themselves.
Score unchanged.

---

### #3565 -- tmux keybindings stale prefix

**Original: full. New: full.**

The binding staleness mechanism remains fully documented. Score unchanged.

---

### #3563 -- Nudge stale pane ID

**Original: partial. New: partial. Gap: failure-mode-gap.**

The nudge command failure modes (Phase 8) cover queue delivery failures,
poller start failures, and the `watchAndDeliver` consumed-but-not-delivered
gap. The session package failure modes now document `GT_PANE_ID capture
failure` at `lifecycle.go:282-284`: "if `GetPaneID` fails, the pane ID
env var is not set, and liveness checks fall back to the slower scanning
path." However, the specific failure mode of `GT_PANE_ID` becoming stale
after a session restart (set once, never refreshed) is still not
explicitly flagged. The nudge delivery path documents the fallback chain
but not the root cause of the stale pane ID. Score unchanged -- the
capture failure is now documented but the staleness-across-restart
scenario is not.

---

### #3562 -- Rig init duplicate Dolt databases

**Original: full. New: full.**

Still fully documented. The rig package notes section even explicitly
documents the orphan Dolt DB scenario: "if `InitRig` creates a Dolt
database and then bd init fails mid-way, the orphan Dolt DB persists."
Phase 8 failure modes on the rig package add session lifecycle cleanup
coverage. Score unchanged.

---

### #3554 -- Daemon wisp config warning spam

**Original: partial. New: partial. Gap: failure-mode-gap.**

The daemon failure modes section (Phase 8) covers patrol loop partial
failures and compactor zombie sessions but does not document the specific
wisp-config warning message or its trigger condition (missing
`.beads-wisp/config/<rig>.json` for rigs never parked). The wisp package
documents the config storage API but not the daemon consumer that reads
it on a loop. Score unchanged -- the specific warning message and its
per-tick emission pattern remain undocumented.

---

### #3539 -- Sling bd update type rejection

**Original: full. New: full.**

The custom type mechanism, registration flow, and version dependency
remain comprehensively documented. Phase 8 failure modes on sling add
partial completion and rollback gap coverage. Score unchanged.

---

### #3538 -- Windows not viable: tmux dependency

**Original: full. New: full.**

Phase 8 failure modes on multiple pages now explicitly flag cross-platform
concerns (tmux, session, doltserver all have "Untested" Windows shim
annotations). Score unchanged but the existing full score is reinforced
by the new cross-platform failure mode annotations.

---

### #3537 -- Crew start outside tmux

**Original: partial. New: partial. Gap: failure-mode-gap.**

The crew command failure modes (Phase 8) cover town log event suppression
and capture pane error suppression. The crew package failure modes cover
session lifecycle cleanup. The session package failure modes document
`GT_PANE_ID` capture failure and `SetEnvironment` failures. However, the
specific failure mode where crew `Start()` creates a session on one tmux
socket but `crew at` / `list-sessions` queries a different socket remains
undocumented. The socket consistency requirement across tmux operations
is still only implicit in the tmux package page. Score unchanged.

---

### #3516 -- Rig names underscores, dolt prereq

**Original: full. New: full.**

Both issues (hyphen rejection in rig names, dolt prerequisite) remain
fully documented. Score unchanged.

---

## Gap type distribution (remaining partials)

| Gap type | Count | Issues |
|---|---|---|
| failure-mode-gap | 4 | #3570, #3563, #3554, #3537 |
| detail-gap | 4 | #3661, #3651, #3623, #3614 |
| cross-page-inference | 2 | #3653, #3652 |

## Analysis

### What improved

Two issues moved from partial to full:

1. **#3658 (polecat remove zombies):** The `gt polecat` command page's
   zombie detection documentation at `polecat.go:474-493` and the
   `remove` section's limited cleanup scope combine to make the
   session-without-worktree failure mode diagnosable.

2. **#3604 (refinery stall):** The refinery package failure modes section
   directly documents the "failed merge leaves partial state" scenario
   including persistent merge state files and branch references.

Both improvements come directly from Phase 8's failure mode sections
surfacing issues that were previously implicit in happy-path
documentation.

### What did not improve

The remaining 10 partials fall into three categories:

**Failure-mode-gap (4):** The Phase 8 sections added failure modes but
not the *specific* failure mode that the issue describes. These are cases
where the wiki's failure mode audit found different problems than the
ones reported in the issues:
- #3570: daemon startup vs `gt down` socket cleanup asymmetry
- #3563: `GT_PANE_ID` staleness across session restart
- #3554: wisp config warning message spam on every daemon tick
- #3537: tmux socket consistency across crew start/attach

**Detail-gap (4):** The wiki covers the right area at structural level
but stops one level short of the specific parameter, flag, or detection
logic:
- #3661: `--rig` vs `--repo` flag name in subprocess calls
- #3651: npm scope in release notes vs source tree
- #3623: `wait_timeout` default and connection lifecycle
- #3614: dashboard agent-type detection filtering

**Cross-page-inference (2):** The information exists across multiple
pages but the asymmetry is not synthesized:
- #3653: `HandlePolecatDone` is dead code (needs explicit annotation)
- #3652: witness/refinery `Start()` lack agent bead creation (needs
  explicit asymmetry note)

### What would reach the 60% target

To reach 12/20 full, three more partials must be converted. The
highest-value targets:

1. **#3652 + #3653 (cross-page-inference):** Add explicit annotations to
   the witness package page: (a) flag `HandlePolecatDone` and
   `HandlePolecatDoneFromBead` as dead code exercised only in tests, and
   (b) note that `Start()` does not create agent beads, unlike polecat's
   `createAgentBeadWithRetry`. These are single-line additions to an
   existing page. (+2 fulls)

2. **#3563 (failure-mode-gap):** Add a note to the session or nudge
   package page about `GT_PANE_ID` staleness across session restarts.
   The capture failure is already documented; the missing piece is
   "set once, never refreshed on restart." (+1 full)

These three changes would yield 12/20 = 60%.
