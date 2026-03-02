# AGRc Migration Learnings (Reusable for New Repos)

This note captures practical lessons from a real Beads source-of-truth migration and can be reused when onboarding a new repository.

## Key Lessons

- Run migration as a **process overlay first** before runtime changes.
- Use **devcontainer-first** setup to avoid host-toolchain inconsistency.
- Install prerequisites at **image build time** in Dockerfile, not only in post-create/manual steps.
- Treat Beads runtime state as local/ephemeral and snapshot issue state for recovery.
- Avoid force re-init unless explicitly approved and logged.

## Build-Time Prerequisites (Required)

Ensure the devcontainer build installs:
- `node`, `npm`
- `make`, `jq`, `go`, `curl`
- `dolt`
- `bd`

Use post-create scripts for verification and fallback only.

## Offline `bd` Binary Option (When Network Install Is Blocked)

If your environment blocks install scripts/npm for `bd`, use the prepackaged binary:
- Binary path: `binaries/linux-arm64/bd`
- Checksum file: `binaries/linux-arm64/bd.sha256`

Install steps:

```bash
sha256sum -c binaries/linux-arm64/bd.sha256
install -m 0755 binaries/linux-arm64/bd /usr/local/bin/bd
bd version
```

Notes:
- This bundle is architecture-specific (`linux-arm64`).
- Keep `dolt` installed via normal package/build process; this offline bundle is only for `bd`.

## Setup Corrections That Reduced Failure Rate

1. Prefer reachable install source for bd:
   - `curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash`
   - fallback: `npm install -g @beads/bd`
2. Ensure Dolt is installed and running before Beads commands.
3. Standardize startup checks:
   - `make dolt-start`
   - `bd dolt test`
   - `make mem-ready`

## State Safety Defaults

Add to `.gitignore`:

```gitignore
.beads/dolt-data/
.beads/dolt-server.log
.beads/issues.jsonl
.tmp/
```

Do not use broad `git stash -u` if Beads runtime paths are present.

## Recommended Recovery Flow

1. Normal restart:
   - `make dolt-start`
   - `bd dolt test`
   - `make mem-ready`
2. If state mismatch:
   - Compare live issue count vs latest snapshot.
   - Do not force-init first.
3. Last resort:
   - approved force-init + immediate rehydration + ID/status verification.

## Governance Controls That Helped

- Claim-before-work policy.
- Reviewer/user closes tasks.
- PASS/NITS/FAIL review protocol.
- Human approval for agent-generated planning artifacts.

## Metrics Quality Improvement

Capture explicit `Execution Start Timestamp (UTC)` to separate:
- queue latency (`opened` → `execution start`)
- implementation time (`execution start` → `close`)
- end-to-end cycle (`opened` → `close`)

## Suggested Template Enhancements

- Add Dolt runtime preflight checks to setup docs.
- Add `.gitignore` runtime defaults.
- Add `mem-recover` helper target.
- Add execution-start timestamp to pilot tracking template.
