# Hive Fusion: Cross-Organ Symbiotic Contribution

**Date:** 2026-03-21
**Status:** APPROVED (v2 — post-review fixes)
**Target:** AdenHQ/Hive #6613 — Improve reproducibility and versioning for self-evolving agents
**Approach:** Cross-Organ Symbiote (Approach B)
**Review:** Code review completed 2026-03-21. Critical/important findings addressed in v2.

---

## 1. What This Is

A public-process contribution to AdenHQ/Hive that exercises the ORGANVM eight-organ model as a living system. The contribution is not a PR — it is the entire cross-organ process that *produces* a PR, along with theory, visualizations, journal artifacts, and a public narrative. The Hive PR is one organ's output in a symbiotic organism.

The organism has a lifecycle:

```
JOURNAL → EXTRACT → GENERATE → BUILD → SHIP → NARRATE
   IV        I         II        III     III       V
```

This is not a pipeline. It is a living cycle — shipping the PR generates observations that feed back into the journal, which triggers new theory extraction, which refines the next iteration.

**Note on feedback loops vs dependency edges:** The feedback cycle above is a *process* feedback loop (human-in-the-loop iteration), not a code dependency back-edge. ORGANVM's Article II governs dependency direction (`I→II→III`); the process loop operates at a different layer. No back-edges exist in the code/data dependency graph.

---

## 2. Organ-by-Organ Artifact Map

### ORGAN-IV (Taxis) — The Conductor

Hub: `organvm-iv-taxis/contrib--adenhq-hive/`

| Artifact | Purpose |
|----------|---------|
| `journal/` | Timestamped session logs. Every working session gets a dated entry: what was attempted, what was learned, what fed back. This IS the public process — the audit trail. |
| `seed.yaml` | Formal contract declaring produces/consumes edges to I, II, III, V. |
| `CONTRIBUTION-PROMPT.md` | Session prompt for contribution sessions (exists, evolves). |
| `relationship/` | CRM artifacts — outreach log, contact signals, network graph entries. |
| `repo/` | The Hive fork (`4444J99/hive`). Working code. |

The journal serves the same role as decision logging in Hive's evolution loop and `AssemblyEvent` in agentic-titan — every transition is recorded, every decision traceable.

### ORGAN-I (Theoria) — Theory Extraction

New documents in an existing ORGAN-I repo. The formal fusion:

| Artifact | Content |
|----------|---------|
| `agent-design-lifecycle-algebra.md` | Formal state machine mapping: ORGANVM promotion states → Hive evolution stages. Isomorphism between `LOCAL→GRADUATED` and `DRAFT→PROMOTED`. Forward-only transitions, quality gates. |
| `assembly-dynamics-in-agent-evolution.md` | agentic-titan's Deleuze vocabulary (territorialization, deterritorialization, fission-fusion, lines of flight) as operational semantics for agent design evolution. Graph rewriting = deterritorialization. Version stabilization = reterritorialization. Over-fitting to past failures = CRYSTALLIZED assembly state. |
| `state-snapshot-integrity-protocol.md` | `StateSnapshot.verify()` checksum pattern generalized for versioned agent designs: integrity verification, diffing, migration. |

**ORGANVM gains:** Battle-tested theory validated against a 9,600-star production framework.
**Hive gains:** A conceptual vocabulary richer than "try, fail, fix."

### ORGAN-II (Poiesis) — Generative Artifacts

| Artifact | Content |
|----------|---------|
| State transition diagram | Agent design lifecycle as visual state machine — `DRAFT → CANDIDATE → VALIDATED → PROMOTED → ARCHIVED` with quality gates. SVG/Mermaid. |
| Before/after flowchart | Example Hive agent `flowchart.json` at v1 vs v3, showing how evolution changed graph structure. Visual proof of versioning value. |
| Fusion architecture diagram | The organ map itself — how ORGANVM's organs relate to Hive's components. |

### ORGAN-III (Ergon) — Shipped Code

The PR payload landing in `adenhq/hive`:

| Component | Description |
|-----------|-------------|
| `DesignVersion` schema | Pydantic model: snapshot of `GraphSpec` + `Goal` + metadata (timestamp, description, parent_version, quality_metrics, starred). Integrity checksum from agentic-titan's `StateSnapshot` pattern. |
| `DesignVersionStore` | File-based store at `~/.hive/agents/{name}/versions/`. Atomic writes via Hive's existing `atomic_write`. Index manifest. List/load/diff/restore/prune. |
| `DesignLifecycleState` | State machine: `DRAFT → CANDIDATE → VALIDATED → PROMOTED → ARCHIVED`. Forward-only transitions with quality gates. Modeled after ORGANVM promotion states, adapted to Hive vocabulary. |
| Auto-snapshot hooks | Wired into `queen_lifecycle_tools.py` at `confirm_and_build()` and evolution regeneration path. Every design change → version captured. |
| CLI commands | `hive versions list`, `hive versions show <v>`, `hive versions diff <v1> <v2>`, `hive versions restore <v>`, `hive versions star <v>`. Follows Hive's existing `framework/cli.py` patterns. |
| Tests | Unit + integration. pytest, `make test` passing. Following Hive's patterns in `core/tests/`. |

