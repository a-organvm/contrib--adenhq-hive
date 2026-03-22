# Hive Design Versioning — Phase 1 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a design versioning system to AdenHQ/Hive that captures snapshots of agent graph definitions across evolution cycles, with a governed lifecycle state machine.

**Architecture:** Three new files following the existing `Checkpoint`/`CheckpointStore`/`CheckpointIndex` triad pattern. Schemas in `core/framework/schemas/`, store in `core/framework/storage/`, CLI registration in `core/framework/runner/cli.py`. Event type added to `core/framework/runtime/event_bus.py`. All persistence uses Hive's existing `atomic_write` from `core/framework/utils/io.py`.

**Tech Stack:** Python 3.11+, Pydantic v2, pytest, ruff (line-length=100, target py311)

**Spec:** `.claude/plans/2026-03-21-hive-fusion-design.md`

**Working directory:** All paths relative to `repo/` (the Hive fork at `/Users/4jp/Workspace/organvm-iv-taxis/contrib--adenhq-hive/repo/`)

---

## File Map

| Action | Path | Responsibility |
|--------|------|---------------|
| Create | `core/framework/schemas/design_version.py` | `DesignLifecycleState`, `DesignVersion`, `DesignVersionSummary`, `DesignVersionIndex` |
| Create | `core/framework/storage/design_version_store.py` | `DesignVersionStore` — save/load/list/prune/promote/restore |
| Create | `core/tests/test_design_version.py` | Schema tests — creation, checksum, verify, lifecycle transitions |
| Create | `core/tests/test_design_version_store.py` | Store tests — save/load/list/prune/restore, atomic writes, index consistency |
| Modify | `core/framework/runtime/event_bus.py:155` | Add `DESIGN_VERSION_SAVED` to `EventType` |
| Modify | `core/framework/schemas/__init__.py` | Export new schemas |
| Modify | `core/framework/storage/__init__.py` | Export `DesignVersionStore` |
| Modify | `core/framework/runner/cli.py` | Register `version-list` and `version-show` subcommands |

---

### Task 1: Create Feature Branch

**Files:** None (git only)

- [ ] **Step 1: Create branch from main**

```bash
cd /Users/4jp/Workspace/organvm-iv-taxis/contrib--adenhq-hive/repo
git fetch upstream
git checkout main
git merge upstream/main
git checkout -b feature/design-versioning
```

- [ ] **Step 2: Verify clean state**

Run: `git status`
Expected: `On branch feature/design-versioning`, clean working tree

---

### Task 2: DesignLifecycleState and DesignVersion Schema

**Files:**
- Create: `core/framework/schemas/design_version.py`
- Test: `core/tests/test_design_version.py`

- [ ] **Step 1: Write failing tests for schema creation and lifecycle**

Create `core/tests/test_design_version.py`:

