---
title: Cross-page inference validation
type: drift
status: active
topic: gastown
created: 2026-04-17
updated: 2026-04-17
phase: cross-page
---

# Cross-page inference validation

Re-scored the same 20 open gastown issues tested in validation rounds 1-4
and the Phase 8 retest against the current wiki state, which now includes
6 investigation workflows, 26 detail tables, and 7 comparison tables from
the cross-page inference batches.

## Summary table

| # | Issue | Phase 8 | New | Delta | Gap type |
|---|---|---|---|---|---|
| 3665 | Centralize cross-platform sleep strings | miss | miss | unchanged | -- |
| 3661 | `bd` `--rig` vs `--repo` | partial | partial | unchanged | detail-gap |
| 3658 | polecat remove zombie sessions | full | full | unchanged | -- |
| 3653 | Dead code: HandlePolecatDone | partial | partial | unchanged | cross-page-inference |
| 3652 | witness/refinery Start() no agent beads | partial | full | improved | -- |
| 3651 | Incorrect install instructions | partial | partial | unchanged | detail-gap |
| 3648 | doctor claude-settings convergence | full | full | unchanged | -- |
| 3626 | `go install` Linux macOS complaint | full | full | unchanged | -- |
| 3623 | Dolt connection exhaustion | partial | partial | unchanged | detail-gap |
| 3614 | Dashboard pi/omp detection | partial | partial | unchanged | detail-gap |
| 3604 | Refinery stall branch consolidation | full | full | unchanged | -- |
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

| Score | Phase 8 | Cross-page | Change |
|---|---|---|---|
| Full | 9 (45%) | 10 (50%) | +1 |
| Partial | 10 (50%) | 9 (45%) | -1 |
| Miss | 1 (5%) | 1 (5%) | 0 |

**Result: 10 full (50%), 9 partial (45%), 1 miss (5%).**

Improvement of +5 percentage points from 45% to 50%. One issue moved
from partial to full.

---

## Per-issue scoring rationale

### #3665 -- Centralize cross-platform sleep command strings

**Phase 8: miss. New: miss.**

The issue proposes a new `internal/shellcmd` package that does not exist.
The wiki has no page for this package. The investigation workflows do not
cover cross-platform shell command generation because it is a feature
request for code that does not exist, not a failure in existing code. No
investigation workflow applies. Score unchanged.

---

### #3661 -- `bd` only has `--repo` flag, not `--rig`

**Phase 8: partial. New: partial. Gap: detail-gap.**

**Investigation workflow tried:** data-plane. The data-plane workflow
covers beads subprocess invocations (Step 7-8) and documents the
subprocess path, stale PID cleanup, and `BEADS_DIR` handling. The
connection management comparison table at data-plane.md documents
"Subprocess: one `bd` process per call (~600ms overhead)" and the
`bdError` wrapping. However, neither the workflow nor the comparison
tables document the specific `--rig` vs `--repo` flag name mismatch at
`beads.go:1229-1231`. The wiki points to the right file and the right
subprocess invocation pattern, but not the wrong flag name.

Score unchanged.

---

### #3658 -- gt polecat remove leaves zombie tmux sessions

**Phase 8: full. New: full.**

The agent-lifecycle investigation workflow Step 11 (Zombie/hung
detection) and Step 2 (Polecat start failures) provide additional
navigation to the existing entity-page coverage. Score was already full.

---

### #3653 -- Dead code: HandlePolecatDone / HandlePolecatDoneFromBead

**Phase 8: partial. New: partial. Gap: cross-page-inference.**

**Investigation workflow tried:** agent-lifecycle. The workflow covers
agent start/stop/crash failures but does not address dead code
identification -- that is a static analysis concern, not a runtime
diagnostic.

The witness package page still documents both `HandlePolecatDone`
(`handlers.go:118+`) and `DiscoverCompletions` (`handlers.go:1700+`) as
separate entry points without flagging that the former is dead code
exercised only in tests. The protocol package page at
`gastown/packages/protocol.md` also documents `HandlePolecatDone` as a
live handler branch.

The cross-page inference work did NOT add a comparison table contrasting
`HandlePolecatDone` vs `DiscoverCompletions` with a dead-code
annotation. The polecat-lifecycle comparison table
(`gastown/workflows/polecat-lifecycle.md`) compares Start() behavior
across agents but does not cover completion-handling code paths.