**What ships upstream:** PR to `adenhq/hive` addressing #6613.
**What stays in ORGANVM:** Theory (I), visualizations (II), journal (IV), narrative (V).

### ORGAN-V (Logos) — Public Narrative

| Artifact | Content |
|----------|---------|
| `how-governance-taught-agents-to-version.md` | Public-facing essay. How a 113-repo governance system applied its promotion state machine to an open-source AI agent framework's versioning problem. Technical + narrative. |
| Cross-post via ORGAN-VII | POSSE distribution through Kerygma profiles. |

---

## 3. The Symbiosis

### ORGANVM → Hive
- Promotion state machine (governance DNA)
- `StateSnapshot` integrity checksums
- Assembly state vocabulary for evolution dynamics
- Forward-only transitions with quality gates
- Tested, production-grade versioning code

### Hive → ORGANVM
- Real-world validation of governance patterns against 9,600-star framework
- Evolution data: how agents actually change across generations
- Fission-fusion dynamics tested against real agent coordination
- Relationship capital with YC-backed team (Vincent Jiang, Timothy Zhang, Adel Burieva)
- Public proof that ORGANVM produces tangible open-source value

---

## 4. Technical Design: The Hive PR (ORGAN-III Detail)

### 4.1 Schemas

Three Pydantic models, following the `Checkpoint`/`CheckpointSummary`/`CheckpointIndex` triad
already in `core/framework/schemas/checkpoint.py`:

```python
class DesignLifecycleState(StrEnum):
    DRAFT = "draft"           # Queen is building
    CANDIDATE = "candidate"   # Built, not yet validated
    VALIDATED = "validated"   # Passed success criteria on test inputs
    PROMOTED = "promoted"     # Marked as "good" by user (starred)
    ARCHIVED = "archived"     # Superseded by newer version

class DesignVersion(BaseModel):
    """Full design snapshot. Stored as individual JSON files."""

    version_id: str                    # v_{timestamp}_{uuid_8char}
    parent_version_id: str | None      # For lineage tracking
    lifecycle_state: DesignLifecycleState
    created_at: str                    # ISO 8601
    description: str                   # Human-readable (auto or manual)

    # The actual design snapshot
    # Obtained via GraphSpec.model_dump() and Goal.model_dump()
    # Contains nested NodeSpec/EdgeSpec objects fully serialized
    graph_spec: dict[str, Any]         # Full GraphSpec serialized
    goal: dict[str, Any]              # Full Goal serialized
    flowchart: dict[str, Any] | None  # flowchart.json if present

    # Quality metrics (populated as lifecycle advances)
    quality_metrics: dict[str, Any] = Field(default_factory=dict)
    starred: bool = False             # User bookmark

    # Integrity — SHA-256 of json.dumps(graph_spec, sort_keys=True) +
    # json.dumps(goal, sort_keys=True). Canonical sorted serialization
    # ensures deterministic hashes regardless of dict insertion order.
    checksum: str

    model_config = {"extra": "allow"}

    @classmethod
    def create(
        cls,
        graph_spec: dict[str, Any],
        goal: dict[str, Any],
        lifecycle_state: DesignLifecycleState = DesignLifecycleState.DRAFT,
        description: str = "",
        parent_version_id: str | None = None,
        flowchart: dict[str, Any] | None = None,
    ) -> "DesignVersion":
        """Create with auto-generated ID, timestamp, and checksum.
        Follows Checkpoint.create() pattern."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        short_uuid = uuid.uuid4().hex[:8]
        checksum_input = json.dumps(graph_spec, sort_keys=True) + json.dumps(goal, sort_keys=True)
        return cls(
            version_id=f"v_{timestamp}_{short_uuid}",
            parent_version_id=parent_version_id,
            lifecycle_state=lifecycle_state,
            created_at=datetime.now().isoformat(),
            description=description,
            graph_spec=graph_spec,
            goal=goal,
            flowchart=flowchart,
            checksum=hashlib.sha256(checksum_input.encode()).hexdigest()[:16],
        )

    def verify(self) -> bool:
        """Verify integrity (from agentic-titan StateSnapshot pattern)."""
        checksum_input = json.dumps(self.graph_spec, sort_keys=True) + json.dumps(self.goal, sort_keys=True)
        return self.checksum == hashlib.sha256(checksum_input.encode()).hexdigest()[:16]


class DesignVersionSummary(BaseModel):
    """Lightweight metadata for index listings (avoids loading full graph_spec)."""

    version_id: str
    lifecycle_state: DesignLifecycleState
    created_at: str
    description: str
    parent_version_id: str | None = None
    starred: bool = False
    checksum: str = ""

    model_config = {"extra": "allow"}

    @classmethod
    def from_version(cls, version: DesignVersion) -> "DesignVersionSummary":
        return cls(
            version_id=version.version_id,
            lifecycle_state=version.lifecycle_state,
            created_at=version.created_at,
            description=version.description,
            parent_version_id=version.parent_version_id,
            starred=version.starred,
            checksum=version.checksum,
        )


class DesignVersionIndex(BaseModel):
    """Manifest of all versions for an agent. Fast lookup without loading full snapshots."""

    agent_id: str
    versions: list[DesignVersionSummary] = Field(default_factory=list)
    latest_version_id: str | None = None
    current_promoted_id: str | None = None  # The active PROMOTED version
    total_versions: int = 0

    model_config = {"extra": "allow"}

    def add_version(self, version: DesignVersion) -> None:
        summary = DesignVersionSummary.from_version(version)
        self.versions.append(summary)
        self.latest_version_id = version.version_id
        self.total_versions = len(self.versions)
        if version.lifecycle_state == DesignLifecycleState.PROMOTED:
            self.current_promoted_id = version.version_id
```

