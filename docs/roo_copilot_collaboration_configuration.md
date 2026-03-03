# Roo + Copilot Collaboration Configuration (AGRc)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-03-03

This document explains how [`Roo`](../.agent/roo.md) and [`GitHub Copilot`](../.agent/copilot.md) are configured to work together in this repository.

## 1) Coordination Backbone

Both agents are configured to use **Beads (`bd`) as the single task system**:
- Canonical rule set: [`AGENTS.md`](../AGENTS.md)
- Team entrypoint: [`docs/START_HERE_BEADS.md`](START_HERE_BEADS.md)
- Command map and authority index: [`docs/beads_docs_index.md`](beads_docs_index.md)

Core requirement: all work is tracked in Beads issues (no duplicate markdown TODO systems).

## 2) Shared Workflow Contract

Roo and Copilot both follow the same lifecycle contract:
1. `make mem-show ISSUE=<id>`
2. `make mem-claim ISSUE=<id>`
3. Implement scoped work
4. Record review evidence
5. Close only after merge/push
6. End session with `make mem-handoff`

References:
- Roo workflow: [` .agent/roo.md`](../.agent/roo.md)
- Copilot workflow: [`.agent/copilot.md`](../.agent/copilot.md)
- Make targets: [`Makefile.workteam`](../Makefile.workteam)

## 3) Role Split Between Roo and Copilot

### Roo (execution + QA focus)
Configured in [`.agent/roo.md`](../.agent/roo.md):
- Scoped implementation tasks
- Documentation/process artifact updates
- Peer review and QA on completed work
- Explicit actor identity: `export BD_ACTOR="agrc/roo"`

### Copilot (broader implementation focus)
Configured in [`.agent/copilot.md`](../.agent/copilot.md):
- Complex feature work and architecture changes
- Multi-file refactors
- Debugging and peer reviews

This creates a practical pattern: Copilot handles broad changes, Roo executes scoped/process work and reciprocal review.

## 4) Review Handshake (How They Verify Each Other)

Both agents are configured to use a common review protocol:
- Checklist: [`docs/02-review-checklist.md`](02-review-checklist.md)
- Output format: `PASS | NITS | FAIL` with severity tags (`high|med|low`)
- Required evidence links in review notes

If they disagree, escalation is standardized through:
- [`docs/beads_dispute_resolution_template.md`](beads_dispute_resolution_template.md)
- Decision authority: Rob Turvey (as documented in template and migration governance)

## 5) Governance and Safety Constraints

Shared constraints that govern both Roo and Copilot:
- Claim before execution
- Handoff record for ownership transfer
- Reviewer/user close policy
- Human approval for agent-generated planning artifacts

Primary governance references:
- [`docs/onboarding/project_rules.md`](onboarding/project_rules.md)
- [`docs/beads_source_of_truth_migration_plan.md`](beads_source_of_truth_migration_plan.md)

Process-only safety boundary used during migration/pilot phases:
- No runtime auction behavior changes in protected no-change zones:
  - `scripts/auctions/scheduler.ts`
  - `scripts/auctions/monitor.ts`
  - `scripts/auctions/reporter.ts`

## 6) Operational Tooling that Enables Collaboration

The collaboration is implemented through shared make targets in [`Makefile.workteam`](../Makefile.workteam):
- Issue visibility: `mem-ready`, `mem-show`, `mem-next`, `mem-active`, `mem-ops`
- Ownership and lifecycle: `mem-claim`, `mem-close`, `mem-handoff`
- Persistence/recovery support: `mem-snapshot`, `mem-recover`, `dolt-start`

This ensures both agents use the same commands, same status model, and same issue evidence trail.

## 7) Session Identity and Auditability

Auditability is enforced by setting actor identity before work:
- Roo: `export BD_ACTOR="agrc/roo"`
- Copilot: `export BD_ACTOR="agrc/copilot"`

Because both use Beads with actor attribution and issue notes, handoffs/reviews remain traceable across sessions.

## 8) Practical Day-to-Day Collaboration Pattern

Typical loop:
1. Copilot or user opens/scopes issue
2. Assigned agent claims and executes
3. Counterpart agent performs checklist-based review (PASS/NITS/FAIL)
4. Reviewer/user closes when criteria are met
5. Session handoff is synced with `make mem-handoff`

This is the configured mechanism for Roo + Copilot coordination in this project.

