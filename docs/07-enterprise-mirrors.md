# 07 — Enterprise Mirrors and Proxy Setup (Copilot Runbook)

Use this guide in corporate environments where public package registries are blocked or restricted.

## Goal

Ensure Copilot configures approved internal mirrors/proxies before running setup commands.

## What Copilot Should Check First

Run these checks:

```bash
echo "HTTP_PROXY=$HTTP_PROXY"
echo "HTTPS_PROXY=$HTTPS_PROXY"
echo "NO_PROXY=$NO_PROXY"
npm config get registry
```

If values are empty but your org requires mirrors, configure them before any install.

## Common Mirror Targets

- apt package mirror (for devcontainer/docker builds)
- npm registry mirror
- Go module proxy (if using Go tools)
- Internal git host/CA trust settings (if required)

## Example Configuration Steps

### npm mirror

```bash
npm config set registry https://<internal-npm-mirror>/
npm config get registry
```

### Proxy variables (session scope)

```bash
export HTTP_PROXY=http://<proxy-host>:<port>
export HTTPS_PROXY=http://<proxy-host>:<port>
export NO_PROXY=localhost,127.0.0.1,<internal-domains>
```

### Go proxy (optional)

```bash
go env -w GOPROXY=https://<internal-go-proxy>,direct
```

## Devcontainer Note

If using devcontainers, mirror settings should be baked into:

- Dockerfile package source configuration
- build args/env for proxy values
- any postCreate bootstrap scripts

Do this once centrally so all team members get the same behavior.

## Validation Steps

Before running the full setup flow, Copilot should validate:

- `npm view beads-ui version` resolves successfully
- package installs do not attempt blocked public endpoints
- `make mem-ready` works after setup
- optional: `make beads-ui` starts dashboard

## Failure Pattern to Watch

Symptoms of missing mirror configuration:

- install commands hanging or timing out
- TLS/certificate errors against public registries
- 403/401 from unapproved external package hosts

When seen, pause setup and apply mirror/proxy configuration first.
