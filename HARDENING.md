<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rtCamp--action-deploy-wordpress/v3.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable version tag instead of an immutable SHA digest. The reference `docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.2.0` uses the tag `v3.2.0`, which can be silently replaced by a different image, enabling supply-chain attacks. It should be pinned to a SHA digest, e.g. `docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest>`

Locations:

- `action.yml:6`

### unsafe-shell (severity: high)

Remote content is piped directly to bash without first downloading to a file. This means the script executed is not verified before running. Three occurrences found:
1. Dockerfile: `curl -sL https://deb.nodesource.com/setup_16.x | bash` — NodeSource setup script piped directly to bash.
2. main.sh (maybe_install_node_dep): `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash` — NVM installer piped directly to bash, with the version determined dynamically from the GitHub API.
3. main.sh (maybe_install_node_dep): `curl -fsSL https://www.npmjs.com/install.sh | bash` — npm installer piped directly to bash.

Locations:

- `Dockerfile:44`
- `main.sh:116`
- `main.sh:123`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell

**Notes:**

Fixed 3 security issues across 3 files:

1. action.yml: Pinned Docker image from mutable tag `v3.2.0` to immutable SHA256 digest `sha256:dd33d7096c39479faa03f07170ae48bc531199f494fc430382bda2aa985bce09`, preserving the tag as a comment.

2. Dockerfile: (a) Pinned ubuntu base image from `ubuntu:24.04` to `ubuntu@sha256:786a8b558f7be160c6c8c4a54f9a57274f3b4fb1491cf65146521ae77ff1dc54`. (b) Replaced `curl -sL https://deb.nodesource.com/setup_16.x | bash` with download-to-file-then-execute pattern using `/tmp/nodesource_setup.sh`.

3. main.sh: (a) Replaced NVM installer `curl | bash` with download to `/tmp/nvm_install.sh` then `bash /tmp/nvm_install.sh` then cleanup. (b) Replaced npm installer `curl | bash` with download to `/tmp/npm_install.sh` then `bash /tmp/npm_install.sh` then cleanup.

