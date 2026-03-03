# Beads UI Troubleshooting + Backup/Recovery Runbook (AGRc)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-03-03

This runbook explains how to troubleshoot **Beads UI** and execute the **backup/recovery process** used in this project.

## 1) Scope and Safety

- Scope is process/tooling only.
- Do not change runtime auction behavior while performing Beads operations.
- No-change-zone runtime files:
  - `scripts/auctions/scheduler.ts`
  - `scripts/auctions/monitor.ts`
  - `scripts/auctions/reporter.ts`

Primary references:
- [`docs/beads_docs_index.md`](beads_docs_index.md)
- [`docs/beads_container_handover.md`](beads_container_handover.md)
- [`docs/beads_setup_runlog.md`](beads_setup_runlog.md)
- [`docs/beads_template_hardening_defaults.md`](beads_template_hardening_defaults.md)
- [`Makefile.workteam`](../Makefile.workteam)

---

## 2) Quick Start Commands

From repo root:

```bash
make dolt-start
export BD_ACTOR="agrc/<your-actor>"
make mem-ready
make beads-ui
```

Expected:
- `make dolt-start` confirms Dolt is reachable on `127.0.0.1:3307`.
- `make mem-ready` returns Beads ready queue without backend errors.
- `make beads-ui` starts UI (default `http://127.0.0.1:3002`).

---

## 3) Beads UI Troubleshooting

## A. `make beads-ui` fails immediately

### Symptom
- Error indicates `npx` missing or command not found.

### Checks
```bash
command -v npx
command -v node
command -v npm
```

### Fix
- Install Node.js/npm in environment.
- Retry `make beads-ui`.

Reference target: [`beads-ui`](../Makefile.workteam)

## B. UI starts but page does not load

### Symptom
- Terminal shows process start, browser cannot connect to `127.0.0.1:3002`.

### Checks
```bash
make beads-ui
```
- Confirm you are using `http://127.0.0.1:3002`.

### Fix
- Restart UI process:
```bash
make beads-ui-stop
make beads-ui
```

Reference targets: [`beads-ui-stop`](../Makefile.workteam), [`beads-ui`](../Makefile.workteam)

## C. UI opens but shows empty/inconsistent issue state

### Symptom
- Queue looks empty or mismatched with expected issue state.

### Checks
```bash
make mem-ready
make mem-active
make mem-ops
```

### Fix path
1. Run recovery preflight:
```bash
make mem-recover
```
2. Re-open UI after recovery:
```bash
make beads-ui-stop
make beads-ui
```
3. If mismatch persists, log incident in [`docs/beads_setup_runlog.md`](beads_setup_runlog.md) and compare snapshot counts (see section 5).

---

## 4) Backend Troubleshooting (Dolt + Beads CLI)

## A. Dolt unreachable (`127.0.0.1:3307`)

### Symptom
- `make mem-ready` / `bd` commands fail with connection errors.

### Checks
```bash
bd dolt test
make dolt-start
bd dolt test
```

### Fix
- Always start via `make dolt-start` before Beads workflow commands.
- If still failing, follow environment recovery steps in [`docs/beads_container_handover.md`](beads_container_handover.md).

## B. Beads schema/state issues (e.g., missing tables)

### Symptom
- Errors such as missing `issues` table.

### Fix
- Follow logged mitigation path in [`docs/beads_setup_runlog.md`](beads_setup_runlog.md) and project handover guidance in [`docs/beads_container_handover.md`](beads_container_handover.md).
- Do not force re-init unless recovery guidance explicitly requires it.

## C. Actor/audit attribution missing

### Symptom
- Notes/updates not attributed to the expected agent identity.

### Fix
```bash
export BD_ACTOR="agrc/<actor>"
```

Set this before `make mem-claim`, `bd update`, or `bd close` operations.

---

## 5) Backup Process (What to Preserve)

This project treats snapshot files under [`docs/beads_snapshots/`](beads_snapshots) as the git-backed backup source.

## Standard backup steps
```bash
make mem-snapshot
git add docs/beads_snapshots/
git commit -m "beads: snapshot issue state"
```

What this gives you:
- Timestamped snapshot file (`issues-<UTC>.json`).
- Versioned recovery point in git history.

