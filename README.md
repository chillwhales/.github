# chillwhales/.github

Org-wide GitHub Actions workflows and composite actions for the chillwhales organization.

## Composite Actions

### `setup-pnpm`

Checkout, install pnpm, setup Node.js with cache, and install dependencies. Eliminates 4-step boilerplate from every job.

**Location:** `.github/actions/setup-pnpm/action.yml`

| Input          | Required | Default | Description                                     |
| -------------- | -------- | ------- | ----------------------------------------------- |
| `node-version` | No       | `22`    | Node.js version to use                          |
| `fetch-depth`  | No       | `1`     | Number of commits to fetch (0 for full history) |

```yaml
steps:
  - uses: chillwhales/.github/.github/actions/setup-pnpm@main
    with:
      node-version: 22
```

### `build-and-upload`

Run a build command and upload the output as an artifact.

**Location:** `.github/actions/build-and-upload/action.yml`

| Input            | Required | Default        | Description                              |
| ---------------- | -------- | -------------- | ---------------------------------------- |
| `build-command`  | Yes      | —              | The build command to run                 |
| `artifact-name`  | No       | `build-output` | Name for the uploaded artifact           |
| `artifact-paths` | Yes      | —              | Newline-separated paths for the artifact |
| `retention-days` | No       | `1`            | Number of days to retain the artifact    |

```yaml
steps:
  - uses: chillwhales/.github/.github/actions/build-and-upload@main
    with:
      build-command: pnpm build
      artifact-paths: |
        packages/*/dist
```

## Reusable Workflows

### `ci-build-lint-test`

Full CI pipeline: install, format, lint, build, typecheck, test, and coverage reporting.

**Location:** `.github/workflows/ci-build-lint-test.yml`

| Input                    | Required | Default            | Description                                   |
| ------------------------ | -------- | ------------------ | --------------------------------------------- |
| `node-version`           | No       | `22`               | Node.js version                               |
| `build-command`          | Yes      | —                  | Build command                                 |
| `lint-command`           | No       | `''`               | Lint command (skipped if empty)               |
| `format-command`         | No       | `''`               | Format check command (skipped if empty)       |
| `test-command`           | No       | `''`               | Test command (skipped if empty)               |
| `typecheck-command`      | No       | `''`               | Typecheck command (skipped if empty)          |
| `pre-lint-command`       | No       | `''`               | Pre-lint build command (e.g., build deps)     |
| `test-node-versions`     | No       | `[20, 22]`         | JSON array of Node versions for test matrix   |
| `artifact-paths`         | Yes      | —                  | Newline-separated paths to upload after build |
| `coverage-artifact-name` | No       | `coverage-reports` | Name for coverage artifact                    |

### `ci-quality`

Package verification (publint, attw) and changeset status checks.

**Location:** `.github/workflows/ci-quality.yml`

| Input             | Required | Default | Description                                     |
| ----------------- | -------- | ------- | ----------------------------------------------- |
| `node-version`    | No       | `22`    | Node.js version                                 |
| `verify-command`  | No       | `''`    | Package verification command (skipped if empty) |
| `changeset-check` | No       | `true`  | Whether to run changeset status on PRs          |

### `ci-publish-validation`

Preview releases via pkg-pr-new on pull requests.

**Location:** `.github/workflows/ci-publish-validation.yml`

| Input              | Required | Default | Description                                 |
| ------------------ | -------- | ------- | ------------------------------------------- |
| `node-version`     | No       | `22`    | Node.js version                             |
| `preview-packages` | No       | `''`    | Space-separated package dirs for pkg-pr-new |

### `ci-changeset-check`

Standalone PR check that fails if no changeset is present for the diff against the base branch. Use this in repos that don't run the full `ci-quality` workflow (e.g. apps that don't publish packages but still want changeset-driven release notes).

**Location:** `.github/workflows/ci-changeset-check.yml`

| Input          | Required | Default | Description                          |
| -------------- | -------- | ------- | ------------------------------------ |
| `node-version` | No       | `22`    | Node.js version                      |
| `base-branch`  | No       | `main`  | Base branch for changeset comparison |

### `release-changesets`

Wraps `changesets/action@v1`. On push to the release branch, opens or updates a Version PR; when that PR merges, runs `publish-command` and exposes outputs so downstream jobs (e.g. Docker publish) can fan out per published package.

**Location:** `.github/workflows/release-changesets.yml`

