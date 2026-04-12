---
title: gastown docs/ tree inventory
type: note
status: verified
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:
  - /home/kimberly/repos/gastown/docs/
tags: [inventory, enumeration, docs]
---

# gastown `docs/` tree inventory

Every file under `/home/kimberly/repos/gastown/docs/` with line count
and path, as of 2026-04-11. Neutral enumeration — no summaries of
content, no claim about audience.

**Capture method:** `find /home/kimberly/repos/gastown/docs -type f` +
`wc -l` per file.

**Total:** 60 files.

## docs/ (root level — 13 files)

| lines | path                                          |
|------:|-----------------------------------------------|
|   835 | `docs/agent-provider-integration.md`          |
|   170 | `docs/CLEANUP.md`                             |
|   251 | `docs/HOOKS.md`                               |
|   321 | `docs/INSTALLING.md`                          |
|   469 | `docs/WASTELAND.md`                           |
|    94 | `docs/glossary.md`                            |
|   299 | `docs/otel-data-model.md`                     |
|   204 | `docs/overview.md`                            |
|    39 | `docs/phase4-minimum-fix-acceptance.md`       |
|   621 | `docs/proxy-server.md`                        |
|   767 | `docs/reference.md`                           |
|   264 | `docs/why-these-features.md`                  |

## docs/concepts/ (6 files)

| lines | path                                          |
|------:|-----------------------------------------------|
|   251 | `docs/concepts/convoy.md`                     |
|   286 | `docs/concepts/identity.md`                   |
|   590 | `docs/concepts/integration-branches.md`       |
|   132 | `docs/concepts/molecules.md`                  |
|   381 | `docs/concepts/polecat-lifecycle.md`          |
|   102 | `docs/concepts/propulsion-principle.md`       |

## docs/design/ (root level — 23 files)

| lines | path                                                 |
|------:|------------------------------------------------------|
|   853 | `docs/design/agent-api-inventory.md`                 |
|   381 | `docs/design/architecture.md`                        |
|   307 | `docs/design/directives-and-overlays.md`             |
|    83 | `docs/design/dog-execution-model.md`                 |
|   509 | `docs/design/dog-infrastructure.md`                  |
|   545 | `docs/design/dolt-storage.md`                        |
|   214 | `docs/design/escalation.md`                          |
|   299 | `docs/design/factory-worker-api.md`                  |
|   209 | `docs/design/federation.md`                          |
|   249 | `docs/design/formula-resolution.md`                  |
|   640 | `docs/design/ledger-export-triggers.md`              |
|   565 | `docs/design/mail-protocol.md`                       |
|   635 | `docs/design/model-aware-molecules.md`               |
|   475 | `docs/design/mol-mall-design.md`                     |
|   203 | `docs/design/persistent-polecat-pool.md`             |
|   276 | `docs/design/plugin-system.md`                       |
|   664 | `docs/design/polecat-lifecycle-patrol.md`            |
|   462 | `docs/design/polecat-self-managed-completion.md`     |
|   414 | `docs/design/property-layers.md`                     |
|   662 | `docs/design/sandboxed-polecat-execution.md`         |
|   459 | `docs/design/scheduler.md`                           |
|    74 | `docs/design/tmux-keybindings.md`                    |
|   985 | `docs/design/witness-at-team-lead.md`                |

## docs/design/convoy/ (4 files)

| lines | path                                          |
|------:|-----------------------------------------------|
|   382 | `docs/design/convoy/convoy-lifecycle.md`      |
|   544 | `docs/design/convoy/mountain-eater.md`        |
|   375 | `docs/design/convoy/roadmap.md`               |
|   640 | `docs/design/convoy/spec.md`                  |

## docs/design/convoy/stage-launch/ (3 files)

| lines | path                                                          |
|------:|---------------------------------------------------------------|
|     1 | `docs/design/convoy/stage-launch/bv-insights.json`            |
|   237 | `docs/design/convoy/stage-launch/prd.md`                      |
|   532 | `docs/design/convoy/stage-launch/testing.md`                  |

## docs/design/otel/ (2 files)

| lines | path                                          |
|------:|-----------------------------------------------|
|   694 | `docs/design/otel/otel-architecture.md`       |
|   471 | `docs/design/otel/otel-data-model.md`         |

## docs/examples/ (4 files)

| lines | path                                                |
|------:|-----------------------------------------------------|
|   372 | `docs/examples/agent-validation.formula.toml`       |
|   169 | `docs/examples/hanoi-demo.md`                       |
|    77 | `docs/examples/rig-settings.example.json`           |
|   102 | `docs/examples/town-settings.example.json`          |

## docs/gas-city/ (1 file)

| lines | path                                          |
|------:|-----------------------------------------------|
|   402 | `docs/gas-city/crew-specialization-design.md` |

## docs/guides/ (2 files)

| lines | path                                          |
|------:|-----------------------------------------------|
|    39 | `docs/guides/local-rig-bootstrap.md`          |
|  1217 | `docs/guides/mvgt-integration.md`             |

## docs/research/ (2 files)

| lines | path                                                    |
|------:|---------------------------------------------------------|
|   315 | `docs/research/macos-sandbox-exec.md`                   |
|   647 | `docs/research/w-gc-004-agent-framework-survey.md`      |

## docs/skills/ (1 file)

| lines | path                                      |
|------:|-------------------------------------------|
|   390 | `docs/skills/convoy/SKILL.md`             |

## Totals

- **60 files total** under `docs/`.
- **10 subdirectories** (`concepts`, `design`, `design/convoy`,
  `design/convoy/stage-launch`, `design/otel`, `examples`, `gas-city`,
  `guides`, `research`, `skills`).
- **Line count total:** ~23,600 lines of documentation (sum of all
  line counts above).
- **Largest single file:** `docs/guides/mvgt-integration.md` at 1,217
  lines.
- **Second largest:** `docs/design/witness-at-team-lead.md` at 985
  lines.
- **Smallest substantive file:** `docs/guides/local-rig-bootstrap.md`
  at 39 lines (tied with `docs/phase4-minimum-fix-acceptance.md`).
- **One-byte stub:** `docs/design/convoy/stage-launch/bv-insights.json`
  at 1 line (likely an empty or placeholder JSON file).