```python
"""Tests for design version schemas — creation, checksum, lifecycle transitions."""

import pytest

from framework.schemas.design_version import (
    DesignLifecycleState,
    DesignVersion,
    DesignVersionIndex,
    DesignVersionSummary,
)


# === HELPERS ===

def _sample_graph_spec() -> dict:
    """Minimal GraphSpec-shaped dict for testing."""
    return {
        "id": "test-graph",
        "goal_id": "test-goal",
        "version": "1.0.0",
        "entry_node": "start",
        "terminal_nodes": ["end"],
        "nodes": [
            {"id": "start", "name": "Start", "description": "Entry", "node_type": "event_loop"},
            {"id": "end", "name": "End", "description": "Exit", "node_type": "event_loop"},
        ],
        "edges": [
            {"id": "e1", "source": "start", "target": "end", "condition": "on_success"},
        ],
    }


def _sample_goal() -> dict:
    """Minimal Goal-shaped dict for testing."""
    return {
        "id": "test-goal",
        "name": "Test Goal",
        "description": "A test goal",
        "success_criteria": [],
        "constraints": [],
    }


# === DESIGN VERSION CREATION ===

class TestDesignVersionCreate:
    def test_create_generates_id_and_timestamp(self):
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
            description="test version",
        )
        assert version.version_id.startswith("v_")
        assert version.created_at  # ISO 8601
        assert version.description == "test version"
        assert version.lifecycle_state == DesignLifecycleState.DRAFT

    def test_create_computes_checksum(self):
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
        )
        assert version.checksum
        assert len(version.checksum) == 16  # SHA-256 truncated to 16 hex chars

    def test_checksum_deterministic(self):
        gs = _sample_graph_spec()
        g = _sample_goal()
        v1 = DesignVersion.create(graph_spec=gs, goal=g)
        v2 = DesignVersion.create(graph_spec=gs, goal=g)
        assert v1.checksum == v2.checksum

    def test_checksum_changes_with_different_graph(self):
        gs1 = _sample_graph_spec()
        gs2 = _sample_graph_spec()
        gs2["entry_node"] = "different"
        g = _sample_goal()
        v1 = DesignVersion.create(graph_spec=gs1, goal=g)
        v2 = DesignVersion.create(graph_spec=gs2, goal=g)
        assert v1.checksum != v2.checksum

    def test_verify_passes_for_valid_version(self):
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
        )
        assert version.verify() is True

    def test_verify_fails_for_tampered_version(self):
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
        )
        version.graph_spec["entry_node"] = "tampered"
        assert version.verify() is False

    def test_create_with_parent_version(self):
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
            parent_version_id="v_20260321_000000_parent01",
        )
        assert version.parent_version_id == "v_20260321_000000_parent01"

    def test_create_with_flowchart(self):
        fc = {"nodes": [], "edges": [], "entry_node": "start"}
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
            flowchart=fc,
        )
        assert version.flowchart == fc


# === LIFECYCLE STATE TRANSITIONS ===

class TestDesignLifecycleState:
    def test_all_states_exist(self):
        assert DesignLifecycleState.DRAFT == "draft"
        assert DesignLifecycleState.CANDIDATE == "candidate"
        assert DesignLifecycleState.VALIDATED == "validated"
        assert DesignLifecycleState.PROMOTED == "promoted"
        assert DesignLifecycleState.ARCHIVED == "archived"


# === SUMMARY AND INDEX ===

class TestDesignVersionSummary:
    def test_from_version(self):
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
            description="test",
        )
        summary = DesignVersionSummary.from_version(version)
        assert summary.version_id == version.version_id
        assert summary.lifecycle_state == version.lifecycle_state
        assert summary.checksum == version.checksum


class TestDesignVersionIndex:
    def test_add_version(self):
        index = DesignVersionIndex(agent_id="test-agent")
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
        )
        index.add_version(version)
        assert index.total_versions == 1
        assert index.latest_version_id == version.version_id

    def test_add_promoted_version_updates_current(self):
        index = DesignVersionIndex(agent_id="test-agent")
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
            lifecycle_state=DesignLifecycleState.PROMOTED,
        )
        index.add_version(version)
        assert index.current_promoted_id == version.version_id

    def test_add_non_promoted_does_not_update_current(self):
        index = DesignVersionIndex(agent_id="test-agent")
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
            lifecycle_state=DesignLifecycleState.CANDIDATE,
        )
        index.add_version(version)
        assert index.current_promoted_id is None

    def test_get_version_summary(self):
        index = DesignVersionIndex(agent_id="test-agent")
        version = DesignVersion.create(
            graph_spec=_sample_graph_spec(),
            goal=_sample_goal(),
        )
        index.add_version(version)
        summary = index.get_version_summary(version.version_id)
        assert summary is not None
        assert summary.version_id == version.version_id

    def test_get_version_summary_not_found(self):
        index = DesignVersionIndex(agent_id="test-agent")
        assert index.get_version_summary("v_nonexistent") is None

    def test_filter_by_state(self):
        index = DesignVersionIndex(agent_id="test-agent")
        v1 = DesignVersion.create(
            graph_spec=_sample_graph_spec(), goal=_sample_goal(),
            lifecycle_state=DesignLifecycleState.DRAFT,
        )
        v2 = DesignVersion.create(
            graph_spec=_sample_graph_spec(), goal=_sample_goal(),
            lifecycle_state=DesignLifecycleState.CANDIDATE,
        )
        index.add_version(v1)
        index.add_version(v2)
        drafts = index.filter_by_state(DesignLifecycleState.DRAFT)
        assert len(drafts) == 1

    def test_get_starred(self):
        index = DesignVersionIndex(agent_id="test-agent")
        v1 = DesignVersion.create(
            graph_spec=_sample_graph_spec(), goal=_sample_goal(),
        )
        v1.starred = True
        v2 = DesignVersion.create(
            graph_spec=_sample_graph_spec(), goal=_sample_goal(),
        )
        index.add_version(v1)
        index.add_version(v2)
        starred = index.get_starred()
        assert len(starred) == 1

    def test_empty_graph_and_goal(self):
        """Edge case: empty dicts should still version cleanly."""
        version = DesignVersion.create(graph_spec={}, goal={})
        assert version.checksum
        assert version.verify() is True
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `cd core && uv run python -m pytest tests/test_design_version.py -v`
Expected: FAIL — `ModuleNotFoundError: No module named 'framework.schemas.design_version'`

- [ ] **Step 3: Implement the schema**

Create `core/framework/schemas/design_version.py`:

```python
"""
Design Version Schema — Agent design snapshots for reproducibility.

Captures the complete agent definition (GraphSpec + Goal + flowchart)
at a point in time, enabling version history, rollback, and lifecycle
governance for self-evolving agents.

Follows the Checkpoint/CheckpointSummary/CheckpointIndex pattern
from framework.schemas.checkpoint.
"""

