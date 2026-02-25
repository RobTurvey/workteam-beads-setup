# 01 — Beads-First Workflow (Team Standard)

Use Beads as the shared source of truth for all delivery work, independent of coding tool.

## Canonical Lifecycle (Required)

1. `make mem-ready` (or explicit assignment) to select work
2. `make mem-show ISSUE=<id>` to review context and acceptance criteria
3. `make mem-claim ISSUE=<id>` to atomically claim work
4. Implement and commit with issue ID in commit message
5. If blocked: set status `blocked` and create linked escalation issue
6. After merge/push: `make mem-close ISSUE=<id>`
7. At session boundary: `make mem-handoff` (must include sync)

## Non-Negotiable Rules

- Do not skip claim
- Do not close before merge/push
- Do not end session without handoff/sync
- Beads issue state is the source of truth (not chat memory)

## Baseline Beads Configuration

Recommended defaults:

- `create.require-description: true`
- `validation.on-create: warn`
- `validation.on-sync: warn`
- `sync.mode: git-portable`
- `sync.branch: ""`

Actor identity convention:

- Use `BD_ACTOR=<rig>/<agent>`
- Example: `team-rig/copilot`

## Delivery vs Operations Triage

Use separate views to avoid operational noise burying delivery work:

- `make mem-next` — strict claimable delivery queue (open, non-epic, non-ops)
- `make mem-active` — active delivery context (in_progress + epics, non-ops)
- `make mem-ops` — operational/housekeeping queue

## Issue Type Discipline

- `epic` — parent outcome with child decomposition
- `feature` — user-visible capability
- `bug` — incorrect behavior/regression
- `task` — implementation/documentation/test/refactor unit
- `chore` — maintenance and support work

## Dependency Discipline

- `blocks` — hard execution dependency
- `discovered-from` — lineage/traceability
- `related` — informational reference

Use `blocks` only when upstream completion is truly required.

## Definition of Done (Issue)

An issue is done only when:

- Acceptance criteria met
- Code merged/pushed
- Review comments resolved
- `make mem-close ISSUE=<id>` executed
- Session ended with `make mem-handoff`