An investigator must still read the witness package page, notice both
code paths, and infer from the absence of production callers that
`HandlePolecatDone` is dead code. Score unchanged.

---

### #3652 -- witness and refinery Start() never create agent beads

**Phase 8: partial. New: full. IMPROVED.**

**Investigation workflow tried:** agent-lifecycle Step 4 (Post-start
failures) covers what happens after an agent starts, including
`gt prime` failure and startup dialog issues. But the actual improvement
comes from the lifecycle comparison table.

The cross-page inference Batch 3 added an explicit **lifecycle comparison
table** at `gastown/workflows/polecat-lifecycle.md` (line 568-594). The
table row `Creates agent bead` shows:

| Polecat | Witness | Refinery | Crew |
|---|---|---|---|
| Yes (`createAgentBeadWithRetry` at `manager.go:308-326`) | No | No | No |

The accompanying asymmetry annotation is explicit:

> **Asymmetry: agent bead creation.** Only polecat Start() creates an
> agent bead. Witnesses, refineries, and crew do not have agent beads
> tracking their lifecycle in the beads database. **By design:** polecats
> have persistent identity and a CV chain (work history); infrastructure
> agents (witness, refinery) are replaceable and restarted by the daemon's
> heartbeat loop...

This directly addresses the issue. An investigator following the
agent-lifecycle workflow would find the comparison table and see the
explicit No/No/No for witness/refinery/crew agent bead creation. The
asymmetry note even names the specific function (`CreateOrReopenAgentBead`)
and confirms its absence in the other managers' source files. The "By
design" rationale also provides useful context about whether this is
intentional or a bug.

Score improved from partial to full. The comparison table is exactly the
kind of cross-page synthesis that was needed.

---

### #3651 -- Incorrect install instructions on 1.0 release

**Phase 8: partial. New: partial. Gap: detail-gap.**

**Investigation workflow tried:** workspace-setup. Step 2 covers
`gt install` failures in detail, including dependency checks, port
conflicts, and partial scaffolding. However, the workflow covers runtime
installation failures, not documentation errors in release notes.

The wiki documents the npm package scope as `@gastown/gt` (from
`goreleaser-yml.md` and `inventory/auxiliary.md`), while the issue
reports the release notes incorrectly say `@anthropic/gastown`. The wiki
has the correct scope but the discrepancy with release notes is not
within the wiki's scope (release notes are on GitHub, not in source
code). Score unchanged.

---

### #3648 -- doctor claude-settings convergence

**Phase 8: full. New: full.**

The monitoring investigation workflow Step 2 (Status data accuracy) is
tangentially related but the diagnosis relies on the doctor and hooks
entity pages, which have been thorough since Phase 4. Score unchanged.

---

### #3626 -- `go install` Linux macOS complaint

**Phase 8: full. New: full.**

The workspace-setup investigation workflow Step 2a (dependency checks)
and the existing `gt` binary page's `BuiltProperly` ldflag documentation
continue to cover this fully. Score unchanged.

---

### #3623 -- Dolt connection exhaustion

**Phase 8: partial. New: partial. Gap: detail-gap.**

**Investigation workflow tried:** data-plane. Step 6 (Is the database
healthy?) covers health metrics including `GetActiveConnectionCount` and
`DefaultMaxConnections = 1000`. The connection management comparison
table documents per-call vs pooled connections, timeout mechanisms, and
the timeout/capacity thresholds table lists all relevant values.

The data-plane investigation mentions `wait_timeout` at line 276 but
only in passing: "Doltserver's 5-minute timeouts are for the MySQL
`wait_timeout` equivalent, not for individual queries." This is NOT the
same as documenting Dolt's own `wait_timeout` default of 28800s (8
hours), which is the root cause of the death spiral described in the
issue.

The comparison table documents:
- Max connections: 1000
- Read/write timeout: 5 min each
- `bd` subprocess timeouts: 60s
- In-process store timeout: 30s

But it does NOT document:
- Dolt's `wait_timeout` default (28800s)
- Connection accumulation pattern from short-lived `bd` processes
- The death spiral under concurrent agent load

