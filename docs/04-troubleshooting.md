# 04 — Troubleshooting

## `bd` command not found

Cause: Beads CLI is not installed or not on PATH.

Fix:

- Install Beads CLI per your platform
- Re-open terminal/session
- Verify with `command -v bd`

## `make mem-*` commands fail

Cause: Missing `jq` or Makefile not copied correctly.

Fix:

```bash
command -v jq
cat Makefile
```

Ensure targets exist and dependencies are installed.

## Claim conflicts

Cause: Another agent already claimed the issue.

Fix:

```bash
make mem-show ISSUE=<id>
```

Coordinate reassignment or pick another item from `make mem-next`.

## Issue won't close

Cause: Status or sync mismatch.

Fix:

```bash
bd sync
make mem-show ISSUE=<id>
make mem-close ISSUE=<id>
```

If still blocked, check Beads config and actor permissions.

## Too many noisy untracked files

Cause: Missing ignore patterns for generated artifacts.

Fix:

- Apply `templates/gitignore.template`
- Ensure `node_modules/` and Beads ephemeral paths are ignored

## Team sees inconsistent queue views

Cause: Different local Makefile versions.

Fix:

- Standardize from `templates/Makefile.template`
- Commit Makefile updates centrally
- Ask team to pull latest and rerun commands
