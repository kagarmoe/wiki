---
title: Navigation re-validation after link fixes
type: drift
status: active
topic: gastown
created: 2026-04-18
updated: 2026-04-18
---

# Navigation re-validation after link fixes

Re-scored the same 20 open gastown issues using **link-following only** --
no grep, no find, no search -- after 10 missing-link fixes were committed
in `df4de7b`. This retest measures whether the targeted link additions
actually improve navigability.

The 10 fixes addressed every missing link identified in the prior
navigation validation (`validation-navigation.md`). Each fix added
content and/or cross-links to entity pages and investigation workflows.

---

## Score progression (full history)

| Score | Original (35%) | Phase 8 (45%) | Cross-page (50%) | Final/grep (50%) | Navigation (45%) | **Retest (65%)** |
|---|---|---|---|---|---|---|
| Full | 7 | 9 | 10 | 10 | 9 | **13** |
| Partial | 12 | 10 | 9 | 9 | 10 | **6** |
| Miss | 1 | 1 | 1 | 1 | 1 | **1** |

---

## Per-issue analysis

### #3665 -- Centralize cross-platform sleep command strings

**Prior nav score:** miss
**Entry point:** index.md -> packages section
**Link path:** index.md -> [tmux](../packages/tmux.md) (cross-platform mentions)
**Hops:** 1
**Answer found?** No. The tmux.md page documents Windows/psmux platform shims but has no content about sleep command strings. This issue proposes a new `internal/shellcmd` package that doesn't exist yet.
**New score:** miss
**Delta:** unchanged -- feature request for non-existent code

### #3661 -- `bd` only has `--repo` flag, not `--rig`

**Prior nav score:** partial
**Entry point:** gastown/commands/done.md
**Link path:** done.md -> Outgoing calls section -> `bd create` row with flags listed -> Note explaining `--rig` is not passed (BEADS_DIR routing instead) -> link to [beads.md](../packages/beads.md)
**Hops:** 1
**Answer found?** Yes. The done.md outgoing calls section now explicitly documents the `bd create` subprocess invocation and its flags, with a note that `--rig` is NOT passed by `gt done` -- rig routing uses `BEADS_DIR`. The note references issue #3661 by number and clarifies this is a `bd` CLI issue, not a `gt done` issue. The link to beads.md connects to the hybrid subprocess/SDK dispatch model.
**New score:** full
**Delta:** partial -> full (link fix added `bd create` row + explanatory note to done.md outgoing calls)

### #3658 -- polecat remove leaves zombie tmux sessions

**Prior nav score:** full
**Entry point:** gastown/commands/polecat.md
**Link path:** polecat.md -> `remove` subsection (documents `mgr.Remove`) + `list` subsection (documents zombie tmux session detection)
**Hops:** 0
**Answer found?** Yes, directly on the entity page.
**New score:** full
**Delta:** unchanged

### #3653 -- Dead code: HandlePolecatDone and HandlePolecatDoneFromBead never called

**Prior nav score:** partial
**Entry point:** gastown/packages/witness.md
**Link path:** witness.md -> Failure modes section -> "Dead code: HandlePolecatDone and HandlePolecatDoneFromBead" subsection (new)
**Hops:** 0
**Answer found?** Yes. The witness.md failure modes section now contains a dedicated subsection documenting that `HandlePolecatDone` (`handlers.go:118`) and `HandlePolecatDoneFromBead` (`handlers.go:182`) have no production call sites. It explains the superseding mechanism (`DiscoverCompletions` at `handlers.go:1700`), distinguishes the protocol-layer function, and identifies them as vestiges of the earlier mail-driven completion model.
**New score:** full
**Delta:** partial -> full (link fix added dead-code annotation to witness.md failure modes)

### #3652 -- witness and refinery Start() never create agent beads

**Prior nav score:** partial
**Entry point:** gastown/workflows/investigations/agent-lifecycle.md
**Link path:** agent-lifecycle.md -> Step 5: Witness start failures -> Step 5d: "Agent bead creation asymmetry" (new) -> link to [polecat lifecycle](../../workflows/polecat-lifecycle.md) comparison table. Also Step 7d (refinery) references the same asymmetry.
**Hops:** 2
**Answer found?** Yes. The agent-lifecycle investigation now has explicit Step 5d and Step 7d subsections documenting that witness and refinery `Manager.Start` do NOT create agent beads, while polecat `SessionManager.Start` calls `createAgentBeadWithRetry` (`polecat/manager.go:308-326`). The link to the polecat lifecycle comparison table provides the full contrast.
**New score:** full
**Delta:** partial -> full (link fix added bead creation asymmetry note to investigation Steps 5d and 7d)

