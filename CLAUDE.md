# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Contribution workspace for [AdenHQ/Hive](https://github.com/adenhq/hive) — a YC-backed Python framework for autonomous, adaptive AI agents (9,600+ stars, Apache 2.0). Part of ORGAN-IV (Taxis/Orchestration), tier: contrib.

The actual Hive fork lives (or will live) in `./repo/` once cloned from `4444J99/hive`. The root directory contains coordination docs only.

## Setup

```bash
# Fork and clone (one-time)
gh repo fork adenhq/hive --clone=false --org 4444J99
git clone https://github.com/4444J99/hive.git repo
cd repo

# Hive quickstart
./quickstart.sh
```

## Commands (inside `repo/`)

```bash
make check              # ruff check + ruff format --check on core/ and tools/
make test               # cd core && pytest tests/ -v
```

Both must pass before submitting any PR.

## Upstream Repo Structure

```
adenhq/hive/
├── core/               # Framework core (Python)
│   ├── framework/      # Agent framework
│   ├── frontend/       # UI
│   ├── examples/       # Example agents
│   └── tests/          # Core tests (pytest)
├── tools/              # Tool servers and integrations (102 MCP tools)
│   ├── src/            # Tool implementations
│   └── tests/          # Tool tests
├── docs/               # Documentation + i18n
├── CONTRIBUTING.md     # Contribution rules (read before any PR)
├── CLAUDE.md           # Upstream Claude Code instructions (defer to this inside repo/)
└── Makefile            # make check, make test
```

## Contribution Rules (from upstream CONTRIBUTING.md)

- **Claim before PR:** Comment "I'd like to work on this!" on the issue, wait for assignment (24h window)
- **Exceptions:** Doc fixes and micro-fixes (<20 lines, no logic changes) skip assignment
- **PRs for unassigned issues may be delayed or closed**
- Branch naming: `feature/your-feature-name`
- Run `make check` and `make test` before submitting

## Target Issues

| Issue | Topic | Fit |
|-------|-------|-----|
| #2805 | Expand integrations/tools | MCP tool server — direct ORGANVM overlap |
| #6613 | Reproducibility for self-evolving agents | Promotion state machine pattern |
| #6612 | Parallel test execution (pytest-xdist) | Deep pytest infrastructure experience |
| #6646 | HTTP API authentication | Security implementation |
| Docs | MCP integration guides, examples | No assignment needed |

## ORGANVM Context

This workspace connects the ORGANVM multi-agent orchestration system to an external open-source project. Patterns to cross-pollinate:

- **Promotion state machine** (LOCAL → GRADUATED) → agent versioning/snapshots
- **MCP server infrastructure** → new Hive tool integrations
- **Dependency validation** (50 edges, 0 violations) → reproducibility guarantees
- **pytest infrastructure** (23K+ tests system-wide) → parallel test execution

## Key People

- Vincent Jiang (CEO) — initiated outreach
- Timothy Zhang (Co-founder)
- Adel Burieva (Co-founder)

## Working in This Directory

- `CONTRIBUTION-PROMPT.md` — full session prompt for contribution sessions
- `README.md` — relationship map and pipeline cross-references
- Once `repo/` exists, defer to its own CLAUDE.md/AGENTS.md for code conventions

<!-- ORGANVM:AUTO:START -->
## System Context (auto-generated — do not edit)

**Organ:** ORGAN-IV (Orchestration) | **Tier:** standard | **Status:** LOCAL
**Org:** `organvm-iv-taxis` | **Repo:** `contrib--adenhq-hive`

### Edges
- *No inter-repo edges declared in seed.yaml*

### Siblings in Orchestration
`orchestration-start-here`, `petasum-super-petasum`, `universal-node-network`, `.github`, `agentic-titan`, `agent--claude-smith`, `a-i--skills`, `tool-interaction-design`, `system-governance-framework`, `reverse-engine-recursive-run`, `collective-persona-operations`, `contrib--ipqwery-ipapi-py`, `contrib--primeinc-github-stars`, `contrib--temporal-sdk-python`, `contrib--dbt-mcp` ... and 2 more

### Governance
- *Standard ORGANVM governance applies*

*Last synced: 2026-03-25T22:27:15Z*

## Session Review Protocol

