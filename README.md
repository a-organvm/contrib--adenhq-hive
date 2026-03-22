# contrib--adenhq-hive

Open-source contribution workspace for [AdenHQ/Hive](https://github.com/adenhq/hive) — a YC-backed framework for autonomous, adaptive AI agents.

## Why This Exists

Inbound lead from Vincent Jiang (CEO, AdenHQ). Team starred our portfolio, emailed requesting open-source contributions. Direct domain overlap with ORGANVM multi-agent orchestration.

**Origin:** `~/Workspace/4444J99/application-pipeline/` → doubling-back outreach plan → AdenHQ (Tier 1, entry #1)

## Relationship Map

| Artifact | Location |
|----------|----------|
| **Pipeline origin** | `~/Workspace/4444J99/application-pipeline/applications/2026-03-18/outreach-plan-doubling-back.md` (entry #1) |
| **DM follow-ups** | `~/Workspace/4444J99/application-pipeline/applications/2026-03-19/dm-followups.md` |
| **Contribution prompt** | `./CONTRIBUTION-PROMPT.md` (this directory) |
| **Outreach log** | `~/Workspace/4444J99/application-pipeline/signals/outreach-log.yaml` (AdenHQ entries) |
| **Contact CRM** | `~/Workspace/4444J99/application-pipeline/signals/contacts.yaml` (Vincent Jiang, Timothy Zhang, Adel Burieva) |
| **Network graph** | `~/Workspace/4444J99/application-pipeline/signals/network.yaml` (AdenHQ node) |
| **Hive repo (upstream)** | `https://github.com/adenhq/hive` |
| **Hive repo (fork)** | TBD — fork when ready to PR |

## Hive Overview

| Metric | Value |
|--------|-------|
| Stars | 9,633 |
| Forks | 5,519 |
| Open Issues | 1,374 |
| Language | Python |
| License | Apache 2.0 |
| Discord | 6,300+ members |
| MCP Tools | 102 |

## ORGANVM Alignment

| Hive Concept | ORGANVM Equivalent |
|-------------|-------------------|
| Node-graph architecture | 8-organ dependency graph, 50 edges |
| Self-healing agents | Promotion state machine, forward-only transitions |
| Human-in-the-loop | AI-conductor methodology (human directs, AI generates) |
| MCP integration (102 tools) | MCP servers (filesystem, memory, sequential thinking) |
| Multi-agent coordination | Cross-organ orchestration via ORGAN-IV |
| Credential management | `.submit-config.yaml`, secrets isolation |
| Self-improving agents | Evaluative authority, IRA facility, recalibration |

## Target Issues

1. **#2805** — Expand integrations/tools (explicitly requested in CONTRIBUTING.md)
2. **#6613** — Reproducibility and versioning for self-evolving agents
3. **#6612** — Parallel test execution with pytest-xdist
4. **Documentation** — MCP integration docs, examples (no assignment needed)

## Organ Membership

- **Organ:** IV (Taxis — Orchestration)
- **Tier:** contrib (external open-source contribution)
- **Status:** LOCAL
- **Produces:** PRs to adenhq/hive, relationship capital with AdenHQ team
- **Consumes:** ORGANVM patterns (state machine, MCP, testing infrastructure)
