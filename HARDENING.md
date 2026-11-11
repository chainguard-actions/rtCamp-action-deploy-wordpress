<!-- markdownlint-disable -->

# Hardening Report: rtCamp--action-deploy-wordpress/v3.4.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rtCamp--action-deploy-wordpress/v3.4.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml runs.image refers to the Docker image by a mutable tag instead of a SHA digest: `image: 'docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.3'`. Pin it to an immutable digest, e.g. `docker://ghcr.io/rtcamp/action-deploy-wordpress@sha256:<digest> # v3.4.3`.

Locations:

- `action.yml:6`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned runs.image to docker://ghcr.io/rtcamp/action-deploy-wordpress:v3.4.3@sha256:73296f9aedfc536d515da3bb68c506daf431286313da778297b0d53100196708, keeping the docker:// scheme and the inline tag.