At the end of each session that produces or modifies files:
1. Run `organvm session review --latest` to get a session summary
2. Check for unimplemented plans: `organvm session plans --project .`
3. Export significant sessions: `organvm session export <id> --slug <slug>`
4. Run `organvm prompts distill --dry-run` to detect uncovered operational patterns

Transcripts are on-demand (never committed):
- `organvm session transcript <id>` — conversation summary
- `organvm session transcript <id> --unabridged` — full audit trail
- `organvm session prompts <id>` — human prompts only


## Active Directives

| Scope | Phase | Name | Description |
|-------|-------|------|-------------|
| system | any | prompting-standards | Prompting Standards |
| system | any | research-standards-bibliography | APPENDIX: Research Standards Bibliography |
| system | any | phase-closing-and-forward-plan | METADOC: Phase-Closing Commemoration & Forward Attack Plan |
| system | any | research-standards | METADOC: Architectural Typology & Research Standards |
| system | any | sop-ecosystem | METADOC: SOP Ecosystem — Taxonomy, Inventory & Coverage |
| system | foundation | agent-seeding-and-workforce-planning | agent-seeding-and-workforce-planning |
| system | foundation | architecture-decision-records | architecture-decision-records |
| system | any | autonomous-content-syndication | SOP: Autonomous Content Syndication (The Broadcast Protocol) |
| system | any | autopoietic-systems-diagnostics | SOP: Autopoietic Systems Diagnostics (The Mirror of Eternity) |
| system | any | background-task-resilience | background-task-resilience |
| system | any | cicd-resilience-and-recovery | SOP: CI/CD Pipeline Resilience & Recovery |
| system | any | community-event-facilitation | SOP: Community Event Facilitation (The Dialectic Crucible) |
| system | any | context-window-conservation | context-window-conservation |
| system | any | conversation-to-content-pipeline | SOP — Conversation-to-Content Pipeline |
| system | any | cross-agent-handoff | SOP: Cross-Agent Session Handoff |
| system | any | cross-channel-publishing-metrics | SOP: Cross-Channel Publishing Metrics (The Echo Protocol) |
| system | any | data-migration-and-backup | SOP: Data Migration and Backup Protocol (The Memory Vault) |
| system | any | document-audit-feature-extraction | SOP: Document Audit & Feature Extraction |
| system | any | dynamic-lens-assembly | SOP: Dynamic Lens Assembly |
| system | any | essay-publishing-and-distribution | SOP: Essay Publishing & Distribution |
| system | any | formal-methods-applied-protocols | SOP: Formal Methods Applied Protocols |
| system | any | formal-methods-master-taxonomy | SOP: Formal Methods Master Taxonomy (The Blueprint of Proof) |
| system | any | formal-methods-tla-pluscal | SOP: Formal Methods — TLA+ and PlusCal Verification (The Blueprint Verifier) |
| system | any | generative-art-deployment | SOP: Generative Art Deployment (The Gallery Protocol) |
| system | foundation | legal-compliance-matrix | legal-compliance-matrix |
| system | any | market-gap-analysis | SOP: Full-Breath Market-Gap Analysis & Defensive Parrying |
| system | any | mcp-server-fleet-management | SOP: MCP Server Fleet Management (The Server Protocol) |
| system | any | multi-agent-swarm-orchestration | SOP: Multi-Agent Swarm Orchestration (The Polymorphic Swarm) |
| system | any | network-testament-protocol | SOP: Network Testament Protocol (The Mirror Protocol) |
| system | foundation | ontological-renaming | ontological-renaming |
| system | any | open-source-licensing-and-ip | SOP: Open Source Licensing and IP (The Commons Protocol) |
| system | any | performance-interface-design | SOP: Performance Interface Design (The Stage Protocol) |
| system | any | pitch-deck-rollout | SOP: Pitch Deck Generation & Rollout |
| system | any | polymorphic-agent-testing | SOP: Polymorphic Agent Testing (The Adversarial Protocol) |
| system | any | promotion-and-state-transitions | SOP: Promotion & State Transitions |
| system | foundation | readme-and-documentation | readme-and-documentation |
| system | any | recursive-study-feedback | SOP: Recursive Study & Feedback Loop (The Ouroboros) |
| system | any | repo-onboarding-and-habitat-creation | SOP: Repo Onboarding & Habitat Creation |
| system | any | research-to-implementation-pipeline | SOP: Research-to-Implementation Pipeline (The Gold Path) |
| system | any | security-and-accessibility-audit | SOP: Security & Accessibility Audit |
| system | any | session-self-critique | session-self-critique |
| system | any | smart-contract-audit-and-legal-wrap | SOP: Smart Contract Audit and Legal Wrap (The Ledger Protocol) |
| system | any | source-evaluation-and-bibliography | SOP: Source Evaluation & Annotated Bibliography (The Refinery) |
| system | any | stranger-test-protocol | SOP: Stranger Test Protocol |
| system | any | strategic-foresight-and-futures | SOP: Strategic Foresight & Futures (The Telescope) |
| system | any | styx-pipeline-traversal | SOP: Styx Pipeline Traversal (The 7-Organ Transmutation) |
| system | any | system-dashboard-telemetry | SOP: System Dashboard Telemetry (The Panopticon Protocol) |
| system | any | the-descent-protocol | the-descent-protocol |
| system | any | the-membrane-protocol | the-membrane-protocol |
| system | any | theoretical-concept-versioning | SOP: Theoretical Concept Versioning (The Epistemic Protocol) |
| system | any | theory-to-concrete-gate | theory-to-concrete-gate |
| system | any | typological-hermeneutic-analysis | SOP: Typological & Hermeneutic Analysis (The Archaeology) |

