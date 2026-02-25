# workteam-beads-setup: Multi-Agent Environment Distribution Package

Enable your entire team to work with multi-agent environments using Beads as a shared memory layer for coordination, peer review, and issue tracking.

## What's Included

This package provides everything your Copilots need to implement a scalable, multi-agent workflow:

- **Canonical Beads workflow contract** — deterministic issue lifecycle with atomic claiming and synchronization
- **Peer review standardization** — structured review checklist with consistent severity and recommendation templates
- **Queue management guidance** — delivery vs operational triage views to keep work organized
- **Beads UI dashboard support** — optional local web UI for issue visibility and triage
- **Configuration templates** — reusable Makefile, .beads/config.yaml, agent personas, and .gitignore
- **Step-by-step setup guide** — for Copilots to initialize Beads and validate the environment
- **Real-world examples** — sample epic and issue snapshots showing the workflow in action

## Quick Start

1. **Clone this package** (or copy to your team's git repo):
   ```bash
   git clone <repo-url> workteam-beads-setup
   cd workteam-beads-setup
   ```

2. **Follow SETUP.md** — Your Copilot will walk through initialization

3. **Enterprise networks** — run mirror checks in [docs/07-enterprise-mirrors.md](./docs/07-enterprise-mirrors.md) before installs

4. **For a brand-new project repo** — run the fast bootstrap in [docs/05-new-repo-bootstrap.md](./docs/05-new-repo-bootstrap.md)

5. **(Optional) Enable the Beads UI** — install Node/npm and run `make beads-ui`

6. **Test with a pilot epic** — Create a small 2-3 task epic to validate the workflow

7. **Dispatch to team** — Share the repo link; each team member clones and follows SETUP.md

## Integration Modes

- **New project repo**: copy templates directly (`Makefile.template` → `Makefile`), run `bd init`, then start workflow.
- **Existing project repo**: keep existing build/deploy Makefile, copy workflow template as `Makefile.workteam`, and add `include Makefile.workteam` to your existing `Makefile`.

## Team Setup Options

- **Option A (Recommended): Devcontainer-first**
   - Standardize the team on one container image for consistent versions of `bd`, `node`, `make`, and `jq`.
   - Lowest support overhead and minimal machine-to-machine drift.
- **Option B: Local-native install**
   - Install prerequisites directly on each laptop.
   - More flexible, but each machine must keep versions aligned.

Work laptop path convention:

- Create project repos under `C:/Dev/<project-name>/`
- Example: `C:/Dev/sasmigration/`
- Keep this package repo under `C:/Dev/workteam-beads-setup/`

## Features

- ✅ **Beads CLI** — Issue lifecycle management (open, in_progress, blocked, closed)
- ✅ **Makefile workflow** — Canonical memory commands (`make mem-ready`, `make mem-claim`, `make mem-close`, `make mem-handoff`)
- ✅ **Queue views** — Separate delivery (`mem-next`, `mem-active`) from operational work (`mem-ops`)
- ✅ **Beads UI** — Optional dashboard launch via `make beads-ui` on port 3002
- ✅ **Peer review** — Standardized checklist with severity tags (high/med/low) and pass/nit/fail decision framework
- ✅ **Multi-agent dispatch** — Clear task assignment and bilateral review between agents
- ✅ **Git discipline** — Issue IDs in commit messages, atomic sync at session boundaries
- ✅ **Blocking protocol** — When work is blocked, escalate via linked issues and `bd update --status blocked`

## Prerequisites

- **Beads CLI** — `bd` command available on PATH (install from https://github.com/withbeads/beads)
- **Git** — Version control for source code and Beads state
- **Make** — For workflow orchestration
- **jq** — JSON query tool (for Makefile queue filtering)
- **Node.js + npm** — Required only if using Beads UI (`make beads-ui`)
- **A code editor with agent support** — E.g., VS Code with GitHub Copilot or compatible agent extension

For full setup instructions, see [SETUP.md](./SETUP.md).

## Documentation

- [**SETUP.md**](./SETUP.md) — Installation, initialization, and validation
- [**docs/01-beads-workflow.md**](./docs/01-beads-workflow.md) — Canonical issue lifecycle and memory contract
- [**docs/02-review-checklist.md**](./docs/02-review-checklist.md) — Peer review protocol and review template
- [**docs/03-queue-management.md**](./docs/03-queue-management.md) — Understanding mem-next, mem-active, and mem-ops views
- [**docs/04-troubleshooting.md**](./docs/04-troubleshooting.md) — Common issues and resolution steps
- [**docs/05-new-repo-bootstrap.md**](./docs/05-new-repo-bootstrap.md) — Fast start for a new project in a separate repo
- [**docs/06-setup-options.md**](./docs/06-setup-options.md) — Devcontainer-first vs local-native rollout guidance
- [**docs/07-enterprise-mirrors.md**](./docs/07-enterprise-mirrors.md) — Internal mirrors/proxy checks for Copilot setup runs
- [**templates/**](./templates/) — Configuration file templates for .beads/config.yaml, Makefile, agent personas, etc.

## How It Works

### The Workflow

1. **Pick work** from `make mem-ready` (strict claimable queue)
2. **Review context** via `make mem-show ISSUE=td-abc`
3. **Claim atomically** with `make mem-claim ISSUE=td-abc`
4. **Implement and test** with any tool (Copilot, Roo Code, manual development)
5. **Commit with issue ID** in message: `git commit -m "td-abc: build feature X"`
6. **Push and merge** to main branch
7. **Close in Beads** with `make mem-close ISSUE=td-abc`
8. **Sync session** with `make mem-handoff` (includes `bd sync`)

### Multi-Agent Coordination

- **Epic decomposition** — Break high-level features into tracked child tasks
- **Task assignment** — Assign tasks to specific agents (Copilot A, Roo Code, Copilot B)
- **Bilateral peer review** — Agent A reviews Agent B's completed work; Agent B reviews Agent A's
- **Dispute resolution** — Team lead (human) resolves disagreements via Beads comments
- **Async safety** — All state in Beads; agents can work out-of-sync without conflicts

## Integration with Your Workflow

This package is designed to be **minimal overhead** and **tool-agnostic**:

- Use with **GitHub Copilot** (VS Code), **Roo Code**, **Claude**, or manual development
- Works with any **Git-based repo** (GitHub, GitLab, internal Git server)
- Integrates with existing **build/test/deploy pipelines** (Makefile targets are extensible)
- Compatible with **DevContainers** and distributed team environments

## Example: Pilot Epic

See [examples/sample-epic.md](./examples/sample-epic.md) for a real-world trace of a 3-task epic from creation through closure.

## Support & Next Steps

- **Question about workflow?** → See [docs/01-beads-workflow.md](./docs/01-beads-workflow.md)
- **Stuck during setup?** → See [docs/04-troubleshooting.md](./docs/04-troubleshooting.md)
- **Customizing for your team?** → See [templates/](./templates/) for starting points
- **Running a pilot?** → Start with an epic of 2-3 tasks and run through the full lifecycle once

## License

This package is provided as-is for team use. Adapt and customize as needed for your environment.

---

**Ready?** Start with [SETUP.md](./SETUP.md) to initialize your environment.
