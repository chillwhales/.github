# Custom Actions Runner Image

Cross-org GitHub Actions runner image with Node 22 + pnpm + corepack baked in.

## Why

Stock `ghcr.io/actions/actions-runner:latest` ships without a Node toolchain. Every job that uses `actions/setup-node@v4` spends ~1m25s downloading and installing Node from scratch on the ephemeral pod. Baking the toolchain into the image cuts that to ~2s.

## Image

- **Registry:** `ghcr.io/chillwhales/actions-runner`
- **Tags:**
  - `node22-<sha>` — immutable, recommended for Helm pins
  - `node22` — latest Node 22 build (mutable)
  - `latest` — alias for current default
- **Base:** `ghcr.io/actions/actions-runner:latest`
- **Includes:** Node 22 LTS, pnpm 10.18.2 (via corepack), git, curl, build-essential, python3

## Build

Published by `.github/workflows/runner-image.yml`:
- on push to `main` touching `runner/**`
- weekly cron rebuild for upstream base image patches
- manual `workflow_dispatch`

## Consumers

ARC `runnerScaleSetName: self-hosted` deployments in `k8s-cluster/infrastructure/actions-runner-set-*/values.yaml`. Pin `image.repository` + `image.tag` to an immutable `node22-<sha>` tag and bump deliberately.

## Updating pnpm version

Edit the `PNPM_VERSION` build arg in `Dockerfile`. Builds are cheap; let the cron rebuild pick it up or trigger `workflow_dispatch`.
