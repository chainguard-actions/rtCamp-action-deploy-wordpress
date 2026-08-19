<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rtCamp--action-deploy-wordpress/v3.4.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image referenced by a mutable tag ('docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.1') instead of a SHA digest. This means the image can be silently replaced with a different version, enabling supply-chain attacks. Additionally, build.yaml uses 'docker/setup-qemu-action@v3' (a mutable tag, not a 40-character commit SHA), which is also vulnerable to supply-chain substitution.

Locations:

- `action.yml:6`
- `.github/workflows/build.yaml:18`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In build.yaml line 21, '${{ github.actor }}' is interpolated unquoted directly into the shell command 'docker login ghcr.io -u ${{ github.actor }} --password-stdin'. In build.yaml line 24, '${{ github.repository }}' and '${{ github.ref_name }}' are interpolated directly into a shell variable assignment. These values flow through YAML template substitution before the shell sees them, allowing an attacker-controlled value (e.g. a tag or repository name containing shell metacharacters) to inject arbitrary commands.

Locations:

- `.github/workflows/build.yaml:21`
- `.github/workflows/build.yaml:24`

### github-env-injection (severity: high)

Unsanitized values derived from GitHub-controlled inputs are written to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). (1) In build.yaml line 25, IMAGE_TAG_LOWER is derived from '${{ github.repository }}' and '${{ github.ref_name }}' and written directly to $GITHUB_ENV via 'echo "IMAGE_TAG_LOWER=$IMAGE_TAG_LOWER" >> $GITHUB_ENV'. A newline embedded in the tag or repository name could inject additional environment variables. (2) In release.yaml line 17, the shell expansion '${GITHUB_REF_NAME/v/Version }' (a GitHub-controlled env var) is written directly to $GITHUB_ENV via 'echo "release_name=${GITHUB_REF_NAME/v/Version }" >> $GITHUB_ENV' without sanitization, allowing a crafted tag name to inject arbitrary environment variables.

Locations:

- `.github/workflows/build.yaml:25`
- `.github/workflows/release.yaml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned the Docker image in action.yml to its SHA digest (docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.1@sha256:7f470e9faecee45df54ae61de30d5afbc5d02127931d38135aa4b56036a6a344). Pinned docker/setup-qemu-action@v3 in build.yaml to full commit SHA c7c53464625b32c7a7e944ae62b3e17d2b600130.

2. script-injection: In build.yaml, moved ${{ github.actor }}, ${{ github.repository }}, and ${{ github.ref_name }} out of run: shell strings into env: blocks, referencing them as plain environment variables ($GITHUB_ACTOR, $GITHUB_REPOSITORY, $GITHUB_REF_NAME) in the shell scripts.

3. github-env-injection: In build.yaml, sanitized IMAGE_TAG_LOWER with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV. In release.yaml, sanitized the release name derived from GITHUB_REF_NAME with the same pattern before writing to $GITHUB_ENV.