## Appendix — Inlined Reference Documents

The following sections inline the full current contents of the referenced Markdown files so this document can be consumed in one pass in external projects.

### Appendix A: `AGENTS.md`

~~~~markdown
# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

<!-- BEGIN BEADS INTEGRATION -->
## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Auto-syncs to JSONL for version control
- Agent-optimized: JSON output, ready work detection, discovered-from links
- Prevents duplicate tracking systems and confusion

### Quick Start

**Check for ready work:**

```bash
bd ready --json
```

**Create new issues:**

```bash
bd create "Issue title" --description="Detailed context" -t bug|feature|task -p 0-4 --json
bd create "Issue title" --description="What this issue is about" -p 1 --deps discovered-from:bd-123 --json
```

**Claim and update:**

```bash
bd update bd-42 --status in_progress --json
bd update bd-42 --priority 1 --json
```

**Complete work:**

```bash
bd close bd-42 --reason "Completed" --json
```

### Issue Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default, nice-to-have)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

### Workflow for AI Agents

1. **Check ready work**: `bd ready` shows unblocked issues
2. **Claim your task**: `bd update <id> --status in_progress`
3. **Work on it**: Implement, test, document
4. **Discover new work?** Create linked issue:
   - `bd create "Found bug" --description="Details about what was found" -p 1 --deps discovered-from:<parent-id>`
5. **Complete**: `bd close <id> --reason "Done"`

### Auto-Sync

bd automatically syncs with git:

- Exports to `.beads/issues.jsonl` after changes (5s debounce)
- Imports from JSONL when newer (e.g., after `git pull`)
- No manual export/import needed!

### Important Rules

- ✅ Use bd for ALL task tracking
- ✅ Always use `--json` flag for programmatic use
- ✅ Link discovered work with `discovered-from` dependencies
- ✅ Check `bd ready` before asking "what should I work on?"
- ❌ Do NOT create markdown TODO lists
- ❌ Do NOT use external issue trackers
- ❌ Do NOT duplicate tracking systems

For more details, see README.md and docs/QUICKSTART.md.

<!-- END BEADS INTEGRATION -->

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
~~~~

### Appendix A: `.agent/roo.md`

~~~~markdown
# Agent: Roo

## Role
- Scoped implementation tasks
- Documentation and process artifacts
- Peer review and QA on completed work

## Identity
- Use actor: `agrc/roo`
- Set each session: `export BD_ACTOR="agrc/roo"`

## Required Workflow
1. Read issue context with `make mem-show ISSUE=<id>`
2. Claim atomically via `make mem-claim ISSUE=<id>`
3. Implement only scoped changes
4. Commit using issue ID in message
5. Mark blocked if necessary and escalate with linked issue
6. Close only after merge/push with `make mem-close ISSUE=<id>`
7. End session with `make mem-handoff`

## Review Discipline
- Follow `docs/02-review-checklist.md` exactly
- Use PASS/NITS/FAIL with severity tags (high/med/low)
- Record evidence links and recommendation in Beads notes
- Escalate disagreements using `docs/beads_dispute_resolution_template.md`

## Rules
- Do not work unclaimed items
- Do not invent scope beyond issue acceptance criteria
- No runtime auction behavior changes during process-only phases
- Keep Beads status current throughout execution
- Recovery steps after restart/rebuild: `docs/beads_container_handover.md`
~~~~

### Appendix A: `.agent/copilot.md`

~~~~markdown
# Agent: GitHub Copilot

## Role
- Complex feature work and architectural changes
- Multi-file refactors
- Debugging and peer reviews

## Required Workflow
1. `make mem-show ISSUE=<id>`
2. `make mem-claim ISSUE=<id>`
3. Implement and commit with issue ID in message
4. If blocked: set `blocked` + create escalation issue
5. After merge/push: `make mem-close ISSUE=<id>`
6. At session boundary: `make mem-handoff`

## Review Discipline
- Follow `docs/02-review-checklist.md` exactly
- Use PASS/NITS/FAIL rubric
- Record severity for findings (high/med/low)

## Rules
- Never close issues yourself before merge/push
- Never skip claim step
- Beads issue state is source of truth
- Recovery steps after restart/rebuild: `docs/beads_container_handover.md`
~~~~

### Appendix A: `docs/START_HERE_BEADS.md`

~~~~markdown
# START HERE: Beads Workflow (AGRc Team)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-02-28