| Input             | Required | Default                                 | Description                                                      |
| ----------------- | -------- | --------------------------------------- | ---------------------------------------------------------------- |
| `node-version`    | No       | `22`                                    | Node.js version                                                  |
| `version-command` | No       | `pnpm changeset version`                | Command Changesets runs to bump versions and write changelogs    |
| `publish-command` | No       | `''`                                    | Command Changesets runs to publish (e.g. `pnpm release`)         |
| `setup-command`   | No       | `''`                                    | Optional command run after install, before version/publish      |
| `pr-title`        | No       | `chore(release): version packages`      | Title for the Version PR                                         |
| `commit-message`  | No       | `chore(release): version packages`      | Commit message Changesets uses when versioning                   |

| Secret      | Required | Description                                                              |
| ----------- | -------- | ------------------------------------------------------------------------ |
| `NPM_TOKEN` | No       | npm auth token. Required when `publish-command` publishes to npm         |
| `GH_PAT`    | No       | PAT used by `changesets/action` to open the Version PR. Falls back to `GITHUB_TOKEN` |

| Output               | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| `published`          | `'true'` when changesets/action published one or more packages |
| `published-packages` | JSON array of `{ name, version }` for packages published     |

### `publish-docker-ghcr`

Builds a single Docker image and pushes it to GHCR with `:v<version>`, `:sha-<short>`, and (optionally) `:latest` tags. No-ops when `version` is empty so callers can wire it unconditionally and feed it directly from `release-changesets` outputs.

**Location:** `.github/workflows/publish-docker-ghcr.yml`

| Input         | Required | Default      | Description                                                    |
| ------------- | -------- | ------------ | -------------------------------------------------------------- |
| `image-name`  | Yes      | —            | Image name under `ghcr.io/<owner>/`. Lowercased automatically. |
| `version`     | No       | `''`         | Semver version. When empty, the job is skipped.                |
| `context`     | No       | `.`          | Docker build context                                           |
| `dockerfile`  | No       | `Dockerfile` | Path to the Dockerfile                                         |
| `platforms`   | No       | `linux/amd64`| Comma-separated build platforms                                |
| `build-args`  | No       | `''`         | Newline-separated `KEY=VALUE` build args                       |
| `push-latest` | No       | `true`       | Also tag and push `:latest`                                    |
| `registry`    | No       | `ghcr.io`    | Container registry host                                        |

Permissions required on the caller: `contents: read`, `packages: write`.

## Recommended Usage

Two ways consumer repos pick up these workflows:

1. **Org Ruleset (zero per-repo files)** — GitHub auto-runs the workflow on every targeted repo. Use for PR gates that need no repo-specific inputs. Configured once in the org's Rulesets UI under "Required Workflows", pointing at `chillwhales/.github/.github/workflows/<name>@main`. See `RULESET.md`.
2. **Caller workflow (`uses:`)** — repo commits a thin `.github/workflows/*.yml` that calls the reusable workflow with repo-specific inputs and secrets. Use whenever the workflow needs context (publish command, image name, dockerfile path).

### Pick by workflow

| Workflow                  | Mechanism            | Why                                                                        |
| ------------------------- | -------------------- | -------------------------------------------------------------------------- |
| `ci-changeset-check`      | Ruleset              | Same gate everywhere; no inputs needed beyond the default base branch       |
| `ci-build-lint-test`      | Caller               | Needs repo-specific build/lint/test/typecheck commands and artifact paths   |
| `ci-quality`              | Caller               | Needs repo-specific verify command                                          |
| `ci-publish-validation`   | Caller               | Needs repo-specific package list                                            |
| `release-changesets`      | Caller               | Triggered on `push` to main with repo-specific publish command + secrets    |
| `publish-docker-ghcr`     | Caller (fan-out)     | Needs per-image `image-name`, `context`, `dockerfile`                       |

### Recipes by repo type

**Library / package (publishes to npm, no Docker)**
- Ruleset: `ci-changeset-check`
- Caller `.github/workflows/ci.yml`: calls `ci-build-lint-test` + `ci-quality` + `ci-publish-validation`
- Caller `.github/workflows/release.yml`: calls `release-changesets` with `publish-command: pnpm release` and `NPM_TOKEN`

**App with one Docker image**
- Ruleset: `ci-changeset-check`
- Caller `.github/workflows/ci.yml`: calls `ci-build-lint-test`
- Caller `.github/workflows/release.yml`: calls `release-changesets`, then `publish-docker-ghcr` with the published version