Important local-only runtime paths (do not treat as canonical backup):
- `.beads/dolt-data/`
- `.beads/dolt-server.log`
- `.beads/issues.jsonl`
- `.tmp/`

Reference: [`docs/beads_template_hardening_defaults.md`](beads_template_hardening_defaults.md)

---

## 6) Recovery Process (Normal + Emergency)

## Normal restart recovery (preferred)

```bash
make mem-recover
```

This performs:
- Dolt start/test
- live issue count check
- latest snapshot discovery under `docs/beads_snapshots/`
- readiness verification (`make mem-ready`)

## Emergency recovery (state mismatch)

When queue/state still looks wrong after normal recovery:
1. Do **not** immediately run broad destructive cleanup.
2. Confirm latest snapshot exists in `docs/beads_snapshots/`.
3. Compare live issue count vs snapshot issue count.
4. Record incident in [`docs/beads_setup_runlog.md`](beads_setup_runlog.md).
5. Follow project restore path in [`docs/beads_container_handover.md`](beads_container_handover.md).

---

## 7) Verification Checklist After Recovery

Run and confirm all pass:

```bash
bd dolt test
make mem-ready
make mem-next
make mem-active
make mem-ops
```

Optional UI verification:
```bash
make beads-ui
```

Final checks:
- Queue views are populated as expected.
- UI reflects expected issue state.
- No unintended edits in runtime no-change-zone files.

---

## 8) Session-End Operational Routine

Recommended sequence:
1. Sync/handoff:
   ```bash
   make mem-handoff
   ```
2. Snapshot backup:
   ```bash
   make mem-snapshot
   ```
3. Commit snapshot file(s):
   ```bash
   git add docs/beads_snapshots/
   git commit -m "beads: snapshot issue state"
   ```

This is the project-standard approach for Beads UI continuity and recoverable issue-state history.

## Appendix — Inlined Referenced Docs

The following sections inline full text of the Markdown docs referenced by this runbook so an external agent can consume everything in one pass.

### Appendix 1: `docs/beads_docs_index.md`

~~~~markdown
# Beads Docs Index + Command Map (AGRc)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-02-28

This index defines the authoritative Beads documentation path for AGRc.

## Canonical Entry
- Team entrypoint: [START_HERE_BEADS.md](START_HERE_BEADS.md)