This is the active entrypoint for multi-agent delivery in AGRc.

## 1) Read First
- Docs index + command map: [beads_docs_index.md](beads_docs_index.md)
- Migration plan: [beads_source_of_truth_migration_plan.md](beads_source_of_truth_migration_plan.md)
- Phase 5 cutover decision (AGRc): [beads_cutover_decision_2026-03-01.md](beads_cutover_decision_2026-03-01.md)
- Phase 0 baseline evidence: [beads_phase0_baseline_freeze_2026-02-28.md](beads_phase0_baseline_freeze_2026-02-28.md)
- Devcontainer setup runbook: [beads_devcontainer_setup.md](beads_devcontainer_setup.md)
- Container handover (resume here after reopen): [beads_container_handover.md](beads_container_handover.md)

## 2) Follow Team Rules
- Governance and controls: [onboarding/project_rules.md](onboarding/project_rules.md)
- Peer review checklist: [02-review-checklist.md](02-review-checklist.md)
- Dispute resolution template: [beads_dispute_resolution_template.md](beads_dispute_resolution_template.md)
- Required controls:
  - Claim before execution
  - Handoff record for ownership transfer
  - Reviewer/user closes tasks
  - 1 human approval for agent-generated planning artifacts

## 3) Run Pilot Work Correctly
- Use this for each pilot task: [beads_pilot_tracking_template.md](beads_pilot_tracking_template.md)
- Pilot scope: 2–3 non-critical tasks
- Phase 1 safety: no runtime behavior changes

## 4) Framework Reference
- Workteam Beads setup: https://github.com/RobTurvey/workteam-beads-setup/

## 5) Team Rollout Intent
- AGRc adopts Beads first.
- Presale website team reuses the same governance + pilot model.
- Beads is now source of truth for AGRc (cutover approved 2026-03-01).
- Phase 6 website rollout remains deferred until remaining AGRc follow-ups complete.

## Legacy Notes
- Files under [AI_Engineering](AI_Engineering) are retained for context.
- Active workflow decisions are governed by Beads migration docs above.
~~~~

### Appendix A: `docs/beads_docs_index.md`

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

### Appendix A: `docs/02-review-checklist.md`

~~~~markdown
# 02 — Peer Review Checklist (Team Standard)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-02-28

Use this checklist exactly for all peer reviews between agents.

## Review Protocol

1. Confirm issue scope and acceptance criteria
2. Inspect changed files and commit messages
3. Run relevant checks (tests/lint/build where applicable)
4. Validate workflow discipline (claim, commit message issue ID, close timing)
5. Record findings with severity
6. Publish verdict (`PASS`, `NITS`, `FAIL`)
7. If disputed, escalate using `docs/beads_dispute_resolution_template.md`

## Required Review Comment Template

```text
Review: <PASS|NITS|FAIL>
Issue: <id>
Scope:
- <what was reviewed>

Checks:
- <check 1>
- <check 2>

Findings:
- [high|med|low] <finding>

Recommendation:
- <merge now / fix before merge / blocked pending decision>

Disputes:
- <none | summary + escalation issue>
```

## Verdict Rules

- **PASS** — No blocking issues; acceptable quality for merge
- **NITS** — Non-blocking improvements requested
- **FAIL** — Blocking issue(s) must be resolved before merge

## Severity Guidance

- **high** — correctness/security/data loss/workflow integrity risk
- **med** — significant quality/maintainability risk
- **low** — style/clarity/minor improvement

## PASS Criteria

All must be true:

- Implementation matches issue scope
- No known blocking defects introduced
- Workflow protocol followed
- Commit message includes issue ID
- Appropriate tests/checks performed or justified
~~~~

### Appendix A: `docs/beads_dispute_resolution_template.md`

~~~~markdown
# Beads Dispute Resolution Template (AGRc)

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-02-28

Use this template whenever two reviewers/agents disagree on PASS/NITS/FAIL, scope, risk severity, or close-readiness.

## When to use
- Reviewer verdicts conflict (e.g., PASS vs FAIL).
- Material disagreement on risk severity (`high`/`med`/`low`).
- Dispute over whether acceptance criteria are met.
- Disagreement on whether issue should be closed.

## Process (Short)
1. Both reviewers post findings on the same Beads issue.
2. One reviewer posts this dispute template in issue notes.
3. Tag Rob Turvey as decision authority in the note.
4. Freeze close action until decision is recorded.
5. Record final decision + rationale + required follow-ups.

