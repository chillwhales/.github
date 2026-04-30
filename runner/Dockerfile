# Custom GitHub Actions runner image with Node 22 + pnpm + corepack baked in.
#
# Why: stock ghcr.io/actions/actions-runner:latest has no Node toolchain, so every
# job spends ~1m25s on actions/setup-node@v4 downloading + installing Node from
# scratch. Baking it into the image drops setup-node to ~2s (toolchain present;
# the action just resolves it).
#
# Tagging: published as ghcr.io/chillwhales/actions-runner:{latest,node22,node22-<sha>}
# from chillwhales/.github by .github/workflows/runner-image.yml. Helm values in
# k8s-cluster pin to the immutable node22-<sha> tag for deliberate upgrades.
FROM ghcr.io/actions/actions-runner:latest

ARG NODE_MAJOR=22
ARG PNPM_VERSION=10.18.2

USER root

# Base utilities + Node 22 LTS via NodeSource.
# git/curl/ca-certificates are required by most CI flows; build-essential
# covers native module compiles (better-sqlite3, etc.) without surprise failures.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
         ca-certificates \
         curl \
         git \
         gnupg \
         build-essential \
         python3 \
    && curl -fsSL "https://deb.nodesource.com/setup_${NODE_MAJOR}.x" | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Corepack ships with Node; activate the pinned pnpm version system-wide so
# every job sees the same pnpm regardless of packageManager field in repo.
RUN corepack enable \
    && corepack prepare "pnpm@${PNPM_VERSION}" --activate

USER runner

RUN node --version && pnpm --version