### 4.2 DesignVersionStore

Storage structure:
```
~/.hive/agents/{agent_name}/
├── agent.json              # Current (HEAD) version
├── agent.py
├── flowchart.json
├── versions/
│   ├── index.json          # Version manifest (DesignVersionIndex)
│   ├── v_20260321_143022_abc12345.json
│   ├── v_20260321_160045_def67890.json
│   └── ...
├── sessions/               # (existing)
└── ...
```

Operations:
- `save_version(design_version)` — Atomic write + index update
- `load_version(version_id)` — Load specific version
- `list_versions(lifecycle_state?, starred?)` — Filtered listing
- `diff_versions(v1_id, v2_id)` — Structural diff of graph specs
- `restore_version(version_id)` — Copy version's graph_spec/goal back to agent.json
- `promote_version(version_id, target_state)` — Forward-only state transition with gate validation
- `prune_versions(max_age_days, keep_starred)` — Cleanup with starred retention

### 4.3 Lifecycle State Machine

Transitions (forward-only):
```
DRAFT → CANDIDATE        # Queen confirms build
CANDIDATE → VALIDATED    # Agent passes success criteria (automated gate)
VALIDATED → PROMOTED     # User stars / approves (manual gate)
PROMOTED → ARCHIVED      # Superseded by newer PROMOTED version
CANDIDATE → ARCHIVED     # Abandoned without validation
```

Quality gates:
- `DRAFT → CANDIDATE`: Graph validates (no structural errors)
- `CANDIDATE → VALIDATED`: At least 1 session completed with success=True
- `VALIDATED → PROMOTED`: User explicit action (star)

Back-transitions are forbidden. If a PROMOTED version needs to be superseded, the new version goes through the full lifecycle. This mirrors ORGANVM's Article VI.

### 4.4 Integration Points

**Architecture decision: Event bus over closure modification.**

`confirm_and_build()` in `queen_lifecycle_tools.py` is a local `async def` inside a closure
within `register_queen_lifecycle_tools()`. It closes over `phase_state`, `session_manager`, and
other locals — modifying the closure body is invasive and fragile for a first contribution.

Instead, we use Hive's existing event bus (`framework/runtime/event_bus.py`). The bus already
carries `QUEEN_PHASE_CHANGED`, `DRAFT_GRAPH_UPDATED`, `WORKER_LOADED`, etc. We add one new
event type:

```python
# In EventType(StrEnum):
DESIGN_VERSION_SAVED = "design_version_saved"
```

**Auto-snapshot on build:**
A small addition inside the `confirm_and_build` closure (after the existing
`save_flowchart_file` call at ~line 2146) emits the event with the graph/goal data:

```python
# Inside confirm_and_build, after flowchart save:
if event_bus:
    await event_bus.publish(AgentEvent(
        event_type=EventType.DESIGN_VERSION_SAVED,
        data={
            "graph_spec": graph.model_dump(),
            "goal": goal.model_dump(),
            "flowchart": phase_state.original_draft_graph,
            "agent_path": str(phase_state.agent_path),
            "description": f"Built by queen: {goal.name}",
        },
    ))
```

