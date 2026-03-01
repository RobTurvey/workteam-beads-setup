# Beads Migration Learnings (AGRc → mysasmigrationtopython)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-03-01

## Purpose

This handoff captures what actually worked (and what failed) during AGRc Beads migration so a Copilot agent can implement the same pattern in `mysasmigrationtopython` with lower risk.

## Executive Summary

- Beads migration worked best as a **process-only overlay first**.
- **Devcontainer-first** execution prevented host-toolchain drift.
- Prerequisites must be installed at **devcontainer image build time** (Dockerfile), not deferred to manual/host setup.
- Operational reliability improved after adding:
  - Dolt runtime checks
  - snapshot-first recovery workflow
  - strict claim/handoff/review/close governance
- The highest-risk failures were not code bugs—they were **state management mistakes** (force init, broad stash of runtime files, unclear restore path).

## Non-Negotiable Controls

1. Claim before starting work.
2. Reviewer/user closes tasks.
3. Human approval for agent-generated planning artifacts.
4. Snapshot issue state before risky operations.
5. No broad `git stash -u` over Beads runtime state.

## What Broke in AGRc (and Required Fixes)

### 0) Prerequisites not guaranteed at image build time
- Problem: if core tools are only installed manually/post-create, startup became inconsistent across sessions.
- Rule:
  - Install critical tools in `.devcontainer/Dockerfile` build layer.
  - Use `postCreate` for verification/fallback only.
- Required build-time tools:
  - `node`, `npm`
  - `make`, `jq`, `go`, `curl`
  - `dolt`
  - `bd`

### 1) Upstream install URL mismatch
- Problem: setup references pointed to unavailable `withbeads/beads`.
- Working source: `https://github.com/steveyegge/beads`.
- Fix:
  - Primary install: `curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash`
  - Fallback: `npm install -g @beads/bd`

### 2) Dolt runtime missing / server not started
- Problem: `bd` commands failed when Dolt binary/server not ready.
- Fix:
  - Install Dolt in devcontainer.
  - Add startup target and health check (`make dolt-start`, `bd dolt test`).

### 3) Schema reset and empty-state confusion
- Problem: force re-init can wipe usable local state and create restore complexity.
- Rule:
  - Avoid `bd init --force` unless explicitly approved and logged.
  - Prefer normal recovery and snapshot comparison first.

### 4) Runtime files accidentally stashed/committed
- Problem: `.beads` runtime data in stash/working set created difficult recovery.
- Fix:
  - Ignore runtime paths:
    - `.beads/dolt-data/`
    - `.beads/dolt-server.log`
    - `.beads/issues.jsonl`
    - `.tmp/`

### 5) CLI drift for `beads-ui`
- Problem: older command shape failed.
- Working command:
  - `npx -y beads-ui@latest start --port 3002`

## Proven Operating Pattern

## Phase 0 — Baseline + Safety
- Freeze baseline behavior and identify no-change zones.
- Define rollback boundaries by phase.

## Phase 1 — Governance Overlay
- Publish lifecycle controls and reviewer policy.
- Enforce PASS/NITS/FAIL review protocol.

## Phase 2 — Devcontainer Enablement
- Verify `go`, `make`, `jq`, `dolt`, `bd`, `node`, `npm`.
- Validate full command surface (`mem-ready/next/active/ops`).

## Phase 3 — Docs Authority Tidy
- Single entrypoint and docs index with lifecycle/authority metadata.
- Keep legacy docs but clearly mark non-authoritative.

## Phase 4 — Pilot with Metrics
- Run 2–3 non-critical tasks end-to-end.
- Measure handoff quality, traceability, incidents, and cycle-time.
- Capture explicit `Execution Start Timestamp (UTC)` to separate queue time from implementation time.

## Phase 5 — Cutover Decision
- Record formal go-live decision and owner.
- Update navigation to Beads-primary path.

## Phase 6 — Cross-team Rollout
- Reuse templates and controls in second repo after cutover criteria pass.

## Command Pack (Copy/Paste)

```bash
# Start session
make dolt-start
export BD_ACTOR="<project>/<agent>"
make mem-ready

# Work lifecycle
make mem-show ISSUE=<id>
make mem-claim ISSUE=<id>
# implement/test
make mem-handoff

# Session safety
make mem-snapshot
git add docs/beads_snapshots/
git commit -m "beads: snapshot issue state"
```

## Recovery Decision Path (Use in New Repo)

1. Normal path:
   - `make dolt-start`
   - `bd dolt test`
   - `make mem-ready`
2. If state appears wrong:
   - Compare live issue count vs latest snapshot.
   - Do **not** force init immediately.
3. Last resort:
   - approved `bd init --force`
   - immediate rehydration + verification against snapshot.

## Definition of Done for mysasmigrationtopython Migration

- Devcontainer image build installs required prerequisites (including `dolt` and `bd`) without manual host steps.
- Devcontainer runs Beads commands reproducibly.
- Governance controls are active and auditable.
- Pilot tasks complete with PASS/NITS/FAIL evidence.
- Snapshot + recovery process documented and tested.
- Formal cutover decision recorded.

## Suggested Upstream Additions to workteam-beads-setup

1. Add a migration-learnings doc (state/recovery pitfalls).
2. Update install guidance to prefer `steveyegge/beads` source.
3. Include Dolt runtime checks in setup checklist.
4. Add required `.gitignore` runtime defaults.
5. Add a standard `make mem-recover` helper recipe.
6. Add execution-start timestamp guidance in pilot metrics template.

## Files in AGRc that back these learnings

- `docs/beads_source_of_truth_migration_plan.md`
- `docs/beads_setup_runlog.md`
- `docs/beads_template_hardening_defaults.md`
- `docs/beads_container_handover.md`
- `docs/02-review-checklist.md`
- `docs/issues/agrc-g9n.7_pilot_execution_run.md`
- `docs/beads_cutover_decision_2026-03-01.md`
