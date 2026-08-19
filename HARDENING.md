<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rtCamp--action-deploy-wordpress/v3.1.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable tag (`v3.1.2`) instead of an immutable SHA digest. This means the image could be silently replaced with a different (potentially malicious) version without any change to the action definition, creating a supply-chain attack risk. The image reference `docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.1.2` should be pinned to a full SHA256 digest, e.g. `docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest> # v3.1.2`.

Locations:

- `action.yml:6`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker container image in action.yml from `docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.1.2` to `docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.1.2@sha256:44ade946449e54e172e2775138f22261ea2a530ea64420192265d445201bf40c`. The `docker://` scheme and the `:v3.1.2` tag are preserved inline alongside the digest for readability and correctness.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed all four curl-to-shell patterns by downloading scripts to temp files before executing them:

1. **Dockerfile (composer installer)**: Changed `curl -sS https://getcomposer.org/installer | php` to download to `/tmp/composer-installer.php` first, then run `php /tmp/composer-installer.php`, then remove the temp file.

2. **Dockerfile (NodeSource setup)**: Changed `curl -sL https://deb.nodesource.com/setup_16.x | bash` to download to `/tmp/nodesource_setup.sh` first, then run `bash /tmp/nodesource_setup.sh`, then remove the temp file.

3. **main.sh (NVM install)**: Changed `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash` to download to `/tmp/nvm_install.sh` first, then run `bash /tmp/nvm_install.sh`, then remove the temp file.

4. **main.sh (npm install)**: Changed `curl -fsSL https://www.npmjs.com/install.sh | bash` to download to `/tmp/npm_install.sh` first, then run `bash /tmp/npm_install.sh`, then remove the temp file.

All other logic in both files was preserved unchanged.