The `DesignVersionStore` subscribes to this event and handles persistence. This keeps
versioning logic out of the queen's lifecycle tools and follows Hive's existing pub/sub pattern.

**Auto-snapshot on evolution regeneration:**
Same event type — when a coding agent rewrites `agent.json`, the runner (or whichever
component saves the new graph) publishes `DESIGN_VERSION_SAVED` with `lifecycle_state=DRAFT`.

**Runner integration:**
In `runner.py`, after loading `agent.json` in `AgentRunner.load()`, compute the checksum of
the loaded graph and match it against the version index to record which version is being
executed. This populates `quality_metrics.sessions_run` on the matching version, enabling
the `CANDIDATE → VALIDATED` quality gate.

### 4.5 CLI Extension

Hive's CLI uses flat subcommands (`hive run`, `hive info`, `hive test-run`, `hive test-debug`),
not nested sub-subcommands. Follow the flat pattern with `version-` prefix:

```bash
hive version-list                     # List all versions for current agent
hive version-list --starred           # Only starred versions
hive version-show v_20260321_143022   # Show version details
hive version-diff v1 v2              # Diff two versions (nodes added/removed/changed)
hive version-restore v1              # Restore agent.json from version
hive version-star v1                 # Promote to PROMOTED state
```

Registered via `register_version_commands(subparsers)` in `framework/cli.py`,
following the existing `register_skill_commands` / `register_test_commands` pattern.

### 4.6 Tests

| Test file | Coverage |
|-----------|----------|
| `test_design_version.py` | Schema validation, checksum integrity, lifecycle state transitions (forward-only enforced, back-transitions rejected) |
| `test_design_version_store.py` | Save/load/list/diff/restore/prune. Atomic writes. Index consistency. Concurrent access safety. |
| `test_design_lifecycle.py` | State machine transitions, quality gate enforcement, starred retention during prune |
| `test_version_integration.py` | Auto-snapshot on queen build, version recorded during runner execution |

---

## 5. seed.yaml Wiring

```yaml
name: contrib--adenhq-hive
organ: IV
tier: contrib
promotion_status: LOCAL
produces:
  - pr_to_adenhq_hive
  - theory_extraction
  - generative_artifacts
  - public_narrative
  - relationship_capital
consumes:
  - agentic_titan_patterns
  - organvm_governance_rules
  - hive_evolution_data
subscriptions:
  - product.release
  - distribution.dispatched
```

---

## 6. Phased PR Strategy

To reduce review friction and increase merge probability for a first contribution:

**Phase 1 (core — the initial PR):**
- `DesignVersion` + `DesignVersionSummary` + `DesignVersionIndex` schemas
- `DesignVersionStore` (save/load/list/prune)
- `DesignLifecycleState` with forward-only transitions
- CLI: `version-list`, `version-show`
- Unit tests for schemas, store, state machine
- `DESIGN_VERSION_SAVED` event type added to `EventType`

**Phase 2 (lifecycle — follow-up PR):**
- Auto-snapshot hook via event bus subscription in `queen_lifecycle_tools.py`
- `version-diff`, `version-restore`, `version-star` CLI commands
- Quality gates (automated CANDIDATE→VALIDATED based on session success)
- Runner integration (version tracking during execution)
- Integration tests

Phase 1 is self-contained and solves the core reproducibility ask. Phase 2 adds governance.

---

## 7. Execution Order

1. **Create seed.yaml** (IV) — Wire formal edges
2. **Journal first** (IV) — Start the dated session log
3. **Theory extraction** (I) — Formalize the state machine mapping
4. **Build Phase 1 PR code** (III) — Schemas, store, basic CLI, tests
5. **Generate visualizations** (II) — State diagram, before/after, fusion map
6. **Ship Phase 1 PR** (III → upstream) — Claim #6613, submit
7. **Write the narrative** (V) — Essay, POSSE distribution
8. **Feed back** — PR review feedback → journal → theory refinement
9. **Build Phase 2** (III) — Lifecycle hooks, advanced CLI, integration tests

---

## 8. Success Criteria

- [ ] Hive PR addressing #6613 merged (or in active review)
- [ ] Theory documents in ORGAN-I formalize the fusion
- [ ] Journal in ORGAN-IV traces the full public process
- [ ] Essay in ORGAN-V published and distributed
- [ ] seed.yaml edges wired and validated by `validate-deps.py`
- [ ] The process itself demonstrates ORGANVM working as designed