Linked skills: api-design-patterns, cicd-resilience-and-recovery, coding-standards-enforcer, continuous-learning-agent, contract-risk-analyzer, cross-agent-handoff, evaluation-to-growth, gdpr-compliance-check, genesis-dna, multi-agent-workforce-planner, planning-and-roadmapping, promotion-and-state-transitions, quality-gate-baseline-calibration, repo-onboarding-and-habitat-creation, security-threat-modeler, structural-integrity-audit


**Prompting (Anthropic)**: context 200K tokens, format: XML tags, thinking: extended thinking (budget_tokens)


## Live System Variables (Ontologia)

| Variable | Value | Scope | Updated |
|----------|-------|-------|---------|
| `active_repos` | 64 | global | 2026-03-25 |
| `archived_repos` | 54 | global | 2026-03-25 |
| `ci_workflows` | 106 | global | 2026-03-25 |
| `code_files` | 0 | global | 2026-03-25 |
| `dependency_edges` | 60 | global | 2026-03-25 |
| `operational_organs` | 8 | global | 2026-03-25 |
| `published_essays` | 29 | global | 2026-03-25 |
| `repos_with_tests` | 0 | global | 2026-03-25 |
| `sprints_completed` | 33 | global | 2026-03-25 |
| `test_files` | 0 | global | 2026-03-25 |
| `total_organs` | 8 | global | 2026-03-25 |
| `total_repos` | 127 | global | 2026-03-25 |
| `total_words_formatted` | 0 | global | 2026-03-25 |
| `total_words_numeric` | 0 | global | 2026-03-25 |
| `total_words_short` | 0K+ | global | 2026-03-25 |

Metrics: 9 registered | Observations: 15536 recorded
Resolve: `organvm ontologia status` | Refresh: `organvm refresh`


## System Density (auto-generated)

AMMOI: 56% | Edges: 41 | Tensions: 33 | Clusters: 5 | Adv: 7 | Events(24h): 23754
Structure: 8 organs / 127 repos / 1654 components (depth 17) | Inference: 98% | Organs: META-ORGANVM:64%, ORGAN-I:55%, ORGAN-II:47%, ORGAN-III:55% +4 more
Last pulse: 2026-03-25T22:27:04 | Δ24h: +3.5% | Δ7d: n/a


## Dialect Identity (Trivium)

**Dialect:** GOVERNANCE_LOGIC | **Classical Parallel:** Rhetoric | **Translation Role:** The Meta-Logic — governance rules ARE propositions

Strongest translations: I (formal), V (structural), META (structural)

Scan: `organvm trivium scan IV <OTHER>` | Matrix: `organvm trivium matrix` | Synthesize: `organvm trivium synthesize`

<!-- ORGANVM:AUTO:END -->
