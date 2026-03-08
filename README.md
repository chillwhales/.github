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

## Adding New Workflows

1. Create a new `.yml` file in `.github/workflows/` with `on: workflow_call`
2. Define typed inputs with sensible defaults
3. Keep workflows generic — no repo-specific logic
4. Update this README with the new workflow's inputs
5. Consumer repos reference via `uses: chillwhales/.github/.github/workflows/<name>@main`

## Versioning

All workflows are consumed via `@main`. For stability, consider tagging releases (e.g., `@v1`) once workflows stabilize. Breaking changes should be communicated across consumer repos before merging.
