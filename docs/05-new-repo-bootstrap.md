# 05 — New Repo Bootstrap (Project Isolation)

Use this when starting Beads in a completely new project repository (example: `sasmigration`).

Work laptop path convention:

- Package repo at `C:/Dev/workteam-beads-setup/`
- New project at `C:/Dev/sasmigration/`

## Key Principle

- One project repo = one Beads issue universe.
- Different repos never share issue IDs/history unless you intentionally migrate/sync them.
- Local Dolt DB is runtime state; `.beads/issues.jsonl` in git is the durable team history.

## Quick Start (5 commands)

From your new project repo root:

```bash
git init -b main
mkdir -p .beads .agent docs
cp templates/Makefile.template Makefile
make project-bootstrap PREFIX=sa
make mem-ready
```

Notes:

- Set `PREFIX` to a short project prefix (`sa` for sasmigration, `web`, `api`, etc.).
- If your repo already has a Makefile, use `Makefile.workteam` + `include Makefile.workteam` instead.

## Existing Repo Pattern

If the project already has build targets:

```bash
cp templates/Makefile.template Makefile.workteam
echo "\ninclude Makefile.workteam" >> Makefile
make project-bootstrap PREFIX=sa
```

## First Day Workflow

```bash
make mem-ready
bd create --type epic --title "pilot: <project> workflow validation" --description "Run one end-to-end cycle"
make mem-next
```

## Recovery After Rebuild/New Machine

If local DB is empty but `.beads/issues.jsonl` exists:

```bash
bd init --from-jsonl --prefix sa
make mem-ready
```

This restores live DB state from git-tracked issue history.
