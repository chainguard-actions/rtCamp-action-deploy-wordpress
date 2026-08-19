<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rtCamp--action-deploy-wordpress/v3.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Unpinned action/image references found. In build.yaml, `docker/setup-qemu-action@v3` uses a mutable tag instead of a full 40-character commit SHA, making it vulnerable to supply-chain attacks. In action.yml, the Docker image `docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.0` uses a mutable tag instead of a SHA digest.

Locations:

- `.github/workflows/build.yaml:17`
- `action.yml:6`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are directly interpolated inside run: shell command strings. In build.yaml line 19, `${{ github.actor }}` is interpolated directly in a docker login command. In build.yaml lines 21-22, `${{ github.repository }}` and `${{ github.ref_name }}` are interpolated directly inside a run: block to construct IMAGE_TAG. These values flow through YAML template substitution before the shell sees them, enabling script injection if an attacker can influence them (e.g. via a malicious repository name or ref).

Locations:

- `.github/workflows/build.yaml:19`
- `.github/workflows/build.yaml:21`

### github-env-injection (severity: high)

Unsanitized values are written to $GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. (1) In build.yaml, IMAGE_TAG is constructed from `${{ github.repository }}` and `${{ github.ref_name }}` (untrusted inputs) and then written to $GITHUB_ENV via `echo "IMAGE_TAG_LOWER=$IMAGE_TAG_LOWER" >> $GITHUB_ENV` with no newline stripping — a newline in the repository name or ref could inject arbitrary environment variables. (2) In release.yaml, `$GITHUB_REF_NAME` (an inherited process env var set by the calling workflow/runner) is written directly to $GITHUB_ENV via `echo "release_name=${GITHUB_REF_NAME/v/Version }" >> $GITHUB_ENV` without sanitization.

Locations:

- `.github/workflows/build.yaml:23`
- `.github/workflows/release.yaml:17`

### unsafe-shell (severity: high)

Remote scripts are fetched and piped directly to bash without first downloading and verifying them. (1) In main.sh, the nvm install script is fetched from raw.githubusercontent.com and piped to bash: `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash`. The version `$NVM_LATEST_VER` is itself fetched dynamically from the GitHub API, making this a double-fetch TOCTOU risk. (2) In main.sh, the npm install script is piped directly to bash: `curl -fsSL https://www.npmjs.com/install.sh | bash`.

Locations:

- `main.sh:130`
- `main.sh:135`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, unsafe-shell

**Notes:**

Fixed all four findings: (1) Pinned docker/setup-qemu-action@v3 to full SHA c7c53464625b32c7a7e944ae62b3e17d2b600130 in build.yaml; pinned ghcr.io/rtcamp/action-deploy-wordpress:v3.4.0 to SHA digest sha256:927df7c782d83bd982ee462ad55afa1b1fdfb88a952686b7b17cf8d4742b27d0 in action.yml (preserving docker:// scheme and tag). (2) Moved ${{ github.actor }}, ${{ github.repository }}, and ${{ github.ref_name }} out of run: shell strings into env: blocks in build.yaml to prevent script injection. (3) Added printf/tr -d newline sanitization before writing to $GITHUB_ENV in both build.yaml and release.yaml. (4) In main.sh, replaced both curl-pipe-to-bash patterns (nvm and npm install scripts) with download-to-tempfile-then-execute pattern to prevent unsafe shell execution of remote scripts.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script-injection finding in .github/workflows/build.yaml at the 'Login to GitHub Container Registry' step. Moved `${{ secrets.GITHUB_TOKEN }}` from the run: shell command into the step's env: block as `GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}`, and updated the shell command to reference it as `$GH_TOKEN` instead of the inline expression. This ensures the token value flows through an environment variable rather than being interpolated directly into the shell command string via YAML template substitution.

### Iteration 3

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe curl-pipe-to-bash pattern in Dockerfile line 40. Changed `curl -sL https://deb.nodesource.com/setup_24.x | bash` to download the NodeSource setup script to `/tmp/nodesource_setup.sh` first, then execute it separately with `bash /tmp/nodesource_setup.sh`, and clean up the temp file. This prevents a compromised remote server from executing arbitrary code directly in the build environment via the pipe.