## Authority Map (High-Traffic Docs)
- [START_HERE_BEADS.md](START_HERE_BEADS.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Team operating entrypoint
- [beads_source_of_truth_migration_plan.md](beads_source_of_truth_migration_plan.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Phased migration strategy and gates
- [beads_cutover_decision_2026-03-01.md](beads_cutover_decision_2026-03-01.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Formal Phase 5 go-live decision for AGRc source-of-truth cutover
- [beads_devcontainer_setup.md](beads_devcontainer_setup.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Container-first setup and validation runbook
- [beads_container_handover.md](beads_container_handover.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Reopen/resume checklist and persistence steps
- [beads_setup_runlog.md](beads_setup_runlog.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Setup incident log and mitigation record
- [beads_template_hardening_defaults.md](beads_template_hardening_defaults.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Reusable Beads safety defaults (ignore/stash/recovery helpers)
- [02-review-checklist.md](02-review-checklist.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: PASS/NITS/FAIL peer-review protocol and template
- [beads_dispute_resolution_template.md](beads_dispute_resolution_template.md)
  - Lifecycle: Active
  - Authority: Yes
  - Purpose: Standard format for peer-review disagreement escalation and authority decisions
- [AI_Engineering](AI_Engineering)
  - Lifecycle: Legacy/Reference
  - Authority: No
  - Purpose: Historical context and prior prompt/playbook materials

## Command Map

### Session start
```bash
make dolt-start
export BD_ACTOR="agrc/copilot"
make mem-ready
```

Expected outcomes:
- `make dolt-start` reports successful Dolt reachability test.
- `make mem-ready` returns at least one ready/open issue view without command errors.
- `BD_ACTOR` is set before any `make mem-claim` call for audit traceability.

### Work lifecycle
```bash
make mem-show ISSUE=<id>
make mem-claim ISSUE=<id>
# implement/test
make mem-close ISSUE=<id>
make mem-handoff
```

Reproducibility checks:
- `make mem-show ISSUE=<id>` should render title/scope/notes for that issue.
- `make mem-claim ISSUE=<id>` should either claim successfully or explicitly report already-claimed owner.
- At implementation start, record `Execution Start Timestamp (UTC)` in issue notes before substantive changes.
- `make mem-handoff` should complete with `bd sync` + handoff confirmation line.

### Views
```bash
make mem-next
make mem-active
make mem-ops
```

### Visibility
```bash
make beads-ui
```
UI default: http://127.0.0.1:3002

### Rebuild-safe persistence
```bash
make mem-snapshot
git add docs/beads_snapshots/
git commit -m "beads: snapshot issue state"
```

### Recovery helper
```bash
make mem-recover
```

## Phase Boundary Reminder
- During Phase 1/2/3 execution, do not change auction runtime behavior.
- Keep no-change zones intact (`scripts/auctions/scheduler.ts`, `scripts/auctions/monitor.ts`, `scripts/auctions/reporter.ts`).
~~~~

### Appendix 2: `docs/beads_container_handover.md`

~~~~markdown
# Beads Container Handover

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-02-28

## Purpose
Use this file to resume Beads setup immediately after reopening in the devcontainer.

## Current State (2026-02-28)
- Phase 0 completed and recorded.
- Phase 1 docs/governance updates applied.
- Existing-repo Beads overlay applied using `Makefile.workteam` included by `Makefile`.
- Devcontainer scaffolding added for Beads-first workflow.
- Host-side validation failed as expected (`make` missing on Windows host shell).

## Files Already Prepared
- `.devcontainer/devcontainer.json`
- `.devcontainer/Dockerfile`
- `.devcontainer/postCreate.sh`
- `Makefile` (includes `Makefile.workteam`)
- `Makefile.workteam`
- `.beads/config.yaml`
- `.agent/copilot.md`
- `.agent/agent-x.md`
- `docs/beads_setup_runlog.md`
- `docs/beads_devcontainer_setup.md`

## Important Source Correction
- `workteam-beads-setup` references `withbeads/beads` in places.
- Active/working upstream confirmed as: `https://github.com/steveyegge/beads`
- Default install flow in post-create now uses steveyegge install script, with npm fallback.

## Resume Steps (Inside Devcontainer)
1. Verify tools:
   - `command -v git`
   - `command -v go`
   - `command -v make`
   - `command -v jq`
   - `command -v dolt`
   - `command -v bd`
2. If `bd` missing, install:
   - `curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash`
   - fallback: `npm install -g @beads/bd`
3. Verify bd:
   - `bd version`
   - `bd help`
4. Initialize repo state:
   - `bd init --prefix agrc`
5. Set actor for session:
   - `export BD_ACTOR="agrc/copilot"`
6. Start local Dolt server (standalone, no GT):
   - `make dolt-start`
7. Validate Beads command surface:
   - `make mem-ready`
   - `make mem-next`
   - `make mem-active`
   - `make mem-ops`
8. Persist state before rebuild/session end:
   - `make mem-snapshot`
   - `git add docs/beads_snapshots/`
   - `git commit -m "beads: snapshot issue state"`

## Persistent TODO (Update During Session)
Use this checklist as the rebuild-safe working memory. Update status before/after each action.

- [ ] Rebuild devcontainer image (after Dockerfile change adding `golang-go`)
- [ ] Verify tools: `command -v git go make jq dolt bd curl`
- [ ] Verify Beads CLI: `bd version` and `bd help`
- [ ] Initialize repo (if needed): `bd init --prefix agrc`
- [ ] Set actor: `export BD_ACTOR="agrc/copilot"`
- [ ] Start backend: `make dolt-start`
- [ ] Validate command surface: `make mem-ready`, `make mem-next`, `make mem-active`, `make mem-ops`
- [ ] Persist snapshot for git: `make mem-snapshot` then commit `docs/beads_snapshots/`
- [ ] Log outcomes/issues in `docs/beads_setup_runlog.md`

## Expected Validation Gate
Proceed only if:
- Existing project behavior still unchanged
- All four `make mem-*` commands execute without command errors
- `git status` remains clean except intended setup files

## Expected Devcontainer Build Issues (Do Not Assume Green First Try)

1. `postCreate` dependency failures (`npm install` errors)
   - Re-run manually: `bash .devcontainer/postCreate.sh`
   - If still failing, run `npm install` and capture exact error in setup log.

2. `bd` install script/download failures
   - Primary: `curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash`
   - Fallback: `npm install -g @beads/bd`
   - Verify path/version: `command -v bd` and `bd version`

3. Missing tools in container (`go`, `make`, `jq`, `dolt`, `curl`)
   - Rebuild container after Dockerfile edits.
   - Verify with `command -v go`, `command -v make`, `command -v jq`, `command -v dolt`, `command -v curl`.

4. PATH mismatch after install
   - Check effective binary: `command -v bd`
   - Restart shell in container and retry.

5. Beads init or state errors
   - Retry init: `bd init --prefix agrc`
   - If recovering state later: `bd init --from-jsonl --prefix agrc`

6. Make targets fail after successful install
   - Confirm include chain: `Makefile` includes `Makefile.workteam`
   - Re-run: `make mem-ready`, `make mem-next`, `make mem-active`, `make mem-ops`

Any of the above should be logged in `docs/beads_setup_runlog.md` with command, output, impact, and mitigation.

## If Validation Fails
- Log details in `docs/beads_setup_runlog.md`
- Do not modify runtime auction scripts
- Keep to Phase 1 scope (process overlay only)

## Pilot Seed Commands (After Validation)
```bash
bd create --type epic --title "pilot: validate AGRc multi-agent workflow" --description "Run one full lifecycle"
bd create --parent <epic-id> --title "pilot task 1" --description "Implement a small change"
bd create --parent <epic-id> --title "pilot task 2" --description "Peer review and finalize"
```

## Cross-References
- Team entrypoint: `docs/START_HERE_BEADS.md`
- Migration plan: `docs/beads_source_of_truth_migration_plan.md`
- Devcontainer runbook: `docs/beads_devcontainer_setup.md`
- Setup issue log: `docs/beads_setup_runlog.md`
~~~~

### Appendix 3: `docs/beads_setup_runlog.md`

~~~~markdown
# Beads Setup Run Log

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-02-28

## Date
- 2026-02-28

## Objective
Follow `workteam-beads-setup` in existing AGRc repo, devcontainer-first, with minimal workflow disruption.

## Actions Completed
- Cloned reference package to `.tmp/workteam-beads-setup`
- Applied Beads template artifacts:
  - `Makefile`
  - `.beads/config.yaml`
  - `.agent/copilot.md`
  - `.agent/agent-x.md`
- Added devcontainer files:
  - `.devcontainer/devcontainer.json`
  - `.devcontainer/Dockerfile`
  - `.devcontainer/postCreate.sh`

## Issues Encountered

### Issue 1 — Beads CLI upstream link mismatch in setup package
- Context: `workteam-beads-setup` README references `https://github.com/withbeads/beads`, which is not reachable.
- Checks performed:
  - `git ls-remote https://github.com/withbeads/beads.git` → not found
  - `git ls-remote https://github.com/steveyegge/beads.git` → reachable
- Resolution:
  - Adopted `https://github.com/steveyegge/beads` as the CLI source.
  - Updated `.devcontainer/postCreate.sh` default install to:
    - `curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash`
  - Added fallback install: `npm install -g @beads/bd`
  - Kept `BD_INSTALL_CMD` override for controlled/custom installs.

## Validation Status
- Pending full validation inside devcontainer:
  - `make mem-ready`
  - `make mem-next`
  - `make mem-active`
  - `make mem-ops`

### Issue 2 — Host validation toolchain mismatch (expected with devcontainer-first)
- Check performed: `make mem-ready` on Windows host shell.
- Result: `make` command not found.
- Impact: workflow validation cannot run reliably on host.
- Mitigation: proceed with devcontainer-first path where `make` and `jq` are installed via `.devcontainer/Dockerfile`.

### Issue 3 — Beads backend unavailable (`dolt` missing in container)
- Check performed inside devcontainer:
  - `bd init --prefix agrc`
  - `bd dolt show`
  - `bd dolt test`
- Result:
  - `bd init` failed with Dolt server connection error at `127.0.0.1:3307`.
  - `dolt` binary was missing from container PATH.
- Impact:
  - `make mem-ready` and full Beads validation blocked.
- Mitigation applied:
  - Updated `.devcontainer/Dockerfile` to install Dolt using official installer:
    - `curl -L https://github.com/dolthub/dolt/releases/latest/download/install.sh | bash`
  - Added `command -v dolt` check in `.devcontainer/postCreate.sh`.
  - Updated Beads handover/setup docs to include Dolt in prerequisite checks.
- Next action:
  - Rebuild devcontainer, then run:
    - `command -v dolt && dolt version`
    - `bd init --prefix agrc`
    - `make mem-ready`, `make mem-next`, `make mem-active`, `make mem-ops`

## Next Commands (inside devcontainer)
1. Reopen workspace in container.
2. Provide `BD_INSTALL_CMD` in container environment (once confirmed).
3. Run: `bash .devcontainer/postCreate.sh`
4. Run: `bd init --prefix agrc`
5. Run validation commands:
   - `make mem-ready`
   - `make mem-next`
   - `make mem-active`
   - `make mem-ops`

## Notes
- Existing project build/run paths were not replaced.
- Existing repo mode now uses `Makefile.workteam` + `include Makefile.workteam`.
- This implementation follows Phase 1 additive overlay behavior.

## Post-Rebuild Verification (2026-02-28)
- Verified binaries present in container:
  - `git`, `go`, `make`, `jq`, `dolt`, `bd`, `curl`, `node`, `npm`
- Verified versions:
  - `go version go1.19.8`
  - `dolt version 1.83.0`
  - `bd version 0.56.1`

### Issue 4 — Dolt server not running by default
- Initial result after rebuild:
  - `bd dolt show` reported server unreachable at `127.0.0.1:3307`.
  - `make mem-ready` failed due to backend connection refusal.
- Mitigation used (no GT):
  - Started standalone server:
    - `dolt sql-server --host 127.0.0.1 --port 3307 --data-dir .beads/dolt-data`

### Issue 5 — Existing Beads DB missing `issues` table
- Result:
  - `make mem-ready` failed with `table not found: issues`.
- Mitigation used:
  - Reinitialized Beads metadata/schema:
    - `bd init --force --prefix agrc`
  - Then reran:
    - `make mem-ready`
    - `make mem-next`
    - `make mem-active`
    - `make mem-ops`
- Final status:
  - All four `make mem-*` commands executed without command errors.

### Session actor
- Set for validation session:
  - `export BD_ACTOR="agrc/copilot"`
~~~~

### Appendix 4: `docs/beads_template_hardening_defaults.md`

~~~~markdown
# Beads Template Hardening Defaults (Reusable)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-03-01

Use these defaults when applying Beads in a new or existing repository.

## 1) Required .gitignore Defaults

Always include local runtime/state paths:

```gitignore
# Beads local runtime artifacts (do not commit)
.beads/dolt-data/
.beads/dolt-server.log
.beads/issues.jsonl

# Local temporary repositories/workspaces
.tmp/
```

## 2) Stash Safety Policy

- Do not use broad `git stash -u` when `.beads/` runtime files are present.
- Prefer scoped stash for code/docs paths only.
- Take a Beads snapshot before any risky cleanup/rebase:

```bash
make mem-snapshot
```

## 3) Normal vs Emergency Recovery Path

### Normal restart path (preferred)

```bash
make mem-recover
```

Expected outcome:
- Dolt reachable
- `mem-ready` runs
- Latest snapshot path and count comparison displayed

### Emergency path (state mismatch/empty queue)

1. Do not immediately run force re-init.
2. Confirm latest snapshot exists in `docs/beads_snapshots/`.
3. Compare live issue count vs snapshot count.
4. If mismatch persists, log incident in `docs/beads_setup_runlog.md` and follow project restore runbook.

## 4) Safe Helper Commands (Template)

```bash
# Recovery preflight + readiness + snapshot comparison
make mem-recover

# Session-end persistence
make mem-snapshot
git add docs/beads_snapshots/
git commit -m "beads: snapshot issue state"
```

## 5) Post-Recovery Verification Checklist

- `bd dolt test` succeeds
- `make mem-ready` returns expected queue
- `make mem-next`, `make mem-active`, `make mem-ops` run without command errors
- No unintended runtime script changes in no-change zones
~~~~
