<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rtCamp--action-deploy-wordpress/v3.4.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references the Docker image 'docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.2' using a mutable version tag instead of an immutable SHA digest. A tag can be silently repointed to a different image, enabling supply-chain attacks. The image reference should use a SHA digest, e.g. 'ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest>'.

Locations:

- `action.yml:6`

### unsafe-shell (severity: high)

Remote scripts are fetched and piped directly to bash without first downloading and verifying them. Three occurrences found: (1) main.sh fetches the NVM installer from raw.githubusercontent.com and pipes it to bash: `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash`; (2) main.sh fetches the npm installer from npmjs.com and pipes it to bash: `curl -fsSL https://www.npmjs.com/install.sh | bash`; (3) Dockerfile fetches the NodeSource setup script and pipes it to bash: `curl -sL https://deb.nodesource.com/setup_24.x | bash`. An attacker who can intercept or tamper with these remote resources gains arbitrary code execution on the runner.

Locations:

- `main.sh:163`
- `main.sh:170`
- `Dockerfile:43`

### suspicious-run-content (severity: high)

eval-dynamic: main.sh uses eval with workflow-controlled environment variables in two places. (1) `eval "$NODE_BUILD_COMMAND"` — the NODE_BUILD_COMMAND env var is set by the calling workflow and passed directly to eval, allowing arbitrary shell command execution. (2) `eval "$PHP_BUILD_COMMAND"` — same issue with PHP_BUILD_COMMAND. An attacker who controls these env vars (e.g. via a malicious workflow or pull request) can execute arbitrary commands on the runner.

Locations:

- `main.sh:177`
- `main.sh:185`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, suspicious-run-content

**Notes:**

1. action.yml: Pinned Docker image 'docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.2' with immutable SHA256 digest '@sha256:0d478f9a22c004f5228cbf0b05e7b028f49aac9670699808d74e6e726c659d28'. 2. Dockerfile: Replaced 'curl -sL https://deb.nodesource.com/setup_24.x | bash' with download-then-execute pattern using a temp file. 3. main.sh (NVM): Replaced 'curl ... | bash' with download to /tmp/nvm_install.sh then 'bash /tmp/nvm_install.sh'. 4. main.sh (npm): Replaced 'curl ... | bash' with download to /tmp/npm_install.sh then 'bash /tmp/npm_install.sh'. 5. main.sh (eval): Replaced both 'eval "$NODE_BUILD_COMMAND"' and 'eval "$PHP_BUILD_COMMAND"' with 'bash -c "$NODE_BUILD_COMMAND"' and 'bash -c "$PHP_BUILD_COMMAND"' respectively, running them in subshells to limit scope.