import hashlib
import json
import uuid
from datetime import datetime
from enum import StrEnum
from typing import Any

from pydantic import BaseModel, Field


class DesignLifecycleState(StrEnum):
    """Lifecycle state of an agent design version.

    Forward-only transitions:
        DRAFT → CANDIDATE → VALIDATED → PROMOTED → ARCHIVED
        CANDIDATE → ARCHIVED (abandoned)
    """

    DRAFT = "draft"
    CANDIDATE = "candidate"
    VALIDATED = "validated"
    PROMOTED = "promoted"
    ARCHIVED = "archived"


# Legal forward transitions
ALLOWED_TRANSITIONS: dict[DesignLifecycleState, set[DesignLifecycleState]] = {
    DesignLifecycleState.DRAFT: {DesignLifecycleState.CANDIDATE},
    DesignLifecycleState.CANDIDATE: {
        DesignLifecycleState.VALIDATED,
        DesignLifecycleState.ARCHIVED,
    },
    DesignLifecycleState.VALIDATED: {DesignLifecycleState.PROMOTED},
    DesignLifecycleState.PROMOTED: {DesignLifecycleState.ARCHIVED},
    DesignLifecycleState.ARCHIVED: set(),
}


def _compute_checksum(graph_spec: dict[str, Any], goal: dict[str, Any]) -> str:
    """Compute deterministic checksum from graph_spec and goal.

    Uses sorted JSON serialization to ensure key ordering does not
    affect the hash.
    """
    content = json.dumps(graph_spec, sort_keys=True) + json.dumps(goal, sort_keys=True)
    return hashlib.sha256(content.encode()).hexdigest()[:16]


class DesignVersion(BaseModel):
    """Full design snapshot. Stored as individual JSON files."""

    version_id: str
    parent_version_id: str | None = None
    lifecycle_state: DesignLifecycleState = DesignLifecycleState.DRAFT
    created_at: str
    description: str = ""

    graph_spec: dict[str, Any]
    goal: dict[str, Any]
    flowchart: dict[str, Any] | None = None

    quality_metrics: dict[str, Any] = Field(default_factory=dict)
    starred: bool = False

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
        """Create with auto-generated ID, timestamp, and checksum."""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        short_uuid = uuid.uuid4().hex[:8]
        return cls(
            version_id=f"v_{timestamp}_{short_uuid}",
            parent_version_id=parent_version_id,
            lifecycle_state=lifecycle_state,
            created_at=datetime.now().isoformat(),
            description=description,
            graph_spec=graph_spec,
            goal=goal,
            flowchart=flowchart,
            checksum=_compute_checksum(graph_spec, goal),
        )

    def verify(self) -> bool:
        """Verify integrity of stored design."""
        return self.checksum == _compute_checksum(self.graph_spec, self.goal)


class DesignVersionSummary(BaseModel):
    """Lightweight metadata for index listings."""

    version_id: str
    lifecycle_state: DesignLifecycleState
    created_at: str
    description: str = ""
    parent_version_id: str | None = None
    starred: bool = False
    checksum: str = ""

    model_config = {"extra": "allow"}

    @classmethod
    def from_version(cls, version: DesignVersion) -> "DesignVersionSummary":
        """Create summary from full version."""
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
    """Manifest of all versions for an agent."""

    agent_id: str
    versions: list[DesignVersionSummary] = Field(default_factory=list)
    latest_version_id: str | None = None
    current_promoted_id: str | None = None
    total_versions: int = 0

    model_config = {"extra": "allow"}

    def add_version(self, version: DesignVersion) -> None:
        """Add a version to the index."""
        summary = DesignVersionSummary.from_version(version)
        self.versions.append(summary)
        self.latest_version_id = version.version_id
        self.total_versions = len(self.versions)
        if version.lifecycle_state == DesignLifecycleState.PROMOTED:
            self.current_promoted_id = version.version_id

    def get_version_summary(self, version_id: str) -> DesignVersionSummary | None:
        """Get summary by ID."""
        for s in self.versions:
            if s.version_id == version_id:
                return s
        return None

    def filter_by_state(
        self, state: DesignLifecycleState
    ) -> list[DesignVersionSummary]:
        """Filter versions by lifecycle state."""
        return [v for v in self.versions if v.lifecycle_state == state]

    def get_starred(self) -> list[DesignVersionSummary]:
        """Get all starred versions."""
        return [v for v in self.versions if v.starred]
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd core && uv run python -m pytest tests/test_design_version.py -v`
Expected: All 19 tests PASS

- [ ] **Step 5: Run ruff**

Run: `cd core && uv run ruff check framework/schemas/design_version.py && uv run ruff format --check framework/schemas/design_version.py`
Expected: No errors

- [ ] **Step 6: Commit**

```bash
git add core/framework/schemas/design_version.py core/tests/test_design_version.py
git commit -m "feat(schemas): add DesignVersion schema with lifecycle state machine