The wiki improved on this issue (the comparison table makes it easier to
see the connection overhead and the timeout landscape), but the core gap
remains: `wait_timeout` and connection lifecycle under load. Score
unchanged, but the investigation workflow provides better navigation.

---

### #3614 -- Dashboard doesn't show pi/omp polecats

**Phase 8: partial. New: partial. Gap: detail-gap.**

**Investigation workflow tried:** monitoring. Step 6-7 cover dashboard
startup and data issues. Step 7a (Empty convoy panel) addresses empty
data but attributes it to Dolt being down or no convoys existing.

The monitoring workflow Step 2a (Zombie sessions shown as "running")
documents the `IsAgentAlive` process-name check and `GT_PROCESS_NAMES`,
which is the mechanism that likely causes pi/omp polecats to be
invisible. The workflow says: "Compare the pane process against what
`GT_PROCESS_NAMES` expects (set at agent startup). If the process name
doesn't match, `IsAgentAlive` returns false."

This is closer than before -- the monitoring workflow's zombie detection
section implicitly explains why non-Claude agents might be invisible
(their process names don't match `GT_PROCESS_NAMES`). However, the
connection from `GT_PROCESS_NAMES` to the dashboard fetcher is not
explicit. The monitoring workflow documents the status path's process-name
detection, but the dashboard fetcher may use a different code path
(`web.NewLiveConvoyFetcher`) that is not covered in the same detail.

Score unchanged. The monitoring workflow improved navigation but the
dashboard-specific agent-type detection filtering remains undocumented.

---

### #3604 -- Refinery stall branch consolidation

**Phase 8: full. New: full.**

The agent-lifecycle investigation workflow Step 7-8 covers refinery
start/operational failures. The existing entity page coverage from
Phase 8 remains sufficient. Score was already full.

---

### #3570 -- Daemon legacy tmux sockets

**Phase 8: partial. New: partial. Gap: failure-mode-gap.**

**Investigation workflow tried:** daemon-infrastructure. Step 2
(Why isn't the daemon running?) covers shutdown sentinel blocking,
lock held by another process, and rig backend check failure. Step 8
(Shutdown failures) covers `gt down` Phase 4c legacy socket cleanup
explicitly. The lifecycle comparison table contrasts daemon/boot/deacon
startup and shutdown behavior.

However, the gap persists: the investigation workflow documents the
legacy cleanup in `gt down` (Step 8c) but does NOT flag that the daemon's
`ensureDeaconRunning` only checks the new socket. The asymmetry between
"cleanup exists in shutdown" and "cleanup absent from startup" is still
implicit. The lifecycle comparison table covers startup behavior but
does not include a row for "legacy socket check."

Score unchanged.

---

### #3565 -- tmux keybindings stale prefix

**Phase 8: full. New: full.**

Score was already full. The monitoring workflow Step 4 (Agent listing
issues) tangentially relates but the entity page coverage was already
sufficient. Score unchanged.

---

### #3563 -- Nudge stale pane ID

**Phase 8: partial. New: partial. Gap: failure-mode-gap.**

**Investigation workflow tried:** message-delivery. Step 5 (Did tmux
send-keys actually deliver?) covers the `NudgeSession` protocol in
detail, including 7 specific failure points (a-g). Step 5a covers
"Target session doesn't exist" and mentions `session.ParseAddress`. The
workflow documents the fallback chain but focuses on the `NudgeSession`
protocol steps, not on the pane ID resolution that happens before
`send-keys`.

The session package documents `GT_PANE_ID capture failure` at
`lifecycle.go:282-284` in its failure modes section. The nudge command
page documents the delivery path. But the specific failure mode of
`GT_PANE_ID` becoming stale after a crew stop/start cycle (set once at
session creation, never refreshed on restart) is still not explicitly
documented anywhere.

The message-delivery investigation workflow is the right entry point for
this issue, and it provides excellent coverage of the `NudgeSession`
protocol, but the pane ID staleness is upstream of the protocol (it's in
the pane resolution step before `send-keys` is called). Score unchanged.

---

### #3562 -- Rig init duplicate Dolt databases

**Phase 8: full. New: full.**

The workspace-setup investigation workflow Step 3-4 covers rig add
failures including beads database initialization. The data-plane workflow
Step 4 covers orphan databases. The existing entity page coverage was
already sufficient. Score unchanged.

---

### #3554 -- Daemon wisp config warning spam

**Phase 8: partial. New: partial. Gap: failure-mode-gap.**

**Investigation workflow tried:** daemon-infrastructure. Step 4 (Is the
daemon healthy?) covers heartbeat checking, binary version mismatch, and
agent restart tracking. The lifecycle comparison table covers startup
and shutdown behavior.

The daemon-infrastructure workflow does not mention wisp config checks or
the per-tick warning pattern. The wisp package page documents the config
storage API but not the daemon consumer that reads it on every tick.
The specific warning message ("no wisp config for <rig> - parked state
may have been lost") and its per-tick emission pattern remain
undocumented. Score unchanged.

---

### #3539 -- Sling bd update type rejection

**Phase 8: full. New: full.**

Score was already full. Score unchanged.

---

### #3538 -- Windows not viable: tmux dependency

**Phase 8: full. New: full.**

The workspace-setup investigation workflow Step 2 (gt install failures)
documents dependency checks including dolt and beads, but the tmux
dependency is documented across multiple entity pages. The cross-platform
concerns were already thoroughly documented from Phase 8. Score unchanged.

---

### #3537 -- Crew start outside tmux

**Phase 8: partial. New: partial. Gap: failure-mode-gap.**

**Investigation workflow tried:** agent-lifecycle Step 9 (Crew start
failures) covers crew workspace not found, session already running,
invalid crew name, and clone failure. Step 3 (Session creation failures)
covers tmux session creation including socket permissions and TOCTOU
races. The monitoring workflow Step 4d (Session on wrong tmux socket)
documents per-town socket naming.

However, the specific failure mode where `crew Start()` creates a
session on one tmux socket but `crew at` / `tmux list-sessions` queries
a different socket remains undocumented in the investigation workflows.
The monitoring workflow Step 4d explains per-town socket naming but does
not connect it to crew start as a failure scenario.

The polecat-lifecycle comparison table shows crew uses
`NewSessionWithCommandAndEnv` (with `-e` env flags) but does not cover
socket selection behavior. Score unchanged.

---

### #3516 -- Rig names underscores, dolt prereq

**Phase 8: full. New: full.**

The workspace-setup investigation workflow Step 2b (Dolt dependency) and
Step 3c (Invalid git URL / rig names) provide additional navigation but
the entity page coverage was already sufficient. Score unchanged.

---

## Gap type distribution (remaining partials)

| Gap type | Count | Issues |
|---|---|---|
| failure-mode-gap | 4 | #3570, #3563, #3554, #3537 |
| detail-gap | 4 | #3661, #3651, #3623, #3614 |
| cross-page-inference | 1 | #3653 |

## Investigation workflows used

| Workflow | Issues tested against | Helped? |
|---|---|---|
| [message-delivery](../workflows/investigations/message-delivery.md) | #3563 | Partial -- excellent NudgeSession protocol coverage but pane ID staleness is upstream of the protocol |
| [data-plane](../workflows/investigations/data-plane.md) | #3661, #3623 | Partial -- connection management comparison table is useful but `wait_timeout` default and `--rig`/`--repo` flag detail still missing |
| [agent-lifecycle](../workflows/investigations/agent-lifecycle.md) | #3652, #3653, #3658, #3604, #3537 | Strong -- directly led to the #3652 improvement via the comparison table; good navigation for existing fulls; did not help #3653 (dead code) or #3537 (socket) |
| [daemon-infrastructure](../workflows/investigations/daemon-infrastructure.md) | #3570, #3554 | Limited -- lifecycle comparison table is detailed but neither legacy socket startup gap nor wisp config warning are covered |
| [workspace-setup](../workflows/investigations/workspace-setup.md) | #3651, #3626, #3516, #3538, #3562 | Good -- provides navigation to existing coverage; dependency checks well documented |
| [monitoring](../workflows/investigations/monitoring.md) | #3614, #3648 | Partial -- `GT_PROCESS_NAMES` mechanism documented but not connected to dashboard fetcher specifically |

## Detail tables and comparison tables used

| Table | Location | Issues it helped |
|---|---|---|
| Start() lifecycle comparison | `polecat-lifecycle.md` lines 562-584 | #3652 (improved to full), #3653 (partial -- table doesn't cover completion handlers) |
| Connection management comparison | `data-plane.md` lines 240-298 | #3623 (better navigation, still partial -- missing `wait_timeout` default) |
| Daemon/boot/deacon startup comparison | `daemon-infrastructure.md` lines 370-406 | #3570 (context but no improvement -- table lacks legacy socket row) |
| Daemon/boot/deacon shutdown comparison | `daemon-infrastructure.md` lines 409-427 | General context; no direct scoring improvement |
| Timeout and capacity thresholds | `data-plane.md` lines 280-289 | #3623 (useful reference, still missing `wait_timeout`) |

## Analysis

### What improved

One issue moved from partial to full:

1. **#3652 (witness/refinery Start() no agent beads):** The lifecycle
   comparison table at `polecat-lifecycle.md` explicitly shows "Creates
   agent bead: Yes / No / No / No" with a named asymmetry annotation
   citing the specific function and confirming its absence in the other
   managers. This is the canonical example of cross-page inference
   working as intended -- information that previously required comparing
   three separate package pages is now synthesized into a single
   comparison row.

### What did not improve

The remaining 9 partials and 1 miss break down as:

**Failure-mode-gap (4):** The investigation workflows cover the right
symptom domains but do not flag the specific failure mode:
- #3570: daemon startup vs `gt down` legacy socket cleanup asymmetry
  (workflow covers shutdown cleanup but not startup omission)
- #3563: `GT_PANE_ID` staleness across session restart (workflow covers
  NudgeSession protocol but not pane ID lifecycle)
- #3554: wisp config warning message spam on every daemon tick (workflow
  covers daemon health but not wisp config checks)
- #3537: tmux socket consistency across crew start/attach (workflow
  covers session creation but not socket selection mismatch)

