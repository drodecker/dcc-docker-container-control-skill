---
name: dcc-docker-container-control
description: Operate Docker Compose deployments on a known remote Docker server with docker-container-control. Use for dcc list, status, dry-run, deploy, restart, or recreate requests.
---

# DCC Docker Container Control

Use the bundled `scripts/docker-container-control` on the target Docker server to discover and operate Compose deployments stored under `/opt`.

## Required target server

Resolve the Docker server from the user's request, current thread, or authoritative environment documentation. If the Docker server is unknown or ambiguous, ask the user which server to use before copying the script or running any command. Do not guess.

Use the configured SSH skill/helper for remote execution. Do not expose credentials.

## Ensure the command exists

On the selected server, check:

```bash
test -x /opt/docker-container-control && sudo /opt/docker-container-control list
```

If `/opt/docker-container-control` is absent or not executable:

1. Copy the bundled `scripts/docker-container-control` to a temporary remote path.
2. Validate it with `python3 -m py_compile`.
3. Install it as `/opt/docker-container-control`, owned by root and executable (mode 755).
4. Verify with `sudo /opt/docker-container-control list`.

Do not overwrite a differing existing script without first comparing it and telling the user what would change.

## Command mapping

- `dcc list` → `sudo /opt/docker-container-control list`
- `dcc status <name>` → `sudo /opt/docker-container-control status <name>`
- `dcc dry-run <name>` → `sudo /opt/docker-container-control dry-run <name>`
- `dcc deploy <name>` → `sudo /opt/docker-container-control deploy <name>`
- `dcc restart <name>` → `sudo /opt/docker-container-control restart <name>`
- `dcc recreate <name>` → `sudo /opt/docker-container-control recreate <name>`

Example: `dcc deploy website_app` runs:

```bash
sudo /opt/docker-container-control deploy website_app
```

This selects by folder-style name, Compose project name, or running container name. For Git checkouts it requires a clean tracked worktree, fetches/prunes origin, fast-forward pulls the current branch, pulls Compose images, runs `compose up -d --build`, waits for health/running state, and prints status.

## Operating rules

- Treat `list`, `status`, and `dry-run` as read-only.
- `deploy`, `restart`, and `recreate` change running services. Execute them only when the user requests that action.
- Prefer `dry-run` when the deployment name may be unclear or the user asks what will happen.
- Never substitute a fuzzy deployment name. If selection fails or matches multiple deployments, show the detected names and ask the user to choose.
- Preserve complete command output and report final Compose status.
- If health does not become ready within 120 seconds, report failure and printed status; do not claim success.
- If a local SSH wrapper is interrupted, inspect the remote process and verify whether it stopped before reporting termination.
- Do not run Docker cleanup, prune, remove, down, Git reset, force pull, or branch changes.

## Script behavior

The controller discovers deployments from Docker Compose labels and Compose files no deeper than two levels under `OPT_ROOT` (default `/opt`). Supported filenames are `docker-compose.clawdydaapps.yml`, `docker-compose.yml`, `compose.yml`, and `compose.yaml`. It uses the first discovered Compose config for operations.