Introduces DesignVersion, DesignVersionSummary, and DesignVersionIndex
for capturing agent design snapshots across evolution cycles. Includes
SHA-256 integrity checksums and forward-only lifecycle transitions
(DRAFT → CANDIDATE → VALIDATED → PROMOTED → ARCHIVED)."
```

---

### Task 3: DesignVersionStore

**Files:**
- Create: `core/framework/storage/design_version_store.py`
- Test: `core/tests/test_design_version_store.py`

- [ ] **Step 1: Write failing tests for store operations**

Create `core/tests/test_design_version_store.py`:

```python
"""Tests for DesignVersionStore — save, load, list, prune, promote, restore."""

import json

import pytest

from framework.schemas.design_version import (
    DesignLifecycleState,
    DesignVersion,
    DesignVersionIndex,
)
from framework.storage.design_version_store import DesignVersionStore


def _sample_graph_spec() -> dict:
    return {
        "id": "test-graph",
        "goal_id": "test-goal",
        "version": "1.0.0",
        "entry_node": "start",
        "terminal_nodes": ["end"],
        "nodes": [
            {"id": "start", "name": "Start", "description": "Entry", "node_type": "event_loop"},
            {"id": "end", "name": "End", "description": "Exit", "node_type": "event_loop"},
        ],
        "edges": [
            {"id": "e1", "source": "start", "target": "end", "condition": "on_success"},
        ],
    }


def _sample_goal() -> dict:
    return {
        "id": "test-goal",
        "name": "Test Goal",
        "description": "A test goal",
        "success_criteria": [],
        "constraints": [],
    }


def _make_version(
    lifecycle_state: DesignLifecycleState = DesignLifecycleState.DRAFT,
    description: str = "test",
) -> DesignVersion:
    return DesignVersion.create(
        graph_spec=_sample_graph_spec(),
        goal=_sample_goal(),
        lifecycle_state=lifecycle_state,
        description=description,
    )


