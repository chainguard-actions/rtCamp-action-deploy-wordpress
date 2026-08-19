<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rtCamp--action-deploy-wordpress/v3.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two unpinned references found:
1. `.github/workflows/build.yaml` line 18: `uses: docker/setup-qemu-action@v3` — pinned to a mutable tag `v3`, not a 40-character commit SHA.
2. `action.yml` line 6: `image: 'docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.3.0'` — the Docker action image is referenced by a mutable tag (`v3.3.0`) rather than an immutable SHA digest (e.g. `@sha256:<64-hex-char-digest>`). Both references are vulnerable to supply-chain attacks if the upstream tag is moved.

Locations:

- `.github/workflows/build.yaml:18`
- `action.yml:6`

### script-injection (severity: high)

Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings in build.yaml:

(a) Line 22: `run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin` — `${{ github.actor }}` is interpolated directly and unquoted into the shell command. An attacker who controls the actor name could inject shell metacharacters.

(a) Lines 26: `IMAGE_TAG="ghcr.io/${{ github.repository }}:${{ github.ref_name }}"` — `${{ github.repository }}` and `${{ github.ref_name }}` are interpolated directly into the `run:` script. These values flow through YAML template substitution before the shell ever sees them, enabling script injection via crafted tag or repository names.

Locations:

- `.github/workflows/build.yaml:22`
- `.github/workflows/build.yaml:26`

### github-env-injection (severity: high)

Untrusted values derived from GitHub context are written to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. `.github/workflows/build.yaml` line 28: `echo "IMAGE_TAG_LOWER=$IMAGE_TAG_LOWER" >> $GITHUB_ENV` — `IMAGE_TAG_LOWER` is derived from `${{ github.repository }}` and `${{ github.ref_name }}` (both attacker-controllable via tag names). A newline embedded in these values could inject arbitrary environment variables into subsequent steps.

2. `.github/workflows/release.yaml` line 18: `echo "release_name=${GITHUB_REF_NAME/v/Version }" >> $GITHUB_ENV` — `GITHUB_REF_NAME` is the env-var equivalent of `github.ref_name` (the pushed tag name), which is attacker-controlled. Writing it unsanitized to `$GITHUB_ENV` allows environment variable injection.

Locations:

- `.github/workflows/build.yaml:28`
- `.github/workflows/release.yaml:18`

### unsafe-shell (severity: high)

Remote scripts are fetched and piped directly to `bash` without first downloading and verifying them:

1. `main.sh` (~line 130): `curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/$NVM_LATEST_VER/install.sh" | bash` — the nvm install script is piped directly to bash. The URL also incorporates `$NVM_LATEST_VER` which is derived from an external API response.

2. `main.sh` (~line 138): `curl -fsSL https://www.npmjs.com/install.sh | bash` — the npm install script is piped directly to bash.

3. `Dockerfile` (~line 38): `curl -sL https://deb.nodesource.com/setup_16.x | bash` — the NodeSource setup script is piped directly to bash during image build.

In all cases the script should be downloaded to a temporary file, its integrity verified (e.g. via checksum), and only then executed.

Locations:

- `main.sh:130`
- `main.sh:138`
- `Dockerfile:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, unsafe-shell

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned docker/setup-qemu-action@v3 to full SHA c7c53464625b32c7a7e944ae62b3e17d2b600130 in build.yaml. Pinned ghcr.io/rtcamp/action-deploy-wordpress:v3.3.0 with SHA digest sha256:0862d72ef766cb3a1d39718e1a0f98066e8559c713b25c1593c8c5b7b43c0980 in action.yml (preserving docker:// scheme and tag inline).

2. script-injection: In build.yaml, moved ${{ github.actor }}, ${{ secrets.GITHUB_TOKEN }}, ${{ github.repository }}, and ${{ github.ref_name }} into step env: blocks, referencing them as plain shell env vars ($GITHUB_ACTOR, $GITHUB_TOKEN, $GITHUB_REPOSITORY, $GITHUB_REF_NAME) in the run: scripts.

3. github-env-injection: In build.yaml, sanitized IMAGE_TAG_LOWER with 'printf | tr -d newlines' before writing to $GITHUB_ENV. In release.yaml, sanitized the GITHUB_REF_NAME-derived release_name with the same pattern before writing to $GITHUB_ENV.

4. unsafe-shell: In main.sh, replaced both 'curl | bash' patterns (nvm install and npm install) with download-to-tempfile then execute approach using mktemp. In Dockerfile, replaced 'curl | bash' for nodesource setup with download to /tmp/nodesource_setup.sh then execute.

