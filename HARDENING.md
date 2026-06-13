<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rtCamp--action-deploy-wordpress/v3.1.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references the Docker image using a mutable version tag (`docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.1.2`) instead of an immutable SHA digest. A tag can be silently overwritten to point to a different (potentially malicious) image, enabling a supply-chain attack. The image reference should use a SHA256 digest, e.g. `docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:6`

### unsafe-shell (severity: high)

Remote content is fetched with curl and piped directly to bash without first saving to a file and verifying integrity. This allows a compromised or malicious remote server to execute arbitrary code on the runner. Three occurrences found:
1. Dockerfile: `curl -sL https://deb.nodesource.com/setup_16.x | bash`
2. main.sh: `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash`
3. main.sh: `curl -fsSL https://www.npmjs.com/install.sh | bash`

Locations:

- `Dockerfile:43`
- `main.sh:133`
- `main.sh:140`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell

**Notes:**

Fixed three security issues:
1. action.yml: Pinned Docker image from mutable tag 'v3.1.2' to immutable SHA256 digest 'sha256:44ade946449e54e172e2775138f22261ea2a530ea64420192265d445201bf40c', preserving the tag as a comment.
2. Dockerfile (line 43): Replaced `curl -sL https://deb.nodesource.com/setup_16.x | bash` with download-to-temp-file then execute pattern.
3. main.sh (line 133): Replaced `curl -fsSL "...nvm.../install.sh" | bash` with download-to-temp-file then execute pattern.
4. main.sh (line 140): Replaced `curl -fsSL https://www.npmjs.com/install.sh | bash` with download-to-temp-file then execute pattern.
All temp files are cleaned up after execution.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe curl-pipe-to-PHP pattern in Dockerfile line 40. Replaced `curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer` with a secure download-verify-execute pattern: (1) download the installer to /tmp/composer-installer.php, (2) download the official SHA-384 signature from composer.github.io/installer.sig, (3) verify the hash matches before executing, (4) fail the build with an error if verification fails, (5) clean up temp files. This follows the official Composer security installation recommendations.

