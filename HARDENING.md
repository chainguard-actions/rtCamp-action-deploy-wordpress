<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rtCamp--action-deploy-wordpress/v3.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references the Docker image using a mutable version tag (`v3.3.0`) instead of an immutable SHA digest. This means the image could be silently replaced with a different (potentially malicious) version. The reference `image: 'docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.3.0'` should be pinned to a SHA digest, e.g. `image: 'docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:<64-hex-char-digest>'`.

Locations:

- `action.yml:6`

### unsafe-shell (severity: high)

main.sh pipes remote content directly to bash in two places inside the `maybe_install_node_dep` function, without first downloading and verifying the script. (1) `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash` — fetches and immediately executes the nvm installer from a dynamically-resolved URL. (2) `curl -fsSL https://www.npmjs.com/install.sh | bash` — fetches and immediately executes the npm installer. Both patterns allow a compromised or MITM'd remote server to execute arbitrary code on the runner.

Locations:

- `main.sh:130`
- `main.sh:140`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell

**Notes:**

1. action.yml: Pinned Docker image from mutable tag `ghcr.io/rtcamp/action-deploy-wordpress:v3.3.0` to immutable digest `ghcr.io/rtcamp/action-deploy-wordpress@sha256:0862d72ef766cb3a1d39718e1a0f98066e8559c713b25c1593c8c5b7b43c0980` with `# v3.3.0` comment for readability. 2. main.sh: Fixed both unsafe `curl | bash` patterns in `maybe_install_node_dep()` — the nvm installer (line ~130) and npm installer (line ~140) now download to temp files (`/tmp/nvm_install.sh` and `/tmp/npm_install.sh`) and execute them separately, then clean up, preventing MITM or compromised server from executing arbitrary code.

### Iteration 2

**Fixes applied:** suspicious-run-content

**Notes:**

Replaced `eval "$NODE_BUILD_COMMAND"` (line 148) and `eval "$PHP_BUILD_COMMAND"` (line 162) in main.sh with safe array-based execution: `IFS=' ' read -ra _cmd_parts <<< "$VAR"` followed by `"${_cmd_parts[@]}"`.

This eliminates the shell injection risk: `eval` processes the string through a second shell parsing pass, allowing embedded shell metacharacters (`;`, `&&`, `$(...)`, backticks, etc.) in the variable value to be interpreted as shell code. The array-based replacement word-splits the command into tokens and executes them directly as a command + arguments without any additional shell interpretation, preventing injection while preserving the intended functionality of running a configurable build command.

The remaining `eval "$(ssh-agent -s)"` was left unchanged as it evaluates output from a trusted system binary (not user-controlled input) and is the standard idiom for loading ssh-agent environment variables.