**Detail-gap (4):** The wiki covers the right structural area but stops
one level short:
- #3661: `--rig` vs `--repo` flag name in subprocess calls
- #3651: npm scope in release notes vs source tree
- #3623: `wait_timeout` default (28800s) and connection death spiral
  (comparison table shows timeouts but not the Dolt-side default)
- #3614: dashboard agent-type detection filtering (monitoring workflow
  documents `GT_PROCESS_NAMES` but not the dashboard fetcher's
  agent-type logic)

**Cross-page-inference (1):** Still requires comparing code paths across
pages:
- #3653: `HandlePolecatDone` is dead code (needs explicit annotation or
  a completion-handler comparison table)

**Miss (1):**
- #3665: feature request for code that does not exist

### What the cross-page inference batches accomplished

The investigation workflows provide excellent **navigation** for
diagnosing issues -- they route investigators to the right entity pages
and the right code locations. The comparison tables provide **explicit
asymmetry annotations** that eliminate the need to compare multiple pages.

The single improvement (#3652) demonstrates the value model: when a
comparison table explicitly names an asymmetry that previously required
cross-page inference, the issue becomes fully diagnosable from the wiki.

The investigation workflows did not convert any additional partials
because the remaining gaps are at a different level:
- **Failure-mode-gap** issues need specific failure scenarios added to
  entity pages, not better navigation
- **Detail-gap** issues need specific parameter values or configuration
  defaults, not broader coverage
- The remaining **cross-page-inference** issue (#3653) needs a dead-code
  annotation, which is a different kind of synthesis than lifecycle
  comparison

### Remaining path to 60% target

To reach 12/20 full (60%), two more partials must be converted beyond
the current 10. The highest-value targets:

1. **#3653 (cross-page-inference):** Add a dead-code annotation to the
   witness package page flagging `HandlePolecatDone` and
   `HandlePolecatDoneFromBead` as test-only code superseded by
   `DiscoverCompletions`. (+1 full)

2. **#3563 (failure-mode-gap):** Add a note to the session or nudge
   package page about `GT_PANE_ID` staleness across session restarts:
   "set once at session creation, never refreshed on restart." The
   message-delivery investigation workflow already routes to the right
   area. (+1 full)

These two changes would yield 12/20 = 60%.
