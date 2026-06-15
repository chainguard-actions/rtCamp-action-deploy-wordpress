<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rtCamp--action-deploy-wordpress/v3.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references the Docker image using a mutable version tag (`docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.0`) instead of an immutable SHA digest. A tag can be silently overwritten to point to a different (potentially malicious) image, enabling a supply-chain attack. The image reference should use a SHA256 digest, e.g. `docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest>`

Locations:

- `action.yml:6`

### unsafe-shell (severity: high)

Remote content is piped directly to a shell/interpreter without first downloading and verifying the script. This allows a compromised or malicious remote server to execute arbitrary code on the runner.

- Dockerfile line ~38: `curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer` — Composer installer piped directly to PHP.
- Dockerfile line ~42: `curl -sL https://deb.nodesource.com/setup_24.x | bash` — NodeSource setup script piped directly to bash.
- main.sh line ~115: `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash` — NVM installer piped directly to bash.
- main.sh line ~120: `curl -fsSL https://www.npmjs.com/install.sh | bash` — npm installer piped directly to bash.

In all cases the script should be downloaded to a temporary file, its integrity verified (e.g. via a checksum), and only then executed.

Locations:

- `Dockerfile:38`
- `Dockerfile:42`
- `main.sh:115`
- `main.sh:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell

**Notes:**

Fixed all 4 findings:
1. action.yml: Pinned Docker image from mutable tag 'v3.4.0' to immutable SHA256 digest 'sha256:927df7c782d83bd982ee462ad55afa1b1fdfb88a952686b7b17cf8d4742b27d0', preserving the tag as a comment.
2. Dockerfile line ~38: Composer installer now downloaded to /tmp/composer-setup.php, its SHA384 checksum verified against https://composer.github.io/installer.sig, then executed — download aborts if checksum mismatches.
3. Dockerfile line ~42: NodeSource setup script now downloaded to /tmp/nodesource_setup.sh and executed separately (not piped to bash).
4. main.sh line ~115: NVM installer now downloaded to /tmp/nvm_install.sh and executed separately (not piped to bash).
5. main.sh line ~120: npm installer now downloaded to /tmp/npm_install.sh and executed separately (not piped to bash).

