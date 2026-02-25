# SETUP.md — Workteam Multi-Agent Beads Environment

This guide is written so a Copilot agent can execute it end-to-end on a team member's laptop.

## Goal

Establish a reproducible, team-wide multi-agent workflow with:

- Shared issue memory in Beads
- Canonical command surface (`make mem-*`)
- Standardized peer reviews
- Safe git and session handoff discipline

## Workspace Location Convention (Work Laptops)

Use `C:/Dev/` as the standard root for team work.

Examples:

- Package repo: `C:/Dev/workteam-beads-setup/`
- New project repo: `C:/Dev/sasmigration/`

## Setup Option Selection

- **Option A: Devcontainer-first (recommended)**
	- Use repo-provided devcontainer config and run all commands inside the container.
- **Option B: Local-native**
	- Install required tools on host machine and run commands directly in terminal.

See `docs/06-setup-options.md` for full trade-offs and rollout guidance.

## Enterprise Mirror Check (Do this before installs)

If your organization requires internal mirrors/proxies, Copilot should check and configure them first.

Copilot runbook:

1. Check existing environment variables: `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`, `NPM_CONFIG_REGISTRY`
2. Confirm approved internal endpoints with your team/platform docs
3. Configure package sources before installing tools:
	- apt mirror (Linux/devcontainer image build)
	- npm registry mirror (Node packages)
	- Go module proxy (if Go tools are installed)
4. Verify mirror reachability with a dry-run install or metadata query
5. Only then run normal setup commands from this guide

Reference: `docs/07-enterprise-mirrors.md`

## 0) Prerequisites Check

Run:

```bash
command -v bd || echo "Missing: beads CLI"
command -v git || echo "Missing: git"
command -v make || echo "Missing: make"
command -v jq || echo "Missing: jq"
command -v npm || echo "Missing: npm (required for Beads UI)"
```

If any are missing, install them before continuing.

## 1) Initialize Repository

If starting fresh:

```bash
git init
git checkout -b main
```

If using an existing project, ensure you're at repo root and on your primary branch.

Choose integration mode:

- **New repo mode**: copy workflow template directly to `Makefile`
- **Existing repo mode**: keep your current `Makefile`, copy workflow template to `Makefile.workteam`, then add:

```makefile
include Makefile.workteam
```

Fast path for new repos (after template copy):

```bash
make project-bootstrap PREFIX=sa
```

This initializes Beads and creates the standard local structure for the repo.

## 2) Create Required Structure

```bash
mkdir -p .beads .agent docs
```

Copy templates from this package:

- `templates/beads-config.yaml.template` → `.beads/config.yaml`
- `templates/Makefile.template` → `Makefile` (new repo mode)
- `templates/Makefile.template` → `Makefile.workteam` (existing repo mode)
- `templates/agent-copilot.md.template` → `.agent/copilot.md`
- `templates/agent-x.md.template` → `.agent/agent-x.md`
- `templates/gitignore.template` → `.gitignore`

Optional:

- `templates/package.json.template` → `package.json`

If you want the Beads web UI, also run:

```bash
cp templates/package.json.template package.json
npm install
```

Initialize Beads database in this repo:

```bash
bd init
```

## 3) Configure Beads Identity and Baseline

Set actor identity per agent session:

```bash
export BD_ACTOR="<rig>/<agent>"
```

Examples:

- `teamA/copilot`
- `teamA/roo`
- `teamA/dev1`

Verify Beads config is readable:

```bash
bd config get create.require-description
bd config get validation.on-create
bd config get sync.mode
```

Expected values (from template):

- `true`
- `warn`
- `git-portable`

## 4) Validate Makefile Commands

Run:

```bash
make mem-ready
make mem-next
make mem-active
make mem-ops
```

If these run without command errors, command surface is valid.

Optional UI validation:

```bash
make beads-ui
```

Then open http://127.0.0.1:3002 and confirm the Beads dashboard loads.

## 5) Seed Pilot Epic (Validation Run)

Create an epic and 2-3 tasks:

```bash
bd create --type epic --title "pilot: validate team multi-agent workflow" --description "Run one full lifecycle from creation to closure"
bd create --parent <epic-id> --title "pilot task 1" --description "Implement a small change"
bd create --parent <epic-id> --title "pilot task 2" --description "Peer review and finalize"
```

Claim and execute:

```bash
make mem-show ISSUE=<task-id>
make mem-claim ISSUE=<task-id>
# do work
git add .
git commit -m "<task-id>: complete pilot task"
```

Close after merge/push:

```bash
make mem-close ISSUE=<task-id>
make mem-handoff
```

## 6) Peer Review Process

Use `docs/02-review-checklist.md` exactly.

Minimum review output requirements:

- Verdict: `PASS`, `NITS`, or `FAIL`
- Scope reviewed
- Checks performed
- Findings with severity (`high`, `med`, `low`)
- Recommendation and required follow-ups

## 7) Team Operating Rules (Required)

1. **Always claim before coding** (`make mem-claim`)
2. **Always include issue ID in commit message**
3. **Never close before merge/push**
4. **If blocked, set status `blocked` and create escalation issue**
5. **At session boundary, always run `make mem-handoff`**

## 8) Troubleshooting Quick Checks

### `make mem-next` returns nothing

- No open claimable tasks exist
- Run `make mem-active` and check for work in progress
- Create new child tasks under an open epic

### Claim fails

- Another agent may already hold the claim
- Run `make mem-show ISSUE=<id>` and inspect assignee/status

### Sync or state mismatch

```bash
bd sync
git status
```

If needed, reconcile local/remote branch state first, then retry.

## 9) Completion Criteria

Setup is complete when:

- All required commands run successfully
- One pilot epic is executed end-to-end
- At least one bilateral peer review is posted using checklist template
- `git status` is clean
- `make mem-handoff` succeeds at session boundary

If using UI:

- `make beads-ui` starts dashboard successfully on port 3002

---

If all criteria pass, this laptop is ready for team multi-agent delivery.