class TestDesignVersionStoreSaveLoad:
    @pytest.mark.asyncio
    async def test_save_and_load(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        version = _make_version()
        await store.save_version(version)
        loaded = await store.load_version(version.version_id)
        assert loaded is not None
        assert loaded.version_id == version.version_id
        assert loaded.checksum == version.checksum
        assert loaded.verify() is True

    @pytest.mark.asyncio
    async def test_load_nonexistent_returns_none(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        loaded = await store.load_version("v_nonexistent")
        assert loaded is None

    @pytest.mark.asyncio
    async def test_save_creates_index(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        version = _make_version()
        await store.save_version(version)
        index = await store.load_index()
        assert index is not None
        assert index.total_versions == 1
        assert index.latest_version_id == version.version_id


class TestDesignVersionStoreList:
    @pytest.mark.asyncio
    async def test_list_versions(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        v1 = _make_version(description="first")
        v2 = _make_version(description="second")
        await store.save_version(v1)
        await store.save_version(v2)
        versions = await store.list_versions()
        assert len(versions) == 2

    @pytest.mark.asyncio
    async def test_list_by_state(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        v1 = _make_version(lifecycle_state=DesignLifecycleState.DRAFT)
        v2 = _make_version(lifecycle_state=DesignLifecycleState.CANDIDATE)
        await store.save_version(v1)
        await store.save_version(v2)
        drafts = await store.list_versions(lifecycle_state=DesignLifecycleState.DRAFT)
        assert len(drafts) == 1
        assert drafts[0].version_id == v1.version_id

    @pytest.mark.asyncio
    async def test_list_starred(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        v1 = _make_version()
        v1.starred = True
        v2 = _make_version()
        await store.save_version(v1)
        await store.save_version(v2)
        starred = await store.list_versions(starred=True)
        assert len(starred) == 1


class TestDesignVersionStorePromote:
    @pytest.mark.asyncio
    async def test_promote_forward(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        version = _make_version(lifecycle_state=DesignLifecycleState.DRAFT)
        await store.save_version(version)
        result = await store.promote_version(
            version.version_id, DesignLifecycleState.CANDIDATE
        )
        assert result is True
        loaded = await store.load_version(version.version_id)
        assert loaded.lifecycle_state == DesignLifecycleState.CANDIDATE

    @pytest.mark.asyncio
    async def test_promote_backward_rejected(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        version = _make_version(lifecycle_state=DesignLifecycleState.CANDIDATE)
        await store.save_version(version)
        result = await store.promote_version(
            version.version_id, DesignLifecycleState.DRAFT
        )
        assert result is False
        loaded = await store.load_version(version.version_id)
        assert loaded.lifecycle_state == DesignLifecycleState.CANDIDATE

    @pytest.mark.asyncio
    async def test_promote_invalid_transition_rejected(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        version = _make_version(lifecycle_state=DesignLifecycleState.DRAFT)
        await store.save_version(version)
        # DRAFT → VALIDATED is not allowed (must go through CANDIDATE)
        result = await store.promote_version(
            version.version_id, DesignLifecycleState.VALIDATED
        )
        assert result is False


class TestDesignVersionStoreRestore:
    @pytest.mark.asyncio
    async def test_restore_writes_agent_json(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        version = _make_version()
        await store.save_version(version)
        agent_json_path = tmp_path / "agent.json"
        await store.restore_version(version.version_id, agent_json_path)
        assert agent_json_path.exists()
        data = json.loads(agent_json_path.read_text(encoding="utf-8"))
        assert data["graph"] == version.graph_spec
        assert data["goal"] == version.goal

    @pytest.mark.asyncio
    async def test_restore_nonexistent_returns_false(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        result = await store.restore_version("v_nope", tmp_path / "agent.json")
        assert result is False


class TestDesignVersionStorePrune:
    @pytest.mark.asyncio
    async def test_prune_removes_old_unstarred(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        v1 = _make_version(description="old")
        v2 = _make_version(description="new")
        await store.save_version(v1)
        await store.save_version(v2)
        # Use -1 to ensure all versions are older than cutoff (avoids timing flakiness)
        deleted = await store.prune_versions(max_age_days=-1, keep_starred=True)
        assert deleted >= 1

    @pytest.mark.asyncio
    async def test_prune_keeps_starred(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        v1 = _make_version()
        v1.starred = True
        await store.save_version(v1)
        deleted = await store.prune_versions(max_age_days=-1, keep_starred=True)
        assert deleted == 0
        loaded = await store.load_version(v1.version_id)
        assert loaded is not None

    @pytest.mark.asyncio
    async def test_prune_with_empty_store(self, tmp_path):
        store = DesignVersionStore(tmp_path)
        deleted = await store.prune_versions(max_age_days=-1)
        assert deleted == 0
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `cd core && uv run python -m pytest tests/test_design_version_store.py -v`
Expected: FAIL — `ModuleNotFoundError: No module named 'framework.storage.design_version_store'`

- [ ] **Step 3: Implement the store**

Create `core/framework/storage/design_version_store.py`:

```python
"""
Design Version Store — Manages versioned agent design snapshots.

Handles saving, loading, listing, promoting, restoring, and pruning
of design versions for agent reproducibility.

Follows the CheckpointStore pattern from framework.storage.checkpoint_store.
"""

import asyncio
import json
import logging
from datetime import datetime, timedelta
from pathlib import Path

from framework.schemas.design_version import (
    ALLOWED_TRANSITIONS,
    DesignLifecycleState,
    DesignVersion,
    DesignVersionIndex,
    DesignVersionSummary,
)
from framework.utils.io import atomic_write

logger = logging.getLogger(__name__)


class DesignVersionStore:
    """
    Manages design version storage with atomic writes.

    Directory structure:
        versions/
            index.json                      # Version manifest
            v_{timestamp}_{uuid}.json       # Individual versions
    """

    def __init__(self, base_path: Path):
        self.base_path = Path(base_path)
        self.versions_dir = self.base_path / "versions"
        self.index_path = self.versions_dir / "index.json"
        self._index_lock = asyncio.Lock()

    async def save_version(self, version: DesignVersion) -> None:
        """Atomically save version and update index."""

        def _write():
            self.versions_dir.mkdir(parents=True, exist_ok=True)
            version_path = self.versions_dir / f"{version.version_id}.json"
            with atomic_write(version_path) as f:
                f.write(version.model_dump_json(indent=2))
            logger.debug("Saved design version %s", version.version_id)

        await asyncio.to_thread(_write)

        async with self._index_lock:
            await self._update_index_add(version)

    async def load_version(self, version_id: str) -> DesignVersion | None:
        """Load version by ID."""

        def _read() -> DesignVersion | None:
            version_path = self.versions_dir / f"{version_id}.json"
            if not version_path.exists():
                return None
            try:
                return DesignVersion.model_validate_json(
                    version_path.read_text(encoding="utf-8")
                )
            except Exception as e:
                logger.error("Failed to load version %s: %s", version_id, e)
                return None

        return await asyncio.to_thread(_read)

    async def load_index(self) -> DesignVersionIndex | None:
        """Load version index."""

        def _read() -> DesignVersionIndex | None:
            if not self.index_path.exists():
                return None
            try:
                return DesignVersionIndex.model_validate_json(
                    self.index_path.read_text(encoding="utf-8")
                )
            except Exception as e:
                logger.error("Failed to load version index: %s", e)
                return None

        return await asyncio.to_thread(_read)

    async def list_versions(
        self,
        lifecycle_state: DesignLifecycleState | None = None,
        starred: bool | None = None,
    ) -> list[DesignVersionSummary]:
        """List versions with optional filters."""
        index = await self.load_index()
        if not index:
            return []

        versions = index.versions

        if lifecycle_state is not None:
            versions = [v for v in versions if v.lifecycle_state == lifecycle_state]

        if starred is not None:
            versions = [v for v in versions if v.starred == starred]

        return versions

    async def promote_version(
        self,
        version_id: str,
        target_state: DesignLifecycleState,
    ) -> bool:
        """Forward-only state transition with validation.

        Returns True if transition succeeded, False if rejected.
        """
        version = await self.load_version(version_id)
        if version is None:
            logger.warning("Version %s not found", version_id)
            return False

        allowed = ALLOWED_TRANSITIONS.get(version.lifecycle_state, set())
        if target_state not in allowed:
            logger.warning(
                "Transition %s → %s not allowed for %s",
                version.lifecycle_state,
                target_state,
                version_id,
            )
            return False

        version.lifecycle_state = target_state
        if target_state == DesignLifecycleState.PROMOTED:
            version.starred = True

        def _write():
            version_path = self.versions_dir / f"{version_id}.json"
            with atomic_write(version_path) as f:
                f.write(version.model_dump_json(indent=2))

        await asyncio.to_thread(_write)

        async with self._index_lock:
            index = await self.load_index()
            if index:
                for s in index.versions:
                    if s.version_id == version_id:
                        s.lifecycle_state = target_state
                        s.starred = version.starred
                        break
                if target_state == DesignLifecycleState.PROMOTED:
                    index.current_promoted_id = version_id
                await self._write_index(index)

        logger.info("Promoted %s to %s", version_id, target_state)
        return True

    async def restore_version(
        self,
        version_id: str,
        agent_json_path: Path,
    ) -> bool:
        """Restore agent.json from a versioned snapshot.

        Returns True if restored, False if version not found.
        """
        version = await self.load_version(version_id)
        if version is None:
            return False

        def _write():
            agent_data = {"graph": version.graph_spec, "goal": version.goal}
            with atomic_write(agent_json_path) as f:
                json.dump(agent_data, f, indent=2)

        await asyncio.to_thread(_write)
        logger.info("Restored %s to %s", version_id, agent_json_path)
        return True

    async def prune_versions(
        self,
        max_age_days: int = 30,
        keep_starred: bool = True,
    ) -> int:
        """Prune old unstarred versions.

        Returns number of versions deleted.
        """
        index = await self.load_index()
        if not index or not index.versions:
            return 0

        cutoff = datetime.now() - timedelta(days=max_age_days)
        to_delete = []

        for s in index.versions:
            if keep_starred and s.starred:
                continue
            try:
                created = datetime.fromisoformat(s.created_at)
                if created < cutoff:
                    to_delete.append(s.version_id)
            except Exception:
                continue

        deleted = 0
        for vid in to_delete:
            if await self._delete_version_file(vid):
                deleted += 1

        if deleted > 0:
            async with self._index_lock:
                index = await self.load_index()
                if index:
                    index.versions = [
                        v for v in index.versions if v.version_id not in to_delete
                    ]
                    index.total_versions = len(index.versions)
                    if index.latest_version_id in to_delete:
                        index.latest_version_id = (
                            index.versions[-1].version_id if index.versions else None
                        )
                    await self._write_index(index)
            logger.info("Pruned %d versions older than %d days", deleted, max_age_days)

        return deleted

    async def _delete_version_file(self, version_id: str) -> bool:
        def _delete() -> bool:
            path = self.versions_dir / f"{version_id}.json"
            if not path.exists():
                return False
            path.unlink()
            return True

        return await asyncio.to_thread(_delete)

    async def _update_index_add(self, version: DesignVersion) -> None:
        index = await self.load_index()
        if not index:
            index = DesignVersionIndex(agent_id=self.base_path.name)
        index.add_version(version)
        await self._write_index(index)

    async def _write_index(self, index: DesignVersionIndex) -> None:
        def _write():
            self.versions_dir.mkdir(parents=True, exist_ok=True)
            with atomic_write(self.index_path) as f:
                f.write(index.model_dump_json(indent=2))

        await asyncio.to_thread(_write)
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd core && uv run python -m pytest tests/test_design_version_store.py -v`
Expected: All 11 tests PASS

- [ ] **Step 5: Run ruff**

Run: `cd core && uv run ruff check framework/storage/design_version_store.py && uv run ruff format --check framework/storage/design_version_store.py`
Expected: No errors

- [ ] **Step 6: Commit**

```bash
git add core/framework/storage/design_version_store.py core/tests/test_design_version_store.py
git commit -m "feat(storage): add DesignVersionStore with atomic writes and lifecycle promotion

File-based version store for agent design snapshots. Supports save, load,
list (with state/starred filters), forward-only promote, restore to
agent.json, and time-based prune with starred retention."
```

---

### Task 4: EventType Registration and Module Exports

**Files:**
- Modify: `core/framework/runtime/event_bus.py`
- Modify: `core/framework/schemas/__init__.py`
- Modify: `core/framework/storage/__init__.py`

- [ ] **Step 1: Add DESIGN_VERSION_SAVED to EventType**

In `core/framework/runtime/event_bus.py`, after line 163 (`TRIGGER_UPDATED = "trigger_updated"`), add:

```python
    # Design versioning (agent design snapshots for reproducibility)
    DESIGN_VERSION_SAVED = "design_version_saved"
```

- [ ] **Step 2: Update schemas __init__.py**

In `core/framework/schemas/__init__.py`, add:

```python
from framework.schemas.design_version import (
    DesignLifecycleState,
    DesignVersion,
    DesignVersionIndex,
    DesignVersionSummary,
)
```

And add to `__all__`:
```python
    "DesignLifecycleState",
    "DesignVersion",
    "DesignVersionIndex",
    "DesignVersionSummary",
```

- [ ] **Step 3: Update storage __init__.py**

In `core/framework/storage/__init__.py`, add:

```python
from framework.storage.design_version_store import DesignVersionStore
```

And add `"DesignVersionStore"` to `__all__`.

- [ ] **Step 4: Run full test suite to verify no regressions**

Run: `cd core && uv run python -m pytest tests/test_design_version.py tests/test_design_version_store.py -v`
Expected: All 32 tests PASS

- [ ] **Step 5: Run ruff on modified files**

Run: `cd core && uv run ruff check framework/runtime/event_bus.py framework/schemas/__init__.py framework/storage/__init__.py && uv run ruff format --check framework/runtime/event_bus.py framework/schemas/__init__.py framework/storage/__init__.py`
Expected: No errors

- [ ] **Step 6: Commit**

```bash
git add core/framework/runtime/event_bus.py core/framework/schemas/__init__.py core/framework/storage/__init__.py
git commit -m "chore: register DesignVersion types in module exports and event bus"
```

---

### Task 5: CLI Commands (version-list, version-show)

**Files:**
- Modify: `core/framework/runner/cli.py`

- [ ] **Step 1: Add version-list and version-show commands**

At the end of the `register_commands` function in `core/framework/runner/cli.py`, add:

```python
    # version-list command
    vlist_parser = subparsers.add_parser(
        "version-list",
        help="List design versions for an agent",
        description="List all saved design versions with lifecycle state and timestamps.",
    )
    vlist_parser.add_argument(
        "agent_path",
        type=str,
        help="Path to agent folder",
    )
    vlist_parser.add_argument(
        "--starred",
        action="store_true",
        help="Show only starred/promoted versions",
    )
    vlist_parser.add_argument(
        "--state",
        type=str,
        choices=["draft", "candidate", "validated", "promoted", "archived"],
        help="Filter by lifecycle state",
    )
    vlist_parser.set_defaults(func=_handle_version_list)

    # version-show command
    vshow_parser = subparsers.add_parser(
        "version-show",
        help="Show details of a specific design version",
        description="Display full details of a saved design version.",
    )
    vshow_parser.add_argument(
        "agent_path",
        type=str,
        help="Path to agent folder",
    )
    vshow_parser.add_argument(
        "version_id",
        type=str,
        help="Version ID (e.g., v_20260321_143022_abc12345)",
    )
    vshow_parser.set_defaults(func=_handle_version_show)
```

Then add the handler functions (before `register_commands` or at module level):

```python
def _handle_version_list(args: argparse.Namespace) -> None:
    """Handle version-list command."""
    from framework.schemas.design_version import DesignLifecycleState
    from framework.storage.design_version_store import DesignVersionStore

    agent_path = Path(args.agent_path).resolve()
    if not agent_path.is_dir():
        print(f"Error: {agent_path} is not a directory")
        sys.exit(1)

    store = DesignVersionStore(agent_path)

    state_filter = None
    if args.state:
        state_filter = DesignLifecycleState(args.state)

    starred_filter = True if args.starred else None

    versions = asyncio.run(
        store.list_versions(lifecycle_state=state_filter, starred=starred_filter)
    )

    if not versions:
        print("No versions found.")
        return

    for v in versions:
        star = "★" if v.starred else " "
        print(f"  {star} {v.version_id}  [{v.lifecycle_state}]  {v.created_at}  {v.description}")


def _handle_version_show(args: argparse.Namespace) -> None:
    """Handle version-show command."""
    from framework.storage.design_version_store import DesignVersionStore

    agent_path = Path(args.agent_path).resolve()
    store = DesignVersionStore(agent_path)

    version = asyncio.run(store.load_version(args.version_id))
    if version is None:
        print(f"Version {args.version_id} not found.")
        sys.exit(1)

    integrity = "✓ valid" if version.verify() else "✗ CORRUPTED"
    print(f"Version:    {version.version_id}")
    print(f"State:      {version.lifecycle_state}")
    print(f"Created:    {version.created_at}")
    print(f"Parent:     {version.parent_version_id or '(none)'}")
    print(f"Starred:    {'yes' if version.starred else 'no'}")
    print(f"Checksum:   {version.checksum} ({integrity})")
    print(f"Description: {version.description}")
    print(f"Nodes:      {len(version.graph_spec.get('nodes', []))}")
    print(f"Edges:      {len(version.graph_spec.get('edges', []))}")
    if version.quality_metrics:
        print(f"Metrics:    {json.dumps(version.quality_metrics, indent=2)}")
```

- [ ] **Step 2: Write CLI registration test**

Add to `core/tests/test_design_version.py` (or create `core/tests/test_version_cli.py`):

```python
class TestVersionCLI:
    def test_version_list_subcommand_registers(self):
        """Verify version-list subcommand is registered without errors."""
        import argparse
        from framework.runner.cli import register_commands

        parser = argparse.ArgumentParser()
        subparsers = parser.add_subparsers()
        register_commands(subparsers)
        # Should not raise — version-list exists as a subcommand
        args = parser.parse_args(["version-list", "/tmp/fake-agent"])
        assert hasattr(args, "func")

    def test_version_show_subcommand_registers(self):
        """Verify version-show subcommand is registered without errors."""
        import argparse
        from framework.runner.cli import register_commands

        parser = argparse.ArgumentParser()
        subparsers = parser.add_subparsers()
        register_commands(subparsers)
        args = parser.parse_args(["version-show", "/tmp/fake-agent", "v_test"])
        assert hasattr(args, "func")
```

- [ ] **Step 3: Run ruff on cli.py**

Run: `cd core && uv run ruff check framework/runner/cli.py && uv run ruff format --check framework/runner/cli.py`
Expected: No errors

- [ ] **Step 4: Run tests including CLI tests**

Run: `cd core && uv run python -m pytest tests/test_design_version.py -v -k "CLI"`
Expected: 2 CLI tests PASS

- [ ] **Step 5: Run full test suite to verify no regressions**

Run: `cd core && uv run python -m pytest tests/ -v --timeout=30 -x`
Expected: All existing tests still pass

- [ ] **Step 6: Commit**

```bash
git add core/framework/runner/cli.py core/tests/test_design_version.py
git commit -m "feat(cli): add version-list and version-show commands

Registers hive version-list (with --starred and --state filters) and
hive version-show (displays full version details with integrity check).
Follows the existing flat subcommand pattern (test-run, test-debug)."
```

---

### Task 6: Run make check && make test

**Files:** None (validation only)

- [ ] **Step 1: Run make check (CI-safe linting)**

Run: `cd /Users/4jp/Workspace/organvm-iv-taxis/contrib--adenhq-hive/repo && make check`
Expected: All ruff checks pass for core/ and tools/

- [ ] **Step 2: Run make test (full test suite)**

Run: `cd /Users/4jp/Workspace/organvm-iv-taxis/contrib--adenhq-hive/repo && make test`
Expected: All tests pass including new design version tests

- [ ] **Step 3: Final commit if any formatting fixes needed**

If ruff auto-fixes anything:
```bash
git add -A
git commit -m "style: ruff formatting fixes"
```

---

### Task 7: Journal Entry (ORGAN-IV)

**Files:**
- Create: `contrib--adenhq-hive/journal/2026-03-21-session-01.md`
- Create: `contrib--adenhq-hive/seed.yaml`

- [ ] **Step 1: Create journal directory and first entry**

Create `/Users/4jp/Workspace/organvm-iv-taxis/contrib--adenhq-hive/journal/2026-03-21-session-01.md` documenting:
- Design spec completed (Approach B — cross-organ symbiote)
- Phase 1 code built: DesignVersion schema, DesignVersionStore, CLI, tests
- Event bus integration point identified (DESIGN_VERSION_SAVED)
- Review findings addressed (closure→event bus, flat CLI, canonical checksums)
- Next: claim #6613, submit PR

- [ ] **Step 2: Create seed.yaml**

Create `/Users/4jp/Workspace/organvm-iv-taxis/contrib--adenhq-hive/seed.yaml`:

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

- [ ] **Step 3: Commit journal and seed**

```bash
cd /Users/4jp/Workspace/organvm-iv-taxis/contrib--adenhq-hive
git add journal/ seed.yaml
git commit -m "docs(journal): first session entry and seed.yaml wiring"
```
