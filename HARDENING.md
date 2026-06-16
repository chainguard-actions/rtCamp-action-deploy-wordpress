<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rtCamp--action-deploy-wordpress/v3.4.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml references the Docker image by a mutable version tag (v3.4.1) rather than an immutable SHA digest. If the image at ghcr.io/rtcamp/action-deploy-wordpress:v3.4.1 is ever overwritten or compromised, all workflows using this action will silently execute the new (potentially malicious) image. The runs.image field should use a SHA digest, e.g. docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest>.

Locations:

- `action.yml:6`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag reference 'docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.1' with the immutable SHA256 digest 'docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:7f470e9faecee45df54ae61de30d5afbc5d02127931d38135aa4b56036a6a344' in action.yml line 6. The original tag is preserved as a comment outside the YAML string.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed all four unsafe curl-pipe-to-shell patterns:

1. main.sh ~line 122: NVM installer — replaced `curl ... | bash` with download to /tmp/nvm_install.sh, execute separately, then delete.

2. main.sh ~line 130: npm installer — replaced `curl ... | bash` with download to /tmp/npm_install.sh, execute separately, then delete.

3. Dockerfile line 41: Composer installer — replaced `curl ... | php` with download to /tmp/composer-setup.php, SHA-384 checksum verification against https://composer.github.io/installer.sig, execute with php, then delete.

4. Dockerfile line 46: NodeSource setup — replaced `curl ... | bash` with download to /tmp/nodesource_setup.sh, execute separately, then delete.

All scripts are now downloaded to a temporary file before execution, eliminating the risk of arbitrary code execution from tampered remote content piped directly to a shell interpreter.