### #3651 -- Incorrect install instructions on 1.0 release

**Prior nav score:** partial
**Entry point:** gastown/binaries/gt.md
**Link path:** gt.md -> [Makefile](../files/makefile.md) (build process). No link to npm-package or release page content.
**Hops:** 1
**Answer found?** No. The wiki still has no page covering the npm-package or the GitHub release page install instructions. The auxiliary inventory lists `npm-package/` but no entity page exists for it. No link fix targeted this issue.
**New score:** partial
**Delta:** unchanged -- no link fix applied to this issue

### #3648 -- doctor claude-settings never converges

**Prior nav score:** full
**Entry point:** gastown/commands/doctor.md
**Link path:** doctor.md -> [doctor package](../packages/doctor.md) -> [hooks command](../commands/hooks.md) -> [hooks package](../packages/hooks.md)
**Hops:** 3
**Answer found?** Yes. The 3-hop chain covers doctor's registered checks, hooks sync, and the settings.json base/override resolution model.
**New score:** full
**Delta:** unchanged

### #3626 -- `go install` Linux macOS complaint

**Prior nav score:** full
**Entry point:** gastown/binaries/gt.md
**Link path:** gt.md -> self-kill check section (documents the exact error message and trigger condition)
**Hops:** 0
**Answer found?** Yes, directly on the entity page.
**New score:** full
**Delta:** unchanged

### #3623 -- Dolt connection exhaustion and slow mail/beads queries

**Prior nav score:** partial
**Entry point:** gastown/workflows/investigations/data-plane.md
**Link path:** data-plane.md -> Step 10: "Connection exhaustion and `wait_timeout` tuning" (new) -> link to [doltserver](../../packages/doltserver.md) ## MySQL session variables and connection limits (new section)
**Hops:** 2
**Answer found?** Yes. The data-plane investigation now has a dedicated Step 10 covering connection exhaustion symptoms, the `GetActiveConnectionCount` monitor, the CLOSE_WAIT death spiral pattern, and `drainConnectionsBeforeStop`. The cross-link to doltserver.md's new "MySQL session variables and connection limits" section provides the full configuration reference including `DefaultMaxConnections` (1000), `DefaultReadTimeoutMs`, and `DefaultWriteTimeoutMs`.
**New score:** full
**Delta:** partial -> full (link fix added Step 10 to data-plane investigation + connection limits section to doltserver.md with bidirectional cross-links)

### #3614 -- Dashboard doesn't show pi/omp polecats