## Template (Paste into Beads notes)
```text
Dispute: OPEN
Issue: <id>
Date: <YYYY-MM-DD>
Participants:
- Reviewer A: <name/actor>
- Reviewer B: <name/actor>
Decision authority: Rob Turvey

Summary:
- <1-3 bullets describing disagreement>

Reviewer A position:
- Verdict: <PASS|NITS|FAIL>
- Key findings:
  - [high|med|low] <finding>

Reviewer B position:
- Verdict: <PASS|NITS|FAIL>
- Key findings:
  - [high|med|low] <finding>

Evidence checked:
- <issue links, file links, test output, run logs>

Decision (Authority):
- Final verdict: <PASS|NITS|FAIL>
- Rationale:
  - <why this decision>
- Required actions:
  - <follow-up task(s) and owners>
- Close policy:
  - <close now | close after follow-up>

Dispute: RESOLVED
Resolved by: Rob Turvey
Resolved at: <timestamp>
```

## Guardrails
- Do not close disputed issues until decision is marked `RESOLVED`.
- Keep disagreements factual and evidence-based.
- If safety/runtime constraints are implicated, default to conservative decision.

## Related
- [START_HERE_BEADS.md](START_HERE_BEADS.md)
- [beads_source_of_truth_migration_plan.md](beads_source_of_truth_migration_plan.md)
- [beads_docs_index.md](beads_docs_index.md)
~~~~

### Appendix A: `docs/onboarding/project_rules.md`

~~~~markdown
# Project Rules (Persistent)

## Naming conventions
- Solidity contracts use `PascalCase` (e.g., `AgrcPresaleAllocationNFT`).
- Role constants are `UPPER_SNAKE_CASE` with `_ROLE` suffix.
- Tier IDs are **1–4** and map to Bronze/Silver/Gold/Platinum.

## Error handling
- Revert messages in Solidity must use `"AGRC: ..."` prefix.
- Validate tier IDs before state changes.
- Fail fast on missing config or secrets.

## Logging rules
- Use Pino for structured logs in Node scripts.
- Include `tier`, `orderHash`, and key identifiers in logs.
- Log external API failures with status and body.

## Dependency boundaries
- OpenSea interactions stay in `scripts/auctions/`.
- No auction logic inside on‑chain contracts.
- Use OpenZeppelin for AccessControl, ERC‑1155, Pausable.

## External systems & approvals
- Publishing orders to OpenSea requires **explicit human approval**.
- Do not run scripts that place real listings without user confirmation.
- Any TGE redemption actions are **manual and gated**.

## Beads governance (source-of-truth migration)
- Beads is the active multi-agent coordination framework during migration.
- Every task must be **claimed before execution**.
- Every ownership transfer requires a **handoff record**.
- Only a **reviewer or user** can close a task.
- Agent-generated planning artifacts require **at least 1 human approval** before merge.
- During Phase 1, Beads adoption is process-only: no runtime behavior changes.

## Testing expectations
- New Solidity logic requires tests in `test/`.
- Tests must be deterministic; avoid live RPC calls.
- Cover role gating, caps, URI freeze, and pause behavior.

## Performance and safety
- Use `BigInt` for wei calculations in scripts.
- Avoid duplicate listings: honor `lastOrderHash` guards.
- English auctions require `OPENSEA_AUCTION_ZONE`.
~~~~

### Appendix A: `docs/beads_source_of_truth_migration_plan.md`

~~~~markdown
# Beads Source-of-Truth Migration Plan

Lifecycle: Active
Authority: Source of truth (Yes)
Last validated: 2026-02-28

## Executive Summary

This plan establishes Beads as the authoritative multi-agent workflow framework for AGRc once pilot validation is complete. The adoption starts as a process-only overlay to avoid runtime risk, runs container-first in a devcontainer lane for reproducibility, and uses minimal documentation tidy-up to reduce disruption. 

After AGRc pilot success, this same operating model is rolled out to the presale website team so all teams share one memory and coordination framework. The objective is durable agent memory, complete traceability, and predictable multi-week/month multi-agent execution.

## Primary External Reference

- Workteam Beads setup repository: https://github.com/RobTurvey/workteam-beads-setup/

## Scope and Constraints

