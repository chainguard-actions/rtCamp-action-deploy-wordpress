<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rtCamp--action-deploy-wordpress/v3.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable version tag (`v3.2.0`) instead of an immutable SHA256 digest. This means the image could be silently replaced with a different (potentially malicious) image without any change to the action.yml file, creating a supply-chain attack risk. The failing reference is: `image: 'docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.2.0'`. It should be pinned to a full SHA256 digest, e.g. `image: 'ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest> # v3.2.0'`.

Locations:

- `action.yml:6`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker container image in action.yml from the mutable tag 'v3.2.0' to the immutable digest 'sha256:dd33d7096c39479faa03f07170ae48bc531199f494fc430382bda2aa985bce09'. The docker:// scheme and :v3.2.0 tag are preserved inline alongside the digest, following the required format: docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.2.0@sha256:dd33d7096c39479faa03f07170ae48bc531199f494fc430382bda2aa985bce09

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed all three unsafe 'curl | bash' patterns:
1. Dockerfile line 41: Changed `curl -sL https://deb.nodesource.com/setup_16.x | bash` to download the script to /tmp/nodesource_setup.sh first, then execute it with `bash /tmp/nodesource_setup.sh`, then remove the temp file.
2. main.sh line 148: Changed NVM installer from `curl -fsSL ".../$NVM_LATEST_VER/install.sh" | bash` to download to /tmp/nvm_install.sh, execute with bash, then remove.
3. main.sh line 154: Changed npm installer from `curl -fsSL https://www.npmjs.com/install.sh | bash` to download to /tmp/npm_install.sh, execute with bash, then remove.
In all cases the script is now saved to a temporary file before execution, eliminating the direct pipe-to-shell risk. Note: checksum verification was not added since no official checksums are published by these installers, but the download-then-execute pattern eliminates the TOCTOU race condition inherent in piping.