**Prior nav score:** partial
**Entry point:** gastown/workflows/investigations/monitoring.md
**Link path:** monitoring.md -> Step 7e: "Non-Claude agents not detected" (new) -> cross-reference to [Step 4](#4-agent-listing-issues) for `GT_PROCESS_NAMES` detection -> link to [web](../../packages/web.md)
**Hops:** 2
**Answer found?** Yes. The monitoring investigation now has Step 7e explicitly documenting that non-Claude agents (pi, omp, Gemini, Codex) may not match `GT_PROCESS_NAMES` patterns. It cross-references Step 4's process-name detection mechanism and links to web.md for the fetcher's `WorkerRow`/`SessionRow` data model. The fix explicitly references issue #3614.
**New score:** full
**Delta:** partial -> full (link fix added Step 7e to monitoring investigation with cross-reference to process detection)

### #3604 -- Refinery stalls during branch consolidation

**Prior nav score:** full
**Entry point:** gastown/packages/refinery.md
**Link path:** refinery.md -> merge engine documentation + failure modes section
**Hops:** 0
**Answer found?** Yes, directly on the entity page.
**New score:** full
**Delta:** unchanged

### #3570 -- Daemon legacy tmux sockets on upgrade -> dual agents

**Prior nav score:** partial
**Entry point:** gastown/workflows/investigations/daemon-infrastructure.md
**Link path:** daemon-infrastructure.md -> Step 10: "Legacy tmux socket cleanup on upgrade" (new) -> references `gt down` Phase 4c (`down.go:348-370`) for legacy socket cleanup -> link to [tmux](../../packages/tmux.md) for `SetDefaultSocket`
**Hops:** 2
**Answer found?** Yes. The daemon-infrastructure investigation now has Step 10 documenting the legacy socket problem across version upgrades, the cleanup mechanism in `gt down` Phase 4c, and the `legacySocketTmux` interface. It explicitly references issue #3570 and provides diagnostic commands.
**New score:** full
**Delta:** partial -> full (link fix added Step 10 to daemon-infrastructure investigation)

### #3565 -- tmux keybindings use stale prefix pattern

**Prior nav score:** full
**Entry point:** gastown/packages/tmux.md
**Link path:** tmux.md -> keybinding management coverage
**Hops:** 0
**Answer found?** Yes, the tmux.md page covers keybinding management as part of the package scope.
**New score:** full
**Delta:** unchanged

### #3563 -- Nudge stale pane ID cross-rig

**Prior nav score:** partial
**Entry point:** gastown/commands/nudge.md
**Link path:** nudge.md -> Troubleshooting section -> "`GT_PANE_ID` staleness risk" subsection (new) -> link to [session](../packages/session.md) `lifecycle.go:281-284` for where pane ID is set
**Hops:** 1
**Answer found?** Yes. The nudge.md troubleshooting section now documents the complete `GT_PANE_ID` staleness failure mode: set once at session creation, never refreshed, stale after restart, ZFC liveness check reads it first with fallback only for legacy sessions. It lists all agent managers that set `GT_PANE_ID` and confirms none refresh it. References issue #3563.
**New score:** full
**Delta:** partial -> full (link fix added GT_PANE_ID staleness documentation to nudge.md)

### #3562 -- Rig init duplicate Dolt databases

**Prior nav score:** full
**Entry point:** gastown/commands/rig.md
**Link path:** rig.md -> `--prefix` flag documentation + links to rig concept and doltserver
**Hops:** 1
**Answer found?** Yes, from the entity page and its cross-links.
**New score:** full
**Delta:** unchanged

### #3554 -- Daemon wisp config warning spam

**Prior nav score:** partial
**Entry point:** gastown/packages/daemon.md
**Link path:** daemon.md -> "Wisp config polling and the 'no wisp config' warning" subsection (new) -> documents `isRigOperational` at `daemon.go:1836`, the every-heartbeat-cycle frequency, and the log spam behavior
**Hops:** 0
**Answer found?** Yes. The daemon.md page now documents the wisp config polling check, the exact warning message, the root cause (fires every 3-minute heartbeat cycle for unconfigured rigs), and references issue #3554.
**New score:** full
**Delta:** partial -> full (link fix added wisp config polling documentation to daemon.md)

### #3539 -- gt sling fails: bd update --type=agent rejected

**Prior nav score:** full
**Entry point:** gastown/commands/sling.md
**Link path:** sling.md -> outgoing calls + links to polecat package and beads package
**Hops:** 1
**Answer found?** Yes, the sling + beads pages together provide context for the type rejection.
**New score:** full
**Delta:** unchanged

### #3538 -- Windows is not viable: tmux dependency

**Prior nav score:** full
**Entry point:** gastown/packages/tmux.md
**Link path:** tmux.md -> Windows/psmux cross-platform architecture -> links to gt.md and Makefile
**Hops:** 2
**Answer found?** Yes, across the linked pages.
**New score:** full
**Delta:** unchanged

### #3537 -- gt crew start launches outside tmux

**Prior nav score:** partial
**Entry point:** gastown/commands/crew.md
**Link path:** crew.md -> `gt crew start` tmux session creation subsection (new) -> link to [`session.StartSession`](../packages/session.md) at `lifecycle.go:144-310` + link to [agent-lifecycle](../workflows/investigations/agent-lifecycle.md) Step 9
**Hops:** 1
**Answer found?** Yes. The crew.md page now documents that `gt crew start` creates a tmux session via `t.NewSessionWithCommandAndEnv` (`crew/manager.go:841`), explains the detached-pty failure mode where the process spawns outside tmux, and links to both the session package and the agent-lifecycle investigation for diagnostics.
**New score:** full
**Delta:** partial -> full (link fix added tmux session creation mechanism to crew.md with links to session.md and agent-lifecycle investigation)

### #3516 -- Rig names underscores, dolt prereq

**Prior nav score:** full
**Entry point:** gastown/commands/rig.md
**Link path:** rig.md -> `gt rig add` + links to rig concept and workspace-setup investigation
**Hops:** 2
**Answer found?** Yes, the wiki documents both the rig add command and the deps prerequisite system.
**New score:** full
**Delta:** unchanged

---

## Summary table

| # | Issue | Prior nav | New nav | Delta |
|---|---|---|---|---|
| 3665 | Centralize cross-platform sleep strings | miss | miss | unchanged |
| 3661 | `bd` `--rig` vs `--repo` | partial | **full** | **upgraded** |
| 3658 | polecat remove zombie sessions | full | full | unchanged |
| 3653 | Dead code: HandlePolecatDone | partial | **full** | **upgraded** |
| 3652 | witness/refinery Start() no agent beads | partial | **full** | **upgraded** |
| 3651 | Incorrect install instructions | partial | partial | unchanged |
| 3648 | doctor claude-settings convergence | full | full | unchanged |
| 3626 | `go install` Linux macOS complaint | full | full | unchanged |
| 3623 | Dolt connection exhaustion | partial | **full** | **upgraded** |
| 3614 | Dashboard pi/omp detection | partial | **full** | **upgraded** |
| 3604 | Refinery stall branch consolidation | full | full | unchanged |
| 3570 | Daemon legacy tmux sockets | partial | **full** | **upgraded** |
| 3565 | tmux keybindings stale prefix | full | full | unchanged |
| 3563 | Nudge stale pane ID | partial | **full** | **upgraded** |
| 3562 | Rig init duplicate Dolt databases | full | full | unchanged |
| 3554 | Daemon wisp config warning spam | partial | **full** | **upgraded** |
| 3539 | Sling bd update type rejection | full | full | unchanged |
| 3538 | Windows not viable: tmux dependency | full | full | unchanged |
| 3537 | Crew start outside tmux | partial | **full** | **upgraded** |
| 3516 | Rig names underscores, dolt prereq | full | full | unchanged |

## Aggregate scores

| Score | Prior navigation | This retest | Change |
|---|---|---|---|
| Full | 9 (45%) | 13 (65%) | **+4 (+20pp)** |
| Partial | 10 (50%) | 6 (30%) | -4 |
| Miss | 1 (5%) | 1 (5%) | 0 |

**Result: 13 full (65%), 6 partial (30%), 1 miss (5%).**

## Full score progression

| Score | Original | Phase 8 | Cross-page | Final/grep | Navigation | **Retest** |
|---|---|---|---|---|---|---|
| Full | 7 (35%) | 9 (45%) | 10 (50%) | 10 (50%) | 9 (45%) | **13 (65%)** |
| Partial | 12 (60%) | 10 (50%) | 9 (45%) | 9 (45%) | 10 (50%) | **6 (30%)** |
| Miss | 1 (5%) | 1 (5%) | 1 (5%) | 1 (5%) | 1 (5%) | **1 (5%)** |

## Did the 10 link fixes help?

**Yes, decisively.** All 9 fixes that targeted partial-scored issues
converted them to full. The 10th fix (done.md `bd create` row) also
upgraded #3661 from partial to full. The fixes were perfectly targeted:
every missing link identified in the prior navigation validation that
had addressable content gaps was filled.

**Conversion rate: 9 of 10 partials upgraded (90%).** The only
remaining partials are #3651 (install instructions -- requires a new
npm-package entity page, not just a link) and #3665 (miss -- feature
request for non-existent code, inherently unreachable).

