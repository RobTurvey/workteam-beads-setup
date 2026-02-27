# 08 — Ultimate Setup Blueprint (Zero-Regression Adoption)

This is the implementation plan for introducing Beads into a project that is already hard-won and stable.

Use this when you want maximum confidence and minimum disruption.

## Primary Objective

Adopt Beads and multi-agent workflow **without breaking the existing dev workflow**.

## Non-Negotiable Rules

1. Do not modify the existing build/run path until baseline checks pass.
2. Introduce Beads in phases with explicit go/no-go gates.
3. Keep rollback simple: one commit revert per phase.
4. Treat `.beads/issues.jsonl` in git as durable history.
5. Use a pilot first; scale to team only after pilot PASS.

## Operating Model

- One repo = one Beads issue universe
- One team baseline repo for templates = `workteam-beads-setup`
- Work repos live under `C:/Dev/<project>/`
- Prefer devcontainer-first for consistency

## Phase Plan

### Phase 0 — Baseline Freeze (No Beads Changes Yet)

Goal: prove current project is stable before adding Beads.

Checklist:

- Record current dev startup commands (build/run/test)
- Confirm current devcontainer works (if used)
- Confirm internal mirrors/proxy setup is already functional
- Capture a known-good commit SHA

Go/No-Go:

- Go only if baseline workflow is green and repeatable

Rollback:

- No changes in this phase

### Phase 1 — Add Beads with No Workflow Replacement

Goal: add Beads files and commands without replacing existing project commands.

Actions:

1. Copy templates (`.beads/config.yaml`, `.agent/*`, `.gitignore`, Makefile workflow target path)
2. If repo already has a Makefile: use `Makefile.workteam` + `include Makefile.workteam`
3. Run `bd init --prefix <project-prefix>`
4. Run only validation commands:
   - `make mem-ready`
   - `make mem-next`
   - `make mem-active`
   - `make mem-ops`

Go/No-Go:

- Go only if all mem commands run and existing project commands still work unchanged

Rollback:

- Revert the single Beads-introduction commit

### Phase 2 — Enterprise Network Hardening

Goal: ensure setup is compliant with internal repositories and mirrors.

Actions:

1. Follow `docs/07-enterprise-mirrors.md`
2. Configure apt/npm/go mirrors and proxy env vars
3. Validate mirror reachability before installs
4. Re-run mem command checks

Go/No-Go:

- Go only if all installs/metadata lookups resolve through internal approved endpoints

Rollback:

- Revert mirror config changes and return to last passing state

### Phase 3 — Optional UI Enablement

Goal: add Beads UI without affecting core CLI workflow.

Actions:

1. Ensure Node 22+ path is available (container or host)
2. Run `make beads-ui`
3. Verify dashboard on `http://127.0.0.1:3002`

Go/No-Go:

- UI may fail independently; this should not block CLI workflow adoption

Rollback:

- Keep UI disabled and continue CLI-only

### Phase 4 — Pilot Epic Validation

Goal: validate end-to-end team behavior on a small real scope.

Actions:

1. Create one pilot epic with 2–3 child tasks
2. Run full lifecycle: ready → show → claim → work → close → handoff
3. Run reciprocal peer review using checklist

Go/No-Go:

- Go only if pilot completes with no process ambiguity and no workflow regressions

Rollback:

- Keep Beads open for tracking but pause broader rollout

### Phase 5 — Team Rollout

Goal: scale to all team members safely.

Actions:

1. Share `workteam-beads-setup` repo
2. Require path convention `C:/Dev/<project>/`
3. Enforce setup option selection:
   - Option A devcontainer-first (default)
   - Option B local-native (exception)
4. Add a short onboarding runbook per team role

Go/No-Go:

- Rollout continues only if first 2 team members onboard cleanly

Rollback:

- Freeze new onboarding and revert to pilot cohort only

## Validation Matrix (Every Phase)

Run these checks each time:

- Existing project build/test/start still pass
- `make mem-ready` works
- `make mem-next` works
- `make mem-active` works
- `make mem-ops` works
- Optional: `make beads-ui` works
- `git status` is clean after setup steps

## Recovery Strategy

If a container/machine reset causes local DB drift:

```bash
bd init --from-jsonl --prefix <project-prefix>
make mem-ready
```

This rehydrates live state from git-tracked `.beads/issues.jsonl`.

## Copilot Execution Contract

When asked to onboard a new project, Copilot should:

1. Read this blueprint first
2. Confirm Phase 0 baseline checks
3. Apply only one phase at a time
4. Run validation checks after each phase
5. Stop and report before crossing a failed gate
6. Never replace existing build/run commands unless explicitly requested

## Final Outcome

When done correctly, you get:

- Stable existing project workflow preserved
- Beads added as a controlled overlay
- Team-ready multi-agent operating model
- Documented rollback path at every stage