### In Scope (Phase 1)
- Process-only Beads overlay.
- Devcontainer-first workflow for Beads tooling.
- Minimal safety tidy-up (docs index/labels/runbook clarity only).
- Governance policy for claim/close, human approval, and artifact quality.
- Pilot on 2–3 non-critical tasks.

### Out of Scope (Phase 1)
- Runtime behavior changes in auction execution.
- Broad refactors, renames, or architecture rewrites.
- Production container behavior changes.
- Cross-repo automation coupling before pilot results.

## Decision Record

- Adoption scope: **Process-only overlay first**.
- Primary execution environment: **Devcontainer only** initially.
- Tidy depth: **Minimal safety tidy**.
- Governance owner: **Rob Turvey is the named Beads Workflow Authority**.
- PR policy: **Agent-generated planning artifacts require 1 human approval**.
- Convention: **Claim required before work; only reviewer/user closes**.
- Environment/install model: **Home network with full internet access; direct installs allowed; pinned versions still required**.
- Rollout gate: **Pilot first (2–3 non-critical tasks)**.
- Rollback policy: **Single-phase revert boundary (one PR/commit range per phase)**.
- Success target: **No runtime drift + measurable process gains (20–30% pilot cycle-time improvement target)**.

## Operating Principles

1. Runtime safety is non-negotiable during migration.
2. Beads records become canonical only after pilot gate pass.
3. Every meaningful agent action must be traceable from claim to close.
4. Governance and auditability take precedence over speed.
5. Legacy/exploratory docs are retained for context but not operational authority.

## Phased Migration Plan

## Phase 0 — Baseline Freeze and Safety Guardrails

### Goal
Capture a known-good operating baseline and lock migration boundaries.

### Actions
- Record baseline references and operating commands from:
  - `README.md`
  - `BASELINE.md`
  - `docker-compose.yml`
  - `Dockerfile`
  - `docs/auction_runner_howto.md`
  - `docs/auction_runtime_flow.md`
- Declare explicit Phase 1 no-change zones:
  - `scripts/auctions/scheduler.ts`
  - `scripts/auctions/monitor.ts`
  - `scripts/auctions/reporter.ts`
  - production runtime behavior in container paths.
- Define phase rollback boundaries in advance.

### Exit Criteria
- Baseline documented and acknowledged.
- No-change zones published and agreed.
- Rollback strategy documented.

### Evidence
- Phase 0 baseline record: `docs/beads_phase0_baseline_freeze_2026-02-28.md`

---

## Phase 1 — Governance + Beads Process Overlay

### Goal
Install Beads as the active process framework without touching runtime behavior.

### Actions
- Publish Beads governance policy in onboarding path and root docs.
- Define mandatory lifecycle rules:
  - Claim required before starting work.
  - Handoff record required for ownership transfer.
  - Reviewer/user-only close policy.
  - One human approval on agent-generated planning artifacts.
- Define required process artifacts per task:
  - Objective/acceptance criteria.
  - Decision notes.
  - Execution log.
  - Close summary.

### Exit Criteria
- Governance policy visible and enforced in active team flow.
- Team can run Beads lifecycle end-to-end for a task.
- No runtime files changed.

---

## Phase 2 — Devcontainer-First Enablement

### Goal
Standardize execution environment for Beads operations.
Setup 'beads-ui' for visibility of issues and tasks  

### Actions
- Run Beads tooling in devcontainer lane first.
- Pin required tool versions and document install path.
- `beads-ui` is not optional it is a must as it gives visibility to the human on all project acitiviites 
- Keep production runtime container paths unchanged.

### Exit Criteria
- Contributors can reliably run Beads workflow inside devcontainer.
- Toolchain setup is reproducible and compliant.
- No impact to production image/runtime behavior.

---

## Phase 3 — Minimal Safety Tidy (Documentation)

### Goal
Remove ambiguity by clarifying doc authority without broad rewrites.

### Actions
- Add a canonical Beads migration/operations entrypoint.
- Mark exploratory or legacy agent docs as reference-only where needed.
- Add lightweight lifecycle/authority metadata to high-traffic docs:
  - Lifecycle: Active / Legacy / Historical
  - Authority: Source of truth (Yes/No)
  - Last validated date
- Add docs index/command map for discoverability.

### Exit Criteria
- Team has one unambiguous Beads-first entrypoint.
- Legacy docs remain available but clearly non-authoritative.
- No disruptive refactors.