**Monorepo with multiple Docker images (chillpass-style)**
- Ruleset: `ci-changeset-check`
- Caller `.github/workflows/ci.yml`: calls `ci-build-lint-test`
- Caller `.github/workflows/release.yml`: calls `release-changesets`, a `resolve-versions` job that maps published package names to versions, then a `publish-docker-ghcr` job per image — see the worked example below

### Conventions

- Always pin to `@main` for now; switch to `@v1` once tags exist.
- Always set `concurrency` on caller workflows to prevent overlapping runs on the same ref.
- Set `permissions: { contents: read, packages: write }` on any job that calls `publish-docker-ghcr`.
- Pass `NPM_TOKEN` only to `release-changesets`; never expose it to Docker jobs.
- Prefer empty `version` over conditional `if:` for Docker fan-out — the workflow no-ops cleanly when nothing was published.

## Example: Caller Workflow

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: chillwhales/.github/.github/workflows/ci-build-lint-test.yml@main
    with:
      build-command: pnpm build
      lint-command: pnpm lint
      format-command: pnpm format:check
      typecheck-command: pnpm typecheck
      test-command: pnpm test:coverage
      artifact-paths: |
        packages/*/dist

  quality:
    needs: [ci]
    uses: chillwhales/.github/.github/workflows/ci-quality.yml@main
    with:
      verify-command: pnpm validate:publish
      changeset-check: true
```

## Example: Release with multi-image Docker publish (chillpass-style)

For a monorepo that publishes a main app plus several services, run `release-changesets` once on push to `main`, then fan out to `publish-docker-ghcr` per image. Empty `version` inputs cause the publish job to no-op cleanly when there's nothing to release.

```yaml
name: Release

on:
  push:
    branches: [main]

concurrency:
  group: release-${{ github.ref }}
  cancel-in-progress: false

jobs:
  release:
    uses: chillwhales/.github/.github/workflows/release-changesets.yml@main
    with:
      setup-command: pnpm build
      publish-command: pnpm release
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

  resolve-versions:
    needs: [release]
    if: needs.release.outputs.published == 'true'
    runs-on: ubuntu-latest
    outputs:
      app: ${{ steps.pick.outputs.app }}
      api: ${{ steps.pick.outputs.api }}
      worker: ${{ steps.pick.outputs.worker }}
    steps:
      - id: pick
        env:
          PUBLISHED: ${{ needs.release.outputs.published-packages }}
        run: |
          jq_version() { echo "$PUBLISHED" | jq -r --arg n "$1" '.[] | select(.name == $n) | .version'; }
          {
            echo "app=$(jq_version @chillpass/app)"
            echo "api=$(jq_version @chillpass/api)"
            echo "worker=$(jq_version @chillpass/worker)"
          } >> "$GITHUB_OUTPUT"

  docker-app:
    needs: [resolve-versions]
    permissions:
      contents: read
      packages: write
    uses: chillwhales/.github/.github/workflows/publish-docker-ghcr.yml@main
    with:
      image-name: chillpass-app
      version: ${{ needs.resolve-versions.outputs.app }}
      context: apps/app
      dockerfile: apps/app/Dockerfile

  docker-api:
    needs: [resolve-versions]
    permissions:
      contents: read
      packages: write
    uses: chillwhales/.github/.github/workflows/publish-docker-ghcr.yml@main
    with:
      image-name: chillpass-api
      version: ${{ needs.resolve-versions.outputs.api }}
      context: services/api
      dockerfile: services/api/Dockerfile

  docker-worker:
    needs: [resolve-versions]
    permissions:
      contents: read
      packages: write
    uses: chillwhales/.github/.github/workflows/publish-docker-ghcr.yml@main
    with:
      image-name: chillpass-worker
      version: ${{ needs.resolve-versions.outputs.worker }}
      context: services/worker
      dockerfile: services/worker/Dockerfile
```

## Adding New Workflows

1. Create a new `.yml` file in `.github/workflows/` with `on: workflow_call`
2. Define typed inputs with sensible defaults
3. Keep workflows generic — no repo-specific logic
4. Update this README with the new workflow's inputs
5. Consumer repos reference via `uses: chillwhales/.github/.github/workflows/<name>@main`

## Versioning

All workflows are consumed via `@main`. For stability, consider tagging releases (e.g., `@v1`) once workflows stabilize. Breaking changes should be communicated across consumer repos before merging.