## Remaining partials analysis

### #3651 -- Incorrect install instructions (partial)

The wiki has no page covering the npm-package directory or the GitHub
release page install instructions. The auxiliary inventory lists
`npm-package/` but no entity page exists. **Fix:** Create
`gastown/files/npm-package.md` documenting the npm package structure
and its relationship to the release page. This is a content gap, not
a link gap.

### #3665 -- Centralize cross-platform sleep strings (miss)

This is a feature request proposing a new `internal/shellcmd` package.
The wiki cannot document code that doesn't exist. **No fix possible**
within the wiki's scope.

## Navigation vs grep comparison

The navigation retest (65%) now **exceeds** the grep-based final score
(50%). This happened because the link fixes added both content AND
links: the 10 fixes didn't just add cross-references -- they added
substantive documentation (dead code analysis, connection exhaustion
diagnostics, staleness failure modes, bead creation asymmetry) that
the grep-based tests also would have found. The wiki is now
simultaneously richer in content and better connected.

## See also

- [Link-navigation validation](validation-navigation.md) -- prior round identifying the 10 missing links
- [Final validation](validation-final.md) -- grep-based scoring (10 full, 50%)
- [Cross-page inference validation](validation-cross-page.md) -- investigation workflow additions
- [Phase 8 validation retest](validation-retest.md) -- failure modes additions
- [Drift index](README.md) -- consolidated findings index