---

## Phase 4 — Pilot Validation (2–3 Non-Critical Tasks)

### Goal
Prove Beads value and safety with measurable outcomes.

### Actions
- Select 2–3 non-critical tasks.
- Execute full lifecycle with claim/handoff/review/close controls.
- Collect metrics versus baseline:
  - Cycle time
  - Handoff completeness
  - Traceability quality
  - Context loss incidents

### Exit Criteria
- No production runtime behavior change.
- End-to-end lifecycle visibility demonstrated.
- Pilot shows meaningful process improvement (target: 20–30% cycle-time improvement trend).

---

## Phase 5 — Cutover: Beads as Source of Truth

### Goal
Formally declare Beads records as canonical project memory.

### Actions
- Update onboarding and root navigation so Beads path is primary.
- Freeze competing process docs from active use (retain as historical reference).
- Record formal go-live decision and ownership.

### Exit Criteria
- Beads is the default operating framework across AGRc team workflows.
- Team executes and audits through Beads records.

---

## Phase 6 — Cross-Team Rollout (Presale Website Team)

### Goal
Apply the same model to the presale website team.

### Actions
- Package AGRc governance/checklist templates for reuse.
- Run same phased sequence in website repo:
  - baseline
  - governance
  - pilot
  - cutover
- Hold cross-team cadence review to align standards.

### Exit Criteria
- Both teams use the same Beads lifecycle conventions.
- Cross-team handoffs are traceable and consistent.

## Governance Model

## Roles
- **Beads Workflow Authority (Owner):** Final process authority, exception handling, policy updates.
- **Contributors/Agents:** Must claim before execution; must maintain complete logs.
- **Reviewers:** Validate quality/compliance; only reviewers/users can close.

## Mandatory Controls
- Claim before work.
- Explicit acceptance criteria before execution.
- Human approval for agent-generated planning artifacts.
- Required close summary documenting outcome and residual risks.

## Security and Compliance Controls

- Use pinned tool versions.
- Home network allows direct internet installs for required tooling.
- Prefer trusted official package sources and lock versions in setup docs.
- Keep optional UI components gated.
- Maintain audit-friendly logs and decision records.

## Success Criteria

Adoption is successful only if all are true:

1. No production runtime behavior change.
2. Faster coordination/traceability with visible issue lifecycle end-to-end.
3. Reduced context loss across agent handoffs.
4. Measurable pilot improvement target met or trending (20–30% faster cycle on pilot tasks).

## Risk Register and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Toolchain/environment drift | Medium | Devcontainer-first standardization; pinned versions |
| Legacy doc confusion | High | Canonical Beads entrypoint + lifecycle/authority labels |
| Unreviewed agent artifacts | High | Mandatory human approval policy |
| Scope creep into runtime changes | High | Explicit no-change zones and per-phase gates |
| Tooling supply-chain risk | Medium | Trusted official sources + pinned versions + documented install commands |

## Rollback Strategy

- Each phase ships in an isolated PR/commit range.
- If a phase fails gate criteria, revert that phase only.
- Revert must not touch successful prior phases unless explicitly approved.

## Phased Checklist

| Phase | Objective | Scope | Exit Gate | Rollback Boundary |
|---|---|---|---|---|
| 0 | Baseline freeze | Evidence and guardrails only | Baseline + no-change zones confirmed | Single revert of baseline PR |
| 1 | Governance overlay | Policy + lifecycle controls | Rules active; no runtime edits | Single revert of governance PR |
| 2 | Devcontainer enablement | Beads tooling lane only | Reproducible setup confirmed | Single revert of tooling PR |
| 3 | Minimal tidy | Docs authority clarity | Canonical Beads entrypoint live | Single revert of docs PR |
| 4 | Pilot | 2–3 non-critical tasks | Safety + KPI gates pass | Revert pilot PR range |
| 5 | Source-of-truth cutover | Beads primary operating path | Formal go-live recorded | Revert cutover PR |
| 6 | Website team rollout | Same framework in website repo | Cross-team standard in place | Revert per website phase |

## Next Actions (Immediate)

1. Beads workflow owner confirmed: Rob Turvey.
2. Select pilot tasks (2–3 non-critical).
3. Publish governance snippet in onboarding path.
4. Create pilot tracking template for metrics and close summaries.
5. Schedule pilot review checkpoint and go/no-go decision date.
~~~~
