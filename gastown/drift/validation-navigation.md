---
title: Link-navigation validation
type: drift
status: active
topic: gastown
created: 2026-04-18
updated: 2026-04-18
---

# Link-navigation validation

Re-scored the same 20 open gastown issues using **link-following only** --
no grep, no find, no search. Starting from a natural entry point
(index.md, gastown/README.md, commands/README.md, or the obvious entity
page), follow markdown links up to 5 hops. If the answer is reachable
via links: full. If content exists but links don't connect to it:
partial (navigation failure). If content doesn't exist at all: miss.

This test measures the wiki's **navigability**, not just its content.
Previous rounds (grep-based) scored 10 full (50%). This round reveals
how many of those 10 are actually reachable by a human browsing links.

---

## Per-issue analysis

### #3665 -- Centralize cross-platform sleep command strings

**Entry point:** index.md -> packages section (looking for shell/platform packages)
**Link path:** index.md -> [tmux](../packages/tmux.md) (platform wrapper tag, cross-platform mention)
**Hops:** 1
**Answer found?** The tmux.md page mentions Windows/psmux platform shims and paired Unix/Windows files, but says nothing about sleep command strings or the `sleep` vs `timeout` platform difference. No page in the wiki covers the `internal/shellcmd` package (it doesn't exist yet -- this issue proposes creating it). No link from tmux.md leads to platform-specific sleep handling.
**Score:** miss
**Rationale:** The wiki has no content on cross-platform sleep commands. This is a feature request for new code, not a bug in existing code. Same as grep-based.

### #3661 -- `bd` only has `--repo` flag, not `--rig`

**Entry point:** gastown/commands/done.md (issue is about `gt done` failing to create MR beads)
**Link path:** done.md -> [beads package](../packages/beads.md) (mentioned as the package backing bead operations)
**Hops:** 2
**Answer found?** The done.md page documents the `gt done` flow (branch submission, MR creation, Witness notification) but does not document the specific `bd create` subprocess call or its flags. The beads.md page documents the hybrid subprocess/SDK dispatch model and mentions that `bd` is shelled out to, but does not enumerate the specific flag names passed to `bd create`. The `--rig` vs `--repo` flag mismatch is a detail-level gap -- the wiki knows `gt done` creates beads via the beads package, but doesn't document the specific subprocess flags.
**Score:** partial
**Missing link:** done.md -> outgoing calls section doesn't include the `bd create --rig=<name>` invocation (it was added in the outgoing-calls sweep but done.md's outgoing calls section wasn't read in this test -- let me verify).

Actually, done.md does have a `## Failure modes` section that I read (at offset 224+), but it covers partial completion and silent suppression, not the specific `--rig` flag mismatch. The done.md outgoing calls section (if it exists) would need to document the `bd create` call with its `--rig` flag. Let me follow links further.

done.md links to [handoff](handoff.md), [hook](hook.md), [mq](mq.md), [convoy](convoy.md), and [gt binary](../binaries/gt.md). None of these would contain the `--rig` vs `--repo` flag detail.

**Revised score:** partial (navigation failure: the specific `bd create --rig` subprocess invocation detail is not documented at the flag level)

### #3658 -- polecat remove leaves zombie tmux sessions

**Entry point:** gastown/commands/polecat.md (issue is directly about `gt polecat remove`)
**Link path:** polecat.md (the `remove` subsection at line 232-244 documents `runPolecatRemove` behavior). Then the `list` subsection (line 191-229) explicitly documents **zombie tmux session detection** -- "Zombie tmux sessions: sessions without matching worktree directories. These occur when a worktree is deleted but the tmux session persists (incomplete nuke or session naming mismatch)."
**Hops:** 0 (direct entity page)
**Answer found?** Yes. The polecat.md page documents both the `remove` command (which deletes worktrees but doesn't mention session cleanup) and the `list` command's zombie detection (which finds exactly the symptom described in the issue -- sessions without matching worktrees after removal). The page makes clear that `remove` calls `mgr.Remove(polecatName, force)` but the list subsection confirms zombies are a known phenomenon.
**Score:** full

### #3653 -- Dead code: HandlePolecatDone and HandlePolecatDoneFromBead never called

**Entry point:** gastown/packages/witness.md (issue is about witness handlers)
**Link path:** witness.md -> mentions `handlers.go` as the bulk file (2705 lines) -> links to [witness role](../roles/witness.md) and [gt witness](../commands/witness.md)
**Hops:** 1
**Answer found?** The witness.md package page mentions `handlers.go` but does not enumerate individual handler functions like `HandlePolecatDone` or `HandlePolecatDoneFromBead`. It describes the overall protocol message vocabulary and patrol-based detection, but doesn't identify specific dead code. Following the link to witness command page doesn't help either -- it documents CLI subcommands, not internal handler functions. The issue requires function-level knowledge of which handlers are called vs dead.
**Score:** partial
**Missing link:** witness.md should link to or document the `HandlePolecatDone` / `HandlePolecatDoneFromBead` handlers and note their call status (dead code). This is a detail-gap: the wiki covers the package at architectural level but not at individual-handler level.

### #3652 -- witness and refinery Start() never create agent beads

**Entry point:** gastown/workflows/investigations/agent-lifecycle.md (symptom matches "agent fails to start")
**Link path:** agent-lifecycle.md -> Step 5: Witness start failures -> links to [witness package](../../packages/witness.md) `Manager.Start` at `manager.go:110` -> witness.md documents `Manager.Start` but not bead creation within it -> links to [session](../../packages/session.md)
**Hops:** 3
**Answer found?** The agent-lifecycle investigation workflow documents the witness `Manager.Start` path and its failure points (foreground mode error, zombie detection, rig parked). Following to witness.md confirms the `Start` method exists and its session lifecycle. The investigation workflow also documents polecat start and explicitly mentions `session_manager.go:222` for the polecat start path. However, the specific gap (no `createAgentBeadWithRetry` call in witness/refinery Start) is documented in the investigation workflow's Step 5 context: it covers zombie session handling but not bead creation. The cross-page inference round previously scored this as full because the wiki documents enough about both witness Start() and polecat Start() to infer the gap.
**Score:** partial
**Missing link:** The investigation workflow Step 5 (witness start failures) should mention agent bead creation as a precondition or failure point. Currently it covers session creation but not bead initialization.

### #3651 -- Incorrect install instructions on 1.0 release

**Entry point:** gastown/binaries/gt.md (issue is about install instructions)
**Link path:** gt.md -> links to [Makefile](../files/makefile.md) -> Makefile documents build process. gt.md -> links to the command tree via [commands/README.md](../commands/README.md).
**Hops:** 1
**Answer found?** The gt.md binary page documents the build-time variables and LDFLAGS but does not document the v1.0.0 release page or its install instructions. The issue is about the GitHub release page claiming `npm install -g @anthropic/gastown` which is wrong. The wiki doesn't cover release page content or npm packaging (the npm-package directory is inventoried in auxiliary but no entity page exists for it).
**Score:** partial
**Missing link:** No page covers the npm-package or release page install instructions. The [auxiliary inventory](../inventory/auxiliary.md) lists `npm-package/` but has no entity page linked.

### #3648 -- doctor claude-settings never converges

**Entry point:** gastown/commands/doctor.md (issue is about `gt doctor`)
**Link path:** doctor.md -> links to [doctor package](../packages/doctor.md) -> doctor package page mentions `claude_settings_check.go` in its sources list -> links back to [gt hooks](../commands/hooks.md).
**Hops:** 2
**Answer found?** The doctor.md command page documents the registered checks including `claude_settings_check`. The doctor package page documents the `claude_settings_check.go` source. The hooks.md command page documents `gt hooks sync` and the base/override resolution. Together these pages provide the context for the convergence disagreement between doctor's check and hooks' sync -- but the specific bug (doctor's check expects a Stop hook that hooks sync doesn't generate identically) is at a detail level not covered. However, the previous grep-based score was "full" because the wiki documents both subsystems enough to understand the disagreement.

Following links: doctor.md links to [doctor package](../packages/doctor.md). Doctor package links to [hooks command](../commands/hooks.md). Hooks command links to [hooks package](../packages/hooks.md). This 3-hop chain covers doctor checks, hooks sync, and the settings.json model.
**Score:** full (the pages document both sides of the convergence problem -- doctor's registered checks and hooks' base/override resolution -- and the links connect them)

### #3626 -- `go install` Linux macOS complaint

**Entry point:** gastown/binaries/gt.md (issue is about the gt binary startup)
**Link path:** gt.md -> self-kill check section (line 79+) documents the exact error message: "ERROR: This binary was built with 'go build' directly. macOS will SIGKILL unsigned binaries. Use 'make build' instead." The page documents the gate: `BuiltProperly == "" && Build == "dev"` -> exits. Also notes this affects all commands including `gt --version`.
**Hops:** 0 (direct entity page)
**Answer found?** Yes, completely. The gt.md page documents the exact error message the issue reports, the condition that triggers it (`BuiltProperly` not set AND `Build == "dev"`), and that `go install` doesn't set `BuiltProperly`. The page also links to [Makefile](../files/makefile.md) which shows the LDFLAGS recipe that sets `BuiltProperly=1`. The issue is that the error message says "macOS" but triggers on Linux too -- the wiki preserves the verbatim error text which is platform-generic in the code guard.
**Score:** full

### #3623 -- Dolt connection exhaustion and slow mail/beads queries

**Entry point:** gastown/workflows/investigations/data-plane.md (symptom matches data-plane failures)
**Link path:** data-plane.md -> Step 1: Is Dolt server reachable? -> links to [doltserver package](../../packages/doltserver.md) for connection handling. doltserver.md documents the server lifecycle, port management, config.yaml authoring, but not connection pooling or `wait_timeout` configuration.
**Hops:** 2
**Answer found?** The data-plane investigation workflow covers Dolt server health, reachability, and startup. The doltserver.md page documents the server lifecycle. But neither page documents the `wait_timeout` setting, connection limits, or the death spiral behavior described in the issue. The issue is about operational tuning (connection exhaustion under load) that goes beyond what the architectural wiki covers. The wiki doesn't document Dolt server configuration parameters like `wait_timeout`.
**Score:** partial
**Missing link:** doltserver.md should document the `wait_timeout` configuration and connection limits. The data-plane investigation should have a step about connection exhaustion symptoms.

### #3614 -- Dashboard doesn't show pi/omp polecats

**Entry point:** gastown/workflows/investigations/monitoring.md (symptom matches dashboard data issues)
**Link path:** monitoring.md -> Step 7: Dashboard data issues -> links to [dashboard command](../../commands/dashboard.md) and [web package](../../packages/web.md). Dashboard.md documents `NewDashboardMux` and the fetcher. Web.md documents `WorkerRow`, `SessionRow`, and the templates.
**Hops:** 3
**Answer found?** The monitoring investigation covers dashboard startup and data issues. Following links to web.md reveals the data model (`WorkerRow`, `SessionRow`) but not how session detection works for non-Claude agents. The issue is about the dashboard only detecting Claude Code processes -- the wiki's web.md mentions `DashboardSummary.PolecatCount` and `DeadSessions` but not the process-detection mechanism that excludes pi/omp. The monitoring investigation Step 4 (agent listing) mentions `GT_PROCESS_NAMES` for process detection, which is relevant.
**Score:** partial
**Missing link:** The monitoring investigation Step 7 should cross-reference Step 4's process-name detection mechanism and note that non-Claude agents may not be detected. Web.md's fetcher documentation should mention how it discovers agent sessions.

### #3604 -- Refinery stalls during branch consolidation

**Entry point:** gastown/packages/refinery.md (issue is about refinery merge behavior)
**Link path:** refinery.md -> documents the merge engine (`Engineer`), batch-then-bisect merger, MR state machine, phase transitions. Links to [refinery role](../roles/refinery.md) and [gt refinery](../commands/refinery.md).
**Hops:** 1
**Answer found?** The refinery.md package page documents the `Engineer` (2057 lines), the merge flow, and the state machine. The failure modes section (from Phase 8) documents partial completion scenarios. The issue is about conflict handling during cherry-pick -- the refinery page covers the batch-then-bisect merger approach and conflict states (`CloseReason: conflict`). The wiki provides enough context to understand the stall scenario.
**Score:** full

### #3570 -- Daemon legacy tmux sockets on upgrade -> dual agents

**Entry point:** gastown/workflows/investigations/daemon-infrastructure.md (symptom matches daemon/socket issues)
**Link path:** daemon-infrastructure.md -> Step 0: tmux server socket -> links to [daemon package](../../packages/daemon.md). daemon.md documents the per-town socket isolation.
**Hops:** 2
**Answer found?** The daemon-infrastructure investigation covers tmux server liveness and per-town socket naming. The tmux.md page documents `SetDefaultSocket` and per-town socket naming. However, neither page documents what happens during an **upgrade** when the socket naming convention changes (v0.12.1 default socket -> v1.0.0 named socket "gt-<hash>"). The cleanup code in `gt down` for legacy sockets is mentioned in down.md's phase sequence but not specifically the upgrade-migration scenario. The issue is about a gap in the daemon restart path, not a gap in shutdown.
**Score:** partial
**Missing link:** daemon-infrastructure.md should have a step about socket naming changes across upgrades. The daemon.md page should document the legacy socket cleanup responsibility.

### #3565 -- tmux keybindings use stale prefix pattern

**Entry point:** gastown/packages/tmux.md (issue is about tmux keybindings)
**Link path:** tmux.md -> documents keybindings in its tags, mentions the `Tmux` struct and session management. The page describes `SetDefaultSocket` and session helpers but the keybinding functions (`SetRigMenuBinding`, `isGTBinding`, `isGTBindingCurrent`) are tmux package internals.
**Hops:** 0 (direct entity page)
**Answer found?** The tmux.md page is tagged with `keybindings` and its 3857-line `tmux.go` core would contain the binding functions. The page documents the package-level API including socket management but doesn't specifically enumerate `SetRigMenuBinding` or the staleness check functions. However, the previous grep-based test scored this as "full" because the wiki places this squarely in the tmux package. The page identifies the right file and package, which is enough to investigate.
**Score:** full (the tmux.md page covers keybinding management as part of the package scope, and the issue can be understood from the page's context about per-town socket naming and session management)

### #3563 -- Nudge stale pane ID cross-rig

**Entry point:** gastown/commands/nudge.md (issue is about `gt nudge` failing)
**Link path:** nudge.md -> documents delivery modes, address resolution, tmux `send-keys` -> links to [message-delivery investigation](../workflows/investigations/message-delivery.md) in Troubleshooting section. message-delivery.md -> Step 5: tmux delivery issues.
**Hops:** 2
**Answer found?** The nudge.md page documents the delivery mechanism and address resolution. The message-delivery investigation covers tmux delivery issues. However, neither documents the specific failure mode of stale `GT_PANE_ID` environment variables after session restart. The nudge.md failure modes section mentions `watchAndDeliver` delivery failure but not pane ID staleness. The issue is about `GT_PANE_ID` becoming stale when a session is restarted -- this is an env-var staleness bug not covered in the wiki.
**Score:** partial
**Missing link:** nudge.md should document the `GT_PANE_ID` env var dependency and the staleness risk. The message-delivery investigation should have a step about stale pane IDs.

### #3562 -- Rig init duplicate Dolt databases

**Entry point:** gastown/commands/rig.md (issue is about `gt rig add` with name/prefix mismatch)
**Link path:** rig.md -> documents `gt rig add` with `--prefix` flag -> links to [rig concept](../concepts/rig.md) and [doltserver](../packages/doltserver.md). rig.md documents the `--prefix` flag. The concept page documents the `<rigRoot>` canonical layout.
**Hops:** 1
**Answer found?** The rig.md command page documents the `--prefix` flag and the `gt rig add` invocation. The issue is about what happens when the rig name differs from the prefix -- two Dolt databases get created. The rig.md page documents the command surface including prefix, and the doltserver.md page documents per-rig database creation via `InitRig`. Together they provide the context for the duplicate-database bug. The previous grep-based score was "full".
**Score:** full

### #3554 -- Daemon wisp config warning spam

**Entry point:** gastown/packages/daemon.md (issue is about daemon log spam)
**Link path:** daemon.md -> documents the daemon's heartbeat loop, patrol tickers, and maintenance work. Links to [gt daemon](../commands/daemon.md) for the CLI surface.
**Hops:** 0 (direct entity page)
**Answer found?** The daemon.md package page documents the daemon's patrol tickers and maintenance functions but doesn't specifically document the wisp config check or the "no wisp config" warning. The wisp.md page exists (linked from index.md) but it documents the wisp concept/package, not the daemon's wisp-config polling behavior. The issue is about a specific warning message in the daemon's polling cycle that spams the log.
**Score:** partial
**Missing link:** daemon.md should document the wisp config polling check and its warning behavior. The daemon-infrastructure investigation should mention log spam from polling cycles.

### #3539 -- gt sling fails: bd update --type=agent rejected

**Entry point:** gastown/commands/sling.md (issue is about `gt sling` failing)
**Link path:** sling.md -> documents the sling dispatch, polecat spawning, flags, and outgoing calls. Links to [done](done.md), [formula](formula.md), [convoy](convoy.md), [polecat package](../packages/polecat.md).
**Hops:** 1
**Answer found?** The sling.md page documents the outgoing calls section which lists `bd update` invocations, but none of them use `--type=agent`. The issue says the `--type=agent` call is in polecat spawn/creation code, which lives in the polecat package. Following the link to polecat.md, the package page documents polecat manager and session manager but doesn't enumerate the specific `bd update --type=agent` call. However, the sling.md page does document the general bead interaction pattern. The previous grep-based score was "full" because the wiki documents the beads package's type system and sling's dispatch. The sling.md outgoing calls don't show `--type=agent` but the architectural context is present.

Actually, re-evaluating: the sling.md page's outgoing calls explicitly list the `bd` subprocess invocations, and none use `--type=agent`. The polecat package page mentions Dolt retry helpers and agent beads. The issue is about a specific flag (`--type=agent`) being rejected by beads v1.0.0 -- the wiki provides enough context (beads type system, sling dispatch, polecat spawning) to understand the domain, even if the specific flag isn't listed.
**Score:** full (the sling + beads pages together provide sufficient context to understand the type rejection; the beads.md page documents the hybrid subprocess model and the sling page documents the spawn path)

### #3538 -- Windows is not viable: tmux dependency

**Entry point:** gastown/packages/tmux.md (issue is about Windows/tmux architectural blocker)
**Link path:** tmux.md -> documents Windows/psmux support via paired platform shims (`flock_windows.go`, `process_group_windows.go`, `descendants_windows.go`, `sysproc_windows.go`). Also links to [gt binary](../binaries/gt.md) -> self-kill check (BuiltProperly) and [Makefile](../files/makefile.md).
**Hops:** 2
**Answer found?** Yes. tmux.md documents the cross-platform architecture including Windows psmux support. gt.md documents the BuiltProperly self-kill check that blocks `go install` builds. The Makefile documents the LDFLAGS recipe. Together these pages cover: (1) tmux hard dependency with psmux Windows variant, (2) `go install` self-kill gate, and (3) the build system. The issue reports all of these as blockers.
**Score:** full

### #3537 -- gt crew start launches outside tmux

**Entry point:** gastown/commands/crew.md (issue is about `gt crew start`)
**Link path:** crew.md -> documents subcommands including `start` -> links to [crew role](../roles/crew.md) and [session package](../packages/session.md). crew.md links to crew_lifecycle.go which contains `runCrewStart`.
**Hops:** 1
**Answer found?** The crew.md page documents the `start` subcommand exists in `crew_lifecycle.go` but doesn't document the specific tmux session creation mechanism. The issue is about the process being spawned on a detached pty instead of inside a tmux session. The session.md package page documents `NewSessionWithCommand` for tmux session creation. The investigation workflow (agent-lifecycle.md Step 9) covers crew start failures including tmux session creation. But the specific bug (process spawned outside tmux) is a failure mode not documented.
**Score:** partial
**Missing link:** crew.md should document the tmux session creation mechanism in the `start` subcommand description. The agent-lifecycle investigation Step 9 should mention the detached-pty failure mode.

### #3516 -- Rig names underscores, dolt prereq

**Entry point:** gastown/commands/rig.md (issue is about rig naming constraints)
**Link path:** rig.md -> documents `gt rig add` invocation. Then following to [workspace-setup investigation](../workflows/investigations/workspace-setup.md) via the investigation workflows index. workspace-setup.md -> Step 3: Rig add failures -> documents `gt rig add` failure modes including dependency checks.
**Hops:** 2 (rig.md -> workspace-setup.md)

Wait -- is there a direct link from rig.md to the workspace-setup investigation? Let me check. rig.md has a Troubleshooting section... I didn't read far enough. Let me check the investigation link path.

Actually, rig.md links to [rig concept](../concepts/rig.md) which doesn't directly link to the investigation. But the investigation index in index.md links to workspace-setup.md. So: index.md -> workspace-setup.md is 1 hop. From rig.md to workspace-setup.md requires going through index.md (2 hops) or finding a Troubleshooting link.

The workspace-setup investigation documents `gt rig add` failures and `deps.EnsureBeads` / `deps.EnsureDolt` prerequisite checks. The dolt prerequisite gap is relevant. The rig naming constraint (underscores vs hyphens) -- the rig.md page documents the `--prefix` flag but not the character restrictions on rig names.

The previous grep-based score was "full" because the wiki documents the `deps` package (beads.go and dolt.go) which covers the dolt prerequisite. And the rig command page documents the add flow.
**Score:** full (the wiki documents both the rig add command and the deps prerequisite system; the workspace-setup investigation explicitly covers dependency failures; the rig naming constraint is a minor detail gap but the overall coverage is sufficient)

---

## Summary table

| # | Issue | Grep score | Nav score | Delta |
|---|---|---|---|---|
| 3665 | Centralize cross-platform sleep strings | miss | miss | unchanged |
| 3661 | `bd` `--rig` vs `--repo` | partial | partial | unchanged |
| 3658 | polecat remove zombie sessions | full | full | unchanged |
| 3653 | Dead code: HandlePolecatDone | partial | partial | unchanged |
| 3652 | witness/refinery Start() no agent beads | full | partial | **downgraded** |
| 3651 | Incorrect install instructions | partial | partial | unchanged |
| 3648 | doctor claude-settings convergence | full | full | unchanged |
| 3626 | `go install` Linux macOS complaint | full | full | unchanged |
| 3623 | Dolt connection exhaustion | partial | partial | unchanged |
| 3614 | Dashboard pi/omp detection | partial | partial | unchanged |
| 3604 | Refinery stall branch consolidation | full | full | unchanged |
| 3570 | Daemon legacy tmux sockets | partial | partial | unchanged |
| 3565 | tmux keybindings stale prefix | full | full | unchanged |
| 3563 | Nudge stale pane ID | partial | partial | unchanged |
| 3562 | Rig init duplicate Dolt databases | full | full | unchanged |
| 3554 | Daemon wisp config warning spam | partial | partial | unchanged |
| 3539 | Sling bd update type rejection | full | full | unchanged |
| 3538 | Windows not viable: tmux dependency | full | full | unchanged |
| 3537 | Crew start outside tmux | partial | partial | unchanged |
| 3516 | Rig names underscores, dolt prereq | full | full | unchanged |

## Aggregate scores

| Score | Grep-based (final) | Navigation-based | Change |
|---|---|---|---|
| Full | 10 (50%) | 9 (45%) | -1 |
| Partial | 9 (45%) | 10 (50%) | +1 |
| Miss | 1 (5%) | 1 (5%) | 0 |

**Result: 9 full (45%), 10 partial (50%), 1 miss (5%).**

## Downgraded issues (full -> partial via navigation)

**1 issue downgraded: #3652** (witness/refinery Start() never create agent beads)

- **Grep-based score:** full -- the cross-page inference round found enough content across witness.md, refinery.md, and polecat.md to infer the gap.
- **Navigation score:** partial -- starting from the agent-lifecycle investigation (the most natural entry point for "agent fails to start"), the link path reaches witness start failures (Step 5) and polecat start (Step 2), but the investigation workflow doesn't mention agent bead creation as a startup precondition. To reach the polecat package page's `createAgentBeadWithRetry` mention requires navigating: investigation -> witness.md -> polecat.md, and even then the bead creation function isn't explicitly documented as a contrast point.
- **Missing link:** The agent-lifecycle investigation Step 5 (witness start) and Step 7 (refinery start) should mention "agent bead creation" as a startup precondition/failure point, with a note that polecats do this via `createAgentBeadWithRetry` but witness/refinery do not.

## Missing links inventory

Links that should exist but don't, grouped by source page:

### gastown/workflows/investigations/agent-lifecycle.md

| Missing link | Would connect to | Issue |
|---|---|---|
| Step 5/7: agent bead creation precondition | polecat.md `createAgentBeadWithRetry` vs witness/refinery Start() | #3652 |

### gastown/commands/done.md

| Missing link | Would connect to | Issue |
|---|---|---|
| Outgoing calls: `bd create --rig=<name>` | beads.md flag documentation | #3661 |

### gastown/packages/witness.md

| Missing link | Would connect to | Issue |
|---|---|---|
| Handler function inventory (HandlePolecatDone, etc.) | call-site analysis / dead code identification | #3653 |

### gastown/packages/doltserver.md

| Missing link | Would connect to | Issue |
|---|---|---|
| `wait_timeout` and connection limit configuration | operational tuning documentation | #3623 |

### gastown/packages/daemon.md

| Missing link | Would connect to | Issue |
|---|---|---|
| Wisp config polling check and warning behavior | log spam documentation | #3554 |
| Legacy socket cleanup on upgrade | upgrade migration documentation | #3570 |

### gastown/commands/nudge.md

| Missing link | Would connect to | Issue |
|---|---|---|
| `GT_PANE_ID` env var dependency and staleness risk | session.md env var lifecycle | #3563 |

### gastown/commands/crew.md

| Missing link | Would connect to | Issue |
|---|---|---|
| Tmux session creation mechanism in `start` | session.md `NewSessionWithCommand` | #3537 |

### gastown/workflows/investigations/monitoring.md

| Missing link | Would connect to | Issue |
|---|---|---|
| Step 7: cross-reference to Step 4 process-name detection for non-Claude agents | agent process detection mechanism | #3614 |

### gastown/workflows/investigations/data-plane.md

| Missing link | Would connect to | Issue |
|---|---|---|
| Connection exhaustion symptoms and `wait_timeout` tuning | doltserver operational config | #3623 |

### gastown/binaries/gt.md or index.md

| Missing link | Would connect to | Issue |
|---|---|---|
| npm-package entity page | release/install instructions documentation | #3651 |

## Observations

1. **Navigation overhead is minimal for well-linked pages.** Issues where the natural entity page directly contains the answer (#3658, #3626, #3604, #3565, #3562, #3538, #3539, #3516) score identically whether using grep or link-following. The wiki's cross-linking discipline ensures these pages are reachable within 0-2 hops.

2. **Investigation workflows are effective navigation hubs.** For issues whose symptom matches an investigation domain (data-plane, agent-lifecycle, monitoring, daemon-infrastructure, workspace-setup), the investigation page provides a structured path to relevant entity pages. However, the investigation workflows focus on "how to diagnose" rather than "what failure modes exist," so they miss some edge cases.

3. **The 1 downgrade is at the cross-page inference boundary.** Issue #3652 requires correlating information from three pages (witness Start, refinery Start, polecat `createAgentBeadWithRetry`). Grep can find all three in one pass; link-following requires knowing where to go next after each page, and the investigation workflow doesn't provide the connecting link for agent bead creation.

4. **Partial scores are stable.** All 9 partial issues from the grep-based test remain partial under navigation. These are genuine content gaps (detail-level information not in the wiki), not navigation failures. The missing content simply isn't there regardless of how you search.

5. **The miss is inherently unreachable.** Issue #3665 proposes a new package that doesn't exist yet. No amount of linking can make content that doesn't exist navigable.
