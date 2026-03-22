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
