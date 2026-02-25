# 06 — Team Setup Options (Devcontainer vs Local-native)

This guide helps you choose how team members should run Beads workflows on work laptops.

## Recommended Default

Use **Devcontainer-first** for most users.

Why:

- Same toolchain versions for everyone
- Lower support burden
- Less breakage after updates/restarts
- Easier onboarding for new team members

## Option A — Devcontainer-first

### Best for

- Teams prioritizing consistency and speed of onboarding
- Projects with multiple agents and strict workflow discipline

### Requirements

- Docker Desktop (or org-approved container runtime)
- VS Code Dev Containers extension

### Typical flow

1. Clone repo under `C:/Dev/<project>/`
2. Open in VS Code
3. Run **Dev Containers: Reopen in Container**
4. Execute normal setup commands inside container

### Notes

- Keep host setup minimal
- All critical tools (`bd`, `node`, `make`, `jq`) come from container image

## Option B — Local-native

### Best for

- Teams that cannot use containers due to policy/performance restrictions

### Requirements

Install and maintain these on each laptop:

- `git`
- `make`
- `jq`
- `bd` (Beads CLI)
- Node.js 22+

### Typical flow

1. Clone repo under `C:/Dev/<project>/`
2. Open terminal in repo root
3. Run setup directly on host

### Notes

- Version drift risk is higher
- Add periodic version checks to team rituals

## Rollout Strategy

- Start with Option A as team default
- Allow Option B as exception path
- Track exceptions in onboarding notes

## Validation Checklist (Both Options)

- `make mem-ready` works
- `make mem-next` works
- `make mem-active` works
- `make mem-ops` works
- Optional UI: `make beads-ui` and dashboard opens

## Path Convention

Use `C:/Dev/` for all work repos:

- Package repo: `C:/Dev/workteam-beads-setup/`
- Project repo: `C:/Dev/sasmigration/`

This keeps team instructions uniform and easy to support.
